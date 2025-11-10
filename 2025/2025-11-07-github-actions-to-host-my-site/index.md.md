---
date: 2025-11-07T00:00:00+08:00
draft: false
title: GitHub actions to host my Hugo site
summary: How I use GitHub Actions to build and deploy my Hugo site to GitHub Pages.  This is my first real CI/CD pipeline.
tags:
  - blog
  - git
  - ci-cd
categories:
  - hugo
  - automation
series:
  - my-hugo-site
---

# Using GitHub Actions for web hosting

This has been on my to-do list for a long time: stop building my Hugo site manually and let an automated pipeline do it for me.

The goal is simple: every time I push changes to my Hugo repo, I want:
- the site to be built,
- the static files to be published,
- and the whole lot to end up on GitHub Pages without me touching anything.

That’s **Continuous Integration / Continuous Deployment (CI/CD)** in plain language.

GitHub already gives us everything we need for this:
- **GitHub Actions** – a native CI/CD platform that runs workflows on each push, or on demand.
- **GitHub Pages** – static site hosting directly from a repo, free for public sites.

This article walks through the workflow file I’m using, what problems it solves, and how it becomes my first “real” CI/CD pipeline for a Hugo site.

### Pre-requisites

Have already configured GitHub pages for the repo.  See my article on [configuring GitHub Pages](2025-10-29-github-repo-setup) for static site hosting

GitHub has this well documented. Here is the official Pages page [to get you started](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)

