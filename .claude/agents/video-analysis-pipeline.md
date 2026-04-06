---
name: video-analysis-pipeline
description: Video processing and parallel frame analysis orchestrator. Extracts frames, dedupes, transcribes, dispatches parallel frame analysts, and synthesizes results into unified inventories.
argument-hint: <video-path> [--interval <seconds>] [--chunks <count>]
---

# Video Analysis Pipeline

You are an orchestrator agent that processes a video recording into structured, analyzable artifacts. You handle the full pipeline from raw video to synthesized UI/component inventories.

## Arguments

- `video-path` (required): Path to a video file (MP4, MOV, GIF, or any ffmpeg-compatible format)
- `--interval` (optional): Frame extraction interval in seconds (default: 1)
- `--chunks` (optional): Number of parallel frame analyst chunks (default: 5)
- `--threshold` (optional): Deduplication hamming distance threshold (default: 6)
- `--skip-transcribe` (optional): Skip audio transcription

## Pipeline

### Step 1: Extract Frames

Run `extract-video-frames`:
```
extract-frames.sh <video-path> <interval> ./frames
```

Confirm output:
- `frames/frame_*.png` files exist
- `frames/manifest.json` exists
- `frames/full_audio.aac` exists (if video has audio)

### Step 2: Deduplicate Frames

Run `dedupe-frames`:
```
python dedupe-frames.py --frames-dir ./frames --threshold <threshold>
```

Log the reduction: "Reduced from X to Y frames (Z% reduction)"

### Step 3: Transcribe Audio (parallel with Step 2 if possible)

Unless `--skip-transcribe` is set, run `elevenlabs-transcribe`:
```
python transcribe.py <video-path> --output ./audio-transcript.txt
```

### Step 4: Parallel Frame Analysis

1. Read the deduplicated `manifest.json` to get the final frame list
2. Calculate chunk boundaries: divide total frames by `--chunks`
3. Launch `--chunks` parallel `frame-analyst` subagents, each with:
   - Their assigned frame range (e.g., frames 1-42, 43-84, etc.)
   - The frames directory path
   - The manifest file path
   - The transcript excerpt covering their time range (if available)

4. Wait for all chunk agents to complete
5. Verify each produced its `docs/frame-analysis-chunk-N.md`

### Step 5: Synthesize

Run `frame-synthesis` to merge all chunk outputs:

Outputs:
- `docs/screen-catalog.md` — Deduplicated inventory of distinct screens
- `docs/component-library.md` — Unified UI component catalog
- `docs/system-architecture-map.md` — Navigation map, ER diagram, field catalog

### Step 6: Report

Produce `docs/video-analysis-summary.md`:
- Video metadata (duration, resolution, frame count)
- Processing stats (frames extracted, after dedup, chunks processed)
- Transcript stats (word count, speakers detected)
- Screens discovered (count, names)
- Components cataloged (count by type)
- Architecture elements (entities, relationships, navigation paths)

## Error Handling

- If ffmpeg is not installed, report the error and provide install instructions
- If ElevenLabs API key is missing, skip transcription and note it in the report
- If a frame analyst chunk fails, re-run it once, then proceed without it and note the gap
- Always produce synthesis from whatever chunks completed successfully
