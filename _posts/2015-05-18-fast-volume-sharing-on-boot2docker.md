---
ID: 123
post_title: Fast Volume Sharing on Boot2Docker
author: Giovanni Intini
post_date: 2015-05-18 14:57:53
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/18/fast-volume-sharing-on-boot2docker/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/119280676044/fast-volume-sharing-on-boot2docker
tumblr_mikamayhem_id:
  - "119280676044"
---
<p>When discussing docker with fellow developers on macs, I often hear complaints about <a href="http://boot2docker.io">boot2docker</a> doesn’t work as expected, or how the performance of shared volumes is slow.</p>

<p>I hear their complaints, after all I hit my head more than once against the same problems. Eventually I found a sweet solution that works for me, and has been battle tested by my coworkers too, a setup based on <a href="https://vagrantcloud.com/parallels/boxes/boot2docker">parallels, vagrant and boot2docker</a>.</p>

<p>Using <a href="http://www.parallels.com">parallels</a> gives us a less bug ridden NFS implementation, needed to avoid the performance problems of shared folders.</p>

<p>We use <a href="https://www.vagrantup.com">vagrant</a> to tie everything up and we get a nice fast boot2docker that is a bit less magic than the default one, but also less opaque.</p>

<p>Here’s my <code>Vagrantfile</code>, slightly tuned to also support the plain non-parallels boot2docker image.</p>



<pre class="prettyprint"><code class="language-ruby hljs"><span class="hljs-constant">VAGRANTFILE_API_VERSION</span> = <span class="hljs-string">"2"</span>

<span class="hljs-constant">Vagrant</span>.configure(<span class="hljs-constant">VAGRANTFILE_API_VERSION</span>) <span class="hljs-keyword">do</span> |config|
  config.vm.box = <span class="hljs-string">"yungsang/boot2docker"</span>
  config.vm.network <span class="hljs-string">"private_network"</span>, <span class="hljs-symbol">ip:</span> <span class="hljs-constant">ENV</span>[<span class="hljs-string">'BOOT2DOCKER_IP'</span>] || <span class="hljs-string">"10.211.55.5"</span>

  config.vm.provider <span class="hljs-string">"parallels"</span> <span class="hljs-keyword">do</span> |v, override|
    override.vm.box = <span class="hljs-string">"parallels/boot2docker"</span>
    override.vm.network <span class="hljs-string">"private_network"</span>, <span class="hljs-symbol">type:</span> <span class="hljs-string">"dhcp"</span>
  <span class="hljs-keyword">end</span>

  <span class="hljs-comment"># http</span>
  config.vm.network <span class="hljs-string">"forwarded_port"</span>, <span class="hljs-symbol">guest:</span> <span class="hljs-number">80</span>, <span class="hljs-symbol">host:</span> <span class="hljs-number">80</span>, <span class="hljs-symbol">auto_correct:</span> <span class="hljs-keyword">true</span>
  config.vm.network <span class="hljs-string">"forwarded_port"</span>, <span class="hljs-symbol">guest:</span> <span class="hljs-number">443</span>, <span class="hljs-symbol">host:</span> <span class="hljs-number">443</span>, <span class="hljs-symbol">auto_correct:</span> <span class="hljs-keyword">true</span>

  <span class="hljs-comment"># rack</span>
  config.vm.network <span class="hljs-string">"forwarded_port"</span>, <span class="hljs-symbol">guest:</span> <span class="hljs-number">9292</span>, <span class="hljs-symbol">host:</span> <span class="hljs-number">9292</span>, <span class="hljs-symbol">auto_correct:</span> <span class="hljs-keyword">true</span>

  config.vm.synced_folder <span class="hljs-string">"/Users/intinig/src"</span>, <span class="hljs-string">"/Users/intinig/src"</span>, <span class="hljs-symbol">type:</span> <span class="hljs-string">"nfs"</span>, <span class="hljs-symbol">bsd__nfs_options:</span> [<span class="hljs-string">"-maproot=0:0"</span>], <span class="hljs-symbol">map_uid:</span> <span class="hljs-number">0</span>, <span class="hljs-symbol">map_gid:</span> <span class="hljs-number">0</span>
  config.vm.synced_folder <span class="hljs-string">"/Users/intinig/src"</span>, <span class="hljs-string">"/src"</span>, <span class="hljs-symbol">type:</span> <span class="hljs-string">"nfs"</span>, <span class="hljs-symbol">bsd__nfs_options:</span> [<span class="hljs-string">"-maproot=0:0"</span>], <span class="hljs-symbol">map_uid:</span> <span class="hljs-number">0</span>, <span class="hljs-symbol">map_gid:</span> <span class="hljs-number">0</span>
<span class="hljs-keyword">end</span></code></pre>

<p>The last two lines of the <code>Vagrantfile</code> are what allows us to run fast volume sharing. In this case, but you might want to adapt them to your needs, I share my <code>~/src</code> folder as /src and as <code>/Users/intinig/src</code> on the boot2docker vm.</p>

<p>To start the boot2docker vm you just enter the folder where you have the <code>Vagrantfile</code> and launch <code>vagrant up</code>.</p>

<p>Let me know how it works for you, happy dockering!</p>