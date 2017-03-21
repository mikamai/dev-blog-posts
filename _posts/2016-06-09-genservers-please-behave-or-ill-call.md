---
ID: 26
post_title: >
  GenServers please behave, or I’ll call
  the Supervisor
author: Andrea Longhi
post_date: 2016-06-09 12:50:49
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/06/09/genservers-please-behave-or-ill-call/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/145656848024/genservers-please-behave-or-ill-call
tumblr_mikamayhem_id:
  - "145656848024"
---
<p>In my previous <a href="http://dev.mikamai.com/post/141087465149/elixir-an-introduction-to-server-architecture-and">post</a> I explored the basics of Elixir GenServer behaviour, it&rsquo;s now time to jump on the next big topic: <strong>Supervisors</strong>.</p>

<!--more--> 

<p>If GenServers are the building blocks in the OTP world, Supervisors are the glue that keeps them togheter.</p>

<p>First, let&rsquo;s get a definition of supervisors:<br /><cite>A supervisor is a process which supervises other processes, called child processes. Supervisors are used to build a hierarchical process structure called a supervision tree, a nice way to structure fault-tolerant applications.</cite>

</p><p>And that&rsquo;s all there is to it! Supervisors are used to spawn processes (tipically GenServers), to manage them, to restart them when they crash, to handle errors.</p>

<p>Supervisor is another OTP behaviour, just like GenServer, in fact it&rsquo;s built on top of it. Why do we need supervisors in the first place? Because processes can crash for any reason, and in the concurrent world it&rsquo;s better to handle failure and recover from it rather than let the whole application crash.</p><p>Try this, start your iex session and create a <em>linked</em> process that keeps waiting indefinitely. We then ask if the process is alive, and check the pid number of the current iex main process:</p><pre><code>
pid = spawn_link fn -&gt; receive do end end
#PID&lt;0.59.0&gt;

Process.alive? pid
true

