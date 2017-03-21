---
ID: 194
post_title: 'A PrestaShop module helper to display &#8220;boxed&#8221; success and error notifications'
author: Gianluca
post_date: 2014-10-24 12:19:18
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/10/24/a-prestashop-module-helper-to-display-boxed/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/100824458569/a-prestashop-module-helper-to-display-boxed
tumblr_mikamayhem_id:
  - "100824458569"
---
First, a little bit of theory:

PrestaShop (PS) implements the concepts of modularity and extensibility through two different entities:
<ol>
 	<li><a href="http://doc.prestashop.com/display/PS16/Creating+a+PrestaShop+module" target="_blank">modules</a></li>
 	<li><a href="http://doc.prestashop.com/display/PS16/Managing+Hooks?src=search" target="_blank">hooks</a></li>
</ol>
The first ones are actually wrappers for the custom functionality you may need on your store while the latter ones are the connection points offered by the PS core.

<!--more-->

Hooks can be used by modules to directly implement both frontend and  backend functionality.

In the context of a module you can specify as many hooks as you want to extend the default behavior of your store. This is accomplished by defining methods with the same name of the hooks you want to use.

Besides hooks, modules also support some nice features that can be directly used to handle their configuration panel in the backoffice.

The appearance of the panel is handled by the getContent() method.

To see what I'am talking about, have a look at the code of one of the PS default modules. Here’s the <a href="https://github.com/PrestaShop/blockbanner/blob/master/blockbanner.php#L176-237" target="_blank">blockbanner</a>, for example.

The logic related to the validations and saving of the configuration values is usually placed inside the postProcess() method and for the blockbanner module you can see the code right <a href="https://github.com/PrestaShop/blockbanner/blob/master/blockbanner.php#L124-174" target="_blank">here</a>.

Now…enough with the theory!

In the context of the over mentioned validations and saving you may probably want to display some success or error message according to the result of some operation.

To accomplish this goal you should push the messages you want to deliver to the users of your module right into the errors and success arrays (i.e. errors[], success[]). These are defined inside the Module class that every module should extend to be called a module :P

To display these messages, all you have to do is to call the respective display method, i.e. displayError() and displayConfirmation().

In case your module has a complex configuration panel you may encounter the need to display the errors and the confirmations by following some logic.

The one I stick with is to always display each single message in a separate box.

Here’s an example of two boxes, one for an error and one for a success:

<img src="http://68.media.tumblr.com/60186eb9f11ce5b090f150909168ac34/tumblr_inline_ndqezdl6OH1ss63kf.png" alt="image" />

To do this I use a little helper I wrote.

Here it is:
<pre>    
<code>
  private function displayNotifications($notification_types = array())
  {
    $output = '';
    foreach ($notification_types as $notification_types_key =&gt; $notification_type) {
      $notifications = "_{$notification_type}s";
      foreach ($this-&gt;$notifications as $notification_key =&gt; $notification) {
        $output .= call_user_func_array(
          array($this, 'display'.ucfirst($notification_type)),
          array($notification)
        );
      }
    }
    return $output;
  }
</code>
</pre>
It simply loops through the message arrays you pass as parameter and call the correct display method to render each message inside its box.

All the messages with their respective box are then concatenated into a single string.

That’s all folks.

Cheers! :)