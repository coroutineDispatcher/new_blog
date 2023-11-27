---
title: "Exploring SharedFlow in Kotlin: Understanding tryEmit() and the Importance of Buffers"
datePublished: Mon Nov 27 2023 17:17:50 GMT+0000 (Coordinated Universal Time)
cuid: clph69xrl000z0al2534cd2sn
slug: exploring-sharedflow-in-kotlin-understanding-tryemit-and-the-importance-of-buffers
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1701105424928/82a4f4ab-a67c-47ad-be80-7f9cfc01d341.webp
tags: kotlin, coroutines

---

In Kotlin, `SharedFlow` is a powerful tool for implementing reactive and asynchronous programming patterns. It provides a convenient way to share data streams among multiple subscribers. In this article, we will explore the concept of SharedFlow and understand its behavior in broadcasting emitted values to all collectors. Unlike regular Flows, SharedFlow is *hot*, meaning it exists independently of the presence of collectors. We will delve into its unique characteristics and compare it to the cold nature of regular Flows.

**Understanding SharedFlow:**

`SharedFlow` is a type of Flow that allows for the broadcasting of emitted values to all its collectors in a "broadcast fashion." When values are emitted to a SharedFlow, all active collectors, regardless of their number, will receive the emitted values. This broadcasting behavior distinguishes `SharedFlow` from regular Flows, which are *cold* and require separate instances for each collector.

**Hot vs. Cold Flows:**

To comprehend `SharedFlow` better, it's important to grasp the distinction between hot and cold `Flow`s. A hot Flow, like SharedFlow, remains active regardless of the presence or absence of collectors. It maintains an ongoing stream of data that can be consumed by multiple subscribers concurrently. On the other hand, a cold Flow, defined using the flow { ... } function, is started separately for each collector. Each collector begins its own individual stream of data, independent of other collectors.

**Emitting Values with SharedFlow:**

`SharedFlow` offers two ways to emit values to its collectors: `emit()` and `tryEmit()`. The `emit()` function is suspending. On the other hand, the `tryEmit()` function is non-suspending and returns a Boolean indicating the success or failure of the emission.

Example: Emitting Values with `emit()`

```kotlin
fun main() = runBlocking {
    val valuesFlow = MutableSharedFlow<Int>()

    val collectionJob = launch {
        valuesFlow.collect { value ->
            println(value)
        }
    }

    val emissionJob = launch {
        delay(1000)
        valuesFlow.emit(1)
        delay(1000)
        valuesFlow.emit(2)
        delay(1000)
        valuesFlow.emit(3)
        delay(1000)
        valuesFlow.emit(4)
        delay(1000)
        valuesFlow.emit(5)
    }

    collectionJob.join()
    emissionJob.join()
}
```

Output:

```kotlin
1
2
3
4
5
```

In this code example, we create a `MutableSharedFlow<Int>()` instance called `valuesFlow`. We launch two coroutines: `collectionJob` and `emissionJob`. The `collectionJob` collects the emitted values from `valuesFlow` and prints them. The `emissionJob` emits values using `emit()` at one-second intervals. As a result, all values are printed correctly.

Let's consider another code example that uses `tryEmit()` to emit values:

```kotlin
fun main() = runBlocking {
    val valuesFlow = MutableSharedFlow<Int>()

  launch {
      valuesFlow.collect { value ->
          println(value)
      }
   }

    delay(1000)
    valuesFlow.tryEmit(1)
    delay(1000)
    valuesFlow.tryEmit(2)
    delay(1000)
    valuesFlow.tryEmit(3)
    delay(1000)
    valuesFlow.tryEmit(4)
    delay(1000)
    valuesFlow.tryEmit(5)
    ...
}
```

Output: (No values are printed)

**Why Values are Not Printed with** `tryEmit()`?

When using `tryEmit()` without a buffer, the emitted values are not printed, even though `tryEmit()` returns `true` to indicate a successful emission. This happens because `tryEmit()` is a non-suspending function and cannot operate without a buffer to store the emitted values. On the other hand, `emit()` is suspending, so it can suspend if any of the subscribers are not ready to process the emitted value.

**Adding the** `replay` parameter (The wrong way):

