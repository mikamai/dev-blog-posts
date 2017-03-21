---
ID: 94
post_title: 'A month with Atom: my first package'
author: Massimo Ronca
post_date: 2015-07-27 13:24:37
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/07/27/a-month-with-atom-my-first-package/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/125168394569/a-month-with-atom-my-first-package
tumblr_mikamayhem_id:
  - "125168394569"
---
Exactly one month ago, Github relaesed version 1.0 of their open source editor Atom.
Giovanni <a href="http://dev.mikamai.com/post/122755410549/a-month-long-atom-test-drive-from-an-emacs-fanatic">already blogged</a> <a href="http://dev.mikamai.com/post/124331931184/atom-18-days-in">about it</a>, from the perspective of a long time Emacs lover: Atom still has a long way to go, but he also admitted that
<blockquote>There are several things going for Atom though. Coffeescript is a more popular language than Lisp, also, I know it better, so writing my own Atom extensions won’t be as hard as writing my Emacs extensions.</blockquote>
I’ve never been proficient enough with Lisp to write a package for Emacs, and Vimscript is really too hackish for the undisciplined developer I am.
I know Javascript better, but I don’t love it.
Atom packages can be written in Coffeescript, a language that I really enjoy.
Language alone is surely not enough to declare that writing a package is going to be easy, but it gave me the confidence to try.

Our package will be very simple: it will put a live clock on the right side of the status bar that can be toggled on and off.

<!--more-->

To begin, we create a <code>status-bar-clock</code> folder inside <code>~/.atom/packages/</code> and add a <code>package.json</code>, in the same way we dot for regular node packages.
Atom added a <a href="https://atom.io/docs/v0.186.0/creating-a-package#packagejson">few unique keys</a>, where <code>main</code> is the only one required and points to the entry point of our package.

The base version of our <code>package.json</code> will look like this

<pre><code class="json">
{
  "name": "status-bar-clock",
  "main": "./lib/status-bar-clock",
  "version": "0.0.1",
  "description": "Show a clock inside the status bar",
  "repository": "https://github.com/wstucco/status-bar-clock",
  "license": "MIT",
  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  }
}
</code></pre>

Then we create the package main and put it in <code>lib/ststus-bar-clock.coffee</code>.
Atom packages are simple objects that implement the two methods <code>activate</code> and <code>deactivate</code>; only <code>activate</code> is required, <code>deactivate</code> is called when the package is removed or deactivated from the preferences and is useful to clean up: in OO terms it’s the destructor.

We’ll start with the most basic package possible

<pre><code class="coffeescript">
module.exports = StatusBarClock =
  activate: (state) ->
    console.log 'Clock was activated'
  deactivate: ->
    console.log 'Clock was deactivated'
</code></pre>

Save it and press <code>ctrl-alt-cmd-l</code>, it will reload the editor, including our package code. If you open up the developer tools with <code>cmd-alt-i</code> you should see the message <code>Clock was activated</code> in the console.
If you disable the package from the preferences, the message <code>Clock was deactivated</code> will show up.
We’re in business!

Enabling and disabling the package from the preferences is not very convenient, moreover it’s not what we really want, we just want to toggle the clock on or off, without having to disable the entire package.
We’ll add a keyboard shortcut, <code>keymaps</code> in Atom: keymaps are <code>json</code> or <code>cson</code> files inside the <code>keymaps</code> folder (for documentation <a href="https://atom.io/docs/latest/behind-atom-keymaps-in-depth">see here</a>).
We bind the toggle command to <code>ctrl-alt-t</code> (the same action should be present in the action menu if you press <code>cmd-shift-p</code>)

<pre><code class="coffeescript">
# keymaps/status-bar-clock.cson 
'atom-workspace':
  'ctrl-alt-t': 'status-bar-clock:toggle'
</code></pre>

Once we declared the mapping between keys and commands, we write the code to handle it.
<blockquote>note: the binding will only work if we are inside the Atom workspace, so if, for example, the developer console has the focus, the binding won’t work</blockquote>

<pre><code class="coffeescript">
{CompositeDisposable} = require 'atom'
 
