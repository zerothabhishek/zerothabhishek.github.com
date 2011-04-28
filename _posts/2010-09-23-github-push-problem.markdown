---

layout: post
title: github push problem
permalink: github-push-problem.html
date: 23 sep 2010
location: chennai

---

This is error I get when I try pushing my changes to my github repo -   

    error: Cannot access URL http://github.com/zerothabhishek/rc3.git/, return code 22
    fatal: git-http-push failed

A google reveals that the problem could be when you are operating from behind a firewall. See [this](http://www.google.co.in/search?hl=en&safe=off&q=github+git-http-push+failed&aq=f&aqi=&aql=&oq=&gs_rfai=) and [this](http://rubenlaguna.com/wp/2009/09/23/git-error-pushing-via-http-return-code-22/).
I have my ssh-keys and other stuff all set correct, and have done a number of pushes to gihub from the same computer. So this reason for the error surprises me.  

I experiment with a few config changes and it works.  
Try this instead for the command -  

    sudo git push git://github.com/zerothabhishek/rc3.git master

and it comes up with the following -  

    You can't push to git://github.com/zerothabhishek/rc3.git
    Use git@github.com:zerothabhishek/rc3.git

After this, I try -  

    sudo git push git@github.com:zerothabhishek/rc3.git master

And it worked.  

Perhaps changing the origin url in the .git/config file would help for future.  


I guess the problem was the way I pulled in the repo this time. I had it created on another computer, pushed to github, and then used git clone to get it here. I guess the problem lies in that only. Needs to be probed further.

Surprising this the fact that of people face this problem, and don't get satisfactory answers [even at github support](http://support.github.com/discussions/repos/3266-http-push-problem).
