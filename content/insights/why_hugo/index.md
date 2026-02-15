---
title: "The Architecture of My Digital Hub: Why I Built This Site"
date: 2026-02-14
description: "An insight into why I moved away from 'rented' platforms to build a high-agency home using Hugo and Blowfish."
series: ["Building the Hub"]
series_order: 1
tags: ["Hugo", "Engineering", "Web Development"]
---

I’ve always been a bit of a tinkerer, but for a long time, my digital presence was scattered across "rented" spaces. Whether it was social media or templated platforms, I was essentially a guest in someone else’s house. This site is my attempt to build a permanent, high-agency home for my engineering projects, my insights, and my journey towards China.

Here is the "How and Why" behind the build.

---

### The "Why": Ownership and Agency

The primary driver was a desire for digital ownership. I wanted a place that wasn't subject to the shifting sands of platform algorithms or subscription tiers. 

More specifically:
* **Freedom of Expression**: I needed a space to showcase technical skills and personal narratives without the constraints of a "pre-packaged" UI.
* **A Technical Playground**: I wanted to be "techy" and "nerdy" without having to develop a bespoke CMS from scratch. 
* **Sustainability**: A place that remains fast, secure, and accessible, even if I’m halfway across a continent with a spotty connection.

> **[Image Suggestion: A wide-angle shot of your bike fully loaded for the UK-to-China trip, symbolising the journey and the need for a "home base".]**

---

### Selecting the Engine: Hugo vs. WordPress

When deciding on the tech stack, the choice between a Static Site Generator (SSG) like Hugo and a dynamic system like WordPress was a short conversation. 

I chose Hugo for three distinct reasons:
* **Speed is a Feature**: Hugo pre-renders every page at build time. There is no database to query and no server-side processing when a visitor clicks a link. The result is a site that loads near-instantly, which is non-negotiable for a professional brand.
* **Inherently Secure**: WordPress is effectively a giant targets list for bots. By removing the database and the login portal, I’ve eliminated the vast majority of common web vulnerabilities.
* **The Markdown Workflow**: I prefer writing in Markdown. It’s clean, portable, and allows me to manage my content as code in a version-controlled environment rather than wrestling with a clunky visual editor.

---

### The Theme: Why Blowfish?

Choosing a theme can be a rabbit hole. I settled on **Blowfish** because it hits the sweet spot between a minimalist aesthetic and high-end technical features.

* **Tailwind CSS Power**: It’s built on Tailwind, making it incredibly lightweight and easy to customise without bloating the code.
* **Built-in Systems**: It includes native support for things I actually use, such as **Series** for my long-distance cycling logs and **Mermaid diagrams** for system architectures.
* **Authority, without Arrogance**: The design is "dark-first" and measured. It doesn't shout; it provides a clean canvas for complex ideas.

---

### The Tech Stack: Local to Live

My workflow is designed for efficiency and reliability. I manage the site using a professional IDE, version control it via GitHub, and automate the deployment through Cloudflare.

{{< mermaid >}}
graph LR
    A[Local Development] -->|Git Push| B(GitHub Repository)
    B -->|Webhook Trigger| C(Cloudflare Pages)
    C -->|Build & Optimise| D{Global CDN}
    D -->|Serve Site| E[Visitor]
{{< /mermaid >}}

* **The Local Environment**: This is where the heavy lifting happens. I draft my posts in Markdown and use the Hugo CLI to preview changes in real-time. It’s a grounded, local-first approach that ensures I’m never fighting with a web-based CMS.
* **The Source of Truth (GitHub)**: Every comma and configuration change is versioned. By treating content as code, I ensure that my journey is backed up and that I can revert any "experimental" errors with a single command.
* **The Distribution (Cloudflare Pages)**: The final leg of the journey. Once I push my code to GitHub, Cloudflare automatically builds the site and distributes it across its global network. This ensures that someone in London or Lanzhou gets the same snappy experience.

> **[Image Suggestion: A screenshot of your terminal or code editor showing a Markdown file on one side and the 'hugo server' local preview on the other.]**

---

### Closing Thoughts

Building this site wasn't about reinventing the wheel; it was about choosing the right tools to build a better one. By opting for Hugo and Blowfish, I’ve realised a platform that is fast, secure, and entirely under my control. 

It’s ready for the road.