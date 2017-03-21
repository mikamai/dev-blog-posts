---
ID: 34
post_title: 'Phoenix Framework: the assets pipeline'
author: Massimo Ronca
post_date: 2016-04-30 11:46:39
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/04/30/phoenix-framework-the-assets-pipeline/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/143627439174/phoenix-framework-the-assets-pipeline
tumblr_mikamayhem_id:
  - "143627439174"
---
<h4 id="toc_0">Updates</h4>
From the time I wrote <a href="http://dev.mikamai.com/post/138675652509/phoenix-to-the-basics-and-beyond">part 1</a>
of this short series, Atom has gained a new Elixir plugin based on Samuel Tonini’s Alchemist Server.
From the Emacs plugin, it inherits all the most notable features such as
autocomplete, jump to definition/documentation for the function/module under
the cursor, quote/unquote code and interactive macro expansion.
A feature reference along with some screenshots can be found at the <a href="https://atom.io/packages/atom-elixir">atom-elixir</a> page.
It also looks pretty good.

<img src="https://i.github-camo.com/37f447507f446c06c2631a688056c61dea7ef930/68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f6d736172616976612f61746f6d2d656c697869722f6173736574732f73637265656e73686f74732f6175746f636f6d706c657465312e706e673f7261773d74727565" alt="" />
<h2 id="toc_1">The assets pipeline</h2>
Assets pipelines are one of the most important features in modern web frameworks.
When working on this task, Phoenix developers have proven that they value
pragmatism over purity and have chosen to base their implementation on <a href="http://brunch.io/">Brunch</a>, a Node.js build tool that takes care of everything
related to assets management.
This choice has probably saved man-years of work, that would have inevitably delayed the release of a fully working pipeline system.
A very common counter argument is that this adds node as a dependency, but I
think it’s a negligible inconvenient, node is most probably already present on
the majority of developers machines.

<!--more-->

Brunch installation is just a <code>npm install</code> away and it runs automatically when
you create a new Phoenix project
<div>
<pre><code class="bash">$ mix phoenix.new brunch_demo
* creating brunch_demo/config/config.exs
* creating brunch_demo/config/dev.exs
...
Fetch and install dependencies? [Yn] y
* running mix deps.get
* running npm install &amp;&amp; node node_modules/brunch/bin/brunch build</code></pre>
</div>
As you can guess, the local brunch is installed in <code>node_modules/brunch/bin/brunch</code>.
You can install it globally with the usual <code>-g</code> flag: <code>npm install -g brunch</code>.
Phoenix automatically runs Brunch when assets change.
If you launch <code>mix phoenix.server</code> and make some changes to <code>web/static/js/app.js</code> you can see that Brunch is working in the background.
<div>
<pre><code class="bash">[info] Running BrunchDemo.Endpoint with Cowboy using http on port 4000
28 Apr 13:31:56 - info: compiled 5 files into 2 files, copied 3 in 2.2 sec
28 Apr 13:33:08 - info: compiled app.js and 3 cached files into app.js in 118ms</code></pre>
</div>
<h4 id="toc_2">Defaults</h4>
By default Brunch watch for changes in <code>css</code> and <code>js</code> folders inside <code>web/static</code> and automatically recompile and package them for HTTP serving.
It is configured to work with ES6 and transpile it to ES5 through
Babel, so you can start using ES6 today, without hassles.

Javascript files are automatically wrapped into a module, that needs to be required
before being used.
Every file inside <code>web/static/js</code> will be converted and loaded on demand.
<div>
<pre><code class="language-javascript">// web/static/js/app.js
export var App = {
  run: function(){
    console.log("Hello from Phoenix!")
  }
}</code></pre>
</div>
<div>
<pre><code class="html">&lt;!-- web/templates/layout/app.html.eex --&gt;
&lt;script&gt;require("web/static/js/app").App.run()&lt;/script&gt;

&lt;!-- shows "Hello from Phoenix!" in the browser's console --&gt;</code></pre>
</div>
If you have legacy code that won’t work if modularized or doesn’t need modularization,
you can put it in <code>web/static/vendor</code> and it will be copied as it is.
This is also the easier way to create global variables
<div>
<pre><code class="language-javascript">// web/static/vendor/globals.js
global_variable = "this is global";

// open up the console and type global_variable
// can you guess the result?</code></pre>
</div>
Any of this default can be changed in <code>brunch-config.js</code>.
For example this is the part that ignores the module wrapping for the vendor folder.
<div>
<pre><code class="language-javascript">plugins: {
  babel: {
    // Do not use ES6 compiler in vendor code
    ignore: [/web/static/vendor/]
  }
},</code></pre>
</div>
Last but not least, Phoenix automatically loads
<ul>
 	<li>Brunch’s <q>bootstrapper</q> code which provides module management and require() logic</li>
 	<li>Phoenix Channels JavaScript client (deps/phoenix/web/static/js/phoenix.js)</li>
 	<li>Some code from Phoenix.HTML (deps/phoenix<em>html/web/static/js/phoenix</em>html.js)</li>
</ul>
<h4 id="toc_3">Plugins</h4>
Brunch is configured to load a numbers of plugin, specifically
<ul>
 	<li>javascript-brunch: enables processing of Javascript files</li>
 	<li>babel-brunch: the ES6 transpiler</li>
 	<li>uglify-js-brunch: Javascript minifier</li>
 	<li>css-brunch: enables processing of CSS files</li>
 	<li>clean-css-brunch: CSS minifier</li>
