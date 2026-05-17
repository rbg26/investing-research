---
name: strategy-industry-analysis
description: Conduct a detailed Porter's Five Forces analysis on the primary industry of a given stock ticker. Use this skill whenever a user asks for a Porter's Five Forces analysis, industry analysis, competitive analysis, or industry attractiveness assessment for a stock or company. Also trigger when a user asks "how attractive is the industry for [stock/company]?" or "analyze the competitive dynamics of [company]'s industry." Always use this skill for any five forces or industry structure analysis request — even if the user just says "do a five forces on AAPL" or "analyze Tesla's industry."
---

# Porter's Five Forces + Strategy Analysis Skill

## Overview

This skill produces a comprehensive industry and strategy analysis for a given company. The output is a well-researched, factually grounded PDF report — written in flowing analytical prose (not tables or bullet points) — covering Porter's Five Forces, a full strategy analysis (customer value proposition, profit value proposition, strategy coherence, and strategic flexibility), competitive advantage assessment, strategic refinement opportunities, and a closing synthesis.

**Core principle: Never state a fact you did not find through research. If information is not available, omit it or note its absence. Do not assume or invent data points.**

**Source hierarchy**: The company's 10-K and annual report are the most reliable sources. Web search results are useful for market data, competitor context, and industry sizing, but treat them with appropriate skepticism — much of what ranks highly in search results originates from PR sources, analyst summaries, or marketing copy. When a web claim contradicts or cannot be reconciled with 10-K disclosures, prefer the 10-K.

**Inversion**: Before writing any section, ask "what are the ways this business fails badly?" Research those failure modes explicitly. The most important risks are often the ones not foregrounded in the bull case. Strategic Refinement observations should be framed as failure modes — the specific chain of events that leads to permanent capital loss or structural moat erosion — not just gaps in the strategy.

**Circle of Competence**: Be explicit about what you know confidently vs. what you are inferring or cannot verify. Before writing the report, identify 2–4 questions about this business that you genuinely cannot answer from the 10-K or public sources. Surface those explicitly in a dedicated "Analyst Notes & Uncertainty" section in the PDF. Do not paper over genuine uncertainty with confident-sounding prose.

---

## Step 1: Identify the Stock and Industry

1. Ask the user for the stock ticker (if not already provided).
2. Do a quick search to identify the company and the industries it operates in.
3. **Confirm the primary industry with the user** before proceeding. Example:
   > "I've identified [Company] as primarily operating in [Industry A], with meaningful presence in [Industry B]. Which industry should I focus the analysis on?"
4. If the company spans multiple meaningful segments, **ask the user which segment to focus on**.
5. Once confirmed, proceed to research.

---

## Step 1b: Annual Report (10-K) — Prompt, Fetch, or Skip

After confirming the industry in Step 1, **always ask the user**:

> "Do you have the company's most recent 10-K you'd like to upload? If not, I'll pull it from SEC EDGAR automatically. Or let me know if you'd like to skip this step."

Then follow one of three paths:

### Path A — User uploads a 10-K PDF
Proceed directly to the extraction instructions below.

### Path B — No upload; fetch from SEC EDGAR automatically

Use the SEC EDGAR API to locate and download the most recent 10-K filing:

**Step 1: Look up the company's CIK number**
```python
import requests

# Use the EDGAR submissions API directly:
ticker_url = "https://www.sec.gov/files/company_tickers.json"
response = requests.get(ticker_url, headers={"User-Agent": "research@example.com"})
tickers = response.json()

# Find CIK for the ticker (stored as zero-padded 10-digit string)
cik = None
for entry in tickers.values():
    if entry['ticker'].upper() == TICKER.upper():
        cik = str(entry['cik_str']).zfill(10)
        break
```

**Step 2: Get the most recent 10-K filing metadata**
```python
submissions_url = f"https://data.sec.gov/submissions/CIK{cik}.json"
response = requests.get(submissions_url, headers={"User-Agent": "research@example.com"})
data = response.json()

# Find most recent 10-K in filings
filings = data['filings']['recent']
for i, form in enumerate(filings['form']):
    if form == '10-K':
        accession = filings['accessionNumber'][i].replace('-', '')
        filed_date = filings['filingDate'][i]
        break
```

**Step 3: Fetch the actual 10-K document**
```python
# Build the filing index URL
index_url = f"https://www.sec.gov/Archives/edgar/data/{int(cik)}/{accession}/{accession}-index.htm"

# Fetch the index page to find the primary document filename
# Then fetch the full document — typically an .htm file
doc_url = f"https://www.sec.gov/Archives/edgar/data/{int(cik)}/{accession}/"
```

