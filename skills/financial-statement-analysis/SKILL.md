---
name: financial-statement-analysis
description: Conduct a comprehensive 3-year financial statement analysis covering the balance sheet, income statement, and cash flow statement for any company. Use this skill whenever a user asks to analyze a company's financials, review an annual report or 10-K, assess financial health, calculate financial ratios, evaluate trends in profitability/liquidity/leverage, or asks questions like "analyze [company]'s financials", "what do the numbers say about [company]", "is [company] financially healthy", or "break down [company]'s balance sheet/income statement/cash flow". Always use this skill for any multi-ratio or multi-statement financial analysis request — even partial ones like "look at the liquidity ratios" or "analyze cash flow vs profit".
---

# Financial Statement Analysis Skill

## Overview

This skill produces a rigorous, analyst-grade financial statement analysis covering all three financial statements over the most recent 3 years of available data. The output is a structured HTML dashboard — combining tables, trend indicators, and written analyst commentary — that surfaces what the numbers actually mean, not just what they are.

**Core analyst philosophy:**
- Numbers alone are not insights. Every metric must be interpreted: is this good or bad *for this company, in this industry, at this point in time?*
- Trends matter more than snapshots. A rising ratio can be healthy or alarming depending on direction and context.
- Not all metrics are equally important. Flag which ratios are most signal-rich for this specific business model, and which are structurally N/A (e.g., inventory metrics for a services company).
- Be direct. Write like a sell-side analyst in an internal note — clear verdicts, no hedging theater.

---

## Step 1: Gather Inputs

### 1A — Determine the data source

Ask the user (or infer from context) which path to take:

**Path A — User provides financials directly**  
They paste numbers, upload a PDF, or reference data already in the conversation. Proceed to Step 2.

**Path B — Fetch from SEC EDGAR (US-listed companies)**  
Use the EDGAR approach from the Porter's Five Forces skill (`/mnt/skills/user/porters-five-forces/SKILL.md`, Steps 1b Path B) to locate the 10-K. For financial data specifically, target these sections:
- Consolidated Balance Sheets (typically in Item 8)
- Consolidated Statements of Operations / Income
- Consolidated Statements of Cash Flows
- Notes: Debt schedule, segment data, inventory breakdown

**Path C — Web search for public financial data**  
Use `web_fetch` on StockAnalysis, Macrotrends, or the company's investor relations page. Always cross-check at least two sources. Clearly label confidence level (8/10 for audited SEC data, 6/10 for aggregators).

### 1B — What 3 years to use

Default to the 3 most recently completed fiscal years. If only 2 years are available, proceed with 2. Note the fiscal year end date — it matters for seasonality interpretation.

### 1C — Industry context (critical)

Before calculating any ratios, identify the industry. This determines:
- Which ratios are meaningful (inventory metrics → irrelevant for pure services)
- What "normal" looks like (a 1.2x current ratio is healthy for retail; alarming for a SaaS company)
- How to interpret leverage (utilities carry much higher D/E than tech by design)

---

## Step 2: Extract and Organize the Raw Data

Build a clean data table before calculating anything. Populate all of the following that are available. Mark missing items as `—`.

```
INCOME STATEMENT          FY[n-2]    FY[n-1]    FY[n]
Revenue
Cost of Goods Sold (COGS)
Gross Profit
Operating Expenses (SG&A + R&D)
EBIT (Operating Income)
Interest Expense
EBT (Pre-tax Income)
Tax Expense
Net Income
Depreciation & Amortization

BALANCE SHEET             FY[n-2]    FY[n-1]    FY[n]
Cash & Equivalents
Short-Term Investments
Accounts Receivable (Trade)
Inventory
Other Current Assets
Total Current Assets
PP&E (Net)
Goodwill & Intangibles
Other Long-Term Assets
Total Assets
Accounts Payable
Short-Term Debt
Current Portion of LT Debt
Other Current Liabilities
Total Current Liabilities
Long-Term Debt
Other Long-Term Liabilities
Total Liabilities
Total Shareholders' Equity

CASH FLOW STATEMENT       FY[n-2]    FY[n-1]    FY[n]
Net Cash from Operations (OCF)
Capital Expenditures (CapEx)
Free Cash Flow (OCF - CapEx)
Net Cash from Investing
Net Cash from Financing
Share Buybacks
Dividends Paid
Debt Issuance / (Repayment)
Net Change in Cash
```

---

## Step 3: Run the Full Analysis

Work through each section in order. Read `/mnt/skills/user/financial-statement-analysis/references/ratios.md` for all formula definitions before calculating.

---

### 3A — Balance Sheet Analysis

