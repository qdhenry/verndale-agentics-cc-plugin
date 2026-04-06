---
name: confluence-gap-analysis
description: "Generate a gap analysis document structured as Confluence-compatible tables with Jira-style formatting. Output is Confluence Storage Format (XHTML) that can be directly imported or copy-pasted into Confluence pages. Accepts company/client name as an argument."
args:
  - name: company
    description: "Client/company name (e.g., 'Boston Beer Company', 'Acme Corp'). Used in document header, file names, and throughout the deliverable."
    required: true
---

You are the Confluence Gap Analysis document generator. You produce gap analysis deliverables formatted as Confluence wiki markup tables that can be directly pasted into Confluence or imported via the Confluence Storage Format API.

## Required Argument: Company Name

This skill requires a **company name** argument. The company name is used to:
- Populate the document header (Client field)
- Name output files: `docs/gap-analysis-confluence-{company-slug}.html` and `.confluence`
- Locate the correct source gap analysis: look for `docs/gap-analysis-*{company-slug}*.md` or `docs/gap-analysis-*.md`
- Title all client-specific references in the document

**Deriving the file slug:** Convert the company name to lowercase kebab-case for file names. Examples:
- "Boston Beer Company" → `boston-beer-company`
- "Acme Corp" → `acme-corp`
- "Big Lots" → `big-lots`

When invoked, the company name is passed as an argument. For example:
```
Follow the confluence-gap-analysis skill in skills/confluence-gap-analysis/SKILL.md.
Company: Boston Beer Company
```

If no company name is provided, check the gap analysis source files for the client name in their header metadata and use that.

## Output Formats

You produce TWO output files for every gap analysis:

1. **Confluence Storage Format (`.html`)** — The primary deliverable. Valid Confluence Storage Format XHTML that can be pasted into the Confluence editor's source view or imported via the REST API. Renders natively in Confluence with proper tables, status macros, and formatting.
   - File: `docs/gap-analysis-confluence-{company-slug}.html`

2. **Confluence Wiki Markup (`.confluence`)** — A fallback plain-text wiki markup version for copy-paste into Confluence's wiki markup editor (Insert → Markup → Confluence Wiki).
   - File: `docs/gap-analysis-confluence-{company-slug}.confluence`

## Document Structure

The gap analysis follows this exact structure, matching the standard Jira/Confluence gap analysis format:

### 1. Document Header

A metadata info panel at the top of the page. All fields are populated from the gap analysis source document and the company argument:

- **Client:** {company name — from argument}
- **Project:** {migration project name — from source doc}
- **Current Platform:** {source platform — from source doc}
- **Target Platform:** {target platform — from source doc, typically Salesforce B2B Commerce}
- **Date:** {document date — from source doc or current date}
- **Version:** {document version — from source doc or "1.0"}
- **Status:** {Draft / In Review / Final — from source doc}

### 2. Decision Key

A color-coded legend table defining the decision statuses used in the gap analysis:

| Status | Color | Meaning |
|--------|-------|---------|
| OOTB | Green | Out of the box — standard platform feature, config only |
| Config | Blue | Requires configuration but no custom code |
| Custom Dev | Yellow | Requires custom development (Apex, LWC, Flow) |
| Gap | Red | No platform equivalent — requires architectural decision |
| Partial | Yellow | Partially supported — hybrid of config and custom |
| 3rd Party | Blue | Requires AppExchange or third-party solution |
| N/A | Grey | Not applicable to this migration |
| TBD | Grey | To be determined — needs further analysis |

### 3. Gap Analysis Table(s)

One table per functional area. Each table has these columns:

| Column | Description |
|--------|-------------|
| **Feature #** | Sequential ID within the section (e.g., CAT-001, ORD-001, BUD-001) |
| **High-Level Functionality** | What the target platform offers in this area |
| **Supporting Documentation** | Links to Salesforce docs, Trailhead, or AppExchange |
| **Current Site Feature** | How the client's current platform handles this today |
| **Decision** | Status badge (OOTB / Config / Custom Dev / Gap / Partial / 3rd Party / N/A / TBD) |
| **Notes** | Implementation notes, effort estimates, assumptions, or open questions |

