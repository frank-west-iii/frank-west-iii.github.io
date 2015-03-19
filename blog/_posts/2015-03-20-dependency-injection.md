---
title: Dependency Injection in Ruby
layout: post
---

As a .Net developer I found that the use of dependency injection (DI) was much more
prevalent in every day use than I have seen in Ruby. We had tools such as
Ninject, StructureMap and Unity. These made DI part of your everyday thinking
and development process.

When I started developing in Ruby I started like most people in the community
did, by developing Rails applications. Book after book, tutorial after tutorial,
I found no mention of DI anywhere. But of course I moved past that point and
thought that there was no need for this anymore in Ruby and there was another
solution. But alas, just like .Net, I have found that this is not true and I
found my need for DI to be just as strong in Ruby.

For simplicity sakes I would say that if you find yourself using a concrete
class inside another class, you may benefit from dependency injection.

### Without DI
{% highlight ruby %}
class B
  def call
    instance = A.new
    instance.do_something
  end
end
{% endhighlight %}

### With DI
{% highlight ruby %}
class B
  def call(instance)
    instance.do_something
  end
end
{% endhighlight %}

## Our application example
Let's get started with a simple example. Let's say you are building a system
that checks for files in a directory and processes them if they are there. When
the application is done processing the files it emails the administrator.

We need to implement the code that passes the files to a class responsible for
processing.

{% highlight ruby %}
class FileWatcher
  def call
    Dir['incoming/*.log'].each do |file|
      if file_processor.call(File.read(file))
        FileUtils.mv(file, 'completed')
      else
        FileUtils.mv(file, 'errors')
      end
    end
  end

  private

  def file_processor
    FileProcessor.new
  end
end
{% endhighlight %}

And the ```FileProcessor``` test and class might look like this:

{% highlight ruby %}
class FileProcessor
  def call(file)
    # Do some processing
    notifier.call
  end

  private

  def notifier
    EmailNotifier.new
  end
end
{% endhighlight %}

And lastly we have to implement the ```EmailNotifier``` class.

{% highlight ruby %}
class EmailNotifier
  def initialize(email)
    @email = email
  end

  def call
    # Send email
    true
  end
end
{% endhighlight %}

So if we look at these classes we have several classes that are being declared
within other classes.

### Software Changes

Now the requirements changed and we need to be able to send the notifications to
a dynamic list of email addresses specified by the user at runtime. What needs
to change in order for this to happen.

Let's go down the path of passing in the list of email addresses and see what
that gets us.

First we need to change the ```EmailNotifier``` class to accept and array of
email addresses.

{% highlight ruby %}
class EmailNotifier
  def initialize(emails)
    @emails = emails
  end

  def call
    @emails.each do |email|
      # Email someone
    end
  end
end
{% endhighlight %}

Ok, well that broke our tests for ```EmailNotifier``` and ```FileProcessor```.
First we fix the ```EmailNotifier``` tests and then we need to fix the
```FileProcessor``` class. The fix for the ```FileProcessor``` class is to allow
it to receive an array of emails to pass to the ```EmailNotifier```. So after
fixing the ```FileProcessor``` and tests the class may look something like this.

{% highlight ruby %}
class FileProcessor
  def initialize(emails)
    @emails = emails
  end

  def call(file)
    # Do some processing
    notifier.call
  end

  private

  def notifier
    EmailNotifier.new(@emails)
  end
end
{% endhighlight %}

But, now the tests for the ```FileWatcher``` are now broken. So let's fix that
by allowing it to also accept an array of emails to pass into the
```FileProcessor``` class.

{% highlight ruby %}
class FileWatcher
  def initialize(emails)
    @emails = emails
  end

  def call
    Dir['incoming/*.log'].each do |file|
      if file_processor.call(File.read(file))
        FileUtils.mv(file, 'completed')
      else
        FileUtils.mv(file, 'errors')
      end
    end
  end

  private

  def file_processor
    FileProcessor.new(@emails)
  end
end
{% endhighlight %}

In order to make that simple change we had to change every single class. This
breaks the Single Responsibility Principle (SRP) which states that a class
should have one and only one reason to change.

Now let's implement this with dependency injection and see what we we come up
with.

{% highlight ruby %}
class FileWatcher
  def initialize(file_processor)
    @file_processor = file_processor
  end

  def call
    Dir['incoming/*.log'].each do |file|
      if @file_processor.call(File.read(file))
        FileUtils.mv(file, 'completed')
      else
        FileUtils.mv(file, 'errors')
      end
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
class FileProcessor
  def initialize(notifier)
    @notifier = notifier
  end

  def call(file)
    # Do some processing
    @notifier.call
  end
end
{% endhighlight %}

{% highlight ruby %}
class EmailNotifier
  def initialize(email)
    @email = email
  end

  def call
    # Send email
    true
  end
end
{% endhighlight %}

As you can see we no longer have references to concrete classes inside our other
classes and instead the instances of those classes get injected in as a dependency
in the constructor. And we don't care what gets passed in as long as it implements
a call method or property.

And now we need to implement the same change as before and send an email to a
list of people specified by the user at runtime. The change becomes much easier
and truly only affects a single file (and test).

{% highlight ruby %}
class EmailNotifier
  def initialize(emails)
    @emails = emails
  end

  def call
    @emails.each do |email|
      # Send email
    end
    true
  end
end
{% endhighlight %}

These classes now more closely follow SRP and as you can now see the one change
only needed to be made in one class and did not spread throughout our
application making our application easir to change and adapt to future
requirements.

The code for this post can be found here:
<https://github.com/frank-west-iii/dependency-injection-2>

And another example can be found here:
<https://github.com/frank-west-iii/dependency-injection>


