+++
date = '2025-09-21T15:27:00-04:00'
draft = false
title = 'Dns'
tags = ['Understanding Networks']
+++

## Infrastructure

Despite all the arguments to the contrary, I'm convinced the development and spread of internet infrastructure is a miracle. It somehow mirrors things like the development of the United Nations, the eradication of Polio, or the creation of the European Economic Zone. This is partly because the birth and growth of the internet happens at roughly the same time these things do, and responds to a goldilocks moment in sociopolitical history around the world. The post-war momentum of liberal internationalism or institutionalism meant large-scale collaborative projects were seen as being the fundamental drivers of social progress. This ideologic paradigm bled into infrastructure and technology, justifying massive amounts of investment for research and build-out of systems that would eventually be coopted by private interests.

It's why, in part, I also think we're in a sort of Fukuyama-style deadlock when it comes to truly novel, distributed, and accessible infrastructure (both in the USA and worldwide). The initial friction of massive capital requirements paired with the need for cross-industry standardization and private-industry coordination makes it so that the large scale public-private initiatives that marked the 70's, 80's and 90's are no longer really a possibility.

## DNS

When doing a bit of research on DNS beyond the class readings, I found an article by [David Leadbeater](https://dgl.cx/2025/09/images-over-dns) about using DNS servers to host images. 

The premise of this "hack" is that, amongst other things, the DNS protocol allows for every Domain Name to have an associated "A" record (Address - where the name actually points to), as well as additional records like "MX", "AAAA", and "TXT" records (all of which provide additional metadata that varying services can use). 

**TXT** records were added to the DNS protocol in order to future-proof the system: they provide a field that can be used to handle arbitrary data, by holding strings of data. 

Curiously, while `UDP` DNS lookups limit the TXT field to around 1232 bits, `TCP` lookups have a much larger limit of 64kb. What this means is that we can use the `TXT` field to essentially 'host' massively-distributed cached versions of arbitrary data (in Leadbeater's case, an image). 

Reading through the article also clarified some misconceptions I had around the actual structure of these servers, and how anyone knows who the actual authoritative server is for a given domain name. Put simply:

```bash
Query: example.com TXT
1. Resolver asks root: "Who handles .com?"
   Root: "Ask Verisign at a.gtld-servers.net"
2. Resolver asks Verisign: "Who handles example.com?"
   Verisign: "Ask ns1.godaddy.com" 
3. Resolver asks GoDaddy: "What's the TXT record?"
   GoDaddy: "v=spf1 include:_spf.google.com ~all"
```