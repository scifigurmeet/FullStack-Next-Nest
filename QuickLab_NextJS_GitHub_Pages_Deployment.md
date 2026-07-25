# ⚡ Quick Lab — Deploy a Next.js Site to GitHub Pages with GitHub Actions

> **Goal:** scaffold a Next.js project, make one small edit, and set up a pipeline that deploys it to a free public URL on every push. Time: ~20 minutes. 🚀

---

## 1️⃣ Create the Project

```bash
npx create-next-app@latest my-pages-site
```

| Prompt | Choice |
|---|---|
| TypeScript? | No |
| ESLint? | Yes |
| Tailwind CSS? | Yes |
| `src/` directory? | No |
| App Router? | **Yes** |
| Import alias? | No |

```bash
cd my-pages-site
npm run dev        # → http://localhost:3000 ✅
```

## 2️⃣ Make a Minimal Home Page

Replace `app/page.js`:

```jsx
export default function Home() {
  return (
    <main className="min-h-screen flex flex-col items-center justify-center gap-4">
      <h1 className="text-4xl font-bold">🚀 Deployed with GitHub Actions</h1>
      <p className="text-gray-600">
        Next.js → GitHub → Pages, automatically on every push.
      </p>
    </main>
  );
}
```

Save, check the browser, done. (Any edit works — the point is having something recognisably *yours* to see go live.)

## 3️⃣ Configure Next.js for Static Export

GitHub Pages serves **static files only** — no Node server. Next.js supports this via **static export**. Replace the contents of `next.config.mjs`:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: "export",              // build to static HTML in ./out
  images: { unoptimized: true }, // <Image> without the server optimizer
};

export default nextConfig;
```

> ⚠️ **What static export means:** no Route Handlers (`app/api/…`), no server-side data fetching per request, no Server Actions. Perfect for portfolios, docs, landing pages and course sites. An app that needs an API (like the workshop's Notice Board) belongs on Vercel/Render instead — right tool, right host.

## 4️⃣ Push to GitHub

Create an empty repo on **github.com** (e.g. `my-pages-site`, public), then:

```bash
git init
git add .
git commit -m "Next.js site with Pages pipeline"
git remote add origin https://github.com/YOUR-USERNAME/my-pages-site.git
git branch -M main
git push -u origin main
```

## 5️⃣ Enable GitHub Pages (one setting)

Repo → **Settings → Pages → Build and deployment → Source: “GitHub Actions”**.

That's the whole setting — no branch selection needed; the workflow below does the publishing.

## 6️⃣ The Pipeline

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Next.js to GitHub Pages

on:
  push:
    branches: [main]      # every push to main deploys
  workflow_dispatch:      # …and a manual "Run workflow" button

permissions:
  contents: read
  pages: write            # allowed to publish Pages
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - uses: actions/configure-pages@v5
        with:
          static_site_generator: next   # auto-injects the correct basePath

      - run: npm ci
      - run: npm run build              # static export → ./out

      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./out

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

**Reading it in one pass:** two jobs. `build` checks out the code, installs Node, lets `configure-pages` patch the base path (so CSS/links work under `/my-pages-site/`), builds the static site into `./out`, and uploads it as a Pages artifact. `deploy` publishes that artifact. `needs: build` chains them.

## 7️⃣ Ship It

```bash
git add . && git commit -m "Add Pages deployment pipeline" && git push
```

Open the repo's **Actions** tab → watch the two jobs run → green ✅ → the site is live at:

```
https://YOUR-USERNAME.github.io/my-pages-site/
```

From now on, **every push to `main` redeploys automatically**. Edit the heading in `app/page.js`, push, refresh the URL a minute later — that loop is the whole lesson. 🔁

---

## 🧯 Quick Troubleshooting

| Symptom | Fix |
|---|---|
| 404 at the URL | Settings → Pages → Source must be **GitHub Actions**; check the Actions run is green |
| Page loads but CSS/links broken | The `configure-pages` step with `static_site_generator: next` is missing — it injects the `basePath` |
| Build fails on `npm ci` | `package-lock.json` must be committed (create-next-app makes it — don't gitignore it) |
| Build fails mentioning API routes / server features | Static export can't include `app/api/` or server-only features — remove them, or deploy to Vercel instead |
| Old content showing | Hard-refresh (Ctrl+Shift+R); Pages caches briefly |

---

<div align="center">

**One repo · one setting · one YAML file — and a URL that updates itself.**

🌐 codelucky.com · CodeLucky FDP quick lab

</div>
