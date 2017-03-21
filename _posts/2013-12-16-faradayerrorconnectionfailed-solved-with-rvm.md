---
ID: 309
post_title: >
  Faraday::Error::ConnectionFailed solved
  with RVM
author: Matteo Zobbi
post_date: 2013-12-16 10:19:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2013/12/16/faradayerrorconnectionfailed-solved-with-rvm/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/69160066816/faradayerrorconnectionfailed-solved-with-rvm
tumblr_mikamayhem_id:
  - "69160066816"
---
Recently I’ve discovered that recent versions of RVM have a utility to diagnose and resolve errors caused by outdated certificate files.<!--more-->

If you ever get an error like that:

<em>openssl::ssl::sslerror: ssl_connect returned=1 errno=0 state=sslv3 read server certificate b: certificate verify failed</em>

Here are the commands you need:
<pre><code>$ rvm osx-ssl-certs status all
Certificates for /opt/sm/pkg/versions/openssl/1.0.1e/ssl/cert.pem: Old.
Certificates for /usr/local/etc/openssl/cert.pem: Old.
$ rvm osx-ssl-certs update all
Updating certificates for /opt/sm/pkg/versions/openssl/1.0.1e/ssl/cert.pem: Updating certificates in '/opt/sm/pkg/versions/openssl/1.0.1e/ssl/cert.pem'.
Updated.
Updating certificates for /usr/local/etc/openssl/cert.pem: Updating certificates in '/usr/local/etc/openssl/cert.pem'.
Updated.
</code></pre>
The first command should show everything up to date:
<pre><code>$ rvm osx-ssl-certs status all
Certificates for /opt/sm/pkg/versions/openssl/1.0.1e/ssl/cert.pem: Up to date.
Certificates for /usr/local/etc/openssl/cert.pem: Up to date.
</code></pre>
If you still have problems, check this article: <a title="openssl certificate verify failed" href="http://railsapps.github.io/openssl-certificate-verify-failed.html">http://railsapps.github.io/openssl-certificate-verify-failed.html</a>