---
ID: 1346
post_title: >
  Hold on to your seats for the brain
  simulator
author: Cecilia Nardini
post_date: 2017-04-21 09:39:38
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/04/21/hold-on-to-your-seats-for-the-brain-simulator/
published: true
---
Some time ago in this blog <a href="https://dev.mikamai.com/2017/01/27/donkey-kong-and-the-secrets-of-the-brain/">we discussed Neuroscience</a> and we hinted at the question of brain simulation. But what does it actually mean to simulate the brain?

<!--more-->

The little we know about the working of the brain (any animal brain, not just the human) is that it is made up of a specialized type of cells called the neurons. Roughly speaking, each neuron has a long tail called an axon which it uses to transmit signals to other cells, and a head which receives signals from several other neurons' tails attached to it. However, it is not the case that a neuron's tail sends out signals to only one other neuron. In reality connections between neurons (also called synapses) form a huge web whereby each neuron targets (and is targeted by) thousands of other neurons.

<img class="size-medium wp-image-1358 alignleft" src="https://dev.mikamai.com/wp-content/uploads/2017/04/brain_neuron_black-300x115.png" alt="" width="300" height="115" />This also highlights that the most significant difference between the brain and a computer is at the 'hardware' level: the computational unit of the brain, the neuron, has several orders of magnitude more connections than the computational unit of a computer, the transistor. This bars the usage of computers to construct a simulation of the brain in a direct manner, i.e. building a computer where each computational unit represents a neuron and it is connected to other computational units according to known data about the wiring of real neurons.

What most brain simulation programs do instead is building a software model of a neuron, capturing aspects such as its reactions and interactions with other neurons, and then building a network of simulated neurons.
One such projects is <a href="https://www.theguardian.com/science/2015/oct/08/complex-living-brain-simulation-replicates-sensory-rat-behaviour">Blue Brain</a>, an EU-funded initiative that was launched in 2006. The model in the Blue Brain simulation reaches extreme levels of sophistication, modeling aspects such as the 3D structure of neurons, their electrical properties, their connectivity, all to the purpose of simulating in a realistic manner <a href="http://www.nature.com/news/482456a-i2-0-jpg-7.2932?article=1.10066">the network of neurons and their connections</a>.
The computer model <a href="https://bbp.epfl.ch/nmc-portal/downloads">can be accessed online</a> in most of its components.

This level of detail implies that the computational power and the infrastructure needed to support this project are gargantuan*. However, the output of the project is far less impressive than the premises would warrant: manipulating the input in certain ways, such as by simulating a touch stimulus, the pattern of activation of the virtual neurons is similar to that observed in real experiments - not exactly mind-blowing.

<a href="http://www.telegraph.co.uk/technology/10567942/Supercomputer-models-one-second-of-human-brain-activity.html" target="_blank"><img class="size-medium wp-image-1357 alignright" src="https://dev.mikamai.com/wp-content/uploads/2017/04/maxresdefault-300x169.jpeg" alt="" width="300" height="169" /></a>It may be tempting to wonder how this endeavor is related to the quest for artificial intelligence, and the answer is: barely. <a href="http://www.telegraph.co.uk/technology/10567942/Supercomputer-models-one-second-of-human-brain-activity.html" target="_blank">It takes 40 minutes of the most powerful supercomputer in the world to reproduce one second of activity of a tiny chunk of human brain</a> using the same simulation principles as the Blue Brain project, albeit at a coarser level of detail. We are talking of computers the size of a large room, capable of running millions of calculations per microsecond, and they are used to simulate the activity of a grain-sized fraction of a brain - just activity, unrelated to any observable 'behavior'.

Now the Blue Brain project <a href="https://www.theguardian.com/science/2014/jul/07/human-brain-project-researchers-threaten-boycott">underwent strong criticism</a>. It is not so much the huge amount of resources needed to carry on this project that is in question but rather the idea behind this whole enterprise: the idea that reproducing the micro-mechanism of operation could shed light on how complex mechanisms like memory, cognition and conscience arise from such micro-components.

The relevant difference here is between <a href="http://stackoverflow.com/questions/1584617/simulator-or-emulator-what-is-the-difference">simulation and emulation</a>. What is done by projects like Blue Brain is a simulation, i.e. an attempt to reproduce the actual mechanism by which the brain works. An <em>emulation</em>, instead, means reproducing the behavior of the brain through a mechanism which may in principle be completely different. For example, the way we do a division with pen and paper is different from how a calculator does it, but the result is (should be) the same. Or, we are able to create artificial (software) systems <a href="https://en.wikipedia.org/wiki/Machine_learning">which exhibit brain-like behaviors</a> such as plasticity and learning, but there is no neuron, virtual or real, in such systems: the behavior is obtained through a different mechanism.

Brain simulations may help us in the coming years to improve our understanding of the micro-structures of the brain and the way they work. However, to the purpose of creating an artificial intelligence, <em>emulating</em> the brain and its behaviors, rather than simulating it, is likely to be more useful. This approach is likely to also forward our understanding of high-level mechanisms of thought and cognition in the process.

**You know, I've always liked that word... 'gargantuan'... so rarely have an opportunity to use it in a sentence.