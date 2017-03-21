---
ID: 974
post_title: Donkey Kong and the secrets of the brain
author: Cecilia Nardini
post_date: 2017-01-27 11:39:36
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/01/27/donkey-kong-and-the-secrets-of-the-brain/
published: true
---
I don't know about you, but for me there are few topics as complex and as fascinating as the human brain. How it evolved in us as a species, how it develops differently in each individual. How this small, low-power biological device can store memories for years, carry out complex tasks, explore large decision trees in a matter of seconds or compose the 9th Symphony in D Minor. How it enables the existence of the individually unique bundle of thoughts, feelings and self-awareness that is consciousness.

<!--more-->

All of this is still largely uncharted territory, and one that Neuroscience is out to explore. Neuroscientists exploit a host of different techniques and experimental strategies to try and penetrate the secrets of the brain; most recently, data-heavy techniques are being employed, as it is becoming common in other scientific fields as well. These techniques rely on large amounts of data from brain imaging techniques and electrical activity measurements and they deploy data mining algorithms to discover patterns and to match them with the activity of known neural structures.

Sounds a bit like a shot in the dark, and it more or less is. Just how much it is has been illuminated by a <a href="http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005268">recent article</a>, freely available at PLOS Computational Biology, that tells a cautionary tale about neuroscience and some of its most established experimental strategies. Jonas and Kording, the authors, build on an interesting concept: they take a <em>microprocessor</em> (one of the earlier models, thus among the simpler), put it in a black box and analyze it like they would analyze a brain, using the same measurements and data analysis techniques. The reasoning is the following: we rely on these techniques to penetrate that darkest of mysteries, the working of the human brain, but are they truly effective? Can they allow us to learn anything truly meaningful? In order to gauge this, let us apply the same techniques to a system that is  man-designed, man-made and thus completely understood, and see if they give back to us the same knowledge we already have on it.

<img class="size-medium wp-image-991 alignright" src="https://dev.mikamai.com/wp-content/uploads/2017/01/014555-famicom-donkeykong-300x263.gif" alt="" width="300" height="263" />Let us have a look at what they actually did, also because it can help us laymen to understand the way neuroscience techniques work. Jonas and Kording had the chip performing a task, i.e. running "Donkey Kong", while registering the activity of the chip's components. Then they correlated the activity of the components with the task being executed. This is analog to correlation studies where brain imaging scans are taken while the subject is taking a test or watching an emotional sequence. The aim is identifying functional units in the brain: if activation of the area correlates with the execution of a certain task, then it is likely that that area is involved in performing the task.

However, the authors found that  most of the significant findings obtained in this way about the chip were either irrelevant or just plainly misleading. For example, the researchers’ algorithms found five transistors whose activity correlated strongly with the brightness of pixels on the screen (meaning that when they activated, pixels turned on on the screen). However, the researchers know that these transistors are actually not involved in the task of drawing images on the screen - they are just involved in a part of the game program which ultimately decides what will be drawn. So, the functional analysis gets it wrong because something can show correlation just by virtue of being upstream or downstream in the task performance.

There is a very naif remark that comes to us -probably by professional idiosyncrasy. It seems that at least part of the problem with these techniques applied in this manner (either to the chip or to the brain) is that we are critically missing an even rudimentary knowledge of the way we get from electrical activity in transistors to the comple<img class="size-medium wp-image-990 alignleft" src="https://dev.mikamai.com/wp-content/uploads/2017/01/BrainComputer-300x200.jpeg" alt="" width="300" height="200" />x functions involved in running a videogame. The leap is from hardware to observed behavior: where is the software? Indeed, the software is deemed the most challenging part of simulating a human brain with a computer, since simulating the hardware (i.e. building a computer that can) <a href="https://www.newscientist.com/article/dn7470--mission-to-build-a-simulated-brain-begins/">is almost within current capabilities</a>, not very far away.

Actually I started writing this article with the intention to write exactly about this, whole brain simulations and its relationship to the quest for AI; then what I meant as the introduction grew into an extreme WoT without even coming close to touching the subject, and I let go. I might come around to the argument again sooner or later, if you're interested.

&nbsp;