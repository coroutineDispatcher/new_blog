---
title: "Bypass SSL pinning at runtime on Android non-rooted devices"
seoTitle: "Intercepting Android on runtime on non-rooted devices"
seoDescription: "Interestingly enough, Frida tool has its place on non-rooted devices as well. It can be used directly or with the help of another tool called Objection..."
datePublished: Sun Sep 08 2024 14:06:43 GMT+0000 (Coordinated Universal Time)
cuid: cm0tnds5z000409la50ffanus
slug: intercepting-android-at-runtime-on-non-rooted-devices
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725797182983/ddc23c93-a2ec-41b2-a3ac-26c51890027c.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1725804368196/67813941-44a5-4406-a5e6-51184a427e35.jpeg
tags: android, hacking

---

*Disclaimer: Since I'm writing more articles with focus on exploitation rather than development lately, it's worth mentioning that such guidelines are not encouraging anybody to attack Android apps without authorisation. These serve only for educational purposes.*

[My previous article](https://dispatchersdotplayground.hashnode.dev/hacking-android-on-runtime-using-frida-tool) was a brief introduction on how to use Frida tool to hook Java/Kotlin functions during runtime. Some of the comments that I got on Reddit were fairly stating that having a pre-set environment (running as `root`) would make any case relatively easy. Besides, when you think about it, how many Android users actually root their devices? Most of them have no idea what running as root is. The point here is that even though you can perform great things with Frida tool, a rooted device just for demonstration is merely a real life realistic example.

So, what about non-rooted devices? Devices that we buy from any store, and we just plug and play? They come with an included security layer (or limitations of what you can do) and a warranty. Or in case you want to root them, you get warned: you are on your own.

Interestingly enough, Frida tool has its place on non-rooted devices as well. It can be used directly or with the help of another tool called [Objection](https://github.com/sensepost/objection), something that OWASP mentions a lot in their security guidelines when it comes to testing your Android application for security implications. But, it works a bit differently in this case...

Since we are in a non-rooted device, it's not possible for Frida to be attached to the memory space of the running/victim application process. Therefore, what can be done is repackaging the `.apk` of that app with an extra library (`.so` file).  
One might argue again that such a use-case, it's also not that realistic. I would disagree. Since there is a high number of users who do not know what is going on in the background, I'd say that once an attacker is in the phone using network exploitation tools, they can do anything with their `apk`s.

### Example usage: Disabling SSL pinning

This is a pretty common one. Since network requests can be easily intercepted with simple proxies, developers tend to configure their apps to trust certain SSL certificates, thus blocking any other thing that stands between the Android client and the server. This can be easily spotted in the logs:

```bash
Caused by: java.security.cert.CertificateException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.
```

In other words, no attacker can mess with the app's network requests or responses, creating a huge flaw in the dynamic analysis of the app.

This is where Frida + Objection come in handy (there are other ways too of course). Let's see how we can entirely turn off the SSL pinning for the certain application.

#### Pulling the target app

First, we locate the `.apk` in the device:

```bash
adb shell pm path <package_name_of_target_application>
```

If searched correctly, a certain path of the `.apk` should be given back as output:

```plaintext
package:/data/app/<some-hashes>/<package_name>-<some-other-hash>/target_application.apk
```

Then we can pull it:

```plaintext
adb pull /data/app/<some-hashes>/<package_name>-<some-other-hash>/target_application.apk
```

#### Decompilation and repackaging

I am assuming that most of the readers know what I would be doing in this section, but maybe in the future I can show also a couple of techniques about decompilation of `.apk`s. For this case, it's my opinion that [apktool](https://apktool.org/) is enough, even though if you don't feel comfortable searching and editing `.smali` files, then you can go with solutions like [`dex2jar`](https://github.com/pxb1988/dex2jar) or [`jadx`](https://github.com/skylot/jadx). (`.smali` files are created as a result of disassembling Dalvik Bytecode).

After having correctly used `apktool d target_application.apk` a folder will be created with the same name (`target_application`), containing unpacked components that were put in inside the android `.apk` file. Of course, not everything would be readable, but that is also beyond the scope of this article.

Inside the decompiled `.apk` folder, something like this can be seen:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725800224423/f64a2fba-3a48-42e4-9b6f-ac67cde7c1e2.png align="center")

Our tasks here are two:

1 - Put the necessary Frida library inside the app.

2 - Call it, thus exploiting the app.

Adding the library is straightforward as long as the `lib` folder has been located. This one contains important system libraries (mostly written in c/c++) necessary for the app to run. Make sure to put the right [*Frida gadget*](https://github.com/frida/frida/releases) *(ends with* `.so`) according to your Android phone chip architecture. *Nitpick: Give it a shorter name, as it might be a bit overwhelming to work with all those version numbers and dashes.*

The second task is also relatively easy, for as long as you know how an Android app functions. The `.so` library needs to be loaded in the very beginning, ideally in the `Application` level or in the launching `Activity` level (or maybe even in a `SplashScreenActivity` if that exists).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725803996400/f09a880c-a628-456a-91c6-b8d0987254a3.png align="center")

Reading a `.smali` file is not something that is pleasing, but it's good enough to perform what you need:

```yaml
.method public onCreate()V
    #line 1 - 125 are left out
    .line 126
    .line 127
    const-string v0, "frida-gadget"
    invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V

    .line 128
    return-void
.end method
```

This is Android's `onCreate` method in `.smali` which in Kotlin it would be the equivalent of:

```kotlin
override fun onCreate() {
    super.onCreate()
    ...
    System.loadLibrary("frida-gadget")
}
```

The `.smali` file has been prepared and all that is left for this task is repackaging the `.apk` and installing it.

```bash
apktool b target_application -o target_application_2.0.apk
```

*Nitpick: You may want to make the* `<application` *part* `debuggable="true"` *in the* `AndroidManifest.xml`*. Might come in handy later for debugging purposes, just so you don't unpack the* `.apk` *and repack it again. But for this use case, it is irrelevant.*

Don't forget that once the new `.apk` has been compiled, you can't install it yet until you sign it.

```bash
apksigner sign -ks <key_signer_location> target_application_2.0.apk
```

#### Exploitation

For this step, Objection comes into play. Once the application has been launched, hopefully the `frida-gadget` library has been loaded together with it. Therefore, we can do:

```bash
➜ objection -g <target-package-name> explore
```

If all the previous steps are done correctly, the output should be something like this:

```bash
Using USB device `Android Emulator <port>`
Agent injected and responds ok!

     _   _         _   _
 ___| |_|_|___ ___| |_|_|___ ___
| . | . | | -_|  _|  _| | . |   |
|___|___| |___|___|_| |_|___|_|_|
      |___|(object)inject(ion) v1.11.0

     Runtime Mobile Exploration
        by: @leonjza from @sensepost

[tab] for command suggestions
```

You are literally inside the target application process. One command left:

```bash
android sslpinning disable
```

And the SSL pinning is no longer blocking you from intercepting network calls.

*By the way, this should work in iOS too (never tried it).*

Intercepting apps on runtime for me remains to be a fascinating area, along with static and memory analysis, which I am still learning all together. What fascinates me in those few cases that I have participated in vulnerability disclosure programs: After having bypassed this step, most of the apps do no encoding/obfuscation or encryption of the network requests at all. You can literally see your own username and password in `.json` flying from the client to the server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725803207862/ba036fe8-e52f-4161-a279-89e8817b28f9.png align="center")

As a developer now exploring security, I've realized that we often focus more on using cool tools and following architectural guidelines, while sometimes overlooking the security of our apps. But no matter how well-designed your app is, if the security layer isn’t strong, it can lead to serious problems. A poor architecture might cost you some money, but weak security can cost you a fortune.