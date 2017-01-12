---
layout: post
title: Multiple Assignment in Ruby
---

In ruby you can assign to multiple variables like so:

```ruby
a, b, c = [1, 2, 3]
```

But what if you also want to store the whole array in a variable?

In Elixir:

```elixir
iex(1)> d = [a, b, c] = [1,2,3]
[1, 2, 3]
iex(2)> d
[1, 2, 3]
iex(3)> a
1
iex(4)> b
2
iex(5)> c
3
```

How about we try this:

```ruby
d = a, b, c = [1, 2, 3]
```

This gives us some unexpected results!

```ruby
> d = a, b, c = [1,2,3]
=> [1, 2, [1, 2, 3]]
> d
=> [1, 2, [1, 2, 3]]
> a
=> 1
> b
=> 2
> c
=> [1, 2, 3]
```

I'm not sure why - I've looked [here](http://ruby-doc.org/core-2.4.0/doc/syntax/assignment_rdoc.html), but haven't figured it out yet.

We can achieve the desired result with the following:

```ruby
d = (a, b, c = [1, 2, 3])
```
