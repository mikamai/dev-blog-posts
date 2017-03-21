---
ID: 1023
post_title: 'asdf the &#8220;easy to write and hard to read&#8221; version manager'
author: Gianluca
post_date: 2017-02-01 12:08:49
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/02/01/asdf-the-easy-to-write-and-hard-to-read-version-manager/
published: true
---
As a Rubyist one of the first thing you end up doing is to manage many different Ruby versions on the same machine. As a matter of fact, one of the first steps in setting up a new workstation is to install some kind of version manager like <a href="https://rvm.io/" target="_blank">RVM</a> or <a href="https://github.com/rbenv/rbenv" target="_blank">rbenv</a>.

Unfortunately it doesn't end up simply like this...

<!--more-->

After a while you may need to use two different versions of PostgreSQL for two different projects. In this case, if you're on a Mac, you'll go maybe with specific formulae like petere's <a href="https://github.com/petere/homebrew-postgresql" target="_blank">homebrew-postgresql</a>.

Then a new project kicks in and you need to use a specific version of Node (yes...I know...) different then the latest stable. So you install another version manager like for example <a href="https://github.com/creationix/nvm" target="_blank">NVM</a> and then the required version/s you need.

Then, in your free time, you hear about Rust, Clojure, Elixir, [place here the name of any other language you need to give them a try]. So you follow their respective installation instructions and install them on your machine. But even for these languages, in particular for the ones under active and rapid development, you may need to have many different versions sitting one next to the other and the ability to switch between them as needed. For example by jumping from one project to another.

In some cases you may find ad hoc solutions but for others, like for example Elixir, you may end up with many different <a href="https://elixirforum.com/t/elixir-version-managers/317/6" target="_blank">choices</a> each one with its pros and cons.

Moreover, at the end of the day, you may end up feeling like you have a mess on your machine

<img class="aligncenter" src="http://sphereco.com/wp-content/uploads/2014/11/Books_Mess.jpg" alt="null" />

and maybe a complicated/long bootstrapping script. (I hope you have <a href="https://github.com/fusillicode/dotfiles/blob/master/bootstrap.sh" target="_blank">one</a>...)

<img class="aligncenter" src="http://images.dailytech.com/nimage/Internet_Cables_Tangled_Mess_Wide.jpg" alt="null" />

I'm not a fan of having a mess on my "work station" and I really like to keep it tidy and up-to-date as much as possibile. If you read the <a href="https://elixirforum.com/t/elixir-version-managers/317/6" target="_blank">over mentioned Elixir post</a> all the way down you will have noticed a few references to the object of this article.

<a href="https://github.com/asdf-vm/asdf" target="_blank">asdf</a> is indeed the version manager I ended up trying and using on a daily basis for the last two weeks. As mentioned inside the post it does have some drawbacks but I find that they're largely overcome by the number of its plugins and its clean api.

As stated inside its `README.md` it lets you specify different versions of languages and tools both globally and locally for each different project. This is done through configuration files named `.tool-versions` that can be placed inside your home for the global configuration and inside each project root in case of local configurations.

Right now I've installed the following plugins: clojure, elixir, elm, erlang, golang, haskell, nodejs, php, postgres, python, redis, ruby, rust, scala. Guess what? None of them gave me problems! 😉

Moreover I also had the chance to contribute to one plugin, i.e. <a href="https://github.com/smashedtoatoms/asdf-redis" target="_blank">redis</a>, with a particular complex <a href="https://github.com/smashedtoatoms/asdf-redis/pull/1" target="_blank">PR</a> 😜

The only thing that right now I'm missing from asdf is a mysql plugin, but considering the technology on which it is developed, i.e. plain Bash, I think that it would not be too complex to develop an maintain one.

I strongly suggest you to give asdf a try and to contribute with more and more plugins!

Cheers!