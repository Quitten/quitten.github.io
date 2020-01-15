---
layout: post
title: Gem in a box XSS vulnerability - CVE-2017-14506
---

In this short blogpost I will give a short explain of XSS vulnerability i found on [geminabox](https://github.com/geminabox/geminabox) v0.13.5. which is a gems manager like [rubygems.org](http://rubygems.org/) so you can upload and download gems

Geminabox parses the uploaded gems and gives the users list of the gems on the system as the following image:  
  
  

[![](https://1.bp.blogspot.com/-8SC1-3gwyN0/Wb-BDMAAHJI/AAAAAAAAY5s/es2sjYrk94gscRClI-5rWCoypIKWq0luACLcBGAs/s320/Screenshot%2Bfrom%2B2017-09-18%2B11-17-04.png)](https://1.bp.blogspot.com/-8SC1-3gwyN0/Wb-BDMAAHJI/AAAAAAAAY5s/es2sjYrk94gscRClI-5rWCoypIKWq0luACLcBGAs/s1600/Screenshot%2Bfrom%2B2017-09-18%2B11-17-04.png)

As you can see, the system parses the gem's details and present it on the web UI.  
After few times, I succeeded to create a GEM file to exploit XSS, the attack scenario goes as follows:  
  
  

[![](https://1.bp.blogspot.com/-Wlzz0aIqy6s/Wb-Cb98PuAI/AAAAAAAAY58/1adQ43_OErsViSFzV8lBc8De5OJUaIgNQCLcBGAs/s320/Screenshot%2Bfrom%2B2017-09-18%2B11-23-01.png)](https://1.bp.blogspot.com/-Wlzz0aIqy6s/Wb-Cb98PuAI/AAAAAAAAY58/1adQ43_OErsViSFzV8lBc8De5OJUaIgNQCLcBGAs/s1600/Screenshot%2Bfrom%2B2017-09-18%2B11-23-01.png)

  
  
  
  
  

*   Malicious attacker create GEM file with crafted homepage value (gem.homepage in .gemspec file) includes XSS payload as the following image:

*   The attacker access geminabox system and uploads the gem file (or uses CSRF/SSRF attack to do so). 

*   From now on, any user access Geminabox web server, executes the malicious XSS payload, that will delete any gems on the server, and won't let users use the geminabox anymore. (make victim's browser crash or redirect them to other hosts).

  
PoC video: 
[https://www.youtube.com/watch?v=eFIJL7yMYFA](https://www.youtube.com/watch?v=eFIJL7yMYFA)
