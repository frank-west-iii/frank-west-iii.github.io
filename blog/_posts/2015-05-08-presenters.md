---
title: Presenters
layout: post
---

I have been building an application which implements a dashboard for a user. The
user owns several entities such as roasting profiles, roasting sessions and
friends. When a user logs into the application they are taken to their dashboard
which displays various bits of information about all of these items. So we may
implement this in several ways.

{% highlight ruby %}
class DashboardController < ApplicationController

  def index
    @user = User.find(params[:user_id])
   end
end
{% endhighlight %}

And the in our views we would have access to the user's friends, profiles and
sessions all from the user object. This is the quick and easy way to do it. It
will put you into the situation where you will be violating the Law of Demeter,
but that is a personal preference for each developer. I personally try to avoid
it where possible. So I have started to use presenter models that help me solve
this.

My newer code will look more like this instead:

{% highlight ruby %}
class Dashboard
  attr_reader :user, :roast_profiles, :roasting_sessions, :friends

  def initialize(user, roast_profiles, roasting_sessions, friends)
    @user = user
    @roast_profiles = roast_profiles
    @roasting_sessions = roasting_sessions
    @friends = friends
  end
end

class DashboardController < ApplicationController

  def index
    @dashboard = dashboard
   end

   private

   def dashboard
     Dashboard.new(user, user.roast_profiles, user.roasting_sessions,
       user.friends)
   end

   def user
     @user ||= User.find(params[:user_id])
   end
end
{% endhighlight %}

Admittingly at first glance this seems overly complicated. However I do this so
that my view can remain decoupled from my user implementation details. My view
doesn't need to care where the list of roast_profiles comes from, just that it
exists.

If I need a specific representation of a friend within the dashboard this
pattern allows me to manipulate the friend collection within the context of the
dashboard only. I don't have to put any logic inside the view or inside the
model or controller.

This also allows you to solve one of Sandi Metz's rules that a controller can
only instantiate one object. If I normally instantiate a ```@user``` and also a
```@messages```, I could easily create a single object that gets instantiated
with a user and a collection of messages.

As your codebase grows more complex and needs to changes, having less places
where your code is coupled together makes it easier to change in the long run.
In places where you are passing multiple objects into the view this also lets
you resolv
