---
title: Writing an API Wrapper - Part 2
layout: post
---

Last blog post we left off with having refactored out our api call into its own
class which made testing much easier. It also allowed us to simplify our
controller code and follow SRP a little more closely.

But we were not dealing with exceptions and there were a few little tweaks I
wanted to make to the class in order to make working with the response a little
later.

## Bring on Hurley

I am a huge fan of the octokit.rb project. When I was learning to write gems,
this is the project I read through in order to understand how to write a client
API gem.

The project uses a couple of gems, but the underlying gem that is used for
communication is [Faraday](https://github.com/lostisland/faraday). Faraday is a
HTTP client that allows you to implement middleware on top of it. This provides
you the ability to affect every request before it leaves your application fully.

More recently they have moved away from Faraday and have started anew with a
project called [Hurley](https://github.com/lostisland/hurley). This project has
some of the same goals of Faraday, but with different inner workings. Their
GitHub page states that this project is an evolution of Faraday.

## Implementing Hurley into the api wrapper

The great thing about adding something like Faraday or Hurley to your api
wrapper is that it has some built in ways to deal with unsuccessful api calls.
It implements methods such as:

{% highlight ruby %}
success?
redirection?
client_error?
server_error?
{% endhighlight %}

So with these methods we can easily change out our plain NET::HTTP.get calls to
a Hurley.get call and have some baked in error handling (at least from the
response point of view). Now we can add a test and stub out a 500 error and
ensure that we get an Unknown Temperature.

{% highlight ruby %}
it 'should return the unknown forecast when not successful' do
  stub_request(:get, "http://api.openweathermap.org/data/2.5/weather?q=Hanford,CA").
    to_return(:status => 500, :body => {}.to_json)

  Timecop.freeze(Time.now) do
    weather = OpenWeatherMap.new("Hanford,CA")
    result = weather.current_forecast

    expect(result.date).to eq(Time.now)
    expect(result.temperature).to eq('Unknown')
  end
end
{% endhighlight %}

Now let's look at what this would look like when we implement it in the api
wrapper class.

{% highlight ruby %}
class OpenWeatherMap
  def initialize(location)
    @location = location
  end

  def current_forecast
    get 'weather' do |response|
      Weather.new(Time.now,
                  response['main']['temp'])
    end
  end

  def extended_forecast
    #...
  end

  private

  def get(path, &block)
    response = client.get("/data/2.5/#{path}?q=#{@location}")
    if response.success?
      yield JSON.parse(response.body)
    else
      UnknownWeather.new(Time.now)
    end
  end

  def client
    client = Hurley::Client.new base_url
    client.header[:content_type] = "application/json"
    client
  end

  def base_url
    'http://api.openweathermap.org'
  end
end
{% endhighlight %}

We have introduced a block into our get method so that if our request is
successful we can pass the parsed results back to the calling method so it can
further process the results. We are also now checking whether or not the
response from our request was successful or not and return a null object called
UnknownWeather. Unknown weather is a simple ruby class that looks like this:

{% highlight ruby %}
class UnknownWeather
  attr_reader :date

  def initialize(date)
    @date = date
  end

  def temperature
    "Unknown"
  end
end
{% endhighlight %}


## One more item

There is one more thing that I like to do to my api responses when I get them
back to make working with them even easier and cleaner. However I should say
that this has some caveats to it which I will discuss after introducing the
idea.

I like wrapping my responses in a Hashie::Mash simply because I enjoy dealing
with the dot notation rather than the string hash key notation that JSON.parse
gives me. [Hashie::Mash](https://github.com/intridea/hashie) is a part of the
hashie library. The library adds on additional functionality when dealing with
hashes. Mash is the part of the library that allows us to convert a hash into an
object that acts like a regular object. So let's implement this. 

{% highlight ruby %}
require 'hurley'
require 'json'
require 'weather'
require 'hashie'

class OpenWeatherMap
  def initialize(location)
    @location = location
  end

  def current_forecast
    get 'weather' do |response|
      Weather.new(Time.now,
                  response.main.temp)
    end
  end

  def extended_forecast
    result = get 'forecast' do |response|
      response.list.each_with_object([]) do |item, list|
        list << Weather.new(Time.parse(item.dt_txt),
                            item.main.temp)
      end
    end
    Array(result)
  end

  private

  def get(path, &block)
    response = client.get("/data/2.5/#{path}?q=#{@location}")
    if response.success?
      yield Hashie::Mash.new(JSON.parse(response.body))
    else
      UnknownWeather.new(Time.now)
    end
  end

  def client
    client = Hurley::Client.new base_url
    client.header[:content_type] = "application/json"
    client
  end

  def base_url
    'http://api.openweathermap.org'
  end
end
{% endhighlight %}

This is just the tip of the iceberg when it comes to building api clients with
ruby. I write at least one api client every month and am constantly learning
how to improve these wrappers. I have written simple ones like this one and ones
inspired by octokit.rb. Hopefully I can be able to revisit how to wrap this up
into a nice gem in the near future.

The repo can be found [here](https://github.com/frank-west-iii/openweathermap).
