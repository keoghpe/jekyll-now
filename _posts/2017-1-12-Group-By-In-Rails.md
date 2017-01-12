---
layout: post
title: How Rails Returns Grouped Counts
---

Quite often I'll need to get a count grouped by multiple columns but the output is confusing:

```ruby
Comment.all.group(:post_id, :user_id).order(:user_id).count
{[16, 1]=>26, [26, 1]=>29, [18, 1]=>29, [12, 1]=>15, [10, 1]=>29, [25, 1]=>29, [22, 1]=>29, [28, 1]=>29, [15, 1]=>29, [7, 1]=>26, [23, 1]=>27, [20, 1]=>28, [19, 1]=>29, [2, 1]=>12,[21, 1]=>17, [6, 1]=>26, [12, 2]=>12, [21, 2]=>7, [2, 2]=>13, [6, 2]=>3, [16, 2]=>2, [23, 2]=>2, [7, 2]=>3, [16, 3]=>1, [12, 3]=>2, [20, 3]=>1, [21, 3]=>5, [2, 3]=>4}
```

Once you know that the array on the left just represents whats in the group it gets less confusing. The above is just of the form:

```ruby
[post_id, user_id] => count
```

This can be confusing for other developers when reading your code:

```ruby
user_comment_count[[18, 2]]
```

One solution is to use named variables to make it clearer:

```ruby
post_id = 18
user_id = 2
user_comment_count[[post_id, user_id]]
```
