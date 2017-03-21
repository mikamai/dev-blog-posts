---
ID: 85
post_title: Implement a weighted shuffle in Ruby
author: Diomede Tripicchio
post_date: 2015-09-23 14:15:45
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/09/23/implement-a-weighted-shuffle-in-ruby/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/129710741584/implement-a-weighted-shuffle-in-ruby
tumblr_mikamayhem_id:
  - "129710741584"
---
In my [previous post](http://dev.mikamai.com/post/124568650434/implement-fisher-yates-shuffle-in-ruby) I’ve implemented a Fisher-Yates shuffle algoritm in Ruby. Today I want to try somenthing complex: a weighted (or priority) based shuffle.

<!--more-->

Given an array of hashes like this:

```ruby
weighted_array = [
  {value: &quot;HIGH 1&quot;, weight: 4},  
  {value: &quot;HIGH 2&quot;, weight: 4},  
  {value: &quot;MID 1&quot;, weight: 2},  
  {value: &quot;MID 2&quot;, weight: 2},  
  {value: &quot;LOW 1&quot;, weight: 1},  
  {value: &quot;LOW 2&quot;, weight: 1},  
]
```

I want to get an array of 10 elements randomly picked across it based on their weights; something like this:

```ruby
result = [&quot;HIGH 1&quot;, &quot;MID 2&quot;, &quot;HIGH 2&quot;, &quot;HIGH 1&quot;, &quot;LOW 2&quot;, &quot;MID 1&quot;, &quot;HIGH 2&quot;, &quot;HIGH 1&quot;, &quot;MID 1&quot;, &quot;HIGH 2&quot;]
```

To do this I need to calculate the probability of any weight present in the first array. In order to achieve this I have to calculate the sums of all distinct weights.

```ruby
def sum_of_weights weighted
  # .to_f for decimals in probability calculation
  weighted.map{ |w| w[:weight] }.uniq.inject(:+).to_f
end
```

Now I can call `sum = sum_of_weights(weighted_array)` and get **7**, with the `sum` I can get the probability of each weight, simply with `probability = weighted_array[0][:weight] / sum` and so on

Now I can define the cut offs associated to each weight using the probability calculator below.

```ruby
def probabilities weighted_array
  sum_of_weights = sum_of_weights(weighted_array)

  # initial probability
  probability = 0.0

  weights = weighted_array.map{ |w| w[:weight] }.uniq.sort
  weights.map do |w|
    current_probability = (w / sum_of_weights).round(3)
    # sum the current_probability with the previous
    probability += current_probability
    { weight: w, probability: probability }
  end
end
```

Given the ***weighted_array*** the above function returns an array like this:

```ruby
probabilities = [
  { weight: 1, probability: 0.143 },
  { weight: 2, probability: 0.429 },
  { weight: 4, probability: 1.0 },
]
```

The next step is to generate a random number to select a weight based on its probability. Conceptually I need a series of if..elsif, but given the ***probabilities*** array I can use this function:

```ruby
def get_weight probabilities
  r = rand
  probabilities.each do |p|
    return p[:weight] if r &lt; p[:probability]
  end
end
```

Once I get the weight randomly, I can pick an element from `weighted_array`, filtered by the selected weight. With the ruby `Array#select` method:

```ruby
def by_weight a, weight
  a.select{ |el| el[:weight] == weight }
end
```

At this point I can use the `Array#sample` method and get a random element from the filtered array. By repeating this 10 times I get the desidered result!

```ruby
result = []
10.times do
  weight = get_weight probabilities
  result &lt;&lt; by_weight(weighted_array, weight).sample[:value]
end
```

You can find all the code and little more [here](https://gist.github.com/oeN/ec60a04364adaff479ea)

During my research I have discovered various methods to achieve the desidered result. What I presented here is what I think has the best trade off between difficulty and result.

I hope to explore in more detail this sort of thing in my next post.

Bye and see you soon!