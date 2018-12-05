---
layout:        post
title:         "Security: What is Server Side Request Forgery?"
date:          "2018-12-07 00:00:00"
categories:    blog
excerpt:       "Client Side Request Forgery or CSRF is a well known security vulnerability. But what is SSRF?"
preview:       /assets/img/what-is-ssrf.jpg
fbimage:       /assets/img/what-is-ssrf.png
twitterimage:  /assets/img/what-is-ssrf.png
googleimage:   /assets/img/what-is-ssrf.png
twitter_card:  summary_large_image
tags:          [Development, DevOps, Security]
---

CSRF is, while still prevalent, no longer an unknown type of security vulnerability. It's brother, SSRF, however, is.
Most developers, or even DevOps engineers don't know about it. So how does it work?

## SSRF explained

SSRF is, at it's core, very simple. Let's say you have a web application that needs to download some data from external
sources. So we trick this web application to, instead of accessing the external site, download the data from an internal
server that is publicly not accessible.

Sounds unimpressive, right? However, keep in mind that more and more database, like redis, etcd, or ElasticSearch can
be accessed over HTTP. What if I trick the application to download some data from etcd? Maybe your AWS API keys are
stored in there and I can just grab them?

<figure>{% plantuml %}
{% include skin.iuml %}
hide footbox
actor Attacker
database "Internal service"
control "Web application"
group Direct attack
   |||
   Attacker ->x "Internal service": A direct attack does not work due to a firewall
   ||| 
end
...
group SSRF
   Attacker -> "Web application": Tricky request to the application
   activate "Web application"
   "Web application" -> "Internal service": Request to an internal service
   activate "Internal service"
   "Internal service" --> "Web application"
   deactivate "Internal service"
   "Web application" --> Attacker: Data from internal service is returned to attacker
   deactivate "Web application"
end
{% endplantuml %}<figcaption>Figure: How SSRF works</figcaption></figure>

Or, if I can trick the application to make a PUT or POST request instead of a GET request, I may even be able to change
data.

## Where SSRF happens

Now, you may be asking in what kind of scenarios SSRF can even happen. After all, not many web applications download
data from external sources at the request of the user?

You may think that, but just a few obscure examples include:

- Downloading an RSS feed or other data source from a remote server.
- Displaying snippets or previews of links.
- Partial code execution vulnerabilities in the application.

Even if your application *initially* doesn't download data from external sources, later on there may be a business
requirement and you may not think of it.

Additionally, even a developer could be tricked into opening a link that, under certain circumstances, could load data
from the internal service.

## Defense against SSRF

I've talked about layered security before, so my recommendations follow that principle. You will want to have more than
method of defense.

First of all, all internal services, no matter if they are behind a firewall or not, should be put behind some sort
of authentication. Sometimes that will be usernames and passwords, sometimes it will be TLS client certificates. (TLS
certificates will also enable encryption of the traffic.) If you use authentication, that will prevent clients that
don't have access to the authentication data to connect the service.

Second, you should (of course) set up an internal firewall between your application servers and internal services. If
you are using Kubernetes, you should set up a network policy.

And last, your application should vet the URL. At the very least it should check what IP address the URL resolves to.

## Conclusion

SSRF is a tricky vulnerability because it abuses a trusted component of your system: the application code. It's hard to
think of our own components as a potential threat.