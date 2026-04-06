---
name: verndale:enrich-batch
description: Enrich a batch gap analysis document with Salesforce references, client evidence, assigned decisions, and Confluence export
argument-hint: <batch-number> <batch-title> <focus-description> <doc-path> [items-list]
---

<objective>
Enrich Batch gap analysis document on for Boston Beer Company's MerchTank-to-Salesforce B2B Commerce migration.

Batch focus: this

The enriched document must match the depth, format, and quality of completed batches (Batch 1: Global & Content, Batch 2: Products & Path to Purchase). Each feature row gets Salesforce capability descriptions, documentation links, BBC-specific MerchTank evidence, actionable next steps, and a recommended decision.

Both Markdown and Confluence HTML deliverables are produced.
</objective>

<context>
Batch document to enrich: @session,

Reference format (Batch 2): @BostonBeerCompany/gap-analysis-batch-2-products-path-to-purchase.html

Critical requirements: @BostonBeerCompany/Batch-docs/TBBC-BBC Critical Requirements-270326-115727.pdf

Project context: @BostonBeerCompany/CLAUDE.md

Existing meeting analyses:

- @BostonBeerCompany/meetings/02-virtual-warehouse-walkthrough/analysis/gap-analysis-bbc-vw-walkthrough.md
- @BostonBeerCompany/meetings/03-custom-requests/analysis/gap-analysis-bbc-custom-requests-meeting.md
- @BostonBeerCompany/meetings/04-fulfillment-demo/analysis/gap-analysis-boston-beer-company-fulfillment-demo.md

Previous batch deliverables: !`ls BostonBeerCompany/gap-analysis-batch-*.md BostonBeerCompany/gap-analysis-batch-*.html 2>/dev/null`
</context>

<process>
1. **Scan existing deliverables** — Read Batch 1 and Batch 2 gap analysis documents to understand the exact format, column structure, enrichment depth, and Confluence markup patterns.

2. **Extract current batch content** — Read the source document at session,. If it's a .doc file, convert via `textutil -convert html` first. Extract all existing feature rows, their current content depth, and any assigned decisions.

3. **Cross-reference requirements** — Read the BBC Critical Requirements PDF and identify all requirements tagged to Batch on. Read all meeting analysis documents for evidence relevant to this batch's focus area (this).

4. **Coverage gap analysis** — Compare extracted features against the user-provided items list (create-a). Identify:
   - Features present but needing enrichment
   - Features missing entirely that need to be added
   - Features that overlap with other batches (flag but keep)

5. **Ask clarifying questions** — Use AskUserQuestion to confirm:
   - Output format preference (Markdown, Confluence HTML, or both)
   - Whether to assign recommended decisions or leave blank for TA review
   - How to handle any batch overlap items found

6. **Enrich each feature** — For every feature row, populate all 6 columns:
   - **#**: Sequential number within section
   - **Feature & Supporting Documentation**: Feature name + Salesforce Help article link
   - **High-Level Functionality (Salesforce)**: Detailed description of what SF B2B Commerce provides OOTB for this capability
   - **Current Site Observations**: BBC MerchTank evidence from meeting analyses, screenshots, and transcripts. Include "Evidence from MerchTank:" header. Reference Critical Requirements where applicable.
   - **Recommended Next Steps**: Actionable bullet list for implementation
   - **Decision**: One of CONFIGURATION (Green) / CUSTOM (Yellow) / NEW BUILD (Red) / DISABLE (Grey)

7. **Organize into sections** — Group features into logical sections matching the batch focus. Each section gets its own h2 heading and table.

8. **Add summary tables**:
   - Decision Distribution (by section and total with percentages)
   - BBC Critical Requirements Coverage (requirement # -> feature # -> decision)
   - Assumptions Register (ID, Assumption, Impact if Wrong, Validation Method, Owner, Status)
   - Open Questions (prioritized P1/P2/P3 with stakeholder assignments)

9. **Write Markdown** — Save to `BostonBeerCompany/gap-analysis-batch-{number}-{slug}.md`

10. **Write Confluence HTML** — Convert to Confluence Storage Format matching Batch 2 exactly:
    - `<ac:structured-macro ac:name="info">` header block
    - `<ac:structured-macro ac:name="status">` decision badges
    - `<ac:structured-macro ac:name="toc">` table of contents
    - `<table data-layout="full-width">` with colgroup sizing
      Save to `BostonBeerCompany/gap-analysis-batch-{number}-{slug}.html`

11. **Verify coverage** — Cross-check all features against: - User's original items list - BBC Critical Requirements tagged to this batch - Meeting analysis findings relevant to this batch's focus
    </process>

<verification>
Before completing, verify:
- Every item from the user's requirements list has a corresponding feature row
- All BBC Critical Requirements tagged to this batch are cross-referenced in feature tables AND in the coverage summary table
- Every feature has all 6 columns populated (no empty cells)
- Every feature has exactly one decision assigned
- Salesforce Help links are included for every feature
- The Confluence HTML uses the exact same markup patterns as Batch 2
- Summary statistics match the actual feature count and decision distribution
- Assumptions register has at least 5 entries with impact assessments
- Open questions are prioritized (P1/P2/P3) with stakeholder assignments
</verification>

<output>
Files created:
- `BostonBeerCompany/gap-analysis-batch-{number}-{slug}.md` — Enriched markdown source
- `BostonBeerCompany/gap-analysis-batch-{number}-{slug}.html` — Confluence Storage Format export

The markdown serves as the canonical source. The HTML is a direct conversion for Confluence import.
</output>

<success_criteria>

- All features from the user's items list are covered with dedicated rows
- Each feature row has: SF capability description, SF documentation link, BBC MerchTank evidence, actionable next steps, and recommended decision
- BBC Critical Requirements are cross-referenced in feature tables and summary
- Document structure matches Batches 1 and 2 (header, decision key, sectioned tables, summary, assumptions, open questions)
- Both Markdown and Confluence HTML deliverables produced
- Confluence HTML uses correct storage format markup (status macros, info blocks, full-width tables)
- Coverage verification passes with no gaps
  </success_criteria>
