---
ID: 38
post_title: 'Atom: How to create an Autocomplete Provider'
author: Diomede Tripicchio
post_date: 2016-04-19 14:49:33
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/04/19/atom-how-to-create-an-autocomplete-provider/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/143061935049/atom-how-to-create-an-autocomplete-provider
tumblr_mikamayhem_id:
  - "143061935049"
---
<p><i>This post will continue on <a href="http://dev.mikamai.com/post/144849797969/atom-how-to-complete-an-autocomplete-provider"></a><a href="http://dev.mikamai.com/post/144849797969/atom-how-to-complete-an-autocomplete-provider">Atom: How to complete an Autocomplete Provider</a></i></p><p>Hi all, today I’d like to see with you how to create a basic provider for Atom using the <a href="https://github.com/atom/autocomplete-plus/wiki/Provider-API">Autocomplete Provider API</a></p>

<p>In this specific case I‘d like to create a Provider for autocompletion of <a href="https://github.com/thoughtbot/factory_girl">FactoryGirl</a>.</p>
<!--more-->

<p>First of all we need to generate a new package and we can do this  by simply pressing <code>cmd+shit+P</code> and typing <code>generate package</code> then choosing <code>Package Generator: Generate Package</code>.</p>

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/03006f7d352958ccd7285e9db14bb60e/tumblr_inline_o5vv8tLzzN1qzktze_540.png" alt="image" /></figure><p>At this point Atom asks where to save the package we want to create,  by default it saves the new packages in <code>~/github/package-name</code>.  To keep it simple we use the default base location so we also have the package loaded automatically.</p>

<p>The package generator creates a file structure that is more than what we need at the moment. For now we can keep only these files and directory:</p>

<pre><code>lib/
.gitignore
CHANGELOG.md
LICENSE.md
package.json
README.md
</code></pre>

<p>Yes, without the <b>spec</b> dir, at least for now :P</p><p>The main files we need in order to achieve our goal are <code>lib/main.coffee</code> and <code>lib/provider.coffee. </code>Let’s see how they have to be.</p>

<h2>main.coffee</h2>

<p>It’s the simplest one, it creates the package and defines the function <code>getProvider</code></p>

<pre><code>provider = require ‘./provider’

module.exports =
  activate: -&gt; provider.load()

  getProvider: -&gt; provider
</code></pre>

<h2>provider.coffee</h2>

<p>Now the “big” one, this file defines the provider used by the Autocomplete. In this class we need to define the <code>getSuggestions</code> function, clearly to return our suggestions :D</p>

```coffee
fs = require ‘fs’

module.exports =
  selector: ‘.source.ruby’
  disableForSelector: ‘source.ruby .comment’

  load: () -&gt;
    @completions = @scanFactories()

  getSuggestions: (request) -&gt;
    {prefix} = request
    completions = []
    for factory in @completions when not prefix or firstCharsEqual(factory, prefix)
      completions.push(@buildCompletion(factory))
    completions

  scanFactories: () -&gt;
    results = []
    factoryPattern = /factory :(w+)/g
    for factory_file in fs.readdirSync(“#{@rootDirectory()}/spec/factories”)
      data = fs.readFileSync “#{@rootDirectory()}/spec/factories/#{factory_file}”, ‘utf8’
      while (matches = factoryPattern.exec(data)) != null
        results.push matches[1]
    results

  rootDirectory: -&gt;
    atom.project.rootDirectories[0].path;

  buildCompletion: (factory) -&gt;
    text: “:#{factory}”
    rightLabel: ‘FactoryGirl’

firstCharsEqual = (str1, str2) -&gt;
  str1[0].toLowerCase() is str2[0].toLowerCase()
```

<p>By the way, the core function for our provider is <code>scanFactories</code>. This function watches in every file under <code>{project_dir}/spec/factories/*</code> and searches for the regular expression defined in <code>factoryPattern</code>. For example in a file like the following one <code>scanFactories</code> found <code>[“user”, “confirmed_user”]</code></p>

<pre><code>FactoryGirl.define do
  factory :user do
    username ‘foo’
        confirmed false

        factory :confirmed_user do
            confirmed true
        end
    end
end
</code></pre>

<p>With this provider we can get a result like this:</p>

<figure class="tmblr-full"><img src="http://68.media.tumblr.com/ebd87c884a3c5e97e2354aef7e32d2a3/tumblr_inline_o5vv99b6xp1qzktze_540.png" alt="image" /></figure><p>To improve a little bit the performance of this provider we filter the suggestions with <code>firstCharsEqual</code>, in this way we display only the factories that start with the same letter of the <code>prefix</code>, the ‘a’ of admin in the image above.</p>

<h2>package.json</h2>

<p>Last but not least, we need the <b>package.json </b>file, with this we can define the license, repo, author etc..</p>

<pre><code>{
  “name”: “package_name”,
  “main”: “./lib/main”,
  “version”: “0.0.2”,
  “description”: “An autocomplete+ provider for FactoryGirl factories”,
  “keywords”: [
  ],
  “repository”: “https://github.com/package_repo”,
  “license”: “MIT”,
  “engines”: {
    “atom”: “&gt;=1.0.0 &lt;2.0.0”
  },
  “providedServices”: {
    “autocomplete.provider”: {
      “versions”: {
        “2.0.0”: “getProvider”
      }
    }
  },
  “dependencies”: {
  }
}
</code></pre>

<p>But the most important part for us is:</p>

<pre><code>    “providedServices”: {
    “autocomplete.provider”: {
      “versions”: {
        “2.0.0”: “getProvider”
      }
    }
  },
</code></pre>

<p>Thanks to these lines we are telling Autocomplete how to get our new provider.</p>

<p>For now it’s all, now I’m fighting with a property available on the providers <code>filterSuggestions</code> that seem not to work as expected.</p>

<p>This is still a work in progress and not yet released as Atom package but it will be ASAP.</p>

<p>Stay tuned,
Bye!</p><p>Edit: Read the second part <a href="http://dev.mikamai.com/post/144849797969/atom-how-to-complete-an-autocomplete-provider">here</a></p>