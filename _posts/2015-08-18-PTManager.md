---
layout: post
title: PT Vulnerabilities Manager - burp extension
---

Github - [https://github.com/Quitten/PT-Manager](https://github.com/Quitten/PT-Manager)

[![alt tag](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/general.png)](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/general.png)

Installation
===========

1.  Download Burp Suite (obviously): [http://portswigger.net/burp/download.html](http://portswigger.net/burp/download.html)
2.  Download Jython standalone JAR (version >= 2.7): [http://www.jython.org/downloads.html](http://www.jython.org/downloads.html)
3.  Open burp -> Extender -> Options -> Python Environment -> Select File -> Choose the Jython standalone JAR
4.  Install PT Manager from the BApp Store or follow these steps:
5.  Download the PTManager.py file (and XlsxWriter-0.7.3 in case you would like to generate xlsx report).
6.  Open Burp -> Extender -> Extensions -> Add -> Choose PTManager.py file.
7.  See the PT Manager tab and manage your vulnerabilities and project easily :)

[](https://github.com/Quitten/PT-Manager#user-guide---how-to-use)User Guide - How to use?
=========================================================================================

After installation, the PT Manager tab will be added to Burp.

Project Settings Tab:

[![alt tag](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/project_settings.png)](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/project_settings.png)

1.  Open the Project Settings tab (PT Manager -> Project Settings) and create a new project, make sure you are creating it under encrypted partition.
2.  "Details" text area can be used in order to save any details about the project such as URLs, credentials, contact details, or any other comments.
3.  Generate report section can be used in order to generate project report in HTML, XLSX, DOCX, TXT.
4.  import and export buttons can be used in order to send other people reports and allows them import it in the extension.
5.  "Open project directory" - will open your project directory, dont you think? :)

Vulnerability Tab:

[![alt tag](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/vulnerability.png)](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/vulnerability.png)

1.  Open the Vulnerability tab, and create a new vulnerability.
2.  "Color:" combobox can be used in order to change specific vulnerability background color. it can be used in order to let you know if the vulnerability verified (set it green) or if it is false positive (so set it red or just remove the vulnerability)
3.  "Add SS from clipboard" button will copy image that you captured and save in your clipboard, it is also possible to add it manually by pasting jpg file into the vulnerability folder.
4.  it is possbile to right click the on the selected preview image in order to copy the file into the clipboard in case you would like to get speicific image
5.  request and response tab will include the requset and response of the vulnerable requset. it is possible to attach specific request and response to a vulnerability by right click on request/response from anywhere in burp then click on "Send to PT Manager"

[![alt tag](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/send%20to.png)](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/send%20to.png) [![alt tag](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/select.png)](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/select.png)[![alt tag](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/request.png)](https://raw.githubusercontent.com/Quitten/PT-Manager/master/images/request.png)
