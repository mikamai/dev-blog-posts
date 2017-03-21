---
ID: 315
post_title: Save the (Git) Environment
author: Matteo Zobbi
post_date: 2013-11-26 09:01:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/11/26/save-the-git-environment/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/68150265390/save-the-git-environment
tumblr_mikamayhem_id:
  - "68150265390"
---
As we use feature branching in our day-to-day workflow, we’ve got lots of branches left behind after work was complete. <!--more-->Mainly code that’s already merged into master…and so I’ve written a simple ruby script that:

1 Checks current branch
2 Warns if the current branch is not master
3 Prunes remotes
4 Checks merged branches
5 Shows the list of branches that will be removed
6 Asks for a confirmation to delete these branches

They should be safe to delete!

Here is the script: <a title="Clean Branches" href="https://github.com/mapteo/clean-branches">https://github.com/mapteo/clean-branches</a>