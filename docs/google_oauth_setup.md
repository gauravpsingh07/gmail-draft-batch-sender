# Google OAuth Setup

This workflow uses the Gmail API through n8n's Google OAuth2 API credential.

## Google Cloud Project

1. Go to Google Cloud Console.
2. Create a project, or select an existing project you control.
3. Enable the Gmail API for that project.

## OAuth Consent Screen

1. Open the OAuth consent screen settings.
2. Choose the user type that fits your account.
3. Add the app name and contact details.
4. Add yourself as a test user if the app is in testing mode.
5. Add this required scope:

```text
https://www.googleapis.com/auth/gmail.modify
```

The workflow needs `gmail.modify` because it must send existing Gmail drafts using `users.drafts.send`.

## OAuth Client ID

1. Create an OAuth Client ID.
2. Application type: `Web application`.
3. Add this Authorized redirect URI:

```text
http://localhost:5678/rest/oauth2-credential/callback
```

4. Save the client.
5. Copy the Client ID and Client Secret.

## n8n Credential

1. Open n8n at `http://localhost:5678`.
2. Go to Credentials.
3. Create a `Google OAuth2 API` credential.
4. Paste the Client ID and Client Secret.
5. Add the scope:

```text
https://www.googleapis.com/auth/gmail.modify
```

6. Connect the credential and complete the Google OAuth flow.

## Attach Credential

Attach the connected Google OAuth2 API credential to all Gmail HTTP Request nodes:

- `List Gmail Drafts`
- `Get Draft Metadata`
- `Send Draft`

Use the Gmail account that contains the drafts you want to send.
