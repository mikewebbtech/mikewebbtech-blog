---
date: 2025-08-21T00:00:00+08:00
draft: false
title: Decoupling my Content
summary: I'm exploring moving out my content from the Hugo site code  into a seperate isolated direcrory structure and git repository using git submodules. This should create a cleaner, more focused workflow with independent version histories, making it easier to manage and publish my content.
tags:
  - workflow
  - git
categories:
  - hugo
series:
  - my-hugo-site
---

# Divide and Conquer

One of my peeves with wordpress as a platform was the way it stored my content...in a database. I understand the why and that's fine and I have bared my thoughts on this subject in previous articles.

Now that I can create my content "*offline*" as text files, I would like to explore how I can take better(?) advantage of that feature.

I’ve split my blog articles into their own top level directory and own git repository, separate from my Hugo site’s main codebase. The Hugo site will consume that content at build time. This keeps the directory structure and git repo of the Hugo site generator focused on layout, styling, and configuration, while the content will lives in a cleaner, dedicated space with its own version history.

This will, hopefully and magically, work as planned by use of the [*git submodule*](https://git-scm.com/book/en/v2/Git-Tools-Submodules) feature.

Here is an overview of my new directory structure:

```
	      Parent Repo (Hugo)
  /Projects/mikewebbtech-hugo/
	├──.git/
	├── layout/
	├── assets/
	└── content/
		├── index.html
		├── about.html
		└── article --git submodule pointer-
											|
						┌───────────────────
						|
						▼
┌───────────────────────────────────────┐
│        Submodule Repo (Articles)      │
│  /Projects/mikewebbtech-blog/         │
│  ├──.git/                             │
│  ├──2008/2008-01-14-test-article.md   │
│  ├──2025/2025-05-02-article-hugo.md   │
│  ...                                  │
│                                       │
└───────────────────────────────────────┘
```

There are some advantages I would like to achieve with this setup are:

- **Separation of concerns.**  The Hugo site generator only manages the site framework, while the content stays independent. This gives me the freedom to use whatever editing tools I like Obsidian, Neovim, or anything that edits text files, without worrying about mixing them into the site code. It also means the same content repo could be consumed by other static site generators or frameworks that support Markdown.
- **Workflow focused.**  My content lives in its own repository, and Hugo simply consumes it. That separation makes it easier to manage code and content without them stepping on each other.
- **Version control.** Content and site code now have distinct histories. I’m not changing Hugo’s layout code base or configs nearly as often as I’m writing posts, so this keeps commits cleaner. It also opens the door for collaboration and corrections as others can suggest changes to the content directly via Github pull requests.
- **Publishing friendly.** The site build is lighter because Hugo doesn’t need to track content changes beyond pulling them in. Publishing (simply...hopefully) becomes a matter of syncing the repos.

Of course, this approach isn’t free of trade offs.  Adding another repository, especially as a submodule, increases workflow complexity.  I’ll need to refine my process so creating content and publishing online still feels smooth.  For example, scripting repo syncs, automating builds, or making sure Git friction doesn’t get in the way of writing and publishing.
