---
title: OpenStruct
layout: post
---

In the post last week we looked at the ```Struct``` class. This week I am going
to introduce the ```OpenStruct``` class.

OpenStruct is very similar to a hash in ruby and in fact under the hood it
simply is just a hash with some metaprogramming on top to allow for dynamic
definition of attributes on the object. Underneath the covers the OpenStruct object
defines a hash named table and attributes are dynamically added to or
retrieved from this hash.

## Defining

An OpenStruct class can be declared simple by requiring the 'ostruct' library and
then declare a variable as a new OpenStruct.

{% highlight ruby %}
pasta = OpenStruct.new

pasta
    #-> #<OpenStruct>
{% endhighlight %}

A new OpenStruct instance can be initialized by passing a hash, Struct, or
OpenStruct into the initializer. It will take every key/value pair and turn
it into an attribute on the new instance.

{% highlight ruby %}
# Initialized with a hash
pasta = OpenStruct.new(name: 'Pasta Batman', breed: 'Beagle', age: 5)

# Initialized with another OpenStruct instance
my_open_struct = OpenStruct.new(name: 'Pasta Batman', breed: 'Beagle', age: 5)
pasta = OpenStruct.new(my_open_struct)

# Initialized with a Struct
Dog = Struct.new(:name, :breed, :age)
pasta = OpenStruct.new(Dog.new('Pasta Batman', 'Beagle', 5))

# All result in the same OpenStruct representation
    #-> #<OpenStruct name="Pasta Batman", breed="Beagle", age=5>

{% endhighlight %}

Passing in nested hashes will result in the top level attribute having a hash as
a value. Simply meaning it will not recursively define OpenStruct instances beyond
the first level.

{% highlight ruby %}
pasta = OpenStruct.new(name: 'Pasta Batman', breed: 'Beagle', age: 5,
  owner: {name: 'Frank', state: 'CA'})

    #-> #<OpenStruct name="Pasta Batman", breed="Beagle", age=5, owner={:name=>"Frank", :state=>"CA"}>
{% endhighlight %}

## Getting and setting members

The members are accessible in the same way as the Struct class except that
they cannot be accessed via an index operator.

{% highlight ruby %}
pasta.name
    #-> "Pasta Batman"

pasta['name']
    #-> "Pasta Batman"

pasta[0]
    #-> NoMethodError: undefined method `to_sym' for 0:Fixnum

{% endhighlight %}

Also just like a Struct the member names can be set using these method as well.

{% highlight ruby %}
pasta.name = 'Banana'
pasta.name
    #-> "Banana"
pasta['name'] = 'Pasta'
pasta.name
    #-> "Pasta"
{% endhighlight %}

Attributes can be dynamically added to the OpenStruct instance by simply setting
the value of any attribute. When an attribute is set that is not already defined
inside the internal hash table, OpenStruct adds the new attribute as a key and
sets the value to the value that you passed in.

{% highlight ruby %}
pasta
    #-> #<OpenStruct name="Pasta Batman", breed="Beagle", age=5>

pasta.tail = true
    #-> #<OpenStruct name="Pasta Batman", breed="Beagle", age=5, tail=true>
{% endhighlight %}

You are also able to delete fields from your OpenStruct instance. It provides a
method named delete_field, which simply removes the key from the internal
hash.

{% highlight ruby %}
pasta
    #-> #<OpenStruct name="Pasta Batman", breed="Beagle", age=5, tail: true>

pasta.delete_field 'tail'
    #-> #<OpenStruct name="Pasta Batman", breed="Beagle", age=5>
{% endhighlight %}


Unlike Struct the OpenStruct does not include the enumerable module,
but does define the enumeration method each_pair which enumerates through
the internal hash table.

{% highlight ruby %}
pasta.each_pair {|k,v| puts "#{k}: #{v}" }
    #-> name: Pasta Batman
    #-> breed: Beagle
    #-> age: 5
{% endhighlight %}

## Undefined methods

OpenStruct differs from Struct when it comes to undefined methods. Since
it is simply a hash underneath the covers it acts just like a hash would
when you access an unknown key. It returns nil.

{% highlight ruby %}
pasta.owner
    #-> nil
{% endhighlight %}

This can be a double edged sword and one of the main reasons why I think twice
before reaching to use these in my programming. I have found use for them and in
my next post I will put both the Struct and OpenStruct to use in a practical
example.
