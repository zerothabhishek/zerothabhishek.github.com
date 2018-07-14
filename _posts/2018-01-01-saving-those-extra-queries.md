---
layout: post
permalink: /saving-those-extra-queries.html
---

# Saving those extra queries


In Rails apps, eager loading and the ActiveRecord relation chains are good ways for keeping in check the number of queries we fire. But its also easy to lose their benefits. Here are some examples -

**The count** 

The `count` method always introduces a new query. If the query has already been fired, this one is unnecessary. Its not N+1, but still wasteful. The solution is to use `size` instead of count.

    posts = Post.where(user_id: user.id)
    posts.to_a   # Query fired
    posts.count  # Query fired
    posts.size   # No query fired

    posts = Post.where(user_id: user.id).includes(:comments)
    posts.each do |post|           # Query fired   
      puts post.comments.map(&:title)
    end
    post.comments.count # New query
    post.comments.size  # No new query


**The pluck**

*Note: This is fixed in Rails-5*

`pluck` doesn't make use of an existing relation chain - it always fires a new query.

    posts = Post.where(user_id: user.id)
    posts.to_a           # Query fired
    posts.pluck(:title)  # Another query fired
    posts.map(&:title)   # No new query

Fix: Using map instead of pluck

##### The parent reference

The following code will make an N+1

Within a `Comment` model:

    belongs_to :post
    def post_title
      post.title
    end

And then:
   
    posts = Post.where(user_id: user.id).includes(:comments)
    post.comments.map(&:post_title)

The example looks a little absurd because we could have done `post.map(&:title)` instead of the second line. But this is just an (over simplified) example - such a thing can occur when method call sequence more complicated. Its ever hard to notice such a thing without a tool.

The fix - the following slightly ridiculous bit:

    posts = Post.where(user_id: user.id).includes(:comments => [:post])


That's it. If you have any more ideas, please share in comments.

