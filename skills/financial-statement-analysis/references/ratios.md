# Financial Ratio Formulas — Reference

All formulas use trailing twelve months (TTM) or annual figures unless noted.
"Avg" = average of beginning and ending balance for the period (use 2-point average unless rolling average is available).

---

## Group 1 — Activity / Efficiency Ratios

These measure how effectively a company uses its assets to generate revenue.

### Inventory Turnover
```
Inventory Turnover = COGS ÷ Average Inventory
```
- Higher = faster inventory movement = more efficient
- Use COGS (not revenue) — inventory is carried at cost, not selling price
- If COGS is unavailable (services companies), use Revenue as proxy but note the substitution
- **Analyst note**: Compare to industry peers. A sudden spike may indicate inventory write-downs, not efficiency.

### Inventory Days (Days Inventory Outstanding — DIO)
```
DIO = 365 ÷ Inventory Turnover
   = (Average Inventory ÷ COGS) × 365
```
- Lower = faster-moving inventory = healthier
- Rising DIO with flat/falling sales is an early warning of demand weakness or overstocking

### Receivables Turnover
```
Receivables Turnover = Revenue ÷ Average Accounts Receivable (Trade AR only)
```
- Higher = faster collection
- Use trade AR only — exclude loans, notes receivable, financing receivables

### Days Sales Outstanding (DSO)
```
DSO = 365 ÷ Receivables Turnover
    = (Average Trade AR ÷ Revenue) × 365
```
- Lower = faster collection = better cash quality
- Rising DSO with rising revenue may indicate loosening credit terms to drive sales
- Compare to stated payment terms (e.g., net 30 → DSO should be ~30–45 days)

### Payables Turnover
```
Payables Turnover = COGS ÷ Average Accounts Payable
```
- Lower = company is taking longer to pay suppliers = more supplier financing
- Very low payables turnover can signal cash constraints (or strong negotiating power)

### Days Payable Outstanding (DPO)
```
DPO = 365 ÷ Payables Turnover
    = (Average Accounts Payable ÷ COGS) × 365
```
- Higher DPO = company holds cash longer before paying = favorable working capital management
- Extremely high DPO may signal supplier relationship strain

### Cash Conversion Cycle (CCC)
```
CCC = DIO + DSO − DPO
```
- Measures days between paying for inventory/inputs and collecting from customers
- **Negative CCC** (e.g., Amazon, Walmart) = suppliers are financing operations = very healthy
- Rising CCC = deteriorating working capital management
- For services companies: CCC = DSO − DPO (omit DIO)

### Working Capital Turnover
```
Working Capital Turnover = Revenue ÷ Average Working Capital
   where Working Capital = Current Assets − Current Liabilities
```
- Higher = more revenue generated per dollar of working capital
- Negative working capital → ratio is negative and not meaningful; note this explicitly
- **Distortion alert**: Companies with large customer float (PayPal, insurance) have artificially inflated current liabilities — adjust or flag

### Fixed Asset Turnover
```
Fixed Asset Turnover = Revenue ÷ Average Net PP&E
```
- Higher = more revenue per dollar of fixed assets = capital efficiency
- Capital-light companies (software, consulting) will have very high ratios (10x+) — this is normal
- Capital-intensive companies (manufacturing, utilities) will be in the 1–3x range

### Total Asset Turnover
```
Total Asset Turnover = Revenue ÷ Average Total Assets
```
- Key driver in DuPont ROE decomposition
- Low asset turnover + high margin = premium/brand model (luxury, pharma)
- High asset turnover + low margin = volume/commodity model (retail, airlines)

---

## Group 2 — Liquidity Ratios

These measure ability to meet short-term obligations.

### Current Ratio
```
Current Ratio = Current Assets ÷ Current Liabilities
```
- Rule of thumb: >1.5x healthy; <1.0x concerning (but industry-dependent)
- Retail, subscription, and fintech companies often run below 1.0x by design — not inherently risky
- **Distortion**: For companies with large deferred revenue in current liabilities (SaaS, subscriptions), the ratio understates true liquidity

### Quick Ratio (Acid Test)
```
Quick Ratio = (Cash + Short-Term Investments + Trade AR) ÷ Current Liabilities
```
- Strips out inventory and prepaid expenses — a tighter liquidity test
- Rule of thumb: >1.0x generally healthy
- For companies with no inventory (services, fintech): Quick Ratio ≈ Current Ratio

### Cash Ratio
```
Cash Ratio = (Cash + Cash Equivalents + Short-Term Investments) ÷ Current Liabilities
```
- The strictest liquidity measure — uses only the most liquid assets
- Most companies run below 1.0x; this is normal
- Useful for stress-testing: could the company survive with zero collections for [period]?

---

## Group 3 — Leverage / Solvency Ratios

These measure capital structure and debt serviceability.

### Debt to Assets
```
Debt to Assets = Total Debt ÷ Total Assets
   where Total Debt = Short-Term Debt + Current Portion of LT Debt + Long-Term Debt
```
- What fraction of assets are financed by debt
- Higher = more leveraged, more financial risk
- Industry benchmarks vary widely: utilities ~60%, tech ~10–20%, financials require different framework

