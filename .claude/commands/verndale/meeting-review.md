---
name: "meeting-review"
description: "Analyze a gap analysis review meeting against the source document. Extracts agreements, changes, revisions needed, and verbal context not captured in notes."
argument-hint: '<meeting-dir>' '<company-name>' '<meeting-label>' [--skip-transcribe] [--force]
allowed-tools: Agent, Bash, Read, Write, Edit, Glob, Grep, TaskCreate, TaskUpdate, TaskList
---

<objective>
You are the Meeting Review Orchestrator. You analyze a client gap analysis review meeting — where an existing gap analysis document was walked through with the client — to produce a structured review showing:

1. **What was agreed to** — explicit client decisions captured verbally and in notes
2. **What can change** — items where client feedback modifies the recommended approach
3. **What needs to be revised** — contradictions, open questions, missing decisions, blocking items

This is NOT the migration-pipeline (which creates gap analyses from scratch). This command takes an EXISTING gap analysis and a REVIEW MEETING about that document, then cross-references them.
</objective>

<arguments>
Parse from `$ARGUMENTS`:

```
'<meeting-dir>' '<company-name>' '<meeting-label>' [--skip-transcribe] [--force]
```

- `meeting-dir` (required): Path to directory containing meeting artifacts (video, transcript, gap analysis PDF/document)
- `company-name` (required): Client/company name (e.g., `'Boston Beer Company'`)
- `meeting-label` (required): Short label for the meeting (e.g., `'batch-1-review'`)
- `--skip-transcribe`: Skip audio transcription (use if transcript already exists or no video)
- `--force`: Re-run all phases even if outputs exist

If `$ARGUMENTS` is empty or missing required arguments, print this usage guide and stop:

```
Usage: /verndale:meeting-review '<meeting-dir>' '<company-name>' '<meeting-label>' [--skip-transcribe] [--force]

Arguments:
  meeting-dir      Path to directory with meeting artifacts (video, PDF, transcript)
  company-name     Client/company name (quoted if spaces)
  meeting-label    Short meeting identifier (e.g., 'batch-1-review')

Flags:
  --skip-transcribe  Skip audio transcription (use existing transcript or skip)
  --force            Re-run all phases, ignoring existing outputs

Example:
  /verndale:meeting-review 'BostonBeerCompany/meetings/batch-2-meeting' 'Boston Beer Company' 'batch-2-review'
```

</arguments>

<derived_variables>
Compute from the arguments and use consistently:

```
COMPANY_SLUG    = lowercase kebab-case of company-name (e.g., "boston-beer-company")
CLIENT_DIR      = top-level client directory (e.g., "BostonBeerCompany")
MEETING_DIR     = the meeting-dir path provided
FRAMES_DIR      = {MEETING_DIR}/frames
DOCS_DIR        = {CLIENT_DIR}/docs
MEETING         = meeting-label value
```

</derived_variables>

<process>

## Phase 0: Pre-Flight and Discovery

### Step 1: Scan the meeting directory

List all files in `{MEETING_DIR}` and categorize them:

- **Video files**: `.mp4`, `.mov`, `.webm`, `.mkv`
- **PDF files**: `.pdf` (gap analysis documents)
- **Transcript files**: `.txt`, `.vtt`, `.srt`, `.docx` (existing transcripts)
- **Chat files**: `*Chat*.txt`, `*chat*.txt` (Zoom/Teams chat logs)

Report what was found:

```
=== Meeting Review: {company-name} — {meeting-label} ===
Directory:     {MEETING_DIR}
Video:         {filename or "Not found"}
Gap Analysis:  {filename or "Not found"}
Transcript:    {filename or "Not found — will transcribe from video"}
Chat Log:      {filename or "Not found"}
Output:        {DOCS_DIR}/
```

If NO gap analysis PDF/document is found, abort with: "No gap analysis document found in {MEETING_DIR}. This command requires an existing gap analysis to review against. Use /verndale:migration-pipeline to create one from scratch."

If NO video AND no transcript found, abort with: "No video or transcript found in {MEETING_DIR}. At minimum, a transcript is required to analyze the meeting discussion."

### Step 2: Check prerequisites

- `ffmpeg`: Required if video needs audio extraction. Check with `which ffmpeg`.
- `whisper`: Required if transcription is needed. Check with `which whisper`.
- `pdftotext` / `poppler`: Required for PDF parsing. Check with `which pdftotext`. If missing, install: `brew install poppler`.

Create output directories: `{FRAMES_DIR}` and `{DOCS_DIR}`.

### Step 3: Smart resume check (unless --force)

Check for existing outputs:

- `{DOCS_DIR}/{meeting-label}-review-summary.md` — primary deliverable
- `{DOCS_DIR}/{meeting-label}-agreements-analysis.md`
- `{DOCS_DIR}/{meeting-label}-revisions-needed.md`
- `{DOCS_DIR}/{meeting-label}-transcript-findings.md`

If ALL exist and `--force` is not set, report: "All review outputs already exist. Use --force to re-run." and exit.

## Phase 1: Preprocessing (parallel where possible)

