# SecAI Journal

A security & AI research blog built with [Jekyll](https://jekyllrb.com/) and the [Chirpy](https://github.com/cotes2357/jekyll-theme-chirpy) theme, deployed automatically to GitHub Pages at [secai-journal.github.io](https://secai-journal.github.io).

> **What is Jekyll?**
> Jekyll is a static site generator — it takes your Markdown files and turns them into a full website. No database, no server-side code. You write posts in plain text, push to GitHub, and the site builds itself automatically.

---

## Local Development

Running the site locally lets you preview changes before publishing.

### Prerequisites — macOS

1. **Install Homebrew** (if not already installed):
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Install rbenv** (Ruby version manager) and Ruby 3.3:
   ```bash
   brew install rbenv ruby-build
   rbenv install 3.3.0
   rbenv global 3.3.0
   ```

3. **Add rbenv to your shell** (only needed once):
   ```bash
   echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
   exec zsh
   ```

4. **Install Bundler** (Ruby's package manager):
   ```bash
   gem install bundler
   ```

### Prerequisites — Windows

1. **Install Ruby+Devkit** via [RubyInstaller](https://rubyinstaller.org/downloads/):
   - Download the latest **Ruby+Devkit** (e.g. `Ruby 3.3.x (x64)`)
   - During install, keep all default options checked
   - At the end, when asked to run `ridk install`, press Enter and let it finish

2. **Open a new Command Prompt** and verify Ruby installed:
   ```cmd
   ruby -v
   ```

3. **Install Bundler:**
   ```cmd
   gem install bundler
   ```

### Setup (macOS & Windows)

Clone the repo and install dependencies:

```bash
git clone https://github.com/SecAI-Journal/secai-journal.github.io.git
cd secai-journal.github.io
bundle install
```

### Running the site

```bash
bundle exec jekyll serve
```

Open `http://127.0.0.1:4000` in your browser. The site rebuilds automatically when you save a file.

**Useful flags:**

| Flag | Description |
|---|---|
| `--livereload` | Also auto-refreshes the browser tab on changes |
| `--drafts` | Also renders posts in `_drafts/` (unpublished work-in-progress) |

Example with both flags:
```bash
bundle exec jekyll serve --drafts --livereload
```

---

## Writing Posts

### Published posts

Create a file in `_posts/` using the naming format `YYYY-MM-DD-your-title.md`:

```
_posts/2026-03-25-htb-someboxname-writeup.md
```

Every post needs a **frontmatter block** at the top (the section between `---`):

```yaml
---
title: "HTB Writeup: SomeBox"
date: 2026-03-25 10:00:00 +0100
categories: [CTF, HackTheBox]
tags: [linux, privesc, web]
pin: false
---

Your post content here in Markdown.
```

| Field | Description |
|---|---|
| `title` | Post title shown on the site |
| `date` | Publication date and time with timezone offset |
| `categories` | Up to two levels, e.g. `[CTF, HackTheBox]` |
| `tags` | Any number of lowercase tags |
| `pin` | Set to `true` to pin the post to the top of the homepage |

### Drafts

Create files in `_drafts/` **without a date** in the filename:

```
_drafts/my-research-post.md
```

Drafts are never published to the live site. Preview them locally with:

```bash
bundle exec jekyll serve --drafts
```

When ready to publish, move the file to `_posts/` and add the date to the filename.

A template is provided at `_drafts/post-template.md`.

---

## Site Structure

```
secai-journal.github.io/
├── _posts/          # Published blog posts
├── _drafts/         # Work-in-progress posts (never published)
├── _tabs/           # Sidebar navigation pages (About, Categories, Tags, Archives)
├── assets/
│   └── img/
│       ├── avatar.png          # Sidebar profile image
│       ├── favicons/           # Browser tab icons (see below)
│       └── posts/              # Images used in posts
├── _config.yml      # Main site configuration
├── Gemfile          # Ruby dependencies
└── .github/
    └── workflows/
        └── pages-deploy.yml    # Auto-deployment to GitHub Pages
```

### `_tabs/`

Defines the sidebar navigation links. Each file maps to one item. The only file you'll regularly edit is `_tabs/about.md` to update the About page. The others (categories, tags, archives) are handled automatically by Chirpy.

### Favicon

To set the browser tab icon, generate favicon files at [realfavicongenerator.net](https://realfavicongenerator.net) and place them in `assets/img/favicons/`:

```
assets/img/favicons/
├── favicon.ico
├── favicon-16x16.png
├── favicon-32x32.png
├── apple-touch-icon.png
├── android-chrome-192x192.png
└── android-chrome-512x512.png
```

---

## Deployment

Every push to `main` triggers GitHub Actions to build and deploy the site automatically.

**First-time setup** (do this once in the GitHub repo settings):
1. Go to **Settings → Pages**
2. Set **Source** to **GitHub Actions**

**Monitor deployments:**
`https://github.com/SecAI-Journal/secai-journal.github.io/actions`

**Live site:**
`https://secai-journal.github.io`
