---
ID: 101
post_title: Full string power with CFStringTransform
author: Emanuel Carnevale
post_date: 2015-07-08 13:34:36
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/08/full-string-power-with-cfstringtransform/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/123548699509/full-string-power-with-cfstringtransform
tumblr_mikamayhem_id:
  - "123548699509"
---
<p>Powerful languages have great string manipulation capabilities, today we&rsquo;ll have a look at what Objective-C offers.</p>

<p><code>CFStringTransform</code> is part of the <code>Core Foundation</code> framework, it will let you manipulate strings of both NSString and CFString type.</p>

<p>So, in Objective-C we have
<code>Boolean CFStringTransform ( CFMutableStringRef string, CFRange *range, CFStringRef transform, Boolean reverse );</code>
(and in Swift)
<code>func CFStringTransform(_ string: CFMutableString!, _ range: UnsafeMutablePointer&lt;CFRange&gt;, _ transform: CFString!, _ reverse: Boolean) -&gt; Boolean</code></p>

<p>the arguments it requires are really straightforward: a string, a range in which to apply the transformation, the transformation itself and a boolean to specify whether or not use the inverse transformation.</p>

<p>The transformations are very powerful and range from classic Uppercase, Lowercase, Titlecase, Full/Halfwidth conversions to handling Unicode characters. We will look to a couple of examples, but for the whole list of available transformations, please check <a href="https://developer.apple.com/library/prerelease/ios/documentation/CoreFoundation/Reference/CFMutableStringRef/index.html#//apple_ref/doc/constant_group/Transform_Identifiers_for_CFStringTransform">the official list</a></p>

<h2>Give a name to Unicode Characters</h2>

<p>Sometimes you need to normalize Unicode input to ascii, and since modern devices now support emoji, you can now transliterate emoji characters:
so this string: 🐝🌵💀👽
becomes &ldquo;{HONEYBEE}{CACTUS}{SKULL}{EXTRATERRESTRIAL ALIEN}&rdquo;</p>

<p>in swift we just have to do:
<code>var str = NSMutableString(string: "🐝🌵💀👽")
CFStringTransform(str, nil, kCFStringTransformToUnicodeName, Boolean(0))</code></p>

<h2>Transliterate non latin characters</h2>

<p>From emoji to pictograms the step is easy, so we can also transliterate japanese hiragana or chinese characters to their phonetic equivalent:</p>

<p><code>
var japaneseTroublesome = NSMutableString(string: "めんどくさい")
CFStringTransform(japaneseTroublesome, nil, kCFStringTransformLatinHiragana, Boolean(1))
japaneseTroublesome
</code></p>

<p><code>
var chineseHello = NSMutableString(string: "你好")
CFStringTransform(chineseHello, nil, kCFStringTransformMandarinLatin, Boolean(0))
chineseHello
</code></p>

<p>for the japanese transformation note we had to use the inverse transformation since the available one is from Latin to Hiragana.</p>

<p>We used swift to show a couple of examples so you can easily play with it in a playground in XCode.</p>