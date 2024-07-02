---
title: "Interprocess communication and the Binder interface"
seoTitle: "Interprocess communication and the Binder interface"
seoDescription: "While it is quite normal for an Android phone to rely on the HTTP protocol for network calls, that is not the case for inter process communication."
datePublished: Tue Jul 02 2024 18:13:08 GMT+0000 (Coordinated Universal Time)
cuid: cly4q8qv6000908mgbajkc7y3
slug: interprocess-communication-and-the-binder-interface
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719943855072/b4bdabe8-a9a1-4ab7-b354-6cd32d8ec82b.jpeg
tags: android-app-development, linux, java, android, android-studio, android-apps, ipc, linux-kernel, protocols, android-platform

---

Lately I have been messing around with the internals of Android. While I still enjoy architecting apps 'the proper way,' it all comes down to having a fancy endpoint to receive the data using some fancy parsing tool, have some fancy business logic, and a fancy `ViewModel` to transform everything from data to a view (or a `Composable`). The most common use case for such flow is receiving data from an internet server. Android relies on a network protocol which is standardized for most of the devices to make communication between them simple. But how do system components and internals communicate with each other?

# Binders

While it is quite normal for an Android phone to rely on the HTTP protocol for network calls, in order for an app to contact another app in the system, using the HTTP protocol is not necessary. The Binder interface is built as a lightweight component to handle every communication within the Android system. It's as simple and as complicated as that. The simple part is that developers do not have to care how the Binder works internally. [Google also suggests in their documentation](https://developer.android.com/reference/android/os/Binder) that there is no necessity to work directly with the Binder interface, but rather work on top of the AIDL tool. We will get to AIDL soon. The complicated part is that a Binder, for security reasons, performance reasons, and because it interacts directly with Linux processes for more flexible memory management, this component goes down to the Linux kernel. As the name suggests, "inter-process" communication, this literally means plain OS (in this case Linux) process.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719938725218/cf064d95-7a24-464c-b221-b78e84ac68e2.png align="center")

Since an Android developer doesn't have to care much about the internals of `Binder` interface, a recommended way to work with it from the Android Framework is introduced by Google. This is done through the AIDL (Android Interface Definition Language) interface. AIDL is a language that largely follows Java syntax; every Java developer would likely mistake it for Java at first glance, were it not for the `.aidl` file extension.

## A use case

Let's consider this: Your company building their own version of Android. They want an Authenticator service to work in the system background, and the apps they provide (like mail, calendar, etc.) to authenticate via the same service integrated into the system. For the sake of this example, assume the service is a standard service, not a system service, although little would change in this scenario. Let's explore a minimal example focusing solely on a login use case.

### The server

First, make sure your AIDL feature is enabled in `build.gradle`:

```kotlin
buildFeatures {
        compose = true
        aidl = true
    }
```

This should happen in both projects. After this, create a new AIDL file. Android Studio will give you an `aidl` folder with the interface you picked as its name:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719940520481/117e8495-6fef-46b3-9f7c-cf5ca2febcb8.png align="center")

And the interface would be:

```kotlin
package com.dispatchersplayground.ipcserver;

interface IAuthenticator {
    int authenticate(String username, String password);
}
```

If you can read Java, you can read AIDL. Not much going on here—just one functionality that attempts to log the user in. The next step would be to hit the build button. Android Studio will generate a new `build` folder that also contains Java code specifically designed for the system to use on top of the Binder interface for communication, depending on the interface you created. Let's take a brief look at the generated code:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719940719994/b5318711-9ac9-4a55-a425-1e1024938692.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719940751455/febfae58-73f6-4d50-a071-50dd61b5ca30.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719940808623/d75a2607-eb47-4111-a7db-17ebc8721243.png align="center")

You don't have to care about implementation details, but what you should know is that a `Stub` and a `Proxy` are created to handle the communication. The `Stub` is the communication implementation of the interface we created, and the `Proxy` is the component that gives client apps access to this implementation.

Now, let's implement what we want to do after the communication is successful (we will attempt to log the user in). I'll be hardcoding the properties, so this won't be proper authentication:

```kotlin
class AuthenticatorService : Service() {

    private val binder = object : IAuthenticator.Stub() {
        override fun authenticate(username: String?, password: String?): Int {
            return if (username?.isEmpty() == true || password?.isEmpty() == true) {
                -1
            } else 0
        }
    }

    override fun onBind(p0: Intent?): IBinder? = binder.asBinder()
}
```

