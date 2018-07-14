---
layout: post
permalink: /we-need-callback-sometimes.html
---

# We need callbacks, sometimes

ActiveRecord callbacks are the most criticized features of Rails. I too have agreed with the criticism sometimes, because I too find callbacks difficult sometimes. But my opinion has changed a little now.

For the uninitiated, here are some critical posts -

- https://medium.com/planet-arkency/the-biggest-rails-code-smell-you-should-avoid-to-keep-your-app-healthy-a61fd75ab2d3
- http://samuelmullen.com/2013/05/the-problem-with-rails-callbacks/
- https://www.bignerdranch.com/blog/the-only-acceptable-use-for-callbacks-in-rails-ever/
- https://robots.thoughtbot.com/activerecord-caching-and-the-single-responsibility
- http://felixclack.com/post/89947539045/how-should-i-use-activerecord-callbacks

If I could summarize the intent of all the criticism, 

- Callbacks violate the Single-Responsibility-principle.
- They destroy the linear flow of code
- They cause surprising/unexpected side-effects
- They can cause surprise rollbacks and returns
- They are hard to test

Personally, I've found trouble only with the last thing - callbacks make testing difficult because they are hard to turn-off. 

People have also claimed that a __callback should only be used when the logic in it applies to the same object__. That is the only valid use case of callbacks. Sorry but I disagree. *Disagree without significant hyperbole*.

Lets look for a consensus here: if I can find one use-case where callbacks are _actually_ useful, can we call it truce? Isn't that how technical disputes are settled?

>>
    # Consensus building 101  
    Person-A: X language/framework is bullshit. Here are the reasons ....  
    Person-B: We found it useful in these situations (describes personal experience)  
    Person-A: Yea but other from that situation, its bullshit.  
    Truce ☮️ ✌️ (everyone lives happily ever after)  


So here's my use-case - a common e-commerce cart application. Standard stuff - data organized as _Order_, _LineItems_, _Products_. We calculate totals, send user to payment gateway. And then there is a scenario looking somewhat like this -

    As a user,
    When I change the quantity of a purchased product,
    The the order-total should be updated.


![](/content/images/2016/06/amazon-orders-page-1.png)

The `quantity` is a column in the `line_items` table. When the quantity in a line-item changes, the order total also must get updated. I come up with the following code -

    # In LineItem class
    def update_quantity(new_quantity)
      update_attributes(quantity: new_quantity)
      order.update_total
    end

Both the steps must happen together. And the second one should never be missed. **Never miss it.**
I can over-emphasize that idea in comments, and maybe in tests. But there is nothing in the code that can prevent a mistake there.

The update Api is still open to use. I can just as easily do this anywhere I want:

    line_item.update_attributes(quantity: 0)

Ideally we need some kind of automatic trigger that always updates the Order when a line-item gets updated. The least we should have done is expressed this intent clearly somewhere.

**Did you say trigger?**   

As it turns out, relational databases have had a solution for this for a long time. Its called (..drumrolls) **triggers**. The following is an example from Postgresql.

    CREATE TRIGGER sample_trigger
      AFTER UPDATE
      ON line_items
      FOR EACH ROW  
      EXECUTE PROCEDURE update_order();
    # update_order is a PL/pgSQL function

Any query that updates the `line_items` table will trigger the `update_order` function. The BEFORE/AFTER INSERT/UPDATE/DELETE and several other events are monitored. And if we go into the best-practices for triggers, the advise is similar to callbacks - **Avoid using triggers**. Triggers are hard to maintain and can cause unexpected behavior. I've even heard of projects that even forbid triggers.

**Back to callbacks**   

Anyway, I added a callback to the LineItem model. Now, ==anytime, anyhow, anyone changes the quantity, an update will be triggered on the Order model==. And it lies gloriously on top of the model file, along with all the other declarations and macros. Not that easy to miss. That fits my need fairly well.

Lets now see how well I have annoyed the purists.

- Is it hard to test?  
Indeed. The LineItem model can't be tested in isolation. Unless we use stubs, we always need the order object during testing.

- They can cause surprise rollbacks and returns
Not in this case. May be when I add a conditional returning a nil that breaks the callback chain, I can land in trouble. Or if there's an exception in the callback. This behavior too has [improved lately](http://blog.bigbinary.com/2016/02/13/rails-5-does-not-halt-callback-chain-when-false-is-returned.html).

- They destroy the linear flow of code
May be. But not enough to be noticeable here. I too find deeply nested code hard to follow. Moreover, non linear code is  hard reality - I don't think we can avoid it completely.

- They cause surprising/unexpected side-effects
In this case, the side-effect may be surprising but not unexpected. In fact the expectation is the whole point of this example.

- It violates Single Responsibility Principle (SRP).  
Yes and so be it.
Programming is not a design principle adherence contest. We are supposed to solve real world problems, and to solve them economically. I understand that the principle is meant to make code more readable and maintainable. But if you think my code is not maintainable, say *that* and we can talk. Saying it violates some principles is just a religious argument.
Also, the Ruby on Rails framework itself is an achievement of **pragmatic, non-religious programming**. A Rails model too is a violation of SRP. It is useful, it works well and is generally maintainable. 

**You should add a service ...**  

If I had a penny every time I got that advise!
My code lands in the hands of a anti-callback activist, and since all good object oriented programmers write service classes, here's what she does -

    class LinteItemQuantityUpdator
      def initialize(line_item)
        @line_item = line_item
        @order = @line_item.order
      end

      def update_quantity(q)
        @line_item.quantity = q
        @line_item.save
        @order.update_totals
      end
    end

I have no objection to this (actually I do, lets do that later), but it does not solve my original problem. My `LineItems` are still open to an incorrect update. Is there a way I can prevent the incorrect thing in a pure OOPsy way ?

**Something about coupling**  
I know I have introduced coupling, and I know it is such a bad thing to do. But the coupling here is a requirement of my domain logic - my business. Missing this coupling can cost me money. **When coupling is warranted externally, no design principles should be used to hide from it.** 

While we use callbacks mostly to send emails and the such, I'm totally cool with replacing them with whatever *OOPSy* thing you want. But when a callback helps hold together  **coupling that we must never miss**, better control your urges to remove it. 

==Callbacks provide coupling guarantees==. And you need it sometimes.


