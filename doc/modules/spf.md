---
layout: doc_modules
title: SPF module
---
# SPF module

SPF module performs checks of the sender's [SPF](http://www.openspf.org/) policy.
Many mail providers uses SPF records to define which hosts are eligible to send email
for this specific domain. In fact, there are many possibilities to create and use
SPF records, however, all they check merely the sender's domain and the sender's IP.

The specific case are automated messages from the special mailer daemon address:
`<>`. In this case Rspamd uses `HELO` to grab domain information as specified in the
standard.

## Principles of work

`SPF` can be a powerful tool when properly used. However, it is very fragile in many
cases: when a message is somehow redirected or reconstructed by mailing lists software.

Moreover, many mail providers have no clear understanding of this technology and
misuse the SPF technique. Hence, the scores for SPF symbols are relatively small
in Rspamd.

SPF uses DNS service extensively, therefore Rspamd maintain the cache of SPF records.
This cache operates on principle of `least recently used` expiration. All cached item
lifetimes are accordingly limited by the matching DNS record time to live.

You can manually specify the size of this cache by configuring SPF module:

~~~ucl
spf {
	spf_cache_size = 1k; # cache up to 1000 of the most recent SPF records
}
~~~

Currently, Rspamd supports the full set of SPF elements, macros and has internal
protection from DNS recursion.
