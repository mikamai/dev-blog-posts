---
ID: 232
post_title: '[RubyJuice] Public, private, protected methods in ruby'
author: Andrea Longhi
post_date: 2014-06-30 16:31:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/06/30/rubyjuice-public-private-protected-methods-in/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/90363293164/rubyjuice-public-private-protected-methods-in
tumblr_mikamayhem_id:
  - "90363293164"
---
<p>Ruby newbies often get confused when they learn about method access levels. Other languages use the same denomination (notably PHP and Java) but with slightly different meaning, so let&rsquo;s clarify the use of <code>public</code>, <code>private</code> and <code>protected</code> with some examples.</p>
<h4>Public</h4>
<p>Let&rsquo;s start from the basics: by default methods are defined as public, so you don&rsquo;t need to explicitly type that keyword inside your class:</p>
<pre><code>class SshKeyPair
  def id_rsa_pub
    'I am the public key'
  end
end

pair = SshKeyPair.new
c.id_rsa_pub
# =&gt; "I am the public key"</code></pre>
<p>Let&rsquo;s use ruby introspection to confirm that the method has been added to the object methods list:</p>
<pre><code>pair.methods.include? :id_rsa_pub
# =&gt; true

pair.public_methods.include? :id_rsa_pub
# =&gt; true</code></pre>
<h4>Private</h4>
<p>Private methods can be accessed only without receiver, practically this means that only the very same instance can access them, and <strong>only</strong> when the self receiver is <strong>omitted</strong>:</p>
<pre><code>class SshKeyPair
  private

  def id_rsa
    'I am the private key'
  end
end

pair.id_rsa
# NoMethodError: private method `id_rsa' called for #&lt;SshKeyPair:0x007f87f139be28&gt;

pair.methods.include? :id_rsa
# =&gt; false
pair.private_methods.include? :id_rsa
# =&gt; true
pair.protected_methods.include? :id_rsa
# =&gt; false</code></pre>
<p>Of course you can always use <code>send</code> to circumvent restrictions (but remember that probably that method was declared private for good reason):</p>
<pre><code>pair.send :id_rsa
# =&gt; "I am the private key"
</code></pre>
<p>Now let&rsquo;s see the different behavior of the method when called with and without <code>self</code> directly inside the class:</p>
<pre><code>class SshKeyPair
  def leak_private_key
    id_rsa
  end

  def lousy_leak_private_key
    self.id_rsa
  end
end

pair.leak_private_ley
# =&gt; "hello, privately"

pair.lousy_leak_private_key
NoMethodError: private method `id_rsa' called for #&lt;SshKeyPair:0x007f87f139be28&gt;</code></pre>
<p>There&rsquo;s one exception: writer methods. When declared private, they still can be used inside the class using self:</p>
<pre><code>class Album
  def set_name(name)
    self.name = name
  end

  private

  attr_accessor :name
end

album = Album.new
# =&gt; #&lt;Album:0x007fb439125328&gt;
album.set_name 'Pet Sounds'
# =&gt; "Pet Sounds"
album.name
NoMethodError: private method `name' called for #&lt;Album:0x007fb439125328 @name="Pet Sounds"&gt;
album.send :name
# =&gt; "Pet Sounds"
album.name = 'Smile'
NoMethodError: private method `name=' called for #&lt;Album:0x007fb439125328 @name="Pet Sounds"&gt;</code></pre>
<p>I think this happens because there is now way to use a writer method without making <code>self</code> explicit.</p>
<h4>Protected</h4>
<p><code>protected</code> is probably the most misunderstood in the group: it works like <code>private</code>, but the method is visible also by instances of the same class:</p>
<pre><code>class Woman
  attr_accessor :age

  protected :age
end

kate = Woman.new
kate.age
# NoMethodError: protected method `age' called for #&lt;Woman:0x007fb439112340&gt;

kate.protected_methods.include? :age
# =&gt; true
kate.private_methods.include? :age
# =&gt; false
kate.methods.include? :age
# =&gt; true</code></pre>
<p>This makes sense when instance should be allowed to interact with each other while not exposing their behaviour publicly. Let&rsquo;s enhance our Woman class so that it allows age comparisons keeping the age secret:</p>
<pre><code>
class Woman
  def older?(other)
    (age &lt;=&gt; other.age) == 1
  end
end

mother       = Woman.new
daughter     = Woman.new
mother.age   = 35
daughter.age = 10

mother.older? daughter
# =&gt; true</code></pre>
<p>But what happens with subclasses? Will it still work? Of course it does:</p>
<pre><code>class Girl &lt; Woman
  def age=(value)
    raise 'Girls older than 20 years are women!' if value &gt; 20
    @age = value
  end
end

daughter = Girl.new
daughter.age = 10

daughter.older? mother
# =&gt; false</code></pre>
<p>Statements can be mixed and repeated freely to suit your tastes:</p>
<pre><code>class C
  private
    def a; end

  public
    def b; end

  protected
    def c; end

  private
    def d; end
  end
end</code></pre>
<p>or you can change the method visibility on second thought:</p>
<pre><code>class C
  def a; end
end

class C
  private :a
end</code></pre>
<h4>Ruby 2.1 additions</h4>
<p>The recently released Ruby 2.1 introduced a few changes that affect method access level keywords: basically, they&rsquo;re no more simple keywords but full-fledged methods, so they can be redefined:</p>
<pre>class C
  def self.private(*m_names)
    m_names.each {|m_name| puts "#{m_name} is now a private method."}
  end
end

class C
  def a; end

  private :a
end
# a is now a private method.</pre>
<p>The second change is that method definitions with <code>def</code> now return the method name symbol, so we can write:</p>
<pre>class C
  private def b; end
end
# b is now a private method.</pre>