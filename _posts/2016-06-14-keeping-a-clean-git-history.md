---
ID: 25
post_title: Keeping a clean git history
author: Nicola Racco
post_date: 2016-06-14 13:08:27
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/06/14/keeping-a-clean-git-history/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/145908219509/keeping-a-clean-git-history
tumblr_mikamayhem_id:
  - "145908219509"
---
Many think that our work as developers is not to keep a clean git history, but to deliver valuable code, so usually nobody cares about this matter **enough** to discuss with their team the definition of "clean history" and how to obtain it.
<!--more-->

For example I bet this is a common `git log` output:

```text
7449971 add styling in email
bc4fd7e fixes in the account creation form
647e5a1 fix for params in emails
be126dc prepared new email
...
```

Look at it in six months, it will look like the following:

```text
...
7449971 blablablablabla
7449971 blabla
647e5a1 blablablablablablablablabla
be126dc blablablablabla
...
```

This is when the team should ask themselves **"Do we need to manage our git history a little bit better?"**

There are indeed some reasons why you wouldn’t need a clean repo history. For example when the project has a lifespan of a few days. But in all other cases, when multiple devs are working on a project that is supposed to live for a year or more, it should be a moral duty to care about repo history.

This may seem useless when the project is started because as the project moves forward no one needs to look back. But problems will arise soon enough, when you may need to rollback a feature added two months ago, or when someone asks why a certain feature has been removed long time ago. In these cases the ability to look back to some months before and clearly understand what was going on at the time can save hours, if not days.

There are two things to consider: taking care of the history should be a commitment of every member of the team (so everyone should agree on its importance) and, even before you can start worrying about it, the team should have a clear work method.

So, let's get to the point: how we work as a team and manage to keep our repo history clean.

We usually have a small team of 4-5 devs working on a project, following an agile methodology. Usually every story is developed by one or two developers. When a story is executed by two of them, they'll work in pair.
Every story is developed in a separate git branch. And a story will always deliver a feature, a value for the user. Bugs are fixed in another story that spans for the entire iteration.
Once the code is written and tests are green, a `git rebase -i origin/master` is mandatory. This will allow the devs to squash, move, edit the commits. As an example, let’s say the dev just completed the story _Given I am a guest, When I create a user account, I want to receive an email_. The `rebase -i` will print the following commits:

```text
pick be126dc prepared new email
pick 647e5a1 fix for params in emails
pick bc4fd7e added user model
pick acd12bb added user creation form
pick ccc987c send email on user creation
pick 7449971 add styling in email
```

The developer will change it this way:

```text
reword bc4fd7e added user model
fixup acd12bb added user creation form
reword be126dc prepared new email
fixup 647e5a1 fix for params in emails
fixup 7449971 add styling in email
fixup ccc987c send email on user creation
```

As you can see, they changed the commits order. Two commits will be renamed (reword), while the others will be merged with the previous ones.
Once the dev saves this buffer, git will reapply the commits in the order the dev specified, and <code>git log</code> will now print:

```text
bc4fd7e User creation form
be126dc user_created email sent on user creation
```

The log of the story branch is now clean. There are only two commits, each of them is expressing a specific feature.
The next step is the merge in the master branch. A simple merge would result in a log like the following:

```text
abc123c Merge branch &#039;user_and_mail&#039;
bc4fd7e User creation form
cde456f Merge branch &#039;master&#039;
be126dc user_created email sent on user creation
12acd23 Merge branch &#039;bugfixes&#039;
...
```

Not a great result. For this reason, every story should correspond to a commit when it reaches master.
And to do this, each pull request to master is then accepted with a squashed merge. This type of merge will squash all commits in one single commit that is then applied in the destination branch. So, sticking to our `user_and_mail` branch, the developer will do a `git merge --squash user_and_mail` and will enter the following as the commit message:

```text
Allow users to create an account
* User creation form
* user_created email sent on user creation
```

As you can see, the single messages of the original commits have become an additional description of this commit. Doing the following for every branch will produce something like:

```text
abc123c Allow users to create an account
12acd23 Bugfixes on something
```

While doing a `git show abc123c` will show the full message provided during the squashed merge.

It may seem tricky at the beginning, but after some time the results will be priceless.