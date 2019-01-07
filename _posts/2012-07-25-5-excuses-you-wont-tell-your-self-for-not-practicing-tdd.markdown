---
layout: post
title: "5 excuses you won't tell your self for not practicing TDD"
date: 2012-07-26 08:36
comments: true
published: true
featured: false
tags: builders TDD BDD test appsec security-by-design design architecture
hn: http://news.ycombinator.com/item?id=4294771
rd: http://redd.it/x6fw9
---

IT world **do** is a complex world. There are a lot of different people having
their own vision of the world, each of them with their own respectable opinion
about hot to write great software.

Problems arise when people lie to themselves about testing, or best to say when
people lie to themselves about the reason they do not practice testing.

<!-- more -->

## Excuse #1: "There is no time for testing"

Look at this scenario. Your customer forces you and your time with a tight
roadmap in order to hit the market in the next month with the next killing
website. No excuses, just the business.

You and your team look each other saying _"we will never hit the deadline, but
we don't want to disappoint the customer. Let's do it."_ and then you start
hacking over your IDEs or text editor.

You read the specs or at least you look at UI/UX designer mockups and then you
start writing code.
Your HTML/JS ninja will write the UI code and then you plug your business logic
code into it.

At least twice a day, your UI/UX designer change the UI and since you, bunch of
ninjas, are using a great [MVC controller framework](http://rubyonrails.org)
you are willing to adopt the changes without _too much_ work to be done.

You hit the deadline. You go _online_. First bugs are there when users submit
some forms. You fix it over the production code, since you don't have time to
build a staging environment. Then an attacker arrives, SQL injecting your
backend and stealing all credit card number in your shop.

Your users are angry. Your customer **do** is angry. You're in trouble.

{% blockquote %}
You migth say, hey but you don't leave us enough time to test the code, we
don't have resource, it's not our fault.
{% endblockquote %}

Your customer will eventually say that it is your fault. And you must fix it.
For free.

### You may want to
Talk to your customer honestly. Say her you can't bring a full featured
ecommerce website in a couple of weeks. Say her you have to test the code in a
staging environment for security issues she must provide you. 

When she doesn't understand why the hell people should search your site for 
``` sql
' OR '1'='1
``` 
explain that she must take care about her customers data and you want to help
making the website robust and secure to attackers.

May be you want to suggest going online with 2 or 3 less important feature you
would deploy a couple of week after the launch but since you're a professional
you do testing.

## Excuses #2: "My customer doesn't know what she needs"

So you're telling you and your team are engaged to write a mysterious web
application that both you and your committer doesn't know the problem it has to
solve? And you're in rush in writing this?

Really?

If anybody knows what the web application has to do, how can your customer say
_"hey, you performed very well, you deserved to be paid now"_?


### You may want to
Again, solve this problem talking to your customer. He makes bicycle hardware
and he wants to be on the web? Explain him the problems and the opportunties to
be online but you must be very proactive.

You are the expert in the web field. You must suggest timings and which
features to enable in which project lifespan time frame.

People are more keen to understand and to let real expert to drive them in a
successfully experience rather than you expect.

So write down use cases with your customer, you want to start from which is the
expected behavior or the expected user experience from the website. 

Make those scenarios the real contract and be paid in writing a web application
fulfilling your costumer _desiderata_.

## Excuse #3: "Can't write tests since specs are changing"

This is particular subclasses for the excuse number 2. If your customer do
change too much oftne idea maybe it's because she doesn't have a clear strategy
for her website.

Maybe she's that kind of anxious never confident about their own idea and she
spent hours over the Internet choosing _at the right language_ or _at the right
look and feel_ you can copycat in her product.

This doesn't work either.

### You may want to
Be a psychologist here. Talk to your customer saying the first idea she had was
so great and you're pretty sure changing the vision during the development will
slowdown the process. Lags will turn in loss of money, so it's better not
focusing in whistles but build a strong and robust website you can always
improve later on.

## Excuse #4: "Can't write tests before the code dude, where are you coming from?"
This is ery easy and very common. You don't know what
[TDD](http://en.wikipedia.org/wiki/Test-driven_development) is and you believe
you have to write tests that proves your code is rock solid after you develop
your code. This means that you write your test just to let see them to pass.

The idea here is that first you model the expected behavior, the one you agreed
with your customer, the one you will be paid for. And **then** you write the
code that it will go in production making all the tests to pass.

So, I know it may sound weird but you write first test, then you write code and
then you execute tests making them to pass adjusting the code accordingly.

### You may want to
Maybe you may want to [read](http://pragprog.com/book/achbd/the-rspec-book)
[some](http://pragprog.com/book/hwcuc/the-cucumber-book)
[books](http://www.amazon.com/dp/1933988274) about TDD,
[BDD](http://en.wikipedia.org/wiki/Behavior-driven_development) and Testing.

Reading and improve yourself as a software craftman may sounds good for you
carreer.

## Excuse #5: "We don't test the code"
You know, sometimes it happens. You meet someone who knows it all and that he
pretend to have all the answers.

People like this, make all their teammate to believe their doesn't know the
meaning of the word _fault_. 

Software has no bugs for them.
They are not supposed to test or to review their code for security issues since
they are **the best**.

Their code is almost perfect and it is always delivered in time.

As you may argue we're talking about failing people. They are not software
craftmen, instead they are yes-men saying exactly what the customer want to
hear. "We deliver in time", "We build strong software" but they saw those
phrases without any clue about how to achieve those goals.

### You may want to
If your team leader is one of those people, maybe you want to talk to him
and... mmmh... no maybe you have to be strong and let the hurricane to pass and
then look for another job in a smarter and more skilled team. 

Maybe you deserve it.

## Wrap up
Talking is important. As software engineer you must carefully explain to not
skilled people what's behind the scene not using technical words.

You must explain why testing it is so important.

You must read [armoredcode.com](http://armoredcode.com) blog and make sure to
understand application security topics.
Never give up on talking. This is the rule.

## Customers suck
Final word about customers. True to be told, this post doesn't fit any real
world situation. There are customers who don't want to listen any words about
testing, or security or whatever. 

They pay, you commit. And you're already late also if you signed with them
today.

I don't have the answer. If talking to your customer trying to spread awareness
doesn't work.. just keep calm...

![]({{site.url}}/images/online.png)
