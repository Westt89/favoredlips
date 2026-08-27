# FavoredLips By T — Supabase Setup Guide

This version saves everything (products, photos, videos, reviews, contact
info) to Supabase — a free, cloud-hosted backend. No credit card, no local
server, no XAMPP. Just one HTML file that talks straight to the cloud.

Takes about 10 minutes.

## 1. Create your Supabase project

1. Go to **https://supabase.com** → **Start your project** → sign up (GitHub or email).
2. Click **New project**. Pick any name (e.g. `favoredlips-by-t`), set a database password (save it somewhere, you likely won't need it again), pick a region close to Ghana (e.g. an EU region), and click **Create new project**.
3. Wait about a minute while it provisions.

## 2. Get your API keys

1. In your project, go to **Settings → API** (in the left sidebar, under Project Settings).
2. Copy the **Project URL** and the **`anon` `public`** key.
3. Open `index.html` in a text editor, find near the top of the `<script>` tag:

   ```js
   var SUPABASE_URL = "YOUR_SUPABASE_PROJECT_URL";
   var SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
   ```

   and paste your real values in.

## 3. Create your tables

1. In the left sidebar, open the **SQL Editor**.
2. Click **New query**, paste in the following, and click **Run**:

```sql
create extension if not exists pgcrypto;

create table if not exists products (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  category text not null,
  price text not null,
  description text,
  image_urls text[] default '{}',
  video_url text,
  created_at timestamptz not null default now()
);

create table if not exists reviews (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  rating int not null,
  review_text text not null,
  review_date text,
  created_at timestamptz not null default now()
);

create table if not exists business_info (
  id int primary key default 1,
  lede text,
  about text,
  location text,
  hours text,
  whatsapp text,
  instagram text,
  tiktok text,
  snapchat text
);

insert into business_info (id, lede, about, location, hours, whatsapp, instagram, tiktok, snapchat)
values (
  1,
  'Hydrating, non-sticky, long-lasting shine — rich pigment, cruelty-free, and made with purpose. Delivery nationwide, based in Takoradi.',
  'Hydrating, non-sticky, long-lasting shine — rich pigment, cruelty-free, and made with purpose. Delivery nationwide, based in Takoradi.',
  'Takoradi, Ghana — Delivery Nationwide',
  'Mon–Sat, 9am – 7pm',
  '233595717473',
  'favoredlipsbyT',
  '',
  ''
)
on conflict (id) do nothing;

alter table products enable row level security;
alter table reviews enable row level security;
alter table business_info enable row level security;

create policy "Public read products" on products for select using (true);
create policy "Authenticated insert products" on products for insert with check (auth.role() = 'authenticated');
create policy "Authenticated update products" on products for update using (auth.role() = 'authenticated');
create policy "Authenticated delete products" on products for delete using (auth.role() = 'authenticated');

create policy "Public read reviews" on reviews for select using (true);
create policy "Authenticated insert reviews" on reviews for insert with check (auth.role() = 'authenticated');
create policy "Authenticated delete reviews" on reviews for delete using (auth.role() = 'authenticated');

create policy "Public read business_info" on business_info for select using (true);
create policy "Authenticated update business_info" on business_info for update using (auth.role() = 'authenticated');
create policy "Authenticated insert business_info" on business_info for insert with check (auth.role() = 'authenticated');
```

You should see "Success. No rows returned" — that means it worked.

## 4. Create your storage bucket (for photos & videos)

1. In the left sidebar, go to **Storage** → **New bucket**.
2. Name it exactly: `product-media`
3. Toggle **Public bucket** ON (this lets customers see photos without needing to log in — write access still stays locked to you).
4. Click **Create bucket**.
5. Back in the **SQL Editor**, run this once to allow uploads/deletes only when signed in:

```sql
create policy "Public read product media"
  on storage.objects for select
  using (bucket_id = 'product-media');

create policy "Authenticated upload product media"
  on storage.objects for insert
  with check (bucket_id = 'product-media' and auth.role() = 'authenticated');

create policy "Authenticated delete product media"
  on storage.objects for delete
  using (bucket_id = 'product-media' and auth.role() = 'authenticated');
```

## 5. Create your admin login

1. Go to **Authentication → Users** in the left sidebar.
2. Click **Add user** → **Create new user**.
3. Enter an email and password (doesn't need to be a real inbox — just something you'll remember). Leave "Auto Confirm User" checked if asked.
4. Click **Create user**.

This is what you'll sign in with at `#favoredlips-admin` — no separate passcode, this is a real login.

## 6. Open your site

Just open `index.html` directly — double-click it, or upload it to any web host (Hostinger, Netlify, GitHub Pages, your own domain). Unlike the earlier versions, **this one doesn't need a local server at all** — it talks straight to Supabase's cloud API from any browser.

To manage your shop: add `#favoredlips-admin` to the end of the address and press enter, then sign in with the email/password from step 5.

## Costs

Supabase's free tier includes 500MB database, 1GB file storage, 5GB bandwidth/month, and 50,000 monthly active users — no credit card required, and it doesn't expire. The one thing to know: **a free project pauses itself automatically after 7 days with no activity**, and needs a manual "Restore" click in your Supabase dashboard to wake back up. If your shop gets regular visits this basically never happens; if you expect quiet stretches, just check in on the dashboard occasionally.

## Troubleshooting

- **"That email or password isn't right"** → double-check the user exists under Authentication → Users.
- **Uploads fail / permission errors** → recheck you ran all the policy SQL in steps 3 and 4, and that the bucket is named exactly `product-media`.
- **Nothing loads / grid stays empty with a "not connected" message** → double-check SUPABASE_URL and SUPABASE_ANON_KEY are pasted in correctly, with no extra quotes or spaces.
- **Site was working, now nothing loads** → your project may have auto-paused from inactivity; go to your Supabase dashboard and click Restore/Resume on the project.
