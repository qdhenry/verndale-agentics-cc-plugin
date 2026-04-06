---
name: documentation-pipeline
description: Multi-agent Chrome-based documentation orchestrator. Dispatches parallel subagents to visually document backend dashboard pages with screenshots, READMEs, and feature CSVs.
argument-hint: <url-1> <url-2> ... [--output-dir <path>]
---

# Documentation Pipeline Orchestrator

You are an orchestrator agent that takes a list of backend dashboard URLs and dispatches parallel subagents to produce comprehensive visual documentation for each page.

Each subagent operates independently — there are no dependencies between URLs. Launch all subagents simultaneously for maximum throughput.

## Arguments

- `urls` (required): One or more URLs to document (space-separated or newline-separated)
- `--output-dir` (optional): Base output directory (default: `./output/documentation-batch-{YYYY-MM-DD}/`)

## Prerequisites

- Claude-in-Chrome extension must be active and connected
- User must be authenticated in the browser for any pages requiring login
- The documentation protocol prompt must be available at `Prompts/documentation-gathering/Backend-Dashboard-Visual-Documentation-Generator.meta.md`

## Pipeline

### Step 1: Validate

1. Parse and validate all URLs (must start with http:// or https://)
2. Call `mcp__claude-in-chrome__tabs_context_mcp` to verify browser connectivity
3. Read the documentation protocol prompt file
4. Report: "Dispatching N subagents for N URLs"

### Step 2: Create Output Structure

```
{output-dir}/
├── {url-1-slug}/
│   ├── README.md
│   ├── features.csv
│   └── screenshots/
├── {url-2-slug}/
│   └── ...
└── BATCH-SUMMARY.md
```

Derive slugs from URL paths (e.g., `/admin/products` → `admin-products`).

### Step 3: Dispatch Parallel Subagents

For each URL, launch a subagent with:
- **Type**: `general-purpose`
- **Background**: `true` (all run simultaneously)
- **Instructions**:
  1. Create a new browser tab
  2. Navigate to the assigned URL
  3. Follow the full Backend-Dashboard-Visual-Documentation-Generator protocol
  4. Save README.md, features.csv, and screenshots/ to the assigned output directory

### Step 4: Monitor and Collect

1. Wait for all subagents to complete
2. Verify each output directory contains:
   - README.md (non-empty)
   - features.csv (valid header row, at least 1 data row)
   - screenshots/ directory with at least one PNG
3. Track successes and failures

### Step 5: Generate Batch Summary

Create `BATCH-SUMMARY.md`:

```markdown
# Documentation Batch Summary

**Date:** {YYYY-MM-DD}
**Total URLs:** {N}
**Successful:** {X}
**Failed:** {Y}

## Results

| # | URL | Status | Screenshots | CSV Rows |
|---|-----|--------|-------------|----------|
| 1 | ... | ...    | ...         | ...      |

## Failed URLs (if any)

- {URL}: {reason}
```

## Error Handling

- If Chrome extension is not connected, stop and ask the user to activate it
- If a subagent fails or times out, log the failure and continue with remaining agents
- If a URL requires authentication, note "Authentication required — skipped"
- If fewer than half succeed, alert the user before generating the summary
- Do not retry failed agents automatically — report them for manual review
