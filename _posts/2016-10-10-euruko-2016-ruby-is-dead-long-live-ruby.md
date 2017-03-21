---
ID: 13
post_title: 'Euruko 2016 &#8211; Ruby is dead, long live Ruby!'
author: Gianluca
post_date: 2016-10-10 12:33:35
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/10/10/euruko-2016-ruby-is-dead-long-live-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/151608679809/euruko-2016-ruby-is-dead-long-live-ruby
tumblr_mikamayhem_id:
  - "151608679809"
---
It has almost been three weeks since we (me and two colleagues) came back from a wonderful trip to Sofia. Thanks to Mikamai (and I will never stop to be thankful for this! 🤗) we had the opportunity to attend the 2016 Euruko conference.

<img src="https://pbs.twimg.com/media/CtBR3oNWYAAhP9q.jpg:large" alt="Euruko2016" />

Considering the list of <a href="http://euruko2016.org/speakers" target="_blank">speakers</a>, all well known for their accomplishments in and out the Ruby community, I was overexcited by the time we had the confirmation of our trip.
Personally I was particularly thrilled to listen to the two keynotes. It doesn’t happen everyday to listen to <a href="http://euruko2016.org/speakers#matz" target="_blank">Matz</a> and <a href="http://euruko2016.org/speakers#josevalim" target="_blank">José</a> live! 😉
Anyway, enough fangirling!

<!--more-->

In this article I would like to report what I was able to note about Matz’s keynote. If you scroll down <a href="https://twitter.com/euruko" target="_blank">here</a> and <a href="https://twitter.com/search?f=tweets&amp;vertical=default&amp;q=%23euruko&amp;src=tyah" target="_blank">here</a> you may find more info (together with photos and maybe some videos) about it. But now let us begin!

The topic of the keynote is something that rubyists are talking about for a while: Ruby3.

<img src="https://pbs.twimg.com/media/CtB1p0_WcAAA_4u.jpg" alt="Ruby3" />

Considering all the hype about this topic, one of the first things that Matz wanted to highlight was the fact that Ruby3 is still pretty much <a href="https://en.wikipedia.org/wiki/Vaporware" target="_blank">“vaporware”</a>.
And this isn’t actually a bad thing! Vaporware is indeed useful as it fixes goals and objectives at various level of details.
In case of Ruby3, the main goals are the following:
<ul>
 	<li>compatibility</li>
 	<li>performance</li>
 	<li>concurrency</li>
 	<li>typing</li>
</ul>
There isn’t actually much to say about the first point. Differently from the Ruby 1.8 → 1.9 jump, Ruby3 aims to be fully compatible with Ruby2. All the proposals already submitted as well as the future ones will then need to consider this assumption.

Regarding the second point there is actually a lot more to talk about.
The main goal of Ruby3 in this scope is to be 3 times faster than its predecessor: 3x3!
This may sounds much like a commercial slogan but there is some real concrete work going on here. An explanatory example is represented by <a href="https://github.com/shyouhei" target="_blank">Urabe Shyouhei</a> <a href="https://github.com/ruby/ruby/pull/1419" target="_blank">PR #1419</a>. It is known in the wild as “Optimization/Deoptimization” and it is particularly promising considering that Matz cited it in his keynote. If you don’t believe Matz I strongly suggest to take a look at it. It is indeed pretty promising! 😉
Ruby core committers like Shyouhei aren’t however the only ones who are working hard to break the well known and broad assumption that “Ruby is slow”.
<a href="https://github.com/jruby/jruby/wiki/Truffle" target="_blank">JRuby’s Truffle</a> and recent <a href="http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-core/77461" target="_blank">OSS release of IBM OMR</a> are other pretty encouraging signs that now Ruby wants not only to be beautiful (SPOILER ALERT: personal opinion! 😁) but also fast!

Strongly related to the performance point here it is the second one: <strong>concurrency</strong>.

<img src="https://pbs.twimg.com/media/CtHxCP9WEAAR3tm.jpg" alt="JoséValim" />

