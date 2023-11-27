---
title: "Searchable Fragments with the Paging Library"
datePublished: Mon Jan 13 2020 08:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clph5r8w9000a09lfac6f60gs
slug: searchable-fragments-with-the-paging-library

---


_This post is inspired by [@EpicPandaForce](https://stackoverflow.com/a/49193372/8914336) answer in StackOverflow. I faced the same problem which I didn't know how to solve: How to perform search when you are using a Paging Library (or how the hell to refresh after I reperform Rooms query)?_

Let's suppose we have this scenario: I have a list of data, which are shown in the Fragment by LiveData observation, which are retrieved by the ViewModel through `LiveDataPagedListBuilder()`. I'm hoping you know the basics of the [Paging Library](https://developer.android.com/topic/libraries/architecture/paging) already.

The data source:

I'm retrieving the data using a local database and Paging Library's `Datasource.Factory<Key, Value>`:

```kotlin
@Dao  
interface MyDao{  
    @RawQuery(observedEntities = [MyEntityRepresentation::class])  
    fun selectAllMeetingCondition(query: SupportSQLiteQuery): DataSource.Factory<Int , MyEntityRepresentation>:  
}
```

Room queries at Runtime.
------------------------

My use case requires to generate a dynamic query each time the user performs a search. In other words, the query depends on the users "advance" search. Since we know that Room generates queries at compile time, we would need something different. In this case we annotate our method with `@RawQuery` and must place `observedEntities` as dependency of that annotation. The methods parameter is jut a class where you can place your query string later.

{{< admonition >}}
The `DatasourceFactory` cannot be marked with suspend, the compilation would fail if you do so.
{{< /admonition >}}

So, the query should look something like this:

```kotlin "fixed=true"
fun instantiateSearch(  
        field1: String,  
        field2: String,  
        field3: String  
    ) {  
        viewModelScope.launch(appCoroutineDispatchers.ioDispatchers) {  
            this@AdvancedSearchViewModel.field1 = field1.toIntOrNull()  
            this@AdvancedSearchViewModel.field2 = field2.toIntOrNull()  
            this@AdvancedSearchViewModel.field3 = if (field3.isEmpty()) null else field3  
            var selectionQuery = "SELECT * FROM table_name"  
            this@AdvancedSearchViewModel.field1?.let {  
                selectionQuery += "AND field_3_name LIKE '%$it%' "  
            }  
            this@AdvancedSearchViewModel.field2?.let {  
                selectionQuery += "AND field_2_name = $it "  
            }  
            this@AdvancedSearchViewModel.field3?.let {  
                selectionQuery += "AND field_1_name = $it "  
            }  
            val finalSelectionQuery = selectionQuery.replaceFirst("AND", "WHERE")  
            _queryEvent.postValue(Event(finalSelectionQuery))  
        }  
    }
```

{{< admonition >}}
The search query is performed in a `BottomSheetFragment` so it's easy to send the query as an `Event` to the `PagingFragment`.
{{< /admonition >}}

I'm also not dealing with how to send an event through a `SharedViewModel`, but you can check [this](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150) link, or [this](https://www.coroutinedispatcher.com/2019/12/conditional-navigation-and-single-event.html) blog post of mine. But to get the picture, pretend my query is the ball below and players are Fragments or a ViewModel:

[![](https://thumbs.gfycat.com/MadeupOrganicJackal-size_restricted.gif)](https://thumbs.gfycat.com/MadeupOrganicJackal-size_restricted.gif)

After that all we have to do is pass the query. And here is where our problem with Paging Library starts. So let's say that we have a configuration like this:

```kotlin
class HomeViewModel @Inject constructor( //in real project is using @AssistedInject, no matter for this case  
    private val myDao: MyDao,  
) : ViewModel() {  
    var data: LiveData<PagedList<MyEntityRepresentation>>  
    init {  
        val listConfig = PagedList.Config.Builder()  
            .setPageSize(20)  
            .setEnablePlaceholders(false)  
            .build()  
        val dataSourceFactory = myDao.selectAll()  
        data = LivePagedListBuilder(dataSourceFactory, listConfig).build()  
    }  
  
    fun performSearch(query: String) {  
        val newData = myDao.selectAllMettingContition(query)  
        /* Won't work . LiveData already taken, unless you change the value*/  
        data = LivePagedListBuilder(newData, listConfig).build()   
    }  
}
```

This is not such a sophisticated solution because your user would end up seeing no changes in the Fragment

And now let's show the right thing to do it. First, the source should be only the one I describe in the `Dao`. Now let's refactor:

```kotlin
//inside ViewModel  
var data: LiveData<PagedList<MyEntityRepresentation>>  
private val listConfig = PagedList.Config.Builder()  
        .setPageSize(20)  
        .setEnablePlaceholders(false)  
        .build()  
private var finalSelectionQuery = "" //we need this
```

And now we can do something like this:

```kotlin
 init {  
        data = LivePagedListBuilder(  
            myDao.selectAllMeetingCondition(  
                SimpleSQLiteQuery("SELECT * FROM table_name $finalSelectionQuery ORDER BY name")  
            ),  
            listConfig  
        ).build()  
    }
```

Now, let's setup our `PagedFragment` so it can be updated depending on the query:

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {  
        super.onViewCreated(view, savedInstanceState)  
        initialiseComponents(view)  
        afterInitialize()  
        observeDataChanges()//note this  
    }
```

Inside the `observeDataChanges()` is just our data from ViewModel LiveData observation:

```kotlin
homeViewModel.cards.observe(this, Observer {  
            if (it.isEmpty()) {  
               //show empty result view  
            } else {  
                cardAdapter.submitList(it)  
            }  
        })
```

Why have we refactored this method? Because once the new String has been formed, the `PagedFragment` will get notified, will remove it's subscription, perform the query and re-evaluate data LiveData once again:

```kotlin
sharedViewModel.searchObjectLiveData.observe(viewLifecycleOwner, Observer {  
            it.getContentIfNotHandled()?.let { searchQuery ->  
                /*I admit that I have to find a better name but it's doing two things, removing the LiveDataObserver  
                and generating my new query. Best thing is to refactor to do one thing . Project still on going.*/  
                homeViewModel.resetAndPerformSearch(searchQuery, this)  
                observeDataChanges()//here we are again  
            }  
        })
```

Our last step is to write the `resetAndPerformSearch()` method:

```kotlin
fun resetAndPerformSearch(query: String, lifecycleOwner: LifecycleOwner) {  
        data.removeObservers(lifecycleOwner) //the fragment is not observing anymore  
        data = LivePagedListBuilder(  
            myDao.selectAllMeetingCondition(  
                SimpleSQLiteQuery(query)  
            ),  
            listConfig  
        ).build()  
    }
```

Now your search with Paging library will work just fine. Notice that after `resetAndPerformSearch()` we are calling observeDataChanges so that our Fragment will be ready to react after the new query has been performed.

Conclusion.
-----------

Hopefully, this solution would help you to perform an painless search when having Paging library around. Otherwise you would end up with a bunch of flags and a bunch of other configurations instead.

Full repository can be found [here](https://github.com/coroutineDispatcher/your_move). Please excuse typos, poor design or other mistakes because the project is still on going and has a lot of redundant code and files.

Stavro Xhardha
