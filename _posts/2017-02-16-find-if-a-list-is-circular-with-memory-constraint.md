---
ID: 1063
post_title: >
  Find if a list is circular with memory
  constraint
author: Alberto Pellizzon
post_date: 2017-02-16 17:20:58
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/02/16/find-if-a-list-is-circular-with-memory-constraint/
published: true
---
Usually developers when thinking about a solution put their focus on the complexity of the algorithm (O(n), O(n^2) ...) rather than the memory consumption, so lets see an example where the <strong>constraint</strong> is the memory <strong>S</strong>.
I would like to share the solution to an interesting exercise that I was asked to solve during a technical interview.
<h3><!--more-->Problem</h3>
Implement a function which says if a list is <strong>circular</strong>.
A circular list is made by a sequence of elements, each one connected to the next in sequence where the last must be connected to any of the previous ones.

Examples :

<img class="wp-image-1103 aligncenter" src="https://dev.mikamai.com/wp-content/uploads/2017/02/Screen-Shot-2017-02-16-at-09.54.14.png" width="577" height="283" />

The only constraint for the solution proposal is that it has to use constant memory a.k.a <strong>S(1)</strong> .
<h3>Solution</h3>
Let's consider which are the types of lists that we might be handling:
<ol>
 	<li><em>Empty list</em>
<img class="aligncenter wp-image-1070 " style="margin: 20px;" src="https://dev.mikamai.com/wp-content/uploads/2017/02/empty.png" width="95" height="48" /></li>
 	<li><em>Non empty list</em>
<img class="aligncenter wp-image-1071 " style="margin: 20px;" src="https://dev.mikamai.com/wp-content/uploads/2017/02/n_empty.png" width="340" height="53" /></li>
 	<li><em>Ciclic list</em>
<img class=" wp-image-1072 aligncenter" style="margin: 20px;" src="https://dev.mikamai.com/wp-content/uploads/2017/02/circular-1.png" alt="" width="187" height="92" /></li>
 	<li><em>Closed cyclic list</em> (cyclic list which is not knotted to the first element)
<img class="aligncenter wp-image-1073 " style="margin: 20px;" src="https://dev.mikamai.com/wp-content/uploads/2017/02/closed.png" width="356" height="114" /></li>
</ol>
<p class="p1">Given the space constraint, the idea is to iterate the list with two pointers, one <strong>slow</strong> and one <strong>fast</strong> moving them simultaneously: slow one moves by one element at a time and fast one by two. In this case we use only a constant number of pointers respecting the space constraint.</p>

<ul>
 	<li>If the given list is <strong>EMPTY</strong> our function will return <strong>False.</strong></li>
 	<li>If <b>fast</b> or <b>slow</b> will become <strong>EMPTY</strong> while iterating over the list our function will return <strong>False.</strong></li>
 	<li>We can consider the list of type 3  as a special case of the list of type 4.</li>
 	<li>If at some point <b>fast</b> , iterating the list more quickly, will be equal to <b>slow</b> the list is cyclic and so our function will return <strong>True</strong>, this last scenario is graphically explained by the following schema where slow is x and fast is y.</li>
</ul>
<p style="text-align: center;"><img class="alignnone wp-image-1075 " src="https://dev.mikamai.com/wp-content/uploads/2017/02/alg-1.png" width="433" height="494" /></p>

<h3 style="text-align: left;">Implementation</h3>
The following implementation code is written in<strong> Haskell</strong>. <a href="https://www.haskell.org/">Find more about that</a>.
<h4>List data structure definition</h4>
The list in Haskell is recursively defined as <strong>EMPTY</strong> or a node containing a value connected at the rest of the list.
```haskell
data List a = Empty | Cons a (List a)
```
<h4>Creating a non empty list of (type 2)</h4>
The following code will produce a list like : 1 - 2 - 3 - 4 - EMPTY
```haskell
simpleList :: List Integer
simpleList = Cons 1 . Cons 2 . Cons 3 . Cons 4 $ Empty
```
<h4>Creating a cyclic list (type 3)</h4>
The following code will produce a list like ( 5 - 6 - 5 ...) where (5 - 6 - 5 ...) is the cyclic part.
```haskell
loop :: List Integer
loop = Cons 5 knot
where knot = Cons 6 loop
```
<h4>Creating a closed cyclic list (type 4)</h4>
The following code will produce a list like 1 - 2 - 3 - 4 - ( 5 - 6 - 5 ...) where (5 - 6 - 5 ...) is the cyclic part.
```haskell
closedList :: List Integer
closedList = Cons 1 . Cons 2 . Cons 3 . Cons 4 $ loop
```
<h4>Implementing the algorithm</h4>
The<em> isCyclic </em>function matches a list made of at least 2 elements, assign <strong>slow</strong> and <strong>fast</strong> (our pointers), otherwise return <strong>False</strong> because a list shorter than 2  elements is by definition not cyclic.
```haskell
isCyclic :: List Integer -&gt; Bool
isCyclic (Cons slow (Cons k fast)) = isCyclic` fast (Cons slow . Cons k $ fast)
isCyclic _ = False
```
The &lt;em&gt;isCyclic&#039; &lt;/em&gt;function returns&lt;em&gt; &lt;/em&gt;&lt;strong&gt;True &lt;/strong&gt;whether &lt;strong&gt;slow&lt;/strong&gt; and &lt;strong&gt;fast &lt;/strong&gt;matches, otherwise move the pointers respectively by one e two elements, if &lt;strong&gt;fast &lt;/strong&gt;will became&lt;strong&gt; EMPTY &lt;/strong&gt;our function will return&lt;strong&gt; False&lt;/strong&gt;.
```haskell
isCyclic` Empty _ = False
isCyclic` (Cons fast (Cons _ fastRest)) (Cons slow slowRest)
| fast == slow = True
| otherwise = isCyclic` fastRest slowRest
```
<em><a href="https://gist.github.com/apellizzn/240a352be791ab4d4153c8b42880aaff">Here</a> is the gist if you want to try it by yourself.</em>