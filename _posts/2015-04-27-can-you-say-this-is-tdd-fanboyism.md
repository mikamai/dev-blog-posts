---
ID: 131
post_title: Can you say this is TDD fanboyism?
author: Nicola Racco
post_date: 2015-04-27 13:01:14
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/04/27/can-you-say-this-is-tdd-fanboyism/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/117514052104/can-you-say-this-is-tdd-fanboyism
tumblr_mikamayhem_id:
  - "117514052104"
---
As you can guess from my previous [post](https://dev.mikamai.com/2015/02/25/the-sorrows-of-young-developer/) I believe that tests are not only a useful tool for writing code, but also a great way to deliver quality to your customer, even when you are the customer itself.

To be clear: I’m not an agile fanboy nor I believe that tests are the holy grail, like some people do.
Is quality itself always so important? If two developers have 2 weeks to build an e-commerce from scratch, you can bet they will not write a single line of test, if they want to deliver the code. In this case quality is not that important, at least not as much as writing quickly.
<!--more-->

It's [the old-and-always-true triangle of cost/time/quality](http://en.wikipedia.org/wiki/Project_management_triangle) where you cannot pick more than 2 factors at the same time. Higher quality requires more time and delivers higher cost. You can guess by yourself what happens when you want a cheap product developed in just a few days.

![Code/Time/Quality Triangle](https://dev.mikamai.com/wp-content/uploads/2015/04/quality_triangle.jpg) {.aligncenter}

Some triangle configurations seem (maybe only to me) to have sticked over in the programming languages world: your pull request to a ruby gem is usually not accepted if your code isn’t tested, while a lot of javascript libraries are completely left untested.

My rant comes from the fact that lately we started working on a pre-existing web app - **developed, as far as I know, prioritizing quality** - where the backend had a test coverage of nearly 90%, but no tests were written for the frontend.

![Developer, Y U NO TESTS](https://dev.mikamai.com/wp-content/uploads/2015/04/developer_y_no_tests_meme.png) {.aligncenter}

I asked why, and the frontend developers in charge answered with something like _You know, tests are not that useful_, _You know, writing tests requires a lot of time_, _You know, we are not so used to this practice_, _You know, it’s difficult to write tests for every different browser/platform_.

**NO, I DON'T KNOW**. Even if you don’t know how to write tests with selenium, you can always write down your [stories as cucumber features](https://github.com/cucumber/cucumber/wiki/Feature-Introduction) so that another dev (like me) can [implement the steps' code](https://github.com/cucumber/cucumber/wiki/Step-Definitions). There are also tools to [record tests visually](http://testcafe.devexpress.com/) (click here, click there, and it will record the test), there are tools to run your integration tests on different browsers (like [Sauce Labs](https://saucelabs.com/)), there are tools for everything. It’s only your fault if you’re not writing tests.

I’m not saying there should be a test for each animation, but I would expect at least a test for the navigation flow and the main features. I don’t think you need to test that a modal is showing up with a fade-in effect, but you can easily test that the modal is showing. Even if the fade-in doesn’t work you would have a degradation instead of a critical issue.

When you have this unbalanced scenario, with the backend fully tested and the frontend with no tests at all, then you have a big problem on the backend as well.

Backends and frontends have to communicate. Maybe with a REST API, maybe with a socket, in any case there will be a communication point that is fragile by nature, because shared between the 2 components. Besides testing that the backend is serving data correctly, you also need to check that the frontend is parsing the received data as expected. So at least **an integration test should be mandatory to check that the entire chain is working properly**.

And in fact, analyzing the tickets opened during the QA, 5 bugs out of 10 were on the frontend while 4 bugs out of 10 were on the frontend-backend interaction.

This problem isn’t related to javascript. I don’t really think it’s related to the programming language you’re using, although projects written in javascript, php, etc. usually have an undeniable lower test coverage. Maybe it’s because the typical developer is notoriously lazy, and writing tests requires a lot of effort. Or maybe it’s because you don’t know how tests could really save your day if [you never used them **correctly**](http://thedailywtf.com/articles/Testing-Done-Right). I’m not sure about the real reason behind this, but as today, with projects more complex than ever before, tests should be mandatory when the triangle is pointing towards quality.

In the end I can say I’m not a fanboy of testing at every cost. But surely I’m a fanboy of the minimum indispensable integration tests that every application should have. Only this way you can ensure no critical issues are present.