### Step 1: Extract gap analysis text

If the gap analysis is a PDF, use `pdftotext` to extract the full text:

```bash
pdftotext "{gap-analysis-pdf}" "{MEETING_DIR}/gap-analysis-text.txt"
```

If the gap analysis is already markdown or text, copy/reference it directly.

Read the extracted text to understand the document structure. Note:

- How many features/items are covered
- What columns exist (Capability, Functionality, Observations, Next Steps, Decision, Notes)
- Which sessions or groupings are present
- What decision key is used

### Step 2: Transcribe meeting audio (unless --skip-transcribe or transcript exists)

If a transcript already exists in the directory, use it directly.

If only a video exists:

1. Extract audio: `ffmpeg -i "{video}" -vn -acodec aac -b:a 128k "{FRAMES_DIR}/full_audio.aac" -y`
2. Transcribe with Whisper (prefer `small` model for speed on CPU):
   ```bash
   whisper "{FRAMES_DIR}/full_audio.aac" --model small --language en --output_dir "{FRAMES_DIR}/" --output_format txt
   ```
   Run this in the background. If ElevenLabs API key is available, prefer that instead:
   ```bash
   uv run ~/.claude/skills/elevenlabs-transcribe/scripts/transcribe.py "{FRAMES_DIR}/full_audio.aac" --output "{FRAMES_DIR}/audio-transcript.txt"
   ```

### Step 3: Extract video frames (parallel with transcription)

If a video exists, extract frames at 10-second intervals for visual context:

```bash
ffmpeg -i "{video}" -vf "fps=1/10" -q:v 2 "{FRAMES_DIR}/frame_%04d.png" -y
```

This runs in parallel with transcription. Frames help identify which sections of the gap analysis were being discussed at each point in the meeting.

### Step 4: Read chat transcript

If a Zoom/Teams chat file exists, read it for any notes, reactions, or time markers captured during the meeting.

Report Phase 1 results:

```
Phase 1 complete:
  Gap analysis: {N} lines, {N} features identified
  Transcript:   {N} words ({status: ready/processing/skipped})
  Frames:       {N} frames extracted
  Chat:         {N} messages
```

## Phase 2: Parallel Analysis (launch 2 agents simultaneously)

Wait for the transcript to be available before launching these agents. If it's still processing, poll until complete.

### Agent 1: Agreements Analyst

Launch as a background Agent:

```
Agent: "Analyze gap analysis agreements"
Prompt: |
  You are analyzing a gap analysis review document for {company-name}.

  Read the full gap analysis text at: {MEETING_DIR}/gap-analysis-text.txt
  (or the original document path if not PDF)

  This document has columns including Capability, High-Level Functionality, Current Site Observations,
  Recommended Next Steps, Decision, and Notes. The Notes column contains client input from the review meeting.

  For EACH numbered item, extract and report:
  1. The item number and name
  2. The Decision status (which badge(s) appear — CONFIGURATION, CUSTOM, NEW BUILD, DISABLE, or N/A)
  3. What the client AGREED TO — specific decisions captured in the Notes column
  4. Any CRITICAL REQUIREMENTS mentioned
  5. Any ACTION ITEMS or TO-DOs assigned to specific parties
  6. Any ASSUMPTIONS made that need validation

  Output as structured markdown. Reconstruct complete thoughts from fragmented PDF text.
  Include a Summary of Key Agreements table at the end.
  Include Consolidated Action Items by party (client vs. Verndale).
  Include an Assumptions Register.
  List any items with NO client notes that need follow-up.

  Save to: {DOCS_DIR}/{meeting-label}-agreements-analysis.md
```

### Agent 2: Revisions Analyst

Launch as a background Agent simultaneously:

```
Agent: "Analyze revisions needed"
Prompt: |
  You are reviewing a gap analysis review document for {company-name}.

  Read the full gap analysis text at: {MEETING_DIR}/gap-analysis-text.txt
  (or the original document path if not PDF)

  Identify and categorize items that need REVISION based on client feedback:

  1. Items where client input CONTRADICTS the Recommended Next Steps
  2. Items where the Decision is unclear or has multiple conflicting badges
  3. Items where "Assumption:" appears followed by something needing validation
  4. Items where "TO DO:" appears — explicit action items
  5. Items where the client must provide documentation — pending deliverables
  6. Items where "Critical:" appears — critical requirements affecting scope
  7. Items where "Future phase" or deferred scope is mentioned
  8. Open questions — anything implying uncertainty

  For each item: what the document says, what client feedback indicates, what revision is needed, who owns it.

  Include a Priority Actions table at the end with P1 (blocks design), P2 (should resolve before design), P3 (can update immediately).
  Include Document-Level Observations about systemic issues.

  Save to: {DOCS_DIR}/{meeting-label}-revisions-needed.md
```

Wait for both agents to complete.

## Phase 3: Transcript Cross-Reference

Once the transcript AND both Phase 2 agents are complete, launch:

