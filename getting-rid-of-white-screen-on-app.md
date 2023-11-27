---
title: 'Getting rid of the white screen on the app start up'
date: 2020-03-04T21:58:00.000+01:00
draft: false
aliases: [ "/2020/03/getting-rid-of-white-screen-on-app.html" ]
tags : [Xml, Android UX, Splash Screen, Android]
author: "Stavro Xhardha"
---

  

[![](https://1.bp.blogspot.com/-Ilk8HkKHwto/XmAAerl4hxI/AAAAAAAASHA/HM--n5LPclUcnVSoOg1EukqJ4QVSm78pgCLcBGAsYHQ/s1600/johnny-brown-1k35_trK6qo-unsplash.jpg)](https://1.bp.blogspot.com/-Ilk8HkKHwto/XmAAerl4hxI/AAAAAAAASHA/HM--n5LPclUcnVSoOg1EukqJ4QVSm78pgCLcBGAsYHQ/s1600/johnny-brown-1k35_trK6qo-unsplash.jpg)


Details matter especially when it comes to making users happy. We have all kinds of users, those who don't care how the app is designed (just do what he/she needs), those who care, and those who are developers themselves. I have met people who don't use a single app just because it has some strange animation, and they hate it.

For me, it's a little from both. I Like seeing a minimalist design (like Airbnb Android app, which is one of my favorites, for instance), but it is also important to fulfill your needs also.  

I might be wrong, but some design details also define a company on how serious it is with its users. One of them is the Android white startup screen.  

It's just a screen, which stays not more than 2 seconds around before the first Activity of your app is being launched. There are plenty of ways to get rid of it (or just manipulate it), I personally prefer the easiest one.

{{< admonition >}}
Note: If you want to read more about this problem, [here](https://developer.android.com/topic/performance/vitals/launch-time) is the official documentation.
{{< /admonition >}}

So, let's see how to fix this small detail. And I will take twitter app as an example for that. IMO, that's the best example you can give on how to solve it. Now I am not sure if twitter has the same way that I will introduce, but twitters solution is pretty decent, and I assume it's nearly the same as the code below.

[![](https://1.bp.blogspot.com/-cVRSF9MfFRc/Xl_-f33bmcI/AAAAAAAASG0/sBejaXdI88Y6d_ecIIHQSQWsN0UztRGBgCLcBGAsYHQ/s320/twitter_example.jpg)](https://1.bp.blogspot.com/-cVRSF9MfFRc/Xl_-f33bmcI/AAAAAAAASG0/sBejaXdI88Y6d_ecIIHQSQWsN0UztRGBgCLcBGAsYHQ/s1600/twitter_example.jpg)

First, just add an XML file in your drawable folder:

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" android:opacity="opaque">  
    <item android:drawable="@color/twitter_blue_background" />  
    <item>  
        <bitmap  
        android:gravity="center"  
        android:src="@drawable/twitter_bird_icon" />  
    </item>  
</layer-list>
```

Next, all you would need is to call this implementation in your theme in the styles.xml file (make sure you place it in the current theme that is related on your AndroidManifest.xml):

```xml
<item name="android:windowBackground">@drawable/splash_screen</item>
```

So simple. And it's a nice experience also.

## Conclusion

I've seen many confusing articles about solving this problem with a very long implementation. One of those for instance, just tries to keep the app idle after clicking the launcher icon (like making everything transparent) until the Activity starts.... I don't agree with that even though for some might work as well.  

Stavro Xhardha