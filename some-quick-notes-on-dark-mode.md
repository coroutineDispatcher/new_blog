---
title: "Some quick notes on Dark Mode"
date: 2019-09-09T10:03:00.000+02:00
draft: false
aliases: ["/2019/09/some-quick-notes-on-dark-mode.html"]
tags:
  [
    Android Architecture Components,
    Dark theme Android,
    Kotlin,
    Dark Mode,
    Android Development,
  ]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-ur3CRpODGmw/XXVKC9GW7YI/AAAAAAAAPKY/KzYcwcteoqERxQCXTRY0aNi2fQCFumyzACLcBGAs/s1600/elliott-engelmann-DjlKxYFJlTc-unsplash.jpg)](https://1.bp.blogspot.com/-ur3CRpODGmw/XXVKC9GW7YI/AAAAAAAAPKY/KzYcwcteoqERxQCXTRY0aNi2fQCFumyzACLcBGAs/s1600/elliott-engelmann-DjlKxYFJlTc-unsplash.jpg)

The dark mode, perhaps is one of the easiest functionality to implement without breaking literally anything in existing project. However, it has it's own hidden costs, tricks. Before implementing the Dark mode, what is most important is that your project must be ready for dark mode.

**What I had:**

[![](https://1.bp.blogspot.com/-oh1b-Pl6zXQ/XXVLyYcOPRI/AAAAAAAAPKk/UXJJmGpjnBUL4L_rTP8IR1pZ3NtjytNmwCLcBGAs/s400/Screenshot_20190908-204023__01.jpg)](https://1.bp.blogspot.com/-oh1b-Pl6zXQ/XXVLyYcOPRI/AAAAAAAAPKk/UXJJmGpjnBUL4L_rTP8IR1pZ3NtjytNmwCLcBGAs/s1600/Screenshot_20190908-204023__01.jpg)

**What I wanted to achieve:**

[![](https://1.bp.blogspot.com/-BnND1T8D_og/XXVL25pkwkI/AAAAAAAAPKo/3IsLKRX9B5QnqLNE6xFh7DTy8qopaM_WQCLcBGAs/s400/Screenshot_20190908-204008__01.jpg)](https://1.bp.blogspot.com/-BnND1T8D_og/XXVL25pkwkI/AAAAAAAAPKo/3IsLKRX9B5QnqLNE6xFh7DTy8qopaM_WQCLcBGAs/s1600/Screenshot_20190908-204008__01.jpg)

**The setup:**  
The only thing you need to do to get your app ready for dark mode is the themes tag and the AppCompatDelegate class. The rest depends on your project. So instead of some <style name\="AppTheme" parent\="Theme.AppCompat.Light.DarkActionBar"> you just need the <style name\="AppTheme" parent\="Theme.AppCompat.DayNight"> . The code for dark mode/light mode is pretty simple:

```kotlin
//dark mode
AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)

//light mode
AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO)

//asking if dark mode is on:
AppCompatDelegate.getDefaultNightMode() == AppCompatDelegate.MODE_NIGHT_YES

//asking if light mode is on:
AppCompatDelegate.getDefaultNightMode() == AppCompatDelegate.MODE_NIGHT_NO
```

_Note: These are not the [only states.](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme) _

**The problems I faced:**

**The drawer:**  
First of all,  if you notice the drawer on the Light mode, you will see that the text of the menu items, is black. I don't really want that, because in dark mode it will truly suck. Since you cannot change it from xml, here is what you can do for it:

```kotlin
private fun checkNavView() {
        val colorListItems = ColorStateList(
            arrayOf(
                intArrayOf(-android.R.attr.state_enabled),
                intArrayOf(android.R.attr.state_enabled)
            ),
            intArrayOf(
                ContextCompat.getColor(this, R.color.md_black_1000),
                ContextCompat.getColor(this, R.color.colorAccentDark)
            )
        )

        val colorListText = ColorStateList(
            arrayOf(
                intArrayOf(-android.R.attr.state_enabled),
                intArrayOf(android.R.attr.state_enabled)
            ),
            intArrayOf(
                ContextCompat.getColor(this, R.color.md_black_1000),
                ContextCompat.getColor(this, R.color.md_white_1000)
            )
        )

        if (AppCompatDelegate.getDefaultNightMode() == MODE_NIGHT_YES) {
            nav_view.itemTextColor = colorListText
            nav_view.itemIconTintList = colorListItems
        }
    }
```

** Custom inserted colors:**  
There is two options in this part of the problem. Well, my background colors or my text colors had a color set by the xml and not by the theme. So what you can do here is choose colors that don't really affect the visibility on theme change, or leave Androids default.

Since I am using a single Activity and making these changes in the activity (because the drawer is located there) there is no more need to handle things in fragments (except when having to replace an icon depending on the theme).

**The toolbar:**  
Yes. We should be careful with the toolbar. Once you implement the dark theme, you may not want your toolbar to have a black text as well. So you just change it's behavior to:

```kotlin
private fun checkToolbar() {
        if (AppCompatDelegate.getDefaultNightMode() == MODE_NIGHT_YES) {
            toolbar.context.setTheme(R.style.ThemeOverlay_MaterialComponents_Dark)
        } else {
            toolbar.context.setTheme(R.style.ThemeOverlay_MaterialComponents_Light)
        }
    }
```

**The savedInstanceState.**  
One mistake I used to make during my implementation was that I tried to recreate() the activity in order for the onSavedInstanceState to be called. No need for that, once you call AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.WHATEVER_MODE) the activity will recreate for you. Once the activity is recreated, in your onCreate method you can check for the drawer or toolbar as discussed above.

**Is there a better approach?**  
Perhaps yes. You may try to code all this from the theme tag on the styles.xml. But since xml is not my thing,  I chose this approach.

Hope I helped. Full project on [Github](https://github.com/coroutineDispatcher/pocket_treasure).
