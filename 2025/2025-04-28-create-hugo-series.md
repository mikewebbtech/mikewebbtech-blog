---
date: 2025-04-28T00:00:00+08:00
draft: false
title: Creating a Hugo Taxonomy
summary: I outline the process I used to create a custom Hugo Taxonomy so could have articles ordered into a series.  There are a lot of moving parts in getting this to work.  Hopefully gets the broad strokes explained with the resources I used to help me.
tags:
  - blog
  - howto
  - series
categories:
  - hugo
series:
  - my-hugo-site
---

# Creating a Series in Hugo

I wanted a way to organise multi-part content on my site into something more sequential then the standard Hugo categories and tags.  The solution I wanted needed creating a custom **Series** taxonomy in Hugo.  This not only organises articles but also provides easier navigation for a topic.

This is an outline of the process I followed,  I'll copy  code snippets from my files that I created.  All the templates and partials that I created are available in [my git repo](https://github.com/mikewebbtech/mikewebbtech-hugo) if you would like a complete overview.

### Creating a Custom a Taxonomy

 Hugo has the builtin taxonomies for **Categories** and **Tags**.  "Hugo includes support for user-defined taxonomies" as well and  has some [great documentation](https://gohugo.io/content-management/taxonomies/)  on this along with [how to configure](https://gohugo.io/configuration/taxonomies/) them for a site.  Also important is an understanding of Hugo [Term templates](https://gohugo.io/templates/types/#term) and how they work with taxonomies. 
 
 The first up, tell Hugo about the new taxonomy.  In my hugo.toml configuration file I added the `series` key under the `[taxonomies]` block:

[hugo.toml](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/hugo.toml)
```toml
[taxonomies]
  tag = "tags"
  category = "categories"
  # This line helps create the magic:
  series = "series"
```

This instructs Hugo to:
1. Treat any `series` parameter in front matter as a taxonomy.
2. Generate the main list page for all series at `/series/`.
3. Generate individual term pages for each series (e.g., `/series/my-series-name/`).

### Creating the Taxonomy Pages with `_index.md` 🧠

The taxonomy definition above in the config file tells Hugo *what* a series is.  Creating a series subfolder in the content/ directory  and using `_index.md` files in the `content/series/` directory tells Hugo *how to build* the landing pages for the taxonomy.

#### A.  The Role of `_index.md`

In Hugo, any directory containing an `_index.md` file becomes a **[Branch Bundle](https://gohugo.io/content-management/page-bundles/#branch-bundles)**.  This bundle acts as a logical grouping, and the `_index.md` itself represents the list page for that branch.

**`content/series/_index.md`** creates the **Taxonomy List Page**.  This is the master page that lists **all** available series terms and is rendered using the `layouts/series/terms.html` template (or `layouts/_default/terms.html` if the series-specific one is missing).
I used this file to define the global series title and an introduction:
content/series/_index.md
```toml
    +++
    title = 'Series'
    [params]
        subtitle = 'Knowledge Stacks '
    +++
    
    Sometimes a topic is too big to write or read in a single article...
```

**`content/series/series-name/_index.md`** creates a **Taxonomy Term Page**. This is the page that lists all the articles belonging to a *series* (e.g. all posts tagged with "[my-hugo-site](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/content/series/my-hugo-site/_index.md?plain=1)").  This page is rendered using the [layouts/_default/list.html](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/layouts/_default/list.html) template file.

#### B.  Connecting Terms to Content

When I create a series like **"My Hugo Site"** by making the directory structure `content/series/my-hugo-site/`, the folder name, `my-hugo-site`, becomes the actual **taxonomy term** that Hugo uses for grouping.

My `content/series/my-hugo-site/_index.md` looks like this:

```toml
+++
title = "My Hugo Site"
[params]
  subtitle = "Go Static Site"
+++

How I am creating this site using the HUGO Static Site Generator framework
```

**How Hugo Taxonomies work with this file:**

1. **Term Page URL:** Hugo uses the folder structure to generate the URL: `/series/my-hugo-site/`.


2. **`list.html` Logic:** When this page loads, the `list.html` template detects that it is rendering a taxonomy term page for the `series` plural (`eq .Kind "term"` and `eq .Data.Plural "series"`), and ensures the articles are correctly sorted by date:
    
    Code snippet:
    
    ```go-html-template
    {{ if and (eq .Kind "term") (eq .Data.Plural "series") }}
    {{ $pages = sort $pages "Date" }}
    {{ end }}
    ```


2. **Term Page Title:** By giving the `_index.md` a `title` of `"My Hugo Site"`, that title is used on the term landing page, overriding the default behaviour of simply capitalising the folder name (`My Hugo Site`) in the navigation.

### Tagging Content in Front Matter

After the series taxonomy is defined, I can now create new series and content to them .  For any article I wanted to be part of a series, I added the `series` parameter to its front matter.  Importantly, I used an array (`[]`) because an article could potentially belong to more than one series (this is a logic choice in my template code).

front matter example
```yaml
---
title: "Part 2: Configuring the Layout"
date: 2024-09-28T20:00:00+08:00
series: ["hugo-build-guide"] # The name for the series
tags: ["hugo", "series"]
---
```

### Enhancing the Single Article View

The important part was making sure if an article was part of a series , it could easily navigate as a series.  I implemented two key components in my `single.html` layout template.

##### A.  The Series Metadata in the Header

Inside the `.article-meta` div in my `layout/_default/single.html` template file, I added logic to display the series name, link, and the article's position within the series ( "Article 3 of 5" ).

```go-html-template
{{- with .Params.series }}
<div>
  <i class="fa-solid fa-layer-group"></i>
  Series: {{- range $seriesName := . -}} 
    {{ with $.Site.GetPage (printf "/%s/%s" "series" $seriesName ) }}
      <a class="tag" href="{{ .RelPermalink }}">{{ .Title }}</a>
    {{- end }} 
    {{ $currentSeries := index $.Site.Taxonomies.series ($seriesName | urlize) }} 
    {{ if $currentSeries }} 
      {{ $total := len $currentSeries.Pages }} 
      {{ range $index, $page := $currentSeries.Pages }} 
        {{ if eq $page.Permalink $.Permalink }} 
          {{ $articleNumber := add $index 1 }} 
          (Article {{ $articleNumber }} of {{ $total }}) 
        {{ end }}
      {{end }} 
    {{ end }} 
  {{- end }}
</div>
{{- end }}
```

#### B.  The Collapsible Series List

I wanted a collapsable list of all articles in the series near the top of the post for easy reference an navigation. So I created the `series-list.html` partial to handle this.  

This partial uses the `<details>` and `<summary>` HTML tags, which I styled with the `.series-list` class to make it collapsible and neat.

```go-html-template
{{- with (.GetTerms "series") -}} 
  {{- range . -}} 
    <details class ="series-list">
      <summary>
        <span> All articles in the <strong>"{{ .Title }}"</strong> series </span>
      </summary>
      <ol>
        {{- range .Pages.ByDate -}} 
          {{- if eq .File.UniqueID $.File.UniqueID -}}
            <li class="active">
              <b>{{ .Title }}</b>
            </li>
          {{- else -}}
            <li>
              <a href="{{ .Permalink }}">{{ .Title }}</a>
            </li>
          {{- end -}} 
        {{- end -}}
      </ol>
    </details>
  {{- end -}}
{{- end -}}
```

This partial gets embedded near the top of  the `single.html` template with the following logic to first check of if the page is a part of a series.

```go-html-template
{{- if .Params.series -}}
  <div class="post-series-list">{{ partial "series-list.html" . }}</div>
  {{- end -}}
```

#### C.  In-Series Pagination

Finally, to create a smooth flow, I make use of a hugo built in feature called [Pagination](https://gohugo.io/templates/pagination/#pagination) to add familiar next/previous navigation buttons using the [series-pagination.html](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/layouts/partials/series-pagination.html) partial embbed at the bottom of the article `single.html` template.

The `series-pagination.html` partial uses the `Pages.ByDate` collection of the current series to find the indices (`$prevPageIndex` and `$nextPageIndex`) of the articles before and after the current one, providing seamless navigation.

Code snippet:
```
{{- with (.GetTerms "series") -}} 
  {{- range . -}} 
    {{- $curPageIndex := 0 -}} 
    {{- range $idx, $page := .Pages.ByDate -}} 
      {{- if eq $page $ -}} 
        {{ $curPageIndex = $idx }} 
      {{- end -}} 
    {{- end -}} 
    {{- $prevPageIndex := sub $curPageIndex 1 -}}
    {{- $nextPageIndex := add $curPageIndex 1 -}}
    <div class="pagination-buttons">
      {{- with (index .Pages.ByDate $prevPageIndex) -}}
      <span class="button previous">
        <a href="{{ .Permalink }}" title="PREVIOUS ARTICLE">
          <span class="button-icon">←</span>
          <span class="button-ext">{{ add $prevPageIndex 1 }}. {{ .Title }}
          </span>
        </a>
      </span>
      {{- end -}} 
      {{- with (index .Pages.ByDate $nextPageIndex) -}}
        <span class="button next">
        <a href="{{ .Permalink }}" title="NEXT ARTICLE">
          <span class="button-text">{{add $nextPageIndex 1 }}. {{ .Title }}</span>
          <span class="button-icon">→</span>
       </a>
       </span>
     {{- end -}}
    </div>
  {{- end -}} 
{{- end -}}
```

This partial gets embedded near the base of  the `single.html` template with the following logic to first check of if the page is a part of a series.

```go-html-template
<div class="post-footer"></div>
  {{- if .Params.series -}}
  <div class="post-series-bottom">{{ partial "series-pagination.html" . }}</div>
  {{- end -}}
```

### Listing All Series and Ordering Articles

#### A.  The List of Series

The main `/series/` page lists all available series. This is created using the the `layout/series/terms.html` layout, Taxonomy [Terms](https://gohugo.io/templates/types/#term) is an important concept to understand in Hugo.  This terms.html template iterates over all [Page.Kinds](https://gohugo.io/methods/page/kind/) that for the series terms and shows the number of articles in each (`.Count`).  I also added some CSS to give them some style for visual separation.

[layout/series/term.html](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/layouts/series/terms.html)
```go-html-template
{{ define "main" }}
<!-- this layout is used by the series branch page to list out the all series -->
<article class="cat">
  <div class="home-image">
    {{ with .Resources.Get "image-header.png" }} 
    {{ with .Process "resize 900x200 CatmullRom webp photo q99" }}
    <img
      src="{{ .RelPermalink }}"
      width="{{ .Width }}"
      height="{{ .Height }}"
      alt=""
    />
    {{ end }} {{ end }}
  </div>
  <h2 class="subtitle">
    <i class="fa-solid fa-layer-group"></i> {{ .Params.Subtitle}}
  </h2>
  {{ .Content }}
  <ul class="cat-list">
    {{ range .Data.Terms.Alphabetical }}
    <li class="cat-list-item">
      <a class="cat-list-item-link" href="{{ .Page.RelPermalink }}">
        <div class="cat-list-img">
          {{ with .Page.Resources.Get "image.png" }} 
          {{ with .Process "resize 90x CatmullRom webp photo q99" }}
          <img
            src="{{ .RelPermalink }}"
            width="{{ .Width }}"
            height="{{ .Height }}"
            alt=""
          />
          {{ end }} {{ end }}
        </div>
        <div class="cat-list-item-info">
          <div class="cat-list-item-title">{{ .Page.Title }}</div>
          (This series has {{ .Count }} pages)
          <!-- if there a series description. show it -->
          {{ with .Page.Content }} {{ . }} {{ end }}
        </div>
      </a>
    </li>
    {{ end }}
  </ul>
</article>
{{ end }}
```

#### B.  Ensuring Correct Series Order

when viewing an *individual series page* (like `/series/hugo-build-guide/`), it uses the site [layout/_default/list.html](https://github.com/mikewebbtech/mikewebbtech-hugo/blob/main/layouts/_default/list.html)  template. I added a conditional statement here to ensure that articles in a series are *always* sorted by their publication date, (oldest article listed first) guaranteeing the correct reading order.

Code snippet:
```go-html-template
    <ul class="article-list">
    {{ $pages := .Pages }}
    {{ if and (eq .Kind "term") (eq .Data.Plural "series") }}
    	{{ $pages = sort $pages "Date" }}
    {{ end }}
```


Lurking in the background is lot of CSS to complement the static HTML files rendered.  I have recycled a lot of the stying used when I created my Hugo categories.  I incorporated a lot of the precesses I used for creating my categories, It gave me the confidence to tackle creating a custom *series* Taxonomy.

I used this as an opportunity to learn how to implement Hugo's  builtin [Pagination](https://gohugo.io/templates/pagination/#pagination) feature, both in the standard page footer navigation style and as a way to built a list of articles in a series at the top of a page.

So far, I am satisfied with the result and fill find it a helpful feature of my site.  I gained greater insight into how Hugo works and this all builds a foundation for adding more functionality and features to my site.

I very much like the idea of thinking of something and also having the tools to create it. 

The complete code and site layout is available in my [github repo](https://github.com/mikewebbtech/mikewebbtech-hugo).