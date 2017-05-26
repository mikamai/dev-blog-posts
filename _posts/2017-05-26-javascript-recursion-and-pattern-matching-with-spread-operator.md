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
Given an array of measurements ordered by time we have implement a function that will return an array which will be filled with null measurements for every missing one.

This is necessary since we want to display this data in a chart which will try to connect this values with a line and we want to highlight missing measurements problems.
<h2>Example:</h2>
<strong><em>input</em></strong>:
```javascript
[
[&quot;2017-05-01 : 09:30&quot;, 45.6 ],
[&quot;2017-05-01 : 10:50&quot;, 58.6 ],
[&quot;2017-05-01 : 12:30&quot;, 65.6 ],
[&quot;2017-05-01 : 15:30&quot;, 45.6 ]
]
```
<strong><em>output</em></strong>:
```javascript
[
[&quot;2017-05-01 : 09:30&quot;, 45.6 ],
[&quot;2017-05-01 : 10:30&quot;, null ],
[&quot;2017-05-01 : 10:50&quot;, 58.6 ],
[&quot;2017-05-01 : 11:50&quot;, null ],
[&quot;2017-05-01 : 12:30&quot;, 45.6 ]
[&quot;2017-05-01 : 13:30&quot;, null ]
[&quot;2017-05-01 : 14:30&quot;, null ]
[&quot;2017-05-01 : 15:30&quot;, 45.6 ]
]
```

NB. time is expressed as string for clarity
<h2>Data structure:</h2>
<pre>measurements       :: Array [ [timeInMilliseconds, value], ... ]
timeInMilliseconds :: Integer
value              :: Float
</pre>
<h2>Code:</h2>
```javascript
function normalize(measurements) {
const [first, ...rest] = measurements;
return [first, ...addMeasureForEveryMissingHours(rest, moment(first[0]))]
}
```
```javascript
function addMeasureForEveryMissingHours(measurements, start) {
if(measurements.length === 0) return measurements;
const [next, ...rest] = measurements;
const nextTime = moment(next[0]);
const difference = moment.duration(nextTime.diff(start)).hours();
const missing = _.times(difference, hour =&gt; [moment(start).add(hour + 1, &quot;hours&quot;).valueOf(), null]);
return [...missing, next, ...addMeasureForEveryMissingHours(rest, nextTime)];
}
```
<h2>Solution</h2>
<strong>normalize</strong> perform <a href="https://en.wikipedia.org/wiki/Pattern_matching">pattern matching</a> against the measurements getting the <em>first</em> measure and return an array containing <em>first</em> followed by the application of <strong>addMeasureForEveryMissingHours</strong> to the <em>rest</em> of the array passing the <a href="https://momentjs.com/">moment</a> version of the <em>first</em> measure time.

The <strong>addMeasureForEveryMissingHours</strong> function is recursive so we split de definition in
<ul>
 	<li><strong>Ground case: </strong> return en empty array if there are no measurements taken</li>
</ul>
<ul>
 	<li><strong>Recursive case:</strong> perform pattern match on the measurements getting the <em>next</em> measure and the <em>rest,</em>
calculates the difference in hours from the <em>start</em> and the <em>next</em> measure time
creates an array of null measurements for every missing hour (using <a href="https://lodash.com/docs/4.17.4#times">lodash times</a>) between  <em>start</em> and <em>next</em>
return  an array containing the <em>missing</em> measurements followed by <em>next</em> and the application of recursion on <em>rest</em> and <em>nextTime</em> as the new start</li>
</ul>
<h2>Observation</h2>
This is a very simple example showing how powerful Javascript has became with the spread operator allowing us to solve problems in an elegant way.