Yes I know he’s not Matz! But you know what? The second day at Euruko we had the pleasure to listen to José Valim talking about Elixir! And here’s a thing that I really love about Ruby community. It is radically (mostly) openminded. It is made by people who love to study and learn new and better ways to tackle and solve problems.
Sorry for the little digression, but it has to be done! 😛

Anyway, what José stated in his keynote (see picture above) is undoubtedly true but there are two things that I want to stress out here.
First, Ruby was conceived, as a general purpose language, 23 years ago. Just to give you an idea it was born, more or less, when Intel released the first <a href="https://en.wikipedia.org/wiki/Pentium" target="_blank">Pentium</a>. At that time, concurrency was something that was needed to solve problems like handling the telephony connectivity at the switches level (( ͡° ͜ʖ ͡°) Erlang). To speak it wasn’t the day by day tool to solve common problems of a software engineer.
Second, Elixir was born in a totally different era in which concurrency is permeant in many more software development areas. To cite one: the Web.
Moreover it really stands on the shoulder of giants (once again ( ͡° ͜ʖ ͡°) Erlang, OTP).
From its birth, Ruby did try to keep up with the emerging concurrency trend but it was (and sill it is) hard considering its roots. By the way, concurrency is in general an hard problem to tackle and even more without the proper abstractions.
Right now Ruby approach towards concurrency is bounded to a tradeoff between performance and safety.

<img src="https://pbs.twimg.com/media/CtBYMN3WEAAjbCj.jpg" alt="PerformanceSafety" />

With the <a href="https://en.wikipedia.org/wiki/Global_interpreter_lock" target="_blank">GIL (Global Interpreter Lock)</a>, or better GVL (Global VM Lock), Ruby chooses safety over performance. However the GIL itself can be actually removed by deleting just a few lines inside Ruby core. Obviously this would lead us all to a danger place of indeterminism, heisenbugs, race conditions and so on.
Moreover the GIL does not magically protect us when we begin to play around with threads (i.e. <code>Thread.new</code>) and this is why we need more powerful abstractions on which we can rely if we want to do multithread programming.
Ruby 1.9’s <a href="https://ruby-doc.org/core-2.2.0/Fiber.html" target="_blank">Fibers</a> were a first step in this direction. But they are not the only abstractions that were studied, investigated and implemented in Ruby ecosystem!
Here’s some others:
<ul>
 	<li>the actor model (e.g. <a href="https://github.com/celluloid/celluloid" target="_blank">Celluloid</a>)</li>
 	<li>the <a href="https://github.com/matz/streem" target="_blank">stream model</a></li>
 	<li>the ownership/membership model</li>
</ul>
The last one in particular has recently received more attention thanks to the work of Ruby core committer and YARV developer <a href="https://twitter.com/_ko1" target="_blank">Koichi Sasada</a>. Just a few weeks before Euruko he presented at the Ruby Kaigi conference a <a href="http://rubykaigi.org/2016/presentations/ko1.html" target="_blank">proposal</a> regarding a possible membership model implementation. So Ruby3 may have a “brand new” abstraction for concurrent programming! This abstraction is by now known as Guilds (yes, exactly like RPG guilds 😁). Underlying they leverage Fibers and <a href="https://en.wikipedia.org/wiki/Green_threads" target="_blank">green threads</a> to ease developers from the burdens of multithread programming (e.g. the implementation and handling of lock and synchronization strategies). This is accomplished by making available a simple and ease to use API.
A good overview about Guilds can be found <a href="http://olivierlacan.com/posts/concurrency-in-ruby-3-with-guilds" target="_blank">here</a> but for more “hardcore” details I strongly suggest to take a look at Koichi’s <a href="http://www.atdot.net/~ko1/activities/2016_rubykaigi.pdf" target="_blank">presentation</a> 😉.

The last topic that Matz addressed in his keynote is definitely one of my favorites. It was right after listening to one of his early talks about it that I decided to make it the main topic of my MSCS thesis: <strong>typing</strong>!
Regarding this topic, one of the things that I appreciate the most of Matz’s keynote is how he successfully identified a “pendulum” in the scope of programming languages. Here’s a few pictures that should describe what I’m talking about.