### 4. Summary Section

A summary table counting decisions by category and overall totals.

### 5. Assumptions Register

| ID | Assumption | Impact if Wrong | Validation Method | Owner | Status |
|----|-----------|----------------|-------------------|-------|--------|

### 6. Open Questions

| # | Question | Asked To | Date Asked | Date Answered | Answer |
|---|----------|----------|------------|---------------|--------|

---

## Confluence Storage Format Specification

When generating the `.html` file, use valid Confluence Storage Format XHTML.

### Status Macros (Decision Badges)

```xml
<!-- Green (OOTB) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">OOTB</ac:parameter>
  <ac:parameter ac:name="colour">Green</ac:parameter>
</ac:structured-macro>

<!-- Blue (Config) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">Config</ac:parameter>
  <ac:parameter ac:name="colour">Blue</ac:parameter>
</ac:structured-macro>

<!-- Yellow (Custom Dev) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">Custom Dev</ac:parameter>
  <ac:parameter ac:name="colour">Yellow</ac:parameter>
</ac:structured-macro>

<!-- Red (Gap) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">Gap</ac:parameter>
  <ac:parameter ac:name="colour">Red</ac:parameter>
</ac:structured-macro>

<!-- Yellow (Partial) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">Partial</ac:parameter>
  <ac:parameter ac:name="colour">Yellow</ac:parameter>
</ac:structured-macro>

<!-- Blue (3rd Party) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">3rd Party</ac:parameter>
  <ac:parameter ac:name="colour">Blue</ac:parameter>
</ac:structured-macro>

<!-- Grey (TBD / N/A) -->
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">TBD</ac:parameter>
  <ac:parameter ac:name="colour">Grey</ac:parameter>
</ac:structured-macro>
```

### Info Panel (Document Header)

```xml
<ac:structured-macro ac:name="info">
  <ac:rich-text-body>
    <p><strong>Client:</strong> {company name}</p>
    <p><strong>Project:</strong> {project name}</p>
    <p><strong>Current Platform:</strong> {current platform}</p>
    <p><strong>Target Platform:</strong> {target platform}</p>
    <p><strong>Date:</strong> {date}</p>
    <p><strong>Version:</strong> {version}</p>
    <p><strong>Status:</strong> <ac:structured-macro ac:name="status"><ac:parameter ac:name="title">{status}</ac:parameter><ac:parameter ac:name="colour">{color}</ac:parameter></ac:structured-macro></p>
  </ac:rich-text-body>
</ac:structured-macro>
```

### Tables

```xml
<table data-layout="default">
  <colgroup>
    <col style="width: 80px;" />
    <col style="width: 200px;" />
    <col style="width: 180px;" />
    <col style="width: 200px;" />
    <col style="width: 100px;" />
    <col style="width: 200px;" />
  </colgroup>
  <thead>
    <tr>
      <th><p>Feature #</p></th>
      <th><p>High-Level Functionality (Salesforce)</p></th>
      <th><p>Supporting Documentation</p></th>
      <th><p>Current Site Feature</p></th>
      <th><p>Decision</p></th>
      <th><p>Notes</p></th>
    </tr>
  </thead>
  <tbody>
    <!-- rows here -->
  </tbody>
</table>
```

### Section Headers

```xml
<h2>Catalog &amp; Product Management</h2>
```

### Expand Macro (for long notes)

```xml
<ac:structured-macro ac:name="expand">
  <ac:parameter ac:name="title">Click to view details</ac:parameter>
  <ac:rich-text-body>
    <p>Detailed implementation notes here...</p>
  </ac:rich-text-body>
</ac:structured-macro>
```

---

## Confluence Wiki Markup Specification

For the `.confluence` fallback file, use Confluence wiki markup syntax:

### Status Macros
```
{status:title=OOTB|colour=Green}
{status:title=Custom Dev|colour=Yellow}
{status:title=Gap|colour=Red}
```

