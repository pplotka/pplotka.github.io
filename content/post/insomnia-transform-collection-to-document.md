---
title: "Insomnia - transform Collection to Document."
date: 2021-07-20T15:10:00+02:00
draft: false
categories: 
  - Tools
tags:
  - tools
  - every day issues
---

{{< callout emoji="💡" text="This post comes from a series of short tips, where I publish brief stories on how I resolve every day small (and sometimes easy) issues." >}}

Some time ago, Insomnia added a new type of project called Document Design. It allows you to group and share API specifications, requests, and unit tests in one application. 

The implementation of this change has brought forth the following result: there are now two types of workspace - **Design Document** and **Request Collection**. Having run the research, I discovered that the new features may be attractive for me and might improve my work with APIs. This has encouraged me to try a new type.

However, I came across one problem: how to transform one project type into another. A built-in feature is nowhere to be found. So, the first thing I did was create a ticket to support. The answer came in immediately (big thanks to Insomnia's support). Next, following the support’s suggestion, I created the [issue on Github](https://github.com/Kong/insomnia/discussions/3833). But…I needed solutions for that moment. Thus, I compared two import files and found certain differences, the result of which I‘m presenting the instructions below. 

When you have a project of Request Collection type in Insomnia and you’d like to transform it into Document Design type, you must follow these steps:

1. Export current project (Request Collection) with `Insomnia v4 JSON` format.
2. Remove this project from Insomnia.
3. Open the exported file in your favorite text editor. 
4. Find JSON section where parameter `_type` has value `workspace`.
5. Change the value of parameter `scope` from `collection` to `design`.
6. Import (go to **Application** -> **Preferences** -> **Data** -> **Import** -> **From File**) this document as the new project. 