Use `web_fetch` to retrieve the filing index, parse it to find the primary 10-K document filename, then fetch that document. Most modern 10-K filings are available as `.htm` files that `web_fetch` can retrieve directly.

**Important**: Always set a descriptive `User-Agent` header when calling SEC EDGAR — the SEC requires this and will block requests without it.

### Path C — User skips 10-K step
Proceed directly to Step 2 (web research). Note in the report that analysis is based on public sources only, without primary management disclosures.

### Extracting the Right Sections from the 10-K

A typical 10-K is 100–200 pages. Only the following sections are relevant to this analysis:

| Section | Why It Matters |
|---|---|
| **Item 1 — Business** | Market definition, competitive strengths, supplier relationships, product/segment breakdown, revenue mix, history and founding context |
| **Item 1A — Risk Factors** | Management's candid view of competitive threats, substitute risks, supplier concentration, buyer churn — legally required to be forthright |
| **Item 7 — MD&A** | Actual financial performance by segment, take rates, volume trends, margin trends — ground the strategy analysis in what has happened, not just what management intends |

> The Risk Factors section is the most honest section of any 10-K — companies are legally incentivised to disclose material risks. Treat it as management's unfiltered competitive intelligence.

> The MD&A is essential for grounding the strategy analysis in operating reality. Segment revenue, margin trends, and volume data tell you what the strategy has actually delivered, which may differ from what management describes in Item 1.

### What to Extract Per Force and Per Strategy Dimension

**For the Five Forces**, look for:
- How management defines their competitive set (often more expansive than the obvious)
- Explicit statements about competition intensifying or easing
- Named suppliers or supplier concentration disclosures
- Cancellation terms, churn admissions, customer retention risks
- Admissions that the business model is easy to copy
- Scale advantages quantified; regulatory barriers named
- How broadly management defines their competitive universe; behavioral or technological shifts flagged

**For the Strategy Analysis**, look for:
- The company's own description of its value proposition and competitive advantages (Item 1)
- How long the company has operated; founding history; when key partnerships or integrations were built
- Segment revenue and margin breakdown (MD&A) — what is actually generating profit and at what margin?
- Disclosed switching costs or integration depth with customers/merchants/partners
- Concentration disclosures: any single customer, supplier, or partner representing a material share
- Volume, take rate, or pricing trends over the past 2–3 years — are the profit mechanics improving or deteriorating?
- Workforce headcount changes — are they investing or rationalizing?

---

## Step 2: Deep Research (15–25 searches)

Conduct thorough web research to gather factual data for each of the five forces and the strategy analysis. Use the search and web_fetch tools extensively. Aim for 15–25 distinct searches.

**Source discipline**: Web results are useful for market sizing, competitor data, and industry context but carry inherent risk — many highly-ranked pages are PR-driven, analyst marketing copy, or aggregated from uncited sources. When using web data:
- Prefer primary sources (company filings, regulatory bodies, industry trade associations, government statistics) over aggregators
- Note when a claim comes from a research firm report vs. a primary disclosure
- When a web claim seems unusually specific or favorable and cannot be cross-referenced, either omit it or flag it as unverified

### Research Targets by Force

**Before researching the bull case — research the failure modes first (Inversion)**

Before gathering supporting evidence for each force, spend 3–5 searches explicitly trying to find the bear case: What do the most credible critics say? What has gone wrong for this company or its closest peers? What would have to be true for the moat to be worthless in 10 years? Inversion research often surfaces the most important facts — competitors gaining share, structural cost disadvantages, regulatory threats — that a standard bull-case research sequence would never surface. Document the key failure modes before writing. They will anchor the Strategic Refinement and Closing Synthesis sections.

**Industry Rivalry**
- Number of major competitors; market share distribution
- Whether the product/service is a commodity or differentiated specialty
- Industry revenue growth rate (recent years + forecasts)
- Cost structure: fixed vs. variable intensity; capital intensity
- Evidence of price wars, margin pressure, or competitive differentiation

**Buyer Power**
- Buyer concentration: fragmented (individual consumers) or concentrated (few large buyers)?
- Alternatives available to buyers; ease of switching
- Price sensitivity; switching costs; contracts; lock-in
- Threat of backward vertical integration by buyers

**Supplier Power**
- Who are the main suppliers? How concentrated is the supplier market?
- Alternatives available to the company; ease of switching
- Threat of forward vertical integration by suppliers
- Whether inputs are specialized or commodity; switching costs

