---
ID: 275
post_title: PHP, beware of the Sharingan!
author: Gianluca
post_date: 2014-03-07 08:55:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/03/07/php-beware-of-the-sharingan/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/78834285987/php-beware-of-the-sharingan
tumblr_mikamayhem_id:
  - "78834285987"
---
Do you know what Sharingan is? Yes the one from the Naruto manga/anime :D

<img src="http://68.media.tumblr.com/0e68ac037a80c612ef61e01946075661/tumblr_inline_n1utdowHTv1ss63kf.gif" alt="image" />

<!--more-->

If not, it is basically a visual ability that let their holder (i.e. ninjas of the Uchiha clan) do a lot of creepy/cool things.

Between them there is the capability to create illusions in the minds of the victims that meet their gaze and even trap them in a total imaginary world.

Well, a few weeks ago I partially felt prey of something similar.

I was testing a PHP application (i.e. my thesis first draft) that should be able to parse PHP source code and create an in memory representation of it. During the analysis however the application died silently. No errors, no warnings…

Obviously I began to look around and after twenty minutes I discovered that I was prey of the PHP error suppression operator (i.e. <a href="http://php.net/manual/en/language.operators.errorcontrol.php">@</a>) that was hiding the error. In particular the operator was used to suppress the errors raised by the <a href="http://php.net/token_get_all">token_get_all()</a> function utilized inside the fantastic <a href="https://github.com/nikic/PHP-Parser">Nikic PHP-Parser</a>.

As I discovered this, I released myself from the illusion and discovered the problem (i.e. not enough memory to analyze a file with roughly 30K LOC).

Moral: don’t let the Sharingan (i.e. @) hypnotize you!

P.S: yes I know…@ seems more like Rinnegan! :P