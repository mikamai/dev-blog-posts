---
ID: 219
post_title: 'Web Procedural Map Generation &#8211; Part 3'
author: Nicola Racco
post_date: 2014-07-30 15:13:19
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/30/web-procedural-map-generation-part-3/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/93312455794/web-procedural-map-generation-part-3
tumblr_mikamayhem_id:
  - "93312455794"
---
As I stated in [my last post about map generation](https://dev.mikamai.com/2014/07/14/web-procedural-map-generation-part-2/), I was satisfied by Voronoi as a map representation system, but it was not the right tool for me because I needed a wrap-wround map.
<!--more-->

After some googling I found [another great post by Amit Patel](http://www.redblobgames.com/grids/hexagons/) that describes  in great detail hexagonal maps, so I decided to use an hexagonal grid for my map. It seemed a good compromise.

Good and fascinating. A regular hexagon:

- can be inscribed inside a circle;
- is composed of six equilateral triangles with 60° angles inside;
- is composed by six edges and six vertices, each edge having the same size.

There are a lot of other interesting facts about hexagons. Last but not least hexagons can be easily found in nature (e.g. honeycombs) and it's often considered nature's perfect shape:

![Honeycomb](https://dev.mikamai.com/wp-content/uploads/2014/07/honeycomb-1024x768.jpg) { .aligncenter }

Back to my goal of map generation, hexagonal maps are not so easy to deal with due to the nature of the hexagons themselves. As Amit’s says:

> _Squares share an edge with four neighbors but also touch another four neighbors at just one point. This often complicates movement along grids because diagonal movements are hard to weight properly with integer movement values. You either have four directions or eight directions with squares, but with hexagons, you have a compromise—six directions. Hexagons don’t touch any neighbor at only a point; they have a small perimeter-to-area ratio; and they just look neat. Unfortunately, in our square pixel world of computers, hexagons are harder to use..._

In any case, having found no libraries to deal with hexagons in js, I decided to write one: [hexagonal.js](https://github.com/mikamai/hexagonal.js). I tried to give the library enough flexibility to be used following any of the approaches described by Amit in his post. At the moment some features are still missing (like the cube coordinates) but I'll add them in the next few days along with demos and better documentation.

[Here you can find a codepen demo](http://codepen.io/nicolaracco/pen/rBEbs/) that uses an hexagonal map and a heightmap.