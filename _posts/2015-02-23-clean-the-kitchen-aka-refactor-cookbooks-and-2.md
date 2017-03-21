---
ID: 151
post_title: >
  Clean the kitchen, aka refactor
  cookbooks and recipes
author: Gianluca
post_date: 2015-02-23 10:58:18
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/02/23/clean-the-kitchen-aka-refactor-cookbooks-and-2/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/111852114524/clean-the-kitchen-aka-refactor-cookbooks-and
tumblr_mikamayhem_id:
  - "111852114524"
---
<p>I was planning to continue my series of posts entitled “Wordpress and OpsWorks, Pride and Prejudice”, but today I want to talk about something a little bit less specific. Anyway, for the curious readers, <a href="http://dev.mikamai.com/post/108731475409/wordpress-and-opsworks-pride-and-prejudice">here’s</a> the link with the last post of the aforementioned series.</p><p>The subject of this post doesn’t relate in particular to Opsworks or Wordpress but it is still connected to these topics as it discusses “something” I found myself in need while working on a new cookbook containing some new recipes.</p><p>This something is the wish to refactor and it comes from the fact that the recipes of the new cookbook have a lot in common with the ones of the Wordpress one. Just for the record, the purpose of the new recipes is to handle the deploy of another PHP framework (if Wordpress can be considered a framework) called <a href="https://www.prestashop.com/">PrestaShop</a>.</p><p>Anyway, pushed by the will to improve and dry out the structure of my cookbooks and recipes I started to delve more and more into the Chef docs and to study the other open source cookbooks.</p><p>Among all the sources I stumbled upon let me report this <a href="http://www.prashantrajan.com/posts/2013/06/leveling-up-chef-best-practices/">one</a> as I consider it a great summary of the best practices regarding cookbooks and recipes development.</p><p>One of this best practices is to identify the type of the cookbooks you’re writing. This indeed will let you structure them in a more appropriate way and avoid, in many cases, a lot of duplication.</p><p>In my case for example, the Wordpress and PrestaShop cookbooks can be considered as wrapper cookbooks for a more general one.</p><p>This last, i.e. the mysterious general cookbook, can be considered instead like an application cookbook. You can roughly think it like a sort of parent cookbook whose aim is to reduce the possible duplication between his children.</p><p>An example of these duplication is the recipe needed to handle the import of a database dump.</p><p>In this case the recipe is pretty general as it doesn’t rely on any specific data or behavior concerning the type of the application beeing deployed. <br /></p><p>Here it is:</p><pre>
<code>
node['deploy'].each do |application, deploy|  
  attributes = deploy['set_db']
  
  next if deploy['application_type'] != 'php' or deploy['application'] != application
    
    import_db_dump do
      db deploy['database']
      db_dump attributes['db_dump']
    end
  
  end
  
end
</code>
</pre>
<p>The check I&rsquo;m doing here (i.e. <code>if deploy['application_type'] != 'php' or or deploy['application'] != application</code>) is something I need in order to ensure that the recipe gets executed only for the current deployed application and not for the others allocated on the same OpsWorks stack. Remember, we&rsquo;re using OpsWorks to handle all the operations and their related configurations.<br /></p><p>The <code>import_db_dump</code> is just a <a href="http://docs.chef.io/definitions.html">definition</a> I&rsquo;ve written to add a little bit of magic and hide the details related to the import. Maybe one day I&rsquo;ll show you those details&hellip; ;P<br /></p><p>Now the important thing to remember about the recipe is that it resides inside a (general) application cookbook. I called it <code>phpapps</code>. So whenever I need to use the recipe inside a PHP layer defined inside an OpsWorks stack I must remember to specify the correct &ldquo;path&rdquo; to the recipe, in this case, <code>phpapps::set_db</code>.</p><p>This is really nothing special. The magic here (except from the one related to the <code>import_db_dump</code> definition ;) ) is that OpsWorks will handle for us all the information we need in our recipe by automatically merge them not only with all the attributes specified inside the JSONs (i.e. the stack and the customs one) but also with the defaults attributes written inside the recipe cookbook.</p><p>In this way you can rely on these attributes in order to specialize the recipes.</p><p>As stated before this isn&rsquo;t particularly the case as the recipe doesn&rsquo;t use any application specific information.<br /></p><p>Anyway in another general recipe I&rsquo;ve written I rely on this feature and it seems to work pretty well.</p><p>Next time I’ll maybe describe the particularities of this misterious recipe as it need some tweaks to function properly.</p><p>Moreover it strongly relies on the default attributes of the cookbooks. These elements are indeed from my point of view the real cornerstone to properly structure the cookbooks as wrapper and application specific ones.</p><p>To the next time! ;)</p><p>Cheers!</p>