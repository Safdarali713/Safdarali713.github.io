# Blog CMS Setup Guide — for Android

## Be honest about one thing first

Decap CMS on GitHub Pages **cannot** authenticate with only static HTML — GitHub's login flow needs a small server to talk to. This is **not a paid server**: the free, official-recommended fix is a **Cloudflare Worker** (Cloudflare's free tier, no credit card required for this). It needs a **one-time setup using a terminal**, which is not possible directly from a phone browser. The easiest free way to get a terminal on Android is **GitHub Codespaces**, which runs in your phone's browser — no app install needed. Steps 4A below use that.

Everything else (the blog, the admin panel UI, writing articles, images, SEO, sitemap) is 100% free and works entirely through GitHub Pages + your phone browser.

---

## Step 1: Add the files to your repository

1. Open **github.com** in Chrome on your phone and log in.
2. Go to your repository: `safdarali713/safdarali713.github.io`.
3. For each file in this package, tap **Add file → Create new file**, type the exact path shown (e.g. `_config.yml`, `blog/index.html`, `admin/config.yml`), paste the contents, and tap **Commit changes**.
4. For the one binary file, `images/blog/default-cover.jpg`, use **Add file → Upload files** instead of "Create new file", and upload it into the `images/blog/` folder.
5. `index.html` already exists in your repo — open it, tap the pencil (edit) icon, and just add this one line to the navigation list (don't change anything else):
   ```html
   <li><a href="/blog/">Blog</a></li>
   ```

**File list to create:**
```
/
├── index.html              (EDIT — add one Blog nav link only)
├── _config.yml             (NEW)
├── robots.txt              (NEW)
├── _layouts/
│   └── post.html            (NEW)
├── _posts/
│   └── 2026-09-02-welcome-to-my-blog.md   (NEW — sample article)
├── blog/
│   └── index.html            (NEW)
├── assets/
│   └── blog.css              (NEW)
├── admin/
│   ├── index.html            (NEW)
│   └── config.yml            (NEW — you'll edit one line in Step 4C)
└── images/
    └── blog/
        └── default-cover.jpg (NEW — upload as a file, not text)
```

## Step 2: Enable GitHub Pages (if not already on)

1. In your repo, tap **Settings → Pages**.
2. Under "Build and deployment", Source should be **Deploy from a branch**, Branch: **main**, folder **/ (root)**.
3. Save. Wait 1–2 minutes.
4. Visit `https://safdarali713.github.io/blog/` — you should see the sample article.

## Step 3: Create a GitHub OAuth App

1. On github.com, go to **Settings (your account, not the repo) → Developer settings → OAuth Apps → New OAuth App**.
2. **Application name:** anything, e.g. "My Blog CMS"
3. **Homepage URL:** you'll fill this in Step 4B once you have your Worker URL — you can come back and edit it.
4. **Authorization callback URL:** same — you'll come back and fill in `https://YOUR-WORKER-URL/callback` after Step 4.
5. Click **Register application**, then **Generate a new client secret**. Copy the **Client ID** and **Client Secret** somewhere safe — you'll need both in Step 4.

## Step 4: Deploy the free OAuth proxy (Cloudflare Worker)

### 4A. Open a free browser terminal (GitHub Codespaces)

1. Go to `https://github.com/sterlingwes/decap-proxy`.
2. Tap the green **Code** button → **Codespaces** tab → **Create codespace on main**. This opens a full code editor + terminal in your phone's browser (free tier: 60 hrs/month, no card needed).
3. In the Codespace, open the **Terminal** panel (menu icon → Terminal → New Terminal).

### 4B. Deploy the Worker

Run these commands one at a time in the Codespace terminal:

```bash
cp wrangler.toml.sample wrangler.toml
npx wrangler login
```
This gives you a link — open it, log into (or sign up for, free) Cloudflare, and authorize.

```bash
npx wrangler secret put GITHUB_OAUTH_ID
```
Paste the **Client ID** from Step 3 when prompted.

```bash
npx wrangler secret put GITHUB_OAUTH_SECRET
```
Paste the **Client Secret** from Step 3 when prompted.

```bash
npx wrangler deploy
```
This prints your live Worker URL, e.g. `https://decap-proxy.YOUR-NAME.workers.dev`. **Copy this URL.**

### 4C. Wire everything together

1. Go back to your GitHub OAuth App (Step 3) and edit:
   - **Homepage URL** → your Worker URL
   - **Authorization callback URL** → your Worker URL + `/callback`
   (e.g. `https://decap-proxy.YOUR-NAME.workers.dev/callback`)
2. Edit `admin/config.yml` in your repo and replace:
   ```yaml
   base_url: https://YOUR-PROXY-URL.workers.dev
   ```
   with your real Worker URL. Commit the change.

## Step 5: Nothing else to enable

GitHub Pages already auto-builds Jekyll (which powers your blog, sitemap, and SEO tags) — no extra settings needed.

## Step 6: Wait for deployment

Give it 1–2 minutes after each commit for GitHub Pages to rebuild.

## Step 7: Open the admin panel

Visit `https://safdarali713.github.io/admin/` on your phone.

## Step 8: Log in

Tap **Login with GitHub**, authorize the OAuth app. You're in the dashboard.

## Step 9: Create your first article

Tap **Blog Articles → New Article**, fill in Title, Slug, Image, Category, Tags, Excerpt, SEO fields, and write your content. Use the **Preview** tab to see how it'll look.

## Step 10: Publish

Tap **Save** (creates a draft PR), then when ready tap **Set status → Ready** and finally **Publish** — this merges the change on GitHub, and your live site rebuilds automatically within about a minute.

---

## What's automatic for every article
- SEO `<title>`, meta description, canonical URL, Open Graph tags — generated from your Title/Excerpt/SEO fields via `jekyll-seo-tag`
- `sitemap.xml` — generated automatically by `jekyll-sitemap`, listed in `robots.txt`
- Clean URLs like `/blog/your-slug/`

## Limitations, stated plainly
- **Live "deploy preview" links** (a real staged URL before publishing) are a Netlify-only feature. What you get instead is Decap's built-in in-editor preview pane (styled to match your site) plus GitHub's own PR diff view — good enough to review changes, just not a separate live URL.
- The OAuth Worker deploy in Step 4 is a **one-time** setup. After that, everything else — including writing articles from your phone — needs no terminal at all.
