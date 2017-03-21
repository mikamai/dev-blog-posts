---
ID: 230
post_title: >
  NTML authentication for Rails from
  inside Microsoft™ ActiveDirectory
author: Elia Schito
post_date: 2014-07-04 16:06:58
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/04/ntml-authentication-for-rails-from-inside/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/90765847079/ntml-authentication-for-rails-from-inside
tumblr_mikamayhem_id:
  - "90765847079"
---
<p>I ended up with a decent setup in which the whole authentication is handled by IIS on a Windows machine that lives inside the ActiveDirectory tree. Adapting from <a href="http://blog.trekforth.com/2011/06/installing-spree-on-windows-2003-server.html">these instructions</a>.</p>

<p>IIS will act as a reverse proxy to your Rails app (typically installed on a *nix server, apache+passenger in my case).</p>

<p>The secret resides in configuring IIS to handle NTLM and then adding this nifty plugin that will basically reproduce the <code>mod_proxy</code> api for IIS.</p>

<p>Here&rsquo;s an <code>iirf.ini</code> example:</p>
    <pre><code class="apache"># NOTE: 
# This file should be placed in the IIS document root 
# for the application

StatusInquiry ON
RewriteLogLevel 3
RewriteLog ....TEMPiirf
RewriteCond %{REQUEST_FILENAME} -f
RewriteRule ^.*$ - [L]
ProxyPass ^/(.*)$ <a href="http://1.2.3.4:80/%241">http://1.2.3.4:80/$1</a>
ProxyPassReverse / <a href="http://1.2.3.4/">http://1.2.3.4/</a>
</code></pre>

<p>With this setup you can rely on the fact that the authentication is performed by IIS and you only get authenticated request with the authentication information stored inside <code>HTTP_AUTHORIZATION</code>.</p>

<p>To parse the user data from the auth header I used <code>net-ntlm</code>:</p>
    <pre><code class="ruby">require 'kconv'
require 'net/ntlm'

if /^(NTLM|Negotiate) (.+)/ =~ env["HTTP_AUTHORIZATION"]
  encoded_message = $2
  message = Net::NTLM::Message.decode64(encoded_message)
  user = Net::NTLM::decode_utf16le(message.user)
end
</code></pre>

<p>After that you can even connect to the LDAP ActiveDirectory interface and fetch details about the user.</p>