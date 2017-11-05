---
title: AWS Route53 - DNS Whitelisting using Geolocation Routing
date: 2017-10-07 14:41:25
tags:
- Route53
- Geolocation
- AWS
- Client Subnet
- DNS Blacklist
---

Assumptions:  Basic knowledge of AWS Route53


DNS can be a powerful tool, more so if you are using AWS Route53 as your provider.

In this post I will focus on how to use Route53's [Geolocation routing](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-geo) as a whitelist / blacklist of users originating from around the world.

So, let's assume you only want users from the United Kingdom to access your website. This could be because all your customer base and operations are UK based, or maybe your application was DDoS'ed from various countries around the world, and you want to make sure any DNS query issued from this malicious Origin / IP is null routed.

First, let's go to Route53 and create our first record set :

![Null Route](https://res.cloudinary.com/danielcondemarin/image/upload/v1507497187/nullrouteset.png)


Please note this will be our default route, which will resolve to the IP Address [0.0.0.0](https://en.wikipedia.org/wiki/Null_route). So, whenever a user attempts to reach your website domain (i.e. mywebsite.somedomain.co.uk) Route53 will resolve that DNS Query to `0.0.0.0`, then your machine will attempt to reach the website via the IP 0.0.0.0 which obviously doesn't exist and will fail.

Let's setup now the record for users originating from the UK and assume our webserver will be hosted on a server with IP `54.12.12.12` :

![UK Route](https://res.cloudinary.com/danielcondemarin/image/upload/v1507497187/ukrouteset.png)

That should be it, the A records just created along with the domain NS records should be listed in your website Hosted Zone like this:

![Hosted Zone](https://res.cloudinary.com/danielcondemarin/image/upload/v1507497187/hostedzone.png)

But how do we test this is working correctly? It should be easy be using `dig` via your terminal. Just grab one of the nameservers for your domain, listed above, and run the following command :

`./bin/dig/dig @your-nameserver mywebsite.somedomain.co.uk +client=45.63.111.158/24`

Caveat : You will need a [patched version](https://www.gsic.uva.es/~jnisigl/dig-edns-client-subnet.html) of `dig` in order to use the `+client` option. This option allows you to tell the nameserver the user ip address, which then it uses to return an IP Address from your records, based on the estimated geolocation of the IP Address you passed. This mainly works thanks to the [client subnet in DNS queries](https://tools.ietf.org/html/rfc7871). 

![DNS Query with American IP Address](https://res.cloudinary.com/danielcondemarin/image/upload/v1507500087/usdnsquery.png)

I used the IP Address `45.63.111.158` which is from an American DNS Server I've found here https://public-dns.info/. Make sure you look again for a different IP since this might change.


Now, for the UK:

![DNS Query with UK IP Address](https://res.cloudinary.com/danielcondemarin/image/upload/v1507500087/ukdnsquery.png)


That's about it really, hope you found it interesting and give some real use to it.









