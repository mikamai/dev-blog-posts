---
ID: 205
post_title: Tips to improve WordPress security
author: Andrew Coates
post_date: 2014-09-24 15:17:19
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/24/tips-to-improve-wordpress-security/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/98310949954/tips-to-improve-wordpress-security
tumblr_mikamayhem_id:
  - "98310949954"
---
A few weeks ago, a website I had worked on became unavailable, and I investigated to find that it had been compromised, or hacked. A file, &lsquo;indix.php&rsquo;, was added to the root directory on the fileserver. The file contained some referral to a Ukrainian website, but it was enough to take the site down. 
<br /><br />
The site was a WordPress website. WP is quite renown for poor security, but that&rsquo;s not to say you can&rsquo;t make it secure. I&rsquo;d like to now write about how I have increased the security of this website to prevent future problems.
<br /><br />
The first thing I can recommend is to install the WP Security Scan plugin to see how secure your site is currently <a href="https://wordpress.org/plugins/wp-security-scan/">from here.</a>

<h1>File Permissions</h1>
The general rule for WordPress file permissions is to keep file directories as 755, which means that a user can read, write and execute, and to keep individual files as 644. This means that the owner can read/write the file and other users (the world) can only read it. One exception to this rule is the &rsquo;.htaccess&rsquo; file, which you should keep on 400 permission, meaning that only you and the server can read the file.
<br /><br /><h1>.htaccess</h1>
A very important file to web development. You can use &rsquo;.htaccess&rsquo; to control security throughout your site.
<br /><br />
Prevent directory browsing:
<pre><code>
 Options All -Indexes
</code></pre>
Protecting wp-config.php file:
<pre><code>

Order Allow,Deny
Deny from all
</code></pre>
Restrict access to any .php file:
<pre><code>

AuthUserFile /etc/httpd/htpasswd
AuthType Basic
AuthName "restricted"
Order Deny,Allow
Deny from all
Require valid-user
Satisfy any
<code></code></code></pre>

<h1>AskApache Password Protection Plugin</h1>
Another great plugin that will add a brick wall to your PHP files. <a href="https://wordpress.org/plugins/askapache-password-protect/">Download from here.</a>

<h1>Moving the wp-config.php file from the root</h1>
You should move the 'wp-config.php&rsquo; file one directory up from the root WordPress folder.

This tip is quite controversial, but after reading through the answers posted <a href="http://wordpress.stackexchange.com/questions/58391/is-moving-wp-config-outside-the-web-root-really-beneficial">here</a> I&rsquo;m convinced this is a good decision. 

<h1>Keeping passwords secure</h1>
I&rsquo;m sure this was the reason the site I worked on was hacked; poor passwords. I guess that a hacker ran a pass on millions of WordPress sites using a large word list, effectively guessing the user/password combination. The solution to this is simple: use a better password. One with numbers, capital letters, symbols. I feel <a href="http://xkcd.com/936/">this comic</a> illustrates the idea well.

<h1>Keeping WordPress up-to-date</h1>
This is self-explanatory. Always check for updates.