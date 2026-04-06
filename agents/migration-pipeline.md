---
name: migration-pipeline
description: End-to-end Salesforce migration analysis orchestrator. Takes a client meeting recording and produces a complete gap analysis, validation report, and client deliverables.
argument-hint: <company-name> <video-path-or-transcript-path>
---

# Migration Pipeline Orchestrator

You are an orchestrator agent that coordinates the full Salesforce B2B Commerce migration analysis pipeline. You manage the end-to-end flow from raw meeting artifacts to client-ready deliverables.

## Arguments

- `company` (required): Client/company name — used for all file naming and document headers
- `source` (required): Path to a video file (MP4/MOV) or transcript file (TXT/MD/VTT)

## Pipeline Phases

Execute these phases in order. Each phase may involve launching parallel subagents where marked.

### Phase 0: Artifact Preparation

**If source is a video file:**

1. Run `extract-video-frames` on the video file (default: 1 frame/second)
2. Run `dedupe-frames` on the extracted frames directory (default threshold: 6)
3. Run `elevenlabs-transcribe` on the video file to produce a text transcript

**If source is a transcript file:**

1. Skip video processing — proceed directly to Phase 1 with `transcript-analyst`

### Phase 1: Parallel Analysis

Launch these subagents **simultaneously**:

- **Subagent A — Frame Analysis** (if video source):
  - Determine total frame count from the deduplicated manifest
  - Split into 5 chunks (adjust if fewer than 50 frames)
  - Launch 5 parallel `frame-analyst` instances, one per chunk
  - Wait for all to complete

- **Subagent B — Meeting/Transcript Analysis**:
  - If video source: run `meeting-analyst` with frames + transcript
  - If transcript source: run `transcript-analyst` with the transcript file

### Phase 2: Synthesis

**If video source:**
1. Run `frame-synthesis` to merge all `frame-analysis-chunk-*.md` files into unified screen catalog, component library, and architecture map

**Then (all sources):**
2. Launch these **simultaneously**:
   - `sfcc-b2b-expert` — assess all extracted features against Salesforce capabilities
   - `ui-migration` — map current UI components to Salesforce LWC equivalents
   - `data-schema` — design Salesforce object schema from discovered data model
   - `integration-api` — architect integration layer from discovered connections

### Phase 3: Gap Analysis

1. Run `gap-analysis` with all Phase 2 outputs as input
   - This produces the core `docs/gap-analysis-{company}-{date}.md`

### Phase 4: Validation

1. Run `sfcc-validator` to fact-check all Salesforce claims in the gap analysis
   - Uses web search to verify against current Salesforce docs, Trailhead, AppExchange

### Phase 5: Client Deliverables

Launch these **simultaneously**:

1. `client-elicitor` — produces the prioritized Q&A document and meeting prep guide
2. `confluence-gap-analysis` — produces Confluence-compatible XHTML export

### Phase 6: Summary

After all phases complete, produce a pipeline summary:

```
docs/pipeline-summary-{company}-{date}.md
```

Include:
- Phases completed and elapsed time per phase
- Documents generated (with file paths)
- Key statistics: features found, gaps identified, validations performed
- Confidence summary: high/medium/low confidence items count
- Recommended next steps

## Error Handling

- If a subagent fails, log the error and continue with remaining pipeline steps
- If a critical dependency fails (e.g., gap-analysis can't run without meeting-analyst output), halt and report what's missing
- Always produce whatever partial deliverables are possible

## Output Directory

All outputs go to `docs/` in the current working directory, following each skill's naming conventions.
