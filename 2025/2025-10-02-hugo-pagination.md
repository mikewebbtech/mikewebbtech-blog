---
date: 2025-10-02T00:00:00+08:00
draft: false
title: Hugo Pagination
summary: In implementing a list pagination feature for my site, I pivoted from complex custom code to using Hugo's robust built-in template. This saved development time, which was better spent styling the default HTML output with custom CSS.
tags:
  - blog
  - howto
categories:
  - hugo
series:
  - my-hugo-site
---
# Using Hugo's Built-in Power

The process of building a personal website is often a balance between the fun of custom coding and the efficiency of using existing tools.  My recent work on creating "[breadcrumb feature](../2025-09-26-hugo-breadcrumbs)" is a good example, culminating in a decision that saved time and simplified the code by using Hugo's built-in [Page.ancestors method](https://gohugo.io/methods/page/ancestors/).

#### Ending A Never Ending Scroll

As the article count on the site grew, the main article listing page became an overwhelming, endless scroll. I knew I needed **pagination** a way to limit the articles to 10 per page and provide simple "Previous" and "Next" navigation.

My first thought was to write the code it myself.  I started mapping out the logic:
- How to handle the current page number?
- How to calculate the total number of pages?
- How to implement "smart" pagination that shows ellipses (`...`) and only the pages adjacent to the current one?

To get a head start I found some great examples that others before me have written about with very similar intentions.  Two outstanding sites on this and Hugo in general are:
- [StaticMania](https://staticmania.com/blog/hugo-pagination), and
- [jonifen](https://jonifen.co.uk/blog/how-i-built-my-custom-hugo-pagination/)

I began writing complex Go template code using Hugo's `.Paginate` function, scratchpad variables, and multiple `if/else`conditions to handle the low-page-count versus high-page-count scenarios.

### The Pivot: Recognising Redundancy

After spending a few to many hours deep in the joy of template logic, I realised the irony:  I was manually recreating a feature that Hugo’s core team had already perfected and packaged.  My complicated code would have to been less refined, hard to follow, and prone to bugs by comparison.

Reading through the [Hugo pagination](https://gohugo.io/templates/pagination/) documentation again, it explains how to implement the builtin template into my own project. The solution was a few lines of code, replacing hundreds of lines of complex logic in my pagination partial:

It made sense to me to implement pagination in my default "[list.html](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/layouts/_default/list.html)" template.  This also gives me pagination for other Taxonomy lists 

To implement this, I had to declare some variables, surround the section that iterates through the list items within the pagination code block declaration, then call the the internal pagination template at the base of the list template.

[layout/_default/list.html](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/layouts/_default/list.html)
```go-html-template
<ul class="article-list">
    {{ $pages := .Pages }}
    {{ if and (eq .Kind "term") (eq .Data.Plural "series") }}
    {{ $pages = sort $pages "Date" }}
    {{ end }}
    
    {{ $paginator := .Paginate (where $pages "Type" "!=" "page") }}
    {{ range $paginator.Pages }}
    ...
    (etc)
    ...
    {{ end }}
</ul>
{{ template "_internal/pagination.html" . }}
<!-- 
 the below also applies along as there is not an existing template 
 called pagination.html in layouts/partials/
 
{{ partial "pagination.html" . }}
-->
```

By pivoting and using Hugo's **`_internal/pagination.html`** template, I immediately gained:
1. **Robust Logic:** Perfectly handles "First," "Previous," "Next," and "Last" links.
2. **Smart Numbering:** Automatically shows and manages adjacent page links.
3. **Accessibility:** The built-in template comes with correct `aria` attributes (`aria-current`, `aria-label`, `role="button"`), which are crucial for screen readers.

#### Shifting Focus to CSS

Once the functionality was solved, my focus shifted from programming logic to **visual design**. The HTML rendered by the internal template was plain, using standard classes like `.pagination-default` and `.page-item`, which clashed with my site's custom look and feel.

This became the most effective use of my time. Instead of fighting with Go templates, I spent time in my **style.css** file, using specific CSS selectors to map my custom styles onto the default HTML structure.

[assets/css/style.css](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/assets/css/style.css):
```
/* I mapped my custom class styles to Hugo's generated classes */

/* 
 * Apply Styling to Hugo default pagination template text
 */
 
.pagination-default {
  list-style: none; /* Ensure no bullets */
  margin: 3rem auto 0 auto; /* Margin-top and center the list */
  padding: 0;
  display: flex;
  gap: 1rem; /* Space between buttons */
  justify-content: center;
}

.page-item {
  display: inline-flex; /* Use inline-flex on the li container */
  /* Remove default list styling */
  margin: 0;
  padding: 0;
}


/* Target the list wrapper */
.page-item a.page-link,
.page-item.active a.page-link,
.page-item.disabled a.page-link {
  /* Apply core button styles to the link element (a.page-link) */
  display: inline-flex;
  align-items: center;
  padding: 0.6rem 1rem;
  border-radius: 0.2rem;
  text-decoration: none;
  /* ... etc. ... */
}

...
etc
...


/* Target the specific link with the "Previous" aria-label */
/* Hide the original content "«" (&laquo;) */
.page-item a[aria-label="Next"] span[aria-hidden="true"] {
  display: none;
}
/* Insert the new text "First" using the ::after pseudo-element */
.page-item a[aria-label="Next"]::after {
  content: "Next →";
}

/* Target the specific link with the "Last" aria-label */
/* Hide the original content "»»"" (&raquo;&raquo;) */
.page-item a[aria-label="Last"] span[aria-hidden="true"] {
  display: none;
}
/* Insert the new text "First" using the ::after pseudo-element */
.page-item a[aria-label="Last"]::after {
  content: "Last";
}
```

I am happy with the results and it was a feature I had been putting off, feeling that it was beyond me and take considerable time to learn how to achieve it

This whole process reinforced the principle of: **Don't reinvent the wheel, but perfect the paint job.**  By leveraging Hugo's power for functionality and applying custom CSS for styling,  I achieved my result faster and with cleaner code.

The complete code and site layout is available in my [github repo](https://github.com/mikewebbtech).
