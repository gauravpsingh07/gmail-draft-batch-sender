# Gmail Draft Batch Sender for n8n

This project runs a self-hosted n8n instance with Docker Compose and imports a manual workflow that sends existing Gmail drafts from one Gmail account in randomized batches.

The workflow sends only the drafts already in Gmail Drafts. It does not create new emails, does not edit email content, does not manually delete drafts, does not use Gemini, and sends drafts through the Gmail API `users.drafts.send` endpoint.

This is not Gmail native Schedule Send. Drafts remain in Gmail until you manually run this workflow and the workflow sends them.

Test mode includes a subject prefix safety filter. When `FORCE_TEST_MODE = true`, the workflow fetches draft metadata and only sends drafts whose subject starts with `[N8N TEST]`. Production mode does not require that prefix unless you intentionally keep test mode enabled or customize the filter.

## Manual Operation

There is no weekday schedule and no 10:00 AM to 4:00 PM sending window. Docker and n8n must be running, then you start a sending session by opening the workflow in n8n and clicking `Execute Workflow`.

The workflow keeps looping during that same manual execution until one of these happens:

- 300 drafts have been sent for the current Eastern calendar day
- No Gmail drafts remain
- You manually stop the workflow execution
- Gmail or the Gmail API returns an error

## Production Settings

- Trigger: manual execution only
- Daily limit: 300 drafts per Eastern calendar day
- Batch size: random 9 to 18 drafts
- Per-email delay: random 8 to 25 seconds
- Batch delay: random 5 to 7 minutes

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
7. Create and attach a Google OAuth2 API credential to:
   - `List Gmail Drafts`
   - `Get Draft Metadata`
   - `Send Draft`
8. Test with harmless drafts.
9. For actual sending, open the workflow and click `Execute Workflow` manually whenever you want to start a sending session.

Read the docs in `docs/` before running against real drafts.
