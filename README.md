# Gmail Draft Batch Sender for n8n

This project runs a self-hosted n8n instance with Docker Compose and imports a workflow that sends existing Gmail drafts from one Gmail account in randomized batches.

The workflow sends only the drafts already in Gmail Drafts. It does not create new emails, does not edit email content, does not manually delete drafts, does not use Gemini, and sends drafts through the Gmail API `users.drafts.send` endpoint.

This is not Gmail native Schedule Send. Drafts remain in Gmail until this workflow sends them.

Test mode includes a subject prefix safety filter. When `FORCE_TEST_MODE = true`, the workflow fetches draft metadata and only sends drafts whose subject starts with `[N8N TEST]`. Production mode does not require that prefix unless you intentionally keep test mode enabled or customize the filter.

## How It Runs

The Manual Trigger starts or resumes a sending session. Click `Execute Workflow` once to start the first batch.

After that, the Schedule Trigger runs every 1 minute while the workflow is active. It checks workflow static data and only sends the next batch when `sendingEnabled = true` and the current time is at or after `nextBatchAllowedAt`.

The workflow sends at most one batch per execution. It no longer relies on a long-running `Wait Random Batch Delay` loop between batches. After each batch, it stores a new `nextBatchAllowedAt` value 5 to 7 minutes in the future, then the execution stops cleanly.

n8n and Docker must be running, and the workflow must be active, for the Schedule Trigger to continue batches automatically.

## Production Settings

- Daily limit: 300 drafts per Eastern calendar day
- Batch size: random 9 to 18 drafts
- Per-email delay: random 8 to 25 seconds
- Batch delay: random 5 to 7 minutes, controlled by `nextBatchAllowedAt`
- Schedule check: every 1 minute

The workflow stops automatic sending by setting `sendingEnabled = false` when the daily limit is reached, when no drafts remain, or when a Gmail/API error is logged.

Gmail limits still apply. This workflow does not attempt to bypass Gmail sending limits, and Gmail may throttle or block sending based on account type, reputation, recipient patterns, or other Google policy checks.

## Files

- `docker-compose.yml`: local n8n Docker service
- `.env.example`: example environment file with `N8N_ENCRYPTION_KEY`
- `workflows/gmail_draft_batch_sender.json`: importable n8n workflow
- `docs/setup.md`: local setup steps
- `docs/google_oauth_setup.md`: Google Cloud OAuth setup
- `docs/workflow_runbook.md`: daily operation and tuning
- `docs/testing_checklist.md`: safe test checklist

## Quick Start

1. Start Docker Desktop.
2. Copy `.env.example` to `.env`.
3. Set a stable, long `N8N_ENCRYPTION_KEY`.
4. Run:

```bash
docker compose up -d
```

5. Open `http://localhost:5678`.
6. Import or update `workflows/gmail_draft_batch_sender.json`.
7. Attach a Google OAuth2 API credential to:
   - `List Gmail Drafts`
   - `Get Draft Metadata`
   - `Send Draft`
8. Activate the workflow.
9. Click `Execute Workflow` once to start a sending session.

Read the docs in `docs/` before running against real drafts.
