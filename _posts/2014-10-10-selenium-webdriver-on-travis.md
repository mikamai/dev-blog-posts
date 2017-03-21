---
ID: 199
post_title: Selenium WebDriver on Travis
author: Nicola Racco
post_date: 2014-10-10 13:30:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/10/selenium-webdriver-on-travis/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/99643679599/selenium-webdriver-on-travis
tumblr_mikamayhem_id:
  - "99643679599"
---
Selenium on Travis may seem impossible because Travis has no window manager, instead it's possible thank to xvfb and even easy to setup.

First of all, ensure to setup your app to use selenium for integration tests. Then push your stuff. Travis tests will fail (because it doesn't know about selenium).

Now, **let Travis meet Selenium!**

We have to enable the virtual framebuffer on travis. You can do this appending the following to your `.travis.yml`:

```yaml
before_script:
- &quot;sh -e /etc/init.d/xvfb start&quot;
- &quot;export DISPLAY=99:0&quot;
```

We also need to download and execute selenium. To do this add the following (always inside `before_script`):

```yaml
- &quot;wget http://selenium-release.storage.googleapis.com/2.43/selenium-server-standalone-2.43.1.jar&quot;
- &quot;java -jar selenium-server-standalone-2.43.1.jar &gt; /dev/null &amp;&quot;
```

Commit your `.travis.yml` changes and push. Travis tests will **NOW** pass :-)