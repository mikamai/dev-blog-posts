---
ID: 48
post_title: Testing with Espec and Elixir
author: Andrea Longhi
post_date: 2016-02-23 14:34:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/02/23/testing-with-espec-and-elixir/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/139850279124/testing-with-espec-and-elixir
tumblr_mikamayhem_id:
  - "139850279124"
---
<p>Every (new) language comes with a test library, and Elixir is no exception: the name of the built-in one is ExUnit and 
you can find the documentation <a href="http://elixir-lang.org/docs/stable/ex_unit/ExUnit.html">here</a>.</p>

<p>ExUnit is a good start: it is natively integrated in <code>mix</code> and has a few options to customize its output, 
still when I use it I feel its assertion syntax is a bit harsh. So I&rsquo;ve recently started using <a href="https://github.com/antonmi/espec">Espec</a>, 
a different testing library ispired by the awesome ruby <a href="https://github.com/rspec/rspec">Rspec</a> testing framework.</p>

<p>Unlike other alternatives this library is not built on ExUnit, so you can&rsquo;t mix the two assertion syntaxes for example, 
but on the other hand you get a more polished finish.</p>

<h3>How it works</h3>
<p>It&rsquo;s very easy to add Espec to your project; first create the project:</p>

 <pre><code>mix new espec_demo --module EspecDemo</code></pre>

 <p>then edit the <code>mix.exs</code> file as follows:</p>

<pre><code>
 def deps do
  [
    ...
    {:espec, "~&gt; 0.8.11", only: :test}
  ]
end

def project do
  [
    ...
    preferred_cli_env: [espec: :test]
  ]
end
</code></pre>

The shell command to run the specs is <code>MIX_ENV=test mix espec</code>, but thanks to the <code>preferred_cli_env: [espec: :test]</code> directive 
you can avoid prepending <code>MIX_ENV=test</code> everytime you run them. 


<p>Run <code>mix deps.get</code> in order to download the dependecies code then run <code>mix espec.init</code> in order to bootstrap Espec.<br />
This will generate a few files under the <code>spec</code> folder: <code>spec_helper.exs</code> which contains some configuration and 
<code>example_spec.exs</code>, which includes a brief example of what can be done with Espec:</p>

<pre><code>
defmodule ExampleSpec do
  use ESpec

  before do
    answer = Enum.reduce((1..9), &amp;(&amp;2 + &amp;1)) - 3
    {:shared, answer: answer} #saves {:key, :value} to `shared`
  end

  example "test" do
    expect shared.answer |&gt; to(eq 42)
  end

  context "Defines context" do
    subject(shared.answer)

    it do: is_expected.to be_between(41, 43)

    describe "is an alias for context" do
      before do
        value = shared.answer * 2
        {:shared, new_answer: value}
      end

      let :val, do: shared.new_answer

      it "checks val" do
        expect val |&gt; to(eq 84)
      end
    end
  end

  xcontext "xcontext skips examples." do
    xit "And xit also skips" do
      "skipped"
    end
  end

  pending "There are so many features to test!"
end
</code></pre>

<p>If you used Rspec with ruby you should feel at home with Espec: <code>before, subject, context, describe, let, pending, expect</code> are all there 
to make you feel comfortable, and if it wasn&rsquo;t for a few details you could think this was actually ruby code ;-)</p>