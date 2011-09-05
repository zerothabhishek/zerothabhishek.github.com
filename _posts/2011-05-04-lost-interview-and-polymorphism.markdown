---

layout: post
title: polymorphism and the lost interview
permalink: interview-polymorphism.html
date: 04 may 2011
location: chennai

---

<div class="notice"> This post is a draft right now. May be changed/deleted soon. Please don't rely on it/link to it right now.</div>

## prelude: the mars-rover problem

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

The above is a snippet of the code I wrote for a recent interview at [ThoughtWorks](http://thoughtworks.com). The recruitment process required me to solve a programming problem before a face-to-face interview. The problem was about a 'mars-rover' object that has to be controlled programmatically, has a position and direction, can turn in one of the four cardinal directions, and move in its current direction. For a given set of movement instructions (move, turn-right, turn-left ...) and initial position, we are supposed to determine the final position and direction of the rover. Full problem statement is [here](/mars-rover-problem.txt).

The above code is a method defined for the mars-rover object, that will do the left turn. `@points` stores the current direction for the rover. The logic is to change its value for left turn, depending on what the current value is. And somewhere else in the code, I have a code sequence somewhat like this-

{% highlight ruby %}

    rover = MarsRover.new(given_initial_position)
    rover.turn_left  if current_instruction.is("turn_left")
    
    # at last-
    rover.get_points  # shows up the value of @points

{% endhighlight %}

I was happy with my solution. The problem was not very difficult, and because the time was sufficient, I was able to write tests (not TDD). The code was well-commented and readable.

## the interview

I was eventually invited for the face-to-face interview. The HR person who I talked to said that the interview will be a pair-programming session where we (the interviewer and I) will re-factor the code. The code lacks application of concepts like polymorphism, and we will try to fix that. I was impressed with her because I had never heard an HR person talk about polymorphism and refactoring.

The assignment for me at the pair-programming session was to remove the _case-when_ block in the above example (top) (and an _if-else_ block in one more place). The session went on for almost an hour and a half, I asked for hints a couple of times. I was unable to make the required changes, and the best I could finally get to was something in the right direction. The interviewer explained that this is quite a basic expectation, and is pretty much the starting point of the how they think at work. He had decided to not hire me.

## a bit about the interviewer and a bit about me

My interviewer was not condescending at all, in fact he was very kind and friendly with me. I also liked his honest and polite way of conveying the rejection. Interviewers I have faced before are mostly diplomatic about this. He later on gave a very good [presentation](http://vimeo.com/25410417) at Ruby-Conf-India.

I am computer science educated, I know the fundamentals of object oriented programming (OOP), and had been using ruby for almost a year (though most of my experience before ruby was with system-programs written in C). Judgements about my programming competence apart, I think this is one of those necessary things I missed out on learning. Or I missed out on _knowing_ it was necessary.

## the convincing explanation 

Because I was so convinced with the goodness of my solution, I asked him to explain the merit of his demand. "What is wrong with this code? It is readable, and it works." (In hindsight, I was defending my work.) And I got a good answer - it is difficult to extend. The code represents the cardinal directions with the characters 'N', 'E', 'W' and 'S', and uses branching on them. What if in future we want eight directions instead of four. We would need to add four more branches here, and in every other place that deals with directions. That may be difficult and error prone.

It made sense. I had seen the pages and pages of code in switch-case constructs at my previous companies (NetApp and HP). Many of them must have started with the humble single liners like this one, but get bloated on additions because the only place to express the related logic is the block within the branch.

## new code

So a better thing at this point would be  -
{% highlight ruby %}

    # in MarsRover class
    def turn_left
      @direction = @direction.left   # @direction is an object of class North
    end

{% endhighlight %}

and then,
{% highlight ruby %}

    class North
      def left
        East.new   # When an object of direction North wants 'left', it gets an object of East
      end
    end

{% endhighlight %}

and much before all this,
{% highlight ruby %}

    direction = case given_dir
      when 'N' ; North.new;
      when 'S' ; South.new;
      when 'E' ; East.new;
      when 'W' ; West.new; end
    r = Rover.new(direction, other_parameters)

{% endhighlight %}

I still could not get rid of the branching. It is not possible to eliminate branching altogether. The rover's initial direction is dependent on the input given, which _has_ to pass through branches at some point. The closer it is to the entry point of the system, the better. Now if we had to add more directions to the system, we will define a few more classes, and a modification to the case-when branching construct at the start. The place for the direction related logic is in the direction's class, so this is much easier to understand and modify. And we get rid of the if-else in the rover's turn_left method.

## post-mortem 

Although I was quite convinced with his argument, I continued thinking about the "extend" part. As soon as he mentions extend, I am reminded of the ugly branching code I had seen, and assume that extending means adding more features/code (more turning direction here, for example). But I have to question that assumption too. The future of most code may be more code, but the future _demands_ from the code may not always be more code. One example of a future demand is performance. All abstractions come with performance overheads. (That now reminds me that NetApp code had very hard requirements for performance). Not saying that we can't write object oriented code having good performance. The point is, we should not assume too quickly about the future of our code.

But what do we do when we are clueless about the future. I suppose we write code that is easier to change - add features, remove features, optimize. Perhaps that is what 'maintainable code' means. Perhaps that's why using object-oriented techniques make sense, even when the future is unclear.

## and ...

And this was an interview. The intent was not really to build a mars-rover controlling software, it was to test my ability to program a certain way. 

## also,

Finally the most dramatic part. After narrating all these thoughts to Uma (my wife), I learn from her that the problem/ polymorphism everything is discussed at length on the internet. I just had to google it, and I could have made it throughs.

## lessons

So my lessons from the experience:

1. What looks readable is not necessarily maintainable
2. Polymorphism makes code more maintainable

.

-----------
