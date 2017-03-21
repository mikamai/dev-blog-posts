---
ID: 53
post_title: Phoenix, to the basics and beyond.
author: Massimo Ronca
post_date: 2016-02-04 17:19:50
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/02/04/phoenix-to-the-basics-and-beyond/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/138675652509/phoenix-to-the-basics-and-beyond
tumblr_mikamayhem_id:
  - "138675652509"
---
<h2 id="toc_0">Introduction</h2>

<p>Phoenix is the exciting new kid on the block in the vast world of web frameworks.<br />
Its roots are in Rails, with the bonus of the performances of a compiled language.<br />
This isn&rsquo;t exactly a getting started guide, but a (albeit short) list of things you&rsquo;ll have to know very soon in the process of writing a Phoenix application, that are just a bit beyond the <q>writing a blog engine in 15 minutes by using only the default generators</q>.<br />
I assume previous knowledge of the Elixir language, the Phoenix framework and the command line tools</p>

<!--more-->

<div><pre><code class="elixir">mix ecto.create         # Create the storage for the repo
mix ecto.drop           # Drop the storage for the repo
mix ecto.gen.migration  # Generate a new migration for the repo
mix ecto.gen.repo       # Generate a new repository
mix ecto.migrate        # Run migrations up on a repo
mix ecto.rollback       # Rollback migrations from a repo
mix phoenix.digest      # Digests and compress static files
mix phoenix.gen.channel # Generates a Phoenix channel
mix phoenix.gen.html    # Generates controller, model and views for an HTML based resource
mix phoenix.gen.json    # Generates a controller and model for a JSON based resource
mix phoenix.gen.model   # Generates an Ecto model
mix phoenix.gen.secret  # Generates a secret
mix phoenix.new         # Create a new Phoenix v1.1.2 application
mix phoenix.routes      # Prints all routes
mix phoenix.server      # Starts applications and their servers
mix test                # Runs a project's tests
iex -S mix              # Starts IEx and run the default task</code></pre></div>

<blockquote>
<p>command line tools cheat sheet       </p>
</blockquote>

<h2 id="toc_1">First, a digression: development environments</h2>

<p>If you want to skip this section, <a href="#models">go directly where the action is</a></p>

<h3 id="toc_2">Emacs</h3>

<p>Emacs and <a href="http://www.alchemist-elixir.org/">Alchemist</a> are the most advanced programming environment for Elixir and Phoenix development.<br />
The features range from the simple syntax highlighting, to autocomplete, mix and iex integration, testing, compiling, running and evaluating code on the fly, everything you would expect from a modern IDE.<br />
Unfortunately (for me) I can&rsquo;t really Emacs, the fact that my favourite Emacs
distribution is <a href="http://spacemacs.org/">Spacemacs</a> says a lot about which one of
the two kind of people I am.<br />
Just for reference, if you are like me, <code>SPC m t a</code> launch <code>mix test</code> in a new buffer and rerun it every time you save a file.<br />
As simple as it might look, Alchemist is the only environment that offers this feature out of the box (everytime in Spacemacs you see <code>, somekey someotherkey</code>, that comma is translated to <code>m</code> in Spacemacs bindings).  </p>

<p><img src="http://i.imgur.com/zkhBOii.png" alt=", somekey someotherkey" /></p>

<blockquote>
<p>, somekey someotherkey</p>
</blockquote>

<p><strong>TL;DR</strong>: for hardcore Emacs users <a href="http://www.alchemist-elixir.org/">Alchemist</a> is a must have.  </p>

<h3 id="toc_3">Sublime/Textmate</h3>

<p>Sublime is my editor of choice on OS X and is the one I use to write Elixir too.<br />
There is a single bundle to be installed, it&rsquo;s called Elixir and it&rsquo;s the same for Sublime and Textmate.<br />
It provides syntax highlighting, autocomplete, snippets and a couple of commands (<code>Build with: elixir-mix-test</code> and <code>Build with: elixir</code>) to run tests and projects directly from the editor.<br />
I&rsquo;ve also installed <a href="https://sublimerepl.readthedocs.org/en/latest/">SublimeREPL</a> which currently supports Elixir and iex, so you can split the screen between and iex shell and a code editor pane.   </p>

<p><strong>TL;DR</strong>: All the usual goodies of Sublime applies.   </p>

<h3 id="toc_4">Atom</h3>

