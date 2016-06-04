---
layout: post
title: Passing Parameters From Rails Routes to Controller Actions
---

Say you have an index action renders a collection of posts. Your posts also have different kinds that have special conditions to render as a collection and you've created scopes to handle this.

```ruby
class Post < ActiveRecord::Base
  scope :development, -> { where(kind: 'development').recently_published }
  scope :climbing, -> { where(kind: 'climbing').unarchived }

  ...
end
```

You want to use the same index template to render the collection rather than duplicating the code. One way to do this is with a case statement and a parameter.

```ruby
def index
  case params[:post_kind]
    when 'development'
      @posts = Post.development
    when 'climbing'
      @posts = Post.climbing
    else
      @posts = Post.all
  end
end
```

A problem with this is that when you add a new scope you'll have to add an additional `when` block. One way around this is to use send.

```ruby
  @posts = params[:post_kind].present? ? Post.send(:post_kind) :
Post.all 
```

One obvious problem with this is that now our users can send any method
they like to the `Post` object (`destroy_all` for example). In order to
prevent this we should sanitize the input.

```ruby

def index
  @posts = Post.send(sanitized_kind)
end

private

def sanitized_kind
  %w( climbing development ).include?(params[:post_kind]) ? params[:post_kind] : 'all'
end
```

Lastly, instead of having the user pass the parameter as a get variable
we can set up additional routes which will provide cleaner URLs.

```ruby
get 'posts/development' => 'posts#index', post_kind: 'development'
get 'posts/climbing' => 'posts#index', post_kind: 'climbing'
```
