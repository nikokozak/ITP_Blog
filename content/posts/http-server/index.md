+++
date = '2025-10-16T17:19:53-04:00'
draft = false
title = 'Http Server'
tags = ['Understanding Networks']
+++

For most of this class, I've been running my ITP blog from the DigitalOcean server we set up the first week. 

Because I've been running Caddy and not Nginx, I figured I'd look into the differences between these two servers a bit more. I've used Caddy for many years, having made the switch after getting tired of Nginx's verbose configuration files (which become a pain to navigate if you're handling deployments often, or want to make small, fast tweaks).   

It first might help to specify what exactly Nginx and Caddy *are*: both are web servers, namely applications that receive requests on open ports, and decide what (if anything) to return to in order to satisfy the request. Of the two, Nginx is the oldest: created by Igor Sysoev in 2004, Nginx has grown to capture most of the web-server "market share", and is used extensively not only as a basic server but also a load-balancer (which we'll get into). That said, Nginx's age and predominance has also meant that it has ballooned in complexity, expanding in order to serve an ever-growing number of use-cases. Nowadays, under the F5 company, it describes itself as a "omprehensive platform for app delivery, optimization, and security in Kubernetes environments." For all intents and purposes, Nginx will fulfill every requirement you have of it, from personal-site serving to large-scale enterprise-level application servicing.

Caddy, in turn, is one of the new(er) kids on the block. Created in 2015, Caddy has since gained a large number of adherents and industry sponsors. It's most recent major version (v2) was launched in 2020. What's interesting about Caddy is that it's not *strictly* a web server; strictly speaking, it's a platform that makes orchestrating and configuring long-running services easy by unifying everything behind understandable JSON documents. It happens to include, by default, an extremely capable HTTPS server. Caddy is also written in Go, which is a more modern proposition compared to Nginx's C (though, granted, there's something nice about keeping your server as low-level as possible, performance-wise). 

Why Caddy?

Both servers cover most of the same bases. They can both be used as reverse proxies (simply put a service that re-routes client requests from a public port - like port 80 - to internal, unexposed ports in your computer). They can both be used as load-balancers (distributing requests amongst different processes in your machine to handle demand and avoid overloading processes). They can both filter and monitor traffic. More importantly, they both serve http and other assets quite capably, 

The main selling points for Caddy are:

- **Easy configuration.** Caddy can be configured with `JSON` or what's called a `Caddyfile`, where setting up a route is as easy as:
```
localhost

respond "Hello, world!"
```
- **Easy reloading and process management.** Caddy can be reloaded to incorporate configuration changes with no downtime. While this is possible in Nginx, in caddy it's as simple as calling `caddy reload` in the console.
- **Automatic HTTPS**. This is a huge one! Caddy incorporates `PKI` (Public Key Infrastructure) processes, which are a set of architectural considerations that mean that Caddy can handle digital certificates and encryption processes. This means that Caddy can *automatically* provision TLS certificates for endpoints it manages, by communicating with services like Let's Encrypt. It's a very pleasant developer experience to be able to spool up a server and immediately have `HTTPS` support, with no aditional work.
- **Easy Load Balancing**. On a more specific note, I quite enjoy how easy Caddy makes load-balancing:
  ```
  reverse_proxy * {
		# Specify backend here
		to 127.0.0.1:8001
		to 127.0.0.1:8002
		to 127.0.0.1:8003

		lb_policy round_robin
        health_path     /health # Backend health check path
  }
  ```
That's all that is needed to have Caddy redirect requests to three running processes, one after the other, while also checking to make sure the processes are alive. This makes rolling updates, no-downtime deployments, etc. very easy.

Perhaps the best part is that all of this only requires a simple `brew install caddy` on a new server. 

So - if you're in the process of provisioning a new VPS, I'd highly recommend giving Caddy a try. And, given it's open-source, perhaps even contribute to the project if you like it! It's a great excuse to learn Go, which (I've heard) is excellent at these types of service-type daemons.