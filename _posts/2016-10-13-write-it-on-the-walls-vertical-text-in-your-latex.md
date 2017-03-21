---
ID: 12
post_title: >
  Write it on the walls! Vertical text in
  your LaTeX Beamer presentations
author: Cecilia Nardini
post_date: 2016-10-13 08:11:13
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/10/13/write-it-on-the-walls-vertical-text-in-your-latex/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/151742403999/write-it-on-the-walls-vertical-text-in-your-latex
tumblr_mikamayhem_id:
  - "151742403999"
---
Ok, so probably you have heard about <a href="https://www.latex-project.org/">LaTeX</a> before. Possibly you have used it for writing your neatly typeset, formula-crowded thesis. Eventually you may have heard about <a href="https://www.ctan.org/pkg/beamer?lang=en">Beamer</a> too, the LaTeX package for presentations.

I take it as a near-impossibility that you have actually <i>used</i> Beamer for creating a presentation, what with Office-like products being so easy and convenient, or Prezi being so astonishing and cool. But keep reading! For by the end of this article you will be wondering why you haven’t tried it <i>yet</i>.

<!--more-->

Contrary to LaTeX, which is extremely well documented and easy to find help about, Beamer is much more obscure and can be at times thoroughly frustrating, since doing things that should be easy-peasy such as moving an image a teensy bit to the right can occasionally turn out an impossible hurdle.

Nonetheless, Beamer has two unparalleled strenghts. The first is, obviously, enabling you to leverage the terrific power of LaTeX for delivering neat, aesthetically pleasing presentations. The second strenght lies with a further TeX library, <a href="https://www.ctan.org/pkg/pgf?lang=en">TikZ</a>. TikZ/pgf is, strictly speaking, a graphic library, and therefore can be used for creating all sorts of complex scientific plots. Used inside Beamer, TikZ allows you to create all sorts of drawings, diagrams and graphics. Combine it with Beamer’s overlay feature - the possibility of piecewise uncovering text and other elements in the slide - and you have the possibility of creating completely custom <i>animations</i> for your slides. Furthermore, since you are writing the instructions for the graphic library directly inside the LaTeX source file, these are animations that you can <i>code yourself</i>. Cool, isn’t it?

But enough with the talk. I have prepared a short example to give you a taste of working with Beamer/TikZ and of the nice tricks you can figure out. Our problem will be to write words in an acronym. The acronym is written normally (horizontal, left-to-right), but we want to have the defining words written <i>vertically</i>, one letter below the other.
The acronym we will be using is the Agile mnemonic for the characteristics of a good User Story, <a href="https://en.wikipedia.org/wiki/INVEST_(mnemonic)">INVEST</a> (Independent, Negotiable, Valuable, Estimable, Small, Testable).

We start with a plain <code>.tex</code> file, all the dependencies we need are taken care of by specifying that the <code>documentclass</code> is <code>beamer</code>; we only need to explicitely include TikZ with <code>usepackage{tikz}</code>.
Inside the LaTeX <code>document</code> we create a <code>frame</code> which will host our <code>tikzpicture</code>, wrapped inside a <code>figure</code> for proper positioning.

Now we will write the acronym by putting each letter inside a TikZ <code>node</code>. We will exploit the possibility of referring to a node that has been already drawn using its identifier, and the possibility of positioning TikZ elements <i>relative</i> to one another using commands such as <code>right of</code> and <code>below of</code>. The resulting code is really pretty obvious:
<pre><code class="tex">
documentclass[xcolor=dvipsnames]{beamer}
usepackage{tikz}

begin{document}

begin{frame}{}
  begin{figure}
    begin{tikzpicture}[node distance=1em] %here we specify the distance between nodes for the whole picture
      node [SeaGreen!80!black] (I) at (0,0) {Large{I}};
      begin{scope}[node distance=3em] %here we define a sub-scope with a different value of the distance
      foreach x [remember=x as lastx (initially I)] in {N,V,E,S,T}
        node (x) [right of=lastx,SeaGreen!80!black] {Large{x}};
      end{scope}
      foreach x [remember=x as lastx (initially I)] in {n,d,e,p,e,n,d,e,n,t}
        node (x) [below of=lastx] {x};
      foreach x [remember=x as lastx (initially N)] in {e,g,o,t,i,a,b,l,e}
        node (x) [below of=lastx] {x};
      foreach x [remember=x as lastx (initially V)] in {a,l,u,a,b,l,e}
        node (x) [below of=lastx] {x};
      foreach x [remember=x as lastx (initially E)] in {s,t,i,m,a,b,l,e}
        node (x) [below of=lastx] {x};
      foreach x [remember=x as lastx (initially S)] in {m,a,l,l}
        node (x) [below of=lastx] {x};
      foreach x [remember=x as lastx (initially T)] in {e,s,t,a,b,l,e}
        node (x) [below of=lastx] {x};
    end{tikzpicture}
  end{figure}
end{frame}

end{document}
</code></pre>
This produces our nice acronym with the definitions placed below vertically:

<figure class="tmblr-full"><img class="aligncenter" src="http://68.media.tumblr.com/e8cb00ebb37f2f8ac860e6a6af586ab7/tumblr_inline_oez766wxmL1sjjac8_540.png" /></figure>But what if we wanted to show the definitions one by one? That’s super-easy, since the Beamer <code>pause</code> command is recognized inside the <code>tikzpicture</code> environment! So we have:
<pre><code class="tex">
begin{tikzpicture}[node distance=1em]
  node [SeaGreen!80!black] (I) at (0,0) {Large{I}};
  begin{scope}[node distance=3em]
  foreach x [remember=x as lastx (initially I)] in {N,V,E,S,T}
    node (x) [right of=lastx,SeaGreen!80!black] {Large{x}};
  end{scope}
  pause %here we pause after writing the capital letters
  foreach x [remember=x as lastx (initially I)] in {n,d,e,p,e,n,d,e,n,t}
    node (x) [below of=lastx] {x};
  pause %then we have a pause after each word
  foreach x [remember=x as lastx (initially N)] in {e,g,o,t,i,a,b,l,e}
    node (x) [below of=lastx] {x};
  pause
  foreach x [remember=x as lastx (initially V)] in {a,l,u,a,b,l,e}
    node (x) [below of=lastx] {x};
  pause
  foreach x [remember=x as lastx (initially E)] in {s,t,i,m,a,b,l,e}
    node (x) [below of=lastx] {x};
  pause
  foreach x [remember=x as lastx (initially S)] in {m,a,l,l}
    node (x) [below of=lastx] {x};
  pause
  foreach x [remember=x as lastx (initially T)] in {e,s,t,a,b,l,e}
    node (x) [below of=lastx] {x};
end{tikzpicture}
</code></pre>
which produces:

&nbsp;

<figure><img class="aligncenter" src="http://i.makeagif.com/media/10-13-2016/kE6pvw.gif" /></figure>And what if we want to uncover single <i>letters</i> piecewise? That’s neatly obtained by wrapping the argument of the corresponding <code>foreach</code> statement inside curly braces, so we can add a pause after drawing each node:
<pre><code class="tex">foreach x [remember=x as lastx (initially I)] in {n,d,e,p,e,n,d,e,n,t}{
  node (x) [below of=lastx] {x};
  pause
  }
</code></pre>
<figure><img class="aligncenter" src="http://i.makeagif.com/media/10-13-2016/VNeBbJ.gif" /></figure>…though I cannot guarantee your audience will not have dozed off at this point.

That’s it, I hope you had fun and above all that you decided to give Beamer a try ;)