---
title: "Screenshot Testing on the JVM. Thanks to Paparazzi."
datePublished: Mon Nov 27 2023 17:19:15 GMT+0000 (Coordinated Universal Time)
cuid: clph6bqut000009jm6yrn9myy
slug: screenshot-testing-on-the-jvm-thanks-to-paparazzi
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701105587148/c75bd668-f51d-4994-84b5-3b9d0a29d13b.webp
tags: android, testing, jetpack-compose

---

Screenshot testing is an essential part of the development process, allowing us to verify the visual appearance of our user interfaces and catch any unintended changes. Traditionally, screenshot testing required an Android emulator or physical device, which can be time-consuming and resource-intensive. However, thanks to the Paparazzi framework, there is now a way to perform screenshot testing on the JVM without the need for an Android emulator.

## Setup

To get started with Paparazzi, you need to install the framework. You can add the following code snippet to your build.gradle file:

```kotlin
plugins {
    id 'com.android.library' <- Pay attention here
    ...
    id 'app.cash.paparazzi' version '1.3.0'
}
```

Now, let's take a look at the composable we want to test:

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}
```

We have a simple `Greeting` composable that displays a text message with the provided name.

To write a screenshot test, we can use the Paparazzi library. Here's an example:

```kotlin
class GreetingsScreenTest {

    @get:Rule
    val paparazzi = Paparazzi(
        deviceConfig = PIXEL_5,
        theme = "android:Theme.Material.Light.NoActionBar"
    )

    @Test
    fun `greetings screen should say Hello Android`() {
        paparazzi.snapshot {
            Greeting(name = "Android")
        }
    }
}
```

In this test, we set up the Paparazzi rule with the desired device configuration and theme. Then, we use the `snapshot` function to capture a screenshot of the `Greeting` composable with the name "Android".

You can configure any device using the Paparazzi constructor and choose any theme you want. Additionally, you can create your own custom device configuration using the `DeviceConfig()` class.

To generate the reference screenshot, you can run the following command:

```bash
./gradlew ui:recordPaparazziDebug
```

By default, the generated screenshot file will be located under `test/snapshots/images/` and it will looks something like this:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tjazj4gp6h2lc4805336.png align="left")

Now, let's say we modify the `Greeting` composable to include a button:

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Column() {
        Button(
            modifier = modifier, onClick = {}
        ) {
            Text(text = "Hello $name!", modifier = modifier)
        }
    }
}
```

After making this change, we can run the verification command:

```bash
./gradlew ui:verifyPaparazziDebug
```

During the verification process, the compiler will detect a difference between the reference screenshot and the new screenshot and throw an error:

```bash
java.lang.AssertionError: Images differ (by 0.7%)
```

This is the intended behavior of screenshot testing with Paparazzi. It helps ensure that the visual appearance of our UI remains consistent.

Screenshot testing with Paparazzi offers several advantages. It helps keep projects safe by detecting unintended UI changes. Additionally, it doesn't require a real emulator, making it faster and more efficient. It's worth noting that Paparazzi works with both Composables and Android Views, providing flexibility for different UI components.

# Limitations

While Paparazzi is a powerful tool for screenshot testing, it has some limitations to consider. Due to the way Android apps ([`com.android`](http://com.android)`.application`) generate resources in bytecode instead of XML format, Paparazzi cannot read these resources. As a result, it is currently necessary to keep composables in a standalone module ([`com.android`](http://com.android)`.library`). You can find more information about this limitation in the following link: [Paparazzi Issue #105](https://github.com/cashapp/paparazzi/issues/105).

In conclusion, screenshot testing using the Paparazzi framework brings great value to the development process. It allows developers to verify the visual appearance of their UI components efficiently and effectively, without the need for an Android emulator. By capturing and comparing screenshots, Paparazzi helps ensure consistent UI behavior, providing a reliable way to catch unintended changes early in the development cycle.

References

Paparazzi documentation: [https://cashapp.github.io/paparazzi/](https://cashapp.github.io/paparazzi/) Paparazzi GitHub repository: [https://github.com/cashapp/paparazzi](https://github.com/cashapp/paparazzi)