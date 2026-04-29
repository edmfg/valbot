# Orbit — Audience Constellation Prototype

## Build instructions for Claude Code

- Build as a **single `index.html` file**. No build step. No external bundler.
- All HTML, CSS, JavaScript inline in one file.
- Deploy target: **GitHub Pages** (just push and serve).
- External libraries via CDN only (e.g. `cdnjs.cloudflare.com`).
- Use **D3.js** for the constellation force simulation (`d3.forceSimulation`, `d3.drag`).
- All data is **fake/illustrative** — generated client-side, deterministic per filter combo so the same selection always produces the same constellation.
- No backend, no API calls, no localStorage (use in-memory state only).
- Should run by opening `index.html` directly in a browser.

---

## Product concept

A media planning prototype for **Google's B2B Small Business** audience targeting. Visualizes the relationship between:

1. **Audience** — defined by seniority, sector, company size, region
2. **LinkedIn content** — the business topics this audience engages with
3. **Reddit content** — the cultural/lifestyle contexts the same audience inhabits outside of work

Goal: identify both **business contexts** (LinkedIn) and **non-business contexts** (Reddit) to reach a target audience with media — bridging professional intent and human identity.

---

## Aesthetic

- **Apple / Teenage Engineering minimalist**: generous whitespace, restrained color, iconic.
- White surfaces, 0.5px borders, flat fills. No gradients, no shadows.
- Typography: sans-serif (Inter or system stack). Sentence case everywhere.
- Two font weights only: 400 regular, 500 medium.
- The constellation is the hero — UI chrome should recede.

---

## Layout

```
┌──────────────────────────────────────────────────────────┐
│  ORBIT · v0.1                  [Filter] [Graph] [Compare]│
│  Audience constellation              [Export] [Notes]    │
├──────────────────────────────────────────────────────────┤
│  Seniority ▾   Sector ▾   Company size ▾   Region ▾      │
├──────────────────────────────────────────────────────────┤
│  [Compare bar — only shown in Compare mode]              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│           ·  LinkedIn topic                              │
│      ·                          · Reddit topic           │
│              ◉  Audience                                 │
│         ·         (center)              ·                │
│                                                          │
│   ·                    ·                                 │
│                                                          │
├──────────────────────────────────────────────────────────┤
│  Hover · preview   Click · drill   Dbl-click · re-pivot  │
└──────────────────────────────────────────────────────────┘
```

---

## Filters (4 dropdowns)

| Filter | Options |
|---|---|
| Seniority | C-suite, VP, Director, Manager, Founder |
| Sector | Fashion retail, Food & beverage, Professional services, Health & wellness, Home services, SaaS / tech |
| Company size | 1–10, 11–50, 51–200 |
| Region | US, EU, APAC |

Default: `C-suite / Fashion retail / 11–50 / US`.

When any filter changes, the constellation re-animates to the new state (D3 force simulation transition).

---

## Constellation visualization

- **Center node**: audience (large circle, gray, gently pulsing).
- **Orbiting nodes**: top 10 LinkedIn topics + top 10 Reddit topics.
- **LinkedIn nodes**: blue (`#185FA5`).
- **Reddit nodes**: orange/coral (`#D85A30`).
- **Affinity strength encoded redundantly**:
  - Distance from center (closer = stronger)
  - Node size (larger = stronger)
  - Connecting line thickness (thicker = stronger)
- D3 force simulation: `forceLink` with distance based on affinity score, `forceManyBody` for repulsion, `forceCenter` for center anchor.
- Nodes are **draggable** (D3 `drag` behavior); positions persist until filter change.

---

## Interactions

| Gesture | Behavior |
|---|---|
| **Hover** node | Show floating card with topic name, affinity score, sample headline/post |
| **Click** node | Drill down — node expands into 3–5 sub-topics that orbit it |
| **Double-click** node | Re-pivot — that node becomes the new center, audience drops to satellite |
| **Drag** node | Rearrange position; simulation respects new placement |
| **Filter change** | Re-animate to new constellation |

---

## Modes (top-right toggle)

### Filter mode (default)
Standard filter-driven view as described above.

### Graph mode
Same data, but no filters applied — show **all** audiences and content as a single network. Useful for "show me the whole map" exploration. Audiences are larger gray nodes, content is sized by total cross-audience affinity.

### Compare mode
Side-by-side. Two filter sets (A and B). Show two constellations stacked or side-by-side. Highlight overlapping content nodes.

---

## Pitch features

- **Export**: download current constellation as PNG (use `html2canvas` or SVG-to-canvas) + a shareable URL with filter state encoded as query params.
- **Notes**: collapsible textarea below the canvas for adding annotations during a stakeholder walkthrough.
- **Reset orbit**: button to re-run the force simulation from scratch.

---

## Fake data structure

Hard-code a lookup keyed by `${sector}|${seniority}` with override fallback. Example seeds:

### LinkedIn topics

