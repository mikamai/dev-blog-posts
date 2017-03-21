---
ID: 241
post_title: 'Download a CSV in PHP&#8230;use ob_clean()!'
author: Gianluca
post_date: 2014-06-09 14:45:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/09/download-a-csv-in-phpuse-obclean/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/88276841584/download-a-csv-in-phpuse-obclean
tumblr_mikamayhem_id:
  - "88276841584"
---
There are many blog posts, guides, code snippets and stackoverflow answers that describe how to create and download a CSV file in PHP.

Anyway none of them helped me the last time I was asked to do that in the context of the configuration panel of a PrestaShop module.

<!--more-->

The problem was that I kept on getting the page constructed by the framework instead of the correct content of the file.

This was related to the fact that before reaching the point where I actually download the CSV, the PHP output (i.e. <a href="http://www.php.net/manual/it/wrappers.php.php" target="_blank">php://output</a>) was polluted by the page created by the framework.

Here how I solved it:
<pre>private function exportCSV($report_data)
  {
    ob_clean();
    header("Content-type: text/csv");
    header("Cache-Control: no-store, no-cache");
    header("Content-Disposition: attachment; filename='{$this-&gt;csv_filename}'");

    $fp = fopen('php://output', 'w');

    foreach ($report_data as $key =&gt; $value) {
      fputcsv($fp, $value, ',', '"');
    }

    fclose($fp);
    die();
  }
</pre>
The function is called after a normal form submission but the “magic” resides right in the <a href="http://www.php.net/manual/en/function.ob-clean.php" target="_blank">ob_clean()</a> call. This indeed clears the output buffer giving the possibility to output and obtain what needed.

P.S: please forgive the die() at the end :P