# Verndale Slash Commands

Slash commands for Verndale's Salesforce B2B Commerce migration practice.

## Commands

### `/verndale:migration-pipeline`

End-to-end migration analysis pipeline. Takes a client video recording of a **platform walkthrough** and produces a complete gap analysis from scratch.

```
/verndale:migration-pipeline '<video-path>' '<company-name>' '<meeting-label>' [interval] [--skip-dedup] [--skip-transcribe] [--force]
```

**When to use:** You have a raw video of someone walking through the client's current platform and need to produce a gap analysis.

**Arguments:**

| Argument | Required | Description |
|----------|----------|-------------|
| `video-path` | Yes | Path to client meeting video (MP4, MOV) |
| `company-name` | Yes | Client name, quoted if spaces (e.g., `'Boston Beer Company'`) |
| `meeting-label` | Yes | Short meeting ID (e.g., `'vw-walkthrough'`) |
| `interval` | No | Frame extraction interval in seconds (default: 5) |
| `--skip-dedup` | No | Skip perceptual frame deduplication |
| `--skip-transcribe` | No | Skip ElevenLabs audio transcription |
| `--force` | No | Re-run all phases, ignoring existing outputs |

**Example:**

```
/verndale:migration-pipeline 'BostonBeerCompany/screencasts/Virtual Warehouse Walk through.mp4' 'Boston Beer Company' 'vw-walkthrough' 5
```

**Pipeline phases:**

1. **Phase 0 — Preprocessing**: Frame extraction, deduplication, audio transcription
2. **Phase 1 — Frame Analysis**: 5 parallel frame analysts + synthesis merge
3. **Phase 2 — Migration Analysis**: Meeting analyst, UI migration, integration, data schema, SFCC expert, gap analysis
4. **Phase 2.5+3 — Delivery**: SFCC validation, client elicitation, Confluence export

**Output** (20+ files in `{client-dir}/docs/`):

| File | Description |
|------|-------------|
| `gap-analysis-{slug}-{meeting}.md` | Primary deliverable — workflow-based gap analysis |
| `feature-inventory-{slug}-{meeting}.md` | Extracted requirements and business rules |
| `sfcc-assessment-{meeting}.md` | Salesforce capability mapping |
| `ui-migration-map-{meeting}.md` | UI component migration plan |
| `integration-assessment-{meeting}.md` | Integration architecture |
| `data-schema-mapping-{meeting}.md` | Salesforce object schema design |
| `sfcc-validation-{slug}.md` | Fact-checked validation report |
| `client-elicitation-{slug}.md` | Prioritized client Q&A |
| `meeting-prep-{slug}-validation.md` | Meeting agenda and talking points |
| `gap-analysis-confluence-{slug}.html` | Confluence Storage Format export |
| `gap-analysis-confluence-{slug}.confluence` | Wiki Markup fallback |

---

### `/verndale:meeting-review`

Analyze a gap analysis **review meeting** against the source document. Cross-references what was discussed verbally against the written gap analysis to extract agreements, changes, and revisions needed.

```
/verndale:meeting-review '<meeting-dir>' '<company-name>' '<meeting-label>' [--skip-transcribe] [--force]
```

**When to use:** You have a gap analysis document AND a recording/transcript of the client review meeting where that document was discussed. You need to understand what was agreed, what changed, and what needs revision.

**Arguments:**

| Argument | Required | Description |
|----------|----------|-------------|
| `meeting-dir` | Yes | Path to directory with meeting artifacts (video, PDF, transcript) |
| `company-name` | Yes | Client name, quoted if spaces |
| `meeting-label` | Yes | Short meeting ID (e.g., `'batch-1-review'`) |
| `--skip-transcribe` | No | Skip audio transcription (use existing transcript) |
| `--force` | No | Re-run all phases, ignoring existing outputs |

**Example:**

```
/verndale:meeting-review 'BostonBeerCompany/meetings/batch-2-meeting' 'Boston Beer Company' 'batch-2-review'
```

**Meeting directory should contain:**
- A video file (`.mp4`, `.mov`) — the meeting recording
- A gap analysis document (`.pdf`) — the document that was reviewed
- Optionally: an existing transcript (`.txt`, `.vtt`, `.srt`)
- Optionally: a chat log (`*Chat*.txt`)

**Pipeline phases:**

