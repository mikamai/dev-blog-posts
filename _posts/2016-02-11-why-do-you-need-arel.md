---
ID: 51
post_title: Why do you need Arel?
author: Nicola Racco
post_date: 2016-02-11 09:34:32
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/02/11/why-do-you-need-arel/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/139103991189/why-do-you-need-arel
tumblr_mikamayhem_id:
  - "139103991189"
---
In my opinion Arel is one of the greatest and most underestimated gems. Of course today Arel is also one of the most used gems because it’s shipped in Rails, but how many people use it actively? And why would they?

Since Rails 3 Arel is the base block of ActiveRecord. Every time you pass a hash to `where`, it goes through Arel eventually. Rails exposes this with a public API that we can use when we have to write complex queries.
<!--more-->

So, if without Arel we would have the following code:

```ruby
def visible_posts
  Post.where &#039;owner_id = ? OR public = ?&#039;, id, true
end
```

With Arel we can write:

```ruby
def visible_posts
  Post.where owned_posts.or public_posts
end

private

def table
  Post.arel_table
end

def owned_posts
  table[:owner_id].eq id
end

def public_posts
  table[:public].eq true
end
```

So yeah, code improved... in a certain way.

But say we have a much more complex scenario:

![Arel Scenario](https://dev.mikamai.com/wp-content/uploads/2016/02/arel-scenario.jpg) {.aligncenter}

And say we want to know how many products colored red have been bought at least 3 times in UK during this month.

With plain ActiveRecord you would do something like:

```ruby
start_of_month = Time.now.beginning_of_month
product_ids = OrderItem.
  joins(:color, order: :address).
  group(:product_id).where(&#039;created_at &gt; ?&#039;,  start_of_month).
  having(&#039;count(order_items.id) &gt;= 3&#039;).
  where(addresses: { country: &#039;UK&#039; }, colors: { code: &#039;red&#039; }).
  select(&#039;product_id&#039;)
Product.where(id: product_ids)
```

That generates the following SQL query:

```sql
SELECT &quot;products&quot;.*
FROM &quot;products&quot; 
WHERE &quot;products&quot;.&quot;id&quot; IN (
  SELECT &quot;order_items&quot;.&quot;product_id&quot; 
  FROM &quot;order_items&quot; 
  INNER JOIN &quot;colors&quot; 
    ON &quot;colors&quot;.&quot;id&quot; = &quot;order_items&quot;.&quot;color_id&quot; 
  INNER JOIN &quot;orders&quot; 
    ON &quot;orders&quot;.&quot;id&quot; = &quot;order_items&quot;.&quot;order_id&quot; 
  INNER JOIN &quot;addresses&quot; 
    ON &quot;addresses&quot;.&quot;id&quot; = &quot;orders&quot;.&quot;address_id&quot; 
  WHERE 
    (created_at &gt; &#039;2016-01-31 23:00:00.000000&#039;) AND 
    &quot;colors&quot;.&quot;code&quot; = &quot;red&quot; AND 
    &quot;addresses&quot;.&quot;country&quot; = &#039;UK&#039; 
  GROUP BY &quot;order_items&quot;.&quot;product_id&quot; 
  HAVING count(order_items.id) &gt;= 3)
```

It's ugly and not so performant as you may think (due to `WHERE IN`).

With Arel you can instead write something like:

```ruby
def export
  Product.find_by_sql subset.
    join(products).on(products[:id].eq order_items[:product_id]).
    project(products[Arel.star])
end

def subset
  order_items.
    join(colors).on(colors[:id].eq order_items[:color_id]).
    join(orders).on(orders[:id].eq order_items[:order_id]).
    join(addresses).on(addresses[:id].eq orders[:address_id]).
    group(order_items[:product_id]).
    having(order_items[:id].count.gteq 3).
    where(red_colored.and during_this_month)
end

def products
  Product.arel_table
end

def order_items
  OrderItem.arel_table
end

def orders
  Order.arel_table
end

def colors
  Color.arel_table
end

def addresses
  Address.arel_table
end

def red_colored
  colors[:code].eq &#039;red&#039;
end

def during_this_month
  orders[:created_at].gteq Time.now.beginning_of_month
end
```

That generates:

```sql
SELECT &quot;products&quot;.* 
FROM &quot;order_items&quot; 
INNER JOIN &quot;colors&quot; 
  ON &quot;colors&quot;.&quot;id&quot; = &quot;order_items&quot;.&quot;color_id&quot; 
INNER JOIN &quot;orders&quot; 
  ON &quot;orders&quot;.&quot;id&quot; = &quot;order_items&quot;.&quot;order_id&quot; 
INNER JOIN &quot;addresses&quot; 
  ON &quot;addresses&quot;.&quot;id&quot; = &quot;orders&quot;.&quot;address_id&quot; 
INNER JOIN &quot;products&quot; 
  ON &quot;products&quot;.&quot;id&quot; = &quot;order_items&quot;.&quot;product_id&quot; 
WHERE 
  &quot;colors&quot;.&quot;code&quot; = &#039;red&#039; AND 
  &quot;orders&quot;.&quot;created_at&quot; &gt;= &#039;2016-01-31 23:00:00.000000&#039; 
GROUP BY &quot;order_items&quot;.&quot;product_id&quot; 
HAVING COUNT(&quot;order_items&quot;.&quot;id&quot;) &gt;= 3
```

Much better, even from the performances point of view.

Now, think about how many medium sized applications with a backend full of reports and exports are out there, and think about how many times you saw Arel in action. It's easy to realize that most developers are underestimating Arel and they shouldn't.