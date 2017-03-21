---
ID: 128
post_title: 'A &#8220;desperate times call for desperate measures&#8221; hack'
author: Gianluca
post_date: 2015-05-06 13:14:14
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/06/a-desperate-times-call-for-desperate-measures/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/118279905554/a-desperate-times-call-for-desperate-measures
tumblr_mikamayhem_id:
  - "118279905554"
---
Last week I had to solve some tricky problems related to a Web application that needs to run locally on some tablets in kiosk mode without any Internet connection.

With the term kiosk mode I mean that the browser used to run the application, in this case Google Chrome, is wrapped in another application that is acting like a sandbox to prevent users from accessing the Operating System or any other application.

<!--more-->

The Web application I am talking about is based on WordPress and on a custom Javascript engine handling some cool and shiny animations.

The problem was that some of the tablets were simply found, after a while, on the “T-Rex page” of Google Chrome:

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/97d20e3fac8cef49454f1de1cb7edb93/tumblr_inline_nnrkpofnu21ss63kf_540.gif" alt="image" /></figure>Furthermore, given the kiosk mode, there was no simple way to go back to the previous page or to restart the application without disassembling the structure holding the tablets.

And the option to let the users play with the T-Rex wasn’t a good one for the client…

Anyway that was strange as we properly replaced all the external links inside the application with local references.

After a little bit of debugging directly on site (i.e. where the tablets were exposed to the public) I found the problem: the double click, or better the long click.

The long press would display the context menu of Chrome, letting the user access some interesting options like “Inspect element”, “View page source” and “Search with Google”. The last option in particular was the source of the over mentioned “T-Rex page”.

To fix the problem I wrote this little and trivial Javascript snippet:
<pre><code>
// PREVENT RIGHT CLICK
$(document).on("contextmenu",function(e){
  e.preventDefault();
});
</code>
</pre>
The Javascript engine was based on jQuery and so I too relied on it for my snippet.

After successfully applying the fix on some of the tablets I noticed that there was still a problem. The right click was properly disabled everywhere except from the video player. Indeed the player, i.e. JWplayer, was still showing its specific context menu with an external link that was still causing the user to land on the “T-Rex page”. Here it is the bad context menu:

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/9a717e39ed698dc5ad1fad40fccf7f1b/tumblr_inline_nnrkynjJEP1ss63kf_540.png" alt="image" /></figure>At the time I became aware of the problem I was on site without any Internet connection and so I couldn’t ask to Google for the proper solution. The first thing that came to my mind was to directly hack the player. We had already created a “local” version of the player to handle another error related to the fetch of the skins that was causing the player to break up.

JWplayer isn’t indeed designed to work offline.

After a few failures in disabling the player right click (I was working directly on the minified version of the local html5 player, i.e. <code>jwplayer.html5.js</code>) I decided to take a different approach.

I looked at the underlying link of the context menu option and I simply replaced it with the link to the current page.

It wasn’t easy because there were many different places where the link was hardcoded and where it was used but after a while I managed to find the last location where it was used right before it was printed out in the context menu.

Just for the record I replaced this line:
<pre><code>
;f.aboutlink=e+("pro"==h?"p":"premium"==h?"r":"enterprise"==h?"e":"ads"==h?"a":"f")}f.abouttext
</code>
</pre>
with this:
<pre><code>
;f.aboutlink=window.location)}f.abouttext
</code>
</pre>
The over mentioned lines may vary based on the version of the player. The one we were using is the 6.12.4956.

I know this is a bad and ugly hack but It saved my day in a situation were I didn’t have the proper knowledge about the application to take the appropriate action.

I hope you will never find in a situation like the one I described but if you will, my personal advice is to follow your instinct and what Elsa say.

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/66b96ca8b704f779293111f707f00233/tumblr_inline_nnrksmsBVG1ss63kf_500.gif" alt="image" /></figure>Cheers! :)