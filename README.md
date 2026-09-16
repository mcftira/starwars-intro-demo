# Star Wars Intro Demo — OpenShift Lightspeed Platform Fórum

A [Slidev](https://sli.dev/) presentation with a Star Wars opening crawl, built with a
custom Prezi-style canvas.

**Live:** https://mcftira.github.io/starwars-intro-demo/

## Repository layout

| Path | Contents |
| --- | --- |
| `/` (root) | Prebuilt static site served by GitHub Pages (`slidev build --base ./`) |
| `/src` | Slidev source: `slides.md`, Vue components, styles, fonts, public assets |

The root of this repo *is* the deployed site. GitHub Pages serves it from the `main`
branch, root folder — no build step runs in CI.

## Why this works on GitHub Pages

- Assets are built with a **relative base** (`--base ./`), so they resolve under the
  `/starwars-intro-demo/` project path.
- The deck uses **`routerMode: hash`**, so no server-side SPA rewrite rules are needed.

## Media note

`video/intro.mp4` is **not** included (see `video/README.txt`). When the video file is
absent the intro automatically falls back to the bundled `audio/fanfare.mp3`, so the
deck plays correctly as deployed. To use the original video, drop it in as
`video/intro.mp4`, rebuild, and re-publish.

## Local development

```bash
cd src
npm install
npm run dev      # http://localhost:3030
npm run build    # outputs to src/dist
```

## Re-publishing after a rebuild

```bash
cd src && npm run build
rsync -a --delete --exclude src --exclude README.md --exclude .git dist/ ../
cd .. && git add -A && git commit -m "Rebuild site" && git push
```
