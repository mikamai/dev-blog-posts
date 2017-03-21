---
ID: 181
post_title: >
  error_log all the things! (in
  Prestashop)
author: Gianluca
post_date: 2014-11-26 15:04:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/11/26/errorlog-all-the-things-in-prestashop/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/103638981984/errorlog-all-the-things-in-prestashop
tumblr_mikamayhem_id:
  - "103638981984"
---
<p><a href="http://en.wikipedia.org/wiki/W._Edwards_Deming" target="_blank">William Edwards Deming</a> once said: &quot;You can&rsquo;t manage what you can&rsquo;t measure.&ldquo;</p>
<p>I find this statement pretty similar to: &quot;If I can&rsquo;t var_dump I can&rsquo;t fix&rdquo;.</p>
<p>Actually this statement is just a bit stronger than Deming&rsquo;s one but I find it true in most of the cases.</p>
<p>It doesn&rsquo;t matter if code is properly written and TDD and/or BDD are actually used. When a program begins to work unexpectedly, one of the things that is usually done to shed some lights on the problem is to put some &ldquo;log&rdquo; directives here and there and look at the results.</p>
<p><span>This kind of debugging strategy, while being rather primitive, can be very useful in many contexts.</span></p>
<p>For example if you&rsquo;re developing something with PrestaShop (PS) (so you&rsquo;re writing PHP&hellip;) you may feel the need to log something inside the apache error log.</p>
<p>This can be easily accomplished by using the PHP function <a href="http://php.net/manual/en/function.error-log.php" target="_blank">error_log</a>.</p>
<p>But this function  accepts only strings, so in order to log a complex structure like an hash you should first convert it to a string using something like <a href="http://php.net/manual/en/function.var-dump.php" target="_blank">var_dump</a> or <a href="http://php.net/manual/en/function.print-r.php" target="_blank">print_r</a>.</p>
<p>Sincerely I was expecting some kind of support from PS core functionality but unfortunately this wasn&rsquo;t the case.</p>
<p>As a result I end up writing a bunch of lines of code to log whatever I need right inside the apache error log.</p>
<p>Here they are:</p>
<pre><code>
class Tools extends ToolsCore
{
  public static function console_log()
  {
    $args = func_get_args();
    $log = '';
    foreach ($args as $arg) {
      $log .= print_r($arg, true);
    }
    error_log($log);
  }
}
</code>
</pre>
<p>I simply added a public static function to the Tools class (by properly overriding it) that takes any number of arguments and builds a string by using the print_r function.</p>
<p>The string is eventually logged on the apache error log.</p>
<p>I hope this can be useful to the ones willing to go with the old style debugging :P</p>
<p>Cheers!</p>