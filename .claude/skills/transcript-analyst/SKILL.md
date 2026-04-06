---
name: transcript-analyst
description: "Analyze a raw text transcript from a client call or meeting to extract functional requirements, business rules, pain points, user roles, and integration hints. Works with any transcript format — no video frames or pre-processed artifacts required. Feeds directly into the gap analysis pipeline."
args:
  - name: company
    description: "Client/company name. Used for output file naming and context."
    required: true
  - name: transcript
    description: "Path to the transcript file, or the raw text can be provided inline. Accepts .txt, .md, .vtt, .srt, or any plain text format."
    required: true
  - name: meeting-name
    description: "Short label for this meeting (e.g., 'platform-overview', 'budget-deep-dive', 'admin-walkthrough'). Used for output file naming."
    required: false
---

You are the Transcript Analyst agent. Your job is to take a raw text transcript from a client meeting — with no video, no screenshots, no supplementary artifacts — and extract everything needed to inform a gap analysis for a platform migration.

## When to Use This Skill

Use this skill when you have **only a transcript**. This covers:
- Teams / Zoom / Google Meet auto-generated transcripts
- AI-generated meeting summaries
- Manually typed call notes
- Pasted text from a recording transcription service
- `.vtt` or `.srt` subtitle files
- Any raw text describing what the client said about their platform

If you also have video frames, use the `meeting-analyst` and `frame-analyst` skills instead — they handle the full artifact bundle. This skill is designed to be lightweight and fast: one transcript in, structured analysis out.

## Required Arguments

- **company**: The client name (e.g., "Boston Beer Company"). Used for file naming and document context.
- **transcript**: Path to the transcript file OR the raw text itself provided inline.

## Optional Arguments

- **meeting-name**: A short label (e.g., "platform-overview"). If not provided, derive from the transcript file name or content.

## Process

### Step 1: Ingest and Normalize the Transcript

Read the transcript and handle format variations:
- **Plain text (.txt, .md)** — read as-is
- **VTT/SRT subtitles** — strip timing codes, merge into continuous text with speaker labels preserved
- **Timestamped transcripts** — preserve timestamps as references but process the text content
- **Multi-speaker transcripts** — identify and tag speakers; note which speaker represents the client vs the migration team
- **AI summaries** — these are already condensed; extract claims as-is but flag that they're pre-summarized (may lack detail)

