---
ID: 211
post_title: Advanced filters in ActiveAdmin
author: Emanuele Serafini
post_date: 2014-09-10 16:39:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/09/10/advanced-filters-in-activeadmin/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/97146381079/advanced-filters-in-activeadmin
tumblr_mikamayhem_id:
  - "97146381079"
---
<h3>Use jQuery plugin Chosen for select boxes filters</h3>

<p>The <a href="http://harvesthq.github.io/chosen/">Chosen</a> plugin aims to improve select boxes by adding search-based filtering. When a user clicks into a Chosen element, their cursor is placed in a search field. Users can select their choice just the same as a standard select element.</p>

<p><img src="http://68.media.tumblr.com/e422c52d7f850f4fa0d4fa1905a828c3/tumblr_inline_nbp231zXNf1qfdn20.jpg" /></p>

<p>Integrate Chosen in active admin is pretty simple:</p>

<ul><li>Add <code>gem 'chosen-rails'</code> in your gemfile</li>
<li>Configure Chosen adding some options (in <code>app/assets/javascripts/active_admin/chosen.js.coffee</code>):</li>
</ul><pre><code>    $ -&gt;
        $('.chosen-filter').chosen
            allow_single_deselect: true
            width: '100%'
</code></pre>

<ul><li>Include Chosen javascript assets in <code>app/assets/javascripts/active_admin.js</code> adding <code>//= require chosen-jquery</code></li>
<li>Include Chosen stylesheet in <code>app/assets/stylesheets/active_admin.css.scss</code> adding <code>@import "chosen";</code></li>
</ul><p>Now you need to add the Chosen HTML class, in this case <code>.chosen-filter</code>, in formtastic select input.
Create a file <code>config/initializers/active_admin/chosen_filter.rb</code> and add:</p>

<pre><code>module ActiveAdmin
  module Inputs
    class Formtastic::Inputs::SelectInput
      include FilterBase

      def extra_input_html_options
        {
          class: 'chosen-filter'
        }
      end
    end
  end
end
</code></pre>

<h3>Chosen with multiple select</h3>

<p>Another option offered by Chosen is multiple select. User-selected options are displayed as boxes at the top of the element and are always visible. They can be removed with a single click or using backspace. Chosen has in-line search that allows you to navigate through associations so easily.</p>

<p><img src="http://68.media.tumblr.com/845ebf5683875b7771516b6c81958674/tumblr_inline_nbp23lFrlH1qfdn20.jpg" /></p>

<p>Create <code>config/initializers/active_admin/multiple_chosen_filter.rb</code> and add:</p>

<pre><code>module ActiveAdmin
  module Inputs
    class FilterMultipleSelectInInput &lt; Formtastic::Inputs::SelectInput
      include FilterBase

      def input_name
        "#{super}_in"
      end

      def extra_input_html_options
        {
          class: 'chosen-filter',
          multiple: true
        }
      end
    end
  end
end
</code></pre>

<p>Active Admin will pick the most relevant filter based on the attribute type. You can force the type by passing the :as option. In this case you need to use <code>multiple_select_in</code> to apply <code>FilterMultipleSelectInInput</code> class above.</p>

<p>Here&rsquo;s an example of filter definition:</p>

<pre><code>filter :country_id, label: 'Country', as: :multiple_select_in,  collection: -&gt; { Country.all }
</code></pre>

<p>Overriding <code>input_name</code> method we configure ransack predicate. In this case we search all selected elements. Predicates are used within Ransack search queries to determine what information to match.
Ransack predicates can be found <a href="https://github.com/activerecord-hackery/ransack/wiki/Basic-Searching">here</a>.</p>