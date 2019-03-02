---
layout: post
title: AliExpress XSS vulnerability - take over any seller account
---

In this blog post I will discuss a XSS vulnerability I’ve found in AliExpress website.

I discovered this vulnerability while i bought items in the website, i wanted to contact with the seller so i sent him a message. As an application security expert i suspected that the messages system might be vulnerable to XSS so i started investigate it.

after a full investigation i found that it is possible to inject HTML \<b\> tag into the message, and it will be rendered as HTML code in the recipients' browser.

By injection the following malicious script payload in a message content parameter, the seller will browse to the message center in AliExpress website, thus, the malicious script will be executed on his browser:

  
```javascript
Hello Seller :)\<b style="position:fixed;top:0;left:0;display:block;width:100%;height:100%" onmouseover="alert('Barak Tawily, AppSec Labs')"\>PoC</b>
```

  
Note: the system doesn't allow send HTML tags in the content of the message, but it allows \<b\> tag only, thats why the payload to exploit the vulnerability is \<b\> tag and not any other.  
  

[![](https://4.bp.blogspot.com/-i_Q0N2SzR2k/VIhITDHTiMI/AAAAAAAAAhA/E9rS2Gfdkkw/s1600/Untitled.png)](http://4.bp.blogspot.com/-i_Q0N2SzR2k/VIhITDHTiMI/AAAAAAAAAhA/E9rS2Gfdkkw/s1600/Untitled.png)

  

  
  

The vulnerability allows the attacker to execute malicious code on the seller's browser, thereby putting in danger all of the AliExpress sellers.

  

The attack scenario:

1\. An attacker sends a message to a store via the "contact now" feature.

2\. The attacker sends a malicious script injected inside the message content.

3\. The seller browses to the AliExpress message center.

4\. The malicious script executed on the seller's browser, which can be lead to several attacks such as perform actions on behalf of a seller, phishing attacks, steal the victim's sessions identifier, etc.

5\. The attacker succeeds in executing malicious code in the seller's browser and will take it over on his store.

  

Skilled hacker might exploit this vulnerability and perform ranged attack by sending malicious messages to all AliExpress sellers and will cause a huge damage to AliExpress website.

  

A Proof of Concept (PoC) video can be found at the following link:  
[https://www.youtube.com/watch?v=78Y0CEQYN1Q](https://www.youtube.com/watch?v=78Y0CEQYN1Q)
