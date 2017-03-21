---
ID: 22
post_title: >
  A little story about brew, Firefox 47,
  Selenium and Capybara
author: Gianluca
post_date: 2016-06-23 11:42:43
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/06/23/a-little-story-about-brew-firefox-47-selenium/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/146352856184/a-little-story-about-brew-firefox-47-selenium
tumblr_mikamayhem_id:
  - "146352856184"
---
I am quite a big fan of <a href="http://brew.sh/"><code>brew</code></a> and <a href="https://caskroom.github.io/"><code>brew cask</code></a>. I switched from Linux to OSX less than a year ago and both of them gave me immediately the feel of being at home with a “standardized” way to handle the installation of applications on my system.

Yes, I know there are different conflicting opinions regarding how <code>brew</code> works under the hood like where it puts the installed applications, how it symlinks them and how it handles different app versions. Sincerely I never paid much attention to these discussions (my bad). For me it works quite well as it gives me the feel to have an organized and clean system.

Personally I also like to have my system steadily up-to-date and to achieve this goal I constantly use a <strong>brew update everything command</strong> that I built (or found?…sorry, I don’t remember XD) which automatically updates all the applications, “CLI and GUI based”, I’ve installed:
<pre class="prettyprint"><code class="hljs brainfuck"><span class="hljs-comment">brew</span> <span class="hljs-comment">update</span> <span class="hljs-comment">&amp;&amp;</span> <span class="hljs-comment">brew</span> <span class="hljs-comment">upgrade</span> <span class="hljs-comment">&amp;&amp;</span> <span class="hljs-comment">brew</span> <span class="hljs-comment">cask</span> <span class="hljs-comment">list</span> <span class="hljs-comment">|</span> <span class="hljs-comment">xargs</span> <span class="hljs-comment">brew</span> <span class="hljs-comment">cask</span> <span class="hljs-comment">install</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">force</span> <span class="hljs-comment">&amp;&amp;</span> <span class="hljs-comment">brew</span> <span class="hljs-comment">cleanup</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">force</span> <span class="hljs-comment">&amp;&amp;</span> <span class="hljs-comment">brew</span> <span class="hljs-comment">cask</span> <span class="hljs-comment">cleanup</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span><span class="hljs-comment">force</span></code></pre>
What it does is to update both <code>brew</code> and <code>brew cask</code> repositories, upgrade the installed applications (again, “CLI and GUI based”) and then do a cleanup by removing the old version of the applications.

Launching this command is quite disruptive considering that it forces the removal of the old version of the apps. So I do not feel like recommend it unless you’re able to handle some occasional problems.

A few days ago I ran indeed in one of these problems.

<!--more-->

After launching the <strong>brew update everything command</strong> and monitoring its output inside the terminal I gladly noticed that among the GUI applications there was a major update of Firefox, i.e. 47.0 release.

Once the command finished I jumped back to a project of mine and launched a Capybara spec I was working on and…well…Firefox crashed pretty badly…

I knew that it was something related to Selenium and after a little bit of googling I ended up <a href="https://github.com/seleniumhq/selenium/issues/2110">here</a>. Yes I know, the issue doesn’t completely align with my setup (i.e. Windows 7 vs. OSX) but the problem was practically the same.

Among the issue comments <a href="https://github.com/seleniumhq/selenium/issues/2110#issuecomment-224413609">this</a> one caught my attention. Considering the <strong>Note</strong> in the comment I decided to briefly give <a href="https://blog.testingbot.com/2016/03/23/marionette-the-next-generation-of-firefoxdriver">Marionette</a> a try but I wasn’t successful and so I rapidly jumped on downgrading Firefox.

I searched around for a <code>tap</code> that would allow me to handle different versions of Firefox but I was once again unsuccessful.