```js
{
  'Fashion retail|C-suite': [
    'DTC margin compression','Resale & circular fashion','AI try-on tech',
    'Tariff exposure 2026','Influencer ROI debates','Store fleet rationalization',
    'Gen Z brand loyalty','Wholesale vs DTC','Inventory turn benchmarks','Founder mode essays'
  ],
  'Fashion retail|VP': [
    'Shopify Plus migrations','Klaviyo email LTV','TikTok Shop attribution',
    'RFID & shrinkage','Visual merch case studies','Returns automation',
    'Influencer contract terms','POS unification','3PL vs in-house','CRO experiments'
  ],
  'Food & beverage|C-suite': [
    'Cold-chain economics','Private label threats','GLP-1 demand shock',
    'Plant-based reset','Ghost kitchen ROI','Commodity hedging',
    'Restaurant M&A','Chick-fil-A culture','Loyalty program math','Tip credit policy'
  ],
  'SaaS / tech|C-suite': [
    'PLG vs sales-led','Rule of 40 in 2026','AI cost gross margin',
    'Vibe coding ops','Series B drought','RIF playbooks',
    'GTM compression','PMF re-finding','Multi-product expansion','Founder/CEO transitions'
  ],
  'default': [
    'Hiring freeze tactics','AI productivity studies','Q3 earnings reads',
    'Remote work data','SMB lending shifts','Founder fatigue',
    'Pricing experiments','Talent density','Board management','Year-end planning'
  ]
}
```

### Reddit topics (subreddit + sample post)

```js
{
  'Fashion retail|C-suite': [
    ['r/malefashionadvice','grail-hunting Saturday'],
    ['r/femalefashionadvice','capsule wardrobe'],
    ['r/AskNYC','best omakase under $200'],
    ['r/FrugalMaleFashion','Aimé Leon Dore drop'],
    ['r/Watches','Tudor BB58 vs GMT'],
    ['r/Sneakers','Sambas dead?'],
    ['r/wine','Burgundy 2022 thread'],
    ['r/HENRYfinance','$1M HHI but feel poor'],
    ['r/Padel','first racquet'],
    ['r/F1','Verstappen contract drama']
  ],
  'Food & beverage|C-suite': [
    ['r/restaurateur','tip pooling'],
    ['r/AskCulinary','butcher cuts'],
    ['r/winemaking','garage Cab'],
    ['r/GLP1','Mounjaro side effects'],
    ['r/Coffee','Eight Ounce vs Trade'],
    ['r/HENRYfinance','school district cost'],
    ['r/tennis','Sinker forehand'],
    ['r/golf','Liv schedule'],
    ['r/woodworking','Festool vs Mafell'],
    ['r/F1','team radio']
  ],
  'SaaS / tech|C-suite': [
    ['r/ChatGPTPro','prompt libraries'],
    ['r/AskHistorians','Roman empire ops'],
    ['r/F1','Adrian Newey'],
    ['r/golf','Scotty Cameron'],
    ['r/HENRYfinance','VC carry tax'],
    ['r/Padel','Madrid open'],
    ['r/Watches','GS spring drive'],
    ['r/winemaking','barrel selection'],
    ['r/AskNYC','best steak'],
    ['r/Skiing','heli vs cat']
  ],
  'default': [
    ['r/personalfinance','HYSA arms race'],
    ['r/AskMen','dating in 30s'],
    ['r/Frugal','grocery prices'],
    ['r/Cooking','Sunday roast'],
    ['r/F1','Sunday GP'],
    ['r/houseplants','overwatering'],
    ['r/Sneakers','what to flip'],
    ['r/HENRYfinance','feeling broke'],
    ['r/podcasts','Acquired ep'],
    ['r/travel','shoulder season']
  ]
}
```

### Affinity scoring

Generate a **deterministic pseudo-random score** per topic based on filter combo so:
- Same filter selection → same constellation every time
- Different selections → different but plausible spreads

```js
const seed = (seniority+sector+size+region).split('').reduce((a,c)=>a+c.charCodeAt(0),0);
const rand = (i) => ((seed*9301 + i*49297) % 233280) / 233280;
// LinkedIn affinity: 0.55–1.00 (stronger on average)
// Reddit affinity: 0.35–0.90 (more varied)
```

Add 3–5 child sub-topics per parent for the click-drill behavior. Generate these with a similar deterministic approach.

---

## Tech stack

- **D3.js v7** — force simulation, drag behavior, SVG rendering
- **Vanilla JS** — no React, no build step
- **Inline SVG** for the constellation
- **CSS variables** for theming (light mode primary; dark mode optional)
- **html2canvas** (optional, for export-as-PNG)

---

## Build sequence (suggested for Claude Code)

1. Scaffold `index.html` with header, filter row, canvas container, footer.
2. Implement filter dropdowns + state object.
3. Wire up the fake-data generator function — make sure same filters → same output.
4. Render static constellation (audience center + 20 satellite nodes) with D3 force simulation.
5. Add hover preview card.
6. Add click-to-drill (sub-topic expansion).
7. Add double-click re-pivot.
8. Add drag behavior.
9. Add mode toggle (Filter / Graph / Compare).
10. Add Export + Notes pitch features.
11. Polish animations (filter-change transitions, pulse on center node).
12. Push to GitHub Pages.

---

## Out of scope (for v0.1)

- Real LinkedIn or Reddit data ingestion
- Authentication / user accounts
- Saved configurations beyond URL state
- Multi-user collaboration
- Mobile-optimized layout (desktop-first for the pitch)
