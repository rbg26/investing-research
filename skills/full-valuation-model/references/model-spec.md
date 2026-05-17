# Full Valuation Model — Tab Specifications

## Table of Contents
1. [Tab 1: Comps Analysis](#tab-1-comps-analysis)
2. [Tab 2: DCF Model](#tab-2-dcf-model)
3. [Tab 3: Sensitivity Analysis](#tab-3-sensitivity-analysis)
4. [Tab 4: Conservative DCF](#tab-4-conservative-dcf)
5. [Tab 5: Dhandho DCF](#tab-5-dhandho-dcf)
6. [Color Palette](#color-palette)
7. [Formula Conventions](#formula-conventions)

---

## Tab 1: Comps Analysis

### Layout
```
Row 1–3:   Title block (company name, peer list, date/units)
Row 5:     "OPERATING STATISTICS & FINANCIAL METRICS" section header
Row 6:     Column headers
Rows 7–N:  Subject company (row 7, lightly shaded), then comps
Row N+2:   Stats block — Maximum, 75th Pct, Median, 25th Pct, Minimum
Row M:     "VALUATION MULTIPLES" section header
Row M+1:   Column headers
Rows M+2…: Same company order, same shading
Row P+2:   Stats block (comps only, excludes subject)
Row Q:     "METHODOLOGY & DATA SOURCES" section header
Row Q+1…:  Notes (one bullet per row)
```

### Operating Metrics Columns
Company | Ticker | Revenue (LTM, $M) | Rev Growth (YoY) | Gross Profit ($M) | Gross Margin | EBITDA ($M) | EBITDA Margin | Net Income ($M) | Net Margin | Op Cash Flow ($M) | FCF ($M) | FCF Margin | Beta

Formulas: Gross Margin = `=E{r}/C{r}`, EBITDA Margin = `=G{r}/C{r}`, Net Margin = `=I{r}/C{r}`, FCF Margin = `=L{r}/C{r}`

### Valuation Multiples Columns
Company | Ticker | Market Cap ($M) | Enterprise Value ($M) | EV/Revenue | EV/EBITDA | P/E (Trailing) | P/E (Forward) | Price/Book | FCF Yield | Div Yield | 52-Wk Return | PEG Ratio | Price/FCF | Price/Sales

Formulas:
- EV/Revenue = `=D{r}/C{op_row}`, EV/EBITDA = `=D{r}/G{op_row}` (cross-reference operating section)
- FCF Yield = `=L{op_row}/C{r}`
- PEG Ratio = `=IFERROR(IF(D{op_row}=0,"",G{r}/(D{op_row}*100)),"")` — P/E Trailing ÷ YoY revenue growth as %; blank if growth is 0 or negative
- Price/FCF = `=IFERROR(C{r}/L{op_row},"")` — blank if FCF is N/A
- Price/Sales = `=C{r}/C{op_row}` — Market Cap ÷ Revenue

### Statistics
- Computed on comps only (exclude subject company row)
- Use MAX, QUARTILE(,3), MEDIAN, QUARTILE(,1), MIN
- Apply to: Rev Growth, Gross Margin, EBITDA Margin, Net Margin, FCF Margin, Beta, EV/Revenue, EV/EBITDA, P/E Trailing, P/E Forward, FCF Yield, PEG Ratio, Price/FCF, Price/Sales
- Do NOT apply to absolute size columns (Revenue $M, EBITDA $M, Mkt Cap, EV)
- PEG and Price/FCF use IFERROR to return "" for comps with 0% growth or no FCF — stats functions naturally ignore text/blank cells so medians still compute correctly

---

## Tab 2: DCF Model

Three scenario blocks stacked vertically — Bear, Base, Bull. Each block is self-contained.

### Each Scenario Block Structure
```
Title row (colored by scenario: red=bear, green=base, blue=bull)
Column headers: [blank] | Historical FY{N}A | FY{N+1}E | ... | FY{N+5}E
Revenue ($M)          — input historical, formula for projections
  YoY Growth          — formula: =C{rev}/B{rev}-1
EBITDA ($M)           — formula: =C{rev}*{margin}
  EBITDA Margin       — formula: =C{ebitda}/C{rev}
D&A ($M)              — formula: =C{rev}*{da_pct}
EBIT ($M)             — formula: =C{ebitda}-C{da}
NOPAT                 — formula: =C{ebit}*(1-{tax_rate})
Less: CapEx ($M)      — formula: =-C{rev}*{capex_pct}
Add: D&A ($M)         — formula: =C{da_row}
Unlevered FCF ($M)    — formula: =C{nopat}+C{capex}+C{da_addback}
Discount Factor       — formula: =1/(1+{wacc})^{period} (mid-year: period=0.5,1.5,...)
PV of FCF ($M)        — formula: =C{ufcf}*C{disc}

[blank row]

TERMINAL VALUE CALCULATION section
Terminal Year FCF     — formula: ={last_fcf_col}{ufcf_row}
Terminal Growth Rate  — hardcoded input (editable)
WACC                  — hardcoded input (editable)
Terminal Value (GGM)  — formula: =B{tv_fcf}*(1+B{tgr})/(B{wacc}-B{tgr})
Exit Multiple         — hardcoded input (editable)
Terminal EBITDA       — formula: ={last_ebitda_col}{ebitda_row}
Terminal Value (Exit) — formula: =B{exit_mult}*B{tv_ebitda}
PV of TV (GGM)        — formula: =B{tv_ggm}*{last_disc_col}{disc_row}
PV of TV (Exit)       — formula: =B{tv_mult}*{last_disc_col}{disc_row}

EQUITY VALUE BRIDGE section
Sum PV FCFs           — formula: =SUM of all PV FCF cells
EV (GGM)              — formula: =B{sum_pv}+B{pv_tv_ggm}
EV (Exit Multiple)    — formula: =B{sum_pv}+B{pv_tv_mult}
Less: Net Debt        — hardcoded input
Diluted Shares (M)    — hardcoded input
Implied Price (GGM)   — formula: =(B{ev_ggm}-B{net_debt})/B{shares}
Implied Price (Exit)  — formula: =(B{ev_mult}-B{net_debt})/B{shares}
Current Price         — hardcoded input
Upside (GGM)          — formula: =B{price_ggm}/B{curr_price}-1
Upside (Exit)         — formula: =B{price_mult}/B{curr_price}-1
```

### Discounting Convention
Mid-year convention: period = yr − 0.5 (Year 1 = 0.5, Year 2 = 1.5, etc.)
Discount Factor = `=1/(1+{wacc})^{period}`

---

## Tab 3: Sensitivity Analysis

Three two-way tables using base case assumptions:

### Table 1: WACC vs Terminal Growth Rate (GGM method)
- Rows: WACC range (e.g. 8.5% to 11.5% in 0.5% steps)
- Cols: Terminal Growth Rate (e.g. 2.0% to 4.5% in 0.5% steps)
- Each cell: full DCF recalculation formula (not approximation)
- Center cell = base case WACC & TGR → must equal base case implied price
- Color coding: green > $60, amber = current price to $60, red < current price

### Table 2: WACC vs Exit EV/EBITDA Multiple
- Rows: WACC range
- Cols: Exit multiple range (e.g. 8x to 14x)
- Same color coding

### Table 3: Revenue CAGR vs Terminal EBITDA Margin
- Rows: Revenue CAGR range
- Cols: Terminal EBITDA margin range
- Fixed exit multiple (base case value)
- Same color coding

### Color Legend Row
Add below tables: green = significant upside, amber = moderate upside, red = downside

---

## Tab 4: Conservative DCF

Modeled on the value investing template from the original PYPL session. Layout matches the screenshot reference (assumptions top-left, summary box top-right, year table middle, final calcs bottom-left).

### Layout
```
Row 1:    Title
Row 2:    Subtitle (discounting method, units)
Row 4:    Initial Cash Flow (OCF − CapEx)      ← editable input, boxed border
Row 5:    [spacer]
Row 6:    "Years" label | "1–5" header (blue) | "6–10" header (blue)
Row 7:    FCF Growth Rate | [input yr1-5] | [input yr6-10]   ← both editable
Row 8:    Discount Rate   | [input]                           ← editable
Row 9:    Terminal Growth Rate | [input]                      ← editable
Row 10:   [spacer]
Row 11:   Net Debt ($M)              ← editable input
Row 12:   Diluted Shares (M)         ← editable input
Row 13:   Current Market Cap ($M)    ← editable input
Row 14:   Current Share Price ($)    ← editable input

Cols F–G (rows 6–11): Summary box
  F6/G6:  DCF Value ($M)               =B35 (or wherever Total DCF EV lands)
  F7/G7:  Current Market Cap ($M)      =B13
  F8/G8:  DCF as % of Market Cap       =F6/F7
  F9/G9:  Implied Share Price ($)      =B39 (or implied price row)
  F10/G10: Current Share Price ($)     =B14
  F11/G11: Upside / (Downside)         =F9/F10-1

Row 17:   Year table header (Year | FCF ($M) | Growth Rate | [spacer cols] | Present Value ($M))
Rows 18–27: Years 1–10
  FCF yr1:  =$B$4*(1+$B$7)
  FCF yr2+: =B{prev}*(1+IF({yr}<=5,$B$7,$C$7))   ← growth switches at yr 6
  Growth:   =IF({yr}<=5,$B$7,$C$7)
  PV:       =B{r}/(1+$B$8)^{yr}                   ← end-of-year discounting

Row 30:   "FINAL CALCULATIONS" section header
Row 31:   Terminal Year FCF       =B27*(1+$B$9)
Row 32:   PV of Yr 1–10 CFs       =SUM(G18:G27)
Row 33:   Terminal Value          =B31/($B$8-$B$9)
Row 34:   PV of Terminal Value    =B33/(1+$B$8)^10
Row 35:   Total DCF Value (EV)    =B32+B34
Row 36:   Less: Net Debt          =$B$11
Row 37:   Equity Value            =B35-B36
Row 38:   Diluted Shares (M)      =$B$12
Row 39:   Implied Share Price     =B37/B38
Row 42:   Notes row (merged, italic)
```

### Key Formula Notes
- Growth switches at year 6: `=IF(year<=5, $B$7, $C$7)` — `$B$7` = yr 1–5 rate, `$C$7` = yr 6–10 rate
- PV uses end-of-year (not mid-year): `=B18/(1+$B$8)^1`
- Terminal Value = GGM only (no exit multiple on this tab — keeps it simple)
- Summary box F6 references Total DCF Value (EV), not equity value — comparison to market cap is EV vs EV

---

## Tab 5: Dhandho DCF

Based on the Pabrai framework from *The Dhandho Investor*. Two side-by-side tables: forward (assumed growth) and Reverse Dhandho (market-implied growth).

### Layout
```
Row 1:    Title
Row 2:    Subtitle (discounting method, units)
Row 3:    spacer
Row 4:    "DHANDHO IV — ASSUMED GROWTH" (A-G) | "REVERSE DHANDHO IV — MARKET-IMPLIED GROWTH" (I-O)
Row 5:    Column headers (Yr | Year | FCF $M | PV of FCF $M) | ASSUMPTIONS | (same for reverse) | EMBEDDED GROWTH
Row 6:    Year 0 — Excess Cash row | FCF Base assumption
Rows 7-16: Years 1–10 FCF and PV | remaining assumptions alongside
Row 17:   Terminal — Sale Price (exit_multiple × FCF_yr10)
Row 18:   spacer
Row 19:   VALUATION SUMMARY header (both sides)
Rows 20-27: Summary rows
Row 29:   Notes
```

### Column Structure
```
A: Year number    B: Year label    C: FCF ($M)    D: PV of FCF ($M)
E: spacer
F: Assumption label    G: Assumption value  (blue font = editable)
H: spacer
I: Year number (Reverse)    J: Year label    K: FCF ($M)    L: PV of FCF ($M)
M: spacer
N: Embedded assumption label    O: Embedded value
```

### Assumptions (cols F-G, rows 6-14)
```
G6:  FCF Base ($M)                   — OCF − CapEx, most recent year
G7:  FCF Growth Rate (Yr 1–5)        — editable, blue
G8:  FCF Growth Rate (Yr 6–10)       — editable, blue
G9:  Discount Rate                   — 12% large-cap / 15% higher-risk, editable
G10: Exit Multiple (× Terminal FCF)  — editable (typically 10–15×)
G11: Margin of Safety                — editable (default 25%)
G12: Excess Cash ($M)                — Cash + Current Investments (balance sheet, no debt subtraction)
G13: Shares Outstanding (M)
G14: Current Share Price ($)
```

### FCF Formulas (left side)
```
C7  (yr 1):     =$G$6*(1+$G$7)
C8–C11 (yr2–5): =C{prev}*(1+$G$7)
C12–C16 (yr6–10): =C{prev}*(1+$G$8)
C17 (terminal): =C16*$G$10          ← Sale price = terminal year FCF × exit multiple
D6  (excess cash PV): =$G$12        ← Added at face value, not discounted
D7–D16: =C{r}/(1+$G$9)^{yr}        ← End-of-year discounting
D17: =C17/(1+$G$9)^10
```

### Reverse Dhandho (cols I-L, N-O)
- O7 = Python-solved flat growth rate (static value, highlighted amber) that makes equity IV/share = current price
- All 10 years use the same flat rate: `=$O$6*(1+$O$7)` for yr 1, `=K{prev}*(1+$O$7)` thereafter
- All other assumptions (discount, exit multiple, MoS, excess cash) link to the left-side cells

### Summary (rows 20-27)
```
Row 20: PV of Year 1–10 FCFs    =SUM(D7:D16)       / =SUM(L7:L16)
Row 21: PV of Sale Price        =D17               / =L17
Row 22: Add: Excess Cash        =D6                / =L6
Row 23: Total Intrinsic Value   =D20+D21+D22       / =L20+L21+L22
Row 24: IV per Share            =D23/$G$13         / =L23/$G$13
Row 25: After Margin of Safety  =D24*(1-$G$11)     / =L24*(1-$G$11)
Row 26: Current Share Price     =$G$14             / =$G$14
Row 27: Premium/(Discount)      =D25/$G$14-1       / =L25/$G$14-1
```

### Key Design Principles
- FCF = OCF − CapEx (equity-level, after interest) — debt is NOT subtracted separately
- Excess Cash = Cash + Current Investments from the balance sheet only (no debt netting)
- Terminal value = exit multiple × terminal year FCF (sale price method, not GGM)
- End-of-year discounting throughout
- Excess Cash added at face value (Year 0, not discounted — it's already present)
- Reverse Dhandho embedded rate is Python-solved via binary search at build time (static)

### CONFIG keys required
```python
"dhandho_fcf_base_m":    <int>    # OCF − CapEx
"dhandho_growth_1_5":    <float>  # e.g. 0.08
"dhandho_growth_6_10":   <float>  # e.g. 0.05
"dhandho_discount":      <float>  # 0.12 or 0.15
"dhandho_exit_mult":     <float>  # e.g. 12.0
"dhandho_mos":           <float>  # e.g. 0.25
"dhandho_cash_m":        <int>    # cash & equivalents (balance sheet)
"dhandho_investments_m": <int>    # short-term / current investments (balance sheet)
"dhandho_debt_m":        <int>    # kept for notes only, not used in IV calculation
```

---

## Color Palette

| Element | Hex | Usage |
|---------|-----|-------|
| Dark blue | `#1F4E79` | Section headers, title bar |
| Medium blue | `#2E75B6` | Sub-headers, column headers background |
| Light blue | `#D9E1F2` | Column header background (operating/valuation tables) |
| Light grey | `#F2F2F2` | Alternating data rows, stats rows |
| Amber | `#FFF2CC` | Implied share price output cells |
| Green | `#E2EFDA` | Equity value, PV of TV rows |
| Red (sensitivity) | `#FFCCCC` | Below current price in sensitivity tables |
| Input blue | `#1F5C99` | All hardcoded input cell font color |
| White | `#FFFFFF` | Data cells |

---

## Formula Conventions

```python
# CORRECT — formula string
ws["B18"].value = "=$B$4*(1+$B$7)"

# WRONG — hardcoded computed value
ws["B18"].value = 5842  # Never do this for derived cells

# Absolute references for assumption cells
# $B$7 = growth rate yr 1-5 (always absolute — copied across rows)
# $B$8 = discount rate (always absolute)

# Cross-tab references not supported in openpyxl formulas
# Keep all formula references within the same sheet
```

All input cells: blue font `#1F5C99`, boxed border for key drivers, `fill = LIGHT_BLUE` on the growth/rate inputs to signal editability.
