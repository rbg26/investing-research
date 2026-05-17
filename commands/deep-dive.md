---
description: Full end-to-end investment research workflow — industry analysis, financial statement analysis, full valuation model, and investment checklist — for any publicly traded ticker. Runs all four stages sequentially and delivers all outputs together.
argument-hint: "[TICKER]"
---

The ticker is: **$ARGUMENTS**

Run the four stages below **sequentially** — do not start the next stage until the previous one is complete. Do not ask clarifying questions between stages; make reasonable inferences and proceed. At the end, deliver all outputs together.

---

## Stage 1 — Industry & Strategy Analysis

Run the `strategy-industry-analysis` skill for **$ARGUMENTS**.

- Identify the company's primary industry automatically from the 10-K and public sources — do not pause to confirm with the user
- Fetch the most recent 10-K from SEC EDGAR automatically (Path B) — do not ask for an upload
- Produce the full Porter's Five Forces + strategy analysis PDF
- Save output as `$ARGUMENTS_industry_analysis.pdf`
- **Carry forward** into subsequent stages: the dominant competitive forces, key moat assessment, and top 2–3 strategic risks identified

---

## Stage 2 — Financial Statement Analysis

Run the `financial-statement-analysis` skill for **$ARGUMENTS**.

- Pull 3 years of financial data from EDGAR MCP and yfinance MCP
- Produce the full HTML dashboard covering income statement, balance sheet, and cash flow statement with ratio analysis and trend commentary
- Save output as `$ARGUMENTS_financial_analysis.html`
- **Carry forward** into subsequent stages: revenue/FCF trends, margin trajectory, balance sheet health, and any red flags flagged in the analysis

---

## Stage 3 — Full Valuation Model

Run the `full-valuation-model` skill for **$ARGUMENTS**.

- Use findings from Stages 1 and 2 to inform assumption-setting:
  - Industry competitive dynamics (Stage 1) → bear/bull spread on revenue growth
  - Margin trends (Stage 2) → EBITDA margin trajectory
  - FCF quality (Stage 2) → Conservative DCF and Dhandho base FCF
- Follow every step in the skill: pull data from yfinance MCP + EDGAR MCP, set assumptions, populate CONFIG, run `scripts/build_model.py`
- Output: `$ARGUMENTS_Valuation_Model.xlsx`
- **Carry forward** into Stage 4: implied price range (Bear/Base/Bull), Conservative DCF implied price, Dhandho IV per share and Reverse Dhandho embedded growth rate, upside/downside vs current price

---

## Stage 4 — Investment Checklist

Run the `investment-checklist` skill for **$ARGUMENTS**.

- Synthesize all prior research: do not re-fetch data already gathered in Stages 1–3
- Use Stage 1 output for competitive moat and industry risk questions
- Use Stage 2 output for all financial health, leverage, and earnings quality questions
- Use Stage 3 output for valuation-related checklist items
- Produce the full three-section checklist PDF (Warnings, Avoid Criteria, Management Quality) with final scorecard and analyst synthesis
- Save output as `$ARGUMENTS_investment_checklist.pdf`

---

## Final Delivery

Present all four outputs together:

1. **`$ARGUMENTS_industry_analysis.pdf`** — Porter's Five Forces + strategy analysis
2. **`$ARGUMENTS_financial_analysis.html`** — 3-year financial statement analysis
3. **`$ARGUMENTS_Valuation_Model.xlsx`** — 5-tab valuation workbook
4. **`$ARGUMENTS_investment_checklist.pdf`** — Investment checklist with scorecard

Follow with a single summary covering:
- **Industry**: dominant force, moat assessment, top risk
- **Financials**: revenue/FCF trend, key ratio signal, any red flags
- **Valuation**: Bear/Base/Bull implied price, Conservative DCF price, Dhandho IV/share, upside vs current price
- **Checklist**: Red Flag / Caution / Clear count and the one-paragraph analyst synthesis from the checklist
