# Go-Live Readiness Dashboard

A single-file steering dashboard for the Quote Management Programme (CPQ platform
deployment), backed by Supabase for storage and authentication.

Everything lives in [`index.html`](index.html) — no build step, no dependencies to
install. The Supabase JS client is loaded from a CDN at runtime.

## How it works

The dashboard's entire contents (metrics, readiness scorecard, risks, issues, defects,
timeline, custom sections, and so on) are held as a single JSON document. That document
is stored in one row of the `dashboards` table in Supabase, so everyone with access sees
and edits the same copy rather than a personal fork in their own browser.

- **Edits save automatically**, about a second after you stop typing. The topbar shows
  the current state: *Unsaved changes…* → *Saving…* → *Saved just now*.
- **Save weekly version** takes a named snapshot into `dashboard_versions`. Snapshots are
  shared with the team and can be restored or deleted from the Version history card.
- A local copy is also kept in the browser as a backup, so a dropped connection or an
  expired session can't lose work in progress.

## Access

Sign-in is email + password via Supabase Auth. Two things must both be true for someone
to get in:

1. They have a user in **Authentication → Users**.
2. Their email address is listed in the **`dashboard_admins`** table.

The second is the real gate. Row-level security checks it on every read and write, so a
Supabase account on its own grants nothing — which matters because the project's anon key
is published in this page's source (that is normal and safe, but it does mean the account
list can't be the only control).

### Adding someone

1. Supabase → **Authentication → Users → Add user**, with their email and a password.
2. Supabase → **Table editor → `dashboard_admins` → Insert row**, with the same email.

Removing their row from `dashboard_admins` revokes access immediately, without deleting
their account.

## Running it locally

Supabase Auth needs a real HTTP origin, so opening the file directly from disk
(`file://`) will not work. Serve the folder instead:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

## Hosting

Published with GitHub Pages from the `main` branch, root folder. Whatever URL you serve
it from must be listed in Supabase under **Authentication → URL Configuration**.

## Sharing a snapshot outside the team

**Share as HTML** downloads a standalone copy with the current data baked in. That file
needs no login and no network — it opens straight from disk and renders read-only, with
all editing controls removed. Use it for people who need to see the dashboard but
shouldn't have an account.

It contains a full copy of the programme data, so treat it as confidential.

## Database

| Table | Purpose |
|---|---|
| `dashboards` | One row per dashboard. `data` holds the whole document as JSONB; `revision` is an optimistic lock that stops two editors silently overwriting each other. |
| `dashboard_versions` | Named snapshots for the version history. Immutable once written. |
| `dashboard_admins` | Email allowlist. Not reachable through the API at all — edit it in the Supabase dashboard. |
