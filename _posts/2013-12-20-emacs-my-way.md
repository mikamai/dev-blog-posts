---
ID: 300
post_title: Emacs My Way
author: Giovanni Intini
post_date: 2013-12-20 14:15:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/20/emacs-my-way/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/70585280775/emacs-my-way
tumblr_mikamayhem_id:
  - "70585280775"
---
<p>Everyone knows <a href="http://www.gnu.org/software/emacs/">Emacs</a> is a way of life.</p>
<p>You start learning it the first day and you never stop learning. If you think you&rsquo;ve learned it all then you&rsquo;re doing it wrong.</p>
<p>I started reading about Emacs when a funny joke about it was &ldquo;EMACS: Eight Megabytes and Constantly Swapping&rdquo;, but I never thought I would eventually get into it.</p>
<p>About twenty years have passed before I decided to give it a try, and since then I had to spend around two full years memorizing key chords. I am a happy user now, and here&rsquo;s some tips to shorten your learning path :)</p>
<p><img alt="image" src="http://68.media.tumblr.com/a0a04dde3bbb2e8af184ef1ec88df3ec/tumblr_inline_my3xllxZER1r11frw.jpg" /></p>
<p>I will assume you have your standard Emacs installation working. I use homebrew on OSX to install it with:</p>
<pre>    brew install emacs --HEAD --use-git-head --cocoa --srgb
</pre>
<p>Once you do that make sure it runs, and then clone <a href="https://github.com/intinig/emacs.d" title="intinig/emacs.d">my emacs.d repo</a>. Save it as ~/.emacs.d, launch emacs and you&rsquo;re almost done :)</p>
<p>It will take a while installing several libraries and it might require you a couple of Emacs reboots to finish installing everything.</p>
<p>Since you&rsquo;re here reading and waiting that Emacs is done doing its magic stuff let&rsquo;s discuss how my setup works.</p>
<p>Everything is based on <a href="https://github.com/dimitri/el-get" title="dimitri/el-get">el-get</a>. El-get is a kind of meta-package-manager for Emacs.</p>
<p>It allows you to install and manage packages from multiple sources. Package.el, Emacswiki, github, you name them!</p>
<p>El-get also allows you to quickly whip up your recipes for forked or newer version of packages, very useful (check ~/.emacs.d/el-get-user-recipes/autotest.rcp to check one).</p>
<p>El-get also has a way to neatly package custom init files for the packages you install with it (check ~/.emacs.d/el-get-user/init). </p>
<p>I use el-get to install all the different packages I use in my daily routine. As of today, but it varies wildly with time, I use:</p>
<ul><li>ag, emacs frontend to the silver_searcher, blazing fast search in your projects. AMAZING!)</li>
<li>autotest, to continously run your test suite in a buffer</li>
<li>bundler, to run bundler commands from emacs, kinda quirky for now</li>
<li>coffee-mode, for your Coffeescript needs</li>
<li>gist, very useful when there&rsquo;s a snippet you want to share with the outside world right now</li>
<li>go-mode, because the cool guys program in Go now, don&rsquo;t they?</li>
<li>ido-ubiquitous, extends the <strong>essential </strong><a href="http://www.emacswiki.org/emacs/InteractivelyDoThings">ido-mode</a> to every type of minibuffer query</li>
<li>magit, because once you learn to use git from emacs you won&rsquo;t ever go back to the terminal anymore</li>
<li>markdown-mode, for your README.md :)</li>
<li>paredit, when you want to hack lisp dialects you also want your parentheses not to get in your way</li>
<li>php-mode, no comment here</li>
<li>powerline, to improve the look and feel of your status line</li>
<li>rspec-mode, run specs from emacs, get back results that are hotlinked to the sources. Very useful.</li>
<li>ruby-compilation, run scripts and rake and other Ruby stuff from Emacs</li>
<li>rvm, allows you to switch ruby versions and gemsets from the editor</li>
<li>smex, similar to ido-mode, for M-x commands</li>
<li>yaml-mode, sooner or later you&rsquo;ll need to edit yaml files too :)</li>
<li>zenburn-theme, for a color theme that improves your health</li>
</ul><p>As you can see from my packages, I am mostly a Ruby developer, but you can easily switch, remove and add to the packages you use by using M-x el-get-install, or edit ~/.emacs.d/init.el.</p>
<p>In addition to init.el and the el-get-user folder, there are four more files in my installation:</p>
<ul><li>functions.el, here you can find some functions I defined for general usage. I am moving most of the things you can find here in el-get-user/init</li>
<li>defaults.el sets up most of your environment. You know, defaults :)</li>
<li>languages.el holds configuration for programming languages major modes. I am also moving code away from here and into their el-get-user/init files</li>
<li>system.el contains information about your system. You should for sure edit the following line to match your system, or lots of things won&rsquo;t work</li>
</ul><pre>    (if (not (getenv "TERM_PROGRAM"))
    (setenv "PATH"
            (shell-command-to-string "source $HOME/.bash_profile &amp;&amp; printf $PATH")))

</pre>
<p>In addition to using lots of packages to help your daily endeavor, there are some defaults I put it to ease the transition from other <strike>less powerful </strike> younger editors (I used to use <a href="http://macromates.com">Textmate</a>).</p>
<p>Remember, you will get lost in Emacs at first, and you will panic. Don&rsquo;t! Just remember that C-x C-c allows you to quit and try again :)</p>
<p>Once your muscle memory wraps around the key chords you need for your programming needs, you&rsquo;ll be a better person, and a happier programmer.</p>
<p>And you&rsquo;ll also love Emacs, like I do ;)</p>
<p><img alt="Kim Jong Un approves" src="http://dev-mkm-pics.s3.amazonaws.com/approval-kim-jong-un.gif" /></p>
<p><em>Note from the editorial staff: This is the last post on dev.mikamai.com until next year! Thanks everyone for following us, on January we&rsquo;ll be back with more Emacs Lisp, Ruby, Arduino, Raspberry PI, Python, Go, PHP, Prestashop, &hellip;</em></p>