---
ID: 223
post_title: Some ways to improve Sublime Text (3)
author: Andrew Coates
post_date: 2014-07-21 13:38:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/21/some-ways-to-improve-sublime-text-3/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/92429950039/some-ways-to-improve-sublime-text-3
tumblr_mikamayhem_id:
  - "92429950039"
---
Sublime Text is a fast, extensive, cross-platform text editor which is supported by a large open-source community and a library of plugins &amp; themes. Here are some of the ways I&rsquo;ve improved my version of Sublime Text: 
<br /><br />
The most essential Sublime Text plugin is <a href="https://sublime.wbond.net/">Package Control:</a>
<br /><br /><img src="http://68.media.tumblr.com/66781e1292675d538a2228dc20286330/tumblr_inline_n9261g3v1w1s7zrue.png" /><br /><br />
This plugin will help you acquire the rest of the plugs I&rsquo;ll mention later, and it&rsquo;s simple to install. If you&rsquo;re using Sublime Text 3 like me, open the console (View &gt; Show Console) and paste the following: 
<br /><pre><code>import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path();urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)</code></pre>
<br />
If you&rsquo;re on Sublime Text 2, use this code: 
<br /><pre><code>import urllib2,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation') </code></pre>
<br />
The next plugin I recommend is <a href="https://github.com/kemayo/sublime-text-git/wiki">this Git plugin</a>. It&rsquo;s fairly self-explanitory: it adds Git traits to the text editor. With Package Control, installation is simple: open Package Control with the &ldquo;⌘+Shift+P&rdquo; keystroke, type &lsquo;Install Package&rsquo;, and select 'Git&rsquo;. 
<br /><br /><img src="http://68.media.tumblr.com/895cd2f1e03acc43910c99dab4916494/tumblr_inline_n92a8bScae1s7zrue.png" /><br /><br />
Third plugin I can recommend is <a href="https://github.com/titoBouzout/SideBarEnhancements">'Sidebar Enhancements&rsquo;</a>. This can be installed similarly to the Git plugin. It adds a wealth of features to the bare default sidebar, of which can be seen below:
<br /><br /><img src="http://68.media.tumblr.com/5e68fcd0d4e7bc2504a266274e2c5116/tumblr_inline_n92b5svr7o1s7zrue.png" /><br /><br />
Fourth: <a href="https://github.com/noklesta/SublimeQuickFileCreator">Quick File Creator</a>. This simple plugins adds the feature to create files with the command palette. 
<br /><br /><img src="http://68.media.tumblr.com/e115823d6079eb104d596f201706e353/tumblr_inline_n92c5k8iT11s7zrue.png" /><br /><br />
Fifth: <a href="https://github.com/lunixbochs/sublimelint">SublimeLint</a> &ndash; error highlighting.
<br /><br /><img src="http://68.media.tumblr.com/6b1f0f99656fde473857cb78e98817dd/tumblr_inline_n92c9rvm8v1s7zrue.png" /><br /><br />
Finally, <a href="https://github.com/sergeche/emmet-sublime">Emmet</a> - an incredibly useful plugin for any web developer. Emmet is an auto-completion engine for HTML and CSS code. Check out an example in this .gif:
<br /><br /><img src="http://68.media.tumblr.com/0b3d2f908cb07a22dc956940e245e4b3/tumblr_inline_n92drxdxz51s7zrue.gif" /><h1>Some useful keyboard shortcuts:</h1>
<br /><b>Show file in project:</b> This shortcut needs to be added to Sublime, but it is one that I find myself constantly using. It will reveal the location of the current file in the sidebar. To map it, open your keymap file (⌘+Shift+P, search for 'key bindings&rsquo;), and add the following line:
<br /><br /><pre><code>	{ "keys": ["super+shift+r"], "command": "goto_symbol_in_project" }, </code></pre>
<br /><br /><b>Multiple cursors:</b> Great for repetitive tasks. The keystroke is: <b>'CTRL+Shift+Arrow Key&rsquo;</b>. This will allow you to add as many cursors as you want!
<br /><br /><b>Close the current HTML tag:</b> a great keystroke for web-developers: <b>'⌘+Alt+.&rsquo;</b>
<br /><h1>Theming</h1>
There are thousands of themes for Sublime, so it&rsquo;s really down to your preference. I&rsquo;m using the <a href="https://github.com/buymeasoda/soda-theme/">Soda theme</a>, but you can find many more <a href="http://colorsublime.com/">here!</a>
<br /><br /><img src="http://68.media.tumblr.com/fb238ef149c16b9b132da0a7d3998f21/tumblr_inline_n92dko3HTc1s7zrue.png" />