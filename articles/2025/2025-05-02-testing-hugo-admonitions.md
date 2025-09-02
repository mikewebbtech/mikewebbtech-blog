---
date: 2025-05-02T10:49:58+08:00
draft: false
title: Testing markup callouts
summary: Testing out markup callout styles from external module and local shortcodes. As I navigate the ever-evolving landscape of technology, I’m always on the lookout for tools that are efficient, powerful, and enjoyable to use.
tags:
  - blog
  - testing
categories:
  - hugo
series:
---
Callouts, or *admonitions*, are styled content blocks used in documentation and blogs to highlight important information such as tips, warnings, or notes. Their purpose is to improve readability by drawing attention to key details. Advantages include clearer communication, reduced risk of overlooked information, and a more structured, professional presentation.

## Error Test

> [!NOSUPPORT]
> Helpful advice for doing things better or more easily.

## GitHub Test

> [!NOTE]
> Useful information that users should know, even when skimming content.
		
> [!TIP]
> Helpful advice for doing things better or more easily.
		
> [!IMPORTANT]
> Key information users need to know to achieve their goal.
		
> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.
		
> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.

## Callout Overview

> [!ABSTRACT]
> Abstract: This paper discusses the advantages and challenges of microservice architecture.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.

> [!CODE]
> Code snippet:
>
> ```javascript
> function fetchData() {
>   return axios.get("/api/data");
> }
> ```

> [!CONCLUSION]
> Conclusion: Based on the analysis above, we've decided to implement Docker containerization.

> [!DANGER]
> Danger! Critical security vulnerability detected in the system. Immediate action required.

> [!ERROR]
> Error: Unable to connect to database. Please check your connection settings.

> [!EXAMPLE]
> Example:
>
> ```python
> def hello_world():
>     print("Hello, World!")
> ```


> [!EXPERIMENT]
> Experiment: Testing the impact of new caching strategies on system performance.
		

> [!GOAL]
> Goal: Reduce service response time by 30% by the end of this quarter.

		
> [!IDEA]
> Idea: Implement a machine learning-based code quality detection system.
		

> [!IMPORTANT]
> Key information users need to know to achieve their goal.


> [!INFO]
> System status: All services are operating normally. Current uptime: 99.99%.


> [!MEMO]
> Memo: Technical review meeting scheduled for next Tuesday at 2:00 PM.


> [!NOTE]
> Useful information that users should know, even when skimming content.


> [!NOTIFY]
> System notification: Your password will expire in 30 days.


> [!QUESTION]
> Question: How can we optimize database query performance?


> [!QUOTE]
> "Code is like humor. When you have to explain it, it's bad." - Cory House


> [!SUCCESS]
> Congratulations! Your code has been successfully deployed to production.


> [!TASK]
> To-do list:
>
> - Update documentation
> - Deploy new version


> [!TIP]
> Helpful advice for doing things better or more easily.


> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

## Customisation

Adding a custom title to a callout style

> [!TIP] Summary
> This is a summary using the `TIP` callout!


> [!IDEA] Summary
> This is a summary using the `IDEA` callout!

## Header Only Mode

It is possible to only show the header!

> [!ABSTRACT] This paper discusses the advantages of microservice architecture


> [!CAUTION] Ensure all tests pass before merging to main branch


> [!CODE] Execute `npm install` to install all dependencies


> [!CONCLUSION] We've decided to implement Docker containerization


> [!DANGER] Critical security vulnerability detected in the system


> [!ERROR] Error: Unable to connect to database. Please check your connection settings


> [!EXAMPLE] Git commit message format: "feat: add user authentication"


> [!EXPERIMENT] Testing new caching strategy with Redis


> [!GOAL] Reduce service response time by 30% by the end of this quarter


> [!IDEA] Implement a machine learning-based code quality detection system


> [!IMPORTANT] Please review and update your security settings


> [!INFO] Current system status: All services are operating normally with 99.9% uptime


> [!MEMO] Technical review meeting scheduled for next Tuesday at 2:00 PM


> [!NOTE] Always backup your data before performing system updates


> [!NOTIFY] System notification: Your password will expire in 30 days


> [!QUESTION] How can we optimize database query performance?


> [!QUOTE] "Code is like humor. When you have to explain it, it's bad." - Cory House


> [!SUCCESS] Congratulations! Your code has been successfully deployed to production


> [!TASK] Review and update API documentation by Friday


> [!TIP] Use `Ctrl + C` to quickly terminate a running program


> [!WARNING] Warning: This operation will delete all data


## Foldable Admonitions

> [!NOTE]- Here are the details regarding API usage:
>
> - **Endpoint**: `/api/v1/users` is used to fetch the user list.
> - **Authentication**: A valid `Bearer` token must be provided in the request header.
> - **Rate Limiting**: 100 requests are allowed per minute.


> [!TIP]- Click here to view the tips

## Nested Admonitions

> [!question] Can admonitions be nested?
>
> > [!todo] Yes!, they can.
> >
> > > # [!example] You can even use multiple layers of nesting.


### Some of the callout styles available for use and examples of thier rendering

> > > > > > > 98cb3b2 (added style and overflow to codeblock render)
> > > > > > > [!NOTE]
> > > > > > > Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.


> [!IMPORTANT]
> Key information users need to know to achieve their goal.


> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.


> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.


> [!WARNING]+ Radiation hazard
> Do not approach or handle without protective gear.


> [!IDEA] IDEA
> This is a summary using the `IDEA` callout!


> [!MEMO] MEMO
> This is a summary using the `MEMO` callout!


> [!TIP] You can choose to only to show the header!


> [!NOTIFY] System notification: Your password will expire in 30 days


> [!SUCCESS] Congratulations! Your code has been successfully deployed to production


> [!WARNING] Warning: This operation will delete all data.


> [!TIP]- Click here to view the tips
> This is a closed tip


> [!TIP]+ Click here to view the tips
> This is the tip YAY


> [!CODE]
> sample code
> here

## Using custom short code and styling

> This is a note.
> {.note}


> This is a warning.
> {.warning}


> This is dangerous.
> {.danger}