The next thing I did was to search a way to manually downgrade Firefox through <code>brew cask</code>. The first option I found was to manually modify the Firefox cask itself. By launching <code>brew cask edit firefox</code> I successfully opened the cask
<pre class="prettyprint"><code class="hljs ruby">cask <span class="hljs-string">'firefox'</span> <span class="hljs-keyword">do</span>
  version <span class="hljs-string">'47.0'</span>
  sha256 <span class="hljs-string">'e8e068a8f87126d1e252a51bbd0d4b20314fef8dc015c70c21468deaab9c4d9d'</span>

  url <span class="hljs-string">"https://ftp.mozilla.org/pub/firefox/releases/<span class="hljs-subst">#{version}</span>/mac/en-US/Firefox%20<span class="hljs-subst">#{version}</span>.dmg"</span>
  appcast <span class="hljs-string">"https://aus5.mozilla.org/update/3/Firefox/<span class="hljs-subst">#{version}</span>/0/Darwin_x86_64-gcc3-u-i386-x86_64/en-US/release/Darwin%2015.3.0/default/default/update.xml?force=1"</span>,
          <span class="hljs-symbol">checkpoint:</span> <span class="hljs-string">'9cce3ec6844e54c07b7870461fdad111af0c4ce9161dd24c8425885b4f62d372'</span>
  name <span class="hljs-string">'Mozilla Firefox'</span>
  homepage <span class="hljs-string">'https://www.mozilla.org/en-US/firefox/'</span>
  license <span class="hljs-symbol">:mpl</span>

  auto_updates <span class="hljs-keyword">true</span>

  app <span class="hljs-string">'Firefox.app'</span>

  zap <span class="hljs-symbol">delete:</span> [
                <span class="hljs-string">'~/Library/Application Support/Firefox'</span>,
                <span class="hljs-string">'~/Library/Caches/Firefox'</span>,
              ]
<span class="hljs-keyword">end</span></code></pre>
and then I modified the version
<pre class="prettyprint"><code class="hljs r">cask <span class="hljs-string">'firefox'</span> do
  version <span class="hljs-string">'46.0'</span>
  <span class="hljs-keyword">...</span></code></pre>
After that I tried to re-install Firefox with <code>brew cask install --force firefox</code> but I ended up with the following expected error:
<pre class="prettyprint"><code class="hljs coffeescript">=<span class="hljs-function">=&gt;</span> Downloading <span class="hljs-attribute">https</span>:<span class="hljs-regexp">//</span><a href="http://ftp.mozilla.org/pub/firefox/releases/">ftp.mozilla.org/pub/firefox/releases/</a><span class="hljs-number">46.0</span>/mac/en-US/Firefox%<span class="hljs-number">2046.0</span>.dmg
<span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span><span class="hljs-comment">######</span> <span class="hljs-number">100.0</span>%
=<span class="hljs-function">=&gt;</span> Verifying checksum <span class="hljs-keyword">for</span> Cask <span class="hljs-function"><span class="hljs-title">firefox</span>
==&gt;</span> <span class="hljs-attribute">Note</span>: running <span class="hljs-string">"brew update"</span> may fix sha256 checksum errors
<span class="hljs-attribute">Error</span>: sha256 mismatch
<span class="hljs-attribute">Expected</span>: e8e068a8f87126d1e252a51bbd0d4b20314fef8dc015c70c21468deaab9c4d9d
<span class="hljs-attribute">Actual</span>: f54b7e3e6bb330754ef4df7f96516fd7279a9fb01aad3db7fd2eadd25d8a279f
<span class="hljs-attribute">File</span>: /Users/user/Library/Caches/Homebrew/firefox-<span class="hljs-number">46.0</span>.dmg
To retry an incomplete download, remove the file above.</code></pre>
The problem was indeed related to the mismatched sha256.

After substituting the sha256 of the opened cask (i.e. the sha256 of the version 47.0 of Firefox) with the one reported by the error and launching <code>brew cask install --force firefox</code> I was finally able to get the old version of Firefox (i.e. 46.0) installed.
<pre class="prettyprint"><code class="hljs r">cask <span class="hljs-string">'firefox'</span> do
  version <span class="hljs-string">'46.0'</span>
  sha256 <span class="hljs-string">'f54b7e3e6bb330754ef4df7f96516fd7279a9fb01aad3db7fd2eadd25d8a279f'</span>
  <span class="hljs-keyword">...</span>
</code></pre>
After that I removed the new version of Firefox (i.e. 47.0) from <code>/usr/local/Caskroom/firefox/</code> and launched the problematic Capybara spec to check that the issue was really resolved and so it was! ;)

This quick and dirty fix needs however one last action: completely disable Firefox automatic updates!

By default (and this is ok if you’re using Firefox as your primary browser) the automatic updates are enabled and as soon as Firefox runs it tries to update to the latestes release. In my case it’s not needed. :P

One last note. The modification of the local cask will be lost after the next <code>brew update &amp;&amp; brew upgrade</code> but, once again, this is not a big deal in my case. ;P

All I need to do after the successive launches of the <strong>brew update everything command</strong> is to manually remove the new version of Firefox in <code>/usr/local/Caskroom/firefox/</code>.

I hope that this little adventure of mine can be useful to people that are encountering the same problem I’ve described in this brief post.

To the next time,
Cheers!