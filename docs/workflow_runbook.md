# Workflow Runbook

## Behavior

- Manual Trigger starts or resumes a sending session
- Schedule Trigger runs every 1 minute as the automatic runner
- Daily limit: 300 drafts per Eastern calendar date
- Batch size: random 9 to 18 drafts
- Batch delay: random 5 to 7 minutes using `nextBatchAllowedAt`
- Per-email delay: random 8 to 25 seconds

The workflow sends at most one batch per execution:

```text
Manual start or scheduled check -> check runner state -> list drafts -> optional test metadata filter -> send one batch -> set nextBatchAllowedAt -> stop
```

It no longer uses a long-running `Wait Random Batch Delay` loop. The only Wait node left is the short per-email delay inside the current batch.

Test mode includes a subject prefix safety filter. When `FORCE_TEST_MODE = true`, the workflow only sends drafts whose subject starts with `[N8N TEST]`. Test mode disables `sendingEnabled` after one safe test batch. Production mode does not use that prefix filter unless you intentionally keep test mode enabled or customize the filter.

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

## Start Docker

1. Start Docker Desktop.
2. Run:

```bash
docker compose up -d
```

3. Open `http://localhost:5678`.

## Start Sending

1. Confirm the workflow is active.
2. Open the workflow.
3. Click `Execute Workflow` once.

The manual run sets `sendingEnabled = true`, sends the first batch if the daily limit allows, and stores the next randomized batch time. After that, the active Schedule Trigger continues automatically.

## Confirm Runner State

Open the latest execution and inspect:

- `Check Daily Limit And Runner State`
- `Set Next Batch Time And Session State`

Useful fields:

- `sendingEnabled`
- `nextBatchAllowedAt`
- `sentToday`
- `dailyLimit`
- `reason`

`NEXT_BATCH_SCHEDULED` means the next one-minute schedule checks will wait until `nextBatchAllowedAt`.

During an active batch, `Check Daily Limit And Runner State` temporarily reserves `nextBatchAllowedAt` to prevent overlapping scheduled executions. `Set Next Batch Time And Session State` overwrites it with the real 5 to 7 minute next-batch time when the batch finishes.

## Stop Sending

Deactivate the workflow in n8n.

This stops future Schedule Trigger runs. If a batch execution is currently running, open the execution and stop it if needed. Messages already sent by Gmail remain sent.

## Resume Sending

1. Activate the workflow.
2. Open the workflow.
3. Click `Execute Workflow` once.

Manual execution starts or resumes the session and allows the next batch immediately if the daily limit allows.

## Static Data

The workflow stores these keys in global workflow static data:

- `date`
- `sentToday`
- `sendingEnabled`
- `nextBatchAllowedAt`

`sentToday` increments only after `Send Draft` succeeds. If an execution is stopped mid-batch, `sentToday` reflects successful sends that already reached `Increment Daily Counter`.

The counter resets automatically when the Eastern calendar date changes. A new day does not automatically enable sending unless `sendingEnabled` was intentionally left true. When the daily limit is reached, the workflow sets `sendingEnabled = false`.

## Errors

If Gmail/API returns an error in `List Gmail Drafts`, `Get Draft Metadata`, or `Send Draft`, the workflow logs the error in `Log Gmail API Error And Disable Session`, sets `sendingEnabled = false`, and stops that execution path.

## Reset Static Data

Usually, do not reset static data. It resets `sentToday` automatically when the Eastern date changes.

If you must reset it:

1. Open `Check Daily Limit And Runner State`.
2. Temporarily add this block immediately after `const staticData = $getWorkflowStaticData('global');`:

```javascript
staticData.date = null;
staticData.sentToday = 0;
staticData.sendingEnabled = false;
staticData.nextBatchAllowedAt = null;
return [{ json: { reset: true, timestamp: new Date().toISOString() } }];
```

3. Save and run one controlled execution.
4. Remove the temporary reset block.
5. Save again.
6. Confirm production settings are restored before real sending.

## Change Settings

Edit `Check Daily Limit And Runner State`.

Change the daily limit:

```javascript
const DAILY_LIMIT = 300;
```

Change batch size:

```javascript
let MIN_BATCH_SIZE = 9;
let MAX_BATCH_SIZE = 18;
```

Change batch interval:

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
