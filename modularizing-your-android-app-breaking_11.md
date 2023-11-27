---
title: "Modularizing your Android app, breaking the monolith (Part 2)"
date: 2019-11-11T09:15:00.000+01:00
draft: false
aliases: ["/2019/11/modularizing-your-android-app-breaking_11.html"]
tags:
  [
    Android Module,
    Micro Frontend,
    Dagger 2,
    Android Architecture Components,
    Android Multi Module,
  ]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-AoX_BAle_hA/XcKhq_qC_eI/AAAAAAAAQNw/fEcEr6z7AwgN29tQZjf6ApJUrYyX5rU4wCLcBGAsYHQ/s1600/jack-anstey-XVoyX7l9ocY-unsplash.jpg)](https://1.bp.blogspot.com/-AoX_BAle_hA/XcKhq_qC_eI/AAAAAAAAQNw/fEcEr6z7AwgN29tQZjf6ApJUrYyX5rU4wCLcBGAsYHQ/s1600/jack-anstey-XVoyX7l9ocY-unsplash.jpg)

_This is part 2 of a series of articles about modularizing Android app. If you haven't yet read the first article, you may find it [here](https://www.coroutinedispatcher.com/2019/11/modularizing-your-android-app-breaking.html)._  
On our first article we just moved some classes outside the application and applied as an independent module. But what if we have dependencies pulled from the application level? This could be a small challenge. First of all, we want to clarify on how are we going to modularize the app. And depending on the previous article, I chose the _by feature_ version of modularization. First of all, let's show some dependencies that are going to be needed in the whole app.

_Note: I'm using Dagger for handling dependencies but manual DI or any dependency tool should be fine to understand this part._

So, this is my dependency schema:

[![](https://1.bp.blogspot.com/-J2BZc0H6ykk/XcKp5gkJ4gI/AAAAAAAAQN8/CCIWlf9O_G8G-_netHp6EEa2Wd0GXzgTwCLcBGAsYHQ/s1600/AppComponentNotBroken.jpg)](https://1.bp.blogspot.com/-J2BZc0H6ykk/XcKp5gkJ4gI/AAAAAAAAQN8/CCIWlf9O_G8G-_netHp6EEa2Wd0GXzgTwCLcBGAsYHQ/s1600/AppComponentNotBroken.jpg)

Well, it's not that bad, but this isn't what we want to transform to when trying to modularize the app. If you think about it, modules that don't need a dependency, can get it quite easily. For example: A FeatureXViewModel is just making a API request and doesn't care at all about my local database.

First of all, I'm going to need some base things which I will put them in to a :core_module. These would be dependencies and components that are required all over the app. That would be my CoreApplication class, where the CoreComponent is going to stay, my MainActivity and the **launching fragment** of my app (personal choice, you can add this as another module too, or you can work on instant apps etc which won't be covered here). After doing so, the :core module is going to be installed in the :app_module and in every single feature which depends on the above elements:

[![](https://1.bp.blogspot.com/-4Q1dGsGjjyg/XcKxcluarYI/AAAAAAAAQOI/TbjawT21uu4sWzO2K9CVL6c-qJx5VHcfgCLcBGAsYHQ/s1600/AppComponentNotBroken%2B%25281%2529.jpg)](https://1.bp.blogspot.com/-4Q1dGsGjjyg/XcKxcluarYI/AAAAAAAAQOI/TbjawT21uu4sWzO2K9CVL6c-qJx5VHcfgCLcBGAsYHQ/s1600/AppComponentNotBroken%2B%25281%2529.jpg)

_Notes:  The Application class should stand in the CoreModule._  
_ Feature 6 doesn't have any dependency thus we are not installing anything over there, we just place it in the :app_module in order to run._  
_When modularizing don't forget all the res files and the **required dependencies in each new module you create**. _  
_All my so called features are {Fragments + ViewModel + Repository (if necessary)}._  
_Small clarification: the capitalized letter **Module** refers to Dagger modules while the non capitalized **module** refers to application modules._

And now let's literally modularize:

[![](https://1.bp.blogspot.com/-wsG12r7rwpQ/XcK3C7G5YiI/AAAAAAAAQOU/rNSd-Ac5D7o0PrMLaCiMOxKMpfN4NXEJwCLcBGAsYHQ/s1600/AppComponentNotBroken%2B%25282%2529.jpg)](https://1.bp.blogspot.com/-wsG12r7rwpQ/XcK3C7G5YiI/AAAAAAAAQOU/rNSd-Ac5D7o0PrMLaCiMOxKMpfN4NXEJwCLcBGAsYHQ/s1600/AppComponentNotBroken%2B%25282%2529.jpg)

Now, the only thing that would immediately break is the ViewModelModule, and that's because :core_module has no idea of my other features ViewModels. So now, we need to set up dagger, for each feature. To set up dagger for multi module project, I held myself loyal to the Google's official documentation about [Setting up Dagger in a Multi Module project](https://developer.android.com/training/dependency-injection/dagger-multi-module), which was a good starting point.  
_TL;DR of the documentation: You need more than one scope._

So now, let's provide a ViewModel for every feature:

```kotlin
//pretending we are inside a module called :gallery_module
@Component(dependencies = [CoreComponent::class])
@FragmentScoped
interface GalleryComponent {
    val galleryViewModelFactory: GalleryViewModel.Factory

    @Component.Factory
    inteface Factory{
      fun create(coreComponent: CoreComponent): GalleryComponent
    }
}

//after build, you can do this in your fragment:

//inside onActivityCreated:
galleryComponent = DaggerGalleryComponent.factory(applicationComponent).create()

//in the variable declaration
val galleryViewModel by savedStateViewModel{
  galleryComponent.galleryViewModelFactory.create(it)
}
```

To know how I end up in details to this solution about my ViewModel (and why I am providing a Factory like that), please refer to this article [here](https://www.coroutinedispatcher.com/2019/08/how-to-produce-savedstatehandle-in-your.html). The only unclear thing is how we brought that applicationComponent. Well, it's just brought from an abstract class from the :core_module which extends Fragment and holds this variable which has a reference to the CoreComponent. Therefore, every fragment will extend a BaseFragment abstract class.

**We are not done yet.**

Now, doesn't that seem to much work for every feature just for the ViewModel? I require from you to be patient as we are covering this in the next part. The problem is simple: We still haven't solve the feature independence problem, we just set up dagger and made a built without errors. If you notice, :core_module is installed in all features (except one), which means that a feature which doesn't need a dependency still knows everything about it. Therefore, the next step will be to fix that problem.

Stavro Xhardha
