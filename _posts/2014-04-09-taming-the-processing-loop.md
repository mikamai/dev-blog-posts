---
ID: 261
post_title: Taming the Processing loop
author: Massimo Ronca
post_date: 2014-04-09 08:47:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/04/09/taming-the-processing-loop/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/82178345433/taming-the-processing-loop
tumblr_mikamayhem_id:
  - "82178345433"
---
<p>In Mikamai we do a lot of reasearch on <a href="http://dev.mikamai.com/post/78652180658/how-to-program-an-attiny85-or-attiny45-with-an">non</a> <a href="http://dev.mikamai.com/post/78453410376/let-your-raspberry-pi-see-this-wonderful-world">conventional</a> <a href="http://dev.mikamai.com/post/69163914657/intel-galileo-getting-started-with-mac-os-x">hardware</a>, we make <a href="http://dev.mikamai.com/post/76945627390/you-cant-touch-this-an-evil-arduino-based-alarm">prototypes</a> and create unusual interfaces that are very domain specific.</p>
<p>Like this one</p>
<p><img alt="image" src="https://scontent-b-ams.xx.fbcdn.net/hphotos-ash3/t1.0-9/994503_10151525258526336_667825845_n.jpg" /></p>
<p>Seriously, we did it.</p>
<p>To quickly sketch ideas, we often rely on <a href="http://www.processing.org/">Processing</a>, it&rsquo;s super easy and its loop based execution model gives the feeling of programming a video game.<br /> The drawback is that it is so fast to get something working, that you will be tempted to make the mistake of creating a <a href="http://foxdellfolio.com/the-perils-of-a-polished-prototype/">polished prototpe</a>.<br /> Your prototype code ends up in production and there&rsquo;s no way back from there.</p>
<p>To resist the temptation of releasing a blob of code, I borrowed <a href="https://www.youtube.com/watch?v=HxaD_trXwRE">a technique from one of the Rob Pike&rsquo;s talks</a> to keep things easy, while keeping them clean at the same time.</p>
<p>It is basically an implementation of a state machime.<br /> We&rsquo;re gonna have a <code>StateMachine</code> class that handles the inputs and the state changes, and several state classes that implement the <code>State</code> interface.<br /> The interface is very simple and contains only one method</p>

<!--more-->

<pre><code>interface State {
      public State nextState();  
}
</code></pre>
<p>The loop of our Processing application is really simple too</p>
<pre><code>StateMachine sm = new StateMachine(initialstate);
void draw() {
  sm = sm.nextState();  
}
</code></pre>
<p>and this is the most basic implementation possible of the <code>StateMachine</code> class</p>
<pre><code>class StateMachine(State initialstate) {
  private State currentstate;

  StateMachine(State initialstate) {
    this.currentstate = initialstate;
  }

  public StateMachine nextState() {
    this.currentstate = this.currentstate.nextState();
    return this; 
  }
}
</code></pre>
<p>Each class must implement the <code>nextState</code> method and return an istance of the next state that will be executed.<br /> With this knowledge in mind, this is how you build an infinite loop inside an inifinite loop</p>
<pre><code>class InfiniteLoopState implements State {
    public State nextState() {
        return this;
    }
}
</code></pre>
<p>But we can do better!<br /> How about a ping pong?</p>
<pre><code>class Ping implements State {
    public State nextState() {
        println("ping?");
        return new PongState();
    }
}

class Pong implements State {
    public State nextState() {
        println("pong!");
        return new PingState();
    }
}
</code></pre>
<p>We moved the logic of the application out of the central <code>switch/case</code> statement in the <code>draw</code> function and deconstruted it into smaller pieces, that only know about themselves and the next state they are going to emit.</p>
<p>As long as your state classes implement the <code>State</code> interface you can exapnd the concept to fit your needs.<br /> For example, if you need to keep track of the environment and/or the previous state, you can adjust the <code>State</code> interface to support it.</p>
<pre><code>interface State {
      public State nextState(State previousstate, StateMachine sm);  
      // StateMachine holds the environment for us
}
</code></pre>
<p>and modify <code>StateMachine</code> accordingly</p>
<pre><code>  public StateMachine nextState() {
    this.currentstate = this.currentstate.nextState(this.currentstate, this);
    return this; 
  }
</code></pre>
<p><strong>TL;DR:</strong> state machines are easy, use them!</p>
<p>You can find examples on how to use this pattern and how to add more features in the <a href="https://github.com/wstucco/processing_state_machine">github repository</a>.</p>