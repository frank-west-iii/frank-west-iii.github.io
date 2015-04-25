---
title: Lesser Used Array Methods
layout: post
---

I am on a plane back from Atlanta after a week at RailsConf and in order to keep
up my commitment to my blog I am working my way through the offline copy of the
ruby core api docs.

I always have a good time reading through the api docs of ruby and discovering
methods or objects that I never knew existed. Today I am going to introduce a
couple of methods that you may not know about.

# permutation
When calling this method on an array it will return an enumerator with all of
the different permutations that can be made of that array. This can be useful to
calculate the number of unique ways you can line up your beer bottle collection.
It takes a single parameter which defines the number of elements to return in the
result set. It also  yields its results to a block if passed.

{% highlight ruby %}

# Passing in nothing or nil will use the size of the original array
[1,2,3,4].permutation.to_a
#=> [[1, 2, 3, 4], [1, 2, 4, 3], [1, 3, 2, 4], [1, 3, 4, 2], [1, 4, 2, 3], [1,4, 3, 2], [2, 1, 3, 4], [2, 1, 4, 3], [2, 3, 1, 4], [2, 3, 4, 1], [2, 4, 1, 3],[2, 4, 3, 1], [3, 1, 2, 4], [3, 1, 4, 2], [3, 2, 1, 4], [3, 2, 4, 1], [3, 4, 1,2], [3, 4, 2, 1], [4, 1, 2, 3], [4, 1, 3, 2], [4, 2, 1, 3], [4, 2, 3, 1], [4, 3,1, 2], [4, 3, 2, 1]]

[1,2,3,4].permutation(2).to_a
#=> [[1, 2], [1, 3], [1, 4], [2, 1], [2, 3], [2, 4], [3, 1], [3, 2], [3, 4], [4,1], [4, 2], [4, 3]]

(1..99).to_a.permutation.count
#=> Memory dump (ruby crashed)
{% endhighlight %}

# combination
This method is similar to the permutation in that it returns an enumerator and
takes a block, but will only return the unique combinations that can be made
with the given array . This method requires that you pass in an integer to
represent how many elements should be included in each set. This can be used to
calculate the number of possible combinations in a given lottery game perhaps.

{% highlight ruby %}
 [1,2,3,4].combination(2).to_a
 #=> [[1, 2], [1, 3], [1, 4], [2, 3], [2, 4], [3, 4]]

 [1,2,3,4].combination(3).to_a
 #=> [[1, 2, 3], [1, 2, 4], [1, 3, 4], [2, 3, 4]]

(1..35).to_a.combination(5).count
#=> 324632
{% endhighlight %}

# assoc & rassoc
This method expects an array of arrays each with at least 2 elements.
```[[1,2],[3,4],[5,6]]```. When matching elements in a "Dictionary" type
of data structure I have always used a hash map such as this:

{% highlight ruby %}
my_hash = {1 => 2, 2 => 3, 4 => 5}

# Find by key
my_hash[2]
# => 3

# Find by the value
my_hash.select {|k,v| v == 3 }
#=> {2 => 3}
{% endhighlight %}

I have found myself in the past performing a conversion of arrays to hashes
which then allows me to perform the same hash lookups.

{% highlight ruby %}
my_hash = [[1,2],[3,4],[5,6]].to_h
#=> {1 => 2, 2 => 3, 4 => 5}
{% endhighlight %}

However the assoc and rassoc methods perform the same basic function. Using
assoc, it will search through the array and find the array where the first
element matches the parameter.

{% highlight ruby %}
my_array = [[1,2],[3,4],[5,6]]
my_array.assoc(3)
#=> [3,4]
{% endhighlight %}

Using rassoc it searches through the arrays and matches on the second element
in the array to the parameter passed in.

{% highlight ruby %}
my_array = [[1,2],[3,4],[5,6]]
my_array.rassoc(4)
#=> [3,4]
{% endhighlight %}

If there are more than 2 elements in the array it will not search any other
elements beyond the second element.

{% highlight ruby %}
my_array = [[1,2,7],[3,4,8],[5,6,9]]
my_array.rassoc(9)
#=> nil
{% endhighlight %}

# rotate

The rotate method allows you to systematically move elements in the array by
telling the array which index point should be moved to the front of the array.

{% highlight ruby %}
my_array = [1,2,3,4,5,6,7,8,9]
my_array.rotate(4)
#=> [5, 6, 7, 8, 9, 1, 2, 3, 4]
{% endhighlight %}

# Until next time

The array and enumerable class are full of methods that you probably don't use
everyday, but probably solve everyday problems that you are running across and
don't realize there is a built in solution for you already.
