# Brian On Things

The source for [Brian on Things](https://brianobot.github.io/brian_on_things), a personal blog about backends,
low-level software and the internals of everyday tools.

It is a static site built with [Zola](https://www.getzola.org): posts are plain Markdown files, Zola turns them
into HTML, and GitHub Actions publishes the result to GitHub Pages on every push to `master`. There is no
JavaScript framework, no database and no server to run. The theme, templates and styles are all hand-written
and live in this repository.

If you want a blog like this for yourself, skip to [Make it your own](#make-it-your-own).

---

## Contents

- [Features](#features)
- [Quick start](#quick-start)
- [Project layout](#project-layout)
- [Writing a post](#writing-a-post)
- [Editing the other pages](#editing-the-other-pages)
- [Configuration reference](#configuration-reference)
- [Styling](#styling)
- [Deployment](#deployment)
- [Make it your own](#make-it-your-own)
- [Troubleshooting](#troubleshooting)

---

## Features

- **Home page** with avatar (or generated initials), tagline, bio, social links, recent posts, education and
  skills, all driven from `zola.toml`.
- **Archive** of every post, grouped by year with a per-year count.
- **Post pages** with publish date, reading time, an optional table of contents, a description pull-quote, and
  a citation card with a "Copy link" button, and a feedback box inviting readers to reach you by email or on
  your socials.
- **Projects page** with featured work, "learning in public" repo groups and a GitHub contribution chart, all
  driven from that page's front matter.
- **Client-side search** over every post, using the index Zola builds (`elasticlunr`).
- **Light / dark / system theme** toggle, remembered per visitor, with matching syntax-highlighting themes
  (Catppuccin Latte and Mocha) and no flash of the wrong theme on load.
- **Reading-progress indicator** that turns into a back-to-top button.
- **SEO and social metadata**: description, canonical URL, Open Graph and Twitter card tags, plus
  `article:*` tags on posts.

---

## Quick start

### 1. Install Zola

The site is built and deployed with **Zola 0.23.4** (see `.github/workflows/deploy.yml`). Any 0.23.x release
works locally.

```sh
# macOS
brew install zola

# Windows
winget install getzola.zola     # or: scoop install zola / choco install zola

# Linux: use your package manager, or grab a binary from
# https://github.com/getzola/zola/releases
```

Check it worked:

```sh
zola --version
```

### 2. Clone the repository

```sh
git clone https://github.com/brianobot/brian_on_things.git
cd brian_on_things
```

The repository also contains a git submodule at `themes/serene`. **It is not used** (the site has no
`theme = ...` key, so Zola ignores `themes/` entirely). You do not need to initialise it.

### 3. Run it locally

```sh
zola serve            # or: make run-local  (same thing, and opens your browser)
```

Open <http://127.0.0.1:1111>. The server rebuilds and reloads the page whenever you save a file.

### Other commands

| Command        | What it does                                                                 |
| -------------- | ---------------------------------------------------------------------------- |
| `zola serve`   | Dev server with live reload at `http://127.0.0.1:1111`.                      |
| `zola build`   | Renders the whole site into `public/`. This is what gets deployed.           |
| `zola check`   | Builds without writing output and checks every external link. Run before pushing. |

There is no test suite; `zola check` is the closest thing to CI.

---

## Project layout

Zola works by convention: where a file lives decides what URL it gets and how it is rendered.

```
.
├── zola.toml                 # Site settings + everything under [extra] the templates read
├── content/                  # All writing, in Markdown
│   ├── blog/
│   │   ├── _index.md         # The /blog section: uses archive.html, posts use post.html
│   │   └── <post-slug>/
│   │       ├── index.md      # A post
│   │       └── *.png         # Images used by that post sit next to it
│   ├── about/index.md        # /about
│   ├── projects/index.md     # /projects (content lives in its front matter)
│   └── search/index.md       # /search
├── templates/                # Tera templates (Zola's templating language)
│   ├── _base.html            # Shared shell: <head>, SEO tags, header/nav, footer, theme toggle
│   ├── index.html            # Home page
│   ├── archive.html          # /blog listing, grouped by year
│   ├── post.html             # A single blog post
│   ├── page.html             # Generic page (About)
│   ├── projects.html         # /projects
│   ├── search.html           # /search and its JavaScript
│   └── _*.html               # Small partials: table of contents, icons, arrows
├── sass/style.scss           # All styling; compiled to /style.css
├── static/                   # Copied as-is to the site root (favicon, avatar)
├── .github/workflows/deploy.yml  # Build + publish to GitHub Pages
├── Makefile                  # `make run-local`
└── themes/serene/            # Unused git submodule (see above)
```

`public/` is build output. Do not edit it.

---

## Writing a post

Each post is a folder under `content/blog/` containing an `index.md`. The folder name becomes the URL:
`content/blog/my-first-post/index.md` is served at `/blog/my-first-post/`.

```sh
mkdir content/blog/my-first-post
$EDITOR content/blog/my-first-post/index.md
```

Start the file with TOML front matter between `+++` lines, then write Markdown:

````markdown
+++
title = "My First Post"
date = 2026-10-30
description = "One sentence that shows under the title, in search results and in link previews."
draft = false

[extra]
toc = true      # show a table of contents built from the ## and ### headings
+++

Intro paragraph.

## A heading

Some text, a [link](https://www.getzola.org), and code:

```python
print("hello")
```
````

### Front matter fields

| Field         | Required | Notes                                                                                       |
| ------------- | -------- | ------------------------------------------------------------------------------------------- |
| `title`       | yes      | Page title, `<title>` tag and social card title.                                            |
| `date`        | yes      | `YYYY-MM-DD`. Sorts the archive and the home page. Posts without a date will not sort properly. |
| `description` | no       | Shown as a pull-quote under the title and used for `<meta name="description">`. Falls back to the site description. |
| `draft`       | no       | `true` hides the post from `zola build` (it still shows under `zola serve`).                |
| `updated`     | no       | A later date; emitted as `article:modified_time`.                                          |
| `[extra] toc` | no       | `true` renders the sidebar table of contents.                                               |
| `[extra] feedback` | no  | Every post ends with a "Thoughts on this post?" box linking to your `[[extra.socials]]`; its email link pre-fills the subject with the post title. Set `false` to hide it on this post. |
| `[extra] og_image` | no  | Filename of an image in the post's folder, used as the link-preview image on social media. Falls back to `[extra].og_image` in `zola.toml`. See [Social preview images](#social-preview-images). |

### Images

Put images in the post's folder and reference them by filename:

```markdown
![Request flow diagram](request-flow.png)
```

Only `.png`, `.jpeg` and `.webp` are committed by default; see [Troubleshooting](#troubleshooting) before
using `.jpg`, `.gif` or `.svg`.

### Social preview images

To give a post its own preview image on X, LinkedIn, Facebook, Slack and similar, put the image in the post's
folder and name it in the front matter:

```
content/blog/my-first-post/
├── index.md
└── cover.png
```

```toml
[extra]
og_image = "cover.png"
```

`templates/_base.html` turns this into absolute `og:image` and `twitter:image` tags and switches the X card to
`summary_large_image`. Posts without their own image fall back to `[extra].og_image` in `zola.toml` (a file in
`static/`). With neither set, no image tag is emitted.

- Use **1200 × 630 px**, as **PNG or JPEG**. Some platforms, LinkedIn in particular, do not reliably show WebP.
- Platforms cache previews. After deploying, refresh them with LinkedIn's
  [Post Inspector](https://www.linkedin.com/post-inspector/) or the
  [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/).

### Publishing

Commit and push to `master`. The site rebuilds and goes live in a minute or two.

---

## Editing the other pages

- **About** (`content/about/index.md`): ordinary Markdown with `template = "page.html"`.
- **Projects** (`content/projects/index.md`): the page body is empty; everything is front matter.
  - `github_user`: used for the contribution chart and to build repo links.
  - `[[extra.featured]]`: one block per featured project, with `name`, `description`, and either `url`, `repo`
    (`owner/name`), or both.
  - `[[extra.learning]]`: a group with `id`, `category`, `blurb` and a `repos` list of
    `{ name, description }`. `name` is a repository under `github_user`.
- **Home page** (`templates/index.html`): has no Markdown file. Everything on it comes from `[extra]` in
  `zola.toml`.
- **Navigation bar**: the links are a hard-coded list near the top of `templates/_base.html`:

  ```jinja
  {% set nav = [
      ["Archives",  "/blog"],
      ["Projects", "/projects"],
      ["About",    "/about"]
  ] %}
  ```

  To add a page, create `content/<name>/index.md` and add a `["Label", "/<name>"]` entry here.

---

## Configuration reference

Everything lives in `zola.toml`.

### Site settings

| Key                  | Purpose                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------- |
| `base_url`           | The full public URL, including any sub-path. All links are generated from it.              |
| `title`              | Site name in the header, page titles and social cards.                                     |
| `description`        | Default meta description when a page has none.                                             |
| `author`             | `<meta name="author">` and `article:author`.                                               |
| `compile_sass`       | Must stay `true`; that is how `sass/style.scss` becomes `style.css`.                        |
| `build_search_index` | Must stay `true`; the search page reads the generated `search_index.en.js`.                 |
| `[markdown.highlighting]` | `style = "class"` plus `light_theme` / `dark_theme`. Zola emits `giallo-light.css` and `giallo-dark.css`, and the theme toggle switches between them. Any theme name Zola supports works. |

### `[extra]` (read by the templates)

| Key                     | Used on     | Purpose                                                                 |
| ----------------------- | ----------- | ----------------------------------------------------------------------- |
| `author`                | everywhere  | Name in the home page heading, footer and post citation card.          |
| `tagline`               | home        | One line under your name.                                               |
| `bio`                   | home        | A short paragraph (triple-quoted string).                               |
| `avatar`                | home        | Filename of an image in `static/`. Leave empty (`""`) to show your initials instead. |
| `recent_posts`          | home        | How many posts the home page lists.                                     |
| `og_locale`             | everywhere  | Open Graph locale, e.g. `en_US`.                                        |
| `og_image`              | everywhere  | Default social preview image, a filename in `static/`. Optional; a post's own `og_image` overrides it. |
| `[[extra.socials]]`     | home        | `name`, `url`, `icon`. `icon` must be `github`, `linkedin`, `twitter` or `email`. |
| `[[extra.education]]`   | home        | `school`, `degree`, `note`. Remove all entries to hide the section.     |
| `[[extra.skills]]`      | home        | `category` and an `items` list. Remove all entries to hide the section. |

---

## Styling

All CSS is in `sass/style.scss`. Colours are defined once as CSS custom properties at the top of the file, as
HSL triplets: a `:root` block for light mode, followed by an override block for dark mode.

```scss
:root {
    --background: 210 33% 99%;
    --primary: 200 29% 45%;     // links and accents
    --highlight-color: #659eb9;
    ...
}
```

Changing `--primary` in both blocks is the quickest way to re-colour the whole site. The body font is Inter,
loaded from Google Fonts in `templates/_base.html`.

---

## Deployment

`.github/workflows/deploy.yml` runs on every push to `master`:

1. **build**: checks out the repo and runs the official
   [`getzola/github-pages`](https://github.com/getzola/github-pages) action with Zola 0.23.4, which builds the
   site and uploads it as a Pages artifact.
2. **deploy**: publishes that artifact with `actions/deploy-pages`.

One-time setup in the GitHub repository: **Settings → Pages → Build and deployment → Source: GitHub Actions**.

Without that setting the workflow builds successfully but nothing is published.

### Custom domain

1. Add the domain under **Settings → Pages → Custom domain**, and create the DNS records GitHub shows you.
2. Change `base_url` in `zola.toml` to the new address, e.g. `https://example.com`.
3. Add a file `static/CNAME` containing only the domain, so it survives each deploy, and allow it in
   `.gitignore` (add `!CNAME`).

### Hosting somewhere else

The output of `zola build` is a plain folder of static files, so any static host works (Netlify, Cloudflare
Pages, Vercel, an S3 bucket, a VPS running nginx). Use `zola build` as the build command and `public` as the
output directory, and set `base_url` to the URL the site will be served from.

---

## Make it your own

You are welcome to fork this repository and use it as the starting point for your own blog. Please replace the
personal content (the posts, the About page, the avatar and the details in `zola.toml`) with your own rather
than republishing it.

### Step 1: Fork and run it

1. Click **Fork** on GitHub. If you want the URL to be `https://<you>.github.io/` with no sub-path, name the
   fork `<you>.github.io`. Any other name gives you `https://<you>.github.io/<repo-name>/`.
2. Clone your fork and run `zola serve`, as in [Quick start](#quick-start). Make sure it works before
   changing anything.

### Step 2: Change the identity in `zola.toml`

- `base_url`: your final URL, e.g. `https://<you>.github.io` or `https://<you>.github.io/<repo-name>`.
  This matters: with the wrong value, CSS and links break on the live site even if they work locally.
- `title`, `description` and `author`.
- Under `[extra]`: `author`, `tagline`, `bio`, `avatar`, the `socials`, `education` and `skills` blocks.
  Delete any block you do not want; its section disappears from the home page.

### Step 3: Replace the images

- `static/favicon.webp`: your browser-tab icon. Keep the filename, or change it in `templates/_base.html`.
- `static/baby_zoe.jpeg`: the avatar. Add your own image to `static/` and point `[extra].avatar` at it, or set
  `avatar = ""` to show your initials.

### Step 4: Replace the content

- Delete the post folders in `content/blog/` (keep `content/blog/_index.md`) and write your first post.
- Rewrite `content/about/index.md`. It also has a hard-coded `mailto:` link to update.
- Rewrite `content/projects/index.md`: set `github_user` and replace the `featured` and `learning` blocks,
  or delete the page and its entry in the nav bar.

### Step 5: Sweep for leftover names

A few fallbacks still name the original author. Search for them and change what you find:

```sh
grep -rn "Brian\|brianobot" zola.toml content templates static
```

The footer in `templates/_base.html`, for example, falls back to `'Brian Obot'` when `[extra].author` is not
set.

### Step 6: Make it look like yours (optional)

- Change the colours at the top of `sass/style.scss` (see [Styling](#styling)).
- Change `light_theme` / `dark_theme` under `[markdown.highlighting]` for different code colours.
- Swap the Inter font link in `templates/_base.html` and the `font-family` in `style.scss`.

### Step 7: Tidy up the extras (optional)

These are specific to how this blog is maintained and can be removed from your fork:

- `themes/serene`: the unused submodule. Remove it with `git rm themes/serene`, then delete `.gitmodules`.
- `auto_push.sh`: a personal helper that runs `git add -A`, commits with the text of `current_feature.md`
  and pushes **every 30 seconds** in a loop. Do not run it unless that is exactly what you want.
- `indea.md`: the original planning notes.

### Step 8: Publish

1. In your fork, go to **Settings → Pages** and set **Source** to **GitHub Actions**.
2. Commit and push to `master`. Watch the run under the **Actions** tab.
3. Open the URL the deploy job prints. Done.

If your default branch is `main` instead of `master`, change `branches:` in `.github/workflows/deploy.yml`
to match.

### Starting from nothing instead

If you prefer to build your own from scratch rather than fork:

```sh
zola init my-blog        # answers a few questions and creates the skeleton
cd my-blog
zola serve
```

Then either write templates yourself (the files in `templates/` here are a readable reference), or pick a
ready-made theme from <https://www.getzola.org/themes/>, add it under `themes/` and set `theme = "<name>"` in
your config. The [Zola documentation](https://www.getzola.org/documentation/getting-started/overview/) covers
both routes.

---

## Troubleshooting

**A new file is not being committed.**
`.gitignore` ignores everything by default and only allows specific extensions: `.md`, `.toml`, `.html`,
`.scss`, `.png`, `.jpeg`, `.webp`, `.yml`, `.js`, `.xml`, plus `Makefile`. A `.jpg`, `.gif`, `.svg`, `.css`,
`CNAME` or font file is silently ignored. Add a `!*.jpg` (or similar) line to `.gitignore`, or convert the file
to an allowed format.

**The live site has no styling, or links point to the wrong place, but local works.**
`base_url` is wrong. It must match the public URL exactly, including the `/<repo-name>` sub-path on a project
Pages site.

**The workflow succeeds but the site does not update.**
Check that **Settings → Pages → Source** is set to **GitHub Actions**, and that you pushed to the branch named
in `deploy.yml`.

**Search returns nothing.**
`build_search_index` must be `true` in `zola.toml`. Draft posts are not indexed in production builds.

**A post does not show up.**
Check it has `draft = false` (or no `draft` key) and a valid `date`, and that the file is named `index.md`
inside its own folder under `content/blog/`.

**Tags do not appear on posts.**
The post template can display tags, but no `taxonomies` are configured in `zola.toml` yet. To enable them, add
`taxonomies = [{ name = "tags" }]` to `zola.toml`, add `[taxonomies] tags = ["rust"]` to a post's front matter,
and create `templates/tags/list.html` and `templates/tags/single.html` for the tag pages.

**`zola check` reports a broken external link.**
It checks every outbound URL. Fix or remove the link; some sites block automated checks, in which case add the
domain to `[link_checker] skip_prefixes` in `zola.toml`.

---

Built with [Zola](https://www.getzola.org).