```
Agent: "Cross-reference transcript with analysis"
Prompt: |
  You are analyzing the meeting transcript from a gap analysis review for {company-name}.

  Transcript: {transcript-path}
  Agreements analysis: {DOCS_DIR}/{meeting-label}-agreements-analysis.md
  Revisions analysis: {DOCS_DIR}/{meeting-label}-revisions-needed.md

  Read the FULL transcript, then read both analysis documents. Find verbal agreements, decisions,
  context, and nuances NOT already captured in the existing analysis documents.

  Look for:
  1. Verbal decisions not typed into Confluence Notes
  2. Important context — WHY something is the way it is
  3. Verbal commitments — things someone agreed to do
  4. Disagreements or hesitations
  5. Clarifications of scope
  6. People mentioned and their roles
  7. Items explicitly deferred — "we'll come back to that"
  8. Whether the meeting ran out of time (which items were NOT discussed)

  For each finding: transcript location, who said it, what was said, how it relates to gap analysis items,
  whether it changes anything in the existing analysis.

  Save to: {DOCS_DIR}/{meeting-label}-transcript-findings.md
```

Wait for completion.

## Phase 4: Synthesis — Primary Deliverable

Read all three analysis documents:

- `{DOCS_DIR}/{meeting-label}-agreements-analysis.md`
- `{DOCS_DIR}/{meeting-label}-revisions-needed.md`
- `{DOCS_DIR}/{meeting-label}-transcript-findings.md`

Also sample key video frames (if extracted) to understand the meeting flow — which sections were shown on screen and when.

Synthesize into a single executive review document at `{DOCS_DIR}/{meeting-label}-review-summary.md` with these sections:

### Document Structure

```markdown
# {meeting-label} Gap Analysis Review — Meeting Summary & Action Plan

**Client:** {company-name}
**Meeting Date:** {date from video/transcript}
**Document Reviewed:** {gap analysis filename}
**Prepared:** {today's date}

---

## Executive Summary

- How many items were covered
- Whether the meeting ran out of time (and which items were NOT discussed)
- High-level breakdown: agreed / modified / needs resolution
- Critical blockers

## Section 1: What Was Agreed To

Items with clear, explicit client decisions. For each:

- The decision
- Impact on scope
- Action required

## Section 2: What Can Change (Modifications to Recommended Approach)

Items the client generally accepted but with specific modifications. For each:

- Current recommendation
- Client modification
- What changes in the approach

## Section 3: What Needs To Be Revised / Resolved

Blockers, contradictions, missing decisions, items not discussed. Including:

- Blocking issues (must resolve before design)
- Decision badges that need resolution
- Items NOT discussed (if meeting ran out of time)

## Section 4: Prioritized Action Plan

P1, P2, P3 action tables with owner and what each blocks.

## Section 5: Assumptions Register

All assumptions with risk-if-wrong assessment.

## Section 6: Critical Requirements Summary

All items tagged as "Critical" by the client.

## Section 7: Key Verbal Context from Transcript

Important details spoken but NOT typed into the gap analysis Notes column.

## Recommended Next Steps

Actionable next steps prioritized by urgency.

## Supporting Documents

Table linking to all generated files.
```

## Phase 5: Summary Report

Print a final summary:

```
===============================================
  Meeting Review Complete: {company-name}
  Meeting: {meeting-label}
===============================================

Phase 1 — Preprocessing
  {status} Gap analysis parsed         ({N} features)
  {status} Audio transcribed           ({N} words)
  {status} Frames extracted            ({N} frames)

Phase 2 — Analysis
  {status} Agreements analysis         ({N} decisions, {N} action items)
  {status} Revisions analysis          ({N} items needing revision)

Phase 3 — Transcript Cross-Reference
  {status} Transcript findings         ({N} new findings)

Phase 4 — Synthesis
  {status} Review summary              ({DOCS_DIR}/{meeting-label}-review-summary.md)

Total deliverables: 4
Primary: {DOCS_DIR}/{meeting-label}-review-summary.md
```

</process>

<error_handling>

- If PDF parsing fails (pdftotext not available), try `brew install poppler` and retry. If still fails, ask user to provide the gap analysis as text/markdown.
- If Whisper takes too long (>15 min wall clock for small model), check if it's running on CPU-only. Consider killing and retrying with `base` model.
- If a subagent fails, log the error and continue with remaining pipeline steps. Produce whatever partial deliverables are possible.
- Phase 3 (transcript cross-reference) depends on Phase 2 AND the transcript both being complete.
- Always produce the Phase 4 synthesis even if Phase 3 fails — the agreements and revisions analyses are sufficient.
  </error_handling>

<success_criteria>

- Gap analysis document fully parsed and understood
- Meeting transcript captured (via Whisper or existing file)
- All 4 deliverables produced in {DOCS_DIR}/
- Primary deliverable ({meeting-label}-review-summary.md) contains:
  - Executive summary with coverage stats
  - Explicit agreements with scope impact
  - Modifications to the recommended approach
  - Blocking issues and items needing resolution
  - Prioritized action plan with owners
  - Verbal context not in the written notes
- Items NOT discussed in the meeting are clearly identified
- All action items have assigned owners (client vs. Verndale)
  </success_criteria>
