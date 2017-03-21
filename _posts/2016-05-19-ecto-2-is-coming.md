---
ID: 29
post_title: Ecto 2 is coming
author: Ivan Lasorsa
post_date: 2016-05-19 12:54:43
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/05/19/ecto-2-is-coming/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/144601142554/ecto-2-is-coming
tumblr_mikamayhem_id:
  - "144601142554"
---
<p>Some days ago Ecto version 2.0.0-rc.5 has been released. So Ecto 2 is coming and exploring how it works and its new features is a good idea .</p>

<p>First, from <a href="https://hex.pm/packages/ecto">hex.pm</a>, <i>Ecto is a domain specific language to write queries and interacting with databases in Elixir.</i></p>

<p>This version has four main components: <code>Ecto.Repo</code>, <code>Ecto.Schema</code>, <code>Ecto.Query</code>, <code>Ecto.Changeset</code>. Note here the absence of <code>Ecto.Model</code> that has been deprecated in favor of a more data-oriented approach.</p>

<p>Let&rsquo;s try it by creating a sample Elixir application.</p>

<p><code>mix new --sup my_shop</code></p>

<p>This command uses <code>mix</code> to create our application while the <code>--sup</code> option generates an OTP application skeleton that includes a supervision tree.</p>

<p>Now we are going to edit mix.exs file in order to include some dependencies at their latest versions: ecto and postgrex.</p>

<pre><code>def application do
    [applications: [:logger, :ecto, :postgrex],
     mod: {MyShop, []}]
end
  
defp deps do
    [
      {:ecto, "~&gt; 2.0.0-rc.5"},
      {:postgrex, "~&gt; 0.11.1"}
    ]
end
</code></pre>

<p>Run <code>mix deps.get</code> and we’re ready to define our repo.</p>

<pre><code>defmodule MyShop do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      supervisor(MyShop.Repo, [])
    ]
    opts = [strategy: :one_for_one, name: MyShop.Supervisor]
    Supervisor.start_link(children, opts)
  end
end

defmodule MyShop.Repo do
  use Ecto.Repo, otp_app: :my_shop
end
</code></pre>

<p>Note that we are defining our repo supervised by our app.
Repositories are the way you use to communicate with datastore, they  are wrappers around our databases and you can define as many as we need and configure them in <code>config/config.exs</code>. This is my configuration:</p>

<pre><code>use Mix.Config

config :my_shop,
  ecto_repos: [MyShop.Repo]

config :my_shop, MyShop.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: "postgres://my_shop_user:my_shop_password@localhost:5432/my_shop_dev"
</code></pre>

<p>Now we can run the specific mix task <code>mix ecto.create</code> and your database should be created.</p>

<p>We need some tables so let’s define a migration. In <code>priv/repo/migrations/20160516233500_create_tables.exs:</code></p>

<pre><code>defmodule MyShop.Repo.Migrations.CreateTables do
  use Ecto.Migration

  def change do
    create table(:products) do
      add :name, :string
      add :description, :text
      add :cost, :integer
    end

    create table(:colors) do
      add :code, :string
    end

    create table(:order_items) do
      add :product_id, references(:products)
      add :color_id, references(:colors)
      add :quantity, :integer
      add :cost, :integer
    end

    create table(:orders) do
      add :order_item_id, references(:order_items)
    end

    create table(:addresses) do
      add :country, :string
    end
  end
end
</code></pre>

<p>Run <code>mix ecto.migrate</code> and we’re done, we have five tables.</p>

<p>Now we are ready to use Ecto.Schema:</p>

<pre><code>defmodule Product do
  use Ecto.Schema

  schema "products" do
    field :name, :string
    field :description, :string
    field :cost, :integer
  end
end
</code></pre>

<p>Schemas are used to map any data source into an Elixir struct. Note that it’s not mandatory to use all the table fields, just those you need.</p>

<p>Now run <code>iex -S mix</code> in order to load your application into iex and verify if it works:</p>

<pre><code>iex(1)&gt; %Product{}
%Product{__meta__: #Ecto.Schema.Metadata&lt;:built&gt;, cost: nil, description: nil,
 id: nil, name: nil}
 
iex(2)&gt; %Unexistent{}
** (CompileError) iex:2: Unexistent.__struct__/0 is undefined, cannot expand struct Unexistent
    (elixir) src/elixir_map.erl:58: :elixir_map.translate_struct/4
</code></pre>

<p>Now, let’s use our repo to insert a record in our data store:</p>

<pre><code>MyShop.Repo.insert(%Product{name: "Programming Elixir"})

13:43:52.298 [debug] QUERY OK db=26.0ms
INSERT INTO "products" ("name") VALUES ($1) RETURNING "id" ["Programming Elixir"]
{:ok,
 %Product{__meta__: #Ecto.Schema.Metadata&lt;:loaded&gt;, cost: nil, description: nil,
  id: 1, name: "Programming Elixir"}}
</code></pre>

<p>Import Ecto.Query and retrieve all the products in our table:</p>

<pre><code>iex(4)&gt; import Ecto.Query
nil
iex(5)&gt; MyShop.Repo.all(from p in Product)

13:46:18.400 [debug] QUERY OK db=1.4ms
SELECT p0."id", p0."name", p0."description", p0."cost" FROM "products" AS p0 []
[%Product{__meta__: #Ecto.Schema.Metadata&lt;:loaded&gt;, cost: nil, description: nil,
  id: 1, name: "Programming Elixir"}]
</code></pre>

<p>It works :)</p>

<p>In my next post I’ll try to go deeper with more complex queries and introduce changesets.</p>

<p>If you are interested in this subject, Plataformatec will release an ebook about Ecto 2.0 written by José Valim, Elixir creator, you can <a href="http://pages.plataformatec.com.br/ebook-whats-new-in-ecto-2-0?utm_source=our-blog&amp;utm_medium=referral&amp;utm_campaign=ebook-ecto-2-0-pre-release&amp;utm_content=sidebar-cta">reserve a copy here</a>.</p>