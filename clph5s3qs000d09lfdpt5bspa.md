---
title: "Why LiveData is the best solution (yet) for UI"
datePublished: Sat May 23 2020 16:41:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5s3qs000d09lfdpt5bspa
slug: why-livedata-is-the-best-solution-yet-for-ui

---


  
Why LiveData is the best solution (yet) for UI

As far as I am concerned, there are many developers who don't like LiveData. However, for the UI part, there is no better API to achieve MVVM (or even MVI,or stateful MVVM). Furthermore, makes the UI control very easy.  

There might have been some small misconceptions regarding the usage of it, but, if you use LiveData for the purpose it was created, I think that's far better than any other API (for the moment).

Some general misconceptions
  

- Should I use LiveData over Flow for network calls?

- Should I use LiveData over RxJava for network calls?

- Should I use LiveData for the Database? (somehow accepted, I disagree)

- Should I replace everything with LiveData?


The right questions to ask:

- Can I use Flow/RxJava instead of LiveData for the UI?
  

And honestly the answer would be a big YES, and it's a nice pattern to use also. But, my strongest point here is that they both were not created for the purpose that they are being used for in this case.


## The LiveData API
  

First of all, let's introduce a myth: LiveData is complicated. Perhaps, the "back-end" of it might be, but that's one of the purposes when someone provides a dependency, and it's not only to reduce boilerplate, but to make the development simpler, even though building it might be complex. If we take Retrofit as an example, it's just a less boilerplate Okhttp thing (or asI like to call it: abstraction over OkHttp). But if we take Android ViewModels as an

example, there is nothing simple behind it. But we just extend a ViewModel....


And the same case with LiveData. Remember, nothing is too simple when it comes to Android and the Observable pattern in general. If you are interested to check the LiveData, [here](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-master-dev:lifecycle/lifecycle-livedata-core/src/main/java/androidx/lifecycle/LiveData.java;l=59?q=LiveData&sq=&ss=androidx%2Fplatform%2Fframeworks%2Fsupport) is the code of it.
  
So, LiveData is simple.


Let's check what I mean with that:

```kotlin
private val _userFirstName = MutableLiveData<String>()

val userFirstName: LiveData<String> = _userFirstName
```

And in order to observe it (from Fragment/Activity):

  
```kotlin
 viewModel.userFirstName.observe(viewLifecycleOwner, Observer{ result ->
     userNameTextView.text = result
 })
```
  

That's it. This is basic observable pattern.

You have forgotten to unsubscribe at some point.

Actually no. And this is really a strong second argument: LiveData is lifecycle aware.

You don't need to (un)subscribe in order to prevent possible issues.

  
{{< admonition >}}
Note: For specific cases, you can use removeObserver() also.
{{< /admonition >}}
  

Can't say it's thread safe, but it actually is.

  

You are free not to switch to the main thread and instead just use the postValue() method:
```kotlin
 withContext(Dispatchers.IO){

 _userName.value = repository.getUserName() //WON'T WORK

 }

 withContext(Dispatchers.IO){

 _userName.postValue(repository.getUserName())

 }
 ```

Less verbose

Even though RxJava and StateFlow are already pretty nice option to replace LiveData with, they have more boilerplate than LiveData in whatever the case.

Let's consider RxJava for example:

  
```kotlin
private val compositeDisposable = CompositeDisposable() 

fun observeViewModelChanges(){

 compositeDisposable.add(viewModel.usernameOservable())

 } 

override fun onDestroy(){

 compositeDisposable.dispose()

 super.onDestory()

 }
```
  

Same case with StateFlow:

```kotlin
fun observeViewModelChanges(){

 myViewModel.myStateFlow()

    .map{userName -> handleUserName() }

 .launchIn(lifeCycleScope)

 }
```
  

Or worse, in case we don't are not using lifecyclescope:

```kotlin
private val job = Job()

private val coroutineScope = CoroutineScope(Dispatchers.Main + job)
  ...  

fun obserViewModelChanges(){

 myViewModel.obserValuesStateFlow()

    .map{userName -> handleUserName() }

 .launchIn(coroutineScope)

 }
  ...  

override fun onDestory(){

 coroutineScope.cancel()

 super.onDestroy()

 }
 ```

While in LiveData there is at least one line less:

  ```kotlin
  viewModel.userFirstName.observe(viewLifecycleOwner, Observer{ result ->  

   handleResult(result)

  })
``` 

Another small misconception:

If I'm right, why is Google making [tutorials](https://github.com/googlecodelabs/kotlin-coroutines) on Flow by removing LiveData? I'm pretty sure that's only for the sake of the tutorial and learning something new. However, there might be some places where LiveData is worth being replaced with Flow API, like Room database calls, for instance.

  

## onDestroy

In my opinion, LiveData is not going anywhere for some time. Perhaps, we could see some new LiveData which implements StateFlow on the background, in the future, (making a new lifecycle aware StateFlow API) from the Android team, but that could be definitely after the StateFlow exits the Experimental status. Use LiveData and you will have a simpler everyday development.

Stavro Xhardha
