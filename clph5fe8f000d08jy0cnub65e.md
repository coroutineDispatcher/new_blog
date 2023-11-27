---
title: "A fusion between WorkManager and AlarmManager"
datePublished: Thu Jul 04 2019 14:50:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5fe8f000d08jy0cnub65e
slug: a-fusion-between-workmanager-and-alarmmanager

---


Androids' WorkManager has been around for a while. However my own expectations about it were a little higher. I wished that the WorkManagercould fire events at exact timing. But since it was made to [respect doze mode](https://issuetracker.google.com/issues/110312013), I should respect that too. That means that if the phone is idle, the WorkManager wont run.

**Lets explore my case: **  
Once the user logs into my app, I need to schedule 5 alarms for him. But the alarm times come from a server and not locally. So basically I need some background work to do:   
Sync timing -> Save to database -> Schedule Alarms. (btw on the time alarm will fire, it's just a notification to be shown)  
Now this has to happen every day at 00:01. The data coming from the server hold the alarm timings for a year, but certainly I can't schedule alarms from current moment till the end of the year. It would drain phones battery. So I need to start a WorkManagerto save data to my local Room database and after that I need to repeatedly  schedule alarms every day.

**On with the show: **  
First of all, if anyone is curious why I chose WorkManagerover JobScheduleror even BroadcastReceiver:

1- Don't have to register nothing to the Manifest file.  
2- Don't have to worry about internet connection, phone reboot etc.  
3- I faced lots of deprecated methods and constants without it.

Let's start:

```kotlin
//starting it in the HomeViewModel's instantiation, handled with a sharedpreferences flag so it can't be refired
val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()
    val compressionWork = OneTimeWorkRequestBuilder<DataSyncWorker>()
        .setConstraints(constraints)
        .build()

    WorkManager.getInstance().enqueue(compressionWork)
```

So the WorkManager has a few things to set up. Since I will get data from API, I need my worker to be started **if and only if** there is internet connection which is specified by the setRequiredNetworkType(NetworkType.CONNECTED) method in the Constraint.Builder(). After that I need to specify how many times will my worker run. On my case there is no comment needed looking at the name OneTimeWorkRequestBuilderwhich has a worker of mine. After setting my defined constraints I can start my DataSyncWorker.

Let's check the class now:

```kotlin
class DataSyncWorker(val context: Context, parameters: WorkerParameters) : CoroutineWorker(context, parameters) {

    private lateinit var api: MyApi
    private lateinit var dao: MyDao
    private lateinit var offlineScheduler: OfflineScheduler

    override suspend fun doWork(): Result = coroutineScope {
        instantiateDependencies() //lateinits are instantiated here
        launch {
            val dataToReceive = api.getAllData()
            if (dataToReceive.isSuccessful) {
                //save data to database
                //init OfflineScheduler object
                offlineScheduler.initScheduler()
                Result.success()
            } else {
                Result.retry()
            }
        }
        Result.success()
    }
}
```

Since I love coroutines and I am using them everywhere, I am using the WorkManagerby extending CoroutineWorker. So the method where all my sync happens is named doWork which btw is a suspending method handled by the Android framework itself. By the way is hell lot of data.

And now, lets jump into my OfflineScheduler:

```kotlin
class OfflineScheduler @Inject constructor(
    val sharedPrefs: MSharedPref,
    val context: Application,
    val dao: MyDao
) {

    suspend fun initScheduler() {
        val currentDaySelection = dao.seletAllSavedData()
        if (currentDaySelection != null) {
            scheduleNotificationTimes(currentDaySelection)
            invokeTomorrowAlarm()
        } else {
           // it's january 1st of next year or I have screwed up everything
        }
    }
 }
```

Looks like I'm done scheduling my notifications at current timing for today and also tomorrows timing alarm, which will schedule other alarms:

```kotlin
// bunch of intent and pending intent set up

val alarmManager = mContext.getSystemService(Context.ALARM_SERVICE) as AlarmManager

alarmManager.setExact(AlarmManager.RTC, myTimeInstance.millis, pendingIntent)
```

Once the alarm has been set, Android "waits" until the time comes and brings my code here:

```kotlin
class NotificationReceiver : BroadcastReceiver() {

    private val CHANNEL_ID = "daily_notification"

    override fun onReceive(context: Context?, intent: Intent?) {
        showNotifications(context,intent)
    }
  }
```

And another one which was scheduled for the midnight, which basically will trigger my OfflineScheduler code again:

```kotlin
class MidnightScheduler : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {
        //it's midnight
        val offlineScheduler = SingletonInstance.getOfflineScheduler()
        GlobalScope.launch(Dispatchers.IO) {
            offlineScheduler.initScheduler()
        }
    }
}
```

And it will fire and fire until the end of time 💪

There are 2 more things:

First of all, for the classes extending BroadCastReceiver to work, need to be registered to Manifest file.  
And second of all how do I schedule my notifications whem the phone reboots? 👀

So I made another Receiver which only needs to be registered on the Manifest and nothing more:

```kotlin
class AlarmRebootReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context?, intent: Intent?) {
        if (intent?.action.equals("android.intent.action.BOOT_COMPLETED")) {
        //phone has rebooted
        val offlineScheduler = SingletonInstance.getOfflineScheduler()
        GlobalScope.launch(Dispatchers.IO) {
            if(workerHasalreadyBeenFiredOnce())
                offlineScheduler.initScheduler()
        }
    }
  }
}
```

Now this one needs a small modification achieved by a boolean flag. What happens if my worker has never started and I request to schedule local data (which never arrived)? A total disaster.

And pretty much that's it. That's the best combination I could do to solve my case, and if you have other suggestions about my case, feel free to comment.