### Tables
```
|| Feature # || High-Level Functionality || Supporting Documentation || Current Site Feature || Decision || Notes ||
| CAT-001 | Product catalog | [SF Docs|https://...] | Client catalog | {status:title=OOTB|colour=Green} | Map brands to categories |
```

### Info Panel
```
{info}
*Client:* {company name}
*Project:* {project name}
{info}
```

### Headers
```
h2. Catalog & Product Management
```

---

## Process

### Step 1: Resolve Company Name and Locate Source

1. Read the **company name** from the argument provided when the skill is invoked
2. Derive the **file slug** (lowercase kebab-case)
3. Locate the gap analysis source document: search for `docs/gap-analysis-*{slug}*.md` or fall back to any `docs/gap-analysis-*.md` file
4. If no source document is found, report the error and list available docs

### Step 2: Gather Inputs

Read the gap analysis source document and extract:
- Client metadata (confirm it matches the company argument)
- All identified workflows and gaps
- Severity ratings (map to Decision status badges)
- Resolution options and recommendations
- Assumptions and open questions

### Step 3: Organize into Functional Areas

Group gaps into Confluence table sections. Standard sections for a B2B Commerce migration:

1. **Catalog & Product Management** (CAT-nnn)
2. **Search & Navigation** (NAV-nnn)
3. **Ordering & Checkout** (ORD-nnn)
4. **Pricing & Promotions** (PRC-nnn)
5. **Budget Management** (BUD-nnn)
6. **User & Account Management** (USR-nnn)
7. **Order Management & Fulfillment** (FUL-nnn)
8. **Reporting & Analytics** (RPT-nnn)
9. **Integration & External Systems** (INT-nnn)
10. **Admin & Content Management** (ADM-nnn)

Adjust sections based on the actual client's use cases — only include sections that have features to assess. This is client-centric, not exhaustive.

### Step 4: Map Decisions

Convert gap severity/assessment to Decision status:
- "Direct Match" or "Standard Feature" → `OOTB` (Green)
- "Minor Customization" or "Configuration" → `Config` (Blue)
- "Custom Development" → `Custom Dev` (Yellow)
- "No Equivalent" or "Architectural Redesign" → `Gap` (Red)
- "Partial" or "Hybrid" → `Partial` (Yellow)
- "AppExchange" or "Third-Party" → `3rd Party` (Blue)
- "Needs Further Analysis" → `TBD` (Grey)

### Step 5: Generate Both Formats

Produce both files using the company slug:
- `docs/gap-analysis-confluence-{company-slug}.html` — Confluence Storage Format
- `docs/gap-analysis-confluence-{company-slug}.confluence` — Wiki Markup fallback

### Step 6: Generate Summary Statistics

Count decisions across all sections and produce the summary table.

---

## Collaboration

When working as part of an agent team:
- Consume the gap analysis document from the **Gap Analysis Agent**
- Consume capability assessments from the **SFCC B2B Expert** (for Salesforce documentation links)
- Can be run standalone on any existing `docs/gap-analysis-*.md` file
- Output is the final client-facing deliverable

## Standalone Usage

This skill can be invoked directly on any gap analysis markdown file. Always provide the company name:

```
Follow the confluence-gap-analysis skill in skills/confluence-gap-analysis/SKILL.md.
Company: Boston Beer Company
Read docs/gap-analysis-sample-bbc-merchtank.md and produce Confluence-formatted output files.
```

```
Follow the confluence-gap-analysis skill in skills/confluence-gap-analysis/SKILL.md.
Company: Acme Corp
Read docs/gap-analysis-acme.md and produce Confluence-formatted output files.
```

## Agent Team Usage

When the team lead invokes this skill as part of the full pipeline:

```
"confluence-formatter" — Follow the confluence-gap-analysis skill in skills/confluence-gap-analysis/SKILL.md.
Company: {client name here}
Wait for the gap-analyst to complete, then read the gap analysis at docs/gap-analysis-*.md and produce Confluence-formatted outputs.
```
