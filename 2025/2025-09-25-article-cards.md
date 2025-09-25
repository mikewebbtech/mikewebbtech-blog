---
date: 2025-09-25T12:10:33+08:00
draft: false
title: Article Cards on Homepage
summary:
tags:
  - blog
  - solution
categories:
  - hugo
series:
  - my-hugo-site
---

# Creating Article Cards

For sometime I have had a place holder on the homepage for a list of the most recent articles.  This had some inhibiters to making that a reality and to, to be honest, that was all me.  
- I had to find the time to work on this
- I had to learn how to do this
- I had to decide on what this looked like, and
- I had to decide on how it was going to work.

Still, a nice little challenge that I was looking forward to making happen.

#### Markdown is a dumb file

Markdown is just a text file with formatting and perfect for creating content for a blog.  The issue is, how is logic created in a markdown file?  I can't, but I can use a framework like  to Hugo parse markdown, process and created a site based on templates with the logic (ranging arrays, if, while etc) contained there. 

So I learned how Hugo works with templates of html with embedded go language snippets,  ingesting markdown front matter and content and parsing it to create a whole site of static pages.  But HOW I do I do the opposite and get simple markdown file to ingest go template logic??

#### Shortcode to the rescue.

Hugo is pretty fancy really,  they created a way to for snippets of logic and formatting to be embedded into markdown files....so the templates  can ingest that along with the markdown file.

Introducing [Hugo Shortcodes](https://gohugo.io/content-management/shortcodes/).  Hugo has list of built in shortcode templates of common items that get embedded in webpages, like YouTube videos, X tweets (Xcrements?) , Figure elements etc.  It also allows for the creation of custom shortcodes and includes very detailed document.  YAY!

**OK sleeves up...read, learn and do**

####  Not so Secret Source 

Ok so what I want is to have :
- The 4 most recent posts at the base of my homepage,
- I want them in card format (different to how I have them formatted in the list of articles),
- The whole card as the anchor link
- Article title on top with the front matter summery as the card content.
- The date the article was created and an indication of the read time.

This needs the shortcode logic with some html formatting, an inline call to the shortcode in the homepage markdown file, and finally some styling in CSS to finish it off.

**SHORTCODE**
First create the to shortcode.  This will need to live in  **(Hugo root)/layouts/shortcodes** directory. I create **layouts/shortcodes/recent_articles_cards.html** with the folloing snippet.  Notice the go templating language sprinkled amongst standard html along with the classes for styling in CSS.  After learning how to create my own Hugo website, I don't know why kept putting this aside, anyone with a basic level of scripting would follow what is going here. 

```html
<div class="latest-articles-cards">
  {{- range first 4 (sort .Site.RegularPages "Date" "desc") }}
  <a href="{{ .Permalink }}" class="post-card-link">
    <div class="post-card">
      <div class="post-title">
        <h3>{{ .Title }}</h3>
      </div>
      <p>{{ .Summary | safeHTML }}</p>
      <div class="card-meta">
        <div>
          <i class="fa-solid fa-calendar-days"></i>
          {{ .Date.Format "January 2, 2006" }}
        </div>
        <div>
          <i class="fa-solid fa-clock"></i>
          {{ .ReadingTime }} min read
        </div>
      </div>
    </div>
  </a>
  {{- end }}
</div>
```

**HUGO CONFIGURATION**
Following the [shortcode documentation](https://gohugo.io/content-management/shortcodes):
> An inline shortcode is a *shortcode* template defined within content.
> 
> Hugo’s security model is based on the premise that template and configuration authors are trusted, but content authors are not. This model enables generation of HTML output safe against code injection.
> 
> To conform with this security model, creating *shortcode* templates within content is disabled by default. If you trust your content authors, you can enable this functionality in your site’s configuration

To enable inline shortcode, create a security statement in the Hugo configuration file.  Mine is hugo.toml, there are other valid files depending if yaml or json was used,  I used toml.
```toml
[security]
  enableInlineShortcodes = true
```
It would be wise to peruse the documentation about the other [security options available](https://gohugo.io/configuration/security/) for configuration.  Also if cutting and pasting the shortcode then editing it for your own use, definitely read up on [safeHTML](https://gohugo.io/functions/safe/html/) that used when calling .Summary, there are pros and cons but I trust my own markdown files.

**CALLING THE CODE**
Shortcodes can be called using two different inline markdown notations , distinguished by their tag delimiters.
``` 
Markdown: {{% foo %}} ## Section 1 {{% /foo %}}
Standard: {{< foo >}} ## Section 1 {{< /foo >}}
```
Not to confuse things, this example is for if {{ .Inner }} is used in the short code, that is an action is being performed on `## section 1` in the markdown file.   My shortcode snippet is purely inline use only and I use standard notation, so I have the following simple call in my markdown where I want the "recent article cards" to appear on the homepage
```markdown
### Latest Articles:

{{< recent_articles_cards >}}
```
Thats is it.  Simply call the name of the shortcode file, sans .html, in the standard notation format.  

**Super Stylin'**
At this stage Hugo produces a homepage with 4 recent articles where I want them to appear with basic formatting and no styling.  Since I have created classes in the shortcode markup, I just need to create the CSS elements and give it all a style that fits with the page.  This is the fun stuff that I can lost.

After some playing around I have the following elements in my style.css file that I am happy with.
```css
/* recent article cards */

.latest-articles-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.post-card-link {
  text-decoration: none;
  font-weight: normal !important;
  color: #d8dee9 !important;
}

.post-card {
  background-color: #4c566a;
  padding: 10px;
  border: 1px solid #4c566a;
  border-radius: 2px;
  height: 100%; /* Ensures all cards are the same height */
  display: flex;
  flex-direction: column;
}

.post-card:hover {
  color: #d8dee9 !important;
  border: 1px solid peru;
}

.post-title {
  margin-top: 0;
  /* Or use a more specific selector for your title container */
  border-bottom: 1px solid #ccc;
  padding-bottom: 5px; /* Adds a little space below the line */
  margin-bottom: 5px; /* Adds space between the line and the content */
}

.post-card p {
  flex-grow: 1; /* Pushes the date to the bottom */
  border-bottom: 1px solid #ccc;
  padding-bottom: 5px; /* Adds a little space below the line */
  margin-bottom: 5px; /* Adds space between the line and the content */
}

.card-meta {
  display: flex;
  flex-wrap: wrap;
  margin-top: 10px;
  font-size: 0.8em;
  gap: 0.8rem;
}
```

**YESS!!**
Alright, I am happy with the outcome and once I took the time read the documentation and have a little play, it was not as bad as I though was going to be.  Now I can start looking at solving, and automating, a few other features I would like to implement.

All the source for this and my entire site can be found [here](https://github.com/mikewebbtech/mikewebbtech-hugo) in my git repo.  [Have a look](https://github.com/mikewebbtech/mikewebbtech-hugo/tree/main/layouts/shortcodes). 