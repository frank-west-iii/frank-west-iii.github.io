---
title: Struct Usages
layout: post
---

# Putting the (Open)Struct classes to use

In the last few posts I went over the ```Struct``` and ```OpenStruct``` classes
and the basics of how they work. I have used both of these classes in the past
and have found use cases for both of them.

## API wrappers using OpenStruct

In both cases I have found it very useful to use these classes when wrapping
APIs.  When I am dealing with an unknown API, I would first start out by using
an OpenStruct and creating a wrapper class with the following code:

{% highlight ruby %}
module UnknownApi

  class Base < OpenStruct
    include HTTParty

    base_uri ENV['UNKNOWN_API_URL']

    class << self

      def auth
        {username: ENV.fetch('UNKNOWN_API_USERNAME'),
          password: ENV.fetch('UNKNOWN_API_PASSWORD')}
      end

      def options
        {body: parameters,
          headers: {'ContentType' => 'application/json',
          'Accept' => 'application/json'}, basic_auth: auth}
      end

      def get(url)
        super(url, options)
      end

      def convert_to_hash(response)
        return nil if response.nil?
        Hash[response.map { |k, v| [k.underscore.to_sym, v.is_a?(Array) ?
                              v.map { |a| convert_to_hash(a) } : v] }]
      end

    end

    private_class_method :convert_to_hash, :auth, :get, :options

  end

end
{% endhighlight %}

And then a class that inherits the base api class:

{% highlight ruby %}
module UnknownApi
  class Orders < UnknownApi::Base
    class << self
      def get(order_id)
        response = get("/orders/#{order_id}")
        if response.success?
          return self.new(response)
        else
          raise Exception.new(response)
        end
      end
    end
  end
end
{% endhighlight %}

### Benefits

The benefits of this is that until I understand the API fully this will
automatically wrap my results in an OpenStruct class and I can play with the
API without having to map each of the values to a class attribute.

This is also very useful when you are developing alongside a pre-release API
that is constantly changing. It reduces the amount of change that you have to
perform within your code until the API stabilizes.

### Downsides

The downsides of using this approach is that if you rely on values that are in
the API and those fields disappear then you will be left scratching your head,
because instead of throwing an exception, you will now be returning nil and in
some cases nil could have been a valid value for some calls, but not for all
calls which is now occurring.

## API wrappers using Struct

After I have worked my way through the API and/or the API has stabilized I move
my return values to a Struct instead. This makes the code relying on the API
more stable. The following code would be what I would transform my API wrapper
into:

{% highlight ruby %}
module KnownApi

  class Base
    include HTTParty

    base_uri ENV['KNOWN_API_URL']

    class << self

      def auth
        {username: ENV.fetch('KNOWN_API_USERNAME'),
          password: ENV.fetch('KNOWN_API_PASSWORD')}
      end

      def options
        {body: parameters,
          headers: {'ContentType' => 'application/json',
          'Accept' => 'application/json'}, basic_auth: auth}
      end

      def get(url)
        super(url, options)
      end

    end

    private_class_method :auth, :get, :options

  end

end
{% endhighlight %}

And we would reimplement our order class like so:

{% highlight ruby %}
module KnownApi
  class Orders < KnownApi::Base
    class << self

      Order = Struct.new(:id, :customer, :address, :city, :state, :zip)

      def get(order_id)
        response = get("/orders/#{order_id}")
        if response.success?
          return Order.new(response['id'], response['customer'],
            response['address'], response['city'], response['state'],
            response['zip'])
        else
          raise Exception.new(response)
        end
      end
    end
  end
end
{% endhighlight %}

### Performance

We can't ignore that there is performance implications of choosing one of these
structures over the other. The clear winner on my early 2013 MacBook Pro with
ruby 2.2.0p0 is the struct:

{% highlight ruby %}
require 'benchmark'
require 'ostruct'

LOOP_COUNT = 100000

Order = Struct.new(:id, :customer, :address, :city, :state, :zip)

Benchmark.bm 20 do |x|
  x.report 'OpenStruct' do
    LOOP_COUNT.times do |index|
       OpenStruct.new({id: 42, customer: 'customer',
          address: 'address', city: 'city', state: 'state',
          zip: 'zip'})
    end
  end

  x.report 'Struct' do
    LOOP_COUNT.times do |index|
       Order.new(42, 'customer', 'address', 'city', 'state', 'zip')
       Order.new()
    end
  end
end

{% endhighlight %}


{% highlight ruby %}
.                          user     system      total        real
OpenStruct             3.420000   0.010000   3.430000 (  3.429038)
Struct                 0.120000   0.010000   0.130000 (  0.127545)

{% endhighlight %}

That is a very large difference in performance and when you are in a high
transactional system, this can be a very performace gain by ensuring that you
are using the Struct instead of the OpenStruct.
