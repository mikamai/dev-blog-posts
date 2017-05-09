---
ID: 1396
post_title: A Little Dive into Styled-Components
author: Matteo Dellea
post_date: 2017-05-09 08:48:10
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2017/05/09/a-little-dive-into-styled-components/
published: true
---
In this last period I've been developing a project with `react`, `redux`, `immutable`, `reselect`, `redux-saga`, and the `styled-components`. I am really satisfied with `styled-components` so I decided to explain you why I think they are so cool.
<!--more-->

The `styled-components` library was written by Glen Maddern and Max Stoiber, and allows you to style your react components directly into js. Using `template literals` it's like writing css but in js!

Assuming we want to create a styled button, we'd write:
```
import styled from 'styled-components';

const Button = styled.button`
  position: relative;
  z-index: 1;
  display: inline-block;
  height: 36px;
  margin: 0px;
  padding: 0 10px;
  border: 10px;
  border-radius: 2px;
  box-sizing: border-box;
  outline: none;
  line-height: 36px;
  font-family: Roboto, sans-serif;
  text-align: center;
  text-decoration: none;
  cursor: pointer;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);  
  transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
  background-color: #00BCD4;  
  color: #FFF;
`;

// now you can call it in a render method, like this
// <Button>very cool button!</Button>
```
Our result should look like this:
<img src="https://dev.mikamai.com/wp-content/uploads/2017/05/start-button-300x96.png" alt="" width="300" height="96" class="alignnone size-medium wp-image-1398" />

Yes, `Button` is a React component!

**What happens in the backstage?**
`styled-components` creates a class with a unique name (a sort of hash), it injects that in the `<head>` of the page, and applies that class to the `className` of the generated component.

**Can I style a component?**
Yes, sure, `styled` is a function that accepts a component. In the example above we called the property `button`, it's just a "shortcut" to tell `styled-components` to create a component that renders the tag.
For example, assume we already have a `Button` component.
```
import styled from 'styled-components';
import Button from './Button';

const StyledButton = styled(Button)`
   here all the styles...
`;
```
But be careful because the `Button` component **must** accept `className` as prop to let `styled-components` inject the generated `className` like I told you before.

The very cool thing about `styled-components` is that you can change styles based on the prop of the component. 😵

**How can I do that?!**
What we're writing is a javascript string, and not a normal string, a `template literal`.
We can interpolate it using `${}` to inject constants and... functions! :O
Functions? In a string? Whaaat? 
`styled-components` take advantage of `tagged template literal` to add more power to styles.
We say `tagged template literal` when we call a function using `template literals`.
Check this example out:
```
function foo(array, ...interpolations) {
  console.log('array', array);
  console.log('interpolations', interpolations);
}
function baz() {}

foo`test${1}test${baz}test`
// => array ["test", "test", "test"]
// => interpolations [1, function baz()]
```
The first argument of the above function `foo` is an array of strings in between interpolations. All remaining arguments are the interpolated values.

**So, how can I change styles based on props?**
Each function you interpolate is called by `styled-components` along with the current props of that component. For this reason, the string returned by your function can change any part of style you'd like.
For example:
```
import styled from 'styled-components';
import Button from './Button';

const StyledButton = styled(Button)`
   here all the styles...

   cursor: ${(props) => props.disabled ? 'auto' : 'pointer'};
   ${(props) => props.disabled ? `
     opacity: .5;
     pointer-events: none;
   ` : ''}
`;

// now you can call it in a render method, like this
// <Button disabled>very cool disabled button!</Button>
```
Our result should now look like this:
&lt;img src=&quot;https://dev.mikamai.com/wp-content/uploads/2017/05/Screen-Shot-2017-05-07-at-15.54.32-300x68.png&quot; alt=&quot;&quot; width=&quot;300&quot; height=&quot;68&quot; class=&quot;alignnone size-medium wp-image-1409&quot; /&gt;

Now we can make different mixins (based or not on props) and make it all even more powerful.
```
const ellipsis = () => `
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
`;

const checkDisabled = (opacity = 0.5) => (props) => {
  if(!props.disabled) return 'cursor: pointer;';
  return `
    cursor: auto;
    opacity: ${opacity};
    pointer-events: none;
  `;
};

// usage example
styled.div`
  width: 250px;
  ${ellipsis()}
`;

styled.button`
  ...more styles
  ${checkDisabled()}
`;
```
If you&#039;re looking for something to make your life easier, you can always count on &lt;a href=&quot;https://github.com/styled-components/polished&quot;&gt;polished&lt;/a&gt;, which is a set of mixins that has been released recently, and that can help you write your styles faster.

Or we can make Higher Order Components like:
```
import styled from 'styled-components';
import Button from './Button';

const withEllipsis = (WrappedComponent) => styled(WrappedComponent)`
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
`;

// usage example
const EllipsisButton = withEllipsis(Button);
```

There's so much more we could say about `styled-components`... but for now this is all, y'all!
Enjoy using `styled-components`!!