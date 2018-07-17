---
layout: post
permalink: /so-your-developer-made-a-git-mistake.html
date: 2016-02-26
---

# So your developer made a git mistake

... and it went in to production.

I have come across at least three such instances over the past year. The incidents were so similar that I couldn't help but see a pattern.

Things happened somewhat like this -

- He pushes wrong code to master
- Wrong code gets deployed on production
- Everyone in the team has merged the wrong code.

<br>

### An urgent chore

The team steps into a frenzy for rollback. If we used something like Capistrano, rolling-back production to the last working state is easy. Then we want to remove the wrong code from master. This becomes a high priority because we can't deliver any more code on top of a broken master. We can either add a fix that reverses the wrong change or revert it to an older, healthier commit-id. Adding a new, _undo_ fix may not be an option if the error needs analysis. 

Reverting to an older stable point may also not be simple. If the failing code has too many commits or merge commits with master, it becomes even more complicated. It is hard to even pin-point which commit-id we should revert to.

This will be an annoying chore if the team doesn't have a lot of experience with reverting in git. In these instances, it takes between a few hours to a full day's work. Its generally the senior-most developer who does it. And all other ==**code delivery has to stop while the revert finishes**==.

<br>

### Fault finding

Since the incident causes such a major disruption in the team's schedule, the manager/product-owner gets involved. And the natural next step is to fix the blame. The developer may have been confident about the code, but could have made an unintentional git mistake, like one of these -

- Committing wrong files
- Mixing-up while merging with master
- Working on an old branch
- Using git push -f

For a non technical person, these details don't matter. He made some kind of mistake. Because he was not careful, or because he didn't follow the process.

And the developer should get punished for that. In all three of my instances the developer gets scolded.


Now lets stop there for a moment.

- Did the developer and the team ever get any training in git? Or any on how to do revert in git?
- Was the developer interviewed for git before joining?
- Is scolding as punishment practiced in your organization?
- Did the team have any process for git? Any document list of dos and don'ts? Anything that could have prevented the occurrence of such an event?

Unfortunately, answers to all these questions were in negative.

**So a junior developer makes a mistake for the lack of training, gets scolded, and the only prevention plan for the future is more scolding?**

***

To be objective, the problem is that we all underestimate the risks of git. Cost of mistakes is much lower with the rest of our development stack. Most of our tools and libraries are much more developer friendly than git. So if there is one take-away from these experiences, it is: ==**git is risky**==

So if you are a junior developer in a company with no processes for git - be very careful. You will make a mistake sometime, and you will get blamed. Learn git as much as possible, and be careful and conservative. Here are some tips:

### Play safe:

- Learn about `git stash`. It will save you a lot of inconvenience. 
- Use `git diff` and `git diff --cached` to inspect changes before committing
- Add useful commit messages. Add text that helps answer *why this?*. for the change. So people can tell why the change was done.
- Prefer `git rebase` over `merge`
- Inspect the 'Diff' of your branch before merging with master or sending pull requests.

And if you're in the position to influence a process, get one adopted. There are many, get any one. Having one is better than not having one. Here's one I like:

<script src="https://gist.github.com/zerothabhishek/49e5156417ea230c361e.js"></script>

<br>

### The manager

People make mistakes. And nearly every occurrence of a mistake can be blamed on someone's carelessness or lack of diligence. Can punishment by scolding prevent future occurrences of a similar problem? Perhaps it can improve the level of caution, and that would help if lack of caution was the real cause of the problem.

From what I claim above, the real reason could have been lack of training. Or a missing process related to git. There should actually be an introspection session after the firefighting finishes. A blame-free analysis on what caused the issue, and what could have been done for the future.

And if you are the manager, also think about the punishment part. **Scolding just became a part of your company culture**. The word will spread, and people will know. Everyone avoids companies that have such problems in work culture. Find better ways of dealing with this.

