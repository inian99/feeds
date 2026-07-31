# Turning on sync

Optional. Without it the app saves to the device it's running on and works exactly
as before. With it, entries follow you between your phone, your desktop and any
other device you install it on.

Takes about five minutes, once.

## 1. Make a Supabase project

Sign up at supabase.com and create a project. The free tier is far more than this
needs — a year of feeds is well under a megabyte.

## 2. Create the table

Project → SQL Editor → paste this and run it:

```sql
create table tracker (
  id         text primary key,
  payload    jsonb not null,
  updated_at timestamptz not null default now()
);

alter table tracker enable row level security;

create policy "anon read write" on tracker
  for all to anon
  using (true) with check (true);
```

## 3. Connect the first device

Project → Settings → API. Copy the **Project URL** and the **anon public** key.

In the app, tap the sync row above the buttons at the bottom, then paste:

```
https://yourproject.supabase.co
eyJhbGciOiJIUzI1NiIsInR5cCI6…
```

URL on the first line, key on the second. Tap **Connect**. The app generates a
random room code and pushes whatever you've already logged.

## 4. Connect your other devices

Open the sync row again on the device you just set up. It shows a **sync token** —
one long string that packs the URL, key and room code together. Copy it, then on the
next device tap the sync row and paste that single token. Nothing else to type.

## How it behaves

Sync runs two seconds after any change, and again whenever you come back to the app.
There's no sync button to remember, though the sheet has one if you want to force it.

Entries merge by id rather than one device overwriting the other, so if you log a feed
on your phone while your desktop is asleep, nothing is lost when they meet. Deletions
travel as tombstones, which is why a deleted entry doesn't reappear from the other
device. Colours stay a per-device choice — a phone in a dark bedroom and a laptop in
daylight want different things.

Offline, everything keeps working against local storage; the row at the bottom shows
the failure and it retries on your next change or next time you open the app.

**Photos stay on the device that took them.** They're around 100KB each and would turn
a trivial JSON sync into a blob-storage problem. Say the word if you want them synced
too — Supabase Storage would handle it, it's just more moving parts.

## About the security of this

Access is controlled by two secrets: the anon key and the random room code. Anyone
holding both can read and write your log. That's a deliberate trade for not building
authentication into a personal feed tracker, but it means:

- Don't publish the sync token anywhere. It packs all three values into one string.
- Neither the URL, the key nor the room code is in this repository. You paste them into
  the running app and they're written to that device's storage, so the source can be
  public without leaking anything.
- Rotate by generating a new anon key in Supabase and reconnecting each device.

If you'd rather have real auth, Supabase's email magic-link plus a `user_id` column
and an RLS policy of `auth.uid() = user_id` is the proper version. It's maybe an hour
of work and adds a login screen at 3am, which is the reason I didn't start there.