**Threat of Substitutes**
- What alternative products/services could meet the same buyer need?
- How available, affordable, and attractive are substitutes?
- Trends (technology, regulation, behavior) driving substitution

**Barriers to Entry**
- Initial capital requirements; economies of scale; economies of scope
- Accumulated investment advantages: experience curves, brand equity, data, relationships
- Pre-emptive investments: patents, key resource capture, market saturation, switching costs, contracting constraints
- Regulatory or certification barriers

**For the Strategy Analysis** (additional research targets):
- Company founding date and founding context — when did it enter the market and what problem did it solve?
- Key acquisitions and their strategic rationale
- Segment-level TPV, revenue, or volume trends from recent earnings or filings
- Competitor landscape for each business segment (not just the primary one)
- Any publicly known leadership changes, restructurings, or strategic pivots in the past 24 months

---

## Step 3: Write the Analysis

Write the full report in **flowing analytical prose**. Do not produce tables or bullet-pointed exhibits — use the frameworks as *thinking tools* internally, then express findings as cohesive paragraphs.

---

### Part 1: Porter's Five Forces

#### Introduction (1 short paragraph)
Briefly identify the company, its primary industry, and the purpose of the analysis.

#### Force 1: Industry Rivalry — [High / Medium / Low]
Analyze using these drivers (woven into prose, not listed):
- Number of firms and market fragmentation; basis for market definition; likelihood of oligopolistic restraint
- Commodity vs. specialty: are products differentiated? Are distinctive attributes measurable? Is the buyer's value calculation simple?
- Cost structure (fixed vs. variable intensity)
- Industry rate of growth
Conclude the section with a clear verdict on rivalry intensity and why.

#### Force 2: Buyer Power — [High / Medium / Low]
Analyze using these drivers:
- Relative numbers: buyer concentration vs. seller concentration
- Alternatives available to buyers vs. alternatives available to sellers
- Threat of backward vertical integration
- Nature of the product (perishable vs. inventoriable)
- Contracting constraints; specialized assets or switching costs
- Buyer leverage; price sensitivity
- Buying process (scientific vs. emotional; how well-informed are buyers)
Conclude with a verdict.

#### Force 3: Supplier Power — [High / Medium / Low]
Analyze using these drivers:
- Relative numbers: supplier concentration; alternatives available; threat of forward vertical integration
- Nature of the product; contracting constraints
- Specialized assets or switching costs; supplier leverage
Conclude with a verdict.

#### Force 4: Threat of Substitutes — [High / Medium / Low]
Analyze what substitutes exist for the buyer's underlying need. What are buyers' alternatives for achieving the same goal? Consider price, availability, and behavioral/technological trends. Conclude with a verdict.

#### Force 5: Barriers to Entry — [High / Medium / Low]
Analyze using these drivers:
- Initial capital requirement
- Economies of scale; economies of scope
- Accumulated investment advantages (experience curve, brand, data, relationships)
- Pre-emptive investment advantages (sunk costs, key resource capture, market saturation, switching costs, contracting restraints)
Conclude with a verdict.

#### Overall Industry Attractiveness (1 paragraph)
Synthesize the five forces into a clear verdict: **attractive**, **moderately attractive**, or **unattractive**. Answer directly: **"If you had to decide whether to enter this industry, would you? Why or why not?"** Write in first person. Reference the most impactful forces. Explain what it would take to succeed if entry were pursued despite an unattractive verdict.

---

### Part 2: Strategy Analysis

The strategy analysis is grounded in facts on the ground — what the company has actually built, invested in, and achieved — not aspirational roadmaps. Use the 10-K's Item 1, Item 1A, and MD&A as primary anchors. Weight historical and current operating facts more heavily than management's forward-looking statements, which should be treated with the same skepticism applied to web sources.

#### Strategy Overview (2–3 paragraphs)
Describe the company's current strategic position: what it is, who it serves, and how it competes. Include founding context and historical investment where relevant — a company that has operated in a market for 25 years has built advantages that a company entering 5 years ago has not. Describe the key strategic choices embedded in the current model (e.g., generalist vs. specialist, intermediary vs. bank, branded vs. unbranded), and identify the central strategic question the business faces.

#### Customer Value Proposition (max 300 words)

Go beyond describing *what* the company offers — explain *why* it creates value and how that value translates into customer behavior and competitive position.

