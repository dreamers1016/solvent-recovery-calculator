# Solvent Recovery ROI Calculator

A browser-based decision support tool for chemical plant operators evaluating solvent recovery equipment investments.

**Live app:** https://solvent-recovery-calculator-theta.vercel.app

---

## Overview

This tool calculates the return on investment (ROI) for solvent recovery equipment over a 10-year horizon. It accounts for annual price volatility, equipment downtime, operating costs, disposal savings, and capital expenditure to produce payback period, NPV, IRR, and cumulative net savings projections.

---

## Features

- **Solvent library** — Acetone, Ethanol, Toluene, Methanol, DCM, Hexane, IPA, or custom solvent with user-defined price and disposal cost
- **Three equipment options** — Distillation unit, Activated carbon adsorption, Membrane separation
- **Custom equipment parameters** — Override capital cost, operating rate, recovery efficiency, and maintenance for any equipment type; defaults are retained if left blank
- **10-year cash flow table** — Year-by-year recovery value, operating cost, net savings, and cumulative balance with price compounding
- **10-year comparison chart** — Cumulative net savings for all three equipment options with legend, axis labels, and break-even line
- **Compounding price volatility** — Annual price growth/decline rate applied as a compound factor each year
- **Equipment downtime modelling** — Downtime % reduces effective recovery efficiency; hover tooltip on Net Savings explains the live impact in numbers
- **Multi-currency display** — USD, EUR, GBP, MYR with live conversion
- **What-if scenarios** — One-click ±20% price multiplier buttons
- **Plant-size presets** — Small (500 L/yr), Medium (2,000 L/yr), Large (5,000+ L/yr)
- **Disposal method selection** — Incineration, landfill, or waste contractor

---

## Usage

Open `index.html` directly in any modern browser — no build step, no dependencies to install. All logic runs client-side.

### Inputs

| Parameter | Description |
|-----------|-------------|
| Solvent | Select from library or enter custom price/disposal cost |
| Annual consumption | Drag slider from 100 to 10,000 L/yr |
| Disposal method | Affects disposal cost multiplier |
| Equipment selection | Click a card to select; click "Customise parameters" to override defaults |
| Annual price growth rate | Slider from −10% to +10%/yr; compounds over 10 years |
| Equipment downtime | Slider from 0% to 15%; reduces effective recovery efficiency |

### Key Outputs

| Metric | Description |
|--------|-------------|
| Payback period | Capital cost ÷ annual net savings |
| Net savings (10-yr) | Total recovery value − total op. costs − capex |
| 10-yr ROI | Net savings ÷ capex × 100% |
| IRR | Internal rate of return solving NPV = 0 over 10 years with variable cash flows |
| NPV | Net present value at 8% discount rate |

---

## Technical Notes

### Price volatility compounding

The annual price growth rate `r` is applied as a compound factor per year:

```
Year y price = Base price × (1 + r)^y
```

At `r = +5%/yr`, the solvent price in Year 10 is 1.629× the base price. At `r = −5%/yr`, it is 0.599×.

### Downtime impact on net savings

Equipment downtime reduces the fraction of operating time available for recovery:

```
Effective efficiency = Base efficiency × (1 − downtime rate)
Annual recovered volume = Annual consumption × Effective efficiency
```

A 10% downtime on a unit with 94% base efficiency yields 84.6% effective efficiency. The hover tooltip on the "Net savings (10-yr)" metric shows the exact volume lost and annual value impact for the current settings.

### Equipment capital cost scaling

Capital cost scales with plant volume between the equipment's low and high cost bounds:

```
Cap cost = capLow + (capHigh − capLow) × min(1, vol / 5000)
```

This can be overridden per equipment via the "Customise parameters" inputs.

### IRR estimation

IRR is solved by binary search (60 iterations) over the range −50% to 500%, finding the discount rate at which the NPV of the variable annual cash flows equals zero.

---

## Equipment Defaults

| Option | Type | Cap cost range | Op. rate range | Efficiency range | Maintenance |
|--------|------|---------------|----------------|-----------------|-------------|
| Distillation unit | A | $50K–$150K | $0.12–$0.18/L | 92–96% | 8 hrs/mo |
| Activated carbon adsorption | B | $15K–$40K | $0.08–$0.14/L | 85–90% | 4 hrs/mo |
| Membrane separation | C | $80K–$200K | $0.15–$0.22/L | 94–98% | 6 hrs/mo |

All values use the midpoint of each range unless overridden by the user.

---

## Deployment

The app is deployed as a static site on Vercel. Any push to the `master` branch triggers an automatic redeploy.

```bash
vercel --prod
```

---

## Disclaimer

Results are estimates only. The 10-year price volatility adjustment compounds annually. Equipment downtime reduces effective recovery efficiency each year. Consult equipment vendors for exact specifications, site-specific costs, and regulatory compliance requirements before making investment decisions.
