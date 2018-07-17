---
layout: post
permalink: /are-your-tests-passing-on-staging.html
date: 2016-05-28
---

# Are your tests passing on staging?


Automated tests are not run on staging or production. This may be obvious if you are a developer. But for the non-technical participants in your project, it may not be as obvious.

I have had to explain this to my team mates at least twice so far. To be honest, at first it sounded a bit ridiculous. And a bit scary too -  *run my tests on production !!*

#### Automated Tests = Computerized-QA ?  

But when we look at it from their side, its not that in-obvious. **Testing can mean different things to different people**. For the product owner (or business analyst), testing is more like QA. We install the software, and run it through a set of use cases. If there are any recent features, we verify them and make sure they work as intended. And then we check the older features - to be sure the new change hasn't broken any of those (regression testing).

If its a website, we typically check if the login flows are working, and then whether some important core functions (like payments) are okay. And all this happens manually.

**Its natural to imagine automated testing as an automation of manual QA** when it comes into picture. More like **computerization**. So instead of a human clicking links and buttons, there is script that does it.

And **manual QA is also performed on production** software, at least partly.  Its generally more rigorous in a staging/UAT type environment, and *lighter* on production. So if the product owner imagines production-QA is also getting automated, its not exactly wrong.

At this point may be you are thinking - the guy has worked with quite a few uninformed people. Any trained BA should/would know how automated testing works? Well then sorry to break your bubble. **Clients, product-owners don't owe us the right assumptions** - its our job to explain and justify it all. And trying to justify our choice of tools and practices helps us get the more real world perspective on their utility. And helps beat tech bandwagon-ism.

#### General conventions

Moving on, here are some general, high level conventions/tenets of the practice of automated testing, that product owners should be explained -

1. **Automated tests are executed by the programmers on their laptops, or on separate test servers** - sometimes called CI servers. They are not run on staging/UAT or production servers.

2. **Automated tests are run with with their own separate configuration** - we also call that the test environment. This includes a separate database and other settings like disabled emails and sandboxes for payments.

3. ==**All the needed test data is created during the test run.**== The test database is empty before test run starts, and is left empty after the test run. 

Having a separate test environment gives us a lot more liberty with what we can do within our tests. If we want to tests deletion of all user records, we can't possibly do that on a live database. 

#### The third point

The third point here is less convincing than the others. Why would we *not* want to pre-create test data? While performing manual tests, QA testers generally maintain some permanent data that is reused in re-runs. Why not do the same here? 

One reason is the need for predictability. **Automated tests should be run from a clear and predictable starting point**. A a completely clean state is a convenient starting point. It helps us write tests such as these -

    test "all created tickets are displayed" do
      10.times { create(:ticket }
      visit ticket_list_page
      page.should_have(10, :ticket)
    end

This test assumes that there are no tickets before it starts, and therefore confidently verifies the existence of all the records it creates. It is possible to verify just the newly created records, but this way the test remains simple and reliable.

Also sometimes creating test data gets difficult if there is pre-existing data. Like here:

    test "user registration" do
      sample_email = 'a@example.com' # should be unique
      post '/register', { email: sample_email }
      assert User.exists?(email: sample_email)
    end

The outcome of this test depends on whether the sample_email id already exists in the db. If we are not assured of a clean db, we'll have to check for that before starting this test. 

Another point to be noticed is that **all the data created during the test also needs to be deleted after test run finishes**. This is generally handled by the test frameworks we use, so we rarely have to worry about it.

#### A bit about fixtures

To be accurate, there is a popular technique in which test-data can pre-exist before a test case starts. Its called **fixtures** (which is also the default in Rails). When using fixtures, we define out test data in simple text files (as YAML in Rails). **All samples defined as fixtures are loaded to the database during the test run.** 

This means that while working with fixtures, both the above tests examples can't assume a zero sized database. This may look slightly less convenient, but is actually not much. The fixture data, being text, is available for developer inspection, and the test-cases can be planned accordingly (I'll use a `sample_email` that does not exist in fixtures).

And **with fixtures too, the test database is loaded only during test runs**. After the run finishes, test database becomes empty again. So from a test-run point of view, it is still a clean slate.

---

#### Final Thoughts

The art of automated testing contains many interesting patterns and techniques, sometimes rooted in philosophy. There are several schools of thought. And we are free to amend these techniques to what suits us. But its also **important to not reject them just because they sound puritanical** at first sight.

As Rails developers, we are fortunate to have most of these techniques built in our frameworks. Minitest and Rspec have a whole lot of differences, around technique and philosophy, but they are both competent - I have not found anything important that they miss.

Also, these things are not as hard to understand for a non-technical person - **we should also not shy away from bringing them up for discussion**. For one, such a discussion allows us question and understand our own end of the argument better. And at times, we will come across something *actually* missing or incorrect in our tooling - an opportunity to start interesting open source projects.






   


