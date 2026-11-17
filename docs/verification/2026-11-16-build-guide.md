# Course build guide implementation

The checkout was fast-forwarded from b17c091 to upstream/starter (59d56fa)
without conflicts. All six exercise gaps are implemented. This is the
single-owner hosted course demo described in `docs/engineering/hosted-demo.md`.

## Verification

- `make setup`: passed with network permission.
- `make check-6`: passed, 13 tests plus TypeScript checking.
- `make build`: passed; Vite reports its existing large-bundle warning.
- Backend startup/owner subset: 6 passed (`test_hosted_demo.py`, selecting
  `hosted_mode_comes or public_host or resolve_returns`).
- Direct signed JWT checks: valid and normalized email accepted; missing or
  uninvited email rejected; empty, malformed, wrong-key, expired, wrong-audience
  and wrong-issuer tokens rejected (10 assertions).
- Checks 1, 2, 4 and 5 could not complete: local Postgres is unavailable.
- Check 3 could not run: Docker is unavailable inside this WSL distro.
- No deployed Google sign-in, live model answer, persistence journey or
  deployed screenshots have been verified.

## Resume local verification

Start Docker Desktop on Windows and enable this distro under Settings →
Resources → WSL Integration. Confirm `docker info` succeeds inside WSL.
The Supabase CLI downloaded for this session lives in
`/tmp/sourcecado-tools/node_modules/.bin`; add that directory to PATH for
this session (or install the CLI persistently).

```bash
export PATH="/tmp/sourcecado-tools/node_modules/.bin:$PATH"
make db-up
make db-reset
make check
```

`db-reset` resets the local development database: use it only for disposable
local data. Then run `make backend-mock` and `make gui` in separate terminals.
At http://127.0.0.1:5180, check the fixed mock answer and empty Contacts table.

## Account setup still required

Use the supplied guide's dashboard steps; never put secret values in Git.
Railway, Supabase and Vercel integrations were discovered but are not connected.

1. Supabase: sign in, link the intended project, push the five migrations,
   and confirm local/remote versions agree. Enable Google with the OAuth
   client ID/secret from Google Cloud.
2. Google Cloud: create a web OAuth client; add the frontend origin and
   `https://PROJECT-REF.supabase.co/auth/v1/callback` redirect; add your own
   and a second test account to the OAuth audience.
3. Vercel: root `frontend`; Production variables `VITE_SUPABASE_URL` and
   `VITE_SUPABASE_ANON_KEY`. Set Supabase Site URL and Redirect URLs to the
   actual frontend URL. Deploy, sign in once, and copy your Supabase user UID.
4. Railway: root `backend`, health path `/healthz`, one replica, volume
   `/data/club`, and `RAILWAY_RUN_UID=0`. Set `DATABASE_URL` (session pooler,
   TLS required), `DEEPSEEK_API_KEY`, `SUPABASE_URL`, `ALLOWED_EMAILS`,
   `WORKSPACE_USER_ID` (your UID), and `ALLOWED_ORIGINS` (frontend origin).
   Generate its public domain and verify `/healthz` returns `{"ok":true}`.
5. Vercel: set `VITE_CLUB_HTTP` to the backend HTTPS origin and redeploy.
6. Complete guide checks 4.1–4.7 and capture screenshots: private-window
   sign-in gate, invited sign-in, real chat answer (391), Contacts, reload
   persistence, uninvited-account rejection, and backend health response.

The mock provider must not be enabled for the real-answer check. The frontend
Supabase variables are build-time settings and require a redeploy after edits.
