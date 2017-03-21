---
ID: 281
post_title: Pimp my TextMate2 — Ruby edition
author: Elia Schito
post_date: 2014-02-24 15:33:18
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/24/pimp-my-textmate2-ruby-edition/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/77705936763/pimp-my-textmate2-ruby-edition
tumblr_mikamayhem_id:
  - "77705936763"
---
<p>Here at Mikamai It&rsquo;s no secret I&rsquo;m a happy TextMate user and early TM2 adopter. I&rsquo;m always there if either an editor war is catching fire or if someone needs help setting up his editor.</p>

<p>Above all I still find that TextMate is the best choice for a Ruby developer, even if SublimeText, emacs and vim seem more fashionable these days. Even if I&rsquo;m saving the full list of reasons for another post I&rsquo;ll tell just this one: <strong>TextMate relies on Ruby for a big part of its implementation that has always been opensource</strong>*.</p>

<p>Of course I&rsquo;m talking about bundles, if you&rsquo;re not convinced <a href="https://github.com/textmate/source.tmbundle/blob/master/Commands/Align%20Assignments.tmCommand#L8-L130">look at the code used to align assignments </a> (<kbd>⌥⌘]</kbd>) from the <a href="https://github.com/textmate/source.tmbundle">source bundle</a> (which is responsible for actions common to any programming language).</p>

<p>Just to be clear I would still use SublimeText if I were to program from Linux or (ugh!) Windows.</p>

<p>That said I want to gather here some of the stuff that makes using TextMate2 for Ruby and Rails development so awesome.</p>

<p><small>* Of course I know about <a href="http://redcareditor.com">redcar</a> (which seems quite dead, but I didn&rsquo;t tried it recently) and other TM <del>clones</del> err enhancements like <a href="https://chocolatapp.com">Chocolat</a></small></p>

<blockquote>
  <p>ALERT: shameless self promotion follows</p>
</blockquote>

<h2>The Bundles and Settings parade</h2>

<h3>1. Effortless opening of Bundled Gems <kbd>⌥⌘O</kbd></h3>

<p>This I do all the time, opening the source of bundled gems. Please behold and don&rsquo;t be horrified. Especially in Ruby-land the source of gems is the best source of documentation, and as <a href="http://www.confreaks.com/videos/282-lsrc2010-real-software-engineering">explained by Glenn Vanderburg</a>) there&rsquo;s probably a good reason for that. Also the README and specs are included most of the time and reading other&rsquo;s code is a healthy activity.</p>

<p>Needless to say that the best place to read source code is your editor.</p>

<p><kbd>⌥⌘O</kbd> will present the complete list of gems from your <code>Gemfile.lock</code>, start typing the first letters of a gem and use arrow if you don&rsquo;t want to touch the mouse (or trackpad).</p>

<p><img src="https://f.cloud.github.com/assets/1051/190195/03670698-7ed5-11e2-983d-6da8b0d0dd7a.png" alt="opening gems from the current bundle" /></p>

<blockquote>
  <p>Source: <a href="https://github.com/elia/bundler.tmbundle">https://github.com/elia/bundler.tmbundle</a></p>
</blockquote>

<h3>2. Beautiful Markdown rendering</h3>

<p>No README.md reading activity would be on par with a the <a href="https://help.github.com/articles/github-flavored-markdown">GFM</a> rendered version without code blocks highlighting.</p>

<p>This bundle almost looks like GFM while typing, press <kbd>⌃⌥⌘P</kbd> (the standard TM key equivalent for preview) to get it rendered to the HTML window.</p>

<p><img src="http://cl.ly/image/1Y071W2A2l1w/Screen%20Shot%202014-02-18%20at%2011.02.32%20am.png" alt="Redcarpet Markdown Bundle in action" /></p>

<p><strong>Bonus</strong> <a href="http://cl.ly/image/2v3v1Z0u3F11">Install the <strong>Scott Web Theme</strong> from <em>Preferences → Bundles</em></a> for a nice looking preview</p>

<blockquote>
  <p>Source: <a href="https://github.com/streeter/markdown-redcarpet.tmbundle">https://github.com/streeter/markdown-redcarpet.tmbundle</a></p>
</blockquote>

