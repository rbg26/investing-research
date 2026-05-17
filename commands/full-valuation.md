---
description: Build a full 5-tab valuation workbook (Comps, DCF, Sensitivity, Conservative DCF, Dhandho DCF) for any publicly traded ticker
argument-hint: "[TICKER]"
---

The ticker is: **$ARGUMENTS**

Run the `full-valuation-model` skill end-to-end for this ticker. Follow every step in the skill exactly — do not skip or abbreviate any step.

Step 1: Pull subject company data in parallel from yfinance MCP and EDGAR MCP.
Step 2: Identify 4–6 comparable peers and pull their data from yfinance MCP in parallel.
Step 3: Set Bear/Base/Bull DCF assumptions, WACC, Conservative DCF growth rates, and Dhandho inputs anchored to the data gathered.
Step 4: Populate the CONFIG dict in `scripts/build_model.py` with all gathered data, then run:
```
pip3 install openpyxl -q
python3 scripts/build_model.py
```
Step 5: Verify formulas are live (not hardcoded), then deliver the .xlsx with the summary described in the skill.
