# verndale-agentics

A Claude Code plugin for Salesforce B2B Commerce migration analysis — end-to-end gap analysis pipelines, meeting review, batch enrichment, video analysis, and documentation generation.

## Installation

```bash
# Add the marketplace
claude plugin install verndale-agentics@<your-github-url>

# Or test locally during development
claude --plugin-dir ./verndale-agentics-plugin
```

## Commands

### Pipeline Commands

| Command | Description |
|---------|-------------|
| `/verndale:migration-pipeline` | Run the end-to-end Salesforce migration analysis pipeline — takes a client video recording and produces gap analysis, validation report, client elicitation, and Confluence export |
| `/verndale:meeting-review` | Analyze a gap analysis review meeting against the source document — extracts agreements, changes, revisions needed, and verbal context |
| `/verndale:enrich-batch` | Enrich a batch gap analysis document with Salesforce references, client evidence, assigned decisions, and Confluence export |

### Usage Examples

```bash
# Full migration analysis from a video recording
/verndale:migration-pipeline 'client/screencasts/walkthrough.mp4' 'Acme Corp' 'platform-overview' 5

# Review a gap analysis meeting
/verndale:meeting-review 'client/meetings/batch-1' 'Acme Corp' 'batch-1-review'

# Enrich a batch document
/verndale:enrich-batch 3 'My Account' 'Post-purchase experience' 'client/docs/batch-3.doc'
```

## Skills

### Migration Analysis Skills

| Skill | Description |
|-------|-------------|
| **gap-analysis** | Produce client-specific gap analysis comparing current platform against Salesforce B2B Commerce |
| **sfcc-b2b-expert** | Map client requirements to Salesforce B2B Commerce capabilities with implementation guidance |
| **sfcc-validator** | Fact-check Salesforce capability claims against current documentation, Trailhead, and AppExchange |
| **ui-migration** | Map current UI components to Salesforce Experience Cloud / LWC equivalents |
| **data-schema** | Design Salesforce object schema from the current platform's data model |
| **integration-api** | Architect the Salesforce integration layer for ERP, fulfillment, and vendor connections |

### Meeting & Transcript Analysis

| Skill | Description |
|-------|-------------|
| **meeting-analyst** | Extract functional requirements, pain points, and migration hints from meeting recordings |
| **transcript-analyst** | Analyze raw text transcripts to extract requirements, business rules, and integration hints |
| **frame-analyst** | Analyze video frame chunks to identify UI components, data structures, and navigation elements |
| **frame-synthesis** | Merge parallel frame analysis outputs into unified screen catalog and component library |

### Client Deliverables

| Skill | Description |
|-------|-------------|
| **client-elicitor** | Consolidate open questions and assumptions into prioritized client Q&A and meeting prep guide |
| **confluence-gap-analysis** | Generate gap analysis in Confluence Storage Format (XHTML) for direct import |

### Utility Skills

Shared across the video and migration pipelines:

| Skill | Description |
|-------|-------------|
| **extract-video-frames** | Extract frames and audio segments from video files using FFmpeg |
| **dedupe-frames** | Remove near-duplicate frames using perceptual hashing |
| **elevenlabs-transcribe** | Transcribe audio/video using ElevenLabs Scribe v2 API |

## Agents

| Agent | Description |
|-------|-------------|
| `migration-pipeline` | End-to-end migration analysis orchestrator — coordinates all phases from video to deliverables |
| `video-analysis-pipeline` | Video processing orchestrator — frame extraction, dedup, transcription, parallel analysis, synthesis |
| `documentation-pipeline` | Chrome-based documentation orchestrator — parallel subagents for visual page documentation |

## Prerequisites

- **FFmpeg**: Required for frame extraction (`brew install ffmpeg`)
- **Python 3.10+**: Required for frame deduplication (`imagehash` and `Pillow` packages)
- **uv**: Required for ElevenLabs transcription (auto-installs dependencies via PEP 723)
- **poppler**: Required for PDF parsing in meeting review (`brew install poppler`)
- **whisper**: Optional for local audio transcription (`brew install openai-whisper`)
- **ElevenLabs API key**: Set `ELEVENLABS_API_KEY` environment variable for transcription
- **Claude-in-Chrome**: Required for documentation pipeline (browser extension must be active)

## Pipeline Architecture

```
Platform Walkthrough Video
    └─► /verndale:migration-pipeline
         ├─ Phase 0: Extract frames → Dedup → Transcribe
         ├─ Phase 1: 5x parallel frame analysts → Synthesis
         ├─ Phase 2: Meeting analyst, UI migration, Integration, Data schema, SFCC expert → Gap analysis
         └─ Phase 3: SFCC validation, Client elicitation, Confluence export

Gap Analysis Document
    ├─► /verndale:meeting-review  (after client review session)
    └─► /verndale:enrich-batch    (for batch-level Confluence docs)
```

## License

MIT
