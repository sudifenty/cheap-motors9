# Cheap Motors — Dealership Website

A car dealership website in ONE file: plain HTML, Tailwind CSS (CDN) and
vanilla JavaScript. The live inventory is stored in a free **Supabase**
cloud database — you add a car in the dashboard, every visitor sees it on
their next refresh. No publish button, no tokens, nothing to remember.

| File | What it is |
|---|---|
| `index.html` | The whole website AND the owner dashboard in one page. Customers see the car grid, photo galleries, search, contact buttons and showroom location. The **Owner Login** button (top right) opens the dashboard on the same page behind a passcode |
| `README.md` | This guide |

## Run it on your computer
Double-click `index.html`. Any modern browser works. Your inventory lives
in the Supabase cloud; uploaded photos are compressed automatically in the
browser (≤760px JPEG, about 80–160 KB each, up to 6 per car).

## Photos, storage and the cloud

- **Photos are compressed before they are saved** (≤760px JPEG, roughly
  80–160 KB each), so cars stay small and the website stays fast even on
  mobile data.
- **Photos live in Supabase Storage, not in the database.** The first time
  you save after updating this file, every photo moves to the `photos`
  bucket automatically (one time — the pill shows "Syncing…" while it
  works). After that the inventory itself is tiny (a few KB per car), so
  the website loads **instantly** and updates appear in **real time**;
  photos are served from the Supabase CDN and cached by every visitor's
  browser individually.
- **The cloud is the real home of your inventory.** The browser's own
  storage (about 5 MB) is only an offline copy. If it ever fills up, cars
  keep saving and syncing normally — the dashboard just tells you the
  offline copy is paused. Your live website is not affected.
- After updating to this version, the **first sync automatically
  re-compresses older, oversized photos** — the website gets lighter and
  the offline copy fits on the device again.
- **All your cars are built into the file.** A compressed snapshot of the
  full stock (taken 6 Oct 2026 — all 14 cars: Evoque, Landcruiser, Mark X,
  Harrier, GLE400d, Noah, Vanguard, Subaru and the rest; photos ~25–55 KB
  each, 2 per car) is baked into `index.html`. A brand-new visitor sees
  ALL your cars on the very first paint — never sample cars, never an
  incomplete list — and the live cloud list (with the full photo
  galleries) takes over as soon as it loads. Repeat visits paint from the
  browser's cached copy instantly. Ask to refresh the baked-in snapshot
  whenever the lineup changes a lot — it does not update itself.

## Put it online with GitHub Desktop (step by step)

1. **Extract this ZIP first.** You end up with **one folder** that directly
   contains `index.html` and `README.md`. Do not drag the ZIP itself into
   GitHub Desktop — it can't read ZIPs.
2. Open **GitHub Desktop → File → Add Local Repository…** and select that folder.
   **Do not use "Clone"** — this repo is on your computer.
3. Click **Publish repository**. Name it e.g. `cheap-motors`, and **UNcheck
   "Keep this code private"** (free GitHub Pages needs a public repository).
4. Turn on the website: github.com → your repository → **Settings → Pages** →
   *Build and deployment* → **Source: Deploy from a branch** → **Branch: main**,
   **Folder: /(root)** → **Save**.
5. Wait 2–10 minutes, then visit `https://YOUR-USERNAME.github.io/REPO-NAME/`

## One-time cloud setup — Supabase (about 10 minutes)

This is what makes added cars visible to every visitor. You do it **once**.

1. Go to **supabase.com** → **Start your project** → sign up (free, no card
   needed) → **New project**. Name it e.g. `cheap-motors`, pick any region,
   choose a database password (save it somewhere, you rarely need it).
2. Wait 1–2 minutes while the project is created.
3. In the left menu open **SQL Editor** → **New query**, paste **all** of the
   setup SQL from the box below, click **Run**. It should say "Success".
4. Create your owner account: **Authentication → Users → Add user →**
   enter your email + a password → **Auto Confirm User: ON** → save.
   *This* email and password are what the dashboard will ask for later.
5. Copy your connection values: **Project Settings (the gear) → API** →
   copy the **Project URL** (looks like `https://abcd1234.supabase.co`) and
   the **publishable** key (older projects call it the “anon public” key —
   either works; it starts with `sb_publishable_`).
