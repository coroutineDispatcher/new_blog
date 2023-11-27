---
title: 'Realm 7, the frozen throne'
date: 2020-05-30T19:52:00.003+02:00
draft: false
aliases: [ "/2020/05/realm-7-frozen-throne.html" ]
tags : [Database, Reactive Programming, Coroutines, Android Realm, Kotlin Flows, Kotlin DSL, Kotlin Coroutines, Realm, Kotlin, Android, Android Development]
author: "Stavro Xhardha"
---

  

[![](https://1.bp.blogspot.com/-aouAB28tBgQ/XtKGgZofynI/AAAAAAAAT0U/oOlK2xPnm3Y_BHflX-db4EFTHjjoUjTDwCK4BGAsYHg/d/noaa-z_9pgECtOvY-unsplash.jpg)  
](https://1.bp.blogspot.com/-aouAB28tBgQ/XtKGgZofynI/AAAAAAAAT0U/oOlK2xPnm3Y_BHflX-db4EFTHjjoUjTDwCK4BGAsYHg/noaa-z_9pgECtOvY-unsplash.jpg)

  

  
[](https://1.bp.blogspot.com/-aouAB28tBgQ/XtKGgZofynI/AAAAAAAAT0U/oOlK2xPnm3Y_BHflX-db4EFTHjjoUjTDwCK4BGAsYHg/noaa-z_9pgECtOvY-unsplash.jpg)

onCreate
--------

As many of us might know, Realm has already introduced [freezing objects](https://realm.io/docs/kotlin/latest/#freezing-objects). Personally, I have been waiting for long time for such feature. So, what actual problem does this solve?

A lot of us might have faced this issue:


```plain
Realm access from incorrect thread. Realm objects can only be accessed on the thread they were created.
```

I believe the error speaks for itself.

### Realm < 7.0:

When you call `Realm.getDefaultInstance()` you are not creating a new object every time you call this method. However, having it on every method is somehow ugly. In cases when you need the object in another thread, it is impossible to use it without a DTO helper data class.

Let's consider the following case:

```kotlin
private lateinit var realm: Realm  
...  
override var onCreate(){  
 ...  
 realm = Realm.getDefaultInstance() // Main thread  
}  
   
suspend fun doSomething() = withContext(Dispatchers.IO){  
    val result = realm.where<MyRealmObject>().findFirst() //Crash  
    result?.let {  
        Timber.d("${it.id}")  
  	}  
 }
```

Every time we have to start a realm interaction, we have to call `Realm.getDefaultInstance()` before, every time you are in a new thread. Otherwise, it would bring the error mentioned above. If you want a sinlge, app-scope `Realm` instance, you have to make [some workaround](https://stackoverflow.com/questions/46334631/dagger-2-should-i-use-a-singleton-realm-instance).

Same would have happened if you were to read an `RealmObject` or a `RealmResults<T>`:

```kotlin
private lateinit var realm: Realm  
private var myObject: MyRealmObject? = null  
...  
override var onCreate(){  
 ...  
 realm = Realm.getDefaultInstance() // Main thread  
 myObject = realm.where<MyRealmObject>().findFirst()  
}  
  
suspend fun doSomething() = withContext(Dispatchers.IO){  
    result?.let {  
        Timber.d("${it.id}") // Crash  
    }  
 }
```

### Realm >= 7.0:

Since this would be a huge obstacle, especially to reactive programming, the Realm team is bringing a new actor in the game: The frozen Realm. Just a new object which can be used across threads, attached to the only instantiated Realm object.

```kotlin
private lateinit var liveRealm: Realm  
private lateinit var frozenRealm: Realm  
   
override var onCreate(){  
...  
  realm = Realm.getDefaultInstance() // Main thread  
  frozenRealm = realm.freeze()   
}  
  
suspend fun doSomething() = withContext(Dispatchers.IO){  
  val result = frozenRealm.where<MyRealmObject>().findFirst()  
  result?.let {  
    Timber.d("${it.id}") // Works  
  }  
}  

```

You can do the above solution, or you can directly freeze the `RealmResults<T>`or directly freeze the object (must be a `RealmObject`). This would be even easier:

```kotlin
private lateinit var realm: Realm  
private var myObject: RealmResults? = null  
   
override var onCreate(){  
 ...  
 realm = Realm.getDefaultInstance() // Main thread  
 myObject = realm.where<MyRealmObject>().findAll().freeze()  
   
}  
   
suspend fun doSomething() = withContext(Dispatchers.IO){  
    myObject?.let {  
        Timber.d("${it.id}") // Works  
    }  
}
```

There are a few more additions to this new feature, but the docs are already perfect for that.

Are coroutines really necessary for this example?
-------------------------------------------------

Not at all, but coroutines give a nice presentation of thread switch with `withContext` block. Realm has its own thread usage:

```kotlin
val result = realm.where<MyRealmObject>().findAllAsync()  
```

The `findAllAsync()` eliminate the necessity to have other kinds of thread pools. However, combining realm with `Flow` (particularly `callbackFlow`) would be one good choice if you want to go more reactive. Let us consider the following case: Select some data from the database and every time data changes, a new toast would appear.

The imperative way:

```kotlin
// In a ViewModel  
fun getData(){  
  val result = realm.where<MyRealmObject>().findAllAsync()  
  result.addChangeListener{ result ->  
      _toastLiveData.value = ShowToastFlag  
  }  
}
```

As an example is more than enough to eliminate usage of coroutines. But let's consider this following case: Fetch a list of objects locally, and make a POST request on the network after serializing it to JSON.

```kotlin
suspend fun sendListToNetwork() = withContext(Dispatchers.IO){  
   val result = frozenRealm.where<MyRealmObject>().findAll()  
   // We are in another thread because of the Network Request and not because of Realm  
   result.addChangeListener{ mResult ->  
   val jsonStringResultObject = serialize(result.toList())  
   val response = request().postRequest(jsonStringResult).execute()  
   when(response){  
      is Success -> doSomething()  
      else -> doSomethingElse()  
   }  
}
```

If the case would have been longer, we would have had a more difficult code to read.

### Let's try to make this more Reactive

Let's make a generic method for all Realm objects that are going to be selected as a list:

```kotlin
inline fun  observeData(): Flow> = callbackFlow{  
   val result = frozenRealm.where<T>().findAllAsync()  
   result.addChangeListener{ mResult ->  
    offer(result.toList())  
   }  
   awaitClose {result.removeAllChangeListeners()}  
}
```

It's time to observe the magic of this new feature with combination to coroutines:

```kotlin
// inside the ViewModel  
suspend fun sendDataToNetwork(){  
  observeData().flowOn(Dispatchers.IO)  
  .map {  
    return@map serialize(it)  
  }.flowOn(Dispatchers.Default) // same Realm object, different threads  
  .map {  
      return@map postRequest(it).execute()  
  }.flowOn(Dispatchers.IO)  
  .collect { networkResult ->  
      handleResult(networkResult)  
  }  
}
```

And that's it.

onDestroy
---------

Without the Realm new feature, this would have been impossible. Instead, we would have wasted DTOs in order to hold reference to each selection that we could get. Allowing to share object across thread is a very convenient and necessary feature to have. Saving time and code is not the only factor that is being touched. This new feature opened a new way to reactive programming also.

Stavro Xhardha