**Framing guidance**: A strong value proposition analysis traces first-order and second-order effects:
- *First-order*: What does the customer actually receive, and why does it matter to them specifically?
- *Second-order*: What does that customer behavior then produce for the company (pricing power, network effects, switching costs, etc.)?

**Example framing (do not copy verbatim, apply the logic)**:
> "Planet Fitness' value proposition lies in its affordable, judgment-free fitness offering for casual gym-goers. Their customers can exercise 24/7 without intimidation — a first-order differentiation that drives customer satisfaction and low churn. The second-order consequence is that low churn sustains the membership base needed to support low prices, which in turn attracts the same non-intimidating demographic, reinforcing the brand positioning."

Apply the same causal chain logic to the company being analyzed. The proposition should explain *why* the company has the pricing power, network position, or margin structure it does — not just describe the products it sells.

#### Profit Value Proposition (max 300 words)

Explain *why* the company is profitable, not just *what* it earns. Anchor in the operating mechanics disclosed in the 10-K (segment margins, take rates, volume trends, revenue mix).

**Key questions to answer**:
- What is the structural reason the company can charge what it charges? (Network effects? Switching costs? Supply scarcity? Trust premium?)
- Which profit levers are durable (rooted in structural advantage) vs. cyclical or at risk of compression?
- What are the first- and second-order effects that sustain or threaten the profit model?

For example: if the profit comes from a transaction margin on branded products, explain *why* that margin exists structurally — is it because merchants cannot drop the product without losing consumer traffic? Because consumers pay a trust premium? Because integration switching costs are high? Then trace whether those structural foundations are strengthening or eroding based on the operating data.

Disclose when profit drivers are under pressure — take rate compression, volume deceleration, segment mix shift — and explain the structural cause, not just the reported outcome.

#### Strategy Coherence and Fit with the Competitive Environment (2–3 paragraphs)

Do not write separate "internal consistency" and "external consistency" sections. Weave both into a single integrated assessment:

- Where does the strategy align well with the competitive environment (Five Forces)? Which strategic choices address the most significant threats?
- Where does the strategy create internal tensions or fail to address identified competitive risks?
- What are the key choices embedded in the strategy (e.g., intermediary vs. bank, branded vs. unbranded) and what structural constraints do those choices impose?
- Ground the coherence assessment in specific operating facts — disclosed metrics, segment trends, disclosed structural risks in the 10-K — not in management's aspirational language

#### Strategic Flexibility (1–2 paragraphs)
Assess the company's ability to adapt. Consider: financial resources (cash, FCF); regulatory or licensing assets that provide optionality; portfolio breadth as a buffer; and constraints (organizational complexity, supplier dependencies, capital allocation tensions). Note any recent leadership transitions or strategic pivots that affect the flexibility assessment.

---

### Part 3: Competitive Advantage Assessment

#### Competitive Advantage (max 300 words)

Identify the company's genuine competitive advantages — the structural reasons a well-managed competitor cannot simply replicate the company's position. Distinguish between:
- **Durable advantages**: rooted in accumulated investment, network effects, regulatory position, or trust — advantages that compound over time and are difficult to displace
- **Fragile or contested advantages**: advantages that exist today but are under active competitive pressure

Conclude with a clear verdict: strong, moderate, or limited competitive advantage, and state the single most important reason.

---

### Part 4: Strategic Refinement Opportunities

#### Strategic Refinement Opportunities (2–3 paragraphs)

**Frame every observation as a failure mode, not a gap.** The question is not "what could be improved?" but "what is the specific chain of events — plausible, grounded in facts already in evidence — that leads to permanent moat erosion or significant capital loss?" Each refinement observation should name the trigger, the mechanism, and the compounding consequence.

Identify the most material structural gaps or tensions in the current strategy. Ground every observation in the Five Forces analysis, the strategy assessment, and — where possible — specific disclosures in the 10-K (volume trends, concentration disclosures, margin trajectories, risk factor language). Do not surface risks that are well-known and already fully reflected in the strategy. Prioritize observations that:
- Flow logically from the Five Forces findings to the strategy (e.g., a force identified as high-threat that the strategy does not address)
- Reveal a tension between two parts of the strategy (e.g., a pricing strategy that improves margins but accelerates customer loss)
- Identify a profit driver that is structurally eroding (e.g., a trust premium that depends on conditions that are diminishing)
- Describe a compounding failure path: if X continues, then Y follows, then Z becomes permanent

Avoid speculating about unannounced products, future M&A, or management aspirations not grounded in current operational facts.

---

