---
layout: post
title: Gem in a box CSRF file upload - CVE-2017-14506
---  

In this blog post I will give a short example of exploiting CSRF vulnerability on Geminabox.  
So [Geminabox](https://github.com/geminabox/geminabox) is an application allows you manage your internal gems was vulnerable to CSRF on upload file.

In order to exploit the CSRF vulnerability I wrote really small tool called csrFile, which allows you to generate HTML that uploads any type of file to the supplied endpoint, you can check it out in the following link:  
[https://github.com/Quitten/csrFile](https://github.com/Quitten/csrFile)  
Usage: python csrFile.py <url> <filePath>  
  
So using the following command, you can easily create an HTML document that exploits the CSRF attack and uploads malicious gem file to the targeted server:  
python csrFile.py https://geminaboxserve/upload xss.gem  
  
Then in case the victim will browse to the attacker's link that contains the HTML generated from csrFile, his browser will automatically will upload the attacker's malicious gem to geminabox system.  
Note: it is possible to exploit persistent XSS attack (CVE-2017-14506) in that way as well.  
  
  
