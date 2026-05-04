# Setup

## 1. Start Docker Desktop

Start Docker Desktop and wait until Docker is running.

## 2. Create `.env`

If you have not already created `.env`, copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

On PowerShell:

```powershell
Copy-Item .env.example .env
```

## 3. Set `N8N_ENCRYPTION_KEY`

Open `.env` and replace the placeholder with a long random value. Keep this value stable after creating n8n credentials. Changing it later can make saved credentials unreadable.

## 4. Start n8n

Run:

```bash
docker compose up -d
```

## 5. Open n8n

Open:

```text
http://localhost:5678
```

Create the local n8n owner account if this is the first run.

## 6. Import or Update the Workflow

In n8n:

1. Create or open the workflow.
2. Import from file.
3. Select `workflows/gmail_draft_batch_sender.json`.
4. Save the workflow.

If you previously imported the scheduled version, replace it with this updated manual-only workflow.

## 7. Create or Verify Google OAuth Credential

Follow `docs/google_oauth_setup.md`.

The required Gmail scope is:

```text
https://www.googleapis.com/auth/gmail.modify
```

## 8. Attach Credential to Gmail HTTP Nodes

Attach the Google OAuth2 API credential to all Gmail HTTP Request nodes:

- `List Gmail Drafts`
- `Get Draft Metadata`
- `Send Draft`

All three nodes must use the same Gmail account that owns the drafts.

The `Send Draft` node must use Gmail API `users.drafts.send` like this:

```text
POST https://gmail.googleapis.com/gmail/v1/users/me/drafts/send
```

JSON body:

```json
{
  "id": "{{ $json.draftId }}"
}
```

## 9. Test with Harmless Drafts

Create 2 to 5 harmless drafts addressed to an alternate email you control. Follow `docs/testing_checklist.md`.

## 10. Start a Sending Session

For actual sending:

1. Start Docker Desktop.
2. Run:

```bash
docker compose up -d
```

3. Open `http://localhost:5678`.
4. Open the Gmail Draft Batch Sender workflow.
5. Click `Execute Workflow` manually whenever you want to start the sending session.

There is no automatic schedule. The workflow sends only when you manually execute it.
