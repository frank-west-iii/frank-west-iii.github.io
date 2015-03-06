---
title: Writing an API Wrapper
layout: post
---

When working with external APIs I have found that wrapping the API calls in a
class of their own not only cleans up the project, but simplifies it and makes
interacting with that API from within my application much more enjoyable.

When starting out you might be just use the Net::HTTP library and and simply
call out to the API directly within your code to perform a get request.

We can start with a simple class to hold the results.

{% highlight ruby %}
class Weather
  attr_reader :date, :temperature

  def initialize(date, temperature)
    @date = date
    @temperature = temperature
  end
end
{% endhighlight %}

And now to consume the api

{% highlight ruby %}
class HomeController < ApplicationController
  def index
    @current_forecast = current_forecast
  end

  private

  def current_forecast
    results = JSON.parse(Net::HTTP.get(
      URI "http://api.openweathermap.org/data/2.5/weather?q=Hanford, CA"))
    Weather.new(Time.now, results["main"]["temp"])
  end
end
{% endhighlight %}

Now what if we want to see the extended forecast as well.

{% highlight ruby %}
class HomeController < ApplicationController
  def index
    @current_forecast = current_forecast
    @extended_forecast = extended_forecast
  end

  private

  def current_forecast
    results = JSON.parse(Net::HTTP.get(
      URI "http://api.openweathermap.org/data/2.5/weather?q=Hanford, CA"))
    Weather.new(Time.now, results["main"]["temp"])
  end

  def extended_forecast
    results = JSON.parse(Net::HTTP.get(
        URI "http://api.openweathermap.org/data/2.5/forecast?q=Hanford,CA"))
    results["list"].each_with_object([]) do |item, result|
      list << Weather.new(Time.parse(item['dt_txt']),
                          item['main']['temp'])
    end
  end
end
{% endhighlight %}

This serves its purpose, but it misses out on several key factors. There
is duplication, there are no tests to cover these methods and there is no error
handling.

Plus having all of this inside our controller just makes the controller handle
more than it needs to and it violates SRP.

As a first stab at it we just need to extract this into its own class.

{% highlight ruby %}
require 'net/http'
require 'json'
require 'weather'

class OpenWeatherMap
  def initialize(location)
    @location = location
  end

  def current_forecast
    result = get 'weather'
    Weather.new(Time.now, result['main']['temp'])
  end

  def extended_forecast
    result = get 'forecast'
    result['list'].each_with_object([]) do |item, list|
      list << Weather.new(Time.parse(item['dt_txt']), item['main']['temp'])
    end
  end

  private

  def get(path)
    JSON.parse(Net::HTTP.get(URI "#{base_url}/#{path}?q=#{@location}"))
  end

  def base_url
    'http://api.openweathermap.org/data/2.5'
  end
end
{% endhighlight %}

We have removed the duplication which makes this code easier to reason about.
Now that we have this code in its own class it can now easily be unit tested. We
will implement testing with [rspec](http://rspec.info),
[timecop](https://github.com/travisjeffery/timecop)
and [webmock](https://github.com/bblimke/webmock).

{% highlight ruby %}
require 'open_weather_map'

RSpec.describe OpenWeatherMap do
  context "getting the current forecast" do
    it 'should return the current forecast when successful' do
        stub_request(:get, "http://api.openweathermap.org/data/2.5/weather?q=Hanford,CA").
          to_return(:status => 200, :body => {main: {temp: 295.39}}.to_json)

      Timecop.freeze(Time.now) do
        weather = OpenWeatherMap.new("Hanford,CA")
        result = weather.current_forecast

        expect(result.date).to eq(Time.now)
        expect(result.temperature).to eq(295.39)
      end
    end

  context "getting the extended forecast" do
    it 'should return the extended forecast when successful' do
      stub_request(:get, "http://api.openweathermap.org/data/2.5/forecast?q=Hanford,CA").
        to_return(:status => 200, :body => {list: [
                  {dt_txt: '2015-03-06 03:00:00',
                    main: {temp: 298.32}},
                    {dt_txt: '2015-03-06 06:00:00',
                      main: {temp: 302.72}}]}.to_json)

      weather = OpenWeatherMap.new("Hanford,CA")
      results = weather.extended_forecast

      expect(results.count).to eq(2)
      expect(results.first.temperature).to eq(298.32)
      expect(results.first.date).to eq(Time.parse('2015-03-06 03:00:00'))
      expect(results.last.temperature).to eq(302.72)
      expect(results.last.date).to eq(Time.parse('2015-03-06 06:00:00'))
    end
  end
end
{% endhighlight %}

Now we can go back to our controller and remove all of the unnecessary code that
does not belong there and has been refactored out.

{% highlight ruby %}
class HomeController < ApplicationController
  def index
    @current_forecast = current_forecast
    @extended_forecast = extended_forecast
  end

  private

  def current_forecast
    open_weather_map.extended_forecast
  end

  def extended_forecast
    open_weather_map.extended_forecast
  end

  def open_weather_map
    OpenWeatherMap.new('Hanford,CA')
  end

{% endhighlight %}

To me this controller code looks much better. It no longer knows how to get
weather information. It only knows about the OpenWeatherMap class and leaves
that knowledge to that class instead. The controller is now focused on just
displaying weather information.

We have a fairly good foundation to start with. Next week we will look at the
unhappy paths and see how we can further refine this class to be resilient
to failures or unexpected results.