</ul>
Plugins are installed through <code>npm</code>, for example if you wanna use Coffeescript in
your application you can simply <code>npm install --save coffee-script-brunch</code> and
your coffee files will be automatically picked up and processed.
<h3 id="toc_4">Tool belt</h3>
Manually copying files or libraries inside the vendor folder is not exactly the
best way to handle external dependencies.
One of the features of Brunch is that it allows the developers to take advantage of the <code>Node</code> ecosystem.
Brunch works seamlessly with Bower, which IMHO is the simplest way to handle
third-party frontend dependencies.
Do you need <code>underscore</code>?
Just <code>bower install --save underscore</code>, (restart the server if the files are not being compiled automatically) and type <code>_</code> in the console
<div>
<pre><code class="language-javascript">function _(obj) {
  if (obj instanceof _) return obj;
  if (!(this instanceof _)) return new _(obj);
  this._wrapped = obj;
}</code></pre>
</div>
One problem I’ve found is that not every folder provided by the packages is being copied (or concatenated) in the output folder (usually <code>priv/static</code> unless you changed it).
I was working with an old version of <code>materialize</code> and the font was not being loaded.
The solution was pretty easy, just open up <code>lib/&lt;app_name&gt;/endpoint.ex</code> and look for this line
<div>
<pre><code class="language-elixir">plug Plug.Static,
  only: ~w(css fonts images js favicon.ico robots.txt)</code></pre>
</div>
In my case <code>materialize</code> was using <code>font</code> as a folder name, but it wasn’t whitelisted
in the Static Plug configuration. I added <code>font</code> to the list and it fixed the issue.
<h3 id="toc_5">There’s more than a way</h3>
One thing that the Phoenix framework does and does really well is not being
strongly opinionated.
Brunch is just the default assets manager, but it’s really simple to use another one, just change this line in <code>dev.ex</code>
<div>
<pre><code class="language-elixir">watchers: [node: ["node_modules/brunch/bin/brunch", "watch", "--stdin"]]</code></pre>
</div>
There’s an example in the Pheonix documentation on <a href="http://www.phoenixframework.org/docs/static-assets#section-phoenix-without-brunch">how to use Phoenix with webpack</a>, I’m gonna go further, I’ll show you how to write the skeleton of an assets manager and get it started by Phoenix.
Create a new file in <code>lib/watcher.exs</code> (exs means Exlixir script) and paste this code inside it
<div>
<pre><code class="language-elixir"># lib/watcher.exs

defmodule Watcher do

  def start do
    IO.puts "* Start monitoring #{Path.absname("web/static")}"
    IO.inspect System.argv
    IO.inspect System.get_env
  end

end

Watcher.start</code></pre>
</div>
and then change the configuration this way
<div>
<pre><code class="language-elixir">watchers: [mix: ["run", "lib/watcher.exs", "random input"]]</code></pre>
</div>
and start the Phoenix server
<div>
<pre><code class="language-bash">mix phoenix.server

[info] Running BrunchDemo.Endpoint with Cowboy using http on port 4000
* Start monitoring &lt;app_path&gt;/web/static
["random input"]
%{"CLICOLOR" =&gt; "1", "PROMPT_COMMAND" =&gt; "_update_ps1; update_terminal_cwd",
  "_system_arch" =&gt; "x86_64",
  "DISPLAY" =&gt; "/private/tmp/com.apple.launchd.4Zqdr48hnr/org.macosforge.xquartz:0",
...</code></pre>
</div>
Ok, it works but it’s not really useful, we’re gonna add a new feature, that watches
for file changes inside the <code>web/static</code> folder and logs to the screen.

First we’re gonna need to install a filesystem watcher component, there is one
written in Erlang that we can import directly from Github.

Add this line to <code>mix.exs</code>
<div>
<pre><code class="language-elixir"># mix.exs
defp deps do
  # ...
   {:fs, github: "synrc/fs", override: true}]
   # override tells the Elixir compiler that this package overrides the default
   # :fs Erlang module
end</code></pre>
</div>
fetch the library with <code>mix deps.get</code>
and then update the <code>watcher.exs</code> code
<div>
<pre><code class="language-elixir"># lib/watcher.exs
defmodule Watcher do

  def start do
    IO.puts "* Start monitoring #{Path.absname("web/static")}"
    IO.inspect System.argv
    IO.inspect System.get_env
    # starts the listener
    :fs.start_link(:watcher, Path.absname("web/static"))
    :fs.subscribe(:watcher)
    loop
  end

  def loop do
    receive do
      {_watcher_process, {:fs, :file_event}, {path, flags}} -&gt;
        # logs events to the screen
        IO.puts("* #{path} -&gt; #{Enum.join flags, ", "}")
    end
    loop
  end
end

Watcher.start</code></pre>
</div>
Start the server again, change some file inside <code>web/static</code> and enjoy the results.
<div>
<pre><code class="language-bash"># save app.js
* &lt;app_path&gt;/web/static/js/app.js -&gt; inodemetamod, modified

# save a new file
* &lt;app_path&gt;/web/static/js/app2.js -&gt; created, modified, finderinfomod, xattrmod

# delete a file
* &lt;app_path&gt;/web/static/js/app2.js -&gt; renamed</code></pre>
</div>
That’s it for now, next time we’ll talk about long running processes.