---
layout: doc_modules
title: Ratelimit module
---
# Ratelimit plugin

Ratelimit plugin is designed to limit messages coming from certain senders, to
certain recipients from certain IP addresses combining these parameters into
a separate limits.

All limits are stored in [redis](http://redis.io) server (or servers cluster) to enable
shared cache between different scanners.

## Module configuration

In the default configuration, there are no cache servers specified, hence, the module won't work unless you add this option to the configuration.

`Ratelimit` module supports the following configuration options:

- `servers` - list of servers where ratelimit data is stored
- `whitelisted_rcpts` - comma separated list of whitelisted recipients. By default
the value of this option is 'postmaster, mailer-daemon'
- `whitelisted_ip` - a map of ip addresses or networks whitelisted
- `max_rcpts` - do not apply ratelimit if it contains more than this value of recipients (5 by default). This
option allows to avoid too many work for setting buckets if there are a lot of recipients in a message).
- `max_delay` - maximum lifetime for any limit bucket (1 day by default)
- `rates` - a table of allowed rates in form:

    type = [burst,leak];

Where `type` is one of:

- `to`
- `to_ip`
- `to_ip_from`
- `bounce_to`
- `bounce_to_ip`

`burst` is a capacity of a bucket and `leak` is a rate in messages per second.
Both these attributes are floating point values.

- `symbol` - if this option is specified, then `ratelimit` plugin just adds the corresponding symbol instead of setting pre-result, the value is scaled as $$ 2 * tanh(\frac{bucket}{threshold * 2}) $$, where `tanh` is the hyperbolic tanhent function

## Principles of work

The basic principle of ratelimiting in rspamd is called `leaked bucket`. It could
be visually represented as a bucket that has some capacity, and a small hole in a bottom.
Messages comes to this bucket and leak through the hole over time (it doesn't delay messages, just count them). If the capacity of
a bucket is exhausted, then a temporary reject is sent. This happens unless the capacity
of bucket is enough to accept more messages (and since messages are leaking then after some
time, it will be possible to process new messages).

Rspamd uses 3 types of limit buckets:

- `to` - a bucket based on a recipient only
- `to:ip` - a bucket combining a recipient and a sender's IP 
- `to:from:ip` - a bucket combining a recipient, a sender and a sender's IP

For bounce messages there are special buckets that lack `from` component and have more
restricted limits. Rspamd treats the following senders as bounce senders:

- 'postmaster',
- 'mailer-daemon'
- '' (empty sender)
- 'null'
- 'fetchmail-daemon'
- 'mdaemon'

Each recipient has its own triple of buckets, hence it is useful
to limit number of recipients to check.

Each bucket has two parameters:
- `capacity` - how many messages could go into a bucket before a limit is reached
- `leak` - how many messages per second are leaked from a bucket.

For example, a bucket with capacity `100` and leak `1` can accept up to 100 messages but then
will accept not more than a message per second.

By default, ratelimit module has the following settings which disable all limits:

~~~lua
-- Default settings for limits, 1-st member is burst, second is rate and the third is numeric type
local settings = {
  -- Limit for all mail per recipient (burst 100, rate 2 per minute)
  to = {0, 0.033333333},
  -- Limit for all mail per one source ip (burst 30, rate 1.5 per minute)
  to_ip = {0, 0.025},
  -- Limit for all mail per one source ip and from address (burst 20, rate 1 per minute)
  to_ip_from = {0, 0.01666666667},

  -- Limit for all bounce mail (burst 10, rate 2 per hour)
  bounce_to = {0, 0.000555556},
  -- Limit for bounce mail per one source ip (burst 5, rate 1 per hour)
  bounce_to_ip = {0, 0.000277778},

  -- Limit for all mail per user (authuser) (burst 20, rate 1 per minute)
  user = {0, 0.01666666667}
}
~~~
