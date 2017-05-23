---
layout: post
title: Searching for Things in Rails
---

One common task that Rails developers use day to day is `rake routes` (or `rails routes` in Rails 5). As your app grows, you'll use `grep` to only show the routes you're interested in:

```
➜  rake routes | grep user
   users POST /users(.:format)     users#create
	 new_user GET  /users/new(.:format) users#new
	     root GET  /                    users#new
```

As of Rails 4 theres another handy way to search for routes. Simply start your `rails server` and navigate to `localhost:3000/rails/info/routes`.

![Rails Info Routes Magic]({{ site.url }}/images/rails_info_routes.png)

I've discovered a third useful way to find this information.

If you're running a rails console you can access your route helpers through the `app` object.

```
2.3.1 :001 > app.new_user_path
 => "/users/new"
```

You can use [Enumerable#grep](https://ruby-doc.org/core-2.1.0/Enumerable.html#method-i-grep) to search the `app`'s `methods` for these helpers.

```
2.3.1 :002 > app.methods.grep(/path/)
 => [:root_path, :rails_info_properties_path, :rails_info_routes_path, :rails_info_path, :rails_mailers_path, :users_path, :new_user_path, :path, :polymorphic_path, :edit_polymorphic_path, :new_polymorphic_path]
```

This contains some additional useful info that you don't get in `rails routes` - like the paths for `rails/info/routes` and `rails/info/properties/`. If also contains some not so useful info like the additional methods `:path` and `:new_polymorphic_path`. 

We can reduce these down even further using a second `grep`:

```
2.3.1 :025 > app.methods.grep(/path/).grep(/user/)
 => [:users_path, :new_user_path]
```

Greping through methods can be a great way to find new methods on objects that you aren't familiar with. This is a great illustration of Ruby optimising for programmer happiness.
