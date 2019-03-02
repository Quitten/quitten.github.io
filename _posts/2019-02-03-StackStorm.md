---
layout: post
title: StackStorm - From Originull to RCE
---

StackStorm (aka "IFTTT for Ops") is event-driven automation for auto-remediation, security responses, troubleshooting, deployments, and more.
In this blogpost I will describe how can you cause RCE on targeted servers which only requires an authenticated user browse to malicious webpage.

So first, lets understand what is StackStorm, StackStorm is a really popular (3K stars on github) devops event-driven automation tool, allows devops to configure actions, workflows, and scheduled tasks, in order to perform some operations on large-scale servers.
As every other new service that I touch, I am directly start to play with it, see how it is written and try to figure out how can I maniluate the system.
Usually StackStorm agent gets quite nice privlieges, because it needs to perform actions on servers, so I was really motivated that time.
So after playing it for a while, tried to break the authorization mechanism (with Autorize - check it out :)), but I failed, seems like they did nice authorization work.
I was managed to see this request:
<img>

I saw that CORS header returning in each request, even if I didnt send the origin header, quite weird but anyway might make sense... then I started to play with the Origin header, then I realzied, that when im changing the Origin header in the request, the server cant handle it properly, and returning CORS: null :
<img>

Right away I knew that there is something in here! I remember that I read once about Originull attack<link to bugsec> and talk with my coulleges about that but never saw it on systems.
Before getting the exploitation, I was really curious of the reason why is this happening? why servers returning CORS: null at all? they can just not returning this header at all.
So I managed to get the [Origin Web Concept RFC](https://tools.ietf.org/html/rfc6454)

blabla 