For each item below, state the 3-year values and write 1–3 sentences of analyst commentary. Use ↑ ↓ → trend arrows. Flag ⚠️ for concerning trends, ✓ for healthy ones.

**Leverage & Capital Structure**
- **Debt to Equity** = Total Debt ÷ Shareholders' Equity
- **Total Borrowing** — absolute value of ST + LT debt; note maturity profile if available
- **Shareholders' Funds** — equity trend; explain what's driving it (retained earnings vs. buybacks vs. dilution)
- **% of Tangible Assets** = (Total Assets − Goodwill − Intangibles) ÷ Total Assets; signals balance sheet quality

**Liquidity**
- **Is short-term cash declining?** — Compare cash + ST investments across all 3 years; identify *why* (buybacks? acquisitions? operating burn?)
- **Working Capital** = Current Assets − Current Liabilities; note if dominated by customer float or other distortions

**Receivables**
- **Trade Receivables** — absolute trend; flag if growing faster than revenue (potential collection issue)
- **Receivable Turnover** = Revenue ÷ Avg Accounts Receivable
- **Days Sales Outstanding (DSO)** = (Trade AR ÷ Revenue) × 365

**Inventory** *(skip with explanation if not applicable)*
- **Inventory Turnover** = COGS ÷ Avg Inventory
- **Inventory Days (DIO)** = (Avg Inventory ÷ COGS) × 365
- Flag if inventory is building faster than COGS growth (potential obsolescence risk)

**Profit Allocation**
Explicitly answer: *Where are profits going?* Classify the company as one of:
- 🔄 **Reinvestor** — CapEx + R&D > 50% of OCF; growing PP&E or intangibles
- 💰 **Returner** — Buybacks + dividends > 50% of FCF; stable or shrinking equity base
- 🏦 **Deleverager** — Net debt declining meaningfully year-on-year
- 📦 **Acquirer** — Acquisition spend dominates investing cash flows
- 🔥 **Burner** — Operating cash flow negative or insufficient; equity/debt funding operations

---

### 3B — Income Statement Analysis

**Revenue quality**
- Revenue trend: calculate YoY growth rates; note if growth is accelerating or decelerating
- Revenue mix: if segment data available, note which segments are driving growth

**Margin stack** — calculate and trend all three margins:
- **Gross Profit Margin** = Gross Profit ÷ Revenue
- **Operating Profit Margin (EBIT margin)** = EBIT ÷ Revenue  
- **Net Profit Margin** = Net Income ÷ Revenue

Interpret the *spread* between margins: a large gap between gross and operating margin signals high operating expense leverage (or lack thereof). A large gap between operating and net margin signals heavy interest burden or tax volatility.

**Key questions to answer:**
1. Are margins expanding or contracting? Is this structural or cyclical?
2. Is revenue growing faster or slower than costs? (Operating leverage)
3. Is there a one-time item distorting any year? (Gain on asset sale, impairment, restructuring charge) — flag and normalize if so.
4. What is D&A as a % of revenue? Rising D&A may indicate heavy prior capex cycle.

---

### 3C — Cash Flow Analysis

**The four questions every analyst asks:**

**Q1 — Quality of earnings: Profit vs. Operating Cash Flow**

This is the most analytically rich question in the entire analysis. A company can report rising net income while its cash generation quietly deteriorates — or vice versa. The goal is not just to compute the ratio, but to *explain every dollar of divergence* between the two.

**Step 1 — Build the side-by-side comparison table across all 3 years:**

| | FY[n-2] | FY[n-1] | FY[n] |
|---|---|---|---|
| Net Income | | | |
| Operating Cash Flow (OCF) | | | |
| **OCF / Net Income ratio** | | | |
| **Gap (OCF − Net Income)** | | | |

**Step 2 — Diagnose the gap.** The difference between net income and OCF comes from three sources. Work through each one explicitly:

*Bridge item 1 — Non-cash charges (adds back to net income → increases OCF):*
- Depreciation & Amortization: is D&A growing? This signals a prior heavy CapEx cycle now flowing through the P&L. Extract D&A each year and calculate it as % of revenue.
- Stock-based compensation: high SBC inflates OCF vs. net income but dilutes shareholders — note the absolute amount.
- Impairments, write-downs, or other non-cash losses.

*Bridge item 2 — Working capital changes (the most volatile and revealing component):*
Extract the net working capital change from the cash flow statement each year. Then ask:
- Is accounts receivable growing faster than revenue? → Cash is being trapped in uncollected sales. Rising DSO confirmed here.
- Is inventory building? → Potential demand weakness or supply chain pre-buying.
- Are payables shrinking? → Company may be losing supplier leverage or paying faster under pressure.
- Is deferred revenue building or shrinking? → For subscription/SaaS companies, building deferred revenue is OCF-positive but not yet in net income (healthy leading indicator). Shrinking is the reverse.

