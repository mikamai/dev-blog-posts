---
ID: 334
post_title: >
  “Use Node.js FOR SPEED” —
  @RoflscaleTips
author: Elia Schito
post_date: 2012-06-11 09:03:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2012/06/11/use-nodejs-for-speed-roflscaletips/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/24875756102/use-nodejs-for-speed-roflscaletips
tumblr_mikamayhem_id:
  - "24875756102"
---
<p>I obeyed, therefore I wrote an <a href="http://search.npmjs.org/#/opal"><code>opal</code></a> NPM package.</p>
<p>Now I can trash Rails and <a href="http://elia.schito.me/post/24087273859/stop-writing-javascript-and-get-back-to-ruby"><code>opal-rails</code></a> and start working on Node only!</p>
<pre class="ruby">server = Server.new 3000
server.start do
  [200, {'Content-Type' =&gt; 'text/plain'}, ["Hello World!n"]]
end
</pre>
<p>For it I had to write the <code>Server</code> class:</p>
<pre class="ruby">class Server
  def initialize port
    @http = `require('http')`
    @port = port
  end

  def start &amp;block
    %x{
      this.http.createServer(function(req, res) {
        var rackResponse = (block.call(block._s, req, res));
        res.end(rackResponse[2].join(' '));
      }).listen(this.port);
    }
  end
end
</pre>
<p>Put them in <code>app.opal</code> and then start the server:</p>
<pre class="bash">$ npm install -g opal      # &lt;&lt;&lt; install the NPM package first!
$ opal-node app
</pre>
<p><img alt="The running app" src="http://f.cl.ly/items/352N3A2U0c013I391i0Q/Schermata%2006-2456088%20alle%206.18.09%20pm.png" /></p>
<h3>YAY! now I can roflSCALE all of my RAils apps!!1</h3>
<p><em>Cross posted from <a href="http://elia.schito.me/post/24751394395/use-node-js-for-speed-roflscaletips">Eliæ</a></em></p>