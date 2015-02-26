---
title: Retryable Module
layout: post
---

Sometimes there are scenarios where exceptions can occur in your code and
simply retrying the operation will allow the operation to succeed.

A scenario where I often run into this is when consuming an external service.
Your application is at the mercy of that service and I have found that the majority
of outages when dealing with an external service are very temporary and could be
handled by simply retrying the call again.

In order to mitigate this problem we can programmatically retry the same call as
many times as we deem necessary in order to achieve a better experience where the
outage is temporary.

To achieve this we first create a module called Retryable:

{% highlight ruby %}
module Retryable
  def perform_retry(retry_count = 3, backoff_seconds = 0.25,
                    backoff_multiplier = 2, &block)
    yield
  rescue
    retry_count -= 1
    raise if retry_count < 0
    sleep backoff_seconds
    backoff_seconds *= backoff_multiplier
    retry
  end
end
{% endhighlight %}

This module implements the keyword ```retry``` which allows us to retry our code
when a rescure has been performed from the nearest begin block. In this case
since no explicit begin block was supplied it will retry from the beginning of
the method. We are also careful to perform an incremental backoff to ensure that
we do not flood the service by continually calling the service in a loop.

We can now make use this in any of our classes by just including this module and
wrapping any code we want to retry in a ```perform_retry``` block.

{% highlight ruby %}
class UnreliableService
  include Retryable

  def initialize
    @client = UnreliableService::Client.new do |config|
      config.token = ENV.fetch('UNRELIABLE_SERVICE_TOKEN')
     end
  end

  def accounts
    perform_retry do
      @client.accounts("frankwestiii")
    end
  end
end
{% endhighlight %}

Doing this allows us to handle those outliers in a more graceful way instead of
failing immediately.

A sample tested implementation of this can be found here:
[frank-west-iii/queue-retry](https://github.com/frank-west-iii/queue-retry)
