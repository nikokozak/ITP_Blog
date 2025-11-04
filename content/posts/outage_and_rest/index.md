+++
date = '2025-11-03T21:55:21-05:00'
title = 'Outage and REST'
tags = ['Understanding Networks']
+++

(I didn't end up writing about REST)

# AWS Outage Updates

Amazon recently released an incident summary regarding the AWS outage that happened on Oct. 19, which took down most of the `us-east-1` region. I thought I might revisit the incident and try to summarize the findings in a more abstract manner. I know this isn't entirely related to this week's topic, but I find it's always interesting to peer into how these large infrastructures are handled, particularly when so much is revealed in an incident report.

## DynamoDB

DynamoDB is a fully managed database provided by Amazon as part of its AWS suite. Interestingly, it is also used quite extensively by internal AWS services (which explains the cascading failures that occurred during the outage). While tempting to think of a database as a single-location entity, DynamoDB is, in reality, a massively-distributed system spread across thousands of servers. Managing how to route access to these different servers is therefore a non-trivial question.

## DNS Planner

Managing routing in a massive service like DynamoDB entails thousands (if not hundreds of thousands) of load balancers, constantly changing IP's (servers drop, etc.), and various endpoints into the system (IPv4, IPv6, etc.). This system has to be managed by a set of processes designed exclusively for this purpose. Enter the DNS Planner. 

Think of the DNS planner as the Brain of internal DNS-handling operation for DynamoDB. Its job is to constantly monitor the various load balancers in the system (which are in turn routing requests to different servers), and creates a DNS plan accordingly. This DNS plan (which in this system is really a list of IP and associated weights), governs how requests are sent to each internal endpoint. As load balancers fail or go offline, the DNS planner adjusts and issues a new DNS plan.

## DNS Enactor

The DNS enactor does just what its name suggests. It runs as three identical processes (for redundancy), and is tasked with taking these DNS plans and updating Route53, which is AWS's DNS service. Their entire purpose is to look for "newer" DNS Plans, and attempt to implement them, while also pruning "older" plans from the system. These enactors are the culprits of the outage, or rather, a small oversight in their design is to blame. 

## The Race Condition

"Race condition" is a term used in computer science to describe what happens when two processes, running simultaneously, affect some sort of shared state in an unexpected way. A good example of this is a database that doesn't guarantee against simultaneous read-writes: one process decides to delete value A at the same time that another process decides to read it. What happens? No one can tell for sure, but it's never good.

In our case, the outage was caused by a race condition between two DNS Enactors. The script goes something like this:

Enactor A:
Starts applying Plan #100 (created 10 mins ago)
- Updating endpoint 1... OK
- Updating endpoint 2... Blocked, retrying
- Updating endpoint 2... Still blocked, retrying

Planner:
Keeps working away, creating Plan #101, #102..., #150

Enactor B:
Picks up Plan #150
- Updating all endpoints... OK
- Cleaning up Plans older than #145...

At the same time:

Enactor A:
Finally manages to apply Plan #100

Route53:
Now is using Plan #100 (as it was the latest applied, not necessarily the newest)

Enactor B:
- Cleaning up Plans older than #145... OK <--- BOOM! 

Enactor B inadvertently cleans up Plan #100, which was the active Plan, therefore leaving the system on an empty state. No DNS for DynamoDB!

## Checks and Safeguards

Why did this happen? Another related term from computer science is TOCTTOU - Time of Check, Time of Use. When Enactor A checked to see if its plan was newer than the one in use (Plan #100 > Plan #99), the check was successful. But by the time it managed to finally apply the action, the check was "stale". Therefore, the update went through, and the DNS system collapsed.

## Cascade

One of the major AWS services that uses DynamoDB is the EC2 system, which manages and provisions VPSes. It holds a significant amount of state in DynamoDB. Thus when Dynamo broke, so did EC2. As things came back online, so did thousands of EC2 instances, which all called out to Dynamo, causing a sort of feedback loop. It was impossible to create new instances, and the backlog of network requests meant that the entire system had to be throttled in order to avoid another outage.

Some other services that were impacted were: Lambda, IAM/STS, Redshift, and Connect. 

## Conclusions

One of my favorite aspects of the way we've configured our computer systems is that the same basic principles apply at small and big scales. You can't seem to get away from fundamental architectural principles. You'd be forgiven for thinking that AWS's systems operated on magical alien dust given their scale and complexity. And yet this incident report is fairly understandable to anyone with a passing knowledge of networks and software architecture. We're talking about load balancers, DNS provisioning, and race conditions - not easy stuff, but not rocket science. The *really* cool thing is that, if you really wanted to, you could easily set up a replica system on your own computer, and explore how a system like this is built, using nothing more than Nginx and your language of choice.

Cool stuff.