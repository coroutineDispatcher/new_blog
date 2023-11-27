---
title: 'Solving Room "cannot find setter for field" error in build time'
date: 2019-08-20T13:55:00.002+02:00
draft: false
aliases: ["/2019/08/solving-room-cannot-find-setter-for.html"]
tags: [Android Room, Android, cannot find setter for field, Room Persistence]
author: "Stavro Xhardha"
---

[![](https://1.bp.blogspot.com/-U7d19cdLM_4/XVvdf_PApeI/AAAAAAAAO9E/CbjeihCRP1YHfP7fWMrJ2OcY-s9uIpVigCLcBGAs/s1600/cristian-baron-dPFaq7RVzbQ-unsplash.jpg)](https://1.bp.blogspot.com/-U7d19cdLM_4/XVvdf_PApeI/AAAAAAAAO9E/CbjeihCRP1YHfP7fWMrJ2OcY-s9uIpVigCLcBGAs/s1600/cristian-baron-dPFaq7RVzbQ-unsplash.jpg)

Room persistence library is one of the easiest one to set up. However, when using data classes for your Room entities, you might face some small problem, which on the first look doesn't really make any sense.

Let's try to compile this class:

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey
    @ColumnInfo(name = "id")
    val number: Int,

    @ColumnInfo(name = "user_name")
    val userName: String,

    @ColumnInfo(name = "user_status")
    val userStatus: String,

    @Ignore
    val englishNameMeaning: UserGrade
)
```

**The problem:**

[![](https://1.bp.blogspot.com/-PPgod4Z5IjM/XVvPC7EQlaI/AAAAAAAAO84/VoCX0RocUjALqQP9_ulYwKZWrPZP2pSfgCLcBGAs/s1600/Screenshot_1.png)](https://1.bp.blogspot.com/-PPgod4Z5IjM/XVvPC7EQlaI/AAAAAAAAO84/VoCX0RocUjALqQP9_ulYwKZWrPZP2pSfgCLcBGAs/s1600/Screenshot_1.png)

And so will happen with other fields

But I already have a data class entity that has only vals.  Why doesn't this class compile?  
The problem here is the class UserGrade that room has no idea how give a value to it. Furthermore, that's a non nullable type. Even though I'm ignoring it from Room, the compiler cannot continue because it's the constructor that is checked first, thus failing all my other values to be set.

So, a small fix for this, is that make every variable a var instead of val and make our object nullable type. It's not a bad solution, but now I have to define a constructor, to tell Room that these are the default values from it:

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey
    @ColumnInfo(name = "id")
    var number: Int,

    @ColumnInfo(name = "user_name")
    var userName: String,

    @ColumnInfo(name = "user_status")
    var userStatus: String,

    @Ignore
    var englishNameMeaning: UserGrade?
){
  constructor(): this(number = 0, userName = "", userStatus = "", null)
}
```

**The solution**

Perhaps it is not such a big deal, but when chances are, why not try a better approach? Also, it might be a little problematic to break the standardization of the entities. Having a data class with vals and a data class with vars for the same purpose, is not a good style, at least for me.

Let's re roll to immutable variables and place a default value on our UserGrade object:

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey
    @ColumnInfo(name = "id")
    val number: Int,

    @ColumnInfo(name = "user_name")
    val userName: String,

    @ColumnInfo(name = "user_status")
    val userStatus: String,

    @Ignore
    val englishNameMeaning: UserGrade? = null
)
```

If you try to build, the same error will appear. The compiler will still yell at you. Even though the problem still persists from Room library, it's not its fault. That's because when Kotlin compiles to Java, it has no idea what default values in parameters are.

The last approach of this use case is to add the @JvmOverloads annotation before the constructor, so Java will know to create the constructor overloading for us:

```kotlin
@Entity(tableName = "users")
data class User @JvmOverloads constructor(
    @PrimaryKey
    @ColumnInfo(name = "id")
    val number: Int,

    @ColumnInfo(name = "user_name")
    val userName: String,

    @ColumnInfo(name = "user_status")
    val userStatus: String,

    @Ignore
    val englishNameMeaning: UserGrade? = null
)
```

And that's it. Good luck!
