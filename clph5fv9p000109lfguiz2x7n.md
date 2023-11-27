---
title: "Coroutines and callbacks"
datePublished: Sun Apr 05 2020 12:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5fv9p000109lfguiz2x7n
slug: coroutines-and-callbacks

---


  
Callbacks in Java are probably the most basic way to perform executions/send actions between classes. If you have chosen to use coroutines in a project, you want to keep the same style everywhere. But what if some of the libraries you use, are still using callbacks?  
  
No worries, Kotlin coroutines are easily integrated with callbacks. It's just a small workaround to make.  
  
Let's think of a very simple use case:  

```kotlin
interface MyAwesomeInterface<T> {  
    fun executeAsync(callback: (T?, Throwable?) -> Unit)  
}  

```  

In this case, what can be done, is to wrap it in a suspendCoroutine. So, le'ts build an extension function for it:  


```kotlin
suspend fun <T> MyAwesomeInterface<T>.execute(): T =  
    suspendCoroutine { continuation -> //notice the continuation that you get  
        executeAsync { value, exception ->  
            if(exception != null)  
                continuation.resumeWithException(exception)  
            else  
                continuation.resume(value as T)  
        }  
    }
```  

The continuation that you get from the suspendCoroutine is nothing else than just a representation of the value returned by your own interface implementation.  
  
But what if our MyAwesomeInterface has a cancel operation? Consider the example below:  

```kotlin
interface MyAwesomeInterface<T> {  
    fun executeAsync(callback: (T?, Throwable?) -> Unit)  
    fun cancel()  
}
```  

In this case, what comes in handy is the suspendCancellableCoroutine. It is nearly the same as the suspendCoroutine but it offers a new aditional operation:  


```kotlin
suspend fun <T> MyAwesomeInterface<T>.execute(): T =  
    suspendCancellableCoroutine { continuation ->  
        executeAsync { value, exception ->  
            if(exception != null)  
                continuation.resumeWithException(exception)  
            else  
                continuation.resume(value as T)  
        }  
       continuation.invokeOnCancellation { cancel() }  
    }
``` 

{{< admonition >}}
These implementations are single-shot operation.
{{< /admonition >}}
  
But what if you would have to observe an asynchronous stream of values through callbacks? This can be achieved with the help of the Flow API. If you still haven't check it up, the [docs](https://kotlinlang.org/docs/reference/coroutines/flow.html) are pretty nice, or you can just go quickly though [this](https://www.coroutinedispatcher.com/2020/01/what-i-learned-from-kotlin-flow-api.html) article (even though it's mostly for Android developers).  
  
In this case, the callbackFlow comes to save the day:  
  
```kotlin
 fun <T : Any> MyAwesomeInterface<T>.execute(): Flow<T> =  
   callbackFlow {  
       executeAsync { value, exception ->  
            when {  
                exception != null -> close(exception)  
                value == null -> close()  
                else-> offer(value as T)  
            }  
        }  
        awaitClose{ cancel() }  
    }
```  

There are some important things to note here:  
 - Our execute() extension function is no longer a suspend function  
 - The close() is used to notify for a failure/success  
 - The offer() is called each time a new data has arrived.  

{{< admonition >}}
Not calling the awaitClose() would take you directly to ClosedSendChannelException if the method is executed n+1 times._  
{{< /admonition >}}

**But my interface doesn't have a cancel() method.**  
  
Than just unregister the listener. The cancel() method is there also to represent  the context of removing it.  
  
**A few notes about backpressure.**  
  
In cases when the operation happens too fast and the coroutine cannot handle it, the best thing is to replace the offer() method with sendBlocking().  
  
In the opposite case, when the values should not arrive too fast, just put a .buffer(Channel.UNLIMITED) after the callbackFlow:  

```kotlin
 fun <T : Any> MyAwesomeInterface<T>.execute(): Flow<T> =  
   callbackFlow {  
       executeAsync { /* same thing here */}  
        awaitClose{ cancel() }  
    }.buffer(Channel.UNLIMITED)
```  
Don't forget that this case, comes with a memory cost.  
  
**Conclusion:**  
  
Actually, these feature of callbacks are awesome and pretty safe in my opinion. They open a wide area of possibilities to improve your libraries, create a new one, or even shorten some parts of code from your current project.  
  
Stavro Xhardha
