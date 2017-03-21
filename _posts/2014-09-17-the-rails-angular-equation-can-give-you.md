---
ID: 208
post_title: >
  The Rails + Angular equation can give
  you satisfaction
author: Nicola Racco
post_date: 2014-09-17 14:31:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/17/the-rails-angular-equation-can-give-you/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/97732414859/the-rails-angular-equation-can-give-you
tumblr_mikamayhem_id:
  - "97732414859"
---
It's now 3 or 4 years that frameworks like [Angular](http://angularjs.org) and [Ember](http://emberjs.com) have made their impressive appearance on the stage.

In time they have generated a lot of hype/discussion and flames too (like the old [jquery](http://www.jquery.com) vs. [prototype](http://prototypejs.org)).
<!--more-->

As a Ruby on Rails lover I've been watching Angular development from distance. Ember seemed more _Rails Way_ to me (i.e. it's _convention over configuration_ style and its separation between controllers, views and models), but the problem with Ember was they were continuing to do breaking changes to the API, and it’s frustrating when you google for a problem and find only solutions that are now old and useless (I tried it again some weeks ago and situation is quite still the same).

So I eventually started to learn Angular and in the end I started loving it. It's minimal, compared to Ember, but it's stable, with a good set of API, and a mature community.

### Chase away your doubts

Using these frameworks may seem a step back for a Rails developer. It may seem even unnecessary to integrate a js framework and lose many good commodities we are used to: remote forms, turbolinks, helpers, and so on.

But in my experience mixing Rails and a framework like Angular allows you to deliver more in the same timeframe. And above all, **it allows you to write better code**.

Indeed when developing rich internet applications with Rails during the Web >= 2.0 era, there are some common problems:

1. there is a lot of html markup and a lot of js/coffee and they are completely separated;
2. dividing your markup in partials will increase the readability of the html, but will create confusion because you have to visit more files to know where a certain information is presented;
3. preventing all your js/coffee to sit in one file in a flood of hundreds and hundreds of lines is not that easy. A lot of people struggled to find a decent solution;
4. and finally, testing. If you tried to write integration tests for a page with a lot of javascript (interactions, transitions and so on) then you already  know how hard it can be. (Angular has its own tool: [Protractor](https://docs.angularjs.org/guide/e2e-testing))

### Let’s take a look at the equation

When you follow this hybrid approach, frontend and backend are completely isolated. Angular handles your frontend and communicates with the Rails app via API calls.

Inside the Rails app, the setup is the following:

- inside the `Gemfile` remove _turbolinks_ and add:
  - `bower-rails`: it provides a rake task to install js dependencies via bower, placing them inside `lib/assets` or `vendor/assets` (I prefer the latter)
  - `angular-rails-templates`: this gem allows you to insert angular templates in `app/assets/templates` or `app/assets/javascripts/templates` and to include them via asset pipeline (and this allows you to create a real dependency between the template and its behavior).
- init bower with `rails g bower_rails:initialize json` and edit it adding angular dependencies:

  ~~~json
    {
      "vendor": {
        "name": "bower-rails generated vendor assets",
        "dependencies": {
          "angular": "latest",
          "angular-route": "latest",
          "angular-resource": "latest"
        }
      }
    }
  ~~~

- install js libraries via bower using `rake bower:install`
- edit `app/assets/javascripts/application.js`:

  ~~~javascript
    //= require jquery
    //= require jquery_ujs
    //= require angular
    //= require angular-rails-templates
    //= require angular-route
    //= require angular-resource
    //= require_tree .
  ~~~

- create a new controller with an empty view and insert a root route in `config/routes.rb`
- inside `app/views/layouts/application.html.erb` insert the directives `ng-app` and `ng-view`:

  ~~~html
    <!DOCTYPE html>
    <html lang="<%= I18n.locale %>" ng-app="my-app">
      <head>
        <title>My Angular + Rails App</title>
        <meta charset="utf-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <%= stylesheet_link_tag 'application', media: 'all' %>
        <%= javascript_include_tag 'application' %>
        <%= csrf_meta_tags %>
      </head>
      <body ng-view>
      </body>
    </html>
  ~~~

It’s Angular turn. As we specified in the layout, we want to create an Angular module called `my-app` that handles the views inside the `<body>` tag.

It’s important here to define our structure inside `app/assets`. I use the following:

```text
my-app/
- app/
  - assets/
    - javascripts/
      - templates/ # angular templates
      - welcome/ # I scope templated by controller / directive
        - home.html.erb
      - controllers/ # angular controllers
        - welcome-controller.js.coffee
      - directives/ # angular directives
        - my-directive.js.coffee
      - services/ # angular services
        - book.js.coffee
      - application.js # root point
      - my-app.js.coffee # angular app
```

As you can see everything is defined inside `app/assets/javascripts`. Your rails app will return only the layout and js and templates files. This way no further requests have to be made to fetch the templates, and template and behavior are strictly bound together because:

- it's now the behavior (the Angular module) that requires a certain template (creating a real dependency)
- template structure is reporting which angular module is requiring the template itself

### Rails as an API service

At the moment Rails is acting like an assets container, but nothing more.

Now it comes the communication between Angular and Rails. The key point here is to treat Rails as an API service. E.g.

We have to handle a collection of books. A simple controller that returns all books could be like the following:

```ruby
# app/controllers/books_controller.rb

class BooksController &lt; ApplicationController
  layout false
  respond_to :json

  def index
    @books = Book.all
    respond_with @books
  end
end
```

The controller returns only json responses. In angular we have now to create the other end of the connection. This is achieved with a service that relies upon `angular-resource`:

```coffee
# app/assets/javascripts/services/book.js.coffee

app = angular.module &#039;bookService&#039;, [&#039;ngResource&#039;]

app.factory &#039;Book&#039;, [&#039;$resource&#039;,
  ($resource) -&gt;
    $resource &#039;/books.json&#039;, {},
      all: {}
]
```

Now, to use this we need only to require the service inside an Angular module, like in the following example:

```coffee
# app/assets/javascripts/directives/books.js.coffee

app = angular.module &#039;booksList&#039;, [&#039;bookService&#039;]

app.directive &#039;booksList&#039;, [&#039;$scope&#039;, &#039;Book&#039;, 
  (Book) -&gt;
    {
      restrict: &#039;EA&#039;
      template: &quot;&quot;&quot;
        &lt;ul&gt;
          &lt;li ng-repeat=&quot;book in books&quot;&gt;{{book.name}}&lt;/li&gt;
        &lt;/ul&gt;
      &quot;&quot;&quot;
      controller: -&gt;
        # it returns a promise, the callback is called once the load is completed
        $scope.books = Book.all {}, -&gt;
          console.log &#039;successfully loaded books&#039;
    }
]
```

And the communication between our two layers is complete.

[On github I uploaded a simple library app with a CRUD that should be enough as a demo.](https://github.com/mikamai/angular-rails-example)