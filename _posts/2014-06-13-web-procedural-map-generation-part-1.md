---
ID: 239
post_title: 'Web Procedural Map Generation &#8211; Part 1'
author: Nicola Racco
post_date: 2014-06-13 22:18:23
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/13/web-procedural-map-generation-part-1/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/88655114619/web-procedural-map-generation-part-1
tumblr_mikamayhem_id:
  - "88655114619"
---
Two weeks ago I started playing with map generation algorithms.

I wanted to come up with a map generator (and eventually a climatic environment simulator) using <del>javascript</del> coffeescript because it's really easy to prototype something and my goal was to learn to do something, not to _really_ do something (so, when talking of "map generation" there is really no need to add obstacles to the quest like OpenGL, strictly typed languages, and so on).
<!--more-->

When talking of map generation algorithms [Polygonal Map generation for Games](http://www-cs-students.stanford.edu/%7Eamitp/game-programming/polygon-map-generation/) by [Amit Patel](http://amitp.blogspot.it/) is like a "Great Introduction to", a "Episode 1", a "Kernigan &amp; Ritchie" of Map Generation algorithms, a... well, you know.

In his post, Amit shows how you can achieve a nice looking map like the following:

![Voronoi Map Goal](https://dev.mikamai.com/wp-content/uploads/2014/06/voronoi-map-goal-16000-shaded.png) {.aligncenter}

So I started following his post and redoing things. My goal was to obtain a similar result, plus spices:

- continents; not only a single island all alone;
- world: a map whose left and right borders were connected, like in a cylinder. A sphere would be even better but it would require to work with 3d;
- more accurate generation of elevation, biome, moisture, etc.

I wanted to write a long post on how to achieve this result following Amit's post, but in the end I decided to publish small posts each one focusing on a different topic.

### KISS, a tile map

I found really useful to start with a simple tilemap: a simple bidimensional array where each x-y coordinate would give me a value to show on the screen.

You could start with something simple like the following code:

```coffee
# return an height value
generateHeight = (x, y) -&gt;
  Math.round Math.random() * 255

# builds a tile map with the given width and height sizes
buildTileMap = (width, height) -&gt;
  tileMap = new Array(width)
  for y in [0...width]
    tileMap[y] = new Array(height)
    for x in [0...height]
      tileMap[y][x] = generateHeight(x, y)
  tileMap
  
drawTile = (ctx, point, size) -&gt;
  ctx.beginPath()
  [x1, y1] = [point.x * size.w, point.y * size.h]
  [x2, y2] = [x1 + size.w, y1 + size.h]
  ctx.moveTo x1, y1
  ctx.lineTo x2, y1
  ctx.lineTo x2, y2
  ctx.lineTo x1, y2
  ctx.lineTo x1, y1
  ctx.closePath()

$ -&gt;
  scene = document.createElement &#039;canvas&#039;
  scene.width = scene.height = 512
  # build a tile map.
  # Tilemap size is the half of the canvas size
  map = buildTileMap 128, 128
  ctx = scene.getContext &#039;2d&#039;
  
  tileSize = 
    w: scene.width / map.length
    h: scene.height / map[0].length

  # draw a 2px circle for each tilemap location
  for y in [0...map.length]
    for height, x in map[y]
      ctx.fillStyle = ctx.strokeStyle = &quot;rgb(#{height},#{height},#{height})&quot;
      drawTile(ctx, { x, y }, tileSize)
      ctx.stroke()
      ctx.fill()

  document.body.appendChild scene
```

[Demo on CodePen](http://codepen.io/nicolaracco/pen/stwIb)

We have a map with a lower size than our _viewport_ (the canvas). For now each tile contains an `height` value (between 0 and 255) that we'll use to represent our height map.

### Heightmap, or a sort of

We need an heightmap to give coherence to elevation data. Generally speaking an heightmap is anything that can provide you a way to set elevation in your map.

For example you can use an image where you check each pixel color to determine the elevation, or you can generate one using various techniques.

### Perlin Noise Function

I started with Perlin Noise, because it seems to be the most opinionated approach. It generates a pattern in a pseudo-random way, like in the following screenshot:

![Perlin Noise](https://dev.mikamai.com/wp-content/uploads/2014/06/perlin_noise.png)

As you can easily presume, each pixel in the above image has its own 'white percentage' that you can use to represent elevation.

In my case, I found several javascript implementations of Perlin Noise, for example [noisejs](https://github.com/josephg/noisejs) which provides you both perlin and simplex noise functions. The previous code requires only a little change to be able to generate elevation via perlin:

```coffee
noise.seed Math.random()
# return an height value
generateHeight = (x, y) -&gt;
  value = noise.perlin2 x / 64, y / 64
  Math.round Math.abs(value) * 255
```

[Demo on CodePen](http://codepen.io/nicolaracco/pen/DcJnt)

See? Much better. We have white shapes on the screen that could be seen like land masses (whiter =&gt; more height) while the black part could be our ocean. As you can see in the `generateHeight` function, we divide `x` and `y` by 64, moving the values in a range between 0 and 2 (because 128 is our tilemap size). The noise function will change accordingly to how you will change the coordinates. Try to reduce/increase the values to see how perlin result will change.

In any case, I wasn’t satisfied by the output of Perlin. After some googling I discovered the _Diamond Square_.

### Diamond Square

The _Diamond Square_ (also called _Random Midpoint Displacement_, _Cloud Fractal_ or _Plasma Fractal_) is one of my preferred algorithms, because its code is easy to understand and it generates coherent and nice looking heightmaps.

![Diamond Square Demo](https://dev.mikamai.com/wp-content/uploads/2014/06/diamond_square_demo.gif) {.aligncenter}

Generally speaking:

- You start setting a prefilled value in the four corners of your map;
- In the _Diamond_ step you set the value of the center of your map using the average value obtained by the four corners plus a little randomisation;
- In the _Square_ step you populate the values of the middle points on the sides (north, east, west and south)
- Now you have four sectors (top-left, top-right, bottom-left, bottom-right) with corner values set. Go to step 2. For each of the sectors and continue this way until the entire map has been populated.

For a more detailed explaination, read [this guide](http://gru.bz/2012/10/diamond-square-algorithm-help/).

I started from an [implementation I found on github](https://github.com/baxter/csterrain/blob/master/src/generate_terrain.coffee) and I cleaned the code: [https://gist.github.com/nicolaracco/519bfdb73597377d2d5f](https://gist.github.com/nicolaracco/519bfdb73597377d2d5f).

Again the only thing to change is the `generateHeight` function:

```coffee
dsquare = new HeightMap(width: 64, height: 64)
dsquare.run()
# return an height value
generateHeight = (x, y) -&gt;
  dsquare.get_cell Math.round(x / 2), Math.round(y / 2)
```

[Demo on CodePen](http://codepen.io/nicolaracco/pen/zGlFu?editors=001)

I found this solution more appealing than perlin because the elevation is more distribuited. If you use imagination, you can see valleys and mountains in all that black, white and shades of gray.

As I said previously, I would like to generate a map whose left and right edges can be wrapped around. With the Diamond Square, I made a simple hack to force the algorithm to use the first row when it need the last one, and to set values in the first row when it tried to set them in the last row: [https://gist.github.com/nicolaracco/70d097180bf4328cf3e4](https://gist.github.com/nicolaracco/70d097180bf4328cf3e4).

[Demo on CodePen](http://codepen.io/nicolaracco/pen/csbeD)

Now comes the best part, we should choose a representation system for our map other than the tilemap, which would constraint us to square tiles.

I’ll talk about Voronoi and HexMaps the next time.