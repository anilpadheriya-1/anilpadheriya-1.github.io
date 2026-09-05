# Deploying the developer website (and fixing AdMob app-ads.txt)

This folder is the **developer website** for *Doodle Merge: Notebook Wars*. Its whole reason for
existing is that AdMob cannot verify `app-ads.txt` until Google Play has a developer website URL
for the app, and the file is served from the **root** of that website.

Do not upload this `DEPLOY.md` — it is notes for you, not part of the site. (Harmless if you do.)

---

## The one rule that decides whether this works

AdMob takes the website URL from your Play listing and fetches `<that-domain>/app-ads.txt`.
GitHub Pages only puts files at the domain root for a **user site**:

| Repo name | Site URL | app-ads.txt ends up at | Works? |
|---|---|---|---|
| `<your-username>.github.io` | `https://<your-username>.github.io/` | `/app-ads.txt` | **yes** |
| anything else | `https://<your-username>.github.io/<repo>/` | `/<repo>/app-ads.txt` | **no — 404** |

**Name the repository exactly `<your-github-username>.github.io`.** Lowercase. Nothing else.
This is the mistake that wastes a week.

---

## 1. Create the repo

1. Go to <https://github.com/new>.
2. Repository name: `<your-username>.github.io` (substitute your real username).
3. Visibility: **Public** (GitHub Pages needs public on the free plan).
4. Do not add a README, .gitignore or licence — keep it empty.
5. Create repository.

## 2. Publish the files

Upload the **contents** of this `site/` folder to the repo root — not the folder itself.
`app-ads.txt` must end up as the top-level `app-ads.txt` in the repo.

**Option A — browser upload (no tools needed)**

1. On the empty repo page, click **uploading an existing file**.
2. Open `site/` in Explorer, select everything inside it, and drag it into the browser.
   Explorer hides `.nojekyll`; that is fine, handle it in step 3.
3. Commit.
4. Click **Add file → Create new file**, name it `.nojekyll`, leave the body empty, commit.
   (This stops GitHub's Jekyll build from touching anything.)

**Option B — git (git 2.52 is already installed on this machine)**

```bash
cd "E:/DOODLE MERGE NOTEBOOK WARS/site"
git init -b main
git add -A
git commit -m "Developer website for Doodle Merge: Notebook Wars"
git remote add origin https://github.com/<your-username>/<your-username>.github.io.git
git push -u origin main
```

## 3. Turn on Pages

Repo → **Settings → Pages** → Source: **Deploy from a branch**, branch `main`, folder `/ (root)`,
Save. Wait for the green *"Your site is live at ..."* banner (usually under a minute, sometimes ten).

## 4. Verify the file before touching Play

```bash
curl -sS -D- https://<your-username>.github.io/app-ads.txt
```

You need **all** of these:

- `HTTP/2 200` — not 404, not a redirect chain
- `content-type: text/plain`
- body exactly: `google.com, pub-8795052891796332, DIRECT, f08c47fec0942fa0`
- first byte is `g` — no invisible BOM in front of it

A 404 almost always means the repo is named wrong. Fix that before going further.

## 5. Add the website to Google Play

Play Console → your app → **Grow → Store presence → Store listing** →
**Store listing contact details** → **Website**:

```
https://<your-username>.github.io
```

Root only. No trailing path, no `/index.html`. Save, then **send the listing for review**.
The URL is not public until that review clears, and AdMob cannot see it before then.

## 6. Tell AdMob to re-crawl

AdMob → **Apps → Doodle Merge: Notebook Wars → App settings → app-ads.txt → Check for updates**.

Expect a wait. The Play listing has to go live first (hours), and AdMob's crawl can take up to
24 hours after that; it also re-crawls on its own roughly weekly. "Not verified" immediately after
you click is normal and not a sign anything is broken.

---

## Notes

- **The privacy policy URL in Play Console is deliberately unchanged.** It still points at
  <https://sites.google.com/view/doodle-merge-notebook-wars>. The copy at `/privacy/` here is the
  same text, published so the site is complete. If you ever switch Play to this domain, update
  `store/RELEASE-STEPS.md` too and remember Play cross-checks the policy against your Data safety
  form.
- **Google Sites can never host `app-ads.txt`.** AdMob would fetch `sites.google.com/app-ads.txt`,
  which belongs to Google, not you. That is why this separate site exists.
- **If you add mediation networks later** (AppLovin, Unity Ads, ironSource…), each one gives you its
  own `app-ads.txt` line. Append them — one per line — and redeploy. Never replace the Google line.
- **Editing the file later:** keep it plain ASCII with no BOM. Windows PowerShell `Out-File` and
  `Set-Content` add a BOM by default and AdMob then fails to parse the first line. Edit it in
  VS Code (status bar should read "UTF-8", not "UTF-8 with BOM") or with `git`-friendly tooling.