1. **Phase 0 — Preprocessing**: Parse gap analysis PDF, transcribe audio (Whisper), extract frames
2. **Phase 1 — Parallel Analysis**: Agreements analyst + Revisions analyst (2 agents)
3. **Phase 2 — Transcript Cross-Reference**: Finds verbal context not in written notes
4. **Phase 3 — Synthesis**: Executive review summary with action plan

**Output** (4 files in `{client-dir}/docs/`):

| File | Description |
|------|-------------|
| `{meeting-label}-review-summary.md` | Primary deliverable — executive summary with agreements, changes, blockers, action plan |
| `{meeting-label}-agreements-analysis.md` | Per-item extraction of decisions, action items, and assumptions |
| `{meeting-label}-revisions-needed.md` | Items needing revision with priority matrix |
| `{meeting-label}-transcript-findings.md` | Verbal context not captured in the written notes |

---

### `/verndale:enrich-batch`

Enrich an existing batch gap analysis document with Salesforce references, BBC-specific evidence, assigned decisions, and Confluence export. Reads the source document, cross-references meeting analyses and critical requirements, identifies coverage gaps, and produces both Markdown and Confluence HTML deliverables matching the quality of completed batches.

```
/verndale:enrich-batch <batch-number> '<batch-title>' '<focus-description>' '<doc-path>' [items-list]
```

**When to use:** You have a batch gap analysis document (from Confluence or another source) that needs enrichment — adding Salesforce capability descriptions, BBC MerchTank evidence, documentation links, recommended decisions, and actionable next steps.

**Arguments:**

| Argument | Required | Description |
|----------|----------|-------------|
| `batch-number` | Yes | Batch number (e.g., `3`, `4`) |
| `batch-title` | Yes | Batch title, quoted (e.g., `'My Account'`) |
| `focus-description` | Yes | What this batch covers (e.g., `'Post-purchase and account management'`) |
| `doc-path` | Yes | Path to source document (`.doc`, `.pdf`, `.html`, `.md`) |
| `items-list` | No | Specific items/topics to cover (freeform text) |

**Example:**

```
/verndale:enrich-batch 3 'My Account' 'Post-purchase and account management experience' 'BostonBeerCompany/Batch-docs/Gap+Analysis+_+Batch+3+_+My+Account.doc' 'Account Structure, Order Management, Integration Touchpoints'
```

**Pipeline steps:**

1. **Format scan** — Read Batches 1 & 2 for format reference
2. **Extract** — Parse source document, extract features and current content
3. **Cross-reference** — Match against Critical Requirements and meeting analyses
4. **Gap analysis** — Identify missing features vs requirements
5. **Clarify** — Ask user about format, decisions, scope overlap
6. **Enrich** — Populate all 6 columns for every feature
7. **Organize** — Group into logical sections
8. **Summarize** — Decision distribution, requirements coverage, assumptions, open questions
9. **Export** — Write Markdown + Confluence HTML
10. **Verify** — Cross-check coverage against all requirement sources

**Output** (2 files):

| File | Description |
|------|-------------|
| `gap-analysis-batch-{N}-{slug}.md` | Enriched markdown — canonical source |
| `gap-analysis-batch-{N}-{slug}.html` | Confluence Storage Format — ready to import |

---

## Command Relationship

```
Platform Walkthrough Video ──► /verndale:migration-pipeline ──► Gap Analysis Document
                                                                       │
                                                                       ├──► /verndale:meeting-review ──► Review Summary & Action Plan
                                                                       │     (after client review meeting)
                                                                       │
                                                                       └──► /verndale:enrich-batch ──► Enriched Batch Deliverables
                                                                             (for batch-level Confluence docs)
```

- Use `migration-pipeline` first to CREATE a gap analysis from a meeting recording.
- Use `meeting-review` after each client review session to track agreements and changes.
- Use `enrich-batch` to take a batch gap analysis document and enrich it with Salesforce references, BBC evidence, decisions, and Confluence export.

---

## Prerequisites

- `ffmpeg` installed (`brew install ffmpeg`)
- `poppler` installed (`brew install poppler`) — for PDF parsing
- `whisper` installed (`brew install openai-whisper`) — for local audio transcription
- Optionally: `ELEVENLABS_API_KEY` in `.env` for higher-quality transcription
- Optionally: `uv` installed for Python script dependencies
