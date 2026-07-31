# Feeds & nappies

One file that runs in two places. Opened as a Claude artifact it saves to your Claude
account; served as a web page it saves to the browser on that device. The app detects
which it's in at startup — you don't configure anything.

The rest of this file is about the second case: installing it on an iPhone.
Five files, no build step, no dependencies.

## Using it

The ring at the top is the whole app at a glance. Between feeds it fills over three
hours, so a half-full ring means roughly ninety minutes since the last one — it's a
reference, not a target, and there's no alarm state. During a feed it fills with one
band per side, so you watch amber hand over to green as you switch.

Tap the date at the top to name the log. The sun/moon button switches between day and
night colours; left alone it follows the clock and goes dark from 7pm.

**A feed is one session, made of segments.** Tap **Left** and the timer starts.
Tap **Right** and the left segment closes at that exact second while the right one
opens — no gap, so the log reads `L 8:34pm – 8:40pm · R 8:40pm – 8:50pm`.
**Pause** stops the clock (a nappy change mid-feed, a burp), and the paused stretch
is excluded from the total. **Finish feed** writes the whole session to the log.
Tapping the side you're already on does nothing, and a session under 3 seconds is
thrown away, so mis-taps in the dark don't pollute the log.

**Nappies** ask before they log. Tap Wet, Dirty or Both and a sheet comes up
naming what you tapped with the time — confirm with **Log it**, or **Cancel**.
The sheet also offers **Take a photo**, which opens the camera (or your library)
and attaches the shot to that entry. Tap the thumbnail in the log to view it full
screen, or delete just the photo and keep the entry.

## Editing and fixing entries

Tap any row in the log to open it — change the side, the times, the date, or the
nappy type, or delete it. Times out by a few minutes because you started the timer
late? Fix them here. Editing a feed that was L→R collapses it to a single stretch on
one side; that's the tradeoff for a simple editor.

**Add a past entry** at the bottom of the log records a feed or nappy after the fact —
for the 3am one you were too tired to log, or anything from before you installed the app.

## CSV — save to your phone, load it back

**Export CSV** downloads a `feeds-YYYY-MM-DD.csv` to your device. It opens in Excel or
Numbers, and it's also a complete, re-importable record: every entry carries a stable
id, one row per feed segment.

**Import CSV** reads such a file back in and *merges* it — entries already present are
skipped by id, so importing your own export twice does nothing, and importing a file
from another device adds only what's new. Deleted entries stay deleted. Photos are not
in the CSV.

**Save backup** is the same idea in JSON (the exact app state, including the sync link).

## Sync

Optional, off by default. See SYNC.md — a Supabase project and one paste per device.
Entries merge by id across devices rather than one overwriting the other; photos stay
on the device that took them.

## What runs when

Nothing ticks in the background. The only clock in the app is the one for a feed
happening right now, and even that stops when the screen goes away — it's restarted
from the stored timestamps when you come back, so the total is always right.

"Since last feed" is a reading, not a counter. It's worked out from the last feed's
start time whenever you open the app, return to it, or touch the screen. Left sitting
open on a table it will sit still; touch it and it catches up.

## Put it online once

The files need to be served over HTTPS one time so iOS will install them. After
that the app runs entirely from the phone.

**GitHub Pages** is the quickest:

1. New repo → upload all five files to the root (not in a folder).
2. Settings → Pages → Source: `Deploy from a branch`, branch `main`, folder `/ (root)`.
3. Wait ~1 minute, then open `https://<you>.github.io/<repo>/` on your iPhone in **Safari**
   (Chrome on iOS can't install to the home screen).

Azure Static Web Apps or any other static host works the same way.

Make the repo private if you'd rather nobody stumbles on it. Pages needs a paid plan
for private repos, so for a free private option use Cloudflare Pages or Netlify.

## Install it

In Safari: **Share → Add to Home Screen → Add**. You get a dark icon called "Feeds"
that opens full screen with no address bar.

## Where the data lives

Entries go to `localStorage`; photos go to IndexedDB (much bigger quota — photos are
downscaled to 1024px and saved as JPEG, roughly 100KB each). Both are on that phone
only. Nothing is uploaded and nothing syncs; a second phone keeps its own log.

**Copy backup** puts every entry on the clipboard as JSON and **Paste backup**
restores it. Where the clipboard is blocked, the text appears in a panel to copy by hand. Photos are not in the backup — they're too large for a clipboard round
trip. If a photo matters, save it to your camera roll as well.

iOS can clear web storage for apps left unopened for long stretches. A tracker opened
every few hours won't hit that, but paste a backup into a note now and then.

## Changing it

Everything is in `index.html` — styles at the top, logic in the one `<script>` block.
After editing, bump `CACHE` in `sw.js` (`feeds-v2` → `feeds-v3`) or the phone will
keep serving the cached copy. Force-close and reopen the app to pick up the change.