<p>Atom is great, but actually is not.<br />
This sentiment strucks me everytime I use it, something is awesome, something else is awful, there is no middle ground when it come to the Github editor.<br />
For example I find unacceptable that in 2016 an editor cannot soft wrap on column 78 by default and I&rsquo;ve found no package that does it.<br />
Coming from node, Atom inherits its problems, specifically the need to install many small packages to just get what you need.<br />
This time I installed <code>language elixir</code>, <code>autocomplete-elixir</code>, <code>elixir-cmd</code> (I&rsquo;m not very convinced it is useful at all) and <code>iex</code>. The last one is the best of them all, it is actually pretty great honestly, letting you run an iex console and tests inside it (by pressing <code>ALT+CMD+a</code>).   </p>

<p><strong>TL;DR</strong>: Sorry Atom, I&rsquo;ve tried, but you did not convince me.</p>

<h3 id="toc_5">Beauty contest</h3>

<p>I use Solarized Dark theme everywhere, from my editor, to the shell, I even used the Solarized palette in my personal website.<br />
I will share my opinions on the different syntax highlighting engines.  </p>

<p>IMHO the winner is SublimeText (and Textmate, the bundle is the same).<br />
The different elements are clearly highlighted and less prominent ones (like comments) are low contrast, you can almost <q>not see them</q> after a little bit of training.  </p>

<p><img src="http://i.imgur.com/kgCAyls.png" alt="SublimeText 3" /></p>

<blockquote>
<p>SublimeText 3</p>
</blockquote>

<p>Emacs comes second, we have a clear distinction here too, but the different colours are too soft/pastel and comments look too bright.  </p>

<p><img src="http://i.imgur.com/CSripmC.png" alt="Emacs" /></p>

<blockquote>
<p>Emacs</p>
</blockquote>

<p>Atom is the loser here.<br />
I like a lot the polished interface and the look of the theme (the best of the lot), but the syntax highlighting is actually pretty cheap.</p>

<p><img src="http://i.imgur.com/aL2Jmot.png" alt="Atom" /></p>

<blockquote>
<p>Atom</p>
</blockquote>

<p><a name="models"></a></p>

<h2 id="toc_6">Models</h2>

<p>Phoenix models are a very thin layer around <a href="https://hexdocs.pm/ecto/Ecto.Model.html"><code>Ecto models</code></a>.<br />
The <a href="https://hexdocs.pm/ecto/Ecto.Schema.html">schema dsl</a> is very lightweight making it super easy to model the data layer.   </p>

<blockquote>
<p>What a minimal model looks like</p>
</blockquote>

<div><pre><code class="elixir">defmodule App.Email do
  use App.Web, :model

  schema "emails" do
    field :address, :string
    timestamps # the ususal timestamps: created_at, updated_at
  end

end</code></pre></div>

<p>With <code>use App.Web, :model</code> you just import some packages defined in <code>web/web.ex</code> that you can customize.<br />
Specifically, by default Phoenix <code>use</code>s <code>Ecto.model</code> and <code>import</code>s from <code>Ecto.Changeset</code> and <code>Ecto.Query</code>.<br />
The difference between <code>use</code> and <code>import</code> is that <code>use</code> calls the <code>__using__</code> macro and inject code in the module that uses it, while <code>import</code> make functions in the imported module available to the caller and let you chose to import everything or just some of them.    </p>

<h3 id="toc_7">Custom primary keys</h3>

<p>The first “beyond the basics” we can do is to personalize the primary key of our model.<br />
By default they are autogenerated, autoincrementing <code>integers</code> and are called <code>id</code>.  </p>

<p>Suppose we want to use a <a href="https://en.wikipedia.org/wiki/Globally_unique_identifier">GUID</a>, how can we do that?  </p>

<p>It&rsquo;s pretty simple: first of all we have to tell the model that we don&rsquo;t want Phoenix to autogenerate the primary key.<br />
We do that at the <code>migration</code> level, where we also tell Phoenix to generate a field of type <code>:uuid</code>, <a href="https://github.com/phoenixframework/phoenix/blob/5f230cf52a5f601e4a1cd35dbb6a2d8ba8082728/lib/mix/tasks/phoenix.gen.model.ex#L209">it&rsquo;s a predefined type that maps to <code>Ecto.UUID</code></a></p>

