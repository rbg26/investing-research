---
name: investment-checklist
description: Run a structured investment checklist on a stock using Warnings, Avoid Criteria, and Management Quality sections. Use this skill after running strategy-industry-analysis and financial-statement-analysis on a stock. Synthesizes prior research outputs plus web research to answer checklist questions. Trigger whenever the user asks for an investment checklist, quality checklist, or wants to vet a stock against Munger/Buffett-style criteria.
---

# Investment Checklist Skill

## Overview

This skill runs a three-section investment checklist on a company, synthesizing:
- Output from the **strategy-industry-analysis** skill (Porter's Five Forces PDF)
- Output from the **financial-statement-analysis** skill (HTML dashboard)
- Any **web research** already conducted in the session

**Core discipline**: Only answer a question if evidence exists from the above sources. If information is insufficient to answer a specific question, explicitly state: *"Insufficient information to assess."* It is always better to acknowledge a gap than to speculate.

**Model**: Use `claude-opus-4-6` with extended thinking enabled for all checklist question answering. This ensures deep, calibrated reasoning rather than surface-level pattern matching.

---

## Step 1: Gather Available Context

Before running the checklist, explicitly inventory what is available:

1. Has the **strategy analysis** been completed? If yes, note the key findings: Five Forces verdicts, competitive advantage verdict, failure modes identified.
2. Has the **financial analysis** been completed? If yes, note the key findings: earnings quality verdict, OCF/NI trend, DuPont decomposition, leverage, FCF trend, profit allocation classification.
3. What **web research** was conducted? Note any management interviews, press releases, earnings call transcripts, or news articles retrieved.

If neither prior skill has been run, tell the user: *"This checklist is most useful after running the strategy analysis and financial analysis. Would you like me to run those first, or proceed with web research only?"*

---

## Step 1b: Management Research (Always Run Before the Checklist)

Before calling the API, always run the following targeted research to feed Section 3 (Management Quality). Do not skip this step — without it, most management questions will return "Insufficient information."

### 1. Proxy Statement (DEF 14A) — compensation, tenure, insider ownership

Fetch the most recent DEF 14A from SEC EDGAR:

```python
# Reuse the CIK lookup from the strategy analysis skill
# Then search filings for form type "DEF 14A"
for i, form in enumerate(filings['form']):
    if form == 'DEF 14A':
        accession = filings['accessionNumber'][i].replace('-', '')
        break
```

From the proxy statement, extract:
- **CEO and CFO tenure** — date first appointed (answers Q4)
- **Executive compensation table** — base salary, annual bonus, long-term equity awards, and whether incentive metrics are tied to revenue, EPS, FCF, or ROIC (answers Q5)
- **Beneficial ownership table** — CEO and CFO share ownership as % of shares outstanding; compare to prior year proxy if available to determine direction of change (answers Q6)
- **Say-on-pay vote result** — if shareholders voted against comp, flag as Caution

### 2. Most Recent Earnings Call Transcript

Search for: `"[Company name] earnings call transcript Q[most recent quarter] [year]"`

Use `web_fetch` on Seeking Alpha, Motley Fool, or the company's IR page. Extract:
- **Guidance language** — does the CEO/CFO give specific EPS or revenue guidance? (answers Q7)
- **Tone of Q&A** — does management deflect hard questions, blame external factors, or take accountability? (answers Q9, Q11)
- **Self-promotional language** — excessive use of superlatives, comparisons to competitors, or claims of unique advantage without evidence (answers Q12)
- **Cost discipline** — any mention of cost reduction programs, efficiency initiatives, or margin improvement targets (answers Q8)

### 3. Recent News Search (last 12 months)

Search: `"[Company name] CEO" OR "[CEO name] interview" site:ft.com OR site:wsj.com OR site:bloomberg.com`

Look for:
- Public statements where management made bold claims about certainty of outcomes (answers Avoid Q2)
- Evidence of contrarian or independent thinking vs. following industry trends (answers Management Q10)
- Any controversies, restatements, or regulatory actions that bear on management integrity

### Organise findings into a "Management Dossier"

Before calling the API, compile a short structured dossier:

```
MANAGEMENT DOSSIER — [TICKER]
CEO: [Name], appointed [year] → tenure = X years
CFO: [Name], appointed [year] → tenure = X years
CEO compensation (FY[n]): Base $Xm | Bonus $Xm | Equity $Xm | Total $Xm
Incentive metrics tied to: [list]
CEO ownership: X shares (X% of shares outstanding) — [increasing / decreasing / stable] vs. prior year
CFO ownership: X shares — [increasing / decreasing / stable]
Say-on-pay vote: X% approval
Guidance policy: [Yes — specific EPS guidance / Yes — range / No — does not guide]
Earnings call tone: [summary: 2–3 sentences]
Key quotes: [1–2 direct quotes from earnings call or interviews, under 15 words each]
Notable news: [any controversies, bold claims, or notable public statements]
```

Pass this dossier into the API call alongside the financial and strategy findings.

---

## Step 2: Run the Checklist via Claude API with Extended Thinking

Make a single call to `claude-opus-4-6` with **extended thinking enabled** (`thinking: { type: "enabled", budget_tokens: 8000 }`).

Pass the following as context in the user message:
- The full strategy analysis findings (paste key sections or summary)
- The full financial analysis findings (paste key ratios, verdicts, earnings quality assessment)
- Any web research findings (management commentary, news, filings)
- The four sections below, verbatim

**System prompt for the API call:**
```
You are a rigorous fundamental investment analyst trained in the Munger/Buffett tradition. You are answering a structured investment checklist for a specific company.

Rules:
1. Only answer a question if you have direct evidence from the provided research. If you do not have sufficient information, respond with: "Insufficient information to assess."
2. Never speculate or fill gaps with assumptions.
3. Be direct and specific. Reference the actual data points (ratios, trends, quotes) that support each answer.
4. For each question in Sections 1–3, provide:
   (a) A verdict: Red Flag / Caution / Clear / Insufficient Information (Section 1), Fail / Caution / Clear (Section 2), or Concern / Neutral / Positive / Insufficient Information (Section 3)
   (b) One concise evidence statement of no more than 20 words citing the specific data point, ratio, or quote that determined the verdict
   (c) For any verdict of Caution, Red Flag, Fail, or Concern ONLY: an expanded explanation of 3–4 sentences (maximum) that unpacks the specific risk or concern with supporting data. This is in addition to, not instead of, the concise evidence line. Clear / Positive / Neutral / Insufficient Information verdicts do NOT receive expanded text.
5. For Section 4 (What Would Munger Ask?), produce the three sub-sections in flowing prose — not verdicts or badges. Ground every concern and question in the specific evidence from the research provided. Write as Munger would reason: inverted, skeptical, focused on permanent capital loss and business quality over the long run.
6. Use extended thinking to carefully weigh the evidence before answering.
```

---

## Step 3: The Three Checklist Sections

### Section 1: Warnings

Answer each of the following. For each item, assess whether the warning sign is **Present (Red Flag)**, **Partially Present (Caution)**, **Not Present (Clear)**, or **Insufficient Information**.

1. Is the company showing receivables rising faster than sales, or simply receivables rising at a fast pace?
2. Is the company showing rising receivable days? This may suggest that revenue increases were achieved by relaxing credit terms.
3. Is cash flow from operations (CFO) lower than operating profit and/or net profit for many years? This is a warning for premature revenue recognition.
4. Is the company capitalizing a high proportion of R&D expenses?
5. Is the company depreciating fixed assets too slowly, creating a boost to income — especially in industries experiencing rapid technological change?
6. Is the company taking big "bad" expenses during difficult times (big bath accounting)?
7. Is the company showing a sharp rise in profits just after a downturn — potentially indicating big bath accounting in the prior year?
8. Is the company boosting income using one-time or unsustainable activities?
9. Is the company turning proceeds from the sale of a business into a recurring revenue stream?
10. Is the company shifting normal operating expenses below the line?
11. Is the company routinely recording restructuring charges?
12. Is the company including proceeds received from selling a subsidiary as revenue?
13. Is the company creating reserves and releasing them into income in a later period?
14. Is the company showing sudden and unexplained declines in deferred revenue?
15. Is the company showing unexpectedly consistent earnings during a volatile time for the industry?

---

### Section 2: Avoid Checklist

For each criterion, assess whether the company **Fails (Avoid)**, **Passes (Clear)**, or **Insufficient Information**.

1. Does the company consume more cash than it generates?
2. Do managers boast of certainties and invincibility in their communications?
3. Does the company earn a poor return on capital?
4. Does the company operate in an industry where it is easy for new companies to enter and succeed?
5. Does the company operate in an unstable industry, potentially due to technological change or government regulations?
6. Does the company require a consistent infusion of new investment to grow?
7. Does the company lack the ability to increase prices (pricing power)?
8. Is the company unable to accommodate large volume increases with only minor additional capital investment?

---

### Section 3: Management Quality Checklist

For each question, provide the answer if evidence exists, or state "Insufficient information to assess."

1. How has the company performed in terms of sales and profit growth under current management over a ten-year period?
2. How has the balance sheet been managed over these years?
3. What is management's record in making capital allocation decisions?
4. For how long have the current managers been managing the business?
5. How are senior managers compensated? (salary, bonus structure, equity alignment)
6. Are senior managers continually increasing or reducing their ownership interest in the business?
7. Do the CEO and CFO issue guidance regarding earnings?
8. Does the management team focus on cutting unnecessary costs?
9. Does management only emphasize good news in its communications?
10. Does management think independently, or do they often follow what others in their industry are doing?
11. Are managers clear and consistent in their communications and actions with stakeholders?
12. Is the CEO self-promoting?

---

### Section 4: What Would Munger Ask?

Using the full body of research — industry analysis, financial analysis, valuation model, and the three checklist sections above — produce a Munger-style synthesis of this investment. Munger's framework centered on: moat durability and the forces that quietly erode it; pricing power as the clearest signal of a genuine franchise; management as stewards of long-term capital rather than short-term operators; inversion (what specific chain of events leads to permanent capital loss?); and circle of competence (what do we genuinely not know, and does that ignorance matter?).

This section has three sub-sections, each written in flowing analytical prose — no verdict badges, no bullet points.

#### 1. What Munger Would Be Worried About
Identify 3–5 specific concerns Munger would raise, grounded in the research. Each concern must name: the specific issue, the evidence from the industry analysis, financials, or checklist that surfaces it, and why Munger — given his emphasis on durable competitive advantage and permanent capital preservation — would find it disqualifying or deeply concerning. Frame these as Munger would: inversions of the bull case, not generic risks. "Increasing competition" is not a Munger concern. "The gross margin compression visible in the financials, combined with the low switching cost structure identified in the Five Forces, suggests the pricing power this business appeared to have was partly a function of a favourable cycle rather than a structural moat" — that is.

#### 2. What Munger Would Want to Know More About
Identify 2–4 specific questions whose answers would most change the investment conclusion. These are not generic data gaps — they are the precise questions a Munger-style analyst cannot answer from public sources but needs answered before committing capital. Each question should identify what information is missing, why it matters for the long-term thesis, and what the answer would have to be to make this a compelling investment.

#### 3. Munger's Verdict
A single paragraph direct synthesis: would Munger put this in the "too hard" pile, find it a genuinely wonderful business at a fair price, or land somewhere in between? Write it as a genuine analytical conclusion that reflects Munger's known preferences — concentration in businesses with durable moats, management he would trust with his family's capital, and prices that don't require heroic growth assumptions to work. Do not summarise the prior sections; synthesise them into a single, honest judgment.

---

## Step 4: Output as PDF

Read the PDF skill before producing output: `/mnt/skills/public/pdf/SKILL.md`

Use `reportlab` with `SimpleDocTemplate` and standard Helvetica styles. Plain white pages throughout. No cover page.

### PDF Structure

**Title block**: Company name in `Helvetica-Bold` 16pt, subtitle with ticker and date in `Helvetica` 10pt grey. Thin `HRFlowable` rule below.

**Section headers**: `Helvetica-Bold` 12pt

**Question rows**: Render as a single-row two-column `Table` per question. Do NOT use a two-row table structure — the evidence paragraph will be collapsed and invisible. Instead, combine all content into the left cell using `<br/>` tags within a single `Paragraph`:

```python
left_html = (
    f"<b>{i}.</b>  {question}<br/>"
    f"<font name='Helvetica-Oblique' size='8' color='#555555'>{evidence}</font>"
)
# For Caution / Red Flag / Fail / Concern verdicts ONLY, append expanded detail:
if detail:
    left_html += f"<br/><br/><font name='Helvetica' size='8.5' color='#333333'>{detail}</font>"

left_para = Paragraph(left_html, left_cell_style)
right_para = Paragraph(verdict_badge, right_cell_style)
tbl = Table([[left_para, right_para]], colWidths=[5.2*inch, 1.6*inch])
```

Set `VALIGN` to `'TOP'` (not `'MIDDLE'`) in the `TableStyle` so expanded cells align correctly.

**Verdict badge format** (right column): Colored bold label with a filled square prefix: `■ CAUTION`, `■ FAIL`, `■ CLEAR`, etc. Use the verdict's foreground color for the text.

**Verdict color coding**:
- 🔴 Red Flag / Fail / Concern → `#C0392B` text on `#FADBD8` background
- 🟡 Caution / Neutral → `#D68910` text on `#FCF3CF` background
- ✅ Clear / Positive → `#1E8449` text on `#D5F5E3` background
- ⬜ Insufficient Information → `#7F8C8D` text on `#F2F3F4` background

**Expanded detail** (for Caution / Red Flag / Fail / Concern only):
- Appears below the concise evidence line within the same left cell, separated by a blank line (`<br/><br/>`)
- 3–4 sentences maximum
- `Helvetica` 8.5pt, color `#333333`
- Should unpack the specific risk with supporting data — not a restatement of the evidence line

**Final Summary block** (after `PageBreak()`):
- Scorecard table: rows = verdict categories (Red Flag/Fail/Concern, Caution/Neutral, Clear/Positive, Insufficient Info), columns = Section 1 / Section 2 / Section 3 / Total
- A 3–5 sentence analyst synthesis paragraph: what the checklist collectively reveals about this company — its financial mechanics, structural risks, and management quality. Write it as a genuine analytical conclusion.
- Do NOT produce a binary Go/No-Go verdict — the checklist is an input to judgment, not a replacement for it.

**What Would Munger Ask? block** (last page, after `PageBreak()`):
- Section header: `Helvetica-Bold` 12pt — "What Would Munger Ask?"
- Thin `HRFlowable` rule below the header
- Sub-section headers (`Helvetica-Bold` 10.5pt): "What Munger Would Be Worried About", "What Munger Would Want to Know More About", "Munger's Verdict"
- Body text: `Helvetica` 9.5pt, `leading=14`, justified — flowing prose, no verdict badges or bullet rows
- Render each sub-section as one or more `Paragraph` flowables with `spaceBefore=8` between paragraphs

### Page order
Title block → Section 1: Warnings → Section 2: Avoid Checklist → Section 3: Management Quality → Summary → What Would Munger Ask?

Use `PageBreak()` between sections.

Save to `/mnt/user-data/outputs/[TICKER]_investment_checklist.pdf` and present with `present_files`.

---

## Analyst Conduct Rules

1. **Evidence-only**: Every answer must cite the specific data point, ratio, or quote from the research that supports it. No assertion without evidence.
2. **Gap honesty**: "Insufficient information" is a valid and respected answer. Do not paper over gaps.
3. **Expanded detail only for concerns**: Clear, Positive, Neutral, and Insufficient Information verdicts receive only the concise evidence line. Caution, Red Flag, Fail, and Concern verdicts receive both the concise evidence line AND 3–4 sentences of expanded explanation.
4. **No double-counting**: If a finding was surfaced in both the strategy analysis and financial analysis, cite both but count it once.
5. **Extended thinking is mandatory**: Do not skip the `thinking` parameter in the API call. These are high-stakes judgments that benefit from deliberate reasoning.