module.exports = StatusBarClock =
  active: false
 
  activate: (state) ->
    @subscriptions = new CompositeDisposable
    # Register command that toggles this view 
    @subscriptions.add atom.commands.add 'atom-workspace', 'status-bar-clock:toggle': => @toggle()
 
    console.log 'Clock was activated'
  deactivate: ->
    console.log 'Clock was deactivated'
 
  toggle: ->
    console.log 'Clock was toggled on' if @active
    console.log 'Clock was toggled off' if !@active
 
    @active = ! !!@active
</code></pre>

Now we need to add the message to the status bar, the status bar in Atom is a <code>service</code> and exposes an API that can be consumed.
We declare this dependency in <code>package.json</code>

<pre><code class="json">
  "consumedServices": {
    "status-bar": {
      "versions": {
        "^1.0.0": "consumeStatusBar"
      }
    }
  }
</code></pre>

When the status bar is initialized, it will call a method in our package, passing an instance of itself as parameter.
Status bar API docs can be found <a href="https://github.com/atom/status-bar">here</a>

<pre><code class="coffeescript">
consumeStatusBar: (statusBar) ->
  @statusBar = statusBar
Status bar can display different blocks called titles on the left and right side. The displayed element must be a view which “can be a DOM element, a jQuery object, or a model object for which a view provider has been registered in the view registry”
We’re going the simple route and create a DOM element

class StatusBarClockView extends HTMLElement
  init: ->
    @classList.add('status-bar-clock', 'inline-block')
    @activate()
 
  activate: ->
    @intervalId = setInterval @updateClock.bind(@), 100
 
  deactivate: ->
    clearInterval @intervalId
 
  getTime: ->
    date = new Date
 
    seconds = date.getSeconds()
    minutes = date.getMinutes()
    hour = date.getHours()
 
    minutes = '0' + minutes if minutes < 10
    seconds = '0' + seconds if seconds < 10
 
    "#{hour}:#{minutes}:#{seconds}"
 
  updateClock: ->
    @textContent = @getTime()
 
module.exports = document.registerElement 'status-bar-clock',
  prototype: StatusBarClockView.prototype, extends: 'div'
and display it

{CompositeDisposable} = require 'atom'
StatusBarClockView = require './status-bar-clock-view'
 
module.exports = StatusBarClock =
  active: false
 
  activate: (state) ->
    console.log 'Clock was activated'
 
    @subscriptions = new CompositeDisposable
    # Register command that toggles this view 
    @subscriptions.add atom.commands.add 'atom-workspace',     'status-bar-clock:toggle': => @toggle()
 
    @statusBarClockView = new StatusBarClockView()
    @statusBarClockView.init()
 
  deactivate: ->
    console.log 'Clock was deactivated'
    @subscriptions.dispose()
    @statusBarClockView.destroy()
    @statusBarTile?.destroy()
 
  toggle: ->
    if @active
      @statusBarTile.destroy()
      @statusBarClockView.deactivate()
    else
      console.log 'Clock was toggled on'
      @statusBarClockView.activate()
      @statusBarTile = @statusBar.addRightTile
        item: @statusBarClockView, priority: -1
 
    @active = ! !!@active
 
  consumeStatusBar: (statusBar) ->
    @statusBar = statusBar
    # auto activate as soon as status bar activates 
    @toggle()
</code></pre>

Last we add a submenu inside the <code>Package</code> menu that executes the toggle command.
Create a <code>menus/status-bar-clock.cson</code> and add

<pre><code class="coffeescript">
# See https://atom.io/docs/latest/hacking-atom-package-word-count#menus for more details 
'menu': [
  {
    'label': 'Packages'
    'submenu': [
      'label': 'Clock'
      'submenu': [
        {
          'label': 'Toggle'
          'command': 'status-bar-clock:toggle'
        }
      ]
    ]
  }
]
</code></pre>

Reload. The clock should be there at the bottom right, updating itself.
The final package can be viewed at <a href="https://github.com/wstucco/status-bar-clock">https://github.com/wstucco/status-bar-clock</a>.