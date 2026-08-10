# Working on this prototype

Three failure modes have actually happened in this repo. All three are cheap to check for and
expensive to miss, so the runbook exists mostly to catch them.

---

## The boot chain will silently eat whole screens

`/* boot */` calls the render functions in sequence. There is no error handling. **If one throws,
every render after it never runs** — and the symptom is not an error the user sees, it's blank
screens further down the page.

This has happened twice. Once when a DOM node was deleted (`#aceChain`, `#waveTable`) but the render
function still wrote to it. `renderAce` threw, and Guardrails, Lifecycle, Conversations and Loyalty
were all empty. Nothing in the UI said why.

**So after every change, run this in the browser console** (or via a browser tool). It calls each
render function in isolation and reports which one breaks:

```js
(()=>{const out={};
[['bench',renderBench],['rev',renderRev],['eng',renderEng],['ace',renderAce],['camp',renderCampaigns],
 ['rules',renderRules],['variants',renderVariants],['guard',renderGuardrails],['excl',renderExclusions],
 ['lc',renderLifecycle],['conv',renderConv],['wiz',renderWiz],['loy',renderLoyalty]]
 .forEach(([k,f])=>{try{f();out[k]='ok'}catch(e){out[k]=e.message.slice(0,60)}});
const n=s=>document.querySelectorAll(s).length;
out._dom={lc:n('#lcList .lcItem'),levers:n('#lcDetail .lever'),modes:n('.modeBig'),
 dom:n('#aceDomainTable tbody tr'),conv:n('#convItems .convItem'),tiers:n('#tierTable tbody tr'),
 eng:n('#engTable tbody tr'),excl:n('#exclusionList > div')};
return JSON.stringify(out)})()
```

Every key must be `ok` and every `_dom` count must be greater than zero. Anything else means a
screen is blank.

## Check the JS parses before you look at anything

The script is one big block; a syntax error means nothing renders at all and the page just sits
there. Cheapest possible check, no browser needed:

```bash
node -e "const m=require('fs').readFileSync('index.html','utf8').match(/<script>([\s\S]*)<\/script>/);try{new Function(m[1]);console.log('JS parses OK')}catch(e){console.log('ERR',e.message)}"
```

## The preview pane serves a stale snapshot

A `file://` page in a preview pane re-snapshots when an editor tool writes the file. If you edit via
a **shell script** (`python3`, `sed`) the pane does not know, and you will debug a version of the
file that no longer exists on disk. This produced a phantom `CAP_UNITS is not defined` that made no
sense against the source.

**Always force a reload from disk after a shell-side edit** before you believe anything you see.
Console error buffers also survive reloads, so an error you already fixed keeps reappearing — trust
the isolation probe above over the error list.

---

## Editing safely

Large structural changes are easier as a line-range splice than as a text match, because the file has
a lot of near-identical markup. The pattern that works:

```python
python3 - <<'EOF'
p="index.html"
lines=open(p).read().split("\n")
new=open("/tmp/new_block.html").read().rstrip("\n").split("\n")
assert "MARKER I EXPECT" in lines[440], repr(lines[440])   # assert both ends
assert lines[582].strip()=="</section>", repr(lines[582])
open(p,"w").write("\n".join(lines[:440]+new+lines[583:]))
EOF
```

Assert on both boundaries. Every time an assert was skipped, the splice landed in the wrong place.

## Changing a chart colour

Re-run the dataviz validator; the current three-colour set passes all six checks and you should not
lose that. From the `dataviz` skill directory:

```bash
node scripts/validate_palette.js "#0d6cf2,#c35e4a,#7e55f6" --mode light
```

Every check must PASS. A contrast WARN is not dismissable — it obligates visible labels or a table
view.

---

## Deploying

`main` is served by GitHub Pages at
https://max-pruv.github.io/journey-engagement-center-prototypes/ — pushing is deploying.

```bash
git add -A && git commit -m "…" && git push origin main
```

Then **verify the live page is byte-identical to your local file** rather than assuming the build
picked it up. Pages takes 30–90 seconds and gives no signal:

```bash
for i in $(seq 1 10); do
  live=$(curl -s https://max-pruv.github.io/journey-engagement-center-prototypes/ | shasum -a 256 | cut -d' ' -f1)
  local=$(shasum -a 256 index.html | cut -d' ' -f1)
  if [ "$live" = "$local" ]; then echo "DEPLOYED — live == local"; break; fi
  sleep 15
done
```

## Before you call it done

1. `node -e …` — JS parses
2. Force-reload from disk
3. The isolation probe — all `ok`, all counts non-zero
4. Click the thing you changed, plus the studio and the Lifecycle stage detail (the two most fragile)
5. Push, then confirm live == local

## A note on scope

The prototype is a spec you can operate. Copy is part of the deliverable, not decoration — most of
the arguments in [PRODUCT.md](PRODUCT.md) are carried by a sentence on a card rather than by a
diagram. If you change a screen, change the sentence that explains it.
