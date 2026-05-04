# Workflow Runbook

## Behavior

- Trigger: manual execution only
- Daily limit: 300 drafts per Eastern calendar date
- Batch size: random 9 to 18 drafts
- Batch delay: random 5 to 7 minutes
- Per-email delay: random 8 to 25 seconds

The workflow loops in the same manual execution:

```text
Check daily limit -> list drafts -> fetch draft metadata -> apply test subject filter -> send random batch -> wait random batch delay -> check daily limit -> list next batch
```

It stops when 300 drafts have been sent for the Eastern date, when no drafts remain, when you manually stop the execution, or when Gmail/API returns an error.

Test mode includes a subject prefix safety filter. When `FORCE_TEST_MODE = true`, the workflow only sends drafts whose subject starts with `[N8N TEST]`. Production mode does not use that prefix filter unless you intentionally keep test mode enabled or customize the filter.

## Gmail Send Endpoint

The `Send Draft` node uses Gmail API `users.drafts.send`:

```text
POST https://gmail.googleapis.com/gmail/v1/users/me/drafts/send
```

JSON body:

```json
{
  "id": "{{ $json.draftId }}"
}
```

The draft ID belongs in the JSON body, not in the URL path.

## Running Manually

1. Start Docker Desktop.
2. Run:

```bash
docker compose up -d
```

3. Open `http://localhost:5678`.
4. Open the workflow.
5. Click `Execute Workflow`.

No workflow activation is required for schedule-based sending, because schedule-based sending has been removed.

## Static Data

The workflow stores these keys in global workflow static data:

- `date`
- `sentToday`

`sentToday` increments only after `Send Draft` succeeds. If the workflow is stopped mid-run, `sentToday` reflects the successful sends that already reached `Increment Daily Counter` during that run.

The counter resets automatically when the Eastern calendar date changes. The reset check runs at the start of the manual execution, before each new batch, and before each successful send is counted.

n8n treats manual canvas executions differently from production trigger executions. Use one manual sending session as the safe operating unit for the day, and inspect node outputs during testing to confirm the counter behavior in your local n8n version.

## Stop the Workflow

To stop a running session:

1. Open the running execution in n8n.
2. Click the stop/cancel control for the execution.
3. Confirm the execution is no longer running.

Messages already sent by Gmail remain sent. The workflow does not edit or delete sent mail.

## View Executions and Logs

In n8n:

1. Open the workflow.
2. Open the Executions view.
3. Inspect the latest execution.
4. Click nodes to view input/output data.

If Gmail/API returns an error in `List Gmail Drafts`, `Get Draft Metadata`, or `Send Draft`, the execution stops so you can inspect the failure before continuing.

## Reset Static Data

Usually, do not reset static data. It resets automatically when the Eastern date changes.

If you must reset it:

1. Open `Check Daily Limit`.
2. Temporarily add this block immediately after `const staticData = $getWorkflowStaticData('global');`:

```javascript
staticData.date = null;
staticData.sentToday = 0;
return [{ json: { reset: true, timestamp: new Date().toISOString() } }];
```

3. Save and run one controlled manual execution.
4. Remove the temporary reset block.
5. Save again.
6. Confirm production settings are restored before real sending.

## Change Settings

Edit `Check Daily Limit`.

Change the daily limit:

```javascript
const DAILY_LIMIT = 300;
```

Change batch size:

```javascript
let MIN_BATCH_SIZE = 9;
let MAX_BATCH_SIZE = 18;
```

Change batch delay:

```javascript
let MIN_BATCH_DELAY_MINUTES = 5;
let MAX_BATCH_DELAY_MINUTES = 7;
```

Change per-email delay:

```javascript
let MIN_EMAIL_DELAY_SECONDS = 8;
let MAX_EMAIL_DELAY_SECONDS = 25;
```

Testing controls:

```javascript
const FORCE_TEST_MODE = false;
const TEST_SUBJECT_PREFIX = '[N8N TEST]';
const TEST_DAILY_LIMIT = 2;
const TEST_MIN_BATCH_SIZE = 1;
const TEST_MAX_BATCH_SIZE = 2;
```

Set `FORCE_TEST_MODE = true` only for controlled tests, then set it back to `false` before real sending.

## Safety

This workflow does not attempt to bypass Gmail sending limits. Gmail may still throttle or block sends. Test with harmless drafts before using real drafts.
