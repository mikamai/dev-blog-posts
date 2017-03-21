---
ID: 2
post_title: Wrap and iterate with the power of yield
author: Gianluca
post_date: 2016-12-21 14:46:33
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/12/21/wrap-and-iterate-with-the-power-of-yield/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/154765165089/wrap-and-iterate-with-the-power-of-yield
tumblr_mikamayhem_id:
  - "154765165089"
---
Iterating over a collection isn’t rocket science. Obviously if you need to iterate over a really big collection you may need to adjust your strategy but it is something that in the vast majority of programming languages you get quite for free.

In Ruby for example you can just send the <code>each</code> message to your collection together with a block of code. The block will be executed for each element of the collection itself.

But what if you need “something more” from the objects you’re iterating on? If, for example, each element of your initial collection is an array and you want to abstract away how you access each of its elements? What options do you have?

<!--more-->

Recently I found myself in such need. In particular I had to write an <code>xlsx</code> importer to seed the DB of a custom Rails CMS.

There were a few gotchas about the task and one of them was that I needed to perform some selection, grouping and filtering on the rows before I could actually do the seeding.

Considering these requirements I quickly ended up with a class, i.e. <code>SpreadsheetParser</code>, aimed to handle these logics. Here it is:
<pre><code class="language-ruby">class SpreadsheetParser
  attr_reader :workbook

  def initialize workbook
    @workbook = workbook
  end

  def total_rows_to_process
    @total_rows_to_process ||= workbook.sheets[0].rows.to_a.drop(2).reject(&amp;:empty?)
  end

  def rows_grouped_by_image_name
    total_rows_to_process.each.group_by { |row| row[3] }
  end
end
</code></pre>
Another gotcha was that I needed some derived values from each column of each row to properly perform the seed. To handle this requirement I decided to add another layer of abstraction aimed to simplify the handling of the rows. Welcome <code>SpreadsheetRow</code>:
<pre><code class="language-ruby">class SpreadsheetRow
  attr_reader :raw_grouped_row

  def initialize raw_grouped_row
    @raw_grouped_row = raw_grouped_row
  end

  def season_code
    raw_grouped_row[1].to_i
  end

  alias_method :collection, :season_code

  def line
    raw_grouped_row[2].to_i
  end

  def name
    raw_grouped_row[4]
  end

  def model
    raw_grouped_row[5].to_i
  end

  def color
    raw_grouped_row[6].to_i
  end

  alias_method :tirella, :color

  def variant
    raw_grouped_row[7].to_i
  end

  def cloth_item_image_file_name
    "#{season_code}#{line}#{model}#{color}#{variant}.jpg"
  end
end
</code></pre>
With this new piece in place I felt quite satisfied but not completely. There was something bothering me:
<pre><code class="language-ruby">...
rows do |raw_row|
  spreadsheet_row = SpreadsheetRow.new raw_row
  ...
...
</code></pre>
<code>rows</code> was actually something given to me by the <code>SpreadsheetParser</code>. It was something I was getting by selecting, filtering and grouping the collection of raw rows. Why did I had to get this collection and then wrap each one of its elements explicitly inside the block?

I found no reason and so I decided to hide the instantiation of each <code>SpreadsheetRow</code> just inside the parser itself. But I also wanted to retain the the possibility to handle custom iteration blocks!
Well then, here you go:
<pre><code class="language-ruby">...
def self.each_rows grouped_rows, &amp;block
  grouped_rows.each_with_index do |grouped_row, row_order|
    yield SpreadsheetRow.new(grouped_row), row_order
  end
end

def each_rows *args, &amp;block
  self.class.each_rows *args, &amp;block
end
...
</code></pre>
What I’m doing here is to use the parser to get the rows that I need and then iterate over them by firstly build a <code>SpreadsheetRow</code> and then passing it to a given (custom) block (i.e. <code>&amp;block</code> in <code>self.each_rows</code> signature).

With <code>yield</code> you can basically wrap and enrich a custom block with the abstraction you need inside it! In this way you can have inside the block you’re writing the behavior that you actually need. This is the power of <code>yield</code> and blocks!

I know it’s not rocket science but I really like to highlight this kind of solutions (maybe “pattern”?) because I find them elegant, concise, and straightforward (at least if you know how <code>yield</code> and blocks works ;P)

Cheers!