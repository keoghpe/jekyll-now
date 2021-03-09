---
layout: post
title: Some CSS Grid
---

I've decided to redesign this site as a way of learning CSS grid. I'm going to code as I put this blog post together.

CSS Grid is a [W3C candidate recommendation](https://en.wikipedia.org/wiki/CSS_grid_layout) that allows developers to 
define layouts with different "track" (row or column) sizes. These sizes can be a mix of fixed and flexible sizes which
means it's a really powerful layout tool.

I'm looking to achieve a pretty simple layout to start with. I want a left sidebar that includes my name, a bit about me and a couple 
of relevant links. To the right of that I want to show the posts. I also want to keep a max width on the paragraph text 
to keep everything legible.

After stripping out some redundant html and clearing the floats out of my CSS, I'm left with this layout:

{% raw %}
```html
<body>

<header>
    <h1 class="site-name"><a href="{{ site.baseurl }}/">{{ site.name }}</a></h1>
    <p class="site-description">{{ site.description }}</p>
</header>

<div id="main" role="main">
    {{ content }}
</div>

</body>

```
{% endraw %}

First thing I need to do is define the body as display grid:

```css
body {
  display: grid;
}
```

Next, I need to defined the "grid tracks" using the `grid-template-columns` and `grid-template-rows` properties.
I'm going to let the rows autosize and use the new fraction (`fr`) grid unit. These fractions define what amount of
the grid each section should take relative to one another. If I want 3 equally sized columns I should use `1fr 1fr 1fr`.
I want my sidebar to be a quarter the size of my content, so I'm going to go with `1fr 4fr`.

```css
body {
  display: grid;
  grid-template-columns: 1fr 4fr;
  grid-template-rows: auto;
}
```

So far, so good. It's looking pretty much how I wanted, except the sections are touching - I can fix this by adding a gutter.

```css
body {
  display: grid;
  grid-template-columns: 1fr 4fr;
  grid-template-rows: auto;
  column-gap: 20px;
}
```

It's all going smoothly, until I check one of my older posts where the content radically
exceed the container and everything looks a bit mad! Apparently, this is known as a "grid blowout". This happens because,
by default the min width of a grid item is auto, so if the item is large (or overflowing) the grid will expand to fill it.
This can be fixed by setting the minimum width of the item to `0` with the `minmax` function.

```css
body {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 4fr);
  grid-template-rows: auto;
  column-gap: 20px;
}
```

I've pretty much gotten a layout I'm happy with, so I'm going to commit and deploy what I have.

Now that I've deployed, I'm going to play around with subgrids for the posts layout.



To do:
- play around with `repeat(3, 1fr)`
- what about mobile? Reordering?