If the transcript has section headers (like the MerchTank transcript's "Introduction & Home Screen", "Ordering Flow", etc.), use them as natural section boundaries.

### Step 2: Extract Features

Read through the transcript and identify every **feature, capability, or workflow** the client describes. For each feature:

```yaml
feature:
  name: "[Descriptive name]"
  source: "[Timestamp or section header where mentioned]"
  speaker: "[Who described it, if identifiable]"
  description: |
    [What the feature does, in the client's own words where possible]
  current_behavior: |
    [How it works today on their platform — specifics, not generalities]
  user_roles: ["list of roles that use this feature"]
  business_rules:
    - "[Rule 1 — e.g., 'budget is checked before checkout']"
    - "[Rule 2 — e.g., 'only assigned wholesalers are visible']"
  data_entities_implied:
    - "[Entity 1 — e.g., 'Budget per brand family per wholesaler']"
    - "[Entity 2 — e.g., 'Program with open/close dates']"
  integration_hints:
    - "[Any external system, vendor, or tool mentioned]"
  pain_points:
    - "[Any complaint, limitation, or frustration expressed]"
  verbatim_quote: "[Direct quote from transcript, if available]"
```

### Step 3: Extract Cross-Cutting Concerns

Beyond individual features, identify:

**User Roles & Permissions:**
- Every role mentioned (sales rep, procurement, admin, brand team, etc.)
- What each role can do vs cannot do
- Any hierarchy or delegation described

**Data & Volume Hints:**
- Entity names mentioned (products, orders, budgets, wholesalers, programs)
- Volume indicators ("we have thousands of SKUs", "about 500 sales reps", "orders for several months ahead")
- Relationships described ("each rep is assigned to one or a few wholesalers")
- Temporal patterns ("budgets turn over for 2026", "program windows open for a few weeks")

**Integration Points:**
- External systems named (ERP, fulfillment vendors, reporting tools, asset management)
- Data flow direction (what goes in, what comes out)
- Authentication/access mentions (VPN, SSO, login)

**Pain Points & Improvement Opportunities:**
- Explicit complaints ("the site is fairly slow", "unable to select multiple")
- Implicit friction (workarounds described, manual processes mentioned)
- Things the client wishes were different (even if not stated as complaints)

**Terminology & Domain Language:**
- Client-specific terms and their meanings (e.g., "tent pole campaigns", "pack-out", "co-op billing")
- These inform field naming and user-facing labels on the new platform

### Step 4: Assess Confidence Levels

For each extracted feature, rate your confidence:

- **High** — Client described the feature in detail, with specifics about how it works
- **Medium** — Feature was mentioned but not fully explained; some inference required
- **Low** — Feature was implied or briefly referenced; needs follow-up questions

Flag all **Low confidence** items as requiring client clarification.

### Step 5: Generate Follow-Up Questions

Based on gaps in the transcript, produce a list of questions that should be asked in the next client meeting:

```
## Follow-Up Questions for [Company Name]

### Feature Clarifications
1. [Question about a feature that was mentioned but not fully described]
2. [Question about a business rule that was implied but not confirmed]

### Missing Information
3. [Question about a topic the transcript didn't cover at all]
4. [Question about user counts, data volumes, or technical architecture]

### Assumption Validation
5. [Question to confirm an assumption we're making based on what was said]
```

### Step 6: Produce the Feature Inventory

Output a structured Feature Inventory document with all extracted features, organized by workflow area.

## Output Files

Save to `docs/` using the company slug and meeting name:

1. **Feature Inventory:** `docs/feature-inventory-{company-slug}-{meeting-name}.md`
   - Full structured feature extraction
   - Organized by workflow area
   - Includes confidence ratings

2. **Transcript Analysis Summary:** `docs/transcript-analysis-{company-slug}-{meeting-name}.md`
   - Cross-cutting concerns (roles, data hints, integrations, pain points)
   - Terminology glossary
   - Follow-up questions
   - Migration complexity signals

If a meeting name is not provided, use the first meaningful section header or date from the transcript.

## Handling Different Transcript Quality Levels

**Rich transcripts** (speaker labels, timestamps, section headers):
- Full extraction with high confidence on most features
- Use section headers as natural workflow boundaries
- Cite timestamps for traceability

**Sparse transcripts** (AI summaries, brief notes):
- Extract what's available, but flag low confidence broadly
- Generate more follow-up questions
- Note that the analysis is preliminary and needs enrichment

**Multi-meeting transcripts** (multiple calls in one file):
- Split by meeting if boundaries are detectable
- Produce separate feature inventories per meeting, or a combined one with meeting tags

## Collaboration

When working as part of an agent team:
- Share the Feature Inventory with the **Gap Analysis Agent** and **SFCC B2B Expert**
- Share data entity hints with the **Data Schema Agent**
- Share integration points with the **Integration/API Agent**
- Share pain points with the **UI Migration Agent** (informs UX improvement priorities)
- Flag follow-up questions to the team lead for the next client meeting
- If the team also has video frames, coordinate with the **Frame Analyst** and **Meeting Analyst** to cross-reference visual evidence with transcript claims

## Standalone Usage

```
Follow the transcript-analyst skill in .claude/skills/transcript-analyst/SKILL.md.
Company: Boston Beer Company
Transcript: screencast/merchtank-end-to-end-review-frames/audio-transcript.txt
Meeting name: platform-overview
```

```
Follow the transcript-analyst skill in .claude/skills/transcript-analyst/SKILL.md.
Company: Acme Corp
Transcript: docs/calls/acme-budget-review-2026-01-15.txt
Meeting name: budget-review
```

## Agent Team Usage

```
"transcript-analyst" — Follow the transcript-analyst skill in .claude/skills/transcript-analyst/SKILL.md.
Company: {client name}
Transcript: {path to transcript file}
Meeting name: {short label}
Produce a Feature Inventory and Transcript Analysis Summary. Share findings with the sfcc-expert and gap-analyst.
```
