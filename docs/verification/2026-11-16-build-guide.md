# Course build guide implementation

The checkout was fast-forwarded from b17c091 to upstream/starter (59d56fa)
without conflicts. All six exercise gaps are implemented. This is the
single-owner hosted course demo described in `docs/engineering/hosted-demo.md`.

## Verification

- `make setup`: passed with network permission.
- After Docker enablement, combined `make check`: all six pieces PASS.
- `make check-6`: passed, 13 tests plus TypeScript checking.
- Full frontend suite: 109 files and 1,479 tests passed.
- `make build`: passed; Vite reports its existing large-bundle warning.
- Backend startup/owner subset: 6 passed (`test_hosted_demo.py`, selecting
  `hosted_mode_comes or public_host or resolve_returns`).
- Direct signed JWT checks: valid and normalized email accepted; missing or
  uninvited email rejected; empty, malformed, wrong-key, expired, wrong-audience
  and wrong-issuer tokens rejected (10 assertions).
- Docker Desktop is now reachable from WSL; backend image builds successfully.
- Checks 1–5 passed individually (17, 11, 6 and 32 backend tests, plus
  all four container assertions).
- Local Supabase database started; all five migrations and seed applied.
  Local reset completed successfully.
- Public container startup with only SUPABASE_URL exits 2 and names
  ALLOWED_EMAILS and WORKSPACE_USER_ID as missing.
- Mock chat/Contacts API smoke passed against the seeded local database:
  fixed reply, empty board (HTTP 200), and transcript reload through a new
  backend instance. Five mock/persistence regression tests also passed.
- No deployed Google sign-in, live model answer, persistence journey or
  deployed screenshots have been verified.

## Local verification completed

Docker Desktop WSL integration is enabled. Supabase CLI 3.2.0 lives in
`/tmp/sourcecado-tools/node_modules/.bin` for this session. The local database
is running on 127.0.0.1:53322. Initial registry downloads encountered EOF
errors; CLI registry fallback succeeded. Backend image is tagged
`sourcecado-backend`. No frontend or backend development server was left running.

To inspect the local app yourself, run `make backend-mock` and `make gui`
in separate terminals, then open http://127.0.0.1:5180. Browser visual QA
and deployed QA remain outstanding; the completed smoke used API clients.

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
