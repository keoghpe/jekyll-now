---
layout: post
title: Some CSS Grid
---

I've decided to redesign this site as a way of learning CSS grid. I'm going to code as I put this blog post together.

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
I'm going to let the rows autosize and use the new fraction (`fr`) grid unit. 