6. Open `index.html` in any text editor (Notepad works). Press **Ctrl+F**,
   search for `SUPABASE_URL` — you will find the connection block near the
   top of the code. Paste your values between the quotes:

   ```
   var SUPABASE_URL = 'https://abcd1234.supabase.co';
   var SUPABASE_ANON_KEY = 'sb_publishable_...your long key...';
   ```

   Save the file, then commit + push in GitHub Desktop.

   > **This copy is already connected** — your project's URL and key are
   > pasted in. Skip to step 7; redo this only if you ever create a new
   > Supabase project.
7. Open your website → **Owner Login** (passcode — see below) → click the
   cloud button at the top (**Local only**) → enter your Supabase owner
   email + password → **Save & Sign In**. Your current cars upload to the
   cloud and the button turns green (**Live**). Done — from now on every
   change you save is live for every visitor instantly.

### The setup SQL (copy the whole box)

```sql
-- Cheap Motors — one-time setup. Paste ALL of this into
-- Supabase → SQL Editor → New query → Run.
-- If a "cars" table was created from a tutorial (title / price / location /
-- image columns), this safely replaces it: the app stores photo galleries,
-- descriptions, sold status and the car order, which that table cannot hold.
do $$
begin
  if exists (select 1 from information_schema.columns
             where table_schema = 'public' and table_name = 'cars'
               and column_name = 'title') then
    execute 'drop table public.cars cascade';
  end if;
end $$;

create table if not exists public.cars (
  id         text primary key,
  data       jsonb not null,
  pos        int not null default 0,
  updated_at timestamptz not null default now()
);

create table if not exists public.app_settings (
  key   text primary key,
  value text
);

-- Visitor counting (the dashboard's "Live visitors" panel). Safe to re-run.
create table if not exists public.visits (
  id         bigint generated by default as identity primary key,
  vid        text,
  device     text,
  source     text,
  created_at timestamptz not null default now()
);

alter table public.cars         enable row level security;
alter table public.app_settings enable row level security;
alter table public.visits       enable row level security;

drop policy if exists "cars readable by everyone"     on public.cars;
drop policy if exists "settings readable by everyone" on public.app_settings;
drop policy if exists "owner manages cars"            on public.cars;
drop policy if exists "owner manages settings"        on public.app_settings;
drop policy if exists "anyone can record a visit"     on public.visits;
drop policy if exists "owner reads visits"           on public.visits;

-- Visitors (and the website) may READ the inventory…
create policy "cars readable by everyone"     on public.cars         for select using (true);
create policy "settings readable by everyone" on public.app_settings for select using (true);

-- …but only YOU (signed in with your owner account) may change anything.
create policy "owner manages cars"     on public.cars         for all to authenticated using (true) with check (true);
create policy "owner manages settings" on public.app_settings for all to authenticated using (true) with check (true);

-- Visitor counting: anyone's browser may ADD a visit record, only you can
-- read them. No names or personal data — just time, device and where the
-- visitor came from.
create policy "anyone can record a visit" on public.visits for insert to anon, authenticated with check (true);
create policy "owner reads visits"        on public.visits for select to authenticated using (true);

-- Fast photos: uploaded photos are stored as files in Supabase Storage
-- and served to visitors from the CDN. The inventory itself stays tiny, so
-- the website loads instantly. (The dashboard moves existing photos there
-- automatically the first time you save after this SQL.)
insert into storage.buckets (id, name, public)
values ('photos', 'photos', true)
on conflict (id) do nothing;

drop policy if exists "everyone reads photos"   on storage.objects;
drop policy if exists "owner uploads photos"    on storage.objects;
drop policy if exists "owner overwrites photos" on storage.objects;
drop policy if exists "owner deletes photos"    on storage.objects;
create policy "everyone reads photos"   on storage.objects for select using (bucket_id = 'photos');
create policy "owner uploads photos"    on storage.objects for insert to authenticated with check (bucket_id = 'photos');
create policy "owner overwrites photos" on storage.objects for update to authenticated using (bucket_id = 'photos') with check (bucket_id = 'photos');
create policy "owner deletes photos"    on storage.objects for delete to authenticated using (bucket_id = 'photos');

-- Live updates: visitors' open tabs refresh themselves when you publish a
-- change (no reload needed anywhere).
do $$
begin
  if not exists (select 1 from pg_publication_tables where pubname = 'supabase_realtime' and tablename = 'cars') then
    execute 'alter publication supabase_realtime add table public.cars';
  end if;
  if not exists (select 1 from pg_publication_tables where pubname = 'supabase_realtime' and tablename = 'app_settings') then
    execute 'alter publication supabase_realtime add table public.app_settings';
  end if;
end $$;
```

