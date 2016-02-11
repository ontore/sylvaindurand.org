---
title: Using <em>CSS</em> to add<br/> line numbering
lang: en
---

When you want to display a code listing with *HTML*, you use a `<pre>` tag in order to indicate that the text is preformatted[[and therefore the spaces and line breaks must be respected]], then inside one or multiple `<code>` tags to specify this text is a code.

On this website, the line numbers appears on the left of code listings, like most word processors. The methods to achieve this vary widely: many use *jQuery* or *Javascript*, ugly HTML codes, or even tables...

Yet it is possible to achieve this in a simple way, using only CSS and HTML. The line numbers won't be selected when the user wants to copy the code.

## HTML
The only thing to do in our HTML code is to use a `<code>` tag for each line of code. This will produce a perfectly valid HTML code, and we will number each line directly with CSS:

```html
<pre>
<code>class Greeter</code>
<code>  def initialize(name)</code>
<code>    @name = name.capitalize</code>
<code>  end</code>
<code>end</code>
</pre>
```

## CSS

The `:before` pseudo-element will allow us to show something before each line, while the CSS counters will allow us to count the lines.

We will first define a counter that starts at one for each block, then which is incremented at each new line of code:

```css
pre{
    counter-reset: line;
}
code{
    counter-increment: line;
}
```

Then we display the number at the beginning of each line:

```css
code:before{
    content: counter(line);
}
```

With *Webkit* browsers, selecting the code to copy it gives the impression that the numbers are selected[[although fortunately they are not copied]]. To avoid this, simply use the property `user-select`:

```css
code:before{
    -webkit-user-select: none;
}
```

It only remains to improve their style as desired. That's it!
