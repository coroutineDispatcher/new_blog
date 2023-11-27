---
title: "Room and coroutines testing"
datePublished: Mon Oct 14 2019 08:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5qw0y000909lfhntq7jgk
slug: room-and-coroutines-testing

---


My last article covered some simple example about Room and RxJava instrumentation testing code. Coroutines also have [great support](https://github.com/Kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-test) in _unit testing_ even though todays topic has nothing to do with it.  What I mean is that we are not going to cover runBlockingTest this time. Android doesn't support that (correct me if I'm wrong please) yet. However, I could schedule a topic about that because it really makes me excited. You can also check [this awesome talk](https://www.droidcon.com/media-detail?video=352671106) from Sean McQuillan, which I found pretty helpful.

Comparing to the last gists, we will try to jump from Rx to coroutines (without implementation) and write test class about it.  Instead of RxJava components, we can just mark methods as `suspend`ed:

```kotlin
@Dao
interface HabitDao {
    @Insert
    suspend fun insertHabit(habit: Habit) //used to return Completable

    @Query("SELECT * FROM user_habits WHERE end_date >= :todaysDate")
    suspend fun selectAllActiveHabits(todaysDate: Date): List<Habit> //used to return Flowable<List<Habit>>

    @Query("SELECT * FROM user_habits WHERE end_date < :todaysDate")
    suspend fun selectAllInactiveHabits(todaysDate: Date): List<Habit> //used to return Flowable<List<Habit>>

    @Query("DELETE FROM user_habits WHERE id = :id")
    suspend fun deleteHabit(id: Long) //used to return Completable

    @Query("UPDATE user_habits SET start_date = :startDate, end_date = :endDate , receive_notification = :notification WHERE id = :id")
    suspend fun updateWhereId(startDate: Date, endDate: Date, notification: Boolean, id: Long) //used to return Completable

    @VisibleForTesting
    @Query("SELECT * FROM user_habits")
    suspend fun selectAll(): List<Habit> //used to return Flowable<List<Habit>>
}
```

This looks easier (or better say less confusing).

{{< admonition >}}
Looking the code in the previous article, notice that you won't be needing the InstantTaskExecutorRule() and **suddenly** we won't be running the queries on the Main thread. That's because we can't do that if methods are marked as `suspend`ed.  
{{</ admonition >}}

Let's start testing:

```kotlin
@RunWith(AndroidJUnit4::class)
class HabitDaoTest{
    private lateinit var careFlectDatabase: CareFlectDatabase //the db instance
    private lateinit var habitDao: HabitDao //the dao

    @Before
    fun setUp() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        careFlectDatabase = Room.inMemoryDatabaseBuilder(context, CareFlectDatabase::class.java)
            .build()

        habitDao = careFlectDatabase.habitDao()
    }

    @After
    fun tearDown() {
        careFlectDatabase.close()
    }
}
```

Notice that we dropped the allowMainThreadQueries().

And now the queries:

```kotlin
 @Test
    fun presentDateShouldReturnInsertion() = runBlocking {
        val today = getInstance().apply {
            set(MONTH, 10)
            set(YEAR, 2019)
            set(DAY_OF_MONTH, 1)
        }

        val yesterday = getInstance().apply {
            set(MONTH, 9)
            set(YEAR, 2019)
            set(DAY_OF_MONTH, 30)
        }

        val habit = Habit(0, "Quit Smoking", yesterday.time, today.time, true)

        habitDao.insertHabit(habit)

        val selection = habitDao.selectAll()

        assertEquals(listOf(habit), selection)
    }
```

Something too familiar? The key ingredient here is just a runBlocking keyword, which makes sure to run your suspending methods. I guess there is no need to add code for the update or delete part of testing. It's just super imperative and there is no secret here which could make you lose your mind (referring to the Rx-Javas blockingawait()) because a coroutine makes sure that the insertion is executed before the code below.

**Not a small comparison:**

I love both. But I think that Rx is really redundant when we are inside the Kotlin (especially coroutines) context.

**Conclusion**

Coroutines are being every day more supported by Google Android team and that's really great. New libraries are being written in Kotlin and coroutines are part of them. What I really like about Room and coroutines in testing is that I never _deliberately_ forget to test Daos on my project, I'm always ready to write tests (even though I haven't been around testing in more than 8 months).

Note: If you want to know more about Room + coroutines, [here](https://medium.com/androiddevelopers/room-coroutines-422b786dc4c5) is a nice article from Florina Muntenescu also covering some deep dive and behind the scenes on how Room Coroutine support has been build.

Stavro Xhardha
