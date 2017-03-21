---
ID: 292
post_title: >
  Rotating an UIView around an arbitrary
  point
author: Emanuel Carnevale
post_date: 2014-01-29 14:33:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/29/rotating-an-uiview-around-an-arbitrary-point/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/74941935186/rotating-an-uiview-around-an-arbitrary-point
tumblr_mikamayhem_id:
  - "74941935186"
---
<p>Developing on iOS often brings joy thanks to default behaviours and design choices made by the Apple designers, but you can get many headaches too when trying to step off the beaten path.
Fortunately this time the solution is easy and straightforward and the headaches are saved for other more consistent problems. :)</p>

<p>Recently I had to let a user rotate a View on an iPhone app using his fingers, but when tested, the view rotated around its center.</p>

<p>I wanted it to mimic the <em>real world</em>, where if you do it on a piece of paper lying on a table, it usually spins  around your two fingers.</p>

<p><img src="http://68.media.tumblr.com/dbbb88868053e06e2e4bb9094168231b/tumblr_inline_n05nkxJCvG1qz4sg2.png" alt="" /><img src="http://68.media.tumblr.com/f6f5b01fc9af2a5d8a261c84f5981980/tumblr_inline_n05nm9RC331qz4sg2.png" alt="" /></p>

<p>To rotate a view, we need to apply to it an affine transformation, and looking at the <a href="https://developer.apple.com/library/ios/documentation/uikit/reference/uiview_class/uiview/uiview.html#//apple_ref/occ/instp/UIView/transform">UIView Class Reference</a> we read:
<em>&ldquo;The origin of the transform is the value of the center property, <strong>or the layer’s anchorPoint property</strong> if it was changed.&rdquo;</em></p>

<p>So the <code>anchorPoint</code> of the view&rsquo;s layer is the point around which transformations (in this case a rotation) are applied. If we change it accordingly to a middle point between our two fingers touches we are set and running!</p>

<p><img src="http://i.imgur.com/VZauulU.jpg" alt="" /></p>

<p>Unfortunately its coordinates are normalised to the UIView boundaries, ranging from 0 to 1, with the center represented by 0.5, 0.5.</p>

<p>The trick will be done by this little snippet of code:</p>

<pre>
CGPoint firstTouch  = [(UIRotationGestureRecognizer*)sender locationOfTouch:0 inView:self];
CGPoint secondTouch = [(UIRotationGestureRecognizer*)sender locationOfTouch:1 inView:self];

self.layer.anchorPoint = CGPointMake(((firstTouch.x + secondTouch.x)/2)/self.bounds.size.width, ((firstTouch.y + secondTouch.y)/2)/self.bounds.size.height);
</pre>

<p>With self being our UIView class. If you want to try a demo, you can clone our simple <a href="https://github.com/mikamai/Rotation-test">demo app on Github</a>.</p>