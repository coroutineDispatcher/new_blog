---
title: "Hacking Android on runtime using Frida tool"
seoTitle: "Hacking Android on runtime using Frida tool"
seoDescription: "Learn how to intercept an Android app at runtime using Frida tool."
datePublished: Thu Aug 15 2024 18:23:48 GMT+0000 (Coordinated Universal Time)
cuid: clzvlzxz400000al719wn1vzw
slug: hacking-android-on-runtime-using-frida-tool
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723742373440/40074c6b-95f6-4a25-8f83-716006d25e55.jpeg
tags: android-app-development, android, hacking, pentesting

---

Lately I've been involved in extending my knowledge in pentesting, reverse engineering and hacking Android apps. It's still a long journey to go. However, I enjoy such a path so much that I can't refrain myself from writing about it.

Of course, there are multiple ways to tamper with an Android `apk`. That includes playing around with the app (behaviour analysis or injecting URL/URI's or even SQLite injection), reverse engineering the `dex` code (static analysis), or messing with the app directly on runtime. There are plenty of tools to be used for each case, but what I find more fascinating is the runtime case. In static analysis, you can't do much about SSL pinning or when the app has a PIN prompt. Therefore, tools like Frida are really valuable.

[Frida](https://frida.re/) is, as the website also claims, *a dynamic instrumentation toolkit for developers, reverse-engineers, and security researchers*. In other words, it's not something that belongs to the hacking or pentesting world. A developer can find much use of it while trying to reproduce bugs that are not easily reproducible. The usage of it is wider than the Android scope though. It can be used in multiple technologies the same way. Let's see a very basic usage of Frida in Android.

### Hello &lt;secret message&gt; (or just a UUID)

*An Android app is producing a secret message every time the user clicks a button, saying "Hello &lt;message&gt;" to the user. In our case, the secret message is just a UUID represented as a String. Our goal is to intercept and modify this message.*

**Because Frida is a Linux daemon, it works only on rooted Android devices.** The way it works is using the client-server architecture. The Frida CLI lies in the client (usually the attacking machine). The server side is the target mobile phone/emulator holding the running `apk`.

#### Enough talk, let's look at some action

The Android app:

```kotlin
FridaexampleTheme {
    val secretMessage = remember {
                mutableStateOf(secretMessage())
            }

    Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
        Column(Modifier.padding(innerPadding)) {
            Text(
                text = "Hello $secretMessage!",
                modifier = modifier
            )
            Button(
                onClick = { secretMessage.value = secretMessage() }) {
                    Text(text = "Hack me if you can")
                }
            }
        }
    }
}

private fun secretMessage(): String = UUID.randomUUID().toString()
```

So all you would see in the screen would be something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723743765705/04e4bb16-4f32-4022-b890-c908fd32e851.png align="center")

***NOTE: Since we know the method name*** `secretMessage()`***, we will skip the reverse-engineering part of the APK.***

Let the hacking begin...

First, the Frida server needs to be downloaded from their [GitHub Release section](https://github.com/frida/frida/releases).

Then the Frida tool needs to be installed on the attacking machine.

```bash
pip install frida-tools
```

Push the Frida server onto the emulator via `adb`:

```bash
adb push frida-server-x.x.x-android-x86 /data/local/tmp/frida-server
```

Afterwards, make sure it has the right executable permissions:

```bash
adb shell chmod 755 /data/local/tmp/frida-server
```

And then you can start it via:

```bash
adb shell /data/local/tmp/frida-server &
```

With all that said, everything is set up. All we need to do know, is to write the scrip to inject. Frida works with either Python or JavaScript.

### JS or Python? For Android runtime?

Unfortunately, I don't know how it works under the hood. But it seems Frida is written in C and developers can easily write JavaScript APIs because of the integration of Google's V8 engine in the tool. I'll leave a short section of what the website mentions this:

> ## Why a Python API, but JavaScript debugging logic?
> 
> Frida’s core is written in C and injects [QuickJS](https://bellard.org/quickjs/) into the target processes, where your JS gets executed with full access to memory, hooking functions and even calling native functions inside the process. There’s a bi-directional communication channel that is used to talk between your app and the JS running inside the target process.
> 
> Using Python and JS allows for quick development with a risk-free API. Frida can help you easily catch errors in JS and provide you an exception rather than crashing.
> 
> Rather not write in Python? No problem. You can use Frida from C directly, and on top of this C core there are multiple language bindings, e.g. [Node.js](https://github.com/frida/frida-node), [Python](https://github.com/frida/frida-python), [Swift](https://github.com/frida/frida-swift), [.NET](https://github.com/frida/frida-clr), [Qml](https://github.com/frida/frida-qml), [Go](https://github.com/frida/frida-go), etc. It is very easy to build additional bindings for other languages and environments.

So let's write our JS script:

```javascript
Java.perform(() => {
    const MainActivity = Java.use('com.dispatchersplayground.fridaexample.MainActivity');
    const secretMessageFunction  = MainActivity.getName;

    secretMessageFunction.implementation = function () {
        send('Hacking the message');
        return 'HACKED';
    }
})
```

Let's break it down a bit. It feels like doing reflection (and indeed, in essence this is somewhat a reflection). The script is telling the Frida server to find such process and the class in the Android system: `com.dispatchersplayground.fridaexample.MainActivity`. Afterwards it is telling to modify the `secretMessage` method, where instead of returning whatever `String` it is supposed to (in our case a UUID), it should directly be 'Hacked'. The `send` method is just for testing purposes on the console:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723745368876/80b1b5f6-5fb0-4250-9b8d-03b54e00baf6.png align="center")

All that is left now is to inject the script to hook the method.

```bash
frida -U -n fridaexample  -l frida_hack.js
```

The `fridaexample` comes from a way that Frida orders and marks the Android processes. If you do:

```bash
adb shell ps -A | grep fridaexample
```

The equivalent in Frida tool would be:

```bash
frida-ps -Ua | grep fridaexample
```

While in `adb` you would get the whole package name, in Frida you would directly get the application name.

After the hook is executed, all that is needed is to try the button in the application. After having clicked the "Hack me if you can" button, the `secretMessage()` method would not return a valid UUID any more, but rather the message that was injected from the JavaScript hook.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723745164902/efddf306-e2aa-4e16-a0c5-e64564942873.png align="center")

Mission accomplished. The Android app has been hacked in runtime.

Luckily for every user out there, this tool cannot work unless the Android device is rooted (and if I am not mistaken, if the iOS device not jailbroken). However, it is one of the most interesting tools I have seen as a developer. With Frida, you can do a lot more than this example. For pentesters especially, this is indeed something that must be mastered.

Thank you for reading. If you enjoy the articles that I write, you can consider subscribing to my Newsletter so that you won't miss another article when I publish it. Enjoy coding (or hacking).