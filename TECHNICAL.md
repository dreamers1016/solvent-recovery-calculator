# Technical Reference

Internal documentation covering the calculation logic, data model, and architecture of the Solvent Recovery ROI Calculator.

---

## Architecture

The app is a **single-file static HTML application** (`index.html`). There is no build step, no framework, and no server-side logic. All calculations run in the browser using vanilla JavaScript.

```
index.html
├── <style>        — All CSS (inline, minified-style)
├── <body>         — All HTML markup
└── <script>       — All application logic
```

**External dependency:** Chart.js 4.4.1 loaded from CDN (`cdnjs.cloudflare.com`).

---

## Data Model

### Solvent (`SOLVENTS`)

```js
{
  name: string,
  bp: number,       // Boiling point °C
  mw: number,       // Molecular weight g/mol
  price: number,    // Market price $/L (USD base)
  disposal: number  // Disposal cost $/L (USD base)
}
```

Custom solvent reads `price` and `disposal` directly from user inputs.

### Equipment (`EQUIP`)

```js
{
  name: string,
  type: 'A' | 'B' | 'C',
  capLow: number,    // Min capital cost $ at low volume
  capHigh: number,   // Max capital cost $ at high volume
  opLow: number,     // Min operating cost rate $/L recovered
  opHigh: number,    // Max operating cost rate $/L recovered
  effLow: number,    // Min recovery efficiency (0–1)
  effHigh: number,   // Max recovery efficiency (0–1)
  setupLow: number,  // Min setup time
  setupHigh: number, // Max setup time
  setupUnit: string, // 'months' or 'weeks'
  maint: number      // Maintenance hours per month
}
```

All calculations use the **midpoint** of each range unless the user overrides via `EQUIP_CUSTOM`.

### Custom Overrides (`EQUIP_CUSTOM`)

```js
[
  { cap: number|null, op: number|null, eff: number|null, maint: number|null },
  // ... one entry per equipment option
]
```

`null` means "use the computed default". User inputs write to this array via `updateCustom(ei, field, val)`. Values survive full re-renders because they are stored in JS state, not the DOM.

---

## Core Calculation Functions

### `defaultCapCost(e, vol)`

Scales capital cost linearly with plant volume between `capLow` and `capHigh`, saturating at 5,000 L/yr:

```
cap = capLow + (capHigh − capLow) × min(1, vol / 5000)
```

### `calcEquip(e, vol, price, dispCost, dispMult, ei)`

Returns all annualised metrics for one equipment option at a given base price.

| Output field | Formula |
|---|---|
| `baseEff` | `(effLow + effHigh) / 2` or custom |
| `effAdj` | `baseEff × (1 − downtime)` |
| `recovered` | `vol × effAdj` |
| `annualRecovery` | `recovered × price` |
| `annualOp` | `recovered × opRate × 1.10 + 8 × maint × 12` |
| `annualNet` | `annualRecovery − annualOp` |
| `paybackYrs` | `cap / annualNet` (999 if `annualNet ≤ 0`) |

The `1.10` factor on operating cost accounts for a 10% maintenance overhead. The `8 × maint × 12` term is a fixed annual maintenance labour cost at $8/hr.

### `renderAnalysis()` — 10-year projection

For the selected equipment, iterates over years 1–10:

```
Year y price   = basePrice × (1 + volRate)^y
Year y recVal  = vol × effAdj × yearPrice
Year y opCost  = annualOp   (constant — does not inflate)
Year y net     = recVal − opCost
Cumulative_y   = Cumulative_{y-1} + net
```

`basePrice = sol.price × whatIfMult`  
`volRate` = annual price growth rate from slider (e.g. 0.03 for +3%/yr)  
`effAdj` = constant across years (downtime applied once at base price year)

The same loop runs for all three equipment options to produce the comparison chart dataset.

### `estimateIRR(r, basePrice, vol, cap, yrs, volRate)`

Binary search over discount rate `d ∈ [−0.5, 5.0]` (60 iterations):

```
NPV(d) = −cap + Σ [ (vol × effAdj × basePrice × (1+volRate)^y − annualOp) / (1+d)^y ]
               y=1..yrs
```

Returns `d × 100` (percentage). Converges to ~10 significant figures in 60 iterations.

---

## State Variables

| Variable | Type | Description |
|---|---|---|
| `curCode` | string | Active currency code (USD/EUR/GBP/MYR) |
| `curSymbol` | string | Active currency symbol |
| `curFX` | number | FX rate relative to USD |
| `whatIfMult` | number | One-time price scenario multiplier (default 1.0) |
| `selectedEquip` | number | Index of selected equipment (0/1/2) |
| `roiChartInst` | Chart\|null | Chart.js instance; destroyed and recreated on each recalc |
| `EQUIP_CUSTOM` | array | Per-equipment custom parameter overrides |

---

## Render Flow

```
User input change
      │
      ▼
  recalc()
   ├── Update top metric cards (disposal cost, solvent value, etc.)
   ├── renderEquipCards()          ← Full DOM re-render of all 3 cards
   │    └── Restore EQUIP_CUSTOM values into new input elements
   └── renderAnalysis()            ← Rebuild table rows + destroy/recreate chart

updateCustom(ei, field, val)       ← Called only when equipment param input changes
   ├── Write to EQUIP_CUSTOM[ei]
   ├── renderEquipCardsDisplay()   ← Partial update (display values only, no DOM re-render)
   └── renderAnalysis()
```

`renderEquipCardsDisplay()` exists specifically to avoid destroying user-typed input values when downtime or price sliders move. It updates only the badge, stat values, and payback display inside each existing card element without touching the input fields.

---

## Currency Conversion

All internal values are stored and calculated in USD. The `fmt(v)` function applies `curFX` at display time:

```js
function fmt(v, dec=0) {
  const c = v * curFX;
  // format with M/K suffix or plain decimal
}
```

FX rates are fixed constants (`FX` object) and do not update from live data.

---

## Known Limitations

- **Operating cost inflation** — Annual operating costs are held constant across 10 years. In practice these would inflate alongside energy and labour costs.
- **Fixed downtime rate** — Downtime is applied as a constant annual rate. Real-world equipment downtime typically increases with age.
- **Fixed discount rate** — NPV uses a hardcoded 8% discount rate (WACC proxy). No input to change this.
- **No tax or depreciation** — The model does not account for capital allowances, depreciation schedules, or tax effects on net savings.
- **FX rates are static** — Currency conversion uses fixed rates; no live FX feed.
- **Single solvent assumption** — The calculator assumes one solvent per session. Mixed-solvent streams are not modelled.
