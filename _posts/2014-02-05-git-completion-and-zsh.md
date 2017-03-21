---
ID: 289
post_title: Git completion and zsh
author: Andrew Coates
post_date: 2014-02-05 09:55:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/05/git-completion-and-zsh/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/75679082254/git-completion-and-zsh
tumblr_mikamayhem_id:
  - "75679082254"
---
<p>I use <a href="http://www.zsh.org">zsh</a> out of habit and reasons highlighted <a href="http://www.slideshare.net/jaguardesignstudio/why-zsh-is-cooler-than-your-shell-16194692">here.</a> <br /><br /> One problem I had with zsh was Git completion. <a href="https://github.com/git/git/blob/master/contrib/completion/git-completion.bash">git-completion.bash</a> is the script that handles the completion. Place it in your home directory and source it in your .bashrc/.zshrc: <br /><br /> Although this didn’t work for zsh users. Fix this by using the <a href="https://git.kernel.org/cgit/git/git.git/tree/contrib/completion/git-completion.zsh">git-completion.zsh</a>, placing it into</p>
<pre>'~/.zsh/_git</pre>
<p>and sourcing it</p>
<pre>fpath=(~/.zsh $fpath)</pre>
<p>in .zshrc. <br /><br /> If you have any problems with this, check out <a href="https://github.com/robbyrussell/oh-my-zsh">oh-my-zsh</a>, an OS X fork of zsh which includes neat things like git completion, git colours, and prompt features that help with command line git, such as working branch, RVM environment, dirty/clean branch indicator etc. <br /><br /> Although, keep in mind that one complaint many people have with oh-my-zsh is that it is bloated.</p>