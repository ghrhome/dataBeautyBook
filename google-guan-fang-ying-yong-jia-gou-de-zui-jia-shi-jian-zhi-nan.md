## Google 官方应用架构的最佳实践指南

> **导语：**
>
> 虽然说 Android 的架构选择一直都很自由，MVP、MVC、MVVM 各有拥趸。但 Google 最近还是推出了一份关于应用架构的实践指南，并给出了相当详尽的步骤和一些指导建议。希望大家都能看一看，学习一下，打造更加优秀易用的 APP，也为 Android 生态的改善做一点贡献。: \)

最近，官方推出了一份关于应用架构的最佳实践指南。这里就给大家简要介绍一下：

首先，Android 开发者肯定都知道 Android 中有四大组件，这些组件都有各自的生命周期并且在一定程度上是不受你控制的。在任何时候，Android 操作系统都可能根据用户的行为或资源紧张等原因回收掉这些组件。

这也就引出了**第一条准则**：「不要在应用程序组件中保存任何应用数据或状态，并且组件间也不应该相互依赖」。

最常见的错误就是在 Activity 或 Fragment 中写了与 UI 和交互无关的代码。尽可能减少对它们的依赖，这能避免大量生命周期导致的问题，以提供更好的用户体验。

**第二条准则：**「通过 model 驱动应用 UI，并尽可能的持久化」。

这样做主要有两个原因：

1. 如果系统回收了你的应用资源或其他什么意外情况，不会导致用户丢失数据。
2. Model 就应该是负责处理应用程序数据的组件。独立于视图和应用程序组件，保持了视图代码的简单，也让你的应用逻辑更容易管理。并且，将应用数据置于 model 类中，也更有利于测试。

### 官方推荐的 App 架构

在这里，官方演示了通过使用最新推出的[Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)来构建一个应用。

想象一下，您正在打算开发一个显示用户个人信息的界面，用户数据通过 REST API 从后端获取。

首先，我们需要创建三个文件：

* user\_profile.xml：定义界面。
* UserProfileViewModel.java：数据类。
* UserProfileFragment.java：显示 ViewModel 中的数据并对用户的交互做出反应。

为了简单起见，我们这里就省略掉布局文件。

```
public class UserProfileViewModel extends ViewModel {
    private String userId;
    private User user;

    public void init(String userId) {
        this.userId = userId;
    }
    public User getUser() {
        return user;
    }
}
```

```
public class UserProfileFragment extends LifecycleFragment {
    private static final String UID_KEY = "uid";
    private UserProfileViewModel viewModel;

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        String userId = getArguments().getString(UID_KEY);
        viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class);
        viewModel.init(userId);
    }

    @Override
    public View onCreateView(LayoutInflater inflater,
                @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.user_profile, container, false);
    }
}
```

