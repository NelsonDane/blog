---
date: '2024-06-25T16:27:33-05:00'
draft: false
title: 'Contributing to Foss Doesn''t Have to Be Hard'
description: 'A guide to making small but meaningful contributions to open source projects.'
series: 'Open Source'
tags: ['foss', 'open-source', 'contributing', 'github', hi.events]
showToc: true
TOCOpen: true
---


## Introduction
Many people are interested in contributing to FOSS projects (Free and Open Source Software), but are unsure of how to start. They know that they want that shiny green square in their GitHub history, as well as their picture in the repository's contributors list, but learning a new codebase enough to fix a problem in it or add a new feature can be a daunting task. However, not all contributions need to be revolutionary. Any quality contributions make a maintainer's life easier, even if they're small. Today I'm going to talk about one specific instance where I made a small but meaningful contribution to a FOSS project, and how you can do the same.

## Finding a Project (And a Problem)
One way that I find new projects is by browsing the subreddit [r/selfhsoted](https://www.reddit.com/r/selfhosted/). I'm a big fan of self-hosted FOSS software, and that's when I saw a new [post](https://www.reddit.com/r/selfhosted/comments/1dabulx/i_built_an_opensource_event_ticketing_platform/) about a self-hosted event ticketing platform called [hi.events](https://hi.events/). I was intrigued, so I scrolled through the comments. One comment caught my eye: the commenter was having trouble building the project with Docker locally, as shown below:

![Reddit Comment](/posts/contributing-to-foss/reddit_comment.png "Reddit Comment")
Source: [Reddit](https://www.reddit.com/r/selfhosted/comments/1dabulx/comment/l7mci9p/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

## _Error: Unable to find /startup.sh_

Ah yes, I've seen this error before. As an enjoyer of Docker (but not Windows), I've come across this error before in both my personal and school projects. Contrary to what you might think from the error message, the problem is not that the file doesn't exist in the Docker container. The problem has to do with how the file was saved. Or more precisely, how it was automatically saved by Windows. The real issue is that the `startup.sh` file was saved using `Windows line endings` when the commenter cloned the repository, instead of with `Unix line endings` which is what the Docker container expects. This is especially clear because the error occured when the commenter attempted to build the project locally, instead of using one of the pre-built Docker images.

So then, what's the fix? As you might've guessed, it's actually quite simple. 