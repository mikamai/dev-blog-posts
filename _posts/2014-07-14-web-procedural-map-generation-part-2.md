---
ID: 226
post_title: 'Web Procedural Map Generation &#8211; Part 2'
author: Nicola Racco
post_date: 2014-07-14 12:04:20
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/14/web-procedural-map-generation-part-2/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/91738790439/web-procedural-map-generation-part-2
tumblr_mikamayhem_id:
  - "91738790439"
---
In my [previous post](https://dev.mikamai.com/2014/06/13/web-procedural-map-generation-part-1/) I created a heightmap and I started looking for a map representation system different from the tilemap.

But a tilemap, composed of square tiles, probably is not the best tool for the job: it would require a lot of tiles to obtain a nice looking result and probably we would suffer from pixeling. A better approach is to use polygons with a larger number of sides such as hexagons.
<!--more-->

[Amit's seminal post](http://www-cs-students.stanford.edu/~amitp/game-programming/polygon-map-generation) about procedural map generation explains the usage of Voronoi relying on irregular polygons, so I sticked with it.

First of all, the definition: a Voronoi diagram is a way of dividing space into regions. Given you have set some random points (called _sites_), once the diagram has been generated, for each site there will be a region consisting of all points closer to that site than to any other. Each region is an irregular polygon and it’s called _cell_.

It's simple to understand. If we have a space in which we define some random X points, the diagram will generate X cells covering all the available space. This means that we have to deal with the following entities:

- **Site**: random point associated with a cell
- **Cell**: irregular polygon of the Voronoi diagram
- **Center**: centroid of the cell
- **Vertex**: a polygon vertex. When it's placed on the border of the diagram it's called _border vertex_
- **Edge**: cell edge

![Voronoi Elements](https://dev.mikamai.com/wp-content/uploads/2014/07/voronoi_elements.jpg) {.aligncenter}

Here some _rules_:

- an edge always belongs to two cells and always contains two vertices;
- border vertices belong to two edges, all the other vertices belong to three edges;
- passing through the _edges relation_, border vertices connect two cells, all the other vertices connect three cells.

Now, if understanding the Voronoi diagram is an easy task, to generate one from scratch is a real pain.

There are mainly two algorithms used to generate Voronoi diagrams:

- [Bowyer–Watson algorithm](http://en.wikipedia.org/wiki/Bowyer%E2%80%93Watson_algorithm)
- [Fortune's algorithm](http://en.wikipedia.org/wiki/Fortune%27s_algorithm)

If you have to implement one of these algorithms on your own, my advice is to try with the Bowyer-Watson one. The Fortune’s algorithm is the most performant with its complexity of `O(n log(n))`, but it's a lot more complicated to understand and the implementations I found are generally a mess. The most readable I found is [this one in python](https://svn.osgeo.org/qgis/trunk/qgis/python/plugins/fTools/tools/voronoi.py).

In javascript there is [this great library](http://www.raymondhill.net/voronoi/rhill-voronoi.html). So, starting from my last codepen, it was a matter of minutes to obtain [a working demo](http://codepen.io/nicolaracco/pen/zGlFu) (_note: to improve readability I moved the heightmap js in an external pen, you can see it in script preferences dialog_).

![Voronoi demo](https://dev.mikamai.com/wp-content/uploads/2014/07/voronoi_demo.jpg) {.aligncenter}

You can see the difference by yourself in the above screenshot (16384 tiles in the tilemap on the left, 16384 seeding sites in the Voronoi diagram on the right).

The code I used to assign elevation is only a stub:

```coffee
for cell in diagram.cells
  # translate voronoi site in a range 0-128 (heightmap size)
  localPoint =
    x: Math.round(cell.site.x / 4)
    y: Math.round(cell.site.y / 4)
  cell.height = heightMap.get_cell localPoint.x, localPoint.y
```

If you want to improve it, you can:

- assign elevation to each vertex;
- define the elevation of each edge as the average value of the elevation of its two vertices;
- define the elevation of a cell as the average value of the elevation of its edges.

### Relax

If you try to reduce the number of sites (or if you enlarge the canvas) you will note that the generated polygons are too much irregular. We can improve the result using the [Lloyd’s relaxation algorithm](http://en.wikipedia.org/wiki/Lloyd's_algorithm).

![Lloyd's relaxation algorithm](https://dev.mikamai.com/wp-content/uploads/2014/07/lloyd_relaxation_method.png) {.aligncenter}

As I stated at the start of the post, each polygon (= a _cell_) has a _center_ and a _site_. The _site_ (represented by a red dot in the above image) is the point used to generate the polygon, the _center_ (represented by a plus sign in the above image) is the centroid of the polygon.

Starting from this premise using the _Lloyd's algorithm_ (also called _Voronoi iteration_) we would basically:

- move each cell site to the center location;
- regenerate the Voronoi diagram.

Every time we execute this algorithm, the polygons will look more regular, like in the following example:

![Voronoi Animation](https://dev.mikamai.com/wp-content/uploads/2014/07/voronoi_animation.gif) {.aligncenter}

Starting from [this demo provided with the Voronoi library I'm using](http://www.raymondhill.net/voronoi/rhill-voronoi-demo5.html) I updated my [demo](http://codepen.io/nicolaracco/pen/vIGdx) to relax sites twice.

### Conclusion

Satisfied with the result, I lost a lot of time having fun with land/water, temperature and a lot of other cool stuff.

But my goal was to obtain a cylindrical map (endless scrolling through left/right edges), and this is not an easy task with a Voronoi diagram. In the following screenshot you can see what I’m talking about:

![Result with Voronoi](https://dev.mikamai.com/wp-content/uploads/2014/07/voronoi_result-1024x623.png) {.aligncenter}

I asked for help on [Reddit](http://www.reddit.com/r/gamedev/comments/27affk/voronoi_for_an_endless_scrolling_map/) and eventually tried to follow some suggestions, but in the end I had to surrender: Voronoi isn't the right tool for an endless scrolling map in a 2d world.

So I had to go back and start from scratch using an HexMap, of which I’ll talk the next time.