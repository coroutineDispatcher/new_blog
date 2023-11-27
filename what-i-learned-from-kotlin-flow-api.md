---
title: 'What I learned from Kotlin Flow API'
date: 2020-01-24T13:54:00.000+01:00
draft: false
aliases: [ "/2020/01/what-i-learned-from-kotlin-flow-api.html" ]
tags : [Observables, Coroutines, Android Architecture Components, Kotlin Flows, LiveData, Asynchronous, Kotlin, Android]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-RkVRvfUOguU/XirklPNZgKI/AAAAAAAARRA/02ijqmfVQskx0mSBd_umY2AoR8Wh8rqBQCLcBGAsYHQ/s1600/tenor.gif)](https://1.bp.blogspot.com/-RkVRvfUOguU/XirklPNZgKI/AAAAAAAARRA/02ijqmfVQskx0mSBd_umY2AoR8Wh8rqBQCLcBGAsYHQ/s1600/tenor.gif)

I used to check the docs and just read a lot about flows but didn't implement anything until yesterday. However, the API tasted really cool (even though some operations are still in Experimental state).

_Prerequisites: If you don't know RxJava it's fine. But a RxJava recognizer would read this faster._

**Cold vs Hot streams**

Well, I really struggled with this concept because it is a little bit tricky. The main difference between cold and hot happened to be pretty simple: Hot streams produce when you don't care while in cold streams, if you don't `collect()` (or RxJava-s equivalent `subscribe()`) the stream won't be activated at all. So, Flows are what we call cold streams. Removing the subscriber will not produce data at all, making the Flows one of the most sophisticated asynchronous stream API ever (in the JVM world).

I tried to make a illustration of hot and cold streams:

[![](https://1.bp.blogspot.com/-tVIgw0QHIuM/XirP-gDj3rI/AAAAAAAARQg/e1ehBZfopmgJJHomA42_GfaifCvJcvNdgCLcBGAsYHQ/s1600/Streams.png)](https://1.bp.blogspot.com/-tVIgw0QHIuM/XirP-gDj3rI/AAAAAAAARQg/e1ehBZfopmgJJHomA42_GfaifCvJcvNdgCLcBGAsYHQ/s1600/Streams.png)

Since I mentioned the word asynchronous this implies that they do support coroutines also.

**Flows vs and LiveData**

Having two kind of observables in your project, LiveData and Flows does not complicate the matter, it just creates a little bit of confusion. I was confused too in the beginning seeing a lot of discussion around LiveData vs Flows. Here is what I learned:

The LiveData is still the one thing to be used to observe the UI changes. What I got wrong in all this discussion was that I thought we should replace LiveData and Flows for this case, which is wrong.

The right thing in a debate like this would be LiveData vs Flows for data processing (like a Room persistence observation). Apparently we know who has the upper hand on this 🌊.

**The use case**

I used this [codelab](https://codelabs.developers.google.com/codelabs/advanced-kotlin-coroutines/#0) to learn Flows in practice. The use case was simple: Receive some data from the network, cache them to the room database and observe changes through LiveData (and load them into a `Recyclerview`, 60% of Android development 😜). Since I had nothing new in LiveData I didn't struggle at all. The task was to change all the LiveData transformations/processing etc into Flow implementation.

Let's do it. Starting from the source of truth we would have this:

```kotlin
@Query("SELECT * FROM plants ORDER BY name")  
fun getPlants(): LiveData<List<Plant>>
```

Which transforms into:

```kotlin
@Query("SELECT * from plants ORDER BY name")  
fun getPlantsFlow(): Flow<List<Plant>>
```

Now, let's perform some operation with the result on our repository:

```kotlin
val plants = liveData<List<Plant>> {  
    val plantsLiveData = plantDao.getPlants()  
    val customSortOrder = plantsListSortOrderCache.getOrAwait()  
    emitSource(plantsLiveData.map { plantList ->  
      plantList.applySort(customSortOrder)  
    })  
  }
```

This is a LiveData builder, which would transform into:

```kotlin
private val customSortFlow = plantsListSortOrderCache::getOrAwait.asFlow()  
  val plantsFlow: Flow<List<Plant>>  
    get() = plantDao.getPlantsFlow().combine(customSortFlow) { plants, sortOrder ->  
      plants.applySort(sortOrder)  
    }  
      .flowOn(Dispatchers.Default) //I changed this for the sake of the guideline  
      .conflate()
```

Now, this is a little bit tricky. Notice the first block that has a LiveData builder. Two operations are being processed here: The first operation is getting safely some data from the network and sorting them by a certain field (id to be more precise), after that we just apply some sorting by a local method which sorts the list by another field. Since the goal of this guide is to show as many operations as they can, they introduce us to the `.combine()` operator and `.asFlow()` extension.

If you come from RxJava the `combine()` is the equivalent of `zip()`. The `asFlow()` extension is a function which allows you to return everything you want into a Flow:

```kotlin
public fun <T> (suspend () -> T).asFlow(): kotlinx.coroutines.flow.Flow<T> { /* compiled code */ }
```

But if we notice we have more operators: `flowOn()`, and `conflate()`. `flowOn()` is actually RxJava's equivalent of `subscribeOn()`. What this operator does is switching the thread to the desired Dispatcher. Since the `applySort()` is just a CPU process and no disk/network operation, we are using `Dispatchers.Default` thread for this. The `conflate()` operation is just used to get the latest value instead of all the values during the operation (should read a little bit about `buffer()` and slow operations [here](https://kotlinlang.org/docs/reference/coroutines/flow.html#buffering) though).

What's left now? Ah, the ViewModel:

```kotlin
val plants: LiveData<List<Plant>> = plantRepository.plants 
//the original has some other operations but for this case we don't care, just return a LiveData
```

Transforms to:

```kotlin
val plantsUsingFlow: LiveData<List<Plant>> = plantRepository.plantsFlow.asLiveData()
```

The missing part for me was the `.asLiveData()` extension. Yes there is an extension function which you can use for every Flow in order to observe LiveData on the Fragment:

```kotlin
@JvmOverloads  
fun <T> Flow<T>.asLiveData(  
  context: CoroutineContext = EmptyCoroutineContext,  
  timeoutInMs: Long = DEFAULT_TIMEOUT  
): LiveData<T> = liveData(context, timeoutInMs) {  
  collect {  
    emit(it)  
  }  
}
```

And all is done. Of course, I didn't cover the second part of the course which was about handling some click event from UI and re-sorting the list, but you can follow the course if you like. It's pretty fun.

Extra material

During the course you would face some other operations:

```kotlin
.flatMapLatest{result -> process(result)}
```

Basically just returns a new Flow each time a method returns a value.

```kotlin
.onStart{doSomethingOnStart()}
```

Well the name speaks for itself. An operation which executes when the stream starts

```kotlin
.onCompletion{doSomething()}
```
This is even easier, operation that executes when the stream ends

```kotlin
.launchIn(viewModelScope)
```

The method that helps to define the coroutine scope for this flow (inside it you can find the `collect()` method) making this method the equivalent of `launch{collect()}`

## Conclusion (what's the next step)

I would recommend hard work and reading the Flow API [over here](https://kotlinlang.org/docs/reference/coroutines/flow.html). Once you have the concept, it's just operations you need to learn how-to and why. Just think of the `Flow<T>` as a `AsyncStreamingList<T>` which has some nice operations about the data. Flows apply to functional programming style and also shorten a lot of boilerplate if you use them right.

Make sure not to over engineer things with Flow because that would be a total shot in the foot.

First of all, I would recommend some really nice resources to learn Flows just like I did. One thing to note is that I'm still no expert in this, and I'm opened for critics. Here are some of the resources I used (except the codelab mentioned above):

1 - Roman Elizarov's [blog](https://medium.com/@elizarov).

2 - Asynchronous data streams with Kotlin Flow in [YouTube](https://www.youtube.com/watch?v=tYcqn48SMT8).

3 - Fragmented podcast [last talk](https://fragmentedpodcast.com/episodes/187/) (even though they talk more about coroutines in general).

Stavro Xhardha