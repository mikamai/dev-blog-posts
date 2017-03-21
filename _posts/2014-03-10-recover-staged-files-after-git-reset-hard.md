---
ID: 274
post_title: 'Recover staged files after git reset &#8211;hard'
author: Gianluca
post_date: 2014-03-10 08:54:34
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/10/recover-staged-files-after-git-reset-hard/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/79151552147/recover-staged-files-after-git-reset-hard
tumblr_mikamayhem_id:
  - "79151552147"
---
<p>Ok you want to get rid off of your unstaged file in a git repo.</p>
<p>Nice:</p>
<pre>git add .  
git reset --hard</pre>
<p>But wait&hellip;what?&hellip;no!!! Some of the unstaged files were actually necessary! They had to be in <code>.gitignore</code>! What about now?!</p>
<p>Don&rsquo;t despair and type:</p>
<pre>git fsck --lost-found</pre>
<p>Now you can go in the newly created directory <code>.git/lost-found/other/</code> and find all the files that were deleted by the destructive <code>git reset --hard</code>.</p>
<p>The only problem is that their original names aren&rsquo;t retained :(</p>
<p>Anyway if you remember the content of the files that should be actually retained you can check manually each one of those in <code>.git/lost-found/other/</code> and copy them back in their original location.</p>