### Debt to Capital
```
Debt to Capital = Total Debt ÷ (Total Debt + Total Equity)
```
- Also called "debt ratio to capitalization"
- Removes total assets (which include current liabilities) — focuses on long-term capital structure
- <40% generally considered moderate; >60% high leverage

### Debt to Equity
```
Debt to Equity (D/E) = Total Debt ÷ Total Shareholders' Equity
```
- Most commonly cited leverage ratio
- Negative equity → D/E is not meaningful; note and explain (e.g., buyback-driven deficit)
- **Analyst note**: Always check if equity is distorted by large treasury stock / buybacks before interpreting D/E

### Financial Leverage Ratio (Equity Multiplier)
```
Financial Leverage = Average Total Assets ÷ Average Total Equity
```
- Key component in DuPont ROE decomposition
- Higher leverage → amplifies both gains and losses
- This is the "A/E" term in DuPont: ROE = Net Margin × Asset Turnover × Financial Leverage

### Interest Coverage (Times Interest Earned)
```
Interest Coverage = EBIT ÷ Interest Expense
```
- How many times over can operating income cover interest payments
- <2x: potential distress territory; 2–4x: adequate; >5x: comfortable
- Use EBIT (not EBITDA) for conservatism unless the industry convention is EBITDA-based (e.g., telecoms, REITs)
- If interest expense is zero: record as "∞ — debt-free" not a calculation error

---

## Group 4 — Profitability: Return on Sales

### Gross Profit Margin
```
Gross Profit Margin = Gross Profit ÷ Revenue
   where Gross Profit = Revenue − COGS
```
- Measures pricing power and direct production efficiency
- Widening gross margins = pricing power gaining or input costs falling
- Narrowing gross margins = competition intensifying, input cost inflation, or product mix shift

### Operating Profit Margin (EBIT Margin)
```
Operating Profit Margin = EBIT ÷ Revenue
   where EBIT = Gross Profit − Operating Expenses (SG&A, R&D)
```
- Reflects operating leverage: how well the company scales its cost base
- If operating margin < gross margin gap is widening: SG&A/R&D is rising as % of revenue (scaling inefficiency)

### Net Profit Margin
```
Net Profit Margin = Net Income ÷ Revenue
```
- All-in profitability after interest and taxes
- Below operating margin due to: interest expense, tax rate, and non-operating items
- Normalize for one-time items where material (note clearly when doing so)

### FCF Margin
```
FCF Margin = (Operating Cash Flow − CapEx) ÷ Revenue
```
- Cash profitability after maintenance and growth investment; strips out accrual distortions
- The most important margin for capital-light, asset-light, or brand-driven businesses with no debt
- Compare to net profit margin: a persistently lower FCF margin signals working capital deterioration or aggressive CapEx; a higher FCF margin signals strong cash conversion
- **Analyst note**: For capital-intensive businesses, distinguish maintenance CapEx (FCF margin using only maintenance CapEx is the "true" recurring cash margin) from growth CapEx (which is discretionary and may be temporarily suppressing FCF)

---

## Group 5 — Profitability: Return on Investment

### Return on Assets (ROA)
```
ROA = Net Income ÷ Average Total Assets
```
- How much profit is generated per dollar of assets employed
- Capital-intensive industries: 2–5% normal; asset-light tech/software: 10–20%+ possible
- Affected by asset base inflation from acquisitions (goodwill drag on ROA)

### Return on Capital Employed (ROCE)
```
ROCE = EBIT ÷ Capital Employed
   where Capital Employed = Total Assets − Current Liabilities
```
- Preferred by many practitioners over ROA because it excludes short-term liabilities
- Measures return on long-term capital (both debt and equity)
- ROCE > WACC = value-creating; ROCE < WACC = value-destroying

### Return on Equity (ROE) — DuPont Decomposition
```
ROE = Net Income ÷ Average Shareholders' Equity

DuPont:
ROE = Net Profit Margin × Total Asset Turnover × Financial Leverage Ratio
    = (Net Income/Revenue) × (Revenue/Avg Assets) × (Avg Assets/Avg Equity)
```

**How to interpret the DuPont:**
Always show each component year-over-year. Then ask: *what is driving the ROE change?*

| Driver | What it signals |
|---|---|
| Margin improving | Better pricing, cost discipline, or mix shift |
| Asset turnover improving | Better capital efficiency, asset sweating, or asset-light shift |
| Leverage increasing | More debt taken on — amplifies ROE but raises risk |
| Leverage decreasing | Deleveraging — ROE may fall even if operations improve |

**Common ROE distortions to flag:**
- Negative equity from buybacks → ROE denominator near zero or negative → ratio meaningless; explain explicitly
- One-time earnings boosting net income → inflates ROE in that year
- Rising ROE driven entirely by financial leverage (not operations) = warning sign

---

## Notes on N/A Ratios

When a ratio is structurally not applicable, state:

| Company type | N/A ratios | Reason |
|---|---|---|
| Pure services / fintech / SaaS | Inventory Turnover, DIO | No physical inventory |
| Debt-free company | Interest Coverage, D/E | No interest expense / no debt |
| Negative equity company | D/E, ROE | Denominator distorted by buybacks |
| Financial institution (bank, insurer) | Most above ratios | Requires different analytical framework |

For financial institutions: note that a fundamentally different set of ratios applies (NIM, efficiency ratio, Tier 1 capital ratio, etc.) and this skill's framework is not appropriate.