<img src="https://lh3.googleusercontent.com/aZB0bUVjDA8q_y-5cNXx4NQiRyPqIJdX7YQZwV3nPBnM-d-bxYzf7yGb06g6oac5AaV46GRUexyNPPzeEIj-fGPp4NPHZZwCFDPpcm1N6gZyCWMgBWlGk5eGJaLHkQ_2KOYRHBnrnXd81sMN4ExuB60fI_tJxRA81BCfUlLwqqK4rkJb4JLUzjf3MGeoSvGxnaZul8hPIh0d43pQlnSq0u8NqxTNZv6S1BSRJMphO7OGgjvFqPCBuvSO3U_m3lh13UhC5n2buWB8LfRMX4fLPaaHBFtnr3IRjCaJNsZCkc-Na548IsTCVqkqf5V5d2lq0URYXPf4sSvkeCrm4EzkMvutAoygmyxNqEEck9KEHtg2EIUbmd7kzyBu6YlOOBLpJtp2xOTT7MexCuy_wXB9JWqGODw8tTSe1Cu3_P9MuW7NUrXV8naxj7m-6_7LmxJHKhVjS4Kco5XTr6XjsMSWmQE6_V67NicrLjVOXCvgKc8knKu8br8aF_4tI_VVsUIyhiR84eDiUQEU6XHT3Q_-dnw1ItKm169p1CQEMFmY2LNeqNoyTwE9puElEUezD_rcc2reVvKpmFSShQ7lX-VV5LIXhslAkhujSTpyhwswpdohszFA=w1280-h848-no" alt="UntypedToTyped" />

<img src="https://lh3.googleusercontent.com/dnRdSelhqBMhB85mu8zFSffM7O-_RWvmaae0CO4BsK4wVoOgmtgCWUd2PgPdEMeE9G9U6lVxoLgDwFjyxLjOlvhanOIzC3xWD7FVEGWUy2vUyFuC-0ElrLSYYlKHrjgA2d5wgzm7FUf_I0M4NZLuVV2H0VpjGME1Pfr-ZusorVsihef1kQ4OpVvoFEim7gQ5oLalvlQEgTZLrQ1TNjdI-szIQJNHjyJdkvFd8LHbLW8eYdJijCcpIWwjuLHe57sTyIWUUjwhEUBINS2KSN3qfePfwDm5G5d4cv7Ms_sl-oyCxjj5GCY_zfUQ8iLwOggEJ3OKO_B6WeH3xlyKnq9c4tymsjNHw0xocfyQgQBk1-W3Axf42jgWvdSw5UJFEEkiQmCyyFU2YPPp9QRawEBglSZPOIITxAk9OPi1MkeJd2sGOQ9ok_qsTSaXqYsukd4misAqbOa1tyCINZmZLOwqeYXNvCCsnl5WVu_YsbTbxlVPfMwhYc6vvIpb7KBjgXTf652GboWtT4O6cexxnS0zO5-YxQt7KAi_FmMCcKFEF7rPtkla6BP8TybgXowGxdUCF6u91XG1ej-Q5XHb7thtHWSJB_3xor_mmmc1dw4TQd2o0Kwh=w1280-h848-no" alt="TypedToUntyped" />

<img src="https://pbs.twimg.com/media/CtBZYZkWIAA9NqG.jpg" alt="TypedTo..." />

Right now static typing seems to be the new hype but we all know that in Ruby we don’t have that.
In Ruby we don’t even have types! Classes aren’t types in Ruby.
In Ruby we have “ducks” 😁 and that’s because we (rubyists and Matz before all) don’t care about types.
We care about behavior. Reasoning about the expected behavior when writing software reduces somehow our brain consumption.

<img src="https://pbs.twimg.com/media/CtBZvurWgAAEEmh.jpg" alt="ThigsShouldJustWork!" />

The aforementioned ducks are exactly our expected behaviors!
With them (duck typing) we gain an intrinsically openness towards extensions.
Unfortunately there is no silver bullet and the dynamism granted by duck typing has also some drawbacks:
<ul>
 	<li>unexpected errors at run time</li>
 	<li>error messages sometimes difficult to understand</li>
 	<li>difficulty to reason about large codebases without tests and documentation</li>
