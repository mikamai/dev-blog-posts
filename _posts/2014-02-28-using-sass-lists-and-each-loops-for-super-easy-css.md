---
ID: 279
post_title: >
  Using Sass lists and each loops for
  super easy CSS branding
author: Ivan Prignano
post_date: 2014-02-28 13:48:34
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/02/28/using-sass-lists-and-each-loops-for-super-easy-css/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/78105748399/using-sass-lists-and-each-loops-for-super-easy-css
tumblr_mikamayhem_id:
  - "78105748399"
---
<p>Lately, here at Mikamai we’ve been working on a large client project that substantially consists in building an API for injecting content in a page. The content is requested via a JS API that accepts some arguments, one of which being the brand the developer needs – based on this brand, the script injects different content that needs to be styled in different ways (one per brand). So, how to approach that last step?</p>
<h3>The problem</h3>
<p>The first solution that came to mind was to assign a different class to the body and, based on that, apply the according styles for each different brand. For example:</p>
<pre>  body {
    /* Default style */
    background-color: #ffffff;

    &amp;.brand-one {
      background-color: #c09853;

      .title {
        color: #fcf8e3;
      }
    }

    &amp;.brand-two {
      background-color: #eaeaea;

      .title {
        color: #a200b8;
      }
    }

    /* And so on.. */
  }
</pre>
<p>This could work, but we&rsquo;ll eventually have some problems. The first (and probably biggest) one is that we have a lot of <strong>duplicated code</strong>, that means an hard-to-mantain codebase and readability concerns.</p>
<h3>Sass to the rescue</h3>
<p>Among Sass&rsquo; awesome features, there are two in particular that could come handy in our case: <strong><a href="http://sass-lang.com/documentation/file.SASS_REFERENCE.html#lists" title="Lists" target="_blank">Lists</a></strong> (arrays, basically) and <strong><a href="http://sass-lang.com/documentation/file.SASS_REFERENCE.html#each-directive" title="Each loops" target="_blank">each loops</a> </strong>(I&rsquo;ll let you guess that). With these tools in our hands, we could convert the above code snippet in something like this:</p>
<pre>$brands:  (brand-one   #c09853 #fcf8e3), 
          (brand-two   #eaeaea #a200b8),
          (brand-three #000000 #222222);

body {
  @each $brand in $brands {
    $division: nth($brand, 1);
    $division-bg-color: nth($brand, 2);
    $division-title-color: nth($brand, 3);

    &amp;.#{$division} {
      background-color: #{$division-bg-color};

      .title {
        color: #{$division-title-color};
      }
    }
  }
}
</pre>
<p>What we&rsquo;re doing here is creating some <strong>nested Sass lists</strong> with 3 values: respectively the <em>brand name</em>, the <em>brand background color</em> and the <em>brand title color</em>. It could be expressed this way:</p>
<pre>$list: (brand-name background-color title-color)</pre>
<p>Then, in the body style declaration, we start a <strong>@each loop</strong> that iterates over each list value and - with the <a href="http://sass-lang.com/documentation/Sass/Script/Functions.html#nth-instance_method" title="nth operation" target="_blank"><strong>nth</strong></a> operation - assigns every nested value to a variable. We then interpolate our newly created variables in our style declaration, resulting in <a href="http://sassmeister.com/gist/9249811" target="_blank">what we wanted at the beginning</a>. Cool, right? But we could improve that solution to be more flexible and clean. How?</p>
<h3>Mixins and variables</h3>
<p>The power of Sass offers again great ways to write modular and easy to read and mantain code. Defining a <strong>mixin</strong> that includes our über-cool function is super easy:</p>
<pre>$brands:  (brand-one $brandone-bgcolor $brandone-title-color), 
          (brand-two $brandtwo-bgcolor $brandtwo-title-color),
          (brand-three $brandthree-bgcolor $brandthree-title-color);

@mixin brand-colors {
  @each $brand in $brands {
    $division: nth($brand, 1);
    $division-color: nth($brand, 2);
    $division-title-color: nth($brand, 3);

    &amp;.#{$division} {
      background-color: #{$division-color};

      .title {
        color: #{$division-title-color};
      }
    }
  }
}
</pre>
<p>As you can see we just defined a mixin (presumably in a separated <em>helper.scss</em> file) that we can trivially include in our main css. Plus, we don&rsquo;t have any <strong>hardcoded color value</strong> in our mixin, since we replaced them with <strong>variables</strong> (that we could declare in a specific <em>var.scss</em> file – separation of concerns, right?). That means that in the end we&rsquo;ll have <strong>3 files</strong>:</p>
<ul><li><span>vars.scss</span></li>
<li><span>helpers.scss</span></li>
<li><span>main.scss</span></li>
</ul><p><span>In <strong>vars.scss</strong> we&rsquo;ll have something like that:</span></p>
<pre>  $brandone-bgcolor: #eaeaea;
  $brandone-title-color: #333333;

  // etc.
</pre>
<p>In <strong>helpers.scss</strong> our &ldquo;brand-colors&rdquo; mixin:</p>
<pre>$brands:  (brand-one $brandone-bgcolor $brandone-title-color), 
          (brand-two $brandtwo-bgcolor $brandtwo-title-color),
          (brand-three $brandthree-bgcolor $brandthree-title-color);

@mixin brand-colors {
  @each $brand in $brands {
    $division: nth($brand, 1);
    $division-color: nth($brand, 2);
    $division-title-color: nth($brand, 3);

    &amp;.#{$division} {
      background-color: #{$division-color};

      .title {
        color: #{$division-title-color};
      }
    }
  }
}
</pre>
<p>In <strong>main.scss</strong> we could do something like that:</p>
<pre>  @import "vars";
  @import "helpers";

  body {
    @include brand-colors;
  }
</pre>
<p>Ta-dah! Modular, extensible code and easy to mantain styles for all the brand variations you wish.</p>
<p>Lists and mixins can do much more than this. Try them and you&rsquo;ll never come back! </p>
<p>Bonus <a href="http://codepen.io/iprignano/full/KcDsd" target="_blank">codepen bonanza</a>.</p>