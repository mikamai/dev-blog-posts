---
ID: 1449
post_title: >
  Javascript recursion and pattern
  matching with spread operator
author: Alberto Pellizzon
post_date: 2017-05-26 11:39:20
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/05/26/javascript-recursion-and-pattern-matching-with-spread-operator/
published: true
---
<h2>Problem</h2>
Given an array of measurements ordered by time we have to implement a function that returns an array which will be filled with null measurements for every missing one.

In particular we want at least a measure for each hour after the time the first measure was taken.

This is necessary since we want to display this data in a chart which will try to connect this values with a line and we want to highlight missing measurements problems like in the following image.

<img class="wp-image-1494 aligncenter" src="https://dev.mikamai.com/wp-content/uploads/2017/05/Screen-Shot-2017-05-29-at-12.47.32.png" alt="" width="685" height="297" />

<!--more-->
<h2>Example</h2>
<strong><em>input</em></strong>:
<pre><code>
[
 ["2017-05-01 : 09:30", 45.6 ],
 ["2017-05-01 : 10:50", 58.6 ],
 ["2017-05-01 : 12:30", 65.6 ],
 ["2017-05-01 : 15:30", 45.6 ]
]
</code></pre>
<strong><em>output</em></strong>:
<pre><code>
[
 ["2017-05-01 : 09:30", 45.6 ],
 ["2017-05-01 : 10:30", null ],
 ["2017-05-01 : 10:50", 58.6 ],
 ["2017-05-01 : 11:50", null ],
 ["2017-05-01 : 12:30", 45.6 ],
 ["2017-05-01 : 13:30", null ],
 ["2017-05-01 : 14:30", null ],
 ["2017-05-01 : 15:30", 45.6 ]
]
</code></pre>
NB. time is expressed as string for clarity
<h2>Data structure</h2>
<pre>measurements       :: Array [ [timeInMilliseconds, value], ... ]
timeInMilliseconds :: Integer
value              :: Float
</pre>
<h2>Code</h2>
<pre><code>
function normalize(measurements) {
  const [first, ...rest] = measurements;
  return [first, ...addMeasureForEveryMissingHours(rest, moment(first[0]))]
}
</code></pre>
<pre><code>
function addMeasureForEveryMissingHours(measurements, start) {
  if(measurements.length === 0) return measurements;
  const [next, ...rest] = measurements;
  const nextTime        = moment(next[0]);
  const difference      = moment.duration(nextTime.diff(start)).hours() - 1;
  const missing         = _.times(
                            Math.max(0, difference), 
                            hour =&gt; [moment(start).add(hour + 1, "hours").valueOf(), null]
                          );
  return [...missing, next, ...addMeasureForEveryMissingHours(rest, nextTime)];
}
</code></pre>
<h2>Solution</h2>
<strong>normalize</strong> performs <a href="https://en.wikipedia.org/wiki/Pattern_matching">pattern matching</a> against the measurements getting the <em>first</em> measure and return an array containing <em>first</em> followed by the application of <strong>addMeasureForEveryMissingHours</strong> to the <em>rest</em> of the array passing the <a href="https://momentjs.com/">moment</a> version of the <em>first</em> measure time.

The <strong>addMeasureForEveryMissingHours</strong> function is recursive so we split de definition in
<ul>
 	<li><strong>Ground case: </strong> return en empty array if there are no measurements taken</li>
</ul>
<ul>
 	<li><strong>Recursive case:</strong></li>
</ul>
<ol>
 	<li>perform pattern match on the measurements using the <a href="https://developer.mozilla.org/it/docs/Web/JavaScript/Reference/Operators/Spread_operator">spread operator</a> get us the <em>next</em> measure and the <em>rest</em> measures</li>
 	<li>calculates the difference in hours from the <em>start</em> and the <em>next</em> measure time</li>
 	<li>creates an array of <em>null</em> measurements for every missing hour (using <a href="https://lodash.com/docs/4.17.4#times">lodash times</a>) between  <em>start</em> and <em>next</em></li>
 	<li>again using the spread operator return an array containing the <em>missing</em> measurements followed by <em>next</em> and the application of recursion on <em>rest</em> and <em>nextTime</em> as the new start</li>
</ol>
<h2>Observation</h2>
This is a very simple example showing how powerful Javascript has become with the spread operator allowing us to solve problems in an elegant way.