The Binder is not a new instance of the `IBinder` interface, as it typically is for services communicating with activities within the same app. Instead, it is the `Stub` that was generated for us. Here is where we handle our business logic. By the way, the `Binder` interface is also used within the same app to establish a handshake between, for instance, an Activity and a Service.

And of course, the `AndroidManifest`:

```xml
<service
            android:name="com.dispatchersplayground.ipcserver.AuthenticatorService"
            android:enabled="true"
            android:exported="true" >
            <intent-filter>
                <action android:name="com.dispatchersplayground.ipcserver.AuthenticatorService" />
            </intent-filter>
        </service>
```

### Client app

The client app will be responsible for pushing a button and authenticating the user using hardcoded values.

```kotlin
                val authResult = remember { mutableStateOf(AuthenticationState.Unauthenticated) }
Surface(...) {
                    Column(...) {
                        Button(onClick = {
                            // TODO contact the remote service
                        }) {
                            Text(text = "Send authentication event")
                        }

                        Text(
                            text = "Result: ${authResult.value}",
                            color = Color.Black,
                            fontSize = 32.sp,
                            fontFamily = FontFamily.Default,
                            fontStyle = FontStyle.Normal
                        )
                    }
                }
```

Now we have to configure AIDL on the client as well. We need to repeat the same process to include the interface in that app.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719941551865/9dc6e233-910f-4825-8a45-9f0961400ea5.png align="center")

And the interface:

```kotlin
package com.dispatchersplayground.ipcserver;

interface IAuthenticator {
    int authenticate(String username, String password);
}
```

Do you notice something interesting? The package name for the AIDL interface should remain the same as the server it will be contacting. In this case: `com.dispatchersplayground.ipcserver`.

After building the client project, Android Studio would generate almost the same Java code in the `build` folder as seen in the screenshots above from the server app.

If it were pre-Android 11, you would be done. However, for security reasons, the Android team has added an extra mechanism. The client's Android Manifest should specify which packages it intends to contact. In this case, we would need to add something like this:

```xml
<manifest> 
    ...
    <queries>
        <package android:name="com.dispatchersplayground.ipcserver" />
    </queries>
</manifest>
```

With that, the setup is complete. Now let's connect the dots.

```kotlin
class MainActivity : ComponentActivity() {
    private lateinit var service: IAuthenticator

    private val serviceConnection = object : ServiceConnection {

        override fun onServiceConnected(componentName: ComponentName?, binder: IBinder?) {
            service = IAuthenticator.Stub.asInterface(binder)
            Toast.makeText(applicationContext, "Service Connected", Toast.LENGTH_SHORT).show()
        }

        override fun onServiceDisconnected(componentName: ComponentName?) {
            Toast.makeText(applicationContext, "Service Disconnected", Toast.LENGTH_SHORT).show()
        }
    }
} 
```

Notice the Binder interface coming from the `onServiceConnected` callback. That's how these two apps will communicate. On the client side, the interface just needs to be instantiated, which also comes from the generated code.

```kotlin
service = IAuthenticator.Stub.asInterface(binder)
```

The binding is straightforward, just as it would be if it were the same app:

```kotlin
override fun onStart() {
    super.onStart()
    val intent = Intent("com.dispatchersplayground.ipcserver.AuthenticatorService").apply {
        setPackage("com.dispatchersplayground.ipcserver")
    }

    bindService(intent, serviceConnection, BIND_AUTO_CREATE)
}

override fun onDestroy() {
    super.onDestroy()
    unbindService(serviceConnection)
}
```

And all that needs to be done is to prepare the UI:

```kotlin
Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Button(onClick = {
        authResult.value = when (service.authenticate(
            "hardcoded_username",
            "hardocded_password"
        )) {
            0 -> AuthenticationState.Authenticated
            -1 -> AuthenticationState.Unauthenticated
            else -> error("Unknown state")
        }
    }) {
        Text(text = "Send authentication event")
    }

    Text(
        text = "Result: ${authResult.value}",
        color = Color.Black,
        fontSize = 32.sp,
        fontFamily = FontFamily.Default,
        fontStyle = FontStyle.Normal
    )
}
```

*Note: When installing, install the server app first.*

And the result:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719942246971/cf1da7d9-affe-4e59-9c87-267285d5a992.gif align="center")

### Closing thoughts

Interprocess communication isn't an everyday use case. In fact, some developers may go through their entire careers without encountering it. It's primarily used by product companies that work mostly in the platform or in rare cases where a product consists of multiple apps.

Understanding how Android works internally, not just on the surface, can be very interesting. It shows you what happens behind the scenes in the Android system. This exploration is a great starting point for anyone wanting to learn more about how Android operates inside. However, AIDL, this is just the beginning of a long learning journey.