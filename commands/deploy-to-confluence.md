---
name: "verndale:deploy-to-confluence"
description: "Deploy a Confluence Storage Format HTML file to a Confluence page, creating or updating it"
argument-hint: "<space-key> <page-id> [html-file-path]"
allowed-tools: Bash, Read, Glob, Grep
---

# Deploy to Confluence

Push a Confluence Storage Format HTML file to a Confluence Cloud page via the REST API.

## Arguments

Parse from `$ARGUMENTS`:

1. **`space-key`** (required) — Confluence space key (e.g., `TBBC`)
2. **`page-id`** (required) — Numeric page ID to update (from the page URL, e.g., `6818562078`)
3. **`html-file-path`** (optional) — Path to the `.html` file containing Confluence Storage Format XHTML. If omitted, search the current directory and `BostonBeerCompany/` for the most recently modified `*.html` file containing `ac:structured-macro` tags.

## Environment

Read credentials from the root `.env` file:

```
ATLASSIAN_API_TOKEN=...
ATLASSIAN_EMAIL=...
ATLASSIAN_SITE=...
```

Load them with:

```bash
set -a; source .env; set +a
```

All three variables are required. If any are missing, stop and tell the user.

## Execution Steps

### Step 1: Validate the HTML file

If no file path was provided, find the best candidate:

```bash
find . -name "*.html" -newer .env -type f | head -20
```

Then check each candidate for `ac:structured-macro` to confirm it's Confluence Storage Format (not plain HTML). Pick the most recently modified match. Confirm with the user before proceeding.

If a file path was provided, validate it:

- File exists
- Contains at least one `ac:structured-macro` tag (sanity check)
- Report file size

If validation fails, stop and explain.

### Step 2: GET current page version

```bash
curl -s "https://${ATLASSIAN_SITE#https://}/wiki/rest/api/content/${PAGE_ID}" \
  -u "${ATLASSIAN_EMAIL}:${ATLASSIAN_API_TOKEN}" \
  -H "Accept: application/json"
```

Extract:

- `version.number` — current version (will increment by 1)
- `title` — current page title (preserve it)
- `type` — should be `page`

If the page is not found (404), stop and report that the page ID is invalid.

### Step 3: Build payload

Use Python to construct the JSON payload — the HTML content must be JSON-escaped:

```python
import json

with open(HTML_FILE_PATH, "r") as f:
    html_content = f.read()

payload = {
    "version": {
        "number": CURRENT_VERSION + 1,
        "message": "Deployed via /verndale:deploy-to-confluence"
    },
    "title": CURRENT_TITLE,
    "type": "page",
    "body": {
        "storage": {
            "value": html_content,
            "representation": "storage"
        }
    }
}

with open("/tmp/confluence-deploy-payload.json", "w") as f:
    json.dump(payload, f)
```

### Step 4: PUT the update

```bash
curl -s -w "\nHTTP_CODE:%{http_code}" -X PUT \
  "https://${ATLASSIAN_SITE#https://}/wiki/rest/api/content/${PAGE_ID}" \
  -u "${ATLASSIAN_EMAIL}:${ATLASSIAN_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d @/tmp/confluence-deploy-payload.json
```

Parse the response:

- On success (200): report page title, new version number, and full URL
- On error: report the status code and error message

### Step 5: Clean up

```bash
rm /tmp/confluence-deploy-payload.json 2>/dev/null
```

## Output

On success, report:

```
Page updated successfully.
  Title:   {title}
  Version: {new_version}
  Space:   {space_key}
  URL:     https://{site}/wiki/spaces/{space_key}/pages/{page_id}/{url_encoded_title}
```

## Error Handling

| Error                                  | Action                                                                                      |
| -------------------------------------- | ------------------------------------------------------------------------------------------- |
| `.env` missing or incomplete           | Stop — tell user to create `.env` with ATLASSIAN_API_TOKEN, ATLASSIAN_EMAIL, ATLASSIAN_SITE |
| HTML file not found                    | Stop — report path and suggest alternatives                                                 |
| HTML file has no `ac:structured-macro` | Warn — file may not be Confluence Storage Format. Ask user to confirm before pushing        |
| Page ID returns 404                    | Stop — page does not exist. Suggest checking the URL                                        |
| API returns 401/403                    | Stop — credentials invalid or insufficient permissions                                      |
| API returns 409                        | Version conflict — re-fetch version and retry once                                          |