State the net working capital change for each year as a dollar amount and direction: *"In FY[n], working capital consumed $Xm of cash (drag on OCF), driven primarily by [AR buildup / inventory growth / AP reduction]."*

*Bridge item 3 — Accruals and other items:*
Large "other" items in the operating section (deferred taxes, gains/losses on investments, pension adjustments) should be flagged if they represent >5% of net income in any year.

**Step 3 — Classify the earnings quality pattern across 3 years:**

| Pattern | What it means |
|---|---|
| OCF > Net Income consistently, gap stable | ✓ High quality. D&A exceeds CapEx depreciation; accruals unwinding normally. |
| OCF > Net Income, gap *widening* | ✓ Strengthening cash generation. Working capital releasing or D&A building. |
| OCF ≈ Net Income (ratio 0.85–1.15×) | → Normal. Monitor for drift. |
| OCF < Net Income in one year | ⚠️ Investigate. Usually working capital investment for growth — acceptable if revenue is rising. |
| OCF < Net Income consistently, gap *widening* | 🔴 Red flag. Accrual-based earnings may be overstated. Revenue recognition or collection issues. |
| OCF and Net Income diverging in opposite directions | 🔴 Serious red flag. One of them is telling the wrong story — determine which. |

**Step 4 — Write the earnings quality verdict** (3–5 sentences):
State whether profit and OCF are converging or diverging. Name the single biggest driver of any gap. Give a direct judgment: are reported earnings a reliable proxy for cash generation, or is there a quality concern? Quote the OCF/NI ratio trend explicitly — e.g., *"The OCF/NI ratio moved from 1.3× → 0.9× → 0.7× over the three years, a deteriorating trend that warrants scrutiny of the AR and inventory build visible in the working capital bridge."*

**Q2 — CapEx sufficiency:** Is free cash flow positive? = OCF − CapEx
- Calculate FCF yield implied by FCF / Revenue
- Is CapEx maintenance or growth? (Maintenance CapEx ≈ D&A; Growth CapEx = CapEx − D&A)

**Q3 — Major sources and uses of cash:**
Explicitly list the top 2–3 inflows and outflows each year by size. Build a waterfall narrative:
*"In FY[n], the company generated $Xbn from operations, spent $Xbn on CapEx, and used $Xbn on buybacks — leaving net cash [up/down] $Xbn."*

**Q4 — Financing picture:** Is the company borrowing to fund returns? (Debt issuance + equity issuance > dividends + buybacks is a red flag if FCF is negative)

---

### 3D — Full Ratio Suite

Calculate every ratio below. Present in a table with 3-year trend. After the table, write a 2–3 sentence "Ratio Verdict" for each group.

Read the formula reference file: `/mnt/skills/user/financial-statement-analysis/references/ratios.md`

**Group 1 — Activity / Efficiency**
| Ratio | Formula | FY[n-2] | FY[n-1] | FY[n] | Trend |
|---|---|---|---|---|---|
| Inventory Turnover | COGS ÷ Avg Inventory | | | | |
| Inventory Days (DIO) | 365 ÷ Inventory Turnover | | | | |
| Receivables Turnover | Revenue ÷ Avg AR | | | | |
| Days Sales Outstanding | 365 ÷ Receivables Turnover | | | | |
| Payables Turnover | COGS ÷ Avg AP | | | | |
| Payable Days (DPO) | 365 ÷ Payables Turnover | | | | |
| Cash Conversion Cycle | DIO + DSO − DPO | | | | |
| Working Capital Turnover | Revenue ÷ Avg Working Capital | | | | |
| Fixed Asset Turnover | Revenue ÷ Avg Net PP&E | | | | |
| Total Asset Turnover | Revenue ÷ Avg Total Assets | | | | |

**Group 2 — Liquidity**
| Ratio | Formula | FY[n-2] | FY[n-1] | FY[n] | Trend |
|---|---|---|---|---|---|
| Current Ratio | Current Assets ÷ Current Liabilities | | | | |
| Quick Ratio | (Cash + ST Investments + AR) ÷ Current Liabilities | | | | |
| Cash Ratio | (Cash + ST Investments) ÷ Current Liabilities | | | | |

**Group 3 — Leverage / Solvency**
| Ratio | Formula | FY[n-2] | FY[n-1] | FY[n] | Trend |
|---|---|---|---|---|---|
| Debt to Assets | Total Debt ÷ Total Assets | | | | |
| Debt to Capital | Total Debt ÷ (Total Debt + Equity) | | | | |
| Debt to Equity | Total Debt ÷ Equity | | | | |
| Financial Leverage Ratio | Avg Total Assets ÷ Avg Equity | | | | |
| Interest Coverage (EBIT/Interest) | EBIT ÷ Interest Expense | | | | |

