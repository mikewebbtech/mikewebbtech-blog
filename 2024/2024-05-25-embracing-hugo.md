---
date: 2025-01-25T12:08:01+08:00
draft: false
title: Embracing Hugo
summary: I’ve embraced Hugo to power my blog, valuing its speed, simplicity, and security over database-driven platforms. Writing in Markdown, version control integration, and easy deployment make it perfect for documenting my journey in cloud, networking, and cybersecurity. Hugo keeps the focus on learning and sharing—not maintenance headaches.
tags:
  - blog
categories:
  - hugo
series:
  - my-hugo-site
---

# Embracing the Speed and Simplicity 

As I navigate the ever-evolving landscape of technology, I'm always on the
lookout for tools that are efficient, powerful, and enjoyable to use. When it
came to building this digital space to document my learning journey in cloud
computing, networking, cybersecurity, and beyond, I landed on Hugo, a static
site generator, and I haven't looked back

#### What is Hugo?

For those unfamiliar, Hugo is a framework for building static websites. Unlike
dynamic sites that rely on databases and server-side processing for each page
request, Hugo pre-renders all your content into static HTML, CSS, and JavaScript
files. This results in incredibly fast loading times and a significant reduction
in server complexity and security risks.

#### Why Hugo? My Key Motivations

**Blazing Speed**: This was a major draw. Hugo is renowned for
online, making it easier to learn and troubleshoot.

**Version Control Friendly**: Static sites are inherently easy to manage with
version control systems like Git. This makes collaboration (if I  decide to create my own open-source  blog theme) and tracking changes a breeze.

**No Database Hassle**: Eliminating the need for a database simplifies
deployment and maintenance considerably. This aligns with my desire to focus on
learning core infrastructure and security concepts without the added overhead
of database management for my blog.

**Content Separation**:  My content is decoupled from the website framework.   My content is just text files in markdown that I can organise separately to how my site is organised.  I can choose the tools I want for editing.  This opens up many advantages and opportunities I will explore as my site develops.

#### My Hugo Workflow (the plan)

My workflow to implementing a Hugo site, hopefully, should be pretty straightforward:

1. **Content Creation:** I write my blog posts in Markdown files, focusing on
   clearly articulating my experiences and insights.
2. **Theme Selection and Customisation:** I've chosen a theme that resonates with my
   style and have tweaked it to personalise the look and feel.  Hugo's templating system makes this customisation relatively accessible.
3. **Configuration:** The config.toml (or hugo.toml) file is where I set up the site's basic
   settings, menus, and parameters. It's surprisingly intuitive.
4. **Building the Site:** With a simple hugo command in the terminal, the magic
   happens! Hugo rapidly transforms my Markdown content and theme into a complete
   static website in the public directory.
5. **Deployment:** Deploying a static site is incredibly easy. I can simply upload
   the contents of the public folder to a web server or use services specifically
   designed for static site hosting like Cloudflare Workers or GitHub Pages.

#### Looking Ahead with Hugo

My journey with Hugo is just beginning, and I'm excited to explore its more
advanced features, such as shortcodes for embedding dynamic content and data-
driven pages. For now, I'm incredibly satisfied with its speed, simplicity, and
the control it gives me over my blog's presentation.

If you're looking for a fast, flexible, and powerful way to build a website,
especially a blog focused on technical content, I highly recommend giving Hugo a
try. It's been an invaluable tool in documenting and sharing my own path of
learning and discovery.