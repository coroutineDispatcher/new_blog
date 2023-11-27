---
title: "Conditional navigation and single event LiveData"
date: 2019-12-09T09:51:00.000+01:00
draft: false
aliases: ["/2019/12/conditional-navigation-and-single-event.html"]
tags:
  [
    Single Event,
    Android Architecture Components,
    Android Lifecycle,
    Android Navigation,
    LiveData,
  ]
author: "Stavro Xhardha"
---

[![](https://static.zerochan.net/Juudai.Yuuki.full.2665499.gif)](https://static.zerochan.net/Juudai.Yuuki.full.2665499.gif)

Conditional navigation is a little tricky when it comes to Navigation Architecture. There are plenty of nice articles and solutions about it, but I'm sharing the way I solve this important problem. So let's consider this use case:

[![](https://1.bp.blogspot.com/-cMm5C3r30C4/Xe0VhMFHkNI/AAAAAAAAQq4/ftOp5qnc8e0mpkdXlfkfx6xxaBKFdXBAQCEwYBhgL/s1600/Untitled%2BDiagram.jpg)](https://1.bp.blogspot.com/-cMm5C3r30C4/Xe0VhMFHkNI/AAAAAAAAQq4/ftOp5qnc8e0mpkdXlfkfx6xxaBKFdXBAQCEwYBhgL/s1600/Untitled%2BDiagram.jpg)

If you don't put HomeFragment as a start destination, you will have some serious trouble with the fragment back stack and you would need to write the onBackPressDispatcher in every fragment. Therefore, don't do it.

There is also an easier step to do this directly in the OnlyActivity you have, and it's pretty nice too, but I've noticed that Google suggest the activities to be kept just for holding <fragment>s and nothing more. Otherwise it would violate the responsibility that the new Architecture Activity behaviour has (like checking from its' ViewModel user authentication etc).

So it's just fragments. A common problem you might face is when you use LiveData<T> the usual way for this case. I'm not handling the way google does it [here](https://developer.android.com/guide/navigation/navigation-conditional), because that's already implemented in their documentation. So for instance, I'm just handling a boolean in this case (authenticated or not):

```kotlin
class HomeViewModel @Inject constructor(repository: Repository) : ViewModel(){
  private val _showLogin = MutableLiveData<Boolean>() //false by default
  val showLogin: LiveData<Boolean> = _showLogin

 init{
   val userAuthenticated = repository.checkUserAuthentication()
   if(!userAuthenticated){
    _showLogin.value = true
   }
 }

}

//inside HomeFragment

homeViewModel.showLogin.observe(this, Observer{
  if(it)
   findNavController().navigate(R.id.loginFragment)
})
```

**The problem**  
Now, if your login goes successful and you close the fragment by popping the back stack after the user authentication (findNavController.popBackStack()), the login screen would reappear. So what could have gone wrong? Well since you are just adding the LoginFragment to the back stack, the HomeFragment never gets closed, and so does its' attached ViewModel. When the HomeFragment is back on top, it start observing the LiveData<Boolean> again, only now, its' value is true because the ViewModel has been alive all the time, making it a never ending cycle.

[![](https://1.bp.blogspot.com/-_FQdO95gx4k/Xe1dl49sUmI/AAAAAAAAQrE/xvfUA2BQd3QbkPMWfo26zq7hcEhmkP5RwCLcBGAsYHQ/s1600/ZDPSWSr.gif)](https://1.bp.blogspot.com/-_FQdO95gx4k/Xe1dl49sUmI/AAAAAAAAQrE/xvfUA2BQd3QbkPMWfo26zq7hcEhmkP5RwCLcBGAsYHQ/s1600/ZDPSWSr.gif)

You can fix this however with a small (but really dangerous) hack:

```kotlin
init{
   val userAuthenticated = repository.checkUserAuthentication()
   if(!userAuthenticated){
    _userAuthenticatedFlag.value = true
    _userAuthenticatedFlag.value = false
   }
 }
```

I wouldn't recommend this way, because it's really a hack and not an actual solution. Furthermore, it's not good code and sometimes happens that the second value might not emit at all because of the nature of LiveData.

**Single events ftw:**

Well, the best solution is to use a mix of an EventBus with LiveData. This would be a perfect duo for Fragment + ViewModel design when it comes to single events:

```kotlin
open class Event<out T>(private val content: T) {
    var hasBeenHandled = false
        private set

    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            hasBeenHandled = true
            content
        }
    }

    fun peekContent(): T = content
}

//HomeViewModel

class HomeViewModel @Inject constructor(repository: Repository) : ViewModel(){
  private val _showLoginEvent = MutableLiveData<Event<String>>() //false by default
  val showLoginEvent: LiveData<Event<String>> = _showLoginEvent

 init{
   val userAuthenticated = repository.checkUserAuthentication()
   if(!userAuthenticated){
    _showLoginEvent.value = Event("ShowLoginEvent")
   }
 }

}

//inside HomeFragment

homeViewModel.showLoginEvent.observe(this, Observer{ event ->
    event.getContentIfNotHandled()?.let{
       findNavController().navigate(R.id.loginFragment)
    }
})
```

It doesn't really look complicated at all and it's pretty safe this way.

{{< admonition >}}
Note: Also notice to place LoginFragment as top level destination, otherwise when you would press the back button in your device, you would infinitely get inside the loop of popping up this fragment.
The actual conditions do not depend just from a boolean value. It's easier with states.
{{</ admonition >}}

Now, what if the user doesn't have an account and needs to register. In this case, you would need to track the back stack. Login -> Register -> Confirmation. And backwards Confirmation -> Register -> Login. But there seems to be a problem with this case. If the registration is successful, you might want to log him directly from there without making him/her put login credentials again (this way would just need to press the back button until arrival to Login screen again). If you think about it, you might wanna pop the back stack twice, once for Confirmation Screen and once for Registration. But this would require a bunch of flags, navigation arguments between 2 fragments and a lot of hacks. However, navigation components do solve something really nicely. There is an option in which you can pop the back stack from one fragment until the desired fragment. This can be achieved just by just typing the desired destination in the ConfirmationFragment (in our case). So, we would want to pop the back stack until we arrive to HomeFragment (which will check authentication and do nothing because we are done with that part): findNavController().popBackStack(R.id.home_fragment, false)

And that's it. A small thing to notice is that Google's suggestion with states looks more complicated but it is more reliable as a solution. I was just handling the problem of single events with live data and showing a simpler way for conditional navigation.

#### **Conclusion.**

The navigation components might have complicated a little bit the conditional navigation. However for people who are willing to use single activity, this solution is pretty nice, I guess. If you have more than one activity, this would be a strong reason to keep it that way. A login/register flow and the base app could be achieved with 2 activities with less work, but anyways, I still think that having more than one Activity in your app is redundant.

Credits: Jose Alcérreca with [his article](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150) about single events.  
If you want to check Googles source for conditional navigation, here is the [link](https://developer.android.com/guide/navigation/navigation-conditional).

Stavro Xhardha
