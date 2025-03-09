---
date: '2025-02-24T17:48:40-05:00'
draft: false
title: 'Purge Your Advent of Code Inputs and Get Off the Naughty List'
description: 'An easy way to remove your unique Advent of Code inputs from your git history.'
series: []
series_order: 0
tags: ['advent-of-code', 'git', 'github']
---

{{< lead >}}
*Now you won't get coal in your stocking, guaranteed!*
{{< /lead >}}

{{< github repo="NelsonDane/advent-of-code" >}}

## Introduction
Advent of Code is a wonderful way to spend the holidays helping virtual elves solve puzzles instead of spending time with your family. However, during this season of giving and sharing, there's one thing you should keep to yourself: your unique puzzle inputs. This is something I (and many others) didn't know when I first started participating in Advent of Code, so my git history was littered with my unique inputs. That's an easy way to end up on Santa's (or Eric's) naughty list.

*`...If you're posting a code repository somewhere, please don't include parts of Advent of Code like the puzzle text or your inputs.`* - [Advent of Code Legal Notice](https://adventofcode.com/2024/about#legal)

*`...No content at adventofcode.com (including the inputs) is licensed for reproduction or distribution without permission from Advent of Code.`* - [r/adventofcode](https://old.reddit.com/r/adventofcode/wiki/faqs/copyright/inputs)

## The Solution
Unfortunately, I didn't realize at the time that storing your inputs online was wrong. I happily pushed my solutions, inputs and all, to my GitHub, then went on my merry (christmas) way. It was years later that I started following the [Advent of Code subreddit](https://old.reddit.com/r/adventofcode/) and saw the warnings about sharing your inputs. The option I immediately thought of was to just nuke the repository and start fresh. But what about the stars I had gotten from my friends, or my long commit history? There has to be a better way!

And there was.

{{< youtubeLite id="w3zB-74JKEE" label="Jack Frost tricks Scott Calvin" >}}
We're gonna rewrite history like Jack Frost in *`The Santa Clause 3: The Escape Clause`*

Fortunately for us, we don't need a magic snow globe to fix this. We can use the `git filter-branch` command to rewrite our git history and remove the text files containing our inputs. This command is a bit dangerous, so make a backup of the repository before you start. However, it isn't as dangerous as angering an ancient man who watches you while you sleep.

```bash
git filter-branch --tree-filter 'rm -f */*/input.txt' HEAD
```
(From [this Reddit comment](https://old.reddit.com/r/adventofcode/comments/18an94z/psa_dont_share_your_inputs_even_in_your_github/kbzkow5/))

You might need to adjust the `rm -f */*/input.txt` part of the command to match the path of your input files. For me, all my inputs were named `input.txt` and were two directories deep (e.g. `2023/day1/input.txt`). If you have a different naming scheme, you'll need to adjust the command accordingly. If you want to see what files are picked up by the glob, you can run `ls */*/input.txt` to see the list of files that will be removed.

You did it! You rewrote history to the way it should've been. Now all that's left is to force push the changes to your remote repository. This will overwrite the history on the remote repository with your new, nice history. It was as if Jack Frost had never upt on the hat and coat to become Santa Claus to begin with.

### Conclusion
Now you can rest easy knowing that you won't be getting coal in your stocking this year. Just be sure to add `input.txt` to your `.gitignore` before next year, otherwise you'll be back on the naughty list like Buddy's dad:

![You see, Buddy, he's on the naughty list.](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExeTdqNHdjZDU1bjhsc3l4eHQ0aXNrbXIzZHJ3bjJrOHVnYmpxYXpydCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/QlculX9SlRm0/giphy.gif)
