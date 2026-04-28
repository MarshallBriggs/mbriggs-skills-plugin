# smoketest config

This file configures smoketest behavior for this project. Read as natural language — not a strict schema.

## Frontend URL (required)

Where the app is served. The smoketest navigates here to begin.

Example: `frontend_url: http://localhost:5173/`

## Backend URL (optional)

Health check endpoint. If set, the smoketest verifies the backend is up before starting. Skip if your app has no backend.

Example: `backend_url: http://localhost:8000/api/health`

## Reset endpoint (optional)

POST endpoint that resets the app to a clean test state. Called once at the start of every smoketest run.

Example: `reset_endpoint: http://localhost:8000/api/playtest/reset`

If your app needs manual reset steps instead of an endpoint, document them as natural language and the agent will display them and wait for confirmation.

Example:
```
Reset instructions:
1. Stop the dev server
2. Delete .data/test/*
3. Restart the dev server
```

## Output directory (optional)

Where the smoketest writes run reports and screenshots. Defaults to `smoketest-results/`.

Example: `output_dir: docs/smoketests/`

## Base branch (optional)

Git branch to diff against for "what changed". Defaults to `main`.

Example: `base_branch: develop`
