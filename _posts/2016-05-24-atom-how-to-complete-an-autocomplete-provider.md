---
ID: 28
post_title: 'Atom: How to complete an Autocomplete Provider'
author: Diomede Tripicchio
post_date: 2016-05-24 08:58:39
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/05/24/atom-how-to-complete-an-autocomplete-provider/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/144849797969/atom-how-to-complete-an-autocomplete-provider
tumblr_mikamayhem_id:
  - "144849797969"
---
<p><i>This is the second part of <a href="http://dev.mikamai.com/post/143061935049/atom-how-to-create-an-autocomplete-provider">Atom: How to create an Autocomplete Provider</a></i></p>

<p>In the previous post we tried to create an Autocomplete Provider for FactoryGirl but we haven’t finished yet, so, let’s go on.</p>
<!--more-->

<p>The first problem we want to solve is the <code>filterSuggestions: true</code> that completely removes the suggestions we provide from the autocomplete drop down. In the end it turned out easier than expected, we can just remove the colon <code>:</code> from the <code>text</code> of our suggestions. I have not delve into this but it seems that the colon <code>:</code> is a ignored character by the Autocomplete+ engine (for now we are fine with the text without the <code>:</code> prefix).</p>

<pre><code class="language-diff">  buildCompletion: (factory) -&gt;
-  text: ":#{factory}"
+  text: "#{factory}"
   rightLabel: 'FactoryGirl'
</code></pre>

<p>Now we can safely add <code>filterSuggestions</code> to our provider settings and our suggestions will be filtered automatically by the fuzzy finder logic:</p>

<pre><code class="language-diff">module.exports =
   selector: '.source.ruby'
   disableForSelector: 'source.ruby .comment'
+  filterSuggestions: true
</code></pre>

<p>The second problem we have to solve is when we open a project that doesn&rsquo;t have the <code>/spec/factories</code> dir; an ugly red message appears on the right top and informs that the directory is not found.
I searched on google and came across some Stack Overflow questions, in the end the best solution I’ve found is to add a <code>try/catch</code> block. I don’t completely like it but for now it works.</p>

<pre><code class="language-diff">scanFactories: () -&gt;
+    try
       results = []
       factoryPattern = /factory :(w+)/g
       for factory_file in fs.readdirSync("#{@rootDirectory()}#{@factoryDirectory()}")
         data = fs.readFileSync "#{@rootDirectory()}#{@factoryDirectory()}/#{factory_file}", 'utf8'
         while (matches = factoryPattern.exec(data)) != null
            results.push matches[1]
        results
+    catch e
</code></pre>

<p>Now we have an autocomplete provider that doesn&rsquo;t annoy us when we open a project without the <code>/spec/factories</code>. One last change we can do is to limit the suggestions only in the RSpecs files, to do this we simply add <code>.rspec</code> to our <code>selector:</code> property.</p>

<pre><code class="language-diff">module.exports =
+  selector: '.source.ruby.rspec'
   disableForSelector: 'source.ruby .comment'
   filterSuggestions: true
</code></pre>
<h3>Loading suggestions</h3>
<p>There is still a problem, our code loads the suggestions only at the first loading of Atom and if a new factory is added it will not be displayed until we close and reopen the project, not much practical.
To solve this we have to reload our suggestions when a factory definition file is saved, in order to do this we can bind to the save event and reload our suggestions if the file is a factory file:</p>

<pre><code class="language-coffee">bindEvents: -&gt;
  atom.workspace.observeTextEditors (editor) =&gt;
    editor.onDidSave (event) =&gt;
      if event.path.includes @factoryDirectory()
        @loadCompletions()
</code></pre>

<p>With <code>atom.workspace.observeTextEditors</code> we are watching on all editor windows, also if a new one is opened after the start of Atom. For now, we can keep it simple and check if the file saved is under the <code>/spec/factories</code> dir, then, reload the suggestions.</p>

<p>Our autocomplete provider is finally completed! And it is ready to be released to the world! You can found it <a href="https://atom.io/packages/autocomplete-factory-girl">here</a></p>

<p>That&rsquo;s all for now, see you next time!</p>