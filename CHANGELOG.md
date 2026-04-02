# Changelog

All notable changes to the Solvent Recovery ROI Calculator are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.1.0] — 2026-04-02

### Added
- **10-year cash flow horizon** — Table and chart extended from 5 to 10 years
- **Compounding price volatility** — Annual price growth rate slider (−10% to +10%/yr) applied as a compound factor `(1 + r)^y` per year, replacing the previous flat one-time adjustment
- **Downtime impact tooltip** — Hover/focus on the ⓘ icon beside "Net savings (10-yr)" to see live figures: efficiency drop in percentage points, unrecovered volume per year, and annual value lost
- **Equipment custom parameter inputs** — Each equipment card now contains editable fields for capital cost ($), operating rate ($/L recovered), recovery efficiency (%), and maintenance (hrs/mo). Original proposed values are shown as placeholders and used as defaults when fields are left blank
- **Chart legend** — All three equipment options and the break-even line are now labelled in the chart
- **Chart title** — Chart heading reflects the active price growth rate (e.g. "10-Year Horizon (+3%/yr price growth)")
- **Axis labels** — X-axis (Year) and Y-axis (Cumulative currency) labels added to the comparison chart
- **Sticky table headers** — Cash flow table header row stays visible while scrolling through 10 years of data
- **Scrollable cash flow table** — Table container capped at 310 px height with vertical scroll

### Changed
- Price volatility slider range changed from ±20% (flat) to ±10%/yr (compounding)
- What-if buttons (±20%) now apply a flat scenario multiplier on the base price independently of the annual growth rate slider
- IRR calculation updated to use variable annual cash flows reflecting compounding price rather than a fixed annual net
- All 10-yr summary metrics updated: "5-yr recovery value", "5-yr operating cost", "Net savings (5-yr)", "5-yr ROI" → 10-yr equivalents
- Disclaimer updated to note compounding price behaviour

### Fixed
- Custom equipment parameter values now persist correctly across recalculations (stored in `EQUIP_CUSTOM` array, not DOM state)
- `renderEquipCardsDisplay()` introduced to update card display values without destroying input fields, preventing loss of user-typed parameters on slider/dropdown changes

---

## [1.0.0] — 2026-04-02

### Added
- Initial release of Solvent Recovery ROI Calculator
- Solvent library: Acetone, Ethanol, Toluene, Methanol, DCM, Hexane, IPA, Custom
- Three recovery equipment options: Distillation unit (A), Activated carbon adsorption (B), Membrane separation (C)
- 5-year cash flow table with annual recovery value, operating cost, net savings, and cumulative balance
- Cumulative net savings chart comparing all three equipment options
- Capital cost scaling by plant volume between equipment low/high bounds
- Equipment downtime slider (0–15%) reducing effective recovery efficiency
- Price volatility adjustment slider (±20% flat)
- What-if scenario buttons: price +20%, price −20%, reset
- Multi-currency support: USD, EUR, GBP, MYR
- Plant-size presets: Small (500 L/yr), Medium (2,000 L/yr), Large (5,000+ L/yr)
- Disposal method selector: incineration, landfill, waste contractor
- Key metrics: payback period, net savings, ROI, IRR, NPV at 8% discount rate
- Risk badge on each equipment card (Low / Medium / Higher risk)
- Deployed to Vercel as a static site with GitHub auto-deploy
