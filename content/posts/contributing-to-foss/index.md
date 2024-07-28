---
date: '2024-06-25T16:27:33-05:00'
draft: false
title: 'Contributing to Foss Doesn''t Have to Be Hard'
description: 'A guide to making small but meaningful contributions to open source projects.'
series: ['Open Source']
series_order: 1
tags: ['foss', 'open-source', 'contributing', 'github', hi.events]
---


## Introduction
Many people are interested in contributing to FOSS projects (Free and Open Source Software), but are unsure of how to start. They know that they want that shiny green square in their GitHub history, as well as their picture in the repository's contributors list, but learning a new codebase enough to fix a problem in it or add a new feature can be a daunting task. However, not all contributions need to be revolutionary. Any quality contributions make a maintainer's life easier, even if they're small. Today I'm going to talk about one specific instance where I made a small but meaningful contribution to a FOSS project, and how you can do the same.

## Finding a Project (And a Problem)
One way that I find new projects is by browsing the subreddit [r/selfhsoted](https://www.reddit.com/r/selfhosted/). I'm a big fan of self-hosted FOSS software, and that's when I saw a new [post](https://www.reddit.com/r/selfhosted/comments/1dabulx/i_built_an_opensource_event_ticketing_platform/) about a self-hosted event ticketing platform called [hi.events](https://hi.events/). I was intrigued, so I scrolled through the comments. One comment caught my eye: the commenter was having trouble building the project with Docker locally, as shown below:

![Reddit Comment](/posts/contributing-to-foss/reddit_comment.png "Reddit Comment")
Source: [Reddit](https://www.reddit.com/r/selfhosted/comments/1dabulx/comment/l7mci9p/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

## _Error: Unable to find /startup.sh_

Ah yes, I've seen this error before. As an enjoyer of Docker (but not Windows), I've come across this error before in both my personal and school projects. Contrary to what you might think from the error message, the problem is not that the file doesn't exist in the Docker container. The problem has to do with how the file was saved. Or more precisely, how it was automatically saved by Windows. The real issue is that the `startup.sh` file was saved using `Windows line endings` when the commenter cloned the repository, instead of with `Unix line endings` which is what the Docker container expects. This is especially clear because the error occured when the commenter attempted to build the project locally, instead of using one of the pre-built Docker images.

So then, what's the fix? As you might've guessed, it's actually quite simple. While the commenter (and anyone else who encounters this issue) could manually change the line endings of the file, it's a much better solution to automatically convert the line endings when the container is built. This way, the issue is fixed everywhere, and not just on the commenter's machine. Thankfully, using Linux (which is what the Docker container is based off of), this is quite easy to do using the `dos2unix` command.

## The Pull Request
Now that I knew how to fix the issue, I checked the project's `issues` to see if anyone else was working on it. You'd hate to do the work of fixing it just to see someone else had already beaten you, or if a fix was already planned. After finding one issue referencing the bug, I commented and said that I knew how to fix it and would create a pull request soon. So that's exactly what I did: I forked the repository and opened it in my editor. After searching for `startup.sh`, I found the Dockerfile that built that part of the container (this particular project uses a multi-stage build). Then I found where `startup.sh` was `chmod`ed, and added the `dos2unix`. And luckily enough for me, it came pre-installed in the base image so I didn't even need to install it. The diff is shown below:

![Pull Request Diff](/posts/contributing-to-foss/diff.png "Pull Request Diff")
Source: [GitHub](https://github.com/HiEventsDev/hi.events/pull/31/files?diff=unified&w=0)

After testing the change locally in a Windows VM and confirming that the line endings were fixed, I opened a [pull request](https://github.com/HiEventsDev/hi.events/pull/31). Being sure to reference the GitHub issue, as well as the Reddit comment, I explained the problem and my proposed solution. I also double checked the diff to make sure that I didn't change anything else by accident. Then I pressed submit, and waited.

## Conclusion
As you can see, not every Pull Request needs to be a massive feature or critical bug fix. It's just a matter of finding a problem, sometimes one that you've encountered before, and contributing a fix. In my case, it was an issue I had faced before, so I could quickly identify the problem and solution. This saves the maintainers time, and helps the project grow. Perhaps other people who encountered the same issue would've just given up on the project entirely, or people who have more expertise than me would've gotten bogged down trying to fix a small bug instead of add new features. But that's the beauty of open source: everyone has faced different problems and have had many different experiences. By sharing these with the community, we can all grow together.

And much to my delight, the maintainer of the project merged by Pull Request not too long after, and now I'm an official contributor to the project. Hopefully this post has helped you see that contributing to FOSS doesn't have to be difficult, and that even the smallest PRs can make a difference. So next time you come across a project and see a way you can help, don't hesitate to make a PR. You never know how much it might help someone else.
