---
ID: 84
post_title: The Paranoia gem
author: Nicola Racco
post_date: 2015-09-25 11:04:33
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/09/25/the-paranoia-gem/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/129839902199/the-paranoia-gem
tumblr_mikamayhem_id:
  - "129839902199"
---
If you didn't know it yet, [paranoia](https://github.com/radar/paranoia) is a gem that handles the [soft deletion](http://www.25hoursaday.com/weblog/2009/11/23/BuildingScalableDatabasesPerspectivesOnTheWarOnSoftDeletes.aspx) for your ActiveRecord models.
<!--more-->

Adding it to your model is straightforward. Given you want to add soft-deletion to your Client model, first run `rails generate migration AddDeletedAtToClients deleted_at:datetime:index` to add the `deleted_at` column in the `clients` table. Then add `acts_as_paranoid` to your Client model, like in the following snippet:

```ruby
class Client &lt; ActiveRecord::Base
  acts_as_paranoid
end
```

From now on, `Client#destroy` sets the current datetime in the `deleted_at` column, while `Client` default scope returns only records which have a nil value for `deleted_at`.

To really destroy a client, you should use `#really_destroy!`.
To list all clients (even the deleted ones) you can use `Client.with_deleted`, while you can use `Client.only_deleted` to get only deleted records.

Paranoia even follows relationships. So if you are in the following scenario:

```ruby
class Client &lt; ActiveRecord::Base
  has_many :addresses, dependent: :destroy
  acts_as_paranoid
end

class Address &lt; ActiveRecord::Base
  belongs_to :client
  acts_as_paranoid
end
```

when you call `Client#destroy` paranoia will soft-delete all its addresses.

The only problem I found after my setup was with the uniqueness validation. In this scenario:

```ruby
class Client &lt; ActiveRecord::Base
  acts_as_paranoid
  validates :name,
    :uniqueness =&gt; true
end
```

the uniqueness validator will check among deleted records too, and in my case I did want to check only among not-deleted records.
For this purpose there is a gem called (paranoia_uniqueness_validator)[https://github.com/anthonator/paranoia_uniqueness_validator] that can be used this way:

```ruby
class Client &lt; ActiveRecord::Base
  acts_as_paranoid
  validates :name,
    :uniqueness_without_deleted =&gt; true
end
```

And it works as expected.

So, easy soft-deletion for all :)