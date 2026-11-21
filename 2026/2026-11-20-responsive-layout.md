---
date: 2026-11-20T00:00:00+08:00
draft: false
title: Site Layout Responsive Design
summary: Making changes to my HUGO site layout so that it is responsive to the changes in screen sizes.  I had to make some layout descisions, but everything could be done in SCSS wth no changes required to the template code. 
tags:
  - blog
categories:
  - hugo
series:
  - my-hugo-site
---

# Responsive Layout Changes

## Scope

This outlines the responsive layout changes on the `feat/responsive-prep` branch. The desktop layout keeps the fixed left sidebar. At viewport widths of `900px` and below, the sidebar moves above the page content and the navigation becomes a horizontally scrollable row.

The `900px` breakpoint also applies to narrower phone screens. There is not yet a separate phone-specific breakpoint or a collapsible menu.

## Desktop Layout

The desktop grid reserves the same `14rem` width as the fixed sidebar. This replaces the earlier mismatch where the grid reserved `12rem` but the sidebar occupied `14rem`.

```scss
.layout {
  display: grid;
  grid-template-columns: 14rem 1fr;
  grid-template-areas:
    "sidebar header"
    "sidebar main";
  grid-template-rows: 4rem 1fr;
  min-height: 100vh;
}

.sidebar {
  grid-area: sidebar;
  position: fixed;
  top: 0;
  width: 14rem;
  height: 100vh;
}
```

The logo and navigation spacing no longer depends on large fixed offsets. The logo image uses a bounded fluid width, and the navigation starts closer to the logo.

```scss
.logo {
  min-height: 4rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
  padding: 1rem 0;
}

.logo img {
  margin-top: 0;
  width: clamp(4rem, 60%, 8rem);
  height: auto;
}

.nav {
  margin-top: 1rem;
}
```

The separate logo background color was disabled so the logo area uses the same background as the sidebar.

## Tablet And Narrower Layout

At `900px` and below, the grid changes to a single column. The sidebar is returned to normal document flow, expands to the viewport width, and no longer uses a fixed viewport height.

```scss
@media (max-width: 900px) {
  .layout {
    grid-template-columns: minmax(0, 1fr);
    grid-template-areas:
      "sidebar"
      "header"
      "main";
    grid-template-rows: auto auto 1fr;
  }

  .sidebar {
    position: static;
    width: 100%;
    height: auto;
    z-index: auto;
  }
}
```

The logo becomes a compact horizontal row that can wrap when necessary. The image and byline are reduced in size for the narrower layout.

```scss
.logo {
  flex-direction: row;
  flex-wrap: wrap;
  min-height: 0;
  padding: 0.75rem 1rem;
}

.logo img {
  width: 3.5rem;
}

.byline {
  font-size: 1rem;
}
```

The page heading is vertically centered in a shorter banner. Main content uses responsive horizontal padding instead of the wider desktop offset.

```scss
.header {
  display: flex;
  align-items: center;
  justify-content: center;
  height: auto;
  min-height: 3rem;
  padding: 0.25rem 1rem;
}

.header h1 {
  margin: 0;
}

.main {
  padding: 1rem clamp(1rem, 3vw, 2rem) 0;
}
```

## Horizontal Navigation

At tablet and narrower widths, navigation items stay on one line. The navigation container scrolls horizontally, supporting touch swipes and horizontal trackpad scrolling. `overscroll-behavior-x` keeps the gesture from unnecessarily chaining to another horizontal scroller.

```scss
.nav {
  margin-top: 0;
  overflow-x: auto;
  overscroll-behavior-x: contain;
}

.nav ul {
  display: flex;
  flex-wrap: nowrap;
  justify-content: flex-start;
  gap: 0.25rem;
  width: max-content;
  min-width: 100%;
}

.nav li,
.nav a {
  white-space: nowrap;
}
```

## Footer And Spacing

The credit and license links were moved out of the sidebar into `layouts/partials/footer.html`. The base template renders that partial through a footer block after the main layout, so the footer is no longer structurally part of the sidebar.

```go-html-template
{{- block "footer" . -}}
  {{- partial "footer.html" . -}}
{{- end -}}
```

The footer text uses the muted theme color:

```scss
.nav-bottom {
  color: var(--color-text-muted);
}
```

The earlier large spacing around the logo and navigation was removed. At tablet widths, `.nav-bottom` no longer uses an automatic top margin, and explicit line breaks inside it are hidden to avoid adding vertical space.

## Result

- Desktop retains the fixed `14rem` sidebar and two-column page grid.
- Tablet and narrower screens use a stacked, full-width sidebar above the page heading and content.
- Tablet navigation remains on one line and can be scrolled horizontally.
- The heading banner and footer spacing are reduced at the tablet breakpoint.
- No JavaScript or navigation markup change was needed for horizontal scrolling.

## Build Check, Commit and push

The site compiled successfully with:

```sh
hugo --renderToMemory
```

Now all I had to do was commit the changes to the `feat/responsive-prep` branch, merge with `main` and push to GitHub, handing the workflow to my actions that builds the site and deploys it to GitHub Pages....for free