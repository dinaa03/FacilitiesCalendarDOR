# Bridgewater PTC – Site Visibility Calendar

Month / Week / Day views, filters, a Today summary, and add / edit / delete for **Events**, **Contractors**, and **PTO / OOO**.

There are two things needed so everyone sees the same calendar:

1. **A shared database** (Supabase, free) – so entries are saved permanently instead of in one browser.
2. **A public link** (Netlify or GitHub Pages, free) – so any device/TV can open it. Live Server only works on your own PC.

The footer shows which mode you're in:
- 🟢 **Shared · live** – connected; every device sees the same data.
- 🟠 **Local only** – Supabase keys not filled in yet; data stays in that one browser.
- 🔴 **Connection issue** – keys wrong or network blocked.

---

## Step 1 – Create the shared database (about 5 min)

1. Go to **supabase.com** → sign up → **New project** (any name, e.g. `bw-calendar`; pick region East US).
2. Left menu → **SQL Editor** → **New query** → paste everything below → **Run**:

```sql
create table if not exists public.entries (
  id          uuid primary key default gen_random_uuid(),
  type        text not null check (type in ('event','contractor','pto')),
  title       text not null,
  purpose     text,
  start_date  date not null,
  end_date    date not null,
  all_day     boolean not null default false,
  start_time  time,
  end_time    time,
  location    text,
  host        text,
  notes       text,
  created_at  timestamptz not null default now()
);

create index if not exists entries_dates_idx on public.entries (start_date, end_date);

alter table public.entries enable row level security;

-- Anyone with the link can view, add, edit and delete (no sign-in, same as today)
create policy "read"   on public.entries for select to anon using (true);
create policy "insert" on public.entries for insert to anon with check (true);
create policy "update" on public.entries for update to anon using (true) with check (true);
create policy "delete" on public.entries for delete to anon using (true);
```

3. Left menu → **Project Settings** → **API** (or **Data API** / **API Keys**). Copy:
   - **Project URL** (looks like `https://abcdxyz.supabase.co`)
   - **anon public** key (long text starting with `eyJ...` or `sb_publishable_...`)
   ⚠️ Never use the `service_role` / secret key.

4. Open `app.js` and paste them at the very top:

```js
const SETTINGS = {
  SUPABASE_URL: "https://abcdxyz.supabase.co",
  SUPABASE_ANON_KEY: "eyJhbGciOi...",
  TABLE: "entries",
  REFRESH_SECONDS: 30
};
```

5. Refresh the page – the footer should say **Shared · live**. Add a test event, refresh: it stays.
   Any entries you previously saved in that browser are uploaded automatically the first time.

## Step 2 – Put it on a link any device can open

**Netlify (easiest):**
1. Go to **app.netlify.com/drop**.
2. Drag the whole folder (`index.html`, `styles.css`, `app.js`) onto the page.
3. You get a link like `https://bw-site-calendar.netlify.app` – open it on the TV, phone, laptop.
4. To update later: Netlify → your site → **Deploys** → drag the folder again.

**GitHub Pages (alternative):** new repo → upload the files → Settings → Pages → Branch `main` / root → Save.

Other screens pull new data every 30 seconds (and instantly when a tab is reopened).

## Run locally in VS Code (testing)
- Keep all files together, open the folder, install **Live Server** (Ritwick Dey), open `index.html` → **Go Live**.

## Notes
- No passkey: anyone with the link can add/edit/delete. Don't put confidential info in the notes.
- Free Supabase projects pause after ~1 week of zero activity. A TV showing the calendar keeps it active; if it ever pauses, click **Restore** in the Supabase dashboard.
- Check with IT if the Nestlé network blocks `*.supabase.co` or `*.netlify.app` (footer would show **Connection issue**).