current = self
#PID&lt;0.68.0&gt;
</code></pre>
<p>Now, let&rsquo;s see what happens when we kill the process:</p><pre><code>
Process.exit pid, :kill
** (EXIT from #PID&lt;0.68.0&gt;) killed

self
#PID&lt;0.74.0&gt;
</code></pre>
<p>See how the pid of the iex process has changed. What happened? Killing the linked process took down the iex main process as well (which was actually restarted behind the scenes, but you can assume that if the iex process were your application, it would be dead right now).</p>
<p>It&rsquo;s now time to build our first supervisor. I promise it&rsquo;s going to be easy. Let&rsquo;s go barebone first:
</p><pre><code>
defmodule A.Supervisor do
  use Supervisor

  def start_link do
    Supervisor.start_link __MODULE__, :ok
  end

  def init _ do
    children = []
    supervise children, strategy: :one_for_all
  end
end

{:ok, sup} = A.Supervisor.start_link
{:ok, #PID&lt;0.103.0&gt;}

Supervisor.count_children sup
%{active: 0, specs: 0, supervisors: 0, workers: 0}
</code></pre>
<p>Just like with GenServer, we define a <code>start_link</code> function that eventually calls the <code>init</code> function. The init function handles all the gory details of the supervision, right now we want to start simple so there is nothing to supervise yet, so the function looks quite empty (<code>children</code> is an empty list).</p>
<p>The function <code>Supervisor.count_children</code> confirms that this supervisor is not very busy at the moment, it is supervising nothing: no <code>worker</code> (that&rsquo;s what generic supervised processes are called) and no <code>supervisor</code> (because supervisors can supervise other supervisors as well) yet.</p><p>Still, we already decided a very important behavior of this supervisor: its children are supervised with the <code>one_for_all</code> strategy, which means that, if one of its child processes dies, then all the others will die as well. But all of them will be restarted soon enough.</p><p>Let&rsquo;s define a simple GenServer:</p>
<pre><code>
defmodule A.Worker do
  use GenServer

  def start_link value \ "" do
    GenServer.start_link __MODULE__, value
  end

  def state pid do
    GenServer.call pid, :state
  end

  def handle_call :state, _, state do
    {:reply, state, state}
  end
end
</code></pre>
<p>It&rsquo;s very basic, we can start it with an explicit state if we&rsquo;re not happy with the default empty string and query for its state:
</p><pre><code>
{:ok, pid} = A.Worker.start_link "my value"
{:ok, #PID&lt;0.125.0&gt;}

A.Worker.state pid
"my value"
</code></pre>
<p>That&rsquo;s it. In order to add a A.Worker child to the supervisor we need to build a <em>worker specification</em>, which is basically a tuple of tuples that contains the &ldquo;recipe&rdquo; for spawning the child. Luckily enough, <code>Supervisor.Spec</code> knows how to build this contrieved tuple:</p>
<pre><code>
import Supervisor.Spec

spec = worker(A.Worker, ["first value"])
{A.Worker, {A.Worker, :start_link, []}, :permanent, 5000, :worker, [A.Worker]}
</code></pre>
<p>At a minimum the required ingredients for the &ldquo;recipe&rdquo; are the module name (<code>A.Worker</code>) and the arguments list (<code>"first value"</code>).<br />The most notable information included in the tuple are: 
</p><ul><li><code>:start_link</code>: the function of the module A.Worker that will be called for bootstrapping the worker</li>
<li><code>:permanent</code>: means that if the child dies, it will be respawned (the other option <code>:temporary</code> won&rsquo;t restart the process)</li>
<li><code>5000</code>: milliseconds that must pass between one restart and the other</li>
</ul><p>
You can find a more detailed explanation <a href="http://elixir-lang.org/docs/stable/elixir/Supervisor.Spec.html">here</a>.<br />With our specification we&rsquo;re now ready to create our child:</p>
<pre><code>
{:ok, worker} = Supervisor.start_child sup, spec
{:ok, #PID&lt;0.133.0&gt;}

Supervisor.count_children sup
%{active: 1, specs: 1, supervisors: 0, workers: 1}
Supervisor.which_children sup                        
[{A.Worker, #PID&lt;0.133.0&gt;, :worker, [A.Worker]}]
</code></pre>
<p>We can see now that 1 worker is reported and active.</p>
<p>It&rsquo;s time to see our supervisor do its job, I mean restart the supervised process, when it dies:</p>
<pre><code>
Process.exit worker, :kill
true

Process.alive? worker
false

Supervisor.count_children sup
%{active: 1, specs: 1, supervisors: 0, workers: 1}
Supervisor.which_children sup
[{A.Worker, #PID&lt;0.146.0&gt;, :worker, [A.Worker]}]
</code></pre>
<p>As you can see, after killing the worker process the supervisor restarted another worker to take its place, this is confirmed by the different pid number (146 instead of the previous 133).</p>
<p>What happens if we try to add a new worker with the same specification? We get an error, because we can&rsquo;t reuse it with the strategy we chose. We can work around this by supplying a different <code>:id</code> identifier (or a different :name):</p>
<pre><code>
{:ok, worker} = Supervisor.start_child sup, spec
** (MatchError) no match of right hand side value: {:error, {:already_started, #PID&lt;0.146.0&gt;}}

spec = worker(A.Worker, [], id: :extra_worker)
{:extra_worker, {A.Worker, :start_link, []}, :permanent, 5000, :worker, [A.Worker]}
{:ok, worker} = Supervisor.start_child sup, spec
{:ok, #PID&lt;0.153.0&gt;}

Supervisor.count_children sup                   
%{active: 2, specs: 2, supervisors: 0, workers: 2}
</code></pre>

<p>There is a more proper way when we need to supervise multiple processes from the same module. Let&rsquo;s creare a new supervisor module that can handle transparently all the A.Worker processes we want:</p>
<pre><code>
defmodule A.WorkerSupervisor do
  use Supervisor

  def start_link do
    Supervisor.start_link __MODULE__, :ok
  end

  def init _ do
    children = [worker(A.Worker, [])]
    supervise children, strategy: :simple_one_for_one
  end
end

{:ok, w_sup} = A.WorkerSupervisor.start_link
{:ok, pid} = Supervisor.start_child w_sup, ["value"]

A.Worker.state pid
"value"

Supervisor.start_child w_sup, []
Supervisor.start_child w_sup, []

Supervisor.count_children w_sup
%{active: 3, specs: 1, supervisors: 0, workers: 3}
</code></pre>
<p>This time the supevisor defines upfront the <code>worker</code> specification inside the <code>init</code> function. The strategy we choose, <code>:simple_one_for_one</code>, allows us to create as many children as we want for this supervisor, that&rsquo;s because this strategy assumes we&rsquo;re going to use always the very same specification. And by  the way, this looks very much like a Factory object in OOP languages.</p><p>In the code above, after creating the supervisor, we add one child with a given value, then we check for its value to be there. A few lines later we add 2 more children, and we verify via <code>count_children</code> that 3 active workers exist and they&rsquo;re all with the same specification (<code>specs: 1</code>).</p>
<p>Let&rsquo;s go back to our first supervisor, the one from the A.Supervisor module. It currently has 2 children from the same module:</p>
<pre><code>
Supervisor.count_children sup
 [{:extra_worker, #PID&lt;0.153.0&gt;, :worker, [A.Worker]},
 {A.Worker, #PID&lt;0.146.0&gt;, :worker, [A.Worker]}]
</code></pre>
<p>We said we could supervise supervisors as well, thus creating a <em>supervision tree</em>, so let&rsquo;s do it:</p>
<pre><code>
Process.exit w_sup, :normal

spec = Supervisor.Spec.supervisor A.WorkerSupervisor, []
{A.WorkerSupervisor, {A.WorkerSupervisor, :start_link, []}, :permanent,
 :infinity, :supervisor, [A.WorkerSupervisor]}
 
{:ok, w_sup} = Supervisor.start_child sup, spec
{:ok, #PID&lt;0.177.0&gt;}

Supervisor.count_children sup
%{active: 3, specs: 3, supervisors: 1, workers: 2}
</code></pre>
<p>Just like before, we created a new child for our A.Supervisor process, but this time we used the <code>supervisor</code> specification instead of the previous <code>worker</code>, of course that&rsquo;s because this time we needed to create a child of type supervisor.</p><p><code>count_children</code> confirms that we now have 3 active supervised processes, and one is a <em>supervisor</em> type.</p><p>Of course we can attach as many workers as we want to our w_sup supervisor:</p><pre><code>
for i &lt;- 0..10 do
  Supervisor.start_child w_sup, [i]
end

Supervisor.count_children w_sup
%{active: 11, specs: 1, supervisors: 0, workers: 11}
</code></pre>
<p>And that&rsquo;s all for now, I hope you enjoyed this introduction to supervisors!</p>