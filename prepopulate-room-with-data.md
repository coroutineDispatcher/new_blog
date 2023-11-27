---
title: 'Prepopulate Room with data.'
date: 2020-01-06T09:04:00.000+01:00
draft: false
aliases: [ "/2020/01/prepopulate-room-with-data.html" ]
tags : [Java, PrePopulate Room, Android Room, Kotlin, Android, Room Persistence, Sqlite]
author: "Stavro Xhardha"
---

[![](https://s1.zerochan.net/Yu-Gi-Oh!.Duel.Monsters.600.605081.jpg)](https://s1.zerochan.net/Yu-Gi-Oh!.Duel.Monsters.600.605081.jpg)

  
There are times, when we just need the data when the app starts, and all the functionality is just a matter of work. Or we just need the app to be independent from the network and we have the data. A simple dog-race database or cat-race database doesn't actually need online interaction at all (if there are not too many data of course). So, Room comes with a nice solution about this. The [docs](https://developer.android.com/training/data-storage/room/prepopulate) are pretty clear and short when it comes to this topic.  
We just write:  
  
```kotlin
Room.databaseBuilder(appContext, AppDatabase.class, "Sample.db")  
    .createFromAsset("database/myapp.db")  
    .build()  

```

And our data are ready to be instantiated when the app starts. One more thing to note is that its extremely fast to do it this way. However, here are some problems that the docs don't even bother to mention (which I think are important for some).  

1 - A .db file, is not a .sql file.
-----------------------------------

The file we are supposed to hold in assets folder which contains a query like: CREATE TABLE IF NOT EXISTS  has nothing to do with the file in the code above, that we are supposed to import. We need to create a database like this. I actually used [SqliteBrowser](https://sqlitebrowser.org/), to execute the query, but there should be other ways too.  

2 - Null values and naming
--------------------------

Our data model needs to be precisely as the data we are supposed to prepopulate. For example if we have a \[name\] TEXT NULL we should make sure that the representation of it must meet the same conditions, including nullability and naming (this also includes table names):  

```kotlin
@ColumnInfo(name = "name")  
val name: String? = "",  
```

Otherwise, we would feel some serious trouble. If we have the correct set up, let Room to connect the rest (the Sqlite table and it's Kotlin/Java object representation).  

3- Must have incremental processor enabled:
-------------------------------------------

Don't forget to have the correct setup in build.gradle (or in my case build.gradle.kts), because this feature is only available after Room 2.2.0: 

```kotlin
 javaCompileOptions {  
            annotationProcessorOptions {  
                arguments = mapOf(  
                    "room.schemaLocation" to "$projectDir/schemas",  
                    "room.incremental" to "true",  
                    "room.expandProjection" to "true"  
                )  
            }  
        }
```

How to know if we are doing it correctly?
-----------------------------------------

There are two cases to spot here:  

1 - We have the incorrect file
------------------------------

In this case Room would automatically throw a RuntimeException yelling: "Cannot copy database file".  

2 - The file is correct but the table in SQL has nothing to do with our model representation
--------------------------------------------------------------------------------------------

In this case Room would throw a SqliteException with some error like: "Database file is corrupt". Trust me whatever you google on this case, cannot help 😅.  
  
{{< admonition >}}
This article is just about importing a .db file from assets folder. You can also import a db file from the device after download. Please check the docs for more.  
{{< /admonition >}}

A wish I made.
--------------

Well, I would like to see an option from Room to import a JSON as a file. Should be cool to have it as a feature.  
  

Conclusion:
-----------

I hope I helped anyone who is willing to write an app with totally offline data. It's pretty good solution, when we don't have too much.  
  
Stavro Xhardha