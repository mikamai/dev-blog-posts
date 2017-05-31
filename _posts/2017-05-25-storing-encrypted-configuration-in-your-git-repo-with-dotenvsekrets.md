---
ID: 1452
post_title: >
  Storing encrypted configuration in your
  git repo with DotenvSekrets
author: Andrea Longhi
post_date: 2017-05-25 13:06:30
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/05/25/storing-encrypted-configuration-in-your-git-repo-with-dotenvsekrets/
published: true
---
<p>Developers who are familiar with the <a href="https://12factor.net/config">"twelve-factor" app</a> know that configuration should be stored in the environment (loaded via env vars), not in the code. If you're writing a Rails application you may want to use the <a href="https://github.com/mikamai/dotenv"><code>dotenv </code></a> gem, which allows you to store env variable in a handy series of files such as <code>.env, .env.development, .env.test</code> and so on. You can read the gem readme for all the details.</p>

<p>Since configuration varies with time and you may be working with many other developers, it is quite handy to store these env files in your git repository, but there's a big <i>caveat</i>: you should not store passwords, secrets and tokens inside a git repo for security reasons, unless they're encrypted.</p>
<!--more-->

<p>Dotenv has no builtin encryption facility, so here at Mikamai we followed this rule for a while, avoiding to commit any sensible config data inside the project repository.</p>

<p>Unfortunately this solution didn't work perfectly: we had <strong>strong security</strong>, but we faced some <strong>practical problems</strong>.

<p>Every time a new developer joined a project they had to ask for the configuration file to another dev, who in turn could have a file which was not updated anymore with old keys/values or missing data. Then developers were asked to document the keys they added, but sometimes they forgot. No need to blame them, after all we're just humans.</p>

<p>So here it comes <a href="https://github.com/mikamai/dotenv_sekrets">DotenvSekrets</a> to the rescue: a new gem that joins the strength of Dotenv and <a href="https://github.com/ahoward/sekrets">Sekrets</a> allowing you to store sensible data via <code>sekrets</code> encrypted <code>dotenv</code> files inside your git repo.</p>

<p>The gem is quite simple, it just patches some dotenv classes in order to allow the loading of the encrypted versions of dotenv files, such as <code>.env.enc, .env.development.enc</code> and so on. The encrypted files have lower priority, so you can always override them locally with the corresponding clear dotenv file (ie. for <code>.env.development.enc</code> you can use <code>.env.development</code>).</p>

<p>So, how does it work? It's quite simple: once you're in your rails repo and you added the <code>dotenv_secrets</code> gem to your Gemfile, you can do in the shell:</p>

<pre><code>
# create the sekrets key file:
touch .sekrets.key 
echo "42" > .sekrets.key

# and gitignore it:
echo ".sekrets.key" >> .gitignore

# create and edit the dotenv file:
touch .env.enc
bundle exec sekrets edit .env.enc
# the file opens in your editor

# verify that when you close the file it 
# gets automatically encrypted:
cat .env.enc

# add the config file to the repo and commit:
git add .env.enc
git add .gitignore
git commit -m 'Add encrypted env configuration'
</pre></code>

<p>And that's it! Now, you need to distribute securely only the key, not the whole configuration file, which remains always available and up to date inside the repository.<p> I hope this gem can be useful to other teams, and of course if you have comments/issues you can reach me on the <a href="https://github.com/mikamai/dotenv_sekrets/issues">gem page on github</a>.