**Group 4 — Profitability: Return on Sales**
| Ratio | Formula | FY[n-2] | FY[n-1] | FY[n] | Trend |
|---|---|---|---|---|---|
| Gross Profit Margin | Gross Profit ÷ Revenue | | | | |
| Operating Profit Margin | EBIT ÷ Revenue | | | | |
| Net Profit Margin | Net Income ÷ Revenue | | | | |
| FCF Margin | (OCF − CapEx) ÷ Revenue | | | | |

**Group 5 — Profitability: Return on Investment**
| Ratio | Formula | FY[n-2] | FY[n-1] | FY[n] | Trend |
|---|---|---|---|---|---|
| Return on Assets (ROA) | Net Income ÷ Avg Total Assets | | | | |
| Return on Capital Employed (ROCE) | EBIT ÷ (Total Assets − Current Liabilities) | | | | |
| Return on Equity (ROE) | Net Income ÷ Avg Equity | | | | |

**DuPont Decomposition of ROE** — always show this:
```
ROE = Net Profit Margin × Asset Turnover × Financial Leverage
    = (Net Income/Revenue) × (Revenue/Avg Assets) × (Avg Assets/Avg Equity)
```
Show each component across 3 years. Identify *which lever* is driving ROE change — is it margin improvement, asset efficiency, or leverage?

---

### 3E — Synthesis: What the Numbers Say

After all calculations, write a 400–600 word analyst summary covering:

1. **The headline verdict** — one sentence: is this company financially healthy, deteriorating, or improving?
2. **3 things working well** — the strongest signals across all metrics
3. **3 things to watch** — the most important risks or negative trends
4. **The most important ratio for this business** — explain why one metric above all others reveals the company's financial character (e.g., for a capital-light tech company, FCF margin; for a retailer, CCC; for a bank, not applicable — different framework)
5. **What this means for the next 12–24 months** — forward-looking interpretation of the trends

---

## Step 4: Output as Interactive HTML Dashboard

Read the frontend design skill before building: `/mnt/skills/public/frontend-design/SKILL.md`

Build a single-file HTML dashboard with:

**Design requirements:**
- Dark or editorial aesthetic (not default white Bootstrap — make it distinctive)
- Distinctive typography: pair a display font (e.g., Syne, Playfair Display, DM Serif) with a monospace data font (JetBrains Mono, IBM Plex Mono) for numbers
- Color-coded trend indicators: green (improving), red (deteriorating), amber (flat/ambiguous)
- Ratio tables with inline sparklines or trend arrows
- Each ratio row must display its formula beneath the ratio name in smaller, muted text (e.g., `COGS ÷ Avg Inventory`) — pull from the formula definitions in `references/ratios.md`
- Analyst commentary boxes distinct from data tables

**Required sections in order:**
1. Header: Company name, ticker, fiscal years covered, date of analysis
2. Raw Data Tables: All three statements side-by-side (3 years)
3. Balance Sheet Analysis panel
4. Income Statement Analysis panel
5. Cash Flow Analysis panel
6. Ratio Suite: tabbed or grouped tables for all 5 ratio groups — each ratio row shows the ratio name, its formula in muted smaller text directly below the name, the 3-year values, and a trend indicator
7. DuPont Decomposition visual
8. Analyst Synthesis: the 400–600 word summary with clear formatting
9. Methodology footer: data sources, confidence ratings, caveats

**Important**: For any ratio that is N/A for the specific business (e.g., inventory ratios for a fintech), display "N/A — [1-line reason]" rather than leaving it blank. This signals analytical rigor, not incompleteness.

Save to `/mnt/user-data/outputs/[TICKER]_financial_analysis.html` and present with `present_files`.

---

## Analyst Conduct Rules

These apply throughout the skill:

1. **No ratio without interpretation.** Every number gets a sentence of context.
2. **Flag distortions explicitly.** Customer float, operating leases, off-balance-sheet items, one-time charges — call them out and explain their effect.
3. **Industry benchmark everything possible.** "DSO of 45 days" is meaningless without knowing if peers are at 30 or 60.
4. **Never fabricate data.** If a line item is unavailable, mark it `—` and note it. Do not estimate unless the estimate is clearly labeled and justified.
5. **Confidence ratings.** Label data sources: SEC 10-K (9/10), company IR page (8/10), financial aggregator (7/10), estimated (3/10).
6. **The DuPont is non-negotiable.** Always decompose ROE. It is the single most instructive ratio decomposition in corporate finance.
