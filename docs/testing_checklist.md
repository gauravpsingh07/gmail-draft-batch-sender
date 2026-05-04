# Testing Checklist

Use harmless drafts and an alternate email account you control. Do not test with real outreach drafts first.

## Prepare Test Drafts

1. Use the same Gmail account connected to n8n through the Google OAuth2 API credential.
2. Create 2 harmless Gmail drafts.
3. Set each draft's `To` field to an alternate email address you control.
4. Make each test draft subject start with:

```text
[N8N TEST]
```

Example subjects:

```text
[N8N TEST] n8n draft sender test 1
[N8N TEST] n8n draft sender test 2
```

In `FORCE_TEST_MODE`, the workflow fetches draft metadata and only sends drafts whose subject starts with `[N8N TEST]`. Drafts without that prefix are skipped and are not sent.

## Temporary Test Settings

In `Check Daily Limit`, temporarily set:

```javascript
const FORCE_TEST_MODE = true;
```

Confirm:

```javascript
const TEST_SUBJECT_PREFIX = '[N8N TEST]';
const TEST_DAILY_LIMIT = 2;
const TEST_MIN_BATCH_SIZE = 1;
const TEST_MAX_BATCH_SIZE = 2;
```

Do not turn `FORCE_TEST_MODE` off until testing is complete.

## Manual Test

1. Save the workflow.
2. Confirm the Google OAuth2 API credential is attached to:
   - `List Gmail Drafts`
   - `Get Draft Metadata`
   - `Send Draft`
3. Click `Execute Workflow`.
4. Confirm only `[N8N TEST]` drafts are sent.

## Confirm Results

Check Gmail:

- The 2 test drafts were sent.
- The test drafts disappeared from Drafts.
- The test messages appear in Sent.
- Message content was not modified.
- Any non-test drafts remain in Drafts.

Check n8n:

- `Check Daily Limit` outputs `FORCE_TEST_MODE_ACTIVE`.
- `Check Daily Limit` includes `testSubjectPrefix`.
- `Apply Test Subject Filter` only passes subjects starting with `[N8N TEST]`.
- `Increment Daily Counter` shows `sentToday` increasing after successful sends.
- The workflow stops at the test daily limit of 2, or stops earlier if no matching test drafts remain.
- `Send Draft` uses `users.drafts.send`.

## Restore Production Settings

Before real sending, restore:

```javascript
const FORCE_TEST_MODE = false;
```

Confirm production settings:

- Manual execution only
- Daily limit: 300 every day
- Batch size: random 9 to 18
- Batch delay: random 5 to 7 minutes
- Per-email delay: random 8 to 25 seconds

Production mode does not require the `[N8N TEST]` subject prefix unless you intentionally keep test mode enabled or customize the filter.

## Real Sending

After confirming the test behavior and restoring production settings, open the workflow and click `Execute Workflow` manually when you want to start a real sending session. Gmail limits still apply, and Gmail may throttle or block sending even when the workflow is operating as configured.