<div><pre><code class="elixir">defmodule App.Repo.Migrations.CreateEmail do
  use Ecto.Migration

  def change do
    create table(:emails, primary_key: false) do
      add :key, :uuid, primary_key: true

      # ... rest of the table definition</code></pre></div>

<p>Then we instruct the model to use our new primary key</p>

<div><pre><code class="elixir">defmodule App.Email do
  use App.Web, :model

  # autogenerate GUIDs through the included GUID generator
  # the type is binary_id
  # by passing --binary-id to 'mix phoenix.new' 
  # binary_id is assumed as the default format for ids
  @primary_key {:key, :binary_id, autogenerate: true}

  # instruct Phoenix on how to extract the primary key for this model
  @derive {Phoenix.Param, key: :key}
</code></pre></div>

<p>That&rsquo;s all: we can now safely browse urls like <code>http://localhost/emails/1b72fe0b-5977-48bd-a19c-38140d1d1fed</code>.</p>

<h3 id="toc_8">Custom validators</h3>

<p>Validators in Phoenix are just functions that accept a changeset and optionally other parameters and return the same changeset or a new one filled with errors.<br /><a href="https://hexdocs.pm/ecto/Ecto.Changeset.html">Changeset</a> are the data structure Phoenix uses to keep track of the changes in the model and provide constraints and validations facilities.   </p>

<p>Changesets provide the following default validators </p>

<div><pre><code class="elixir">validate_change(changeset, field, validator)
##Validates the given field change
validate_confirmation(changeset, field, opts \ [])
##Validates that the given field matches the confirmation parameter of that field
validate_exclusion(changeset, field, data, opts \ [])
##Validates a change is not included in the given enumerable
validate_format(changeset, field, format, opts \ [])
##Validates a change has the given format
validate_inclusion(changeset, field, data, opts \ [])
##Validates a change is included in the given enumerable
validate_length(changeset, field, opts)
##Validates a change is a string or list of the given length
validate_number(changeset, field, opts)
##Validates the properties of a number
validate_subset(changeset, field, data, opts \ [])
##Validates a change, of type enum, is a subset of the given enumerable. 
##Like validate_inclusion/4 for lists</code></pre></div>

<blockquote>
<p>default validators</p>
</blockquote>

<p>On a first look, it seems like Ecto developers have been lazy and provided only a handful of validators.<br />
In reality it&rsquo;s incredibly easy to build custom validators, thank to the <code>|&gt;</code> operator we actually build validation pipelines.</p>

<p>Foe example this code validates an Email model by ensuring the presence of the fields name and address and enforcing the format for the address field.   </p>

<div><pre><code class="elixir">defmodule Wejustsend.Email do
  # all the usual imports and schema omitted  
  
  @doc """
  Creates a changeset based on the `model` and `params`.

  If no params are provided, an invalid changeset is returned
  with no validation performed.
  """
  def changeset(model, params \ :empty) do
    model
    # build a changeset from params
    # cast enforce the type of the fields and the presence of the required 
    |&gt; cast(params, @required_fields, @optional_fields) 
    |&gt; validate_name
    |&gt; validate_address
  end
  
  defp validate_name(changeset) do
    # name cannot be shorter than 4 characters
    changeset |&gt; validate_length(:name, min: 4)
  end
  
  defp validate_address(changeset) do
    changeset
    |&gt; validate_length(:address, min: 5) # a@a.a is 5 characters long
    |&gt; validate_address_list
  end

  defp validate_address_list(changeset) do
    # validate a list of one or more email addresses separated by ','
    # an email address is considered valid if it matches the regex
    # ^S+@S+.S+$
    changeset |&gt; to_address_list |&gt; Enum.reduce(changeset, fn(address, acc) -&gt;
      if address =~ address_format do
        # if matches we return the changeset as it is
        acc
      else
        # if it doesn't, we add the error to the changeset and return the new invalid changeset
        add_error(acc, :address, "#{address} is not a valid email address")
      end
    end)
  end

  defp to_address_list(changeset) do
    # if get_change returns the default value, it means that the change is empty
    # and we return the empty list
    # otherwise we split the string on ',' and strip the spaces around them
    case get_change(changeset, :address, []) do
      []      -&gt; []
      address -&gt; address |&gt; String.split(",") |&gt; Enum.map(&amp;String.strip/1)
    end
  end

  defp address_format do
    ~r/^S+@S+.S+$/
  end
end
</code></pre></div>

