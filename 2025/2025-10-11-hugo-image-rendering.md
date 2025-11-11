---
date: 2025-10-11T08:53:47+08:00
draft: false
title: Hugo rendering images in markdown articles
summary: How I organise, reference, and render images in my Hugo site using page bundles, the static folder, and shortcodes. Use standard markdown or extended attributes.
tags:
  - blog
  - howto
categories:
  - hugo
series:
  - my-hugo-site
---

# Using images in Hugo

I have been writing a lot of articles, well mainly rewording my obsidian notes and having Hugo render and publishing them.  But they are all just text files and some of my notes I would like to publish have images.  Images make ideas easier to understand and make the information more engaging

**I want to start using images embedded in my markdown**

Hugo gives you a few different ways to work with images:
- throw them in the `static/` folder and link to them,
- keep them next to the content as **page bundles**,
- or go further and use Hugo’s **image processing** features in templates and shortcodes.

In this post I’ll walk through the basics of how I’m using images in my Hugo site, and the trade-offs between each option.

> ## Basic mental model: where do images live?
> {.one}

Hugo doesn’t really care _what_ the file is – image, CSS, PDF – it mainly cares _where_ it lives:
- `static/` – copied as-is into the final site, keeping the same path.
- `content/` – Markdown + other files that belong to a specific page or section (page bundles).
- `assets/` – source files used by Hugo Pipes (SCSS, JS, etc.), not our focus here.

For images, the two common patterns are:
1. **Global images** under `static/` (e.g. `/static/images/logo.png`)
2. **Per-page images** stored alongside the Markdown as a page bundle.

I use both, depending on the type of image.

> ## Option 1: images in the `static/` folder
> {.two}

This is the simplest option and works like a traditional web server:
```text
static/
  images/
    logo.png
    profile.jpg
```

Anything under `static/` ends up at the root of the site. So:
 `static/images/logo.png` → `/images/logo.png`  on the live site.
 
You can then reference it in your Markdown like this:
```markdown
![My site logo](/images/logo.png)
```

or in HTML:
```html
<img src="/images/logo.png" alt="My site logo">
```

### When I use this

- Shared assets that are used across multiple pages (logos, favicons, backgrounds).
- Things that don’t belong to a specific article.

### Pros

- Very simple to understand.
- Paths are predictable (`/images/...`).

### Cons

- Easy for the `static/` folder to become messy over time.
- Harder to move a post (its images are not bundled with the content).

> ## Option 2: page bundles (images next to the content)
> {.three}

For blog posts, I usually prefer **page bundles**. The idea is:

> Keep the Markdown and its images together in one folder.

Example **leaf bundle**:
```text
content/
  blog/
    using-images-in-hugo/
      index.md
      diagram-01.png
      diagram-02.png
```

The key detail: the page file is named `index.md`. Hugo treats that folder as a single “page”, and everything else inside becomes **page resources**.

Inside `index.md`, you can reference the images with a relative path:
```markdown
![Diagram 1](diagram-01.png)

![Diagram 2](diagram-02.png)
```

Hugo will resolve those paths when building the site.

### When I use this

- Blog posts or pages where the images are only used in that one article.
- Content that I might move or copy later – images come along automatically.

### Pros

- Content and images stay together.
- Easier to refactor sections or move a post around.
- Cleaner `static/` folder.

### Cons

- You need to stick to the bundle pattern (e.g. using `index.md`).
- Paths change if you move things around outside Hugo’s expectations.

> ## Using Hugo’s built-in `figure` shortcode
> {.four}

Hugo comes with a `figure` shortcode that can wrap an image with a caption and some extra attributes.

Example:

```markdown
{{</* figure src="diagram-01.png" alt="Overview diagram" caption="High-level flow of the pipeline." */>}}
```

This assumes the image is either:
- in the same bundle as `index.md`, or
- in `static/` with a path like `/images/...` (in which case you pass the full path).

I like this for images that need:
- a caption,
- attribution,
- or extra semantics beyond a standard Markdown image.

> ## A simple custom image shortcode
> {.five}

If you want consistent classes, lazy-loading, or a wrapper div, you can create your own shortcode.

Example shortcode: `layouts/shortcodes/figure-img.html`:
```go-html-template
{{- $src := .Get "src" -}}
{{- $alt := .Get "alt" | default "" -}}
{{- $class := .Get "class" | default "post-image" -}}

<figure class="{{ $class }}">
  <img src="{{ $src }}" alt="{{ $alt }}" loading="lazy">
  {{ with .Inner }}
    <figcaption>{{ . }}</figcaption>
  {{ end }}
</figure>
```

Then use it in your Markdown like this:
```markdown
{{</* img src="diagram-01.png" alt="Overview diagram" */>}}
Optional caption text goes here.
{{</* /img */>}}
```

This gives you:
- consistent HTML structure,
- built-in `loading="lazy"`,
- a single place to tweak styling later.

> ## Basic image processing with page resources 
> {.one}

Hugo can do more than just “serve” an image. With page bundles, images become **resources**. I am transform and optimising them in templates.

Example in a template or shortcode:
```go-html-template
{{ with .Resources.GetMatch "hero*"}}
  {{ $resized := .Fit "800x600" }}
  <img src="{{ $resized.RelPermalink }}" alt="Hero image">
{{ end }}
```

This:
- looks for an image whose filename starts with `hero`,
- resizes/crops it to `800x600`,
- and uses the processed version in the final HTML.

I treat this as “phase two”: first get basic images working, then come back and add processing where it makes sense (hero banners, thumbnails, etc.).

> ## Good practices I try to follow
> {.two}

A few practical habits that make life easier in the long run.

### 1. Always write useful alt text

Bad:
```markdown
![Image](diagram-01.png)
```

Better:
```markdown
![Hugo image organisation diagram](diagram-01.png)
```

This helps with:
- accessibility (screen readers),
- SEO,
- your own sanity when skimming Markdown later.

### 2. Sensible naming

Instead of `image1.png`, `image2.png`:
- use descriptive names: `hugo-page-bundle-diagram.png`
- use dashes instead of spaces.

Makes searching and re-use much easier.

### 3. Keep shared vs per-page images separate

Rough pattern I use:
- shared: `static/images/...`
- per-page: inside the page bundle next to `index.md`

If I have to move a post, it’s obvious which images go with it.

### 4. Watch your repo size

High-res screenshots and photos add up fast.
- resize or compress images before committing if possible,
- use formats like WebP for some images if your theme/workflow supports it.

> ## How I’m using images right now
> {.three}

On my site, my current pattern is:
- **Logos / avatars / favicons** → `static/images/`
- **Blog diagrams / screenshots** → page bundles (`content/blog/post-name/index.md`)
- Hugo’s **`figure` shortcode** or a custom `img` shortcode when I want captions or consistent styling.
- Basic Markdown images when I just need something inline and simple.

It’s not the only way to do it, but it keeps things organised and plays nicely with Git, Hugo, and GitHub Pages.

As I improve my templates, I’ll probably lean more on page resources and image processing – but getting these basics right first already makes working with images in Hugo much less painful.

---

If you’re just starting with Hugo, my advice is:
1. Decide on a simple folder structure for your images.
2. Pick one pattern (static vs bundles) and stick to it for a while.
3. Only add fancy stuff (shortcodes, processing) once the basics feel boring.

Boring is good. It usually means your setup is working.  
