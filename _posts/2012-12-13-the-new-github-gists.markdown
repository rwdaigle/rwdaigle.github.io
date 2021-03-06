---
layout: post
title:  "The New Gist: What It Is and What It Could Be"
date:   2012-12-13
author: ryan
tags:
  - github
category: blog
blog: true
---

[Gist](http://gist.github.com) is an incredible tool by Github for quickly sharing code, text and files. It has syntax highlighting and rendering for a huge number of programming languages including Markdown for text. For many techies, including myself, Gist is an indispensable tool for quickly sharing code and content with coworkers.

Gist has been around for several years now and, when compared with the pace of development on the main Github.com property, has been relatively neglected. Thankfully, Github [recently updated Gist](https://github.com/blog/1276-welcome-to-a-new-gist) with a fresh new codebase and UI. As a heavy user of Gist I have some thoughts on this update, where it hits the mark and where it's still lacking.

## Search

Search has long been sorely needed in Gist. It is not uncommon for power users to have several hundred to thousands of gists and the previous linear list-view based on creation date was inadequate. Immediate recall of a gist based on a search query was the primary use-case I had in mind when creating [Gisted - a tool to quickly search and access all your gists](https://gistedapp.herokuapp.com). So, seeing a native Gist search feature was very welcome for me.

![](http://f.cl.ly/items/1J0V2s471k2n0n0M2V3Y/Image%202012-12-12%20at%205.05.36%20PM.png)

Unfortunately, it leaves a bit to be desired. Firstly the search is case *sensitive*. So searching for `proposal` is not the same as `Proposal`. When searching my gists I never care about the case and want to just quickly find the most relevant gist containing that term. Fortunately, I imagine this to be a very easy fix on Github's end and expect it will be remedied shortly (based on nothing but my intuition).

### Indexing

However, more fundamentally, search seems to only apply to the description of your gist (gists don't have titles - the closest thing is what is labeled as the description). While I try to be very conscious of creating meaningful descriptions, when I search for them I often use some distinct term from within the gist content itself. Searching only based on description is like searching Google only based on the title of the web page.

Consider the results from the new gist search for a term I know exists: `dev`

![](http://f.cl.ly/items/1s2n3z0i0u0D3k1a2a06/Image%202012-12-12%20at%205.13.40%20PM.png)

Only one result? I think not. Now against descriptions and file contents:

![](http://f.cl.ly/items/1D0R3e023C0s1x3G1L3R/Image%202012-12-12%20at%205.15.22%20PM.png)

There are lots of relevant results missed by Gist's search mostly due to the lack of content indexing. Search can be a powerful utility for Gists but it needs some indexing refinement yet.

### Advanced operators

Relevant but basic search is a must-have for most users. Search with filtering and other operators is a must-have for power users. For instance, filtering by owner is a great way to quickly list the gists you've starred by others. I like this implemented with the `@` prefix notation and use it frequently:

![](http://f.cl.ly/items/081v1N2W0n2C2i3r1t2g/Image%202012-12-12%20at%208.18.31%20PM.png)

The new Gist doesn't seem to have filtering or any advanced features like phrases (`"exactly this"`) or operators (`AND`, `-`). These tend to be built-in features of any search index so I imagine Github is easing into search and will turn these on once they feel comfortable with the infrastructure.

## Lists

In the old Gist you really only had one way to view your gists. In a list of your created gists or your starred gists, ordered by when they were created. This was incredibly limiting. The new gist ushers in several refinements including:

* The ability to sort gists by their updated date. However, this value is not sticky so I find myself always having to select it when I just want it as my default.
* A much better partial rendering of each gist in the list, allowing you see more of the gist to know which one is the one you're looking for.

These are just a few of the things that make using Gist a little more enjoyable than the old interface.

## Revisions

Although gists have always been backed by a full git repo you didn't see much of that benefit in the web UI. You had to clone the repo locally to see version diffs and manually fetch other remotes to compare and merge forked versions.

The new Gist takes a small step to solving this puzzle, allowing you to view diffs between your gist's revisions.

![](http://f.cl.ly/items/1W2O1A3D3W1b322x2R1J/Image%202012-12-12%20at%208.47.55%20PM.png)

However, it still doesn't provide an easy way to view diffs or perform merges across forks. These are key collaboration features that would remove significant friction from using gists now. I can only hope Github is seriously thinking about enhancing this aspect of the product.

## Content focus

The new Gist has a somewhat awkward focus on the gist files rather than the descriptions. I say awkward because, while I appreciate the direction of putting the content front and center, there are some artifacts that betray the intent.

For instance, when viewing a gist in a list it's the filename of the gist that gets top billing:

![](http://f.cl.ly/items/242h231D36362b0C2J2c/Image%202012-12-12%20at%208.57.37%20PM.png)

However, gists can have multiple files making it an odd decision to choose only one (the first) to key off of.

Additionally, if files/content are the focus, they should be a first-class citizen in search but are instead ignored (as previously discussed).

## Still missing

Some miscellaneous features I was hoping would be added in the new Gist include:

* A resurrection of comment notifications! [Gisted](https://gistedapp.herokuapp.com) will do this for you, but it really should be a natively supported feature.
* Markdown editing and rendering parity with Github proper. If you look at your [standard project README](https://github.com/rwdaigle/miyagi#miyagi) on Github you'll notice you can edit the Markdown inline with decent highlighting, preview your content and, when rendered, sections are automatically anchored. Markdown is such a core feature of text-based collaboration that parity here is essential.
* A de-emphasis of the public gist-stream (now called "Discover Gists"). I just don't see the value of randomly browsing new gists and think that real estate could be better used.

## Summary

The new Gist is definitely an improvement over the old one. However I find it mostly just polishes existing features and doesn't directly address some of larger issues.

I would encourage Github to focus of the main uses of Gist. From my perspective, gists are used mainly as a collaboration tool. While they're backed by a full git repo, that is mostly an implementation detail. Commenting, managing collaborator modifications, and finding gists across several sources should be well supported use-cases.

I suspect we'll see the pace of development on Gist quicken now that a new codebase is in place. Removing technical debt often removes roadblocks that may have prevented a product from evolving. I can only hope if I revisit this post several months from now I'll have to significantly edit some of my more critical points.

Github has been incredibly supportive in my use of the [Gist API](http://developer.github.com/v3/gists/) and my work with them on this front only reinforces their developer-focused reputation. They'll get Gist right, it's just a matter of time.

<p class="note" markdown="1">
Given my dependence on Gist for work I have a vested interest in its success. Any critical points made here were done so only in hopes of seeing it evolve into a better product.
</p>