<blockquote>
<p>validations for an Email model </p>
</blockquote>

<p>It is a little verbose, honestly, but it is also very clear and required a basic knowledge of the framework to build it.<br />
Of course validation code, being just pure functions, can be generalized and moved somewhere else (a different Module for example) to be used in different models or in different applications.  </p>

<h2 id="toc_9">The Repository Pattern and the macros</h2>

<p>Phoenix does not supply any ORM or ORM like library for accesing data, instead it relies on the Repository Pattern (and the awesome <a href="http://hexdocs.pm/ecto/Ecto.Query.html"><code>Ecto.Query</code></a>), which, IMHO, is superior to Active Record any day of the week.<br />
Phoenix implemented it in a very generic way, using the facilities provided by <code>Ecto</code>:</p>

<div><pre><code class="elixir">
defmodule App.Repo do
  use Ecto.Repo, otp_app: :app, adapter: Sqlite.Ecto # default adapter is PostgreSQL
end

##then you can query the repository

Repo.all(Email)                                    # -&gt; all emails
Repo.all(User)                                     # -&gt; all users
Repo.get(User, 2)                                  # -&gt; user with id 2
Repo.get_by(User, {name: 'Rambo')                  # -&gt; first user whose name is Rambo
Repo.insert(%Email{address: 'marsrover@nasa.gov'}) # -&gt; insert a new email
</code></pre></div>

<p>However, Active Record is a very well established and recognized pattern in web applications, it is perfectly possible that its absence could let some developer down.<br />
Let&rsquo;s add something that resembles it to our models:</p>

<div><pre><code class="elixir">
defmodule App.Email do
  # all the usual imports and schema omitted

  alias App.Repo

  # returns all Emails
  def all(opts \ []) do
    Repo.all(Email, opts)
  end

  # find Emails by filtering out all emails not respecting clauses  
  # for example: {domain: 'email.com'}
  def find(clauses, opts \ []) do
    Enum.filter all(opts), fn map -&gt;
      Enum.all?(clauses, fn {key, val} -&gt; Map.get(map, key) == val end)
    end
  end

  # get the Email with id equal to id
  def get(id, opts \ []) do
    Repo.get(Email, id, opts)
  end

  # more functions
  # I suppose you get the pattern…
end

##now you can do this
Email.all
Email.find({domain: 'email.com'})
Email.get(1)
##etc. etc.
</code></pre></div>

<p>Elixir has a powerful mechanism to write generic code, <a href="http://elixir-lang.org/getting-started/meta/macros.html">macros</a>.<br />
Phoenix is an Elixir app after all, so we can use macros to write a generic <code>ActiveRecord</code> like wrapper for the <code>Repo</code> and inject it into the <code>Model</code> code at compile time (yes, at compile time, it will be like we actually wrote that code with no loss of performance whatsoever).  </p>

<div><pre><code class="elixir">defmodule App.Activerecord do
  # when we "use" a Module, Elixir calls the relative __using__ macro inside it  
  defmacro __using__(_) do
    quote do
      alias App.Repo

      # __MODULE__ is resolved at compile time with the name
      # of the caller, inside "quote do" we are in the caller context
      def all(opts \ []) do
        Repo.all __MODULE__, opts
      end

      def find(clauses, opts \ []) do
        Enum.filter all(opts), fn map -&gt;
          Enum.all?(clauses, fn {key, val} -&gt; Map.get(map, key) == val end)
        end
      end

      def get(id, opts \ []) do
        Repo.get __MODULE__, id, opts
      end

      def get_by(clauses, opts \ []) do
        Repo.get_by __MODULE__, clauses, opts
      end

    end

  end

end

##and then do

defmodule App.Email do
  # all the usual imports and schema omitted
  use App.Activerecord # &lt;- this inject the code into the Email module
end

##now you can do this
Email.all
Email.find({domain: 'email.com'})
Email.get(1)

##but also this, if you use the ActiveRecord module in the User model
User.all
User.find({role: 'admin'})
User.get(1)
</code></pre></div>

<p>I don&rsquo;t know if this is a good idea, but it is very easy to implement and the mechanism is so powerful that opens up the way for writing well organized and understandable code.   </p>

<p>Next time we&rsquo;ll talk about the <a href="http://dev.mikamai.com/post/143587193369/phoenix-framework-to-the-basics-and-beyond-the">assets pipeline</a> and long running processes.</p>