注意其中的 ViewModel 和 LifecycleFragment 都是 Android 新引入的，可以参考[官方说明](https://developer.android.com/topic/libraries/architecture/adding-components.html)进行集成。

现在，我们完成了这三个模块，该如何将它们联系起来呢？也就是当 ViewModel 中的用户字段被设置时，我们需要一种方法来通知 UI。这就是 LiveData 的用武之地了。

> [LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html)是一个可被观察的数据持有者（用到了观察者模式）。其能够允许 Activity, Fragment 等应用程序组件对其进行观察，并且不会在它们之间创建强依赖。LiveData 还能够自动响应各组件的声明周期事件，防止内存泄漏，从而使应用程序不会消耗更多的内存。
>
> **注意：**LiveData 和 RxJava 或 Agera 的区别主要在于 LiveData 自动帮助处理了生命周期事件，避免了内存泄漏。

所以，现在我们来修改一下 UserProfileViewModel：

```
public class UserProfileViewModel extends ViewModel {
    ...
    private LiveData<User> user;
    public LiveData<User> getUser() {
        return user;
    }
}
```

再在 UserProfileFragment 中对其进行观察并更新我们的 UI：

```
@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    viewModel.getUser().observe(this, user -> {
      // update UI
    });
}
```

**获取数据**

现在，我们联系了 ViewModel 和 Fragment，但 ViewModel 又怎么来获取到数据呢？

在这个示例中，我们假定后端提供了 REST API，因此我们选用[Retrofit](http://square.github.io/retrofit/)来访问我们的后端。

首先，定义一个 Webservice：

```

public interface Webservice {
    /**
     * @GET declares an HTTP GET request
     * @Path("user") annotation on the userId parameter marks it as a
     * replacement for the {user} placeholder in the @GET path
     */
    @GET("/users/{user}")
    Call<User> getUser(@Path("user") String userId);
}
```

不要通过 ViewModel 直接来获取数据，这里我们将工作转交给一个新的 Repository 模块。

> Repository 模块负责数据处理，为应用的其他部分提供干净可靠的 API。你可以将其考虑为不同数据源（Web，缓存或数据库）与应用之间的中间层。

```
public class UserRepository {
    private Webservice webservice;
    // ...
    public LiveData<User> getUser(int userId) {
        // This is not an optimal implementation, we'll fix it below
        final MutableLiveData<User> data = new MutableLiveData<>();
        webservice.getUser(userId).enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                // error case is left out for brevity
                data.setValue(response.body());
            }
        });
        return data;
    }
}

```

**管理组件间的依赖关系**

根据上面的代码，我们可以看到 UserRepository 中有一个 Webservice 的实例，不要直接在 UserRepository 中 new 一个 Webservice。这很容易导致代码的重复与复杂化，比如 UserRepository 很可能不是唯一用到 Webservice 的类，如果每个用到的类都新建一个 Webservice，这显示会导致资源的浪费。

这里，我们推荐使用[Dagger 2](https://google.github.io/dagger/)来管理这些依赖关系。

现在，让我们来把 ViewModel 和 Repository 连接起来吧：

```
public class UserProfileViewModel extends ViewModel {
    private LiveData<User> user;
    private UserRepository userRepo;

    @Inject // UserRepository parameter is provided by Dagger 2
    public UserProfileViewModel(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    public void init(String userId) {
        if (this.user != null) {
            // ViewModel is created per Fragment so
            // we know the userId won't change
            return;
        }
        user = userRepo.getUser(userId);
    }

    public LiveData<User> getUser() {
        return this.user;
    }
}
```

**缓存数据**

在实际项目中，Repository 往往不会只有一个数据源。因此，我们这里在其中再加入缓存：

```
@Singleton  // informs Dagger that this class should be constructed once
public class UserRepository {
    private Webservice webservice;
    // simple in memory cache, details omitted for brevity
    private UserCache userCache;
    public LiveData<User> getUser(String userId) {
        LiveData<User> cached = userCache.get(userId);
        if (cached != null) {
            return cached;
        }

        final MutableLiveData<User> data = new MutableLiveData<>();
        userCache.put(userId, data);
        // this is still suboptimal but better than before.
        // a complete implementation must also handle the error cases.
        webservice.getUser(userId).enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                data.setValue(response.body());
            }
        });
        return data;
    }
}
```

**持久化数据**

现在当用户旋转屏幕或暂时离开应用再回来时，数据是直接可见的，因为是直接从缓存中获取的数据。但要是用户长时间关闭应用，并且 Android 还彻底杀死了进程呢？

我们目前的实现中，会再次从网络中获取数据。这可不是一个好的用户体验。这时就需要数据持久化了。继续引入一个新组件[Room](https://developer.android.com/topic/libraries/architecture/room.html)。

> Room 能帮助我们方便的实现本地数据持久化，抽象出了很多常用的数据库操作，并且在编译时会验证每个查询，从而损坏的 SQL 查询只会导致编译时错误，而不是运行时崩溃。还能和上面介绍的 LiveData 完美合作，并帮开发者处理了很多线程问题。

现在，让我们来看看怎么使用 Room 吧。: \)

首先，在 User 类上面加上 @Entity，将 User 声明为你数据库中的一张表。

```

@Entity
class User {
  @PrimaryKey
  private int id;
  private String name;
  private String lastName;
  // getters and setters for fields
}
```

再创建数据库类并继承 RoomDatabase：

```
@Database(entities = {User.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {
}
```

注意 MyDatabase 是一个抽象类，Room 会自动添加实现的。

现在我们需要一种方法来将用户数据插入到数据库：

```
@Dao
public interface UserDao {
    @Insert(onConflict = REPLACE)
    void save(User user);
    @Query("SELECT * FROM user WHERE id = :userId")
    LiveData<User> load(String userId);
}
```

再在数据库类中加入 DAO：

```
@Database(entities = {User.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

注意上面的 load 方法返回的是 LiveData，Room 会知道什么时候数据库发生了变化并自动通知所有的观察者。这也就是 LiveData 和 Room 搭配的妙用。

现在继续修改 UserRepository：

```
@Singleton
public class UserRepository {
    private final Webservice webservice;
    private final UserDao userDao;
    private final Executor executor;

    @Inject
    public UserRepository(Webservice webservice, UserDao userDao, Executor executor) {
        this.webservice = webservice;
        this.userDao = userDao;
        this.executor = executor;
    }

    public LiveData<User> getUser(String userId) {
        refreshUser(userId);
        // return a LiveData directly from the database.
        return userDao.load(userId);
    }

    private void refreshUser(final String userId) {
        executor.execute(() -> {
            // running in a background thread
            // check if user was fetched recently
            boolean userExists = userDao.hasUser(FRESH_TIMEOUT);
            if (!userExists) {
                // refresh the data
                Response response = webservice.getUser(userId).execute();
                // TODO check for error etc.
                // Update the database.The LiveData will automatically refresh so
                // we don't need to do anything else here besides updating the database
                userDao.save(response.body());
            }
        });
    }
}
```

可以看到，即使我们更改了 UserRepository 中的数据源，我们也完全不需要修改 ViewModel 和 Fragment，这就是抽象的好处。同时还非常适合测试，我们可以在测试 UserProfileViewModel 时提供测试用的 UserRepository。

> 下面部分的内容在原文中是作为附录，但我个人觉得也很重要，所以擅自挪上来，一起为大家介绍了。: \)

在上面的例子中，有心的大家可能发现了我们没有处理网络错误和正在加载状态。但在实际开发中其实是很重要的。这里，我们就实现一个工具类来根据不同的网络状况选择不同的数据源。

首先，实现一个 Resource 类：

```
//a generic class that describes a data with a status
public class Resource<T> {
    @NonNull public final Status status;
    @Nullable public final T data;
    @Nullable public final String message;
    private Resource(@NonNull Status status, @Nullable T data, @Nullable String message) {
        this.status = status;
        this.data = data;
        this.message = message;
    }

    public static <T> Resource<T> success(@NonNull T data) {
        return new Resource<>(SUCCESS, data, null);
    }

    public static <T> Resource<T> error(String msg, @Nullable T data) {
        return new Resource<>(ERROR, data, msg);
    }

    public static <T> Resource<T> loading(@Nullable T data) {
        return new Resource<>(LOADING, data, null);
    }
}
```

因为，从网络加载数据和从磁盘加载是很相似的，所以再新建一个 NetworkBoundResource 类，方便多处复用。下面是 NetworkBoundResource 的决策树：

![](/assets/20170523100721373)

API 设计：

```
// ResultType: Type for the Resource data
// RequestType: Type for the API response
public abstract class NetworkBoundResource<ResultType, RequestType> {
    // Called to save the result of the API response into the database
    @WorkerThread
    protected abstract void saveCallResult(@NonNull RequestType item);

    // Called with the data in the database to decide whether it should be
    // fetched from the network.
    @MainThread
    protected abstract boolean shouldFetch(@Nullable ResultType data);

    // Called to get the cached data from the database
    @NonNull @MainThread
    protected abstract LiveData<ResultType> loadFromDb();

    // Called to create the API call.
    @NonNull @MainThread
    protected abstract LiveData<ApiResponse<RequestType>> createCall();

    // Called when the fetch fails. The child class may want to reset components
    // like rate limiter.
    @MainThread
    protected void onFetchFailed() {
    }

    // returns a LiveData that represents the resource
    public final LiveData<Resource<ResultType>> getAsLiveData() {
        return result;
    }
}

```

注意上面使用了 ApiResponse 作为网络请求， ApiResponse 是对于 Retrofit2.Call 的简单包装，用于将其响应转换为 LiveData。

下面是具体的实现：

```

public abstract class NetworkBoundResource<ResultType, RequestType> {
    private final MediatorLiveData<Resource<ResultType>> result = new MediatorLiveData<>();

    @MainThread
    NetworkBoundResource() {
        result.setValue(Resource.loading(null));
        LiveData<ResultType> dbSource = loadFromDb();
        result.addSource(dbSource, data -> {
            result.removeSource(dbSource);
            if (shouldFetch(data)) {
                fetchFromNetwork(dbSource);
            } else {
                result.addSource(dbSource,
                        newData -> result.setValue(Resource.success(newData)));
            }
        });
    }

    private void fetchFromNetwork(final LiveData<ResultType> dbSource) {
        LiveData<ApiResponse<RequestType>> apiResponse = createCall();
        // we re-attach dbSource as a new source,
        // it will dispatch its latest value quickly
        result.addSource(dbSource,
                newData -> result.setValue(Resource.loading(newData)));
        result.addSource(apiResponse, response -> {
            result.removeSource(apiResponse);
            result.removeSource(dbSource);
            //noinspection ConstantConditions
            if (response.isSuccessful()) {
                saveResultAndReInit(response);
            } else {
                onFetchFailed();
                result.addSource(dbSource,
                        newData -> result.setValue(
                                Resource.error(response.errorMessage, newData)));
            }
        });
    }

    @MainThread
    private void saveResultAndReInit(ApiResponse<RequestType> response) {
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... voids) {
                saveCallResult(response.body);
                return null;
            }

            @Override
            protected void onPostExecute(Void aVoid) {
                // we specially request a new live data,
                // otherwise we will get immediately last cached value,
                // which may not be updated with latest results received from network.
                result.addSource(loadFromDb(),
                        newData -> result.setValue(Resource.success(newData)));
            }
        }.execute();
    }
}
```

现在，我们就能使用 NetworkBoundResource 来根据不同的情况获取数据了：

```
class UserRepository {
    Webservice webservice;
    UserDao userDao;

    public LiveData<Resource<User>> loadUser(final String userId) {
        return new NetworkBoundResource<User,User>() {
            @Override
            protected void saveCallResult(@NonNull User item) {
                userDao.insert(item);
            }

            @Override
            protected boolean shouldFetch(@Nullable User data) {
                return rateLimiter.canFetch(userId) && (data == null || !isFresh(data));
            }

            @NonNull @Override
            protected LiveData<User> loadFromDb() {
                return userDao.load(userId);
            }

            @NonNull @Override
            protected LiveData<ApiResponse<User>> createCall() {
                return webservice.getUser(userId);
            }
        }.getAsLiveData();
    }
}
```

到这里，我们的代码就全部完成了。最后的架构看起来就像这样：

![](/assets/20170523100426769)



### 最后的最后，给出一些指导原则

下面的原则虽然不是强制性的，但根据我们的经验遵循它们能使您的代码更健壮、可测试和可维护的。

* 所有您在 manifest 中定义的组件 - activity, service, broadcast receiver… 都不是数据源。因为每个组件的生命周期都相当短，并取决于当前用户与设备的交互和系统的运行状况。简单来说，这些组件都不应当作为应用的数据源。
* 在您应用的各个模块之间建立明确的责任边界。比如，不要将与数据缓存无关的代码放在同一个类中。
* 每个模块尽可能少的暴露内部实现。从过去的经验来看，千万不要为了一时的方便而直接将大量的内部实现暴露出去。这会让你在以后承担很重的技术债务（很难更换新技术）。
* 在您定义模块间交互时，请考虑如何使每个模块尽量隔离，通过设计良好的 API 来进行交互。
* 您应用的核心应该是能让它脱颖而出的某些东西。不要浪费时间重复造轮子或一次次编写同样的模板代码。相反，应当集中精力在使您的应用独一无二，而将一些重复的工作交给这里介绍的 Android Architecture Components 或其他优秀的库。
* 尽可能持久化数据，以便您的应用在脱机模式下依然可用。虽然您可能享受着快捷的网络，但您的用户可能不会。

> 另外，Kotlin 版本的多个官方 Sample 也公布啦，感兴趣的同学赶紧去看看吧：[Android Kotlin Samples](https://developer.android.com/samples/index.html?language=kotlin)
> 原文：[Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide.html#building_the_user_interface)