### Part 5: Closing Synthesis + Executive Summary

#### Closing Synthesis (max 300 words)
Synthesize the full analysis into the single most important insight about this company's competitive position. What is the central tension or structural challenge the company faces? What would have to be true for the strategy to succeed? This section should read as the honest conclusion of a thorough analysis — not a summary of prior sections, but a distillation of the most important finding.

---

### Part 6: Analyst Notes & Uncertainty (Circle of Competence)

#### What This Analysis Cannot Confirm (1 paragraph)

This section is not optional. Every analysis has genuine limits — questions that matter for the investment thesis but cannot be answered from the 10-K or public sources. Name them explicitly.

**Format**: List 2–4 specific questions about this business that remain unresolved after full research, and state briefly why they matter and why they are hard to answer. Examples of the type of question to surface:
- "We cannot independently verify the rate at which branded checkout share is shifting to Apple Pay — PayPal does not disclose this, and third-party estimates vary widely. This matters because the entire profit thesis rests on the durability of branded checkout take rates."
- "Management quality and capital allocation discipline under the new CEO are not yet observable from a full operating cycle. This introduces uncertainty into any thesis dependent on execution."
- "The actual switching cost for a mid-size merchant running Braintree alongside Stripe is not quantifiable from public sources. Our switching cost assessment is qualitative."

**The discipline here is Munger's circle of competence test**: if you cannot clearly articulate the boundaries of what you know, you do not know where your circle ends. An honest "I don't know" is more valuable than false precision.

#### Executive Summary (200–300 words)
Write a crisp, standalone summary of the full analysis. Cover: company overview, the verdict on each of the five forces (with 1–2 key reasons each), the overall industry attractiveness verdict, the core of the customer and profit value propositions (with the *why*), the competitive advantage verdict, the key failure mode identified through inversion, and the closing strategic insight. Suitable for a reader who only reads this section.

---

## Step 4: Output as PDF

Read the PDF skill before producing output:
`/mnt/skills/public/pdf/SKILL.md`

Use `reportlab` with `SimpleDocTemplate` and standard Helvetica styles. **No cover page. No colored backgrounds. No custom page templates. No fancy design.** Plain white pages throughout.

### PDF structure

Use only these style elements:
- **Title block** (page 1, top): Company name in `Helvetica-Bold` 16pt, then a subtitle line with industry, ticker, and date in `Helvetica` 10pt grey. Follow with a thin `HRFlowable` rule.
- **Part headers** (`H1`): `Helvetica-Bold` 12pt, `spaceBefore=14`
- **Section headers** (`H2`): `Helvetica-Bold` 10.5pt, `spaceBefore=10`
- **Verdict line**: `Helvetica-Bold` 9.5pt inline label, e.g. `Verdict: HIGH` — placed immediately after the section header, before body prose
- **Body text**: `Helvetica` 9.5pt, `leading=14`, justified
- **Sources**: `Helvetica` 8.5pt, numbered `[1] ...` format
- Thin `HRFlowable` (0.5pt, grey) after each Part header only

### Page order (no cover page)
Page 1 starts directly with the **Executive Summary**, followed by:
Five Forces → Strategy Analysis → Competitive Advantage → Strategic Refinement → Closing Synthesis → Analyst Notes & Uncertainty → Sources

Use `PageBreak()` between major parts.

Present the revised PDF to the user using the `present_files` tool.

---

## Step 5: Self-Assessment and Revision

After producing the first draft PDF, score the analysis as a rigorous MBA strategy professor would. If the score is below 4.0, identify the gaps, fill them with additional research, and produce a revised PDF. Only present the final version to the user.

### Scoring rubric (1–5)

Score the analysis on these five analytical dimensions. Each is worth 1 point. Award partial credit (0.5) where a dimension is partially addressed.

| Dimension | What earns the point |
|---|---|
| **Five Forces depth** | Each force is grounded in specific facts; verdicts are defensible and calibrated; rivalry and substitutes — typically the highest-impact forces — receive the most rigorous treatment |
| **Value proposition causality** | Both VPs trace first- and second-order effects; explains *why* the company earns what it earns structurally, not just *what* it sells |
| **Geographic and segment completeness** | Analysis accounts for material non-domestic revenue, segment-level differences in competitive dynamics, and any business units with structurally distinct economics |
| **Inversion and failure modes** | Bear case is as rigorously developed as the bull case; failure modes are written as specific causal chains (trigger → mechanism → compounding consequence), not generic risks |
| **Circle of competence honesty** | Analyst Notes names 2–4 genuine unresolvable questions; does not paper over uncertainty with confident-sounding prose |

