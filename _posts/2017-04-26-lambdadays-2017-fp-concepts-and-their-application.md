---
ID: 1206
post_title: >
  LambdaDays 2017, FP concepts and their
  application
author: Gianluca
post_date: 2017-04-26 10:00:14
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/04/26/lambdadays-2017-fp-concepts-and-their-application/
published: true
---
In my last post I tried to summarize the main concepts expressed by Prof. John Hughes and Prof. Mary Sheeran in their wonderful keynote at the LambdaDays 2017.

If you didn't read it, well, <a href="https://dev.mikamai.com/2017/03/22/1157/">here</a> it is. Go on, I'll be waiting for you here 😁

...done?

Jokes apart, at the end of my summary I left a little bit of suspense regarding the topics of my next (this) post but I also gave a few hints about them.

So, without further ado, here there are the two "mysterious concepts":

<!--more-->
<ul>
 	<li>Lazyness</li>
 	<li>Consumer-producer</li>
</ul>
The reason why these two ideas strongly resonated within me when I heard Prof. Hughes and Prof. Sheeran talk about them, is due to my brief experience with an Elixir library called <a href="https://github.com/elixir-lang/flow" target="_blank" rel="noopener noreferrer">Flow</a>.

They resonate even more if you consider that right after Prof. Hughes and Sheeran keynote I had the pleasure to attend José Valim talk about Flow.

The main goal of this library is to ease the development of pipelines of transformations that can be run in parallel. In particular it doesn't require you to define ad hoc modules that wrap your transformations, and it abstracts away all the orchestration of the processes required to run them.

With Flow you can basically process bounded or unbounded collections of data and better leverage the resources that you have at disposal. With resources I mean both CPU and I/O like disks and networks.

Laziness is strongly related to the unbounded data case. In general, data is defined as unbounded when it assumes the connotations of a stream. This means that it does not have a predefined known size and that it constantly flows into the system.

Given these features, it cannot be processed eagerly considering (read loading) the entire collection. The collection may indeed be infinite!

This is where a lazy approach, i.e. load and process only one single piece of data at a time, should be applied. In this context Elixir provides you with the <a href="https://hexdocs.pm/elixir/Stream.html" target="_blank" rel="noopener noreferrer">Stream</a> module.
This module mirrors many functions defined inside the <a href="https://hexdocs.pm/elixir/Enum.html" target="_blank" rel="noopener noreferrer">Enum</a> module and can be used to manage and create streams starting from given enumerables.

By definition, I'm just quoting the docs here, streams are
<blockquote><em>"any enumerable that generates items one by one during enumeration"</em></blockquote>
Laziness does not only enable the processing of unbounded data but it also grants the possibility to build data structures that actually represent computations.

In case of Flow it enables the creation of possibly complex processes topologies by writing quite trivial transformations pipelines.
An important aspect about this representational approach is that there is no actual computation going on when the pipelines get defined.
All the functions implemented in the Flow module, like <a href="https://hexdocs.pm/flow/Flow.html#map/2" target="_blank" rel="noopener noreferrer">map</a>, <a href="https://hexdocs.pm/flow/Flow.html#reduce/3" target="_blank" rel="noopener noreferrer">reduce</a>, <a href="https://hexdocs.pm/flow/Flow.html#partition/2" target="_blank" rel="noopener noreferrer">partition</a> and so on, do nothing more than return an Elixir data structure that defines the topology of the particular computational step.

When running the following code in <a href="https://hexdocs.pm/iex/IEx.html" target="_blank" rel="noopener noreferrer">iex</a> for example
```
flow = [1,2,3] |&gt; Flow.from_enumerable() |&gt; Flow.map(&amp; &amp;1 + 1)
```
it does nothing more than returning the following data structure
```
%Experimental.Flow{operations: [{:mapper, :map,
[#Function&lt;6.118419387/1 in :erl_eval.expr/5&gt;]}], options: [stages: 4],
producers: {:enumerables, [[1, 2, 3]]},
window: %Experimental.Flow.Window.Global{periodically: [], trigger: nil}}
```
As you can see there is no trace of the result of the transformation implemented inside the function passed in to the <code>reduce</code> step.
Just to be more clear, if Flow would have been eager instead of lazy, the <code>flow</code> variable should have been bound to the result of the computation, i.e. <code>[2, 3, 4]</code>. Moreover we should have seen the result printed out right inside iex.

What you have to do to actually run the transformation is to explicitly trigger it by calling <a href="https://hexdocs.pm/flow/Flow.html#run/1" target="_blank" rel="noopener noreferrer">Flow.run</a> or by requesting the result with calls to <a href="https://hexdocs.pm/elixir/Enum.html#to_list/1" target="_blank" rel="noopener noreferrer">Enum.to_list</a>. or other Enum functions like for example <a href="https://hexdocs.pm/elixir/Enum.html#sort/1" target="_blank" rel="noopener noreferrer">sort</a>.

Beside laziness, the other key concept that struck me while listening to Prof. Hughes and Prof. Sheeran keynote was the consumer-producer principle.

The reason why it struck me was, once again its relation with Flow.
It is indeed one of the basic concepts on which Flow roots its bases. To be more precise it is the main design principle behind <a href="https://dev.mikamai.com/2017/03/17/caching-images-in-uicollectionview/" target="_blank" rel="noopener noreferrer">GenStage</a>, i.e. the Elixir behavior built by José Valim on which Flow builds up.

What GenStage does is to enable the definition and orchestration of processes in a resilient way by implementing a back-pressure mechanism.

To be more concrete it can be used to implement Elixir modules that can be used in turn to spawn processes that interact with each other by following a precise message contract.

In the context of GenStage each process represents a specific pipeline step and can be accounted for a specific data transformation. In particular they are called stages and can be categorized in three different families: consumer, producer and producer_consumer.
This distinction is based on how they should interact with the other processes defined through modules implementing the GenStage behaviour.

The main goal of message contract underlying GenStage is to push the possible failures to the boundaries of the system you're building.
This is done by implementing, as I cited before, a back-pressure mechanism in which the active part of the processing pipeline are not the producers, i.e. the entities that push the data inside your system, but instead the consumers.
These stages are the ones that subscribe to the producers and ask them the amount of data that they need and, more importantly, the amount of data that they can handle.

I like to think of this like an application of the inversion of control (IoC) principle. Observations and comments about this point of view are very welcome 😀

Anyway, by having the consumers of the data asking the producers for what they need, we get the chance to build systems in which the processing core is kind of protected from eventual spikes of data input. The spikes are indeed handled outside the processing core (i.e. outside the consumers) right at the level of the producers, at the boundary of your system.

Thanks to the relation between GenStage and Flow the back-pressure mechanism is also transparently available in Flow.
Flow is indeed just an abstraction built on top of GenStage and so it get all the GenStage goodies quite for free.

Laziness and consumer-producer design principle are only two of many concepts that arise from the broader topic of Functional Programming (FP) and that find concrete application inside many libraries of many different programming languages.

I hope that this brief summary helped to shed a bit of light on the concept of laziness, on the consumer-producer design principle and on their application in Flow and GenStage libraries.

I also hope to have the chance to write about these topics in my future articles!

Cheers and to the next time! 😄