Previously my site was being *deployed* from a *branch* (and from the *public/* directory.  But I am now going to *build* and deploy to Pages using an *action* so some settings need to be changed.  

The official Hugo "Host and Deploy" documentation for [Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/#customize-the-workflow) covers this and is easy to follow (has pictures).

The workflow file needs to be a yaml (any name works as long as it ends in .yml or .yaml) file saved in the /.github/workflows/  directory of the repo root for GitHub Actions to automatically pick it up and run with it.

My actions workflow is here [mikewebbtech/mikewebbtech-hugo/.github/workflows/site-deploy.yml](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/.github/workflows/site-deploy.yml)

### Resources

I highly recommend reading the following links to better understand GitHub Actions and Pages:

[deploy-pages 🚀  repo](https://github.com/actions/deploy-pages)
[Using custom workflows with GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages)
[Choosing and using a workflow template](https://docs.github.com/en/actions/how-tos/write-workflows/use-workflow-templates#choosing-and-using-a-workflow-template)](https://docs.github.com/en/actions/how-tos/write-workflows/use-workflow-templates)

---

## What problem am I solving?

Before this workflow, a deployment looked roughly like this:

1. In my content repo after creating new content:
    - Commit my changes to the main branch.
    - push to remote repo.
2. Sync my remote content repo submodule to local.
3. Commit the submodule pointer update to the main branch
4. Run `hugo` locally.
5. Hope my Hugo version matched whatever I used last time.
6. Check the contents of `public/` folder have updated.
7. Commit the generated `public/` folder
8. Repeat the same boring steps every time I changed a post or changed my hugo files.
9. Regret life choices and vow to automate.

Problems with that approach:
- **Manual = fragile** – easy to forget a step, or build with the wrong version.
- **No single source of truth** – “What version of Hugo did I use last time?” Good question.
- **Tied to one machine** – if I’m away from my main laptop, I can’t easily deploy.

The new approach:
- I push changes to `main`.
- GitHub Actions syncs to content submodule to the latest commit.
- Spins up a fresh Linux runner, installs Hugo, and builds the site.
- The generated site is uploaded and deployed to **GitHub Pages** automatically.

No more local build steps. No more “oh yeah, I still need to deploy this".  Just commit the main branch as apart of my version control workflow. 🪄 magic. 


---

## High-level view of the workflow

Here’s the workflow file (simplified description, not the full YAML):
- Triggered on:
    - push to `main`
      - manual run (`workflow_dispatch`)
      - `repository_dispatch` (for external triggers, e.g. a submodule update)
- Build job:
      - install Hugo and Dart Sass  
      - checkout repo (including submodules)
      - optionally update submodules to latest remote
      - configure GitHub Pages
      - install Node dependencies (if present)
      - run `hugo` to build the site
      - upload `public/` as an artifact
- Deploy job:
      - takes the built artifact
      - deploys it to GitHub Pages

---

## Let’s go through it section by section.

### Triggers: when the pipeline runs

```yaml
on:
  push:
    branches: ["main"]
  workflow_dispatch:
  repository_dispatch:
    types: ["submodule_event"]
````

This gives me three ways to deploy:

- **`push` to `main`** – the normal case. I commit to `main`, the workflow runs.  
- **`workflow_dispatch`** – I can manually trigger a run from the GitHub UI (useful if Pages had an issue or I changed workflow logic).
- **`repository_dispatch`** – this allows an **external repo** (for example, a content repo used as a Git submodule) to trigger this workflow when it changes.

That last one matters because I’m using Git submodules (e.g. separate repo for articles). If that content repo updates, I can notify the main site repo via `repository_dispatch` so it rebuilds and redeploys.

---

### Permissions and concurrency: safe by default

```yaml
permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false
```

Two important things here:

1. **Minimal permissions**
    - `contents: read` – the workflow only needs to read the repo content.
    - `pages: write` – needed to publish to GitHub Pages.
    - `id-token: write` – used internally for OIDC authentication in the Pages deployment process.
    
This avoids giving the workflow full admin rights in the repo. Least privilege is always a good habit.
    
2. **Concurrency control**
    - Only one deploy to `pages` runs at a time.
    - `cancel-in-progress: false` means an in-progress deploy is allowed to finish, so I don’t end up with half-deployed production.

If I go on a commit spree, I don’t want multiple overlapping deployments fighting each other. This keeps things sane.

---

### Defaults: bash everywhere

```yaml
defaults:
  run:
    shell: bash
```

This just sets the default shell for all `run` steps. Nothing fancy, but it keeps scripts consistent.

---

### The build job: from source to static site

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.151.2
```

The `build` job runs on `ubuntu-latest` and pins a **specific Hugo version** using an environment variable. Pinning versions is critical – you don’t want your site build changing behaviour randomly because Hugo moved on.

### Install Hugo CLI

```yaml
- name: Install Hugo CLI
  run: |
    wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
    && sudo dpkg -i ${{ runner.temp }}/hugo.deb
```

This:
- downloads the **extended** Hugo `.deb` for the version I specify,
- installs it on the runner.

Extended Hugo is required for SCSS/SASS pipelines that many themes use.

### Install Dart Sass

```yaml
- name: Install Dart Sass
  run: sudo snap install dart-sass
```

I am using  Dart Sass for some of my CSS ( well in the process of refactoring all of my CSS to Sass ). Instead of assuming the runner has it, I install it explicitly. Again, this is about reproducibility: **build everything from scratch in a clean environment**.

### Checkout the repository (with submodules)

```yaml
- name: Checkout
  uses: actions/checkout@v4
  with:
    submodules: recursive
```

This pulls down:
- the main Hugo site repo,
- any Git submodules ( my articles ), recursively.

All of my content lives in a separate repo and is referenced as a submodule, this is where it gets pulled in.

###  Update submodules to latest remote

```yaml
- name: Update submodules to latest remote (optional)
  if: ${{ hashFiles('.gitmodules') != '' }}
  run: |
    git submodule sync --recursive
    git submodule update --init --recursive --remote
```

If `.gitmodules` exists, this step:

- syncs submodule configuration, and
- updates all submodules to the **latest commit on the tracked remote branch**.

This is useful as I treat submodules like “live dependencies” (e.g.  separate content repo that I want to track at head) rather than pinning them to a specific commit. This fits how I’m managing my content as an external repos.

### Configure GitHub Pages

```yaml
- name: Setup Pages
  id: pages
  uses: actions/configure-pages@v5
```

This step:
- prepares the environment for deploying to Pages,
- and gives us a `base_url` output that we can feed into Hugo so all URLs are correct.

### Install Node.js dependencies (if any)

```yaml
- name: Install Node.js dependencies
  run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
```

I am not using and Node tooling but I may in the future.

This conditional:
- checks for `package-lock.json` or `npm-shrinkwrap.json`,
- if present, runs `npm ci` for a clean install,
- otherwise does nothing (`|| true` makes the command succeed regardless). At the moment...nothing

This would lest the workflow work for both a “pure Hugo” sites and “Hugo + Node toolchain” sites.

### Build the site with Hugo

```yaml
- name: Build with Hugo
  env:
    HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
    HUGO_ENVIRONMENT: production
  run: |
    hugo \
      --minify \
      --baseURL "${{ steps.pages.outputs.base_url }}/"
```

This is the actual **Continuous Integration** part for the site:

- `HUGO_ENVIRONMENT=production` – so config and templates can behave differently for prod vs dev.
- `HUGO_CACHEDIR` points Hugo’s cache to a temp location on the runner.
- `--minify` – compresses HTML/CSS/JS. (I am using minify)
- `--baseURL` – set from the `configure-pages` step, so all absolute URLs match the final GitHub Pages URL (the importance of this can not be over stated)

The output ends up in `./public`, as usual for Hugo.

### Upload the build artifact

```yaml
- name: Upload artifact
  uses: actions/upload-pages-artifact@v3
  with:
    path: ./public
```

Everything from `public/` is uploaded as an artifact named and formatted for consumption by the GitHub Pages deployment action.

This is the boundary between **build** and **deploy** stages.

---

## The deploy job: pushing to GitHub Pages

```yaml
deploy:
  environment:
    name: github-pages
    url: ${{ steps.deployment.outputs.page_url }}
  runs-on: ubuntu-latest
  needs: build
  steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4
```

Key points:
- `needs: build` – the deploy job only runs if the build job succeeds. No broken builds pushed to production.
- `environment: github-pages` – ties into GitHub’s environment system (nice for audit/history, and for seeing the deployed URL directly).  
- `actions/deploy-pages@v4` – takes the uploaded artifact from the build job and publishes it to GitHub Pages.

At this point, the new version of the site is live.

---

## Yes this is a CI/CD pipeline

Even though this is “just” a static site, it’s still a proper **CI/CD pipeline**:
- **Continuous Integration (CI)**:
    - On every push to `main`, the workflow:
        - recreates the environment from scratch,
        - installs dependencies (Hugo, Dart Sass, Node dependencies),
        - builds the site.
    - If anything breaks (bad config, missing files), the build fails, and I see it in the Actions logs.
- **Continuous Deployment (CD)**:
    - If the build passes, the site is automatically deployed to GitHub Pages.
    - No manual “deploy” step, no copying files, no local scripts.

Did I mention  for web hosting, this whole thing is **free** for a public repo, which was one of my core requirements when I chose Hugo and GitHub Pages.

---

## Taking this to the next level

This workflow is a solid starting point, but there’s plenty of room to grow it as a more advanced CI/CD pipeline:
- add link checking or HTML validation steps before deployment,
- run automated tests on custom shortcodes or scripts,  
- build preview environments on pull requests (PR-based deployments),
- add caching or matrix builds for more complex setups.

For now, though, it solves the key pain points:
- **one push = build + deploy**
- **consistent, reproducible builds**
- **no need to build locally**

Exactly what I wanted from my first CI/CD pipeline for this Hugo site.
