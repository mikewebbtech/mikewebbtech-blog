---
date: 2025-10-09T00:00:00+08:00
draft: false
title: Smart external links in Hugo with a render hook
summary: How I made external links open in a new tab, added security attributes and appended an icon, without touching Markdown content
tags:
  - blog
  - howto
categories:
  - hugo
series:
  - my-hugo-site
---
I wanted two things from links on my Hugo site:
1) **UX:** External links should open in a new tab and show a small icon so readers know they’re leaving.  
2) **Security/Accessibility:** Add `rel="external nofollow noreferrer` and an ARIA cue (`“aria-label="{{ $text }} (opens in a new tab)”`).

Doing this **inside Markdown** is messy.  Doing it **site wide** with a Hugo **render hook** is cleaner, is adopted by existing function rendering and future proof.

# Options to solution
Initially I thought I had to approach this with shortcodes but reading through the documentation to implement another feature for admonitions and code blocks, I learnt that Hugo includes an embedded link render hook to resolve Markdown link destinations. You can adjust its behaviour in the site configuration  that is [documented in detail](https://gohugo.io/render-hooks/links/) and has a code example to get me started

```go-html-template
{{- $u := urls.Parse .Destination -}}
<a href="{{ .Destination | safeURL }}"
  {{- with .Title }} title="{{ . }}"{{ end -}}
  {{- if $u.IsAbs }} rel="external"{{ end -}}
>
  {{- with .Text }}{{ . }}{{ end -}}
</a>
{{- /* chomp trailing newline */ -}}
```

# My approach
- Hugo lets you override how Markdown links render via `/layouts/_default/_markup/render-link.html`.
- I detect **external vs internal** (respecting my own `baseURL`), treat the handling of other uri types (`mailto:`) safely to not add my render hook:
  - For **external**: add `target="_blank"`, `rel="external nofollow noreferrer`, an ARIA label, and a small external-link **inline image** (I am using font awsome icons for this).
  - For **internal**: output a normal link.
- The hook doesn’t change your content; it **wraps Markdown output** consistently across the site.

> [!NOTE]
>  Setting `target="_blank"` on `<a>`, `<area>` and `<form>` elements implicitly provides the same `rel` behavior as setting `rel="noopener"` which does not set `window.opener`. [MDN noopener](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/rel/noopener)

# The render hook
Create: `/layouts/_default/_markup/render-link.html`

```go-html-template
{{- $u := urls.Parse .Destination -}}
  {{- with .Title }} title="{{ . }}" {{ end -}}
  {{- if hasPrefix .Destination "mailto:" -}}
    <a href="{{ .Destination | safeURL }}" {{- with .Title }} title="{{ . }}" aria-label="{{ $text }} (opens in a new tab)" {{ end -}}>
      {{- with .Text }} 
        {{ . }} 
      {{- end -}}
    </a>
  {{- else -}}
    <a href="{{ .Destination | safeURL }}"
    {{- if $u.IsAbs }} rel="external nofollow noreferrer" target="_blank" {{ end -}} >
    {{- with .Text }} 
      {{ . }}
      {{- if $u.IsAbs }} 
        <span style="vertical-align: super; display: inline-block; font-size: 0.7rem; font-weight: 200;" >
          <i class="fa-solid fa-arrow-up-right-from-square"></i>
        </span>
      {{ end -}}
    {{ end -}}
    </a>
  {{- end -}}
```

**Why this logic works :**

- **External** = absolute `http(s)` link whose prefix **is not** the sites `baseURL`.
- **Internal** = relative links (`/articles/…`, `articles/…`) or absolute links that start with your `baseURL`.
- `mailto:` are **not** forced new-tab and don’t get the icon (keeps assistive tech expectations).

# Minimal CSS
In the render-link template I applied some inline styling to the to appended external link icon to give it that *familiar look*

```html
<span style="vertical-align: super; display: inline-block; font-size: 0.7rem; font-weight: 200;" >
    <i class="fa-solid fa-arrow-up-right-from-square"></i>
</span>
```

# Config tweaks

You **don’t** need special settings configured in hugo.toml to use a render hook.  Since I am  sanitising the HTML, keeping the icon inline as above should keep things  safe.  But I if  extending Markdown attributes like `{.btn}`, ensuring I have following goldmark attribute set gives me future proofing if I need it:

```toml
[markup.goldmark.parser.attribute]   
    block = true
    title = true
```

# A quick test matrix

Paste these in a draft and preview:

```markdown
- Internal relative: [Home](/) 
- Internal absolute: [Self]({{< relref "/" >}}) 
- External: [Hugo docs](https://gohugo.io/)
- Mail: [Email me](mailto:hello@example.com)
- Tel: [Call](tel:+61855501234)`
```

**Expected Result:** only “Hugo docs” should open in a new tab and shows the icon.

# Dam edge cases and gotchas

- **Custom domains / GitHub Pages paths:** The hook compares against `site.BaseURL`.  I need Make sure the paths are correct otherwise Hugo may misclassify links.
    
- **Links rendered by shortcodes/HTML:** This hook only affects **Markdown links**. Other build links in shortcodes/partials will  add the same attributes there.  This is a bonus  for my use case but something I need to be aware of.
    
- **Analytics or CSP:** Since I added external icon fonts, I need to be mindfull of *Content Security Policies*.  HTTP header (or `<meta>` tag) that tells the browser **which sources are allowed** for scripts, styles, images, fonts, etc. The goal: **block XSS and rogue resources**. If something loads from a source you didn’t whitelist, the browser refuses it  (and can report it).

- **Accessibility:** The ARIA label announces the new-tab behaviour; don’t also add “(external)” in the visible text unless your style guide requires it.

# Why a render hook (and not JS)?

- **Deterministic:** Works at build time; no FOUC or client-side mutation.  FOUC - *Flash Of Unstyled Content*,   It’s when a page briefly shows raw HTML without your CSS applied, then “snaps” to the styled version once the stylesheet loads. I deal with this in the header loading  the style sheet with `{{ $css := resources.Get "css/style.css" | minify }}`.
- **Consistancy:** Every Markdown link gets the same treatment, perfect for consistency avoiding human error.
- **Fast:** No runtime overhead, no brittle DOM rewrites.