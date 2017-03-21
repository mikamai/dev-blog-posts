---
ID: 134
post_title: 'DIY: Simple Ruby Tokenizer and Parser that recognize Polish notation'
author: Diomede Tripicchio
post_date: 2015-04-17 12:38:40
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/04/17/diy-simple-ruby-tokenizer-and-parser-that/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/116638557159/diy-simple-ruby-tokenizer-and-parser-that
tumblr_mikamayhem_id:
  - "116638557159"
---
<p>These days I had to create a parser that recognized a particular type of string.</p>

<p>With this article I’ll propose you a simple approach to write a simple Tokenizer and Parser, in Ruby, that recognize the <a href="http://en.wikipedia.org/wiki/Polish_notation">Polish notation</a> </p>
<!--more-->

<p>We want to obtain a result from a string like this:</p>

<pre><code>+ 2 2</code></pre>

<p>obviosly <strong>4</strong></p>

<p>In order to do this we need to split the string in “tokens” that we will be able to use.</p>

<p>After the work of the <strong>Tokenizer</strong> we will have a result like this:</p>

<pre><code>[ &lt;Token @type=:operator, @value=‘+’, &lt;Token @type=:number, @value=2.0&gt;, &lt;Token @type=:number, @value=2.0&gt; ]
</code></pre>

<p>With this array of <strong>tokens</strong>, we can loop and choose which action to do for every single element based on its type.</p>

<p>First of all let’s create our Token class. Our string will be splitted in tokens, each one will be an instance of this class.</p>

<p><strong>token.rb</strong></p>

<pre><code class="ruby">
class Token
  attr_reader :type, :value

  def initialize type, value
    @type, @value = type, value
  end
end
</code></pre>

<p>Now we can create the <strong>Tokenizer</strong> class that will split the string.</p>

<p><strong>tokenizer.rb</strong></p>

<pre><code class="ruby">
class Tokenizer
  OPERATORS = ['+', '-', '*', '/']
  NUMBER_REGEXP = /^d*(.d+)?$/

  attr_reader :tokens, :cursor

  def initialize expression
    tokenize expression
    @cursor = -1
  end

  def current_token
    return if cursor == -1
    tokens[cursor]
  end

  def next_token
    cursor += 1
    current_token
  end

  def look_next_token
    tokens[cursor + 1]
  end

  def operator_included?
    tokens.map(&amp;:type).include? :operator
  end

  private

  def tokenize expression
    @tokens = []
    expression_to_parse = expression.strip
    while expression_to_parse.size &gt; 0
      tokens &lt;&lt; read_next_token(expression_to_parse)
    end
  end

  def read_next_token expression_to_parse
    expression_to_parse.strip!
    next_char = expression_to_parse.slice 0
    if OPERATORS.include? next_char
      read_next_operator_token expression_to_parse
    elsif next_char =~ NUMBER_REGEXP
      read_next_numeric_token expression_to_parse
    else
      raise "Unknown token starting with '#{next_char}'"
    end
  end

  def read_next_operator_token expression_to_parse
    Token.new :operator, expression_to_parse.slice!(0)
  end

  def read_next_numeric_token expression_to_parse
    numeric_value = expression_to_parse.slice! 0
    while next_char = expression_to_parse.slice(0) &amp;&amp; "#{numeric_value}#{next_char}" =~ NUMBER_REGEXP
      numeric_value &lt;&lt; expression_to_parse.slice!(0)
    end
    Token.new :number, numeric_value.to_f
  end
end
</code></pre>

<p>This class takes the string and loops through every single char and tries to recognize them. Every time a char is analyzed it is removed from the main string. In this way we avoid the possibility of errors during the tokenizer process.</p>

<p>Pay attetion to <strong>read_next_numeric_token</strong>: this method will continue the loop on <em>expression_to_parse</em> until the char already parsed joined with next character match the <em>NUMBER_REGEXP.</em></p>

<p>Once the expression has been tokenized we are ready to parse the result. Now the code of the <strong>Parser</strong> class:</p>

<p><strong>parser.rb</strong></p>

<pre><code class="ruby">
class Parser

  attr_reader :tokenizer

  # Parse a polish notation expression and return the result
  def parse expression
    @tokenizer = Tokenizer.new expression

    raise 'Expression must start with an operator' if tokenizer.look_next_token.type == :number

    operate
  end

  def tokens
    tokenizer.tokens
  end

  def operate
    number_stack = []
    # continue while the last operator are popped from tokens
    while tokenizer.operator_included?
      current_token = tokens.pop
      if current_token.type == :number
        # push the number in the number_stack
        number_stack &lt;&lt; current_token.value
      elsif current_token.type == :operator
        raise "Not enough operands" if number_stack.size &lt; 2
        # take the last 2 number added to the stack
        first_value, second_value = number_stack.pop(2)
        # Add a new :number Token with the result of th operation
        tokens &lt;&lt; Token.new( :number, first_value.send( current_token.value, second_value ) )
      end
    end
    raise "Not enough operators" if number_stack.size &gt; 0

    tokens.first.value
  end
end
</code></pre>

<p>Following the Polish Notation&rsquo;s rules, the Parser loops through the array of tokens starting from the last element. When a <em>:number</em> is found it will be pushed in the <em>number_stack</em>, whereas when we found an <em>:operator</em> we take (<strong>pop</strong>) the two last elements from the <em>number_stack</em> and then we send the operation (defined by the <em>:operator</em> Token), positioning it in the middle between the two numbers. Once we have the result, we can push it as a new <em>:number</em> Token in the tokens array.</p>

<p>In this specific case the loop through the tokens continues until there are no more <em>:operator</em></p>

<p>In the end we get an array with a single element that contains the final result.</p>

<p>I hope you liked the post, this is my first post on this blog!</p>

<p>I would like to thank <a href="https://github.com/nicolaracco" target="_blank">Nicola</a> for most of the code and the theory behind it.</p>

<p>Thanks for reading and see you soon!</p>