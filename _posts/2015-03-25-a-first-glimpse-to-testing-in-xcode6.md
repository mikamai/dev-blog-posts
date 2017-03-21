---
ID: 141
post_title: A first glimpse to testing in XCode6
author: Emanuel Carnevale
post_date: 2015-03-25 16:34:47
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/03/25/a-first-glimpse-to-testing-in-xcode6/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/114589380274/a-first-glimpse-to-testing-in-xcode6
tumblr_mikamayhem_id:
  - "114589380274"
---
<p>In Mikamai all of our RoR applications are tested: sometimes we start from user stories, sometimes we just unit test some models and sometimes we practice TDD, but the general consensus is that if tests are present (and up to date) it&rsquo;s better for everyone.</p>

<p>Keeping a sane test environment on an Objective-C (and lately swift) project has always been difficult, but fortunately it seems XCode 6 started supporting it.</p>

<p>So, how to start to unit test our code?</p>

<p>The testing framework we are going to use is <a href="https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/Introduction/Introduction.html#//apple_ref/doc/uid/TP40014132-CH1-SW1">XCTest</a>
When we start a new project, XCode takes care for us of the creation of a separate target for our tests, it also creates a group where to include our test classes and it also provides us of an example test we can start looking at.</p>

<p>If we have created an example app creatively named Example, we&rsquo;ll find an <em>Example Tests</em> target and a file named <code>ExampleTests.m</code></p>

<p>Within it we already have some generated code: setUP and tearDown methods necessary to respectively set and clean up the environment for every single test we are going to describe in that testing class.
We then found a testExample and a performanceExample file: XCode allors us to perform a performance test and choose a baseline time to be used as a reference: if the performance test gets completed in a  time greater than the baseline, a flag is raised as a result of the test.</p>

<p>Testing our XCode apps opens up to endless possibilities for their stability and maintenance and we&rsquo;re going to share hints and tricks in the articles to come.</p>