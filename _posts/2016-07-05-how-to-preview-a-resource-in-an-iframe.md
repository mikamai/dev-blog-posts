---
ID: 21
post_title: 'How to: Preview a resource in an iframe'
author: Nicola Racco
post_date: 2016-07-05 12:50:19
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/07/05/how-to-preview-a-resource-in-an-iframe/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/146942910259/how-to-preview-a-resource-in-an-iframe
tumblr_mikamayhem_id:
  - "146942910259"
---
Let's say I have a form to create/edit a resource. On a side of this form I want to show a live preview of my resource. Whenever the form changes I need to update the preview.

At a first glance the solution seems pretty easy, right? Use an IFrame and update the `src` update on every change.<!--more-->

Let's start with a naïve implementation of this preview feature:

```html
&lt;!-- app/views/admin/posts.html --&gt;

&lt;form class=&quot;form&quot; action=&quot;action&quot;&gt;
&lt;div class=&quot;row&quot;&gt;
&lt;div class=&quot;col-sm-6&quot;&gt;
&lt;div class=&quot;form-group&quot;&gt;&lt;label for=&quot;post_title&quot;&gt;Title&lt;/label&gt;
&lt;input id=&quot;post_title&quot; class=&quot;form-control&quot; name=&quot;post[title]&quot; type=&quot;text&quot; /&gt;&lt;/div&gt;
&lt;div class=&quot;form-group&quot;&gt;&lt;label for=&quot;post_text&quot;&gt;Text&lt;/label&gt;
&lt;textarea id=&quot;post_text&quot; class=&quot;form-control&quot; cols=&quot;50&quot; name=&quot;post[text]&quot; rows=&quot;10&quot;&gt;&lt;/textarea&gt;&lt;/div&gt;
&lt;div class=&quot;form-group&quot;&gt;&lt;label for=&quot;post_tags&quot;&gt;Tags&lt;/label&gt;
&lt;input id=&quot;post_tags&quot; class=&quot;form-control&quot; name=&quot;post[tags]&quot; type=&quot;text&quot; /&gt;&lt;/div&gt;
&lt;/div&gt;
&lt;div class=&quot;col-sm-6&quot;&gt;&lt;iframe style=&quot;width: 100%; min-height: 450px;&quot; width=&quot;300&quot; height=&quot;150&quot; data-src=&quot;http://plugins.jquery.com&quot;&gt;&lt;/iframe&gt;&lt;/div&gt;
&lt;/div&gt;
&lt;/form&gt;&lt;script type=&quot;text/javascript&quot;&gt;
refreshPreview = function() {
  var $iframe = $(&#039;iframe&#039;),
      $form = $(&#039;iframe&#039;).parents(&#039;form&#039;);
  $iframe.attr(&#039;src&#039;, $iframe.data(&#039;src&#039;) + &quot;?&quot; + $form.serialize());
};

$(function() {
  $(&#039;form input&#039;).change(refreshPreview);
  refreshPreview();
});
&lt;/script&gt;
```

First problem: I was trying to send form data through a GET request, this entails that if the form params become too long I may end up receiving a "Request URI too large" error from the web server. Second problem: it’s not appropriate security-wise to send form data through GET requests, they may contain sensible informations.

So, the real question is: how can I render a POST response in an IFrame?

First things first, let’s do the server part. In my case, with Rails, this means adding a custom route for my resource:

```ruby
# config/routes.rb

namespace :admin do
resources :posts do
post :preview, on: :collection
end
end
```

And adding a preview action in controller:

```ruby
# app/controllers/admin/posts_controller.rb

class Admin::PostsController &lt; Admin::BaseController
layout &#039;admin&#039;

def preview
@post = Post.new params.require(:post).permit(:title, :text, :tags)
render &#039;posts/show&#039;, layout: &#039;application&#039;
end
end
```

The preview action is rendering the frontend view with the application layout. Let’s now deal with the IFrame. It cannot do POST requests of course, but we can do them with the `XMLHttpRequest` object, and thank to HTML5 there is now the `FormData` object that encapsulates form fields.

This is the final javascript:

```javascript
fetchPreview = function(url, formData, callback) {
var req = new XMLHttpRequest();
req.onreadystatechange = function() {
if (req.readyState != 4) { return false };
callback(req.responseText);
};
req.open(&quot;POST&quot;, url);
req.send(formData);
};

previewRenderCompleted = function() {};

refreshPreview = function() {
var $iframe = $(&#039;iframe&#039;),
$form = $(&#039;iframe&#039;).parents(&#039;form&#039;);

fetchPreview($iframe.data(&#039;src&#039;), new FormData($form[0]), function(content) {
$iframe.one(&#039;load&#039;, previewRenderCompleted);
$iframe.contentDocument.open();
$iframe.contentDocument.write(content != null ? content : &#039;&#039;);
$iframe.contentDocument.close();
});
};

$(function() {
$(&#039;form input&#039;).change(refreshPreview);
refreshPreview();
});
```

In the end what we do is to write the returned content in the iframe using its `contentDocument`. The good thing about this solution is that by using IFrame's `contentDocument` to write the response we can also receive the load event on the iframe itself.

This allowed me to add another piece to my preview: scaling the iframe content to fit the iframe itself. This is pretty easy to do in `previewRenderCompleted`:

```javascript
previewRenderCompleted = function() {
  var $iframe     = $(&#039;iframe&#039;),
      $iframeDoc  = $($iframe.contentDocument),
      iframeWidth = $iframe.width(),
      ratio       = iframeWidth / $iframeDoc.width(),
      cssScale    = &quot;scale(&quot; + ratio + &quot;)&quot;,
      cssOrigin   = &quot;0 0&quot;;
  $(&#039;body&#039;, $iframeDoc).css({
    &quot;-webkit-transform-origin&quot; : cssOrigin,
    &quot;-webkit-transform&quot;        : cssScale,
    &quot;-ms-transform-origin&quot;     : cssOrigin,
    &quot;-ms-transform&quot;            : cssScale,
    &quot;-transform-origin&quot;        : cssOrigin,
    &quot;-transform&quot;               : cssScale
  });
};
```