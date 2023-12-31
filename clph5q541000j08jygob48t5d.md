---
title: "Modularizing your Android app, some quick notes (Part 4)"
datePublished: Mon Nov 25 2019 09:13:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5q541000j08jygob48t5d
slug: modularizing-your-android-app-some-quick-notes-part-4

---


  
Basically, [in part 3](https://www.coroutinedispatcher.com/2019/11/modularizing-your-android-app-breaking_18.html) of this series we managed to fully modularize an Android app.  However there are some notes that need to be taken. We didn't cover too much about resources (res folder) in any of our parts therefore we are handling it now.  
  
Here are some things that we should notice about resources in modularization:  
  
1- Strings. No need there for a large file of it. It's easier when each module has access to it's own string values rather than accessing them all from a :core\_module. It's easier to read, easier to find strings and if you have a double String resource (like having 2 languages or more) the complexity gets lower. Of course, string resources that are present in more than one module, must remain in the only source of truth.  
  
2- Colors & drawables. It's really important to pay attention to drawables because they significantly affect the APK/Bundle size. The same rule should apply here (not to mention that sometimes we forget to delete drawables that are not used at all).  
  
3- Styles. If you have only one theme applied to the whole app you better not touch this file for side effects. Keep it in the core module of the app.  
  
4- Common layout files and fonts. Kotlin synthetics gave me hard time and runtime exception when I added <include></include> tags in other modules which would be referenced by all modules. (I usually keep loading and error layout in a single part and use <include> tags.) Therefore I still think that the findViewById() is safer than them.  
  
APK/Bundle size:  
  
I noticed that my bundle size was reduced by nearly 30% after modularization. Which means more installs and happier users:  
  

  

[![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104544924/1595fb7e-9d9f-461f-b909-eb09087fb66e.png)](https://1.bp.blogspot.com/-7F95cp8XuKQ/Xdj334Y3pdI/AAAAAAAAQcc/hHZTnMQxv0IufCC8s9iAL_TLvP9bBnD5gCLcBGAsYHQ/s1600/chart%2B%25282%2529.png)

  
  
Also builds are significantly faster.  
  
Stavro Xhardha
