---
name: full-valuation-model
description: "Builds a comprehensive stock valuation Excel workbook for any publicly traded ticker. Produces a single .xlsx file with five tabs: (1) Comparable Company Analysis with peer trading multiples and statistics, (2) DCF Model with Bear/Base/Bull scenarios using EBIT-based unlevered FCF, sensitivity tables, and equity bridge, (3) Sensitivity Analysis with three two-way tables, (4) Conservative DCF using OCF-CapEx method with value investing principles (10-year projection, two growth phases, end-of-year discounting), (5) Dhandho DCF using the Pabrai framework - two-phase FCF growth, sale-price terminal value, excess cash addback, margin of safety, and a Reverse Dhandho showing the market-implied growth rate. Uses yfinance MCP and EDGAR MCP for all data - no web search. Use when a user asks to value a stock, build a DCF, run comps, or create a full valuation model for a specific ticker."
---

# Full Valuation Model

## Overview

This skill builds a five-tab Excel valuation workbook for any ticker by pulling live data from yfinance and EDGAR MCPs, then running `scripts/build_model.py`.

## Workflow

### Step 1 — Gather Subject Company Data

Pull in parallel:

```
mcp__yfinance__yfinance_get_ticker_info  →  symbol = <TICKER>
mcp__edgartools__edgar_company           →  identifier = <TICKER>,
                                            include=["financials"],
                                            period="annual", periods=5
```

Record these values — they become hardcoded inputs in the model:

**From yfinance:** `currentPrice`, `marketCap`, `enterpriseValue`, `totalRevenue`, `revenueGrowth`, `grossProfits`, `grossMargins`, `ebitda`, `ebitdaMargins`, `operatingMargins`, `netIncomeToCommon`, `profitMargins`, `freeCashflow`, `operatingCashflow`, `totalDebt`, `totalCash`, `beta`, `trailingPE`, `forwardPE`, `priceToBook`, `sharesOutstanding`, `trailingEps`, `forwardEps`, `52WeekChange`

**From EDGAR (5 years):** Revenue, Operating Income, Net Income, D&A (from CF statement), CapEx (Payments to Acquire PP&E), Operating Cash Flow, Long-term Debt, Cash

Derive: Revenue CAGR (3yr, 5yr), EBITDA = OpInc + D&A, FCF = OCF − CapEx, net debt = totalDebt − totalCash

### Step 2 — Identify and Pull Comps

Select 4–6 publicly traded peers:
- Similar business model (not just same sector)
- Revenue scale within ~0.3x–3x of subject
- Avoid conglomerates where only one segment is comparable
- If including structurally different peers (e.g. Visa/MA alongside a processor), flag it in notes

Pull `mcp__yfinance__yfinance_get_ticker_info` for all comps in parallel. Extract same fields as Step 1.

### Step 3 — Set Model Assumptions

Determine these before running the script:

**DCF Bear/Base/Bull revenue growth:**
- Anchor to historical CAGR and the most recent YoY trend
- Bear = below recent trend; Base = continuation of trend; Bull = modest acceleration
- Do not extrapolate peak growth rates — use the decelerating trajectory as base

**EBITDA margins:** Bear = flat/slight compression from LTM; Base = modest expansion (~50–150bps/yr); Bull = expansion toward peer median

**Conservative DCF — two-phase growth:**
- Yr 1–5: at or slightly above current YoY — hard cap 20%
- Yr 6–10: step down further — hard cap 10%
- Default conservatively; cells are editable in Excel

**Discount rate (Conservative DCF):**
- 12% = large-cap, relatively stable (>$20B mkt cap, beta <1.2)
- 15% = mid/small-cap or high execution risk

**Terminal growth rate (Conservative DCF):** 1% default (range 0–2%, closer to 0)

**WACC (DCF Model):** Build via CAPM: `WACC = E/(D+E) × Cost of Equity + D/(D+E) × Cost of Debt × (1 − tax rate)`. Cost of Equity = risk-free rate + beta × ERP. Use 10-yr Treasury for risk-free rate, 5.5% ERP as default, levered beta from yfinance.

### Step 4 — Populate and Run the Build Script

Edit the `CONFIG` dict at the top of `scripts/build_model.py` with the gathered data, then run:

```bash
pip3 install openpyxl -q
python3 scripts/build_model.py
```

Output: `<TICKER>_Valuation_Model.xlsx` with all five tabs.

See `references/model-spec.md` for the full tab-by-tab layout specification.

**Non-negotiable formula rules:**
- Every projection, margin, discount factor, PV, and sensitivity cell = live Excel formula
- Python computes nothing — only writes formula strings like `"=B17*(1+$B$7)"`
- Hardcoded values only: raw historical inputs, assumption drivers, market prices
- Blue font `#1F5C99` on all editable input cells
- Terminal value should be 40–70% of total EV — flag if outside this range
- Sensitivity table center cell must equal the base case implied share price

### Step 5 — Verify and Deliver

Spot-check before delivering:

```python
import openpyxl
wb = openpyxl.load_workbook("TICKER_Valuation_Model.xlsx", data_only=False)
ws = wb["Conservative DCF"]
# Key FCF cell should be a formula string, not a number
assert ws["B18"].value.startswith("="), "Formula missing — hardcode found"
```

Deliver the file and summarize:
- Peer group with median EV/EBITDA, P/E, PEG, Price/FCF, and Price/Sales
- DCF implied price range (Bear / Base / Bull — GGM and exit multiple)
- Conservative DCF implied price and upside vs current price
- Dhandho IV per share, after-MoS price, and the Reverse Dhandho embedded growth rate
- Key risks and the assumptions that most drive the bull/bear spread

## Value Investing Guardrails (Conservative DCF)

Do not loosen these without explicit user instruction:
- Growth yr 1–5: anchor to recent decelerated YoY, not historical peak
- Growth yr 6–10: step down, max 10%
- Terminal growth: 1% default
- Discount rate: 12% large-cap / 15% higher-risk
- End-of-year discounting (more conservative than mid-year)
- No upward normalization of FCF — use actuals as base

## Data Source Priority

1. **yfinance MCP** — market data, LTM financials, ratios
2. **EDGAR MCP** — 5-year historical financials
3. If a field is unavailable from either, note it and use the closest proxy — do not use web search

## Resources

- **`scripts/build_model.py`** — Parameterized Excel builder. Populate the `CONFIG` dict at the top, run the script. Builds all five tabs with live formulas. Read this file to understand the full structure before modifying for a new ticker.
- **`references/model-spec.md`** — Tab-by-tab layout specification: row positions, formula logic, color palette, sensitivity table structure, Conservative DCF layout, and Dhandho DCF layout.
