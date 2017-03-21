---
ID: 61
post_title: Atom and vim-mode
author: Diomede Tripicchio
post_date: 2015-12-01 13:25:16
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/12/01/atom-and-vim-mode/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/134330175904/atom-and-vim-mode
tumblr_mikamayhem_id:
  - "134330175904"
---
<p>Some years ago I started using VIM, mostly because it was nerdy and cool!</p>

<p>Back to serious, I had to manage some servers, edit nginx/ftp/ssh/mail config etc. all via ssh and at the same time I had to develop the apps that run on these servers, in short some dev-ops tasks.</p>

<p>VIM seemed a natural choice be it just to have a consistency in my development environment. So I started using it, initially the modal mode was strange but now I can&rsquo;t think of using an editor without this mode.</p>
<!--more-->

<h3>Switch to Atom</h3>

<p>I&rsquo;ve always been curious about Atom and about one month ago I decided to try it seriously i.e. without going back to VIM for a while.</p>

<p>First of all, I installed <a href="https://github.com/atom/vim-mode">vim-mode</a>, thanks to this extension Atom acquires the VIM modal mode and all its basic commands.</p>

<p>Initially I used the VIM commands as usual to move through lines, replace etc. but the real power of <strong>vim-mode</strong> is the possibility to use these commands in combination with Atom multiple cursor.</p>

<p>The combination of these two features can be similar to a basic (very basic) implementation of VIM macros.</p>

<p>For example I have this Hash in ruby:</p>

```ruby
hash = {
  :foo =&gt; &#039;bar&#039;,
  :barbar =&gt; &#039;foo&#039;
}
```

<p>I want to change this old assignment mode <code>:foo =&gt; 'bar'</code> with the new <code>foo: 'bar'</code> but I want to do it for all hash attributes at once. In visual mode I can select <code>=&gt;</code> (note the space) and then press <code>Cmd+D</code> (multiple cursor), now I press <code>d</code> to remove then <code>i</code> to insert the &rsquo;:&rsquo; then I press <code>[esc]^</code> to go at the first character of the line and press <code>x</code>, so I obtain this:</p>

<p><em>Short: select <code>=&gt;</code> then <code>Cmd+D</code> for each attribute I want to edit and finally <code>di:[esc]^x[esc]</code></em></p>

```ruby
hash = {
  foo: &#039;bar&#039;,
  barbar: &#039;foo&#039;
}
```

<p>This is a minimal example just to show the possibilities of <strong>vim-mode</strong> and maybe convince you to try Atom. At the moment I have not abandoned VIM, sometimes I find myself thinking &ldquo;I want my VIM back!&rdquo; but the combination of vim-mode and multiple-cursor keeps me and it will keep me on Atom still for a while.</p>