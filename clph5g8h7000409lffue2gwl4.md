---
title: "Fragments ❤  ViewPager2"
datePublished: Mon Dec 23 2019 08:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5g8h7000409lffue2gwl4
slug: fragments-viewpager2

---


  
The [ViewPager2](https://developer.android.com/jetpack/androidx/releases/viewpager2) is a pretty nice rework of the [ViewPager](https://developer.android.com/reference/android/support/v4/view/ViewPager) API. Some new features you may find with the ViewPager2 are:  
  
1- Vertical scrolling. You can simply enable it by adding: android:orientation="vertical" in the <ViewPager2> tag in your xml file.  
2- Right to left support: you can set the android:layoutDirection="rtl" to enable this.  
3- Support for [DiffUtil](https://developer.android.com/reference/kotlin/androidx/recyclerview/widget/DiffUtil), because it is based on RecyclerView.  
4 - Fragments improved support, which we will talk about below.  
  
So, let's suppose that we have 2 Fragments: HomeFragment and SchoolFragment. In the Activity/Fragment xml file, which is going to hold these two sliding by switching with each-other, place the ViewPager2 tag:  
  
```xml
<androidx.viewpager2.widget.ViewPager2  
    xmlns:android="http://schemas.android.com/apk/res/android"  
    android:id="@+id/my_view_pager"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >"
```
Now, let's set up code in the MainActivity (in my case this is the name of the Activity):  
  
```kotlin
class MainActivity : AppCompatActivity() {  
  
    private lateinit var viewPager: ViewPager2  
  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContentView(R.layout.activity_main)  
        viewPager = findViewById(R.id.my_view_pager)  
        val tabLayout = findViewById(R.id.tabs)  
        val pagerAdapter = ScreenSlidePagerAdapter(this)  
        viewPager.adapter = pagerAdapter  
        TabLayoutMediator(tabLayout, viewPager) { tab, position ->  
            when (position) {  
                0 -> tab.text = "Home"  
                1 -> tab.text = "School"  
            }  
        }.attach() //this replaces the tabLayout.setupWithViewPager() from ViewPager API  
    }  
  
    override fun onBackPressed() {  
        if (viewPager.currentItem == 0) {  
            super.onBackPressed() //if I'm in the first Fragment just go back  
        } else {  
            viewPager.currentItem = viewPager.currentItem - 1 //just slide to the first Fragment  
        }  
    }  
  
    private inner class ScreenSlidePagerAdapter(fragmentActivity: FragmentActivity) : FragmentStateAdapter(fragmentActivity) {  
        override fun getItemCount(): Int = 2 //because I have two Fragments  
  
        override fun createFragment(position: Int): Fragment = if (position == 0) {  
            HomeFragment()  
        } else {  
            SchoolFragment()  
        }  
    }  
}
```

Some things to spot here though.  
  
1- Notice that you have the FragmentStateAdapter instead of FragmentStatePagerAdapter.  
  
2- To associate TabLayout with the ViewPager there is no more tabLayout.setupWithViewPager(). Instead you should import _com.google.android.material:material:1.1.0-beta01_ or later version of the dependency. And than you can do:  
  
```kotlin
TabLayoutMediator(tabLayout, viewPager) { tab, position ->  
            when (position) {  
                0 -> tab.text = "Home"  
                1 -> tab.text = "School"  
            }  
        }.attach()
```  
And that's it, now you have your shiny new ViewPager2 set up with Fragments.  
  
Helpful resources:  
[Slide between Fragments using Viewpager2.](https://developer.android.com/training/animation/screen-slide-2#kotlin)  
[Turning the Page: Migrating to ViewPager2 (Android Dev Summit '19).](https://youtu.be/lAP6cz1HSzA)  

Stavro Xhardha
