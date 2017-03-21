---
ID: 102
post_title: Docker on your Mac using Vagrant
author: Emanuele Serafini
post_date: 2015-07-06 12:57:35
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/06/docker-on-your-mac-using-vagrant/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/123367328039/docker-on-your-mac-using-vagrant
tumblr_mikamayhem_id:
  - "123367328039"
---
<p>The first time I tried docker I used boot2docker without complete knowledge of it. The idea behind boot2docker is really quite clever: since we can&rsquo;t run docker natively on Mac, we install a bare-bones Linux VM that can run docker, and then we communicate with it using a docker client running on our host.</p>

<p>After some initial tests I started looking for a different solution considering that I already installed Vagrant on my machine.</p>

<p>This guide explain how to use docker on mac using <a href="https://www.vagrantup.com/">Vagrant</a> with VirtualBox or Parallels as provider without installing boot2docker.</p>

<p>Prerequisites:</p>

<ul><li><a href="http://brew.sh/">Homebrew</a></li>
<li><a href="http://caskroom.io/">Cask</a></li>
</ul><h2>Vagrant installation</h2>

<pre><code>$ brew cask install vagrant
</code></pre>

<h2>Provider installation</h2>

<p>Vagrant has the ability to manage different provider such as VirtualBox, Parallels and VMware. By default it uses VirtualBox which is free, other providers need license to run.</p>

<p>If you want to use default provider just install VirtualBox:</p>

<pre><code>$ brew cask install virtualbox
</code></pre>

<p>Otherwise if you have already installed Parallels on your machine:</p>

<pre><code>$ vagrant plugin install vagrant-parallels
</code></pre>

<p>The Vagrant plugin installer will automatically download and install vagrant-parallels plugin.</p>

<h2>Docker installation</h2>

<pre><code>$ brew install docker
</code></pre>

<h2>Build and run your docker VM</h2>

<p>Create a new directory, name it <code>docker</code> and create <code>Vagrantfile</code>:</p>

<pre><code>$ mkdir docker
$ cd docker
$ touch Vagrantfile
</code></pre>

<p>Now edit your Vagrantfile and add:</p>

<pre><code>Vagrant.require_version "&gt;= 1.6.5"

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "yungsang/boot2docker"
  config.vm.network "private_network", ip: ENV['BOOT2DOCKER_IP'] || "10.211.55.5"

  config.vm.provider "parallels" do |v, override|
    override.vm.box = "parallels/boot2docker"
    override.vm.network "private_network", type: "dhcp"
  end

  # Fix busybox/udhcpc issue
  config.vm.provision :shell do |s|
    s.inline = &lt;&lt;-EOT
      if ! grep -qs ^nameserver /etc/resolv.conf; then
        sudo /sbin/udhcpc
      fi
      cat /etc/resolv.conf
    EOT
  end

  # Adjust datetime after suspend and resume
  config.vm.provision :shell do |s|
    s.inline = &lt;&lt;-EOT
      sudo /usr/local/bin/ntpclient -s -h pool.ntp.org
      date
    EOT
  end
end
</code></pre>

<p>Now you&rsquo;re able to run your VM:</p>

<pre><code>### VirtualBox
$ vagrant up

### Parallels
$ vagrant up --provider parallels
</code></pre>

<p>Finally, you&rsquo;ll need to set an environment variable called DOCKER_HOST that will tell your Docker client on your host machine the URI for the Docker daemon running on the VM.</p>

<pre><code>$ export DOCKER_HOST="tcp://`vagrant ssh-config | sed -n "s/[ ]*HostName[ ]*//gp"`:2375"
</code></pre>

<p>Once you&rsquo;ve completed all these steps, you should have the Docker client installed and a VM capable of starting Docker containers. To test it out, try the command <code>docker run hello-world</code> from your host:</p>

<pre><code>$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from hello-world
a8219747be10: Pull complete
91c95931e552: Already exists
hello-world:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:aa03e5d0d5553b4c3473e89c8619cf79df368babd18681cf5daeb82aab55838d
Status: Downloaded newer image for hello-world:latest
Hello from Docker.
...
</code></pre>

<p>Good Dockering! ;)</p>

<h3>Tip</h3>

<p>In some case you can encounter this error &ldquo;<strong>Error response from daemon: client and server don&rsquo;t have same version</strong>&rdquo;. To solve this problem install older or newer Docker version on your machine.</p>

<pre><code>$ brew tap homebrew/versions
$ brew install homebrew/versions/docker
$ brew search docker
docker
homebrew/versions/docker133
homebrew/versions/docker141
homebrew/versions/docker150
$ brew install homebrew/versions/docker150
</code></pre>