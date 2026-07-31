# Deploying to GitHub Pages

The whole app is static files, so there's no build step — GitHub serves the folder
as-is. Two ways in: the browser (no tools) or the command line. Pick one.

Files that must end up at the repo root:

    index.html
    sw.js
    manifest.webmanifest
    apple-touch-icon.png
    icon-192.png
    icon-512.png
    .nojekyll        (empty file — tells Pages to skip Jekyll and serve as-is)

README.md, SYNC.md and this file can come along too; they're just ignored.

---

## Option A — all in the browser

1. On github.com, click **+** (top right) → **New repository**.
   Name it something like `feeds`. Public is simplest and exposes nothing but the
   code — there are no keys in these files. Create it.

2. On the empty repo page, click **uploading an existing file**.

3. Drag in all the files listed above. `.nojekyll` is easy to miss because it looks
   empty and some file pickers hide dotfiles — if it won't drag, see the note below.
   Commit.

4. **Settings** → **Pages** (left sidebar).
   Under *Build and deployment*: Source = **Deploy from a branch**,
   Branch = **main**, folder = **/ (root)**. Click **Save**.

5. Wait ~1 minute, refresh the Pages settings page. It shows
   *Your site is live at* `https://<you>.github.io/feeds/`.

Skip to **Install on your iPhone** below.

> If `.nojekyll` won't upload: on the repo page click **Add file → Create new file**,
> type `.nojekyll` as the name, leave the body empty, and commit. In practice this
> app doesn't rely on it — no filenames start with an underscore — but it removes one
> class of surprise.

---

## Option B — command line

    # in the baby-tracker folder
    git init
    git add .
    git commit -m "Feeds tracker"
    git branch -M main
    git remote add origin https://github.com/<you>/feeds.git
    git push -u origin main

Then do step 4 above (Settings → Pages) once, in the browser.

---

## Install on your iPhone

Open `https://<you>.github.io/feeds/` **in Safari** — Chrome and other iOS browsers
can't add to the home screen.

Keep the trailing slash. The service worker and manifest use relative paths, so
`…/feeds` without the slash resolves them one level too high and the install breaks.

Then: **Share** → **Add to Home Screen** → **Add**.

You get a dark icon called **Feeds** that opens full screen, no address bar.

---

## Updating it later

Change the file, then in the **same commit** bump the cache version in `sw.js`:

    const CACHE = 'feeds-v6';   →   'feeds-v7'

Push it. Without the bump, the phone keeps serving the copy it cached and your change
won't show. After GitHub redeploys (~1 min), force-close the app and open it twice —
once for the phone to fetch the new service worker, once for it to run.

Command line:

    git add . && git commit -m "…" && git push

Browser: open the file in the repo, click the pencil, edit, commit.

---

## The private question, briefly

GitHub Pages serves from a **public** repo on the free plan. To publish from a
**private** repo you need GitHub **Pro** (US$4/mo) — everything else is identical,
you just flip the repo to private.

Either way the *site itself* is reachable by anyone with the URL; a private repo only
hides the source code and its history, not the page. For this app that costs nothing:
someone who found the URL gets an empty shell, because your entries live in your
phone's storage and your Supabase row, neither of which is on GitHub.

Want the source private for free? Keep the real repo private and have a GitHub Action
mirror the files into a small public repo that Pages serves. More machinery than six
static files really warrant — ask and I'll write the workflow.