```kotlin
fun main() = runBlocking {
    val valuesFlow = MutableSharedFlow<Int>(replay = 5)

    launch {
        valuesFlow.collect { value ->
            println(value)
        }
    }
    
    delay(1000)
    valuesFlow.tryEmit(1)
    delay(1000)
    valuesFlow.tryEmit(2)
    delay(1000)
    valuesFlow.tryEmit(3)
    delay(1000)
    valuesFlow.tryEmit(4)
    delay(1000)
    valuesFlow.tryEmit(5)
    ...
}
```

The SharedFlow also provides a `replay` parameter that determines the number of previously emitted values new collectors receive upon subscription. By specifying a positive value for replay, collectors can receive a certain number of past values when they start observing the SharedFlow. However, in our case, using the replay parameter does not solve the issue because it requires a fixed number of previously emitted values to be replayed. In real-life scenarios, it's often impractical to determine how many values should be replayed, as it depends on dynamic factors and the specific requirements of the application. Therefore, while the replay parameter can be useful in certain cases, using a buffer with appropriate configuration, as discussed earlier, provides a more flexible and adaptable solution for handling emitted values in the SharedFlow.

**Fixing the Issue with Buffers (The right way):**

To solve the problem of values not being printed with `tryEmit()`, we can add a buffer to the SharedFlow using the `extraBufferCapacity` parameter. This buffer provides space for the emitted values to be stored until the subscribers are ready to process them.

**Adding a Buffer to SharedFlow**:

```kotlin
fun main() = runBlocking {
    val valuesFlow = MutableSharedFlow<Int>(
        extraBufferCapacity = 1, // at least 1
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )

    launch {
        valuesFlow.collect { value ->
            println(value)
        }
    }

    delay(1000)
    valuesFlow.tryEmit(1)
    delay(1000)
    valuesFlow.tryEmit(2)
    delay(1000)
    valuesFlow.tryEmit(3)
    delay(1000)
    valuesFlow.tryEmit(4)
    delay(1000)
    valuesFlow.tryEmit(5)
    ...
}
```

Output:

```kotlin
1
2
3
4
5
```

In this modified code, we added a buffer to `valuesFlow` with a capacity of 1 using the `extraBufferCapacity` parameter. We also specified `onBufferOverflow = BufferOverflow.DROP_OLDEST`, which drops the oldest emitted value when the buffer is full.

BufferOverflow Strategies and Extra Buffer Capacity Constants:

When configuring the buffer for a SharedFlow using the `extraBufferCapacity` parameter, you can also specify the `onBufferOverflow` strategy to handle situations where the buffer becomes full. Here are some common strategies:

* `BufferOverflow.SUSPEND`: This strategy causes the emitter to suspend when the buffer is full, allowing it to resume emitting values once there is space available in the buffer. It ensures that no values are lost, but it can potentially lead to a slowdown in the emission process.
    
* `BufferOverflow.DROP_OLDEST`: With this strategy, when the buffer is full, the oldest emitted value in the buffer is dropped to make room for the new value. This strategy prioritizes the most recent values and ensures that the buffer does not exceed its capacity.
    
* `BufferOverflow.DROP_LATEST`: In contrast to `DROP_OLDEST`, this strategy drops the latest emitted value when the buffer is full. It prioritizes the oldest values in the buffer and maintains the capacity limit.
    

By choosing an appropriate buffer overflow strategy and adjusting the extra buffer capacity, you can control how `SharedFlow` handles situations where the buffer becomes full, ensuring the desired behavior for your specific use case.

**Note: A negative buffer capacity does not work with** `SharedFlows`. They are supposed to crash. Refer to this comment (here)\[[https://dev.to/ampeixoto/comment/27lgf](https://dev.to/ampeixoto/comment/27lgf)\]

**Conclusion:**

`SharedFlow` in Kotlin provides a flexible mechanism for emitting values to multiple collectors. While the `emit()` function suspends to handle subscribers not ready to process values, the non-suspending `tryEmit()` function lacks a buffer and does not print values without additional configuration. By understanding the importance of buffers and adding them to SharedFlow, you can ensure that all emitted values are correctly processed by the subscribers.