**Already created the table or policies from a tutorial?** The SQL above
detects a tutorial-style `cars` table and replaces it, and it replaces any
"everyone can write" policies on purpose: with public insert/update/delete,
**anyone on the internet could delete your cars**. After running it, only
your owner account (step 4) can change the inventory — visitors just read it.

## Everyday use (the owner's workflow)

1. Open your website → **Owner Login** → type the dashboard passcode.
2. Add or edit cars — up to 6 photos each, plus a description (customers
   see it in View Details). Every save is pushed to the
   cloud automatically (you'll see the pill flash **Syncing…** then **Live**).
   On a new device or website address, the cloud sign-in form opens **by
   itself** the first time you enter the dashboard — sign in once there and
   every save syncs from then on.
3. Updates are instant everywhere: your website view changes the moment you save, and any open tab — yours or a visitor's — updates itself (instantly once the Realtime lines in the setup SQL have run; otherwise within about half a minute). A “✅ Car published live!” toast confirms every successful sync.
4. **Live visitor stats** — the dashboard counts your visitors for you: a
   “Website Visits” stat box, a feed of recent visits (“Just now · Android
   phone · from facebook.com”) and a 🔔 toast while you work when someone
   opens your website. It needs the `visits` lines in the setup SQL above
   (re-run the whole box — it is safe to repeat); counting then starts from
   zero, and your own signed-in browser is never counted.

### The cloud button (top of the dashboard)

| What you see | What it means |
|---|---|
| 🟢 **Live** | Connected and signed in — saves sync automatically |
| 🟡 **Not signed in** | The file is configured, but you need to sign in (click it, enter your Supabase email + password). **Each website address keeps its own sign-in** — if you start working from a new address (your own domain, a different device), sign in once there |
| ⚪ **Local only** | `index.html` has no Supabase URL/key pasted in yet — see setup step 6 |
| Orange **Not synced** badge | You made changes but the cloud could not be reached (offline?). Your changes are safe on this device — click the cloud button → **Sync now** when back online |

Clicking the cloud button always opens the cloud settings, where you can
sign in, change your owner email/password, use **Sync now**, and set the
**dashboard address** (see the PHP section below).

### Is the key in the file safe?
Yes — it is **public on purpose**. Visitors' browsers need it to *read* your
inventory, and reading is all it allows. *Writing* requires your owner
account (email + password), which is stored only in your browser and never
appears in the file.

## Changing the dashboard passcode
Open `index.html` in any text editor and find the line
`var CM_PASSCODE = 'admin123#';` near the bottom. Change the text between the
quotes, save, commit + push.

## About the passcode's limits (important)
This one-file site is hosted on GitHub Pages, which cannot run server code.
That means the dashboard passcode lives in the page — a technical visitor
could find it in the page source. It keeps casual visitors out, nothing more.
Your inventory itself is safe: nobody can change the cars in the cloud
without your Supabase owner account.

For a password that never touches the browser, use the separate PHP package
(`admin.php` + `logout.php` on PHP hosting). It shares the same Supabase
cloud. Once it is online, put its address in the dashboard's cloud settings
(**Dashboard address**) — after your next save, the website's Owner Login
button automatically sends you there instead.

## Seeing a 404? Check these in order
1. Wait a few minutes; check the repo's **Actions** tab.
2. `index.html` must be at the ROOT of the repo, never inside a folder.
3. GitHub Pages must be on (Settings → Pages → Branch: main, Folder: /root).
4. The repo must be public on a free plan.
5. Check the URL matches your username and repo name exactly.

## Cars not updating for visitors? Check these
1. The cloud button must be green (**Live**) when you save.
2. `index.html` on GitHub must contain your Supabase URL + anon key
   (setup step 6) — if you forgot to push, visitors read nothing.
3. Open tabs update on their own — instantly with the Realtime SQL from the setup section, otherwise within ~30 seconds. A manual refresh always works too.
4. If the badge says **Not synced**, click the cloud button → **Sync now**.
5. A brand-new visitor first sees the built-in stock instantly, then the
   live list arrives from the cloud — with big photo galleries that can
   take a while on mobile data. After your first save with this version
   (cloud photos slimmed automatically), the live list arrives much
   faster.
