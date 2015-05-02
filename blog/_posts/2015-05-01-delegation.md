---
title: Delegation
layout: post
---

I have recently been building an application and find myself really finding joy
in the in making sure that my models and controller are as dead simple as
possible.

For my current project I have been implementing the presenter pattern along with
the delegation pattern. In this post I am going to talk about the delegation
pattern and how it can easily be accomplished and some tools that may be of use
to you in you design.

## First Dive

At first when I was looking to solve this issue I did what most ruby developers
did and decided there had to be a gem for it. I searched and I found draper. So
off to the races I went. It is a dead simple gem to get started with and also
includes generators, model decorators and even collection decorators. So you
might have a decorator that looks like this:

{% highlight ruby %}
class BrewingSessionDecorator < Draper::Decorator
  delegate_all

  def brew_date
    object.brew_date.strftime('%m/%d/%Y')
  end
end
{% endhighlight %}

And to use it you would simply call ```BrewingSession.find(id).decorate```.

See simple... and it even decorates associated models, it solves that for you
by just adding one simple line: ```decorates_association :brewer```. There you
have it. You can now access the brewer instance from the brewing session and
ensure that the decorated brewer instance is available to you as well to use in
your view.

## The other side of the coin

### Dependencies

In my opinion the opportunity to delete gems out of my Gemfile is a very
satisfying feeling. I have lived through the pain point of upgrading plugins
from Rails 2.0 to Rails 3.0 and then migrating applications from Rails 3 to
Rails 4. Each migration had issues with compatibility issues from one version to
the next when it came to the gems or plugins and then finding out that the gem
or plugin does not have a maintainer or port to the new version. Argh. The less
dependencies on outside code the better.

### Coupling

Then there is that cool feature of draper that allows you to decorate associated
models within a decorator. Since they make this so easy to use, it is also easy
to ignore the fact that this most likely leads to code that violates the Law of
Demeter. Oops.

## Possible Solution

There is a much easier way to execute this and protect ourselves in the process.
Ruby provides a much simpler class to solve this problem. The class is called
```SimpleDelegator```.

To implement the same code from above you can just create a class that derives
from SimpleDelegator.

{% highlight ruby %}
require 'delegate'
class BrewingSessionDecorator < SimpleDelegator
  def brew_date
    super.brew_date.strftime('%m/%d/%Y')
  end
end
{% endhighlight %}

This has beauty in its simplicity as it only depends on a core ruby class and
moving versions of ruby or rails for that matter should not create a breaking
change here without being well documented from the core ruby team.

### What about our associated models?

As you probably noticed our new implementation does not have a
```decorates_association``` method for our brewer. And to be honest that is a
good thing in my opinion. I could implement it myself, but this just led me to
believe that there really was no reason to do this at all and maybe I was doing
it wrong in the first place.

My solution to this problem was to start implementing the presenter pattern.
Next week I will dive into how I use the presenter pattern to solve my problems
with the Law of Demeter.
