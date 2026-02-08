---
date: 2025-10-29T00:00:00+08:00
draft: false 
title: Configuring GitHub Pages for free Hugo host my Hugo site 
summary: How I use GitHub Pages as the free hosting solution for my Hugo site. Version controlled, highly available, and HTTPS by default.
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

# Configuring GitHub Pages for static site hosting (free)

I’m building a Hugo site because I want a fast, static site that lives in Git and doesn’t cost me a monthly hosting bill.

**GitHub Pages** fits that perfectly: it’s a static hosting service that publishes a website straight from a GitHub repository (optionally via a build process).  
Official docs: [What is GitHub Pages?](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)

Why it’s a good option for me:

- **Free hosting** for public sites (which matches my “keep it simple and cheap” requirement).
- **Great integration with my existing Git workflow** — commit, push, done.
- **Version control is the truth** — every site change is a commit you can diff and roll back.
- **Easy setup** — Pages is literally a repo setting.
- **Built-in HTTPS / SSL** (and you can enforce HTTPS in one tick box).
- **Good availability** — GitHub is built for serving content reliably.

---

## Pre-requisites

- A working Hugo site (locally).
- A GitHub repo for the site source.
- Decide how you want Pages to publish:
  - **Deploy from a branch** (simple, but you’ll be pushing generated output somewhere), or
  - **GitHub Actions** (my preferred approach: build + deploy automatically)

GitHub’s publishing-source doc covers both:  
[Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

If you’re going to use my GitHub Actions workflow, this “Pages config” is the bit that makes the hosting side work.

---

## User site vs project site (pick your URL)

GitHub Pages usually ends up looking like one of these:

- **User/Org site**: `https://username.github.io` (clean for a main website)
- **Project site**: `https://username.github.io/repository` (fine for projects)

GitHub explains the types here:  
[What is GitHub Pages?](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)

---

## Step 1 — Create (or choose) your Hugo repo

Your repo should contain your Hugo **source**, not just the generated output:

- `content/`
- `themes/`
- `static/`
- `layouts/`
- `hugo.yaml` / `config.toml`
- etc

Commit and push to `main`.

---

## Step 2 — Enable GitHub Pages in the repo settings

In your repo:

1. **Settings**
2. **Pages**
3. Under **Build and deployment**, choose how you want Pages to publish

If you haven’t done this before, GitHub’s quickstart is straightforward:  
[Quickstart for GitHub Pages](https://docs.github.com/en/pages/quickstart)

---

## Step 3 — Choose your publishing method (Branch vs Actions)

This is the key decision.

### Option A: Deploy from a branch (fastest to get online)

This is the “classic” Pages setup:

- You publish from a branch (and either `/` or `/docs`)
- That branch contains the **static site output** (HTML/CSS/JS)

GitHub docs:  
[Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

This works, but it often leads to committing a bunch of generated files (`public/`) somewhere — which I wanted to avoid long-term.

### Option B: GitHub Actions (what I use)

My preferred setup:

- Repo contains Hugo **source**
- GitHub Actions builds Hugo on a clean runner
- Actions deploys the generated output to Pages

In repo **Settings → Pages**, set the source to **GitHub Actions**.  
GitHub docs: [Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

This is the approach I reference in my workflow post, and it keeps the repo clean (no generated output commits).

---

## Step 4 — Make sure Hugo’s `baseURL` matches how you publish

This is where people quietly break their site.

- If your Pages URL is `https://username.github.io/repository/` (project site),
  your site needs to generate links with that base path.
- If your Pages URL is `https://username.github.io/` (user site),
  the base path is just `/`.

If you use GitHub’s official Pages workflow approach, the workflow can inject the Pages URL into Hugo during build (this is what I’m doing in my Actions pipeline).

Hugo’s official guide for Pages is excellent (and includes the workflow pattern):  
[Hugo: Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/)

---

## Step 5 — HTTPS (free SSL) and why it matters

GitHub Pages supports HTTPS and lets you enforce it so all HTTP requests redirect to HTTPS.

GitHub docs:  
[Securing your GitHub Pages site with HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)

This is one of the biggest wins of Pages for me: *no certificate buying, no renewals, no messing around.*

---

## Step 6 — Using a custom domain (like `mikewebb.tech`)

A custom domain is the point where your site stops looking like “a GitHub repo” and starts looking like an actual website.

I’m using `mikewebb.tech`, but the process is the same for any domain you own.

GitHub’s official guide is here:  
[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

### 6.1 — Decide: apex domain vs `www`

You’ve got two common patterns:

- **Apex (root) domain**: `mikewebb.tech`
- **Subdomain**: `www.mikewebb.tech`

You *can* support both, but pick one as your primary and redirect the other.

GitHub explains the options and DNS patterns in the custom domain docs:  
[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

### 6.2 — Set the custom domain in your repo

In your repo:

1. **Settings**
2. **Pages**
3. Under **Custom domain**, enter your domain (e.g. `mikewebb.tech`)
4. Save

This typically creates/updates a `CNAME` file in the deployed site output.

### 6.3 — Configure DNS (the part that actually matters)

This step happens at your domain registrar / DNS provider.

You have two typical DNS setups:

#### Option A — Apex domain (`mikewebb.tech`)
GitHub Pages uses **A records** pointing at GitHub’s Pages IPs (GitHub documents the current IP list, so don’t copy random blog posts).  
Use the IPs listed here:  
[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

#### Option B — `www` subdomain (`www.mikewebb.tech`)
Use a **CNAME record** pointing to your Pages hostname:

- `www` → `username.github.io`

Again, GitHub walks through this with examples:  
[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

### 6.4 — Wait for DNS to propagate (and verify it properly)

DNS changes can take a while depending on your TTL and registrar.

GitHub provides a good troubleshooting page when it doesn’t “just work”:  
[Troubleshooting custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages)

When you’re testing, check:
- Does the domain resolve to the expected target?
- Does GitHub Pages show a green check / “Domain verified” (where applicable)?
- Does the Pages settings page stop complaining?

### 6.5 — Enforce HTTPS (free SSL)

Once DNS is correct, GitHub can issue a certificate and you can enable HTTPS.

1. **Settings → Pages**
2. Tick **Enforce HTTPS**

GitHub docs:  
[Securing your GitHub Pages site with HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)

If HTTPS doesn’t become available, it’s almost always DNS-related. Go back to the troubleshooting page before you start rage-editing settings.

### 6.6 — (Optional, but smart) Domain verification

If you want extra protection against someone else attempting to use your domain on their Pages site, GitHub supports domain verification.  
Docs: [Verifying your custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)

---

## Know the limits (so you don’t accidentally abuse free hosting)

GitHub Pages is generous, but not infinite. The big ones:

- Published sites: **max 1 GB**
- Bandwidth: **soft limit 100 GB / month**
- Builds: **soft limit 10 builds / hour**  
  (and this limit does **not** apply if you build + publish with a custom GitHub Actions workflow)

GitHub docs:  
[GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)

For a Hugo blog/portfolio, these limits are basically a non-issue unless you start dumping huge binaries into `static/`.

---

## Resources (the exact docs worth bookmarking)

- [What is GitHub Pages?](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)
- [Quickstart for GitHub Pages](https://docs.github.com/en/pages/quickstart)
- [Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
- [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
- [Verifying your custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)
- [Troubleshooting custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages)
- [Securing your GitHub Pages site with HTTPS](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)
- [GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)
- [Hugo: Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/)

---

## Next up

Now that Pages is configured as the hosting layer, the next step is automating the build and deployment.

That’s where my GitHub Actions workflow comes in: one push to `main` becomes build + deploy with no local steps.
