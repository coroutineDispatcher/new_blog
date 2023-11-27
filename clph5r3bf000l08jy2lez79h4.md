---
title: "Room, the basics"
datePublished: Wed Jun 19 2019 08:46:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5r3bf000l08jy2lez79h4
slug: room-the-basics

---


It has been a while since Roompersistence library is out. It was about time, the SQLiteimplementation was awful, long work and sometimes…

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701104588906/80c123ff-5ca7-437c-b08d-d896cad4fc26.jpeg)

It has been a while since `Room` persistence library is out. It was about time, the `SQLite` implementation was awful, long work and sometimes confusing.  
Therefore, the Android team built Room:

_The_ [_Room_](https://developer.android.com/training/data-storage/room/index.html) _persistence library provides an abstraction layer over SQLite to allow for more robust database access while harnessing the full power of SQLite._

According to the above statement from the [official documentation](https://developer.android.com/topic/libraries/architecture/room), `Room` is nothing more than a “refactored and improved” SQLite library.

**The setup:**

```groovy
implementation "androidx.room:room-runtime:2.1.0-beta01"
kapt "androidx.room:room-compiler:2.1.0-beta01"
implementation "androidx.room:room-ktx:2.1.0-beta01"
```

The third dependency is not necessary for room but I am using [Room with Coroutines](https://medium.com/androiddevelopers/room-coroutines-422b786dc4c5)

**The Entity:**

After you have imported the dependencies, you may start constructing your database. What your database needs at first is a table (or on `Room` language the Entity):

```kotlin
@Entity(tableName = "names")
data class Name(
    @PrimaryKey
    @ColumnInfo(name = "id")
    var number: Int,

    @ColumnInfo(name = "arabic_name")
    var arabicName: String,

    @ColumnInfo(name = "transliteration")
    var transliteration: String,

    @Ignore
    val englishNameMeaning: EnglishNameMeaning?,

    @ColumnInfo(name = "name_meaning")
    var meaning: String
){
    public constructor(): this("" , "", 0, null, "")
}
```

This is a table in terms of `Room` and a model class in terms of `Kotlin` . It is a representation of your data. To tell `Room` that I have a table to define I need to provide the `@Entity` annotation with the required parameter. Every table in `Room` should have a `Primary Key` . You can use also the `AutoIncrement` but I don’t really need it for my case.With the `@ColumnInfo` I am telling room what name will this field have. Notice the `@Ignore` annotation. This is a special annotation that tells `Room` not to care about that field. Since I am using the same `Name` model for my network request and also for `Room` I need that field for the network but not for the Database.

**The Dao:**

The Dao is nothing more than a way to define your queries based on the Entity you declared. Basically, every Entity must have a Dao in order to deal with the table data. Leave the rest to the `Room`

```kotlin
@Dao
interface NamesDao {

    @Query("SELECT * FROM names")
    suspend fun selectAllNames(): List<Name>

    @Insert
    suspend fun insertName(name: Name)

    @Query("SELECT * FROM names where id =:number")
    suspend fun findName(number: Int): Name
}
```

Dao should be an `interface` or an `abstract class` . Just annotate it with the `@Dao` annotation. I believe there is no need to say what `@Query` or `@Insert` annotations do. One thing I must notice here is that, the table name **must** be the same as your `tableName` defined in your entity. However don’t worry, the IDE will provide it for you as soon as you start typing it. The other thing I have to mention here is passing method parameters in the query. It can be done as the example states by adding `:` before the query parameter:  
`SELECT * FROM names where id = :number` .

**Database Declaration:**

```kotlin
@Database(entities = [Name::class], version = 1, exportSchema = false)
abstract class MyDatabase : RoomDatabase() {
    abstract fun namesDao(): NamesDao
}
```

This is where your database is declared, your entities are defined and also the version and the schema. If you want to keep your database version for testing purposes, or you need other things to check, set the `exportSchema` to true.

**Instatiation:**

```kotlin
    @Provides
    @ApplicationScope
    fun provideRoomDatabase(context: Application): MyDatabase = Room.databaseBuilder(
        context,
        MyDatabase::class.java, MY_DATABASE_NAME
    ).fallbackToDestructiveMigration().build()
```

Remember, Room instance is a `Singleton` and needs to declared only one time for application. With the help of Dagger2 I instantiate it like this:

And after that I only require the Dao according to the module dependencies:

```kotlin
@Provides
    @FragmentScope
    fun provideNamesDao(myDatabase: MyDatabase): NamesDao = myDatabase.namesDao()

```

Notice the `fallbackToDestructiveMigration()` method. This one tells `Room` to delete my data when I migrate to a new database version. It is optional, you can remove it if you want.

**Testing:**

I am pretty new at testing myself, but testing in room is not that difficult. First, go to the `androidtest` package and create a new class:

```kotlin
@RunWith(AndroidJUnit4::class)
class NameReadWriteTest
```

After that you need a `Fake` for `RoomDatabase` . This basically runs the queries in memory without caching data to a real database:

```kotlin
   private lateinit var namesDao: NamesDao
    private lateinit var myDatabase: MyDatabase

    @Before
    fun createDatabase() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        myDatabase = Room.inMemoryDatabaseBuilder(context, MyDatabase::class.java).build()
        namesDao = myDatabase.namesDao()
    }

    @After
    @Throws(IOException::class)
    fun closeDatabase() {
        myDatabase.close()
    }
```

And finally:

```kotlin
    @Test
    @Throws(Exception::class)
    fun writeNameAndReadInList() = runBlocking {
        //Arrange
        val name = Name("ArabicLetters", "ArabicTransliteration", 1, null, "It's just a use case")

        //Act
        namesDao.insertName(name)
        val insertedNames = namesDao.selectAllNames()

        //Assert
        assertEquals(listOf(name), insertedNames)
    }
```

Remember, I am using the `runBlocking` only because I am using `Room` with coroutines. Done!

**Conclusion:**

After I learned how to use `Room` , I have to say that using a database in Android jumped from being my stress to my hobby.

What helped me:

[**Room 🔗 Coroutines**  
\_Add some suspense to your database_medium.com](https://medium.com/androiddevelopers/room-coroutines-422b786dc4c5 "https://medium.com/androiddevelopers/room-coroutines-422b786dc4c5")[](https://medium.com/androiddevelopers/room-coroutines-422b786dc4c5)

[**Testing your database | Android Developers**  
\_Learn to test databases created using the Room Library_developer.android.com](https://developer.android.com/training/data-storage/room/testing-db "https://developer.android.com/training/data-storage/room/testing-db")[](https://developer.android.com/training/data-storage/room/testing-db)

[**Save data in a local database using Room | Android Developers**  
\_Learn to persist data more easily using the Room Library_developer.android.com](https://developer.android.com/training/data-storage/room/index.html "https://developer.android.com/training/data-storage/room/index.html")[](https://developer.android.com/training/data-storage/room/index.html)

If you like my post please check other stories on my profile:

[**Stavro Xhardha - Medium**  
\_Read writing from Stavro Xhardha on Medium. Android Dev . Kotlin Lover. Every day, Stavro Xhardha and thousands of…\_medium.com](https://medium.com/@coroutinedispatcher "https://medium.com/@coroutinedispatcher")[](https://medium.com/@coroutinedispatcher)

By [Stavro Xhardha](https://medium.com/@coroutinedispatcher) on [May 22, 2019](https://medium.com/p/da6b60e14601).

[Canonical link](https://medium.com/@coroutinedispatcher/room-the-basics-da6b60e14601)

Exported from [Medium](https://medium.com) on June 19, 2019.

Post settings Labels Published on 6/19/19, 1:46 AM Pacific Daylight Time Permalink Location Options
