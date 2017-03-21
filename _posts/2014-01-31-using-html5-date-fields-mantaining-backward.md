---
ID: 291
post_title: >
  Using HTML5 date fields mantaining
  backward compatibility
author: Nicola Racco
post_date: 2014-01-31 10:22:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/01/31/using-html5-date-fields-mantaining-backward/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/75139920117/using-html5-date-fields-mantaining-backward
tumblr_mikamayhem_id:
  - "75139920117"
---
Chrome and Opera (respectively from versions 20 and 12) implement a native date picker. Now, when you have an input field with the `date` type:

```html
&lt;input type=&quot;date&quot; name=&quot;user[born_on]&quot; id=&quot;user_born_on&quot; /&gt;
```

these two browsers will automatically inject in the page their native datepicker, according to the [input[type=date] HTML5 specs](http://dev.w3.org/html5/markup/input.date.html).

![Native datepicker](https://dev.mikamai.com/wp-content/uploads/2014/01/datepicker.png) {.aligncenter}

And, at least in Chrome, the widget is really really good.

> _Btw, this doesn’t happen only for date fields. In fact in HTML5 we can find other new input types specifically created to deal with dates and times: `datetime`, `datetime-local`, `time`. Chrome and Opera will show a picker also for these input types._

There is only one little, negligible problem: as I said **this feature is implemented only in Chrome and Opera!**
<!--more-->

Now let’s say you want to have a datepicker in Firefox and IE too, and you are using the JQuery UI Datepicker:

```javascript
$(function() {
  $(&#039;input[type=date]&#039;).datepicker();
});
```

And you will obtain:

![Datepicker issue](https://dev.mikamai.com/wp-content/uploads/2014/01/datepicker_issue.png) {.aligncenter}

Cool, two datepickers at once! :)

Okay, let’s solve this problem the fast way. Modernizr comes with useful tests for these new html5 fields that you can find inside `Modernizr.inputtypes`. Change your js like the following:

```javascript
$(function() {
  if (!Modernizr.inputtypes.date) {
    $(&#039;input[type=date]&#039;).datepicker();
  }
});
```

Better. Chrome, Opera, mobile devices and each browser supporting the new html5 input specifications will show the native datepicker, while the other browsers will use JQuery UI implementation.

But, alas, there is one last problem to solve. The value format. In fact JQuery UI implementation will set the value according to your preferences, while the native implementation will:

- show the value in the format specified within your O.S. Regional Settings
- store the value using the `yy-mm-dd` format

We **need** to store the value using the `yy-mm-dd` format otherwise our backend cannot parse the given value.

One easy way to solve this problem would be to set the dateFormat of the datepicker to `yy-mm-dd`. The result is not so _consistent_ but will work.

![Datepicker fix](https://dev.mikamai.com/wp-content/uploads/2014/01/datepicker_fix.png) {.aligncenter}

The other solution, the better one, is to use an hidden field. In practice, when we have to use JQuery UI we have to:

1. create an hidden input field with the same name;
2. remove the name in the original input field to prevent value collision on submit;
3. setup the datepicker with date format `yy-mm-dd`;
4. set the original value inside the datepicker;
5. change the datepicker format to our preferred configuration (e.g. `dd/mm/yy`);
6. set the datepicker options `altField` and `altFormat`. This way when the datepicker value is changed, JQuery UI will set the value using `altFormat` format inside the hidden field with the correct format.

Steps 3..5 are needed to correctly show the prefilled value. [Here's the code](https://gist.github.com/nicolaracco/8729504).

![That's all folks!](https://dev.mikamai.com/wp-content/uploads/2014/01/thats_all_folks.gif) {.aligncenter}