---
ID: 150
post_title: The Sorrows of Young Developer
author: Nicola Racco
post_date: 2015-02-25 15:17:31
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/25/the-sorrows-of-young-developer/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/112047697144/the-sorrows-of-young-developer
tumblr_mikamayhem_id:
  - "112047697144"
---
This is an almost-true story about a young passionate developer. It was late 2004 and he had just started working in a small company that offered everything he desired: a good salary, working with his preferred programming language, approaching complexities, and modeling architectures.
<!--more-->

This was not the first working experience for the young developer. But his very first project had proven to be _problematic_. At the time, he believed features would never change. But he was wrong, and every feature change would require a complete refactoring, causing bugs and an enormous wasting of time. He had even tried some virtuous approaches like writing tests. But his tests required maintenance, and time to be written, and even more time to be executed.

As every young developer, he was growing up listening an experienced developer who used to say "watch out! premature optimization is evil!", and then "write tests! tests! tests!". Maybe he was just refactoring a small utility method when the experienced developer would come to him with a stern look warning "you're not doing premature optimization, are you?", or "are you writing tests, right?".

But all these warnings were falling on deaf ears. Because the young developer couldn't understand why premature optimization should be evil and tests should be good. From his own previous experience he knew that following specifications doesn't work in the long run (because they tend to change), and writing tests was a wasting of time.

"Why on earth do I have to rewrite my code every time? Why on earth do I have to write the code now and refactor it later, when I can write the best code in the world right now? And why, why on earth do I have to spend all my time writing useless tests?" was wondering the young developer.

One day, the young developer started working on a new project. He decided to ignore the experienced developer warnings; he wanted every piece of code to be fast, configurable, and robust, to overwhelm every specification change. Specifications were clear, but he would go beyond them. For example, when the feature was to build a product code ending with an uppercase 'S', he created a configurator object so that the ending letter could be decided via configuration, and via configuration could also be decided if the letter should be uppercased or lowercased. When the specification required some validations, he created a huge validator that would handle not only what the specification required, but a lot more.

After writing the core of his project, a feeling of perfection pervaded the young developer. "The experienced developer is wrong!" said in triumph the young developer watching his wonderful creation. He worked days and nights to complete the project, that saw its production release some weeks later.

Time passed...

One day the customer notified them of a bug. The experienced developer looked at the bug, remaining horrified from what the monitor was showing: where the young developer had saw a cathedral, the experienced developer was facing a slum, where the young developer had saw patterns, the experienced developer was facing an intricate network of classes, where the young developer had saw a faster-than-light code, the experienced developer was looking at unnecessary complex algorithms. And he had no desire to touch that code. So he asked the young developer to fix his own bug.

The only idea that his code was not considered beautiful by others made the young developer upset. He opened the project, full of rage… only to discover that the code was incomprehensible to him as well! There was no clear meaning behind the code. "This is why I'm not using this language anymore. It has an ugly syntax" was the first reaction of the young developer. But he knew deep down that this wasn't the real problem. The real problem was him.

At the end of the day the bug was fixed, only to generate another bug, discovered the day after. Every fix was altering the thin balance inside the project, like a small black patch on a shiny white dress.

The young developer was now hopeless. His cathedral had begun to falter, and he felt the collapse was imminent. "Maybe I'm not the right person for this job. Why cannot I write code the right way ?" the young developer asked himself. In a mixture of depression and angriness, the young developer opened a project maintained by the experienced developer.

What he saw astonished him: the code was easy to read, with comments and tests. It was not so different from the code he used to write at his beginnings, with some clear exceptions: there was no extensive configuration, each line of code was tested, every method had a meaningful name and was short (max ten lines of code), and did only what was required, and every file contained only the methods it strictly required to do its job.

In that moment of dejection, the experienced developer approached the young developer. While sitting close to him, he started to refactor the code that was causing all those bugs.

They worked together for days, at times the experienced developer was writing and the young one watching at how the experienced developer was solving problems; on other occasions the young developer was writing and the experienced one supervising.

A few days later a new deploy marked the bug as fixed. The small portion of code that was causing bugs was now tested, easy to read, and stable. The experienced developer looked at the young one: "Do you understand now?".

The young developer nodded. He was getting it now. The key to perfection wasn't to predict the future, but to write code that could be easily changed, tested (so that a change would not cause any bug), and that fulfilled only the current requirements. And while he was realizing this, he noticed he was changing, he was becoming the almost experienced developer.

"Can we now refactor the entire project?" asked the young developer.
"Surely not! There's no budget for that" flatly replied the experienced developer.
"But what if other bugs appear?" asked the young developer.
"We'll call a freelancer to fix that shit" replied the experienced one.

And then, the almost experienced developer started writing good code, ready to learn one other lesson. But this is another story.

**Moral for the young developer:** Keep looking back at the code you wrote in the past, and don't feel disappointed if your code doesn't look beautiful as it used to be.

**Moral for the experienced developer:** When there is a young developer around, you'll have to handle some of his shit. Your best chance is that sooner than later he will learn how to write decent code.

**Moral for the freelancer:** You might as well raise your rates :-P