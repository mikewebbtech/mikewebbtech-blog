---
date: 2025-09-27T08:57:08+08:00
draft: false
title: Hugo Breadcrumbs
summary: Getting breadcrumbs to work using Hugo shortcodes is doable with some effort and a lot of CSS.  But it might not work as expected.  Let me explain
tags:
  - blog
categories:
  - hugo
series:
  - my-hugo-site
---
# Navigation Breadcrumbs

Breadcrumbs are a common navigational aid used on websites to help users keep track of their location in the website by providing a *breadcrumb* trail back to the start page.  Like a visual link click history displayed on the webpage or a way directly jump back to any point in the history of links that lead to the user arriving at that webpage, starting with the site root.

Usually they are positioned above or under the page title and I have seen many different stylings but in essence they all have this basic format
```
Home > Categories > Sub-Category > Page 
```

To be honest, this turned into a bit of rabbit hole me.  Beyond researching what built in methods Hugo had to help me achieve my goals, there is a plethora of best practices, guides and structured data schema on how to implement navigational breadcrumb....but yet there is not semantic html element.

#### Diving in

Emboldened with recently working out custom Hugo [shortcodes](https://gohugo.io/content-management/shortcodes/), this approach to creating a separate code snippet and embedding it  was going to be my hammer for this nail.  Hugo has a built in *PAGE* method called [Ancestors](https://gohugo.io/methods/page/ancestors/) that is well documented with an example on how to create breadcrumbs...cool.  Also the Hugo discourse had this [little nugget](https://discourse.gohugo.io/t/generating-breadcrumbs-for-hugo/42059)



[Mozilla MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/Layout_cookbook/Breadcrumb_Navigation) has  a layout cookbook with a CSS recipe that I used as starting point.  

Schema.org has a [good page](https://schema.org/BreadcrumbList) on breadcrumb structured data that helped me understand the standardised approach that is in many examples.

W3.org has an [ARIA APG]  on element attributes with a good example of HTML source code that helped a lot 

**TOOLED UP**

So, after a few versions, this what I ended up hammering out is a code snippet called "*breadcrumbs.html*", since  will be calling this in a Hugo template file and not inline in a markdown file, this doesn't need to be a shortcode but what Hugo calls an [Embedded partial template](https://gohugo.io/templates/embedded/#article).   

I also had to call a couple of other built in methods, [Site.BaseURL](https://gohugo.io/methods/site/baseurl/) and [Page.IsHome](https://gohugo.io/methods/page/ishome/#article),  as I didn't want the range loop grabbing the title of the home page when it collected the title of the other pages.  I wanted the canonical root to be called Home .  This gets called in the *header.html* partial file with the `{{ partial "breadcrumbs.html" . }}` notation

**[breadcrumb.html](https://github.com/mikewebbtech/mikewebbtech-hugo/tree/339f5b149e93c9cf97514f8d5e4e578b4048393d)**
```html
<nav aria-label="breadcrumb" class="breadcrumb">
  <ol>
    {{- $currentPage := . -}}
    <li>
      <a href="{{ .Site.BaseURL }}">Home</a>
    </li>
    {{- range .Ancestors -}} 
      {{- if ne .IsHome true -}}
        <li>
          <a href="{{ .Permalink }}">{{ .Title }}</a>
        </li>
      {{- end -}} 
    {{- end -}} 
    {{- if not .IsHome -}}
      <li class="active">
        <a aria-current="page" href="{{ .Permalink }}">{{ .Title }}</a>
      </li>
    {{- end -}}
  </ol>
</nav>
```

And finally a whole lot of CSS to transform it from a standard looking HTML ordered list, as per the structured data schema, into something resembling a website navigation breadcrumb.

**[style.css](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/339f5b149e93c9cf97514f8d5e4e578b4048393d/assets/css/style.css)**
```css
/* breadrumbs */
.breadcrumb {
  position: absolute; /* Position absolute the .header element */
  bottom: 0; /* Position at the bottom edge of the header */
  left: 3rem;
  font-size: 0.8em;
}

.breadcrumb ol {
  list-style: none; /* Remove default list bullets */
  margin: 0;
  padding: 0;
  display: flex; /* Arrange list items horizontally */
  flex-wrap: wrap; /* Allow breadcrumbs to wrap on small screens */
}

.breadcrumb ol li {
  display: flex;
  align-items: center; /* Center content vertically */
}

.breadcrumb ol li a {
  text-decoration: none;
  white-space: nowrap;
}

.breadcrumb ol li a:hover {
  text-decoration: underline; /* Underline on hover */
}

/* Creates the separator but not on the last child*/
.breadcrumb ol li:not(:last-child)::after {
  content: ">"; /* The separator character */
  margin: 0 8px; /* Space before AND after the separator.*/
}

/* Style for the active breadcrumb item */
.breadcrumb ol li.active a {
  pointer-events: none; /* Make it unclickable */
  text-decoration: none;
}
```

#### It Works as Expected

But as I intended it work and it makes sense after I spent some effort trying to fix an issue that didn't exist.  The issue was translating my experience with database driven dynamic site design to how a static site generator work...and a little bit of my pathways to an article.

Hugo ingests content from the /content directory.  all of my article markdown files are in a subdirectory called articles, grouped by year (a structure I intend to exploit for an archive function) and I have created a Hugo site has multiple pathways to each article.  An article  is rendered into the site via:
- Articles, sorted by most recent, list.
-  Categories, a block grouping of similar articles, sorted by most recent.
- Series, an article can also be be apart of a series, sorted by first created.
-  Tags, a secondary block grouping of similar articles, sorted by most recent.

This grouping is created by use of [front matter](https://gohugo.io/content-management/front-matter/) data for each article, along with an additional directory structure for categories, series and tags.  I have included an example of my Hugo content directory structure to show how this works and to help make sense of the how breadcrumbs work and what I was expecting.

```
.
├── content
    ├── _index.md
    ├── about
    │   └── index.md
    ├── articles
    │   ├── 2008
    │   │   ├── 2008-05-12-article.md
    │   │   └── 2008-11-06-article.md
    │   └── 2025
    │       ├── 2025-02-01-article.md
    │       └── 2025-05-03-article.md
    ├── categories
    │   ├── _index.md
    │   ├── automation
    │   │   ├── _index.md
    │   │   └── image.png
    │   └── cloud
    │       ├── _index.md
    │       └── image.png
    ├── series
    │   ├── _index.md
    │   ├── intro-cyber-security
    │   │   ├── _index.md
    │   │   └── image.png
    │   ├── my-hugo-site
    │   │   ├── _index.md
    │   │   └── image.png
    │   └── proxmox-gitops
    │       ├── _index.md
    │       └── image.png
    └── tags
        └── _index.md
```

> [!IMPORTANT]
> Hugo's template methods  (`.Ancestors`, `.Parent`, etc.) can only build a **hierarchical breadcrumb** based on the static file structure (e.g., `/articels/2025/article` always has `/articles/` as its parent).  It knows nothing about the user's click history. 😒

When navigating to an article via the article pathway:
Expected : HOME > ARTICLES > "TITLE"
✅ Result : HOME >ARTICLES > "TITLE"

When navigating to an article via the categories pathway:
Expected : HOME > CATEGORIES > CLOUD > "TITLE"
❌ Result : HOME >ARTICLES > "TITLE"

When navigating to an article via the tags pathway:
Expected : HOME >TAGS > GIT > "TITILE"
❌ Result : HOME >ARTICLES > "TITLE"

This is the expected behaviour for Hugo breadcrumbs and it makes sense when I stop to think about it.  For Hugo to meet my expectation and render every combination of pathways to an article as static pages, it would need to create thousands of web pages.  This is unrealistic expectation and would produce a massive website.

So in my attempt to help a user know where they are in my site and have a logical way back,  I think I might have created a something confusing and non linear.  I guess I was after something similar to the way a browser back button works.

Maybe I can solve this with a sprinkling of JavaScript 🤔.

The complete code and site layout is available in my [github repo](https://github.com/mikewebbtech).  I will link to directly to the [commit shown in this article](https://github.com/mikewebbtech/mikewebbtech-hugo/tree/339f5b149e93c9cf97514f8d5e4e578b4048393d) for consistency.  There is good chance that this will evolve