</ul>
A way to solve these problems could be to add some kind of abstractions like Go interfaces (and so shift towards structural typing) or to add type annotations. Both of these solutions go however against one of the key Ruby concepts: DRY.
Ruby loves to be DRY and relying on “type abstractions” or type annotations would mean to repeat the expected behavior of a piece of software not only explicitly but also more verbosely.
The possible solution suggested by Matz is to rely on something that helps to reconstruct the missing information that developers may need. That is <strong>type inference</strong>.
With type inference and structural typing, Ruby3 would get “the best of both worlds” (static/dynamic typing).
With something like a “Duck Inference” (trademark ™ and copyright © directly to Matz! 😉) we would get many benefits including:
<ul>
 	<li>improved types (and general) knowledge of software when needed</li>
 	<li>no additional brain cost derived from type annotations and abstractions as inferred types does not have names</li>
 	<li>possibility to detect more errors at compile time</li>
 	<li>improved tools (IDEs) capabilities like refactoring and code completion</li>
</ul>
Duck inference isn’t perfect though and as Matz says:

<img src="https://pbs.twimg.com/media/CtBeyZ8WgAA9cpo.jpg" alt="ALotToDo" />

The last point that I want to stress out before concluding this already long article refers to the first part of its title: “Ruby is dead”.
It seemed to me that this was the thought of many people attending Matz’s keynote when he said that he doesn’t know when Ruby3 will be ready.
Personally I believe that there is indeed a lot of work to do and this is why it is difficult to state precise deadlines.
But please fellow rubyists don’t turn back your shoulders and be kind of “disrespectful” to the language that have made you what you are. The language that makes you happy.
In any case, don’t lose hope!

Matz is not alone.

Ruby’s community isn’t composed only by “language users” (and lovers!).
There are also many “hardcore” engineers working hard on the language. <a href="https://twitter.com/_ko1" target="_blank">Koichi Sasada</a>, <a href="https://twitter.com/shyouhei" target="_blank">Urabe Shyouhei</a>, <a href="https://twitter.com/a_matsuda" target="_blank">Akira Matsuda</a>, <a href="https://twitter.com/hsbt" target="_blank">Hiroshi Shibata</a>, <a href="https://twitter.com/nalsh" target="_blank">Naruse Yui</a>, <a href="https://twitter.com/hone02" target="_blank">Terence Lee</a> are just a few names but there are many others!

<img src="https://pbs.twimg.com/media/CtVCwWHUAAACzzR.jpg:large" alt="RubyKaigi" />

Cheers and be happy! 😃

P.S: for the ones that want to hear <strong>live!!!</strong> about what I wrote here I will give a little presentation this Thursday evening (i.e. 13th October) at the first <a href="https://www.meetup.com/it-IT/Ruby-Nights-Milano/events/234527231/?eventId=234527231" target="_blank">Ruby Night</a> in Milan.
Spoiler alert: I’m not Matz 😝
By the way let me say thank you to the organizer, my friend <a href="https://twitter.com/davighz" target="_blank">Davide</a> and once more to Mikamai for hosting the event in its beautiful <a href="https://www.google.it/maps/uv?hl=en&amp;pb=!1s0x4786c691fa65942d:0x25e0caaa3e7e8d5a!2m10!2m2!1i80!2i80!3m1!2i20!16m4!1b1!2m2!1m1!1e1!3m1!7e115!4s/maps/place/mikamai/@45.4903903,9.2158802,3a,75y,340.72h,90t/data%3D*213m4*211e1*213m2*211sGC_miZeIFDoAAAGuxVBJvw*212e0*214m2*213m1*211s0x0:0x25e0caaa3e7e8d5a!5smikamai+-+Google+Search&amp;imagekey=!1e2!2sGC_miZeIFDoAAAGuxVBJvw&amp;sa=X&amp;ved=0ahUKEwi2-LmbzsrPAhVHShQKHctTBDUQoB8IgwEwDQ" target="_blank">office</a>!