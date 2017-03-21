---
ID: 8
post_title: >
  Flask and python-saml on Amazon Elastic
  Beanstalk
author: Diomede Tripicchio
post_date: 2016-11-08 14:40:04
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/11/08/flask-and-python-saml-on-amazon-elastic-beanstalk/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/152899294619/flask-and-python-saml-on-amazon-elastic-beanstalk
tumblr_mikamayhem_id:
  - "152899294619"
---
In the past weeks I have worked on something different than usual, a python project. This project was started a couple a years ago with [Flask](http://flask.pocoo.org/) and I need to make some improvements.

Back in the days I used to work with Django, so it wasn’t difficult getting familiar with Flask. The major problem was to get python-saml work on Amazon Elastic Benstalk correctly and after a lot of googling I managed to get all the pieces work togheter, but I couldn’t find a solution with all the information in one place, so I decided to write this post.

<!--more-->

I assume you have a basic knoweldge of how AWS EB works :D

As the time I’m writing the platform in use on my AWS EB instances is `64bit Amazon Linux 2016.03 v2.1.6 running Python 2.7` and we are serving our website with `apache` and `mod_wsgi`

First of all, why do we want to use [python-saml](https://github.com/onelogin/python-saml)?

I need to implement OneLogin for a secure login system. Nothing complex from the code point of view, you can follow the various examples that you find in the official repo.

The hardest part is to get all the required libraries installed and **working** on the machine. Let’s start with the library we can install from the default `yum` repository available on AWS EB.

I have a file `00-depedendecies.config` under the folder `.ebextensions` where I put all the extra libraries I need to install:

```yaml
packages:
  yum:
    ...
    libxml2-devel: []
    libtool-ltdl-devel: []
    libgcrypt-devel: []
    libgpg-error-devel: []
    libxslt-devel: []
```

And this is the first step, but are still missing `xmlsec1`, to install this library we have to rely on the RPM packages from the fedora repositories.

Let’s create another file `01-xmlsec.config` always under our `.ebextensions` directory:

```yaml
packages:
  rpm:
    xmlsec1: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-1.2.19-6.fc22.x86_64.rpm
    xmlsec1-devel: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-devel-1.2.19-6.fc22.x86_64.rpm
    xmlsec1-openssl: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-openssl-1.2.19-6.fc22.x86_64.rpm
    xmlsec1-openssl-devel: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-openssl-devel-1.2.19-6.fc22.x86_64.rpm
```

Great, now we have all the libraries we need to work with `python-saml` only it doesn't work. When we try this import `import dm.xmlsec.binding` in a python console we receive a funny error:

`Segmentation fault.`

After some research I ended on [this issue](https://github.com/onelogin/python-saml/issues/30) and added a command to patch the `/usr/bin/xmlsec1-config` file, and now our `01-xmlsec.config` looks like:

```yaml
packages:
  rpm:
    xmlsec1: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-1.2.19-6.fc22.x86_64.rpm
    xmlsec1-devel: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-devel-1.2.19-6.fc22.x86_64.rpm
    xmlsec1-openssl: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-openssl-1.2.19-6.fc22.x86_64.rpm
    xmlsec1-openssl-devel: https://kojipkgs.fedoraproject.org/packages/xmlsec1/1.2.19/6.fc22/x86_64/xmlsec1-openssl-devel-1.2.19-6.fc22.x86_64.rpm
commands:
  01_patch_xmlsec_config:
    command: &quot;sed -i &#039;s/LIBLTDL=1 -I/LIBLTDL=1 -DXMLSEC_NO_SIZE_T -I/&#039; /usr/bin/xmlsec1-config&quot;
```

At this point our app is ready to talk with OneLogin, but if we try to login it responds with a 504 Timeout and in the logs we can see `Script timed out before returning headers`.

Googling the error message points us to the right solution, [WSGIApplicationGroup](http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIApplicationGroup.html):

The last step to get all to work is to add a configuration for `mod_wsgi` and to do this we need a third file under the `.ebextensions` directory.

`02-custom_httpd_conf.config`

```yaml
files:
  &quot;/etc/httpd/conf.d/wsgi_custom.conf&quot;:
    mode: &quot;000644&quot;
    owner: root
    group: root
    content: |
      WSGIApplicationGroup %{GLOBAL}
```

With these lines we are creating a new file `/etc/http/conf.d/wsgi_custom.conf` with `WSGIApplicationGroup %{GLOBAL}` as content, and thanks to the default AWS EB `httpd.conf` all files under `/etc/httpd/conf.d/` are included by default.

*If you want more specific information why we need this configuration you can read the mod_wsgi documentation from the link above.
TL;DR: we use a third party C extension module and we need this configuration to get all work.*

And now we are done! Everything works fine and no more <code>504</code> erorrs are raised.

I hope you have found the post useful.

Bye, see you next time!