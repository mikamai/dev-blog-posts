---
ID: 177
post_title: >
  Create an animated GIF of your console
  sessions
author: Massimo Ronca
post_date: 2014-12-06 10:10:12
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/06/create-an-animated-gif-of-your-console-sessions/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/104480613554/create-an-animated-gif-of-your-console-sessions
tumblr_mikamayhem_id:
  - "104480613554"
---
<img src="http://s2.quickmeme.com/img/32/325fc351053e41d230961a71308d37937e68192130d11a82308ae619571ef942.jpg" alt="animate all the things" />

When I write articles, sometimes I feel the need to show how the commands behave interactively, not only the sequence of commands you have to type.
It’s easier to understand by looking at an animation, than reading “when you hit TAB &lt;this happens&gt;”.

<!--more-->

For example, can you explain how the emmet plugin for VIM works, better than this, using only words?

<img src="https://qiita-image-store.s3.amazonaws.com/0/38647/86b91c27-f894-c969-89b0-5846408ad1db.gif" alt="emmet plugin for VIM" />

Fortunately, the solution is pretty easy.

You need a few open source tools, if you’re on a Mac, like me, you should already have installed <a href="http://brew.sh/">Homebrew</a>, and Imagemagick (<code>brew install imagemagick</code>).

To record your sessions, you need <code>ttyrec</code> (<code>brew install ttyrec</code> on Mac).
<pre><code class="bash">usage: ttyrec [-u] [-e command] [-a] [file]

OPTIONS
      -a     Append the output to file or ttyrecord, rather than overwriting it.

      -u     With this option, ttyrec automatically calls uudecode(1) and  saves  its  output  when  uuencoded  data
             appear on the session.  It allow you to transfer files from remote host.  You can call ttyrec with this
             option, login to the remote host and invoke uuencode(1) on it for the file you want to transfer.

      -e command
             Invoke command when ttyrec starts.
</code></pre>
<code>file</code> is the name of the file that will be used to record the session. If no file name is given, <code>ttyrecord</code> will be used.

A new session is started as soon as you launch <code>ttyrec</code> and is automatically saved when you close the session with <code>CTRL+D</code> or <code>exit</code>.

To replay an already saved session, use
<pre><code class="bash">ttyplay [-s SPEED] [-n] [-p] file

OPTIONS
      -s SPEED
             multiple the playing speed by SPEED (default is 1).

      -n     no wait mode.  Ignore the timing information in file.

      -p     peek another person's tty session.
</code></pre>
Let’s see how it looks

<img src="http://i.imgur.com/q7NHxN0.gif" alt="ttyrec session" />

Now that we have a recorded sessions, we need to convert it to an animated GIF.

We’ll use <a href="https://github.com/icholy/ttygif"><code>ttygif</code></a> for the task.

There’s no installer for it, you must compile it from the sources
<pre><code class="bash">$ git clone <a href="https://github.com/icholy/ttygif.git">https://github.com/icholy/ttygif.git</a>
$ cd ttygif
$ make
</code></pre>
Once make is done, you will find some executable files in the folder
<pre><code class="bash">-rwxr-xr-x   1 maks  staff    881 Nov  5 16:08 concat.sh
-rwxr-xr-x   1 maks  staff    829 Nov  5 16:08 concat_osx.sh
-rwxr-xr-x   1 maks  staff  14836 Nov  5 16:08 ttygif
</code></pre>
In my setup I’ve linked <code>ttygif</code> to <code>/usr/local/bin</code> and <code>concat.sh</code> (<code>concat_osx.sh</code> in case you’re on a Mac) to <code>/usr/local/bin/ttyconcat</code> to avoid name clashes.

Creating the gif is a two steps process: first you launch <code>ttygif &lt;recfile&gt;</code> to generate a sequence of PNGs, then you launch the <code>ttyconcat</code> script we linked before and it automatically creates the animated GIF for you.

Optionally you can pass and output filename to <code>ttyconcat</code>, if omitted the image will be saved as <code>output.gif</code>.

If you are a prefectionist, there’s an optional final step, install <a href="http://www.lcdf.org/gifsicle/"><code>gifsicle</code></a> (<code>brew install gifsicle</code>) and give your animation the final touches.

I usually add a fixed delay between frames, make it loop forever and optimize the size with this command line
<pre><code class="bash">gifsicle --delay=10 --loop=0 -O3 &lt; in.gif &gt; out.gif
</code></pre>
More otpions can be found on the <a href="http://www.lcdf.org/gifsicle/man.html"><code>gifsicle</code> man page</a>.