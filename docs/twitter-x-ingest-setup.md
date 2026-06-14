# X/Twitter ingest setup

Use official X API user-context auth. Do not scrape browser cookies or reuse a logged-in browser session.

## Recommended: OAuth 1.0a read-only user token

1. Create an X Developer account and app.
2. Set the app permissions to read-only.
3. Generate user access tokens for your account.
4. Add these GitHub repository secrets:
   - `X_USER_ID`: your numeric X user id
   - `X_API_KEY`: app API key
   - `X_API_SECRET`: app API key secret
   - `X_ACCESS_TOKEN`: user access token
   - `X_ACCESS_TOKEN_SECRET`: user access token secret
5. Run the `Ingest papers` GitHub Action manually once.

The ingest script calls:

```text
GET https://api.x.com/2/users/:id/timelines/reverse_chronological
```

It stores only sanitized feed metadata in `papers/twitter-feed.json` and uses arXiv links found in tweets as ranking signals for `papers/papers.json`.

## OAuth 2 alternative

You can set `X_USER_ACCESS_TOKEN` and `X_USER_ID`, but OAuth 2 access tokens expire. For a scheduled GitHub Action, OAuth 1.0a user tokens are simpler unless you also implement refresh-token rotation.
