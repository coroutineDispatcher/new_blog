---
title: "Publish your library to JitPack "
date: 2019-09-17T10:16:00.000+02:00
draft: false
aliases: ["/2019/09/publish-your-library-to-jitpack.html"]
tags:
  [
    Android Module,
    Java,
    Kotlin Module,
    Github,
    Git,
    Java Library,
    Kotlin,
    Android,
    JitPack,
    Java Module,
    Kotlin Library,
    Android Library,
  ]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-sg_QzKGtis0/XYCUittBU1I/AAAAAAAAPRU/Gknu578O8Ksje57qv030Dqb0PFFYoQn6gCLcBGAsYHQ/s1600/kira-auf-der-heide-fV4-DdSdcpI-unsplash.jpg)](https://1.bp.blogspot.com/-sg_QzKGtis0/XYCUittBU1I/AAAAAAAAPRU/Gknu578O8Ksje57qv030Dqb0PFFYoQn6gCLcBGAsYHQ/s1600/kira-auf-der-heide-fV4-DdSdcpI-unsplash.jpg)

Once getting inside the open source concept, the idea and desire to publish a library is inevitable. What you need is an idea, or perhaps make a better version of an existing library, or just you need to use some module inside your company for several projects.

With the JitPack, you won't be suffering at all. So how do you do it? For the sake of this tutorial, I'm just gonna make a logging library.

_Note: This article assumes you already know how to use git._

Once you have your IDE opened, create a new project. Since I am using Android Studio, I'm gonna choose an Android app but you can do the same (I guess, never done it) with on Java/Kotlin projects and IntelliJ. Nothing new here. So let's jump further. Once the project is created, create a new module:

[![](https://1.bp.blogspot.com/-G7AiUvhpie8/XYCLw53I-VI/AAAAAAAAPPs/5h8UjqmgeKY7XJWX5HaPXEtqsmepEB_9QCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.30.33.png)](https://1.bp.blogspot.com/-G7AiUvhpie8/XYCLw53I-VI/AAAAAAAAPPs/5h8UjqmgeKY7XJWX5HaPXEtqsmepEB_9QCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.30.33.png)

After that, you can choose whatever you need, but for this article I'm gonna stick to Android library.

[![](https://1.bp.blogspot.com/-RxhUhlU38tY/XYCMONFG7xI/AAAAAAAAPP0/T_1IjnEg7oQ26K3wX98pZ26t_tyQwIhpwCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.32.25.png)](https://1.bp.blogspot.com/-RxhUhlU38tY/XYCMONFG7xI/AAAAAAAAPP0/T_1IjnEg7oQ26K3wX98pZ26t_tyQwIhpwCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.32.25.png)

Give it a name, and the library is ready to be coded. If you notice, in your project, the module will be added:

[![](https://1.bp.blogspot.com/-oi19wbcvdMc/XYCM3xbCklI/AAAAAAAAPP8/Tq20K-cElcEl65wsDalAADymYxs_pHmfgCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.34.10.png)](https://1.bp.blogspot.com/-oi19wbcvdMc/XYCM3xbCklI/AAAAAAAAPP8/Tq20K-cElcEl65wsDalAADymYxs_pHmfgCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.34.10.png)

Now, let's code. I'm going to create an awesome method that logs something in the console, wow. Since I'm using Kotlin, I won't be needing a new class, but if you use Java, create a new class.

[![](https://1.bp.blogspot.com/-IlttyplAD-A/XYCNrNWIRKI/AAAAAAAAPQE/9tQLJ7cbxkkNNF8b0jRMqeaHW9cYhZrKwCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.38.42.png)](https://1.bp.blogspot.com/-IlttyplAD-A/XYCNrNWIRKI/AAAAAAAAPQE/9tQLJ7cbxkkNNF8b0jRMqeaHW9cYhZrKwCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.38.42.png)

_For local usage, you don't need to publish to JitPack, but I am skipping this part here._

Git time

First of all, create a new repository. Don't forget to add an open source license.

[![](https://1.bp.blogspot.com/-a2aMr0xM17Q/XYCP35XuXLI/AAAAAAAAPQY/LhClLDkaBm0uX6qU4gebaDDM7oqDNHmIQCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.48.34.png)](https://1.bp.blogspot.com/-a2aMr0xM17Q/XYCP35XuXLI/AAAAAAAAPQY/LhClLDkaBm0uX6qU4gebaDDM7oqDNHmIQCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.48.34.png)

Now, just normally add your project to via console or git client that you might be using. Please note that if you create a license and a README file, you are going to have to git pull first and then push your project into the repo

Time to publish:  
Once your project is super ready for production, go to releases (normally you would have 0 for the moment):

[![](https://1.bp.blogspot.com/-6CVx8GnzdSw/XYCRxST5OVI/AAAAAAAAPQk/gvbiIbEDSRAUkynKIzWKGatm1hpmkc81QCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.56.21.png)](https://1.bp.blogspot.com/-6CVx8GnzdSw/XYCRxST5OVI/AAAAAAAAPQk/gvbiIbEDSRAUkynKIzWKGatm1hpmkc81QCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.56.21.png)

Well, give the release a name, a version and some description.

[![](https://1.bp.blogspot.com/-5mfI0xU5mb0/XYCSOAJ1OdI/AAAAAAAAPQs/ELkTqMBT8scWr4yd4_u_cXg1F3gt2jR6QCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.58.30.png)](https://1.bp.blogspot.com/-5mfI0xU5mb0/XYCSOAJ1OdI/AAAAAAAAPQs/ELkTqMBT8scWr4yd4_u_cXg1F3gt2jR6QCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B09.58.30.png)

If you have reached something like the below photo, your library is ready for JitPack:

[![](https://1.bp.blogspot.com/-yNr-QA3mdG4/XYCSviyGWSI/AAAAAAAAPQ0/JG2lqMSEqLox043gw4d0YV7pDMArFrzEQCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.00.24.png)](https://1.bp.blogspot.com/-yNr-QA3mdG4/XYCSviyGWSI/AAAAAAAAPQ0/JG2lqMSEqLox043gw4d0YV7pDMArFrzEQCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.00.24.png)

So, let's jump to JitPack. Just add the repo url and let JitPack handle generation for you.

[![](https://1.bp.blogspot.com/-3wjBqroWh0c/XYCTVuI-kkI/AAAAAAAAPQ8/yn6c7yAIH5gahyxixrKzjl4F_6n2oCF-ACLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.02.25.png)](https://1.bp.blogspot.com/-3wjBqroWh0c/XYCTVuI-kkI/AAAAAAAAPQ8/yn6c7yAIH5gahyxixrKzjl4F_6n2oCF-ACLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.02.25.png)

Once a version is found by JitPack, it will generate the code to be imported from you or other people who will use this library:

[![](https://1.bp.blogspot.com/-tGCniNRWX2Y/XYCT0PqxGtI/AAAAAAAAPRE/DywxkBx9_qUvegQO8fpQBvW1StmJO1afgCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.05.13.png)](https://1.bp.blogspot.com/-tGCniNRWX2Y/XYCT0PqxGtI/AAAAAAAAPRE/DywxkBx9_qUvegQO8fpQBvW1StmJO1afgCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.05.13.png)

That's it. If you follow those steps, you are ready to use the library. And indeed you are:

[![](https://1.bp.blogspot.com/-X4wlu7AXoWI/XYCUDnYc29I/AAAAAAAAPRI/4VaY-DrZMq82Z0FFOXRkHzRB2AZ9Qr_lwCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.06.19.png)](https://1.bp.blogspot.com/-X4wlu7AXoWI/XYCUDnYc29I/AAAAAAAAPRI/4VaY-DrZMq82Z0FFOXRkHzRB2AZ9Qr_lwCLcBGAsYHQ/s1600/Screenshot%2B2019-09-17%2Bat%2B10.06.19.png)

Choosing an Android library for cases like this, might be redundant. That's because doing so, the IDE will also create Android Packages along with Java or Kotlin packages. Instead you should stick to Java/Kotlin libraries.

Stavro Xhardha
