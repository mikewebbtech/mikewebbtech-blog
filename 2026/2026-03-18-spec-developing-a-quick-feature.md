---
date: 2026-03-18T00:00:00+08:00
draft: false
title: Using Spec Driven Devlopment for a Quick Feature with AI
summary: After having my feed bombared with the power of AI-assisted coding using spec driven devolpment and this being a more rational approach to outcome success vs the slop of "vide coding". I thought I would examin its use to implement a specific Hugo feature I wanted to create for my site, from requirements to styled solution, in minutes.
tags:
  - howto
  - AI
categories:
  - hugo
series:
  - my-hugo-site
---

When working on  my blog site, I often find myself with specific feature ideas but limited time for the research and implementation required to make them a reality.   This is where AI assisted coding, or "spec coding," I guess, can become a transformative force, enabling me to move from concept to complete solution rapidly.

Recently,  I wanted a feature for each post to have a direct, link at the bottom of that would open the raw markdown source file for that article, hosted in my blog content repository on GitHub.

## The Challenge
**Bridging Project Structure and Repo Layout**

My setup is relatively standard for Hugo:
1.  **Site Project:** My main Hugo site code (layouts, static files, configs) is in one repository (`mikewebbtech-hugo`).
2.  **Content Repository:** My actual blog articles (`.md` files) live in a separate repository (`mikewebbtech-blog`), organized into folders by year (e.g., `2015/`, `2025/`).
3.  **Local Development:** Locally, within my main project, the articles reside under `content/articles/` (e.g., `content/articles/2025/`).
4.  **GitHub Actions:** I use GitHub Actions to build the final site, which pulls from both repositories.

The core issue lay in creating a single, consistent link URL when my local Hugo project structure (`content/articles/2025/myfile.md`) didn't match the final content repository structure (`2025/myfile.md`).  Simply using Hugo's built-in `.File.Path` would normally solve this issue, but it would give the local path relative to `content/`) and would include the `articles/` prefix, leading to a broken GitHub link.  I know Hugo has a string manipulation function called `replace`, I have used it before , but I couldn't remember the correct syntax

Also, I wanted the link to GitHub to be for the **plain text** version of the file, avoiding GitHub's rendered preview.

### The Spec Driven Development Approach
This is putting the effort into expressing the 'What' I want to achieve.  I usually start out with mapping out a plan for, well I guess, my *sprint* if looking at like a dev project.  Creating a mini PRD (Product Requirement Document).  This keeps me focused and away from 11:00pm rabbit holes.

Instead of spending time reading through Hugo documentation on file path manipulation syntax etc, I used AI to "spec code" the feature.  I clearly defined my requirements and constraints:

* **Goal:** Add a styled link to the raw Markdown file on GitHub at the bottom of each article.
* **Source:** A GitHub repo URL (`"https://github.com/mikewebbtech/mikewebbtech-blog/blob/main"`).
* **Constraint 1:** The local path contains "articles/" which needs to be removed.
* **Constraint 2:** I needed to append `?plain=1` to the final URL.
* **Aesthetic:** The icon should be styled with my sites accent colours (`#bf616a`).

## AI Driven Implementation in Minutes

Based on the clearly (IMO) defined specification, the LLM I used was able to immediately "synthesis" the requirements and provide the three parts of the  solution that solved the logic and styling challenges in one go...well a best effort, but maybe thats my limited "prompt-fu" skill.

## 1. Configuration (hugo.toml)
First, we defined the base repository URL as a site parameter, making it easy to update later if needed.

```toml
[params]
  # Base URL for the blog content repository on GitHub
  # This is what the AI produece to add to my cofig file...close but garbage 
  # blog_repo_url = "[https://github.com/mikewebbtech/mikewebbtech-blog/blob/main](https://github.com/mikewebbtech/mikewebbtech-blog/blob/main)"
  # This is the correct paramater
  blog_repo = "https://github.com/mikewebbtech/mikewebbtech-blog/blob/main"
```
## 2. Layout an Logic
For  the `layouts/_default/single.html` template, the AI provided the Go template logic to perform the string replacement and build the dynamic URL.  However, this is where knowing how to achieve the results by hand is import.  The way .File.Path works, it uses the Hugo project directory path that it ingests the article markdown files from (`articles`),  this would create a malformed URL resulting in 404 from GitHub.  This was an easy fix, I just created another string variable  that  used `replace` to strip "articles/" from `.File.Path` and then appended the required parameters.  

```html
  {{ if .File }}
    {{/* Strip the 'articles/' prefix and create the GitHub URL */}}
    {{ $repoPath := replace .File.Path "articles/" "" }}
    
    <div class="post-source-link">
      <i class="fa-brands fa-github"></i> 
      <a href="{{ .Site.Params.blog }}/{{ $repoPath }}?plain=1" target="_blank" rel="noopener">
        View source on GitHub
      </a>
    </div>
  {{ end }}
```

This code snippet for the configured base URL was 95% there.  I just had to clean up thedynamic file path, and and also  add  the  `?plain=1` query parameter, so the resulting link was accurate for every single article.

#### 3. Styling 
Finally, the AI provided the necessary CSS for my style.css to style the new elements, specifically targeting the GitHub icon with my theme's distinctive Nord red (`#bf616a`), maintaining a consistent look with my site.  Oh and and it included inline comments...cool.

```css
/* Container for the source link, aligning elements */
.post-source-link {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.9rem;
  color: #d8dee9; /* Nord Snow Storm */
}

/* Targeting the specific red accent color for the icon */
.post-source-link i {
  color: #bf616a; /* Theme Red */
  font-size: 1.1rem;
}

/* Subtle styling for the link itself with hover effects */
.post-source-link a {
  color: #81a1c1; /* Nord Frost Blue */
  text-decoration: none;
  transition: color 0.2s ease;
}

.post-source-link a:hover {
  color: #88c0d0; /* Lighter Nord Blue */
  text-decoration: underline;
}
```

## The Power of Quick Spec Coding
By utilising AI for this specific, **well-defined coding task**, I was able to quickly implement a fully styled, and feature framework in a matter of minutes from a "wouldn't it be cool to have raw source links" idea.  I didn't have to struggle with remembering the string manipulation how to's in Go templates or  CSS flexbox for alignment.  The LLM understood the 'what' and provided the very good but "not quit there" solution instantly. 

Was it perfect and would I lean on it produce a finished product? No.  However it did the heavy lifting allowing me to focus my limited time to fleshing out the missing bits and deploying my mini feature, rather than getting bogged down in implementation details.  I can see how for developers or project manager, spec coding with AI is a powerful *force multiplier* for rapid feature delivery.