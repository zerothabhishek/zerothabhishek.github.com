---
layout: post
permalink: /the-super-problem-in-ruby.html
---

# The super problem in Ruby


Modules using `super` are perhaps the most difficult Ruby code I've seen. Look at this bit from Rails, for instance:

        def etag=(etag)
          key = ActiveSupport::Cache.expand_cache_key(etag)
          super %(W/"#{Digest::MD5.hexdigest(key)}")
        end

(from [ActionDispatch::Http::Cache::Response](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/http/cache.rb#L91))  
_[Edit: This may not be the right example for my point. Check the update below]_

The is supposed to calculate and assign an Etag to the Http response. It does part of the work - calculating the key and the digest, and then passes on the subsequent handling to another method through the `super` call.

There is no indication of where that super method is defined. The method is expected to be included by some other class or module, so we may get some idea if we check the including class. 

That is also not always simple. The class that includes this module could have included many other modules. `ActiveRecord::Base` includes 33. And any of the included modules could have defined the needed super method (actually only those that get included before our module). We may have to check in a number of modules to reach the one that may have defined the method. Its also possible that those other modules have their own ancestor modules. Module dependencies could be unbearably complex.

#### A simplified example

    module A
      def foo
        'A foo'
      end
    end

    module X
      def foo
        super  # <-------------
        'X foo'
      end
    end

    class Sample
      # Includes A, X and many others
      include A
      include B
      include C
      include X
    end

Whenever `X::foo` has to execute, it expects to also execute the `A::foo` via the `super` call. The fact that the `super` has to translate to `A::foo` is not apparent by looking at the module `X` alone. When we look into a class that finally includes `X` -  like the `Sample` here, we see the other included modules `A`, `B` and `C`. Any of these could have defined `foo`, so we should go check within each of them. This can get difficult if some of these are from completely different projects.

Its much easier to actually set a break-point and see where the flow goes. And the earlier example, it went to the `Rack::Response::Helpers` module in __rack__.

#### Its not just about super

The problem is not limited to `super` alone. The entire pattern of using methods inherited from mixed-in modules is to blame. The more complex module dependencies become, the more difficult it is to follow.

Code like this is inadvertently ambiguous. ==The author is actually clear on which super needs to get invoked== _(Edit: Not always. See update below)_. But there is no way for her to express that. Other than maybe leaving a comment near the code.

Also, it gives the author very little control to limit ambiguity. What if another module re-defines the method, and _that_ unrelated method ends up getting called instead of one that author wanted ?

#### The solution

I don't know. Inherit instead of mixing-in maybe. Redefine the super method within the module maybe. Prefer using module-functions maybe. (If the methods are not part of the external Api, there's a good chance that a instance method is not entirely necessary.) Or maybe invoke the module method directly:

    module X
      def foo
        the_super
        'X foo'
      end

      def the_super
        Class.new.extend(A).foo
      end
    end

I personally find the last one best, but it reads a bit weird. I wish we had a simpler way to directly execute a method defined in a module. Something like this maybe -

    A.execute(:foo)

What do you think?

---
#### Update 

After a few comments and a discussion on [Reddit](https://www.reddit.com/r/ruby/comments/4bwxyu/the_super_problem/), here are my conclusions:

- Usage of `super` in libraries may be done to deliberately introduce flexibility. The example from Rails that I quoted could be one like that. The author perhaps really wants to permit overloading the super `etag=` method. The flexibility you get from super is the primary advantage.

- Inheritance and mix-ins are not that different. Module mix-ins are just another way of implementing inheritance. (Thanks to Josh Bodah for the comment)

- We can use the `super_method` call to pin-down the exact method while debugging. Here are some more info (Thanks to Richard Schneems):  
  http://www.schneems.com/2015/01/14/debugging-super-methods-ruby-22.html
  https://bugs.ruby-lang.org/issues/9781

- To invoke a module method directly we can do something like this:

      `A.method(:foo).bind(self).call`

Thanks to everyone who commented.


