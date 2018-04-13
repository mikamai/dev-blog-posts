---
ID: 1529
post_title: Two days in (F) sharp company
author: Cecilia Nardini
post_date: 2018-04-13 09:32:31
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2018/04/13/two-days-in-f-sharp-company/
published: true
---
On April 9 and 10 I attended a two-days <a href="https://www.avanscoperta.it/en/training/practical-machine-learning-with-functional-programming-workshop/">workshop on machine learning with F#</a> with <a href="https://twitter.com/brandewinder">Mathias Brandewinder</a>.

<!--more-->

This may come as a surprise considering that
<ol>
 	<li>I can be considered at best a novice in machine learning</li>
 	<li>My (limited) programming background is exclusively in OO languages</li>
 	<li>I work on a Mac :)</li>
</ol>
Actually the latter point turned out not to be a problem at all. The F# Ionide extension for Visual Studio Code is installed in a whiff and enables a fully functional IDE for F#, complete with package manager (paket) and interactive terminal. To be honest, some IDE features turned out to be positively sluggish but everything else was -contrary to expectations- absolutely painless.

As for the other two points...well, they weren't a problem either. Mathias is a great teacher and he used a hands-on, step-wise approach with small exercises gradually introducing both the main concepts related to machine learning techniques and the main features of F#. In just under two days we were doing really cool stuff like recognizing handwritten digits or predicting how many people will be around on a bike during a windy day in San Francisco. If you'd like to get a flavor of Mathias' effective training method I can advise this <a href="https://c4fsharp.net/#list-of-dojos">list of F# code Dojos</a>. Many are written by him and some are about machine learning tasks.

All in all, F# impressed me as a beautiful language with a lot of potential. It has a clean, modern syntax all the while relying on stable and strong constructs and maintaining complete interoperability with .NET libraries. It is strongly typed, but the type inference is very effective and does most of the work. One of the most notable features of the language is the Type Provider, a component that allows you to generate new types that your program will then accept and consume. As a notable use case, you can use the <a href="https://fsharp.github.io/FSharp.Data/">F# Data library</a> to cast a JSON API response into a new type with all the attributes contained in the JSON. In just one line! (In case you are wondering, the example is in the link ;))

One of the highlights of the workshop was the concept of functional programming applied to machine learning tasks. Indeed, the functional paradigm resonates very well with the concept of data transformation and processing that is at the basis of machine learning. Most of the basic ML algorithms can be expressed in F# in a few elegant lines of sequential mapping. For instance, this is the code for finding the nearest neighbor of a record within a set of data:
<pre><code>
module NearestNeighbor =
    //define the euclidean distance function
    let distance (xs:float[]) (ys:float[]) =
        Array.map2 (fun x y -&gt; pown (x-y) 2) xs ys
        |&gt; Array.sum
        |&gt; sqrt

    //define the function that returns the item with minimal distance from the given sample
    //example is the set of labeled known records
    let classify examples item = 
        examples 
        |&gt; Array.minBy (fun (label,observation) -&gt; 
            distance item observation) 
        |&gt; fst
</code></pre>
In conclusion I think for us F# could be a good candidate for small data-intelligence scripts that can be integrated in production projects and I hope we will be able to try that soon.

P.S.
I attended the workshop with a <a href="http://foundation.fsharp.org/2018-03-08-diversity-program">Diversity Opportunity ticket</a> supported by the <a href="https://twitter.com/fsharporg">F# Software Foundation</a> and sponsored by <a href="https://twitter.com/avanscoperta">Avanscoperta</a>, who was also behind the flawless organization of the workshop. You rock guys!