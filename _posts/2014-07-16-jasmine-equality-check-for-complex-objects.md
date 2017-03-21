---
ID: 225
post_title: >
  Jasmine equality check for complex
  objects
author: Nicola Racco
post_date: 2014-07-16 14:18:49
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/16/jasmine-equality-check-for-complex-objects/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/91949978979/jasmine-equality-check-for-complex-objects
tumblr_mikamayhem_id:
  - "91949978979"
---
Jasmine’s default equality matcher is really good (it's an adapted version of underscore's `isEqual`) but when you work with complex objects, you may want to have more control on what can be equal to what.
<!--more-->

Let's see an example:

```javascript
var Person = (name, surname) {
  this.name = name;
  this.surname = surname;
};

Person.prototype.render = function() {
  if (!this._cached) {
    this._cached = this.name + &quot; &quot; + this.surname;
  };
};

describe(&#039;Person&#039;, function() {
  var p1, p2;
  beforeEach(function() {
    p1 = new Person(&#039;Indiana&#039;, &#039;Jones&#039;);
    p2 = new Person(&#039;Indiana&#039;, &#039;Jones&#039;);
  });
          
  it(&#039;they are equal&#039;, function() {
    p1.render();
    expect(p1).toEqual(p2);
  });
});
```

In this case the spec will fail because even if the two objects are representing the same person, they have different properties (`p1` has a _cached property that has been set during the invocation of render).

My solution in this case is to create the following custom equality matcher for jasmine:

```javascript
beforeEach(function() {
  jasmine.addCustomEqualityTester(function(actual, expected) {
    var toPrimitive = function(o) {
      if (o == null) { return o; }
      if (o instanceof Array) {
        result = [];
        o.forEach(function(i) { result.push(toPrimitive(i)); });
        return result;
      }
      return o.toPrimitive ? o.toPrimitive() : o;
    },
    actualPrimitive = toPrimitive(actual),
    expectedPrimitive = toPrimitive(expected);
    return jasmine.matchersUtil.equals(actualPrimitive, expectedPrimitive);
  });
});
```

and to change the example code as the following:

```javascript
var Person = (name, surname) {
  this.name = name;
  this.surname = surname;
};

Person.prototype.render = function() {
  if (!this._cached) {
    this._cached = this.name + &quot; &quot; + this.surname;
  };
};    

Person.prototype.toPrimitive = function() {
  return { name: this.name, surname: this.surname };
};
    
describe(&#039;Person&#039;, function() {
  var p1, p2;
  beforeEach(function() {
    p1 = new Person(&#039;Indiana&#039;, &#039;Jones&#039;);
    p2 = new Person(&#039;Indiana&#039;, &#039;Jones&#039;);
  });
          
  it(&#039;they are equal&#039;, function() {
    p1.render();
    expect(p1).toEqual(p2);
  });
          
  it(&#039;a person is equal to a plain object&#039;, function() {
    expect(p1).toEqual({name: &#039;Indiana&#039;, surname: &#039;Jones&#039;});
  });
});
```

Spec passing.

When the object contains the `toPrimitive` function, the equality matcher will rely on this one to obtain a more simple object to use to check the equality.