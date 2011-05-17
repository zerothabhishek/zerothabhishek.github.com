---

layout: post
title: interview lost for polymorphism
permalink: interview.html
date: 04 may 2011
location: chennai

---


Actually for the lack of the knowledge of polymorphism.  

What is wrong with the following ruby code:  

{% highlight ruby %}
    
	def turn_left
        case @points
	      when 'N'  :   @points = 'W'
	      when 'E'  :   @points = 'N'
	      when 'S'  :   @points = 'E'
	      when 'W'  :   @points = 'S'
	    end
	end 
    
{% endhighlight %}

If you think that case-when construct should not have been used, this is for you.  
If you disagree with the above, for you too.  

A recent interview at a respected company (I respect it) revolved around this. The company required me to solve a programming problem before a face-to-face interview, and the above code was a part of it. I was impressed that the HR person who called me used the term _polymorphism_. She told that I'll be asked to _refactor_ the code in a pair-programming round, where I will have to improve my code using _polymorphism_ etc.

I must concede now that I am computer science educated, know the fundamentals of object oriented programming (OOP) and have been programming in ruby for more than a year. And I am an okay types programmer.

So the assignment for me at the pair-programming session was to remove the case-when block in this place, and an if-else block in another place. The session went on for almost an hour and a half, I asked for hints a couple of times, and the best I could finally get to was _something in the right direction_, but not good enough. The interviewer explained that this is pretty much the starting point of the way they think, and is quite a hard pre-requisite. He once asked me once how I wrote the [ruby gem](http://github.com/zerothabhishek/preview) I have written,  and it meant how could I write one if I am not proficient with OOP. By that time I had tried a number of approaches that were getting _eventually procedural_, so I replied - "like this only". 

Most interviews I have attended so far are tests of programming ability, conceptual clarity and intelligence. However rebellious I may be in real life, when I am in the interview, I seldom ask _why?_. If something looks unacceptable to me, I try facing it as a purely intellectual challenge, and try keeping my opinions aside. So even if I was some _procedural-programming-hardliner_ (if there is one such thing :)) I would have tried doing what I was asked to do. Actually the truth is, I like object oriented programming more than procedural programming, but am hardly dogmatic. Whatever works, and looks good (the code should look good) is fine with me.

So after completing the interview, I raised the **why?**. The thing is, object oriented programming is great, and it is a good programming challenge to turn `switch-cases` into something more _OOPy_, what problem will it solve that the current code does not? The code works, and it is readable ( Look at this code for example -
	
	def move
	  @position.change_in @direction unless movement_impossible?
	end
)

The answer was quite convincing. He said that suppose in future you want to extend ht e 