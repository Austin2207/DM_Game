# Beat Our Hand — hosted game (phone/QR version)

Single self-contained page: `index.html`. Blind commit (one shot per device),
scoring engine copied verbatim from `../../model/federal-model-v2.html`
(same seed 20260727, same 1,500 draws — every number matches the report).

## Two modes

- **Offline mode (default, zero setup):** `CLOUD.url` is empty → the page works
  standalone, scoreboard shows this device only. This is the demo-day fallback.
- **Cloud mode:** fill in `CLOUD` at the top of the `<script>` in `index.html`
  → every lock-in is POSTed to a shared table; the scoreboard polls it every
  2.5 s. The presentation site's final scoreboard reads the same table.

## Supabase setup (free tier, ~5 minutes, Austin does this once)

1. Create a project at supabase.com → note the **Project URL** and **anon key**
   (Settings → API).
2. SQL editor → run:

```sql
create table public.scores (
  id bigint generated always as identity primary key,
  name text not null check (char_length(name) <= 24),
  score double precision,
  policy jsonb not null,
  created_at timestamptz default now()
);
alter table public.scores enable row level security;
create policy "anon insert" on public.scores for insert to anon with check (true);
create policy "anon read"   on public.scores for select to anon using (true);
```

3. Paste URL + anon key into `CLOUD={url:'https://xxxx.supabase.co', anonKey:'...', table:'scores'}`
   in `index.html`.
4. To wipe the board before the talk: SQL editor → `truncate public.scores;`

## Deploy

Any static host. Simplest: drop this folder into Vercel (`vercel deploy`) or
GitHub Pages. Then make a QR code pointing at the URL for slide/section 5.

## Anti-tamper note

The scoreboard **recomputes every score from the stored `policy` client-side**
(the engine ships with the page) and ignores the stored `score` field, which is
kept only for debugging. Faking a score is pointless; a faked policy still gets
its true score. Invalid policies are filtered out.

## For the presentation site's live scoreboard

Same read the game uses:

```js
const rows = await (await fetch(URL+'/rest/v1/scores?select=name,policy,created_at&order=created_at.asc&limit=200',
  {headers:{apikey:KEY, Authorization:'Bearer '+KEY}})).json();
// then recompute each score with the same engine (copy evaluate/mc verbatim)
```