A score of **4.0 or above** is acceptable. A score of **3.5 or below** requires revision.

### What to look for when self-scoring

Common gaps that lower scores — check each explicitly:

- **International strategy undertreated**: If the company generates more than 15% of revenue internationally, does the analysis address how competitive dynamics differ outside the home market? (e.g. open banking regulations in Europe, super-app dominance in Asia, different substitute threats by region)
- **Segment economics flattened**: If the company has two or more business segments with structurally different take rates, margins, or competitive positions, does the analysis treat them separately rather than averaging across the whole?
- **Substitutes conflated with rivals**: Substitutes are products that meet the same underlying need through a different mechanism — not just competitors offering the same product cheaper. If the substitutes section describes only cheaper versions of the same product, it is incomplete.
- **Strategy coherence section is descriptive, not evaluative**: Does the coherence section identify specific *tensions* between the strategy and the competitive environment, or does it merely summarize what management says the strategy is?
- **Failure modes are generic**: "Increasing competition" or "macroeconomic headwinds" are not failure modes. A failure mode must name the specific trigger, the mechanism by which it propagates, and the end state.

### Revision process

1. State the score and the specific gaps — one sentence per dimension that fell short.
2. For each gap below full credit, conduct targeted additional research (1–3 searches) to fill it.
3. Revise only the sections that need updating. Do not rewrite sections that already meet the standard.
4. Rebuild the PDF and present the revised version to the user.

---

## Writing Guidelines

- **Prose only**: Write like a strategy consultant. Paragraphs, not lists.
- **Facts on the ground**: Weight the 10-K and demonstrated operating history more heavily than management's forward-looking language or web sources. If the operating metrics tell a different story than the strategic narrative, say so.
- **Why, not just what**: For the value proposition sections especially, go beyond describing what the company does to explain the structural reasons it creates value and earns profit. Trace first- and second-order effects.
- **Inversion first**: Research and articulate the failure modes before writing the analysis. The bear case should be as rigorously developed as the bull case.
- **Name what you don't know**: The Analyst Notes & Uncertainty section is mandatory. An honest acknowledgment of knowledge limits is a sign of analytical rigor, not weakness.
- **Facts only**: Every specific claim must come from research. If you couldn't find a data point, omit it or write "publicly available data on this driver is limited."
- **Verdicts matter**: Each force section must end with a clear High / Medium / Low verdict and a 1-sentence rationale. The competitive advantage section must state a clear verdict.
- **Tone**: Analytical, confident, and concise. Graduate-level business school case analysis quality.
- **Length**: Five Forces: ~800–1,200 words. Strategy Overview + Coherence + Flexibility: ~400–600 words. Customer VP: max 300 words. Profit VP: max 300 words. Competitive Advantage: max 300 words. Strategic Refinement: ~300–500 words. Closing Synthesis: max 300 words. Analyst Notes: ~150–250 words. Executive Summary: 200–300 words.

---

## Citation Requirements

**Every data point, statistic, or sourced fact must be cited inline in the PDF.** This is what distinguishes a credible research report from unsupported assertions.

### Citation Format

Use superscript footnote numbers inline: e.g., `...the top ten players account for approximately 60% of global market share<super>1</super>...`

At the end of the PDF (after the Executive Summary), include a **Sources** section listing every citation in order.

### Rules
- **Cite every specific statistic**: market size figures, growth rates (CAGRs), market share percentages, number of firms, churn rates, take rates, volume figures, and any other quantitative claim.
- **Cite named sources**: whenever you reference a specific organization's data, cite it even if paraphrased.
- **Do not cite general knowledge**: verdicts, logical inferences, and framework-level reasoning do not need citations.
- **No fabricated citations**: if you cannot identify the source of a data point, either omit it or note that the source is unverified.
- **In the PDF**: implement superscript numbers using ReportLab's `<super>` XML tag inside Paragraph objects.
- **Executive Summary citations**: re-use the same footnote numbers from the main body.

### Citing the 10-K

When a claim originates from the annual report:
- Cite inline with a superscript number
- In the Sources section, format as: `[Company] Annual Report (Form 10-K), "[Item/Section]," Fiscal Year [YYYY].`
- **Prefer 10-K citations over web sources** when both support the same claim — management's own disclosures carry more credibility than analyst summaries
- When a 10-K passage *strengthens or contradicts* a finding from web research, note both and reconcile them in the prose
