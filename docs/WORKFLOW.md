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
[['report',renderReport],['bench',renderBench],['eng',renderEng],['ace',renderAce],
 ['camp',renderCampaigns],['rules',renderRules],['variants',renderVariants],['guard',renderGuardrails],
 ['lc',renderLifecycle],['conv',renderConv],['wiz',renderWiz],['loy',renderLoyalty]]
 .forEach(([k,f])=>{try{f();out[k]='ok'}catch(e){out[k]=e.message.slice(0,60)}});
const n=s=>document.querySelectorAll(s).length;
out._dom={tiles:n('#repTiles .tile'),chart:n('#repChart svg'),rev:n('#revTable tbody tr'),
 grList:n('#grList .lcItem'),grDetail:n('#grDetail .card'),lc:n('#lcList .lcItem'),
 levers:n('#lcDetail .lever'),modes:n('.modeBig'),
 domRows:n('#aceDomainTable tbody tr.domRow'),playRows:n('#aceDomainTable tbody tr.playRow'),
 conv:n('#convItems .convItem'),tiers:n('#tierTable tbody tr'),eng:n('#engTable tbody tr')};
return JSON.stringify(out)})()
```

Every key must be `ok` and every `_dom` count must be greater than zero. Anything else means a
screen is blank. `playRows` at zero while `domRows` is 8 is the specific failure of a collapsed
`aceOpen` — the probe runs against a fresh load, where every theme starts expanded, so it should
read 8 and 26.

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

## Sweeping the reporting surface

Ten metrics × five dimensions × three views is 150 states, and a broken one is invisible until
someone lands on it. Sweep them all, and check the projections agree:

```js
(()=>{const errs=[];
Object.keys(METRICS).forEach(mk=>Object.keys(DIMS).forEach(dk=>['time','rank','table'].forEach(v=>{
  rep.metric=mk;rep.dim=dk;rep.view=v;
  try{renderReport(); if(!document.querySelectorAll('#repChart svg, #repTable table').length)
    errs.push(mk+'/'+dk+'/'+v+' rendered nothing')}
  catch(e){errs.push(mk+'/'+dk+'/'+v+': '+e.message)}})));
rep.metric='gmv';rep.dim='source';rep.view='time';renderReport();
const tbl=dimTotals('source').reduce((a,r)=>a+r.gmv,0);
const ser=Math.round(repSeries().reduce((a,s)=>a+s.total,0));
return JSON.stringify({errors:errs.length, first:errs[0]||null, totalsAgree:Math.abs(tbl-ser)<2})})()
```

`errors: 0` and `totalsAgree: true`. The second one is the real test — it proves the chart and the
revenue table are still two projections of the same primitives rather than two hand-maintained
datasets that have drifted apart.

## Checking the two layers still agree

`ACE_DOMAINS` carries no metrics — every theme figure is folded from its plays. If you edit a play's
numbers, nothing else needs editing, but it is worth confirming the fold still behaves. This checks
against every play being on, so it won't itself flag a play you've manually switched off — that's
expected, `domTotals` is supposed to exclude it:

```js
(()=>ACE_DOMAINS.map(d=>{const t=domTotals(d,true),p=domPotential(d);
  const sum=d.plays.filter(x=>x.st!=="setup"&&x.on!==false).reduce((a,x)=>a+(x.reach||0),0);
  return {n:d.n, foldedReach:t?Math.round(t.reach):null, handSum:d.on?sum:null,
          agree:!d.on||Math.round(t.reach)===sum, pot:Math.round(p.gmv)}}))()
```

`agree` must be true on every row. A play with `st:"setup"` or with its own switch off is excluded
from the live fold but stays in the potential — that is deliberate, it is how a blocked integration
or a declined play gets a price tag.

## Changing a chart colour

Re-run the dataviz validator; the current four-colour set passes all six checks on all pairs and
you should not lose that. From the `dataviz` skill directory:

```bash
node scripts/validate_palette.js "#7e55f6,#c35e4a,#149db8,#0e4ea7" --mode light --pairs all
```

Every check must PASS. **Use `--pairs all`, not the default** — the earlier three-colour set passed
adjacent-pairs and still had a blue/purple pairing readers couldn't separate in the line chart,
where every series is compared against every other. A contrast WARN is not dismissable; it
obligates visible labels or a table view.

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
4. Sweep the reporting surface (above)
5. Click the thing you changed, plus all three studio shapes (a theme instruction, a play, a custom
   engagement — they render different When blocks; the custom-engagement one also has the
   sentence/rules toggle), a theme row's click-to-expand and a play's own switch, the Guardrails
   category list and the Lifecycle stage detail
6. Load the page with a page name in the hash (e.g. `#loyalty`) and confirm it opens straight there;
   click a nav item and confirm the address bar's hash updates to match
7. Push, then confirm live == local

## A note on scope

The prototype is a spec you can operate. Copy is part of the deliverable, not decoration — most of
the arguments in [PRODUCT.md](PRODUCT.md) are carried by a sentence on a card rather than by a
diagram. If you change a screen, change the sentence that explains it.