<h3>3. Restart Pow! in a single stroke <kbd>⌃⌥⌘R</kbd></h3>

<p>Restarts the current app detecting a <code>tmp/</code> directory in current project or in a parent dir.</p>

<p><img src="http://cl.ly/image/2b3K3D3w2C11/Screen%20Shot%202014-02-24%20at%2012.24.32%20pm.png" alt="Falls back to /tmp" /></p>

<blockquote>
  <p>Source: <a href="https://github.com/elia/pow-server.tmbundle">https://github.com/elia/pow-server.tmbundle</a></p>
</blockquote>

<h3>4. Open the terminal in your current project folder</h3>

<p>Works with both Terminal and iTerm, just press <kbd>⌃⌥⌘T</kbd> from a project.</p>

<blockquote>
  <p>Source: <a href="https://github.com/elia/avian-missing.tmbundle">https://github.com/elia/avian-missing.tmbundle</a></p>
</blockquote>

<h3>5. Trailing whitespace fix, cross-tab completion and more…</h3>

<table><thead><tr><th>Command</th>
  <th>Description</th>
</tr></thead><tbody><tr><td><kbd>⌃⎋</kbd></td>
  <td>Cross tab completion</td>
</tr><tr><td><kbd>⌃⌥⌘T</kbd></td>
  <td>Open Project directory in Terminal</td>
</tr><tr><td><kbd>⌃⌥⌘L</kbd></td>
  <td>Keep current file as reference</td>
</tr></tbody></table><blockquote>
  <p>Source: <a href="https://github.com/elia/avian-missing.tmbundle">https://github.com/elia/avian-missing.tmbundle</a></p>
</blockquote>

<h2>Installing the whole thing</h2>

<p>Download the latest version of TextMate2 here: <a href="https://api.textmate.org/downloads/release">https://api.textmate.org/downloads/release</a></p>

<pre><code>mkdir -p ~/Library/Application Support/Avian/Bundles
cd ~/Library/Application Support/Avian/Bundles

git clone <a href="https://github.com/elia/avian-missing.tmbundle">https://github.com/elia/avian-missing.tmbundle</a>
git clone <a href="https://github.com/elia/bundler.tmbundle">https://github.com/elia/bundler.tmbundle</a>
git clone <a href="https://github.com/streeter/markdown-redcarpet.tmbundle">https://github.com/streeter/markdown-redcarpet.tmbundle</a>

# Activate the system ruby (if you're using a Ruby version manager):
type rvm &amp;&gt; /dev/null &amp;&amp; rvm use system # for RVM
export RBENV_VERSION="system"           # for rbenv

# Install the required gems
sudo gem install redcarpet -v 2.3.0
sudo gem install pygments.rb

# Trailing whitespace
defaults write com.macromates.TextMate.preview environmentVariables -array-add 
    '{ "enabled" = YES; "name" = "TM_STRIP_WHITESPACE_ON_SAVE"; "value" = "true"; }' # enable trailing whitespace removal and EOF fix

echo &lt;&lt;-INI &gt;&gt; ~/.tm_properties
[ "*.y{,a}ml" ]
# Disable trailing whitespace fix for YAML 
# files that can be broken by this feat.
TM_STRIP_WHITESPACE_ON_SAVE = false
INI
</code></pre>

<p>The following will set the tabs/file-browser/html-windows to my current taste, I don&rsquo;t pretend it matches everyone prefs but can still be useful for cherry-picking.</p>

<pre><code># File browser fixes and general UI fixes
defaults write com.macromates.TextMate.preview fileBrowserStyle SourceList       # lighblue file browser background
defaults write com.macromates.TextMate.preview fileBrowserPlacement left         # keep it on the left
defaults write com.macromates.TextMate.preview tabsAboveDocument -bool YES       # no tabs above the file browser
defaults write com.macromates.TextMate.preview allowExpandingLinks -bool YES     # make symlinks expandable
defaults write com.macromates.TextMate.preview htmlOutputPlacement right         # place the html output to the right
defaults write com.macromates.TextMate.preview disableTabBarCollapsing -bool YES # keep the tab-bar alway visible
defaults write com.macromates.TextMate.preview scrollPastEnd -bool YES           # give me some air after the file ends
</code></pre>