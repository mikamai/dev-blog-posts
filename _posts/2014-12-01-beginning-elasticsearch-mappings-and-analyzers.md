---
ID: 179
post_title: 'Beginning Elasticsearch &#8211; mappings and analyzers'
author: Andrea Longhi
post_date: 2014-12-01 13:04:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/12/01/beginning-elasticsearch-mappings-and-analyzers/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/104070888314/beginning-elasticsearch-mappings-and-analyzers
tumblr_mikamayhem_id:
  - "104070888314"
---
<p>Elasticsearch is one of the most recent and promising full-text search engine. It&rsquo;s built on Lucene, just like Solr, but it comes with nice RESTful JSON API and it was developed with scalability and the cloud in mind.</p>
<p>The <a href="http://www.elasticsearch.org/guide/">documentation</a> on the official website is complete and detailed, but for a newcomer it can be overwhelming, so I&rsquo;ve decided to put together this introductory article.</p>
<p>First, you need to install Elasticsearch. It&rsquo;s fairly easy on OSX: <code>brew install elasticsearch</code> will do the job. If you&rsquo;re on linux there&rsquo;s probably a <a href="http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/setup-repositories.html">package</a> ready for you.</p>
<p>Once installation is done, if you don&rsquo;t know where the executable file is located, you can find out with <code>which elasticsearch</code>. On my machine it&rsquo;s in <code>/usr/local/bin/elasticsearch</code> and that&rsquo;s exaclty what I have to type to start the server manually, but before that, let&rsquo;s install Marvel, the official dashboard/development console. Sense is the name of the console, it&rsquo;s a very nice web interface with some autocomplete features, so it&rsquo;s highly recommended for getting familiar with Elasticsearch. From the Elasticsearch home directory, run <code>bin/plugin -i elasticsearch/marvel/latest</code>. Now you&rsquo;re ready to rock.</p>
<p>In your browser go to http://localhost:9200/_plugin/marvel/sense/. That&rsquo;s the development console. Let&rsquo;s start by indexing some data. Since Elasticsearch can be schemaless, you can just throw data at it:<img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/insert.jpg" /></p>
<p>Move the cursor on each code block and click the green arrow (next to the wrench) to execute the code. The right panel shows the result.</p>
<p>If you want to follow along with the example by typing the code in your machine you can find the code <a href="https://gist.github.com/spaghetticode/807816787fe1c22c384b">here</a>.</p>
<p>While SQL databases have databases and tables, Elasticsearch has indexes and types. Just like a table belongs to a database, a type belongs to an index. In our example the index is &ldquo;examples&rdquo; and the type is &ldquo;movies&rdquo;</p>
<p>Looking at the right side of screenshot you can see that the first record was saved and there are some futher details about the operation outcome.</p>
<h2>Mappings</h2>
<p>I know you&rsquo;re looking forward to make some queries, but your best option in order to get good matches out of your data is to <span>properly </span>index contents. That&rsquo;s what mappings are about.</p>
<p>We didn&rsquo;t need to write any schema upfront, but internally Elasticsearch generated one starting from our data. Let&rsquo;s see it:</p>
<p><img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/mapping.jpg" /></p>
<p>Elasticsearch can infer the field type from the data you insert. If we had numbers and dates they would have been mapped accordingly.</p>
<h2>Analyzers</h2>
<p>What You need to know is that the type &ldquo;string&rdquo; is analyzed before being inserted into the index. An analyzer manipulates the original text and spits it out in a new form. Records will be indexed and retrieved according to their <em>tokenized</em> form.</p>
<p>If no analyzer is explicitly set then Elasticsearch will use the <code>default analyzer</code>. If you&rsquo;re curious you can see what your text will look like after analysis:</p>
<p>Using the standard analyzer, text is basically split into single words and lowercased. The first example analyzes the text out of the context of the index, using the explicitly named &ldquo;standard&rdquo; analyzer. The second one shows how text is analyzed in the context of the &ldquo;examples&rdquo; index and &ldquo;title&rdquo; field. Their results are exactly the same, confirming that the analysis process was identical.<br /> Elasticsearch comes with many builtin analyzers. Some of the most useful ones are <strong>language analyzers</strong> that can stem words according to grammar rules. Try the third example in the above screenshot and see that &ldquo;Saving&rdquo; is stemmed to &ldquo;save&rdquo;.</p>
<p><img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/analyzers.png" /></p>
<p>Why is this useful? The first example of the next image looks for a title that contains the &ldquo;saving&rdquo; word. The record will be found.</p>
<p><img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/query.png" /></p>

<p>The second example looks for a more generic &ldquo;save&rdquo;, but given the fact that the title field was analyzed with the standard analyzer, no record can be found.</p>
<p>If we want it to be found, then we need to update the mapping. Note that I&rsquo;m going to add a new field &ldquo;title.en&rdquo; instead of changing the existing generic &ldquo;title&rdquo; field.</p>
<p>Changing existing fields requires reindexing (reinserting) all the data, while adding new fields doesn&rsquo;t. You just need to update the existing records when necessary. Besides, if in the future we need to index German titles as well, we can just add a new &ldquo;title.de&rdquo; field and we will be able to index both English and German titles.</p>
<p>Let&rsquo;s update the movies type mapping with a new field, &ldquo;title.en&rdquo;, that will be used for english titles:</p>
<p><img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/mapping_update.png" /></p>
<p>Now we need to update the existing record so that we can search also within the &ldquo;title.en&rdquo; field:</p>
<p><img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/record_update.png" /></p>
<p>You&rsquo;re now ready to query the title.en field and get the expected result:</p>
<p><img alt="image" src="https://raw.githubusercontent.com/spaghetticode/screenshots/master/successful_search.png" /></p>
<p>The second example uses the &ldquo;multi_match&rdquo; keyword to allow search within multiple fields. This comes very handy, as we now can query all title fields in one shot by adding an asterisk (&ldquo;title*&rdquo;), without the need to specify which language we&rsquo;re interested in (if we added &ldquo;title.de&rdquo; it would have been searched as well, making our search cross language).</p>
<p>I hope this introduction to Elasticsearch mappings and analyzers was useful!</p>