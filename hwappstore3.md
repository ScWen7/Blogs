本篇作为 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)的第二篇。【MVP使用案例】

 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)连载记录项目[仿华为应用市场](https://github.com/ScWen7/HWAPPStore)的开发过程。

本系列的代码同步更新到[https://github.com/ScWen7/HWAPPStore](https://github.com/ScWen7/HWAPPStore) 。

---------------------------------------------------------------------------------------------------

# 前言

​        在上一篇[MVC与MVP]()中详细介绍了MVC与MVP之间的关联和区别，看了上一篇，相信大家对MVC和MVP都有了一定的认识。有了理论知识，下面让我们结合实际业务和代码来看看MVP 的应用。

# 业务场景

这里为了测试使用Mvp,使用了[干货集中营Api](http://gank.io/api)。

Demo 中的流程大概是是这样的

1.界面启动 请求网络Api 显示列表

2.进行下拉刷新，刷新列表

3.进行上拉加载，加载更多数据

案例会采用Mvp的结构去进行完成。

# 代码分析

## 1、项目的package 的结构

![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/mvp1.png)

关于Mvp 的三个角色：

data与model 对应Mvp 中的Model 层保存实体Bean和数据处理

presenter 对应Mvp 中的Presenter层，逻辑控制层

ui 对应Mvp中的View 层，负责显示，包含了activity、fragment、adapter、view接口等

其他的分包内容可以到[BaseProject](https://github.com/ScWen7/BaseProject)进行查看。

## 2、关于Base

### BasePresenter代码如下

``` java
public abstract class BasePresenter<M, V extends BaseView> {

    private CompositeDisposable disposables;// 管理Destroy取消订阅者

    protected M mModel;
    protected V mView;

    protected Context mContext;

    public BasePresenter(V view) {
        mView = view;
        initContext(view);
        mModel = createModel();
    }

    protected void initContext(V view) {
        if (view instanceof Activity) {
            //Activity
            mContext = (Activity) view;
        } else {
            mContext = ((Fragment) view).getActivity();
        }

    }

    public boolean addRx(Disposable disposable) {
        if (disposables == null) {
            disposables = new CompositeDisposable();
        }
        disposables.add(disposable);
        return true;
    }


    public void removeRx(Disposable disposable) {
        if (disposables == null) {
            disposables.remove(disposable);
        }

    }

    public Context getContext() {
        return mContext;
    }

    protected abstract M createModel();


    public void detachView() {
        if (disposables != null) {
            disposables.dispose();
            disposables = null;
        }
    }
    
}
```

代码简单说明：

1、Presenter依赖Model 与View ,View 在Presenter的构造器中进行初始化（并且View通过泛型限定了必须是BaseView 的子类），Model则通过抽象方法交由子类去进行实例化

2、在很多情况下Presenter 中需要Context对象，Model也需要Context,所以在构造函数中 进行了Context 的初始化

3、由于项目使用了RxJava 需要在结束时，进行及时的取消订阅，所以在BasePresenter 中保持了一个CompositeDisposable管理订阅者，在Activity 或者Fragment销毁的时候 调用detachView()进行资源释放。

### BaseMvpActivity代码如下：

```jav
public abstract class BaseMvpActivity<P extends BasePresenter> extends BaseActivity {

   protected P mPresenter;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mPresenter = createPresenter();
        initData();
    }

    protected abstract void initData();


    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mPresenter != null) {
            mPresenter.detachView();
            mPresenter = null;
        }
    }

    /**
     * 创建 Presenter
     *
     * @return
     */
    public abstract P createPresenter();
}
```

代码很简单，抽象方法初始化Presenter，并且在onDestroy方法中执行Presenter detachView()方法并释放Presenter

### 业务代码

#### 1、Model层

##### Retrofit 的请求ApiService类：

```java
public interface ApiService {
    @GET("{page}")
    Observable<GankRestlt<List<GankBean>>> getGankData(@Path("page") int page);
}
```



##### 请求完成解析的实体类GankBean 

```java
public class GankBean {

    /**
     * _id : 597dce34421aa90ca209c51b
     * createdAt : 2017-07-30T20:16:52.80Z
     * desc : 一个极简但是强大的VR本地播放器，基于IJKPlayer、MD360Player4Android，并使用DataBinding
     * images : ["http://img.gank.io/ea71986c-4e0f-4b21-97a5-06dc311fff0b"]
     * publishedAt : 2017-08-09T13:49:27.260Z
     * source : web
     * type : Android
     * url : https://github.com/wheat7/VRPlayer
     * used : true
     * who : 麦田哥
     */

    private String _id;
    private String createdAt;
    private String desc;
    private String publishedAt;
    private String source;
    private String type;
    private String url;
    private boolean used;
    private String who;
    private List<String> images;

    public String get_id() {
        return _id;
    }

    public void set_id(String _id) {
        this._id = _id;
    }

    public String getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(String createdAt) {
        this.createdAt = createdAt;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    public String getPublishedAt() {
        return publishedAt;
    }

    public void setPublishedAt(String publishedAt) {
        this.publishedAt = publishedAt;
    }

    public String getSource() {
        return source;
    }

    public void setSource(String source) {
        this.source = source;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public boolean isUsed() {
        return used;
    }

    public void setUsed(boolean used) {
        this.used = used;
    }

    public String getWho() {
        return who;
    }

    public void setWho(String who) {
        this.who = who;
    }

    public List<String> getImages() {
        return images;
    }

    public void setImages(List<String> images) {
        this.images = images;
    }
}

```

##### Model提供数据方法

```java
public class MvpModel {

    public Observable<List<GankBean>> getGankData(int page) {
        return RetrofitClient.getInstance().provideApiService()
                .getGankData(page)
                .compose(RxResultCompat.<List<GankBean>>handleResult())  //Retrofit+RxJava 的数据预解析
                .compose(RxSchedulerHepler.<List<GankBean>>io_main());   //线程切换
    }
}
```

#### 2、View 层

#### 界面layout.xml如下

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/refresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.scwen7.mvpdemo.ui.activity.MvpTestActivity">

    <com.scwen7.mvpdemo.weight.loadmoreRecycler.XRecyclerView
        android:id="@+id/recycler_mvp"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</android.support.v4.widget.SwipeRefreshLayout>
```

界面和简单就是一个下拉刷新控件和封装好的加载更多的RecyclerView

##### View接口定义

```java
//接口定义如下
public interface MvpView extends BaseView {

     void loadMoreSuccess(List<GankBean> transactions);  //加载更多完成

     void loadMoreFailed() ;  //加载更多失败


     void refreshFailed() ;  //刷新失败


     void refreshSuccess(List<GankBean> transactions); //刷新成功
}
```

##### MvpActivity代码实现 

```java
public class MvpTestActivity extends BaseMvpActivity<MvpPresenter> implements MvpView {


    @BindView(R.id.recycler_mvp)
    XRecyclerView mRecyclerMvp;
    @BindView(R.id.refresh)
    SwipeRefreshLayout mRefresh;
  	//数据集合
    private List<GankBean> mGankBeanList = new ArrayList<>();
    private GankAdapter mGankAdapter;

    @Override
    protected void initView() {
      //设置下拉刷新控件
        mRefresh.setColorSchemeResources(android.R.color.holo_purple, android.R.color.holo_green_light, android.R.color.holo_red_light, 	android.R.color.holo_blue_light);
        mRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                //下拉刷新
                mPresenter.getGankData(LoadStatus.REFRESH);
            }
        });
		//初始化RecyclerView
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
        mRecyclerMvp.setLayoutManager(linearLayoutManager);
        mRecyclerMvp.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL));
        mGankAdapter = new GankAdapter(this, mGankBeanList);
        mRecyclerMvp.setAdapter(mGankAdapter);

		//上拉加载更多监听
        mRecyclerMvp.setLoadingListener(new XRecyclerView.LoadingListener() {
            @Override
            public void onLoadMore() {
                mPresenter.getGankData(LoadStatus.LOADMORE);
            }
        });

    }

    @Override
    protected int getLayoutId() {
        return R.layout.activity_mvp_test;
    }

    @Override
    protected void initData() {
        mRefresh.setRefreshing(true);  //显示刷新视图
        mPresenter.getGankData(LoadStatus.REFRESH);  //开始请求数据
    }

    @Override
    public MvpPresenter createPresenter() {
        return new MvpPresenter(this);
    }

    @Override
    public void loadMoreSuccess(List<GankBean> transactions) {
      //判断数据是否合适
        if (transactions != null && transactions.size() > 0) {
            mRecyclerMvp.loadMoreComplete();
            mGankAdapter.addData(mGankAdapter.getItemCount(), transactions);
        } else {
            mRecyclerMvp.setNoMore(true); //没有更多的数据了
        }
    }

    @Override
    public void loadMoreFailed() {
        mRecyclerMvp.loadMoreComplete();
    }

    @Override
    public void refreshFailed() {
        mRefresh.setRefreshing(false);
    }

    @Override
    public void refreshSuccess(List<GankBean> transactions) {
        mRefresh.setRefreshing(false);
        mGankAdapter.setData(transactions); //刷新列表
    }
}
```

####  GankAdapter代码

```java
public class GankAdapter extends RecyclerView.Adapter<GankAdapter.MyViewHolder> {

    private Context mContext;
    private List<GankBean> mGankBeanList;
    private LayoutInflater mLayoutInflater;


    public GankAdapter(Context context, List<GankBean> gankBeanList) {
        this.mContext = context;
        this.mGankBeanList = gankBeanList;
        mLayoutInflater = LayoutInflater.from(mContext);
    }

    public void addData(int position, List<GankBean> data) {
        if (data != null && data.size() > 0) {
            mGankBeanList.addAll(position, data);
            notifyItemRangeInserted(position, data.size());
        }
    }

    public void setData(List<GankBean> list) {
        if (list != null) {
            mGankBeanList = list;
            notifyDataSetChanged();
        }
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View contentView = mLayoutInflater.inflate(R.layout.item_list, parent, false);
        return new MyViewHolder(contentView);
    }

    @Override
    public void onBindViewHolder(MyViewHolder holder, int position) {
        holder.setData();
    }

    @Override
    public int getItemCount() {
        return mGankBeanList == null ? 0 : mGankBeanList.size();
    }

    class MyViewHolder extends RecyclerView.ViewHolder {

        private TextView tvTitle;
        private TextView tvUrl;

        public MyViewHolder(View itemView) {
            super(itemView);
            tvTitle = (TextView) itemView.findViewById(R.id.tv_title);
            tvUrl = (TextView) itemView.findViewById(R.id.tv_url);
        }

        public void setData() {
            int position = getAdapterPosition();
            GankBean gankBean = mGankBeanList.get(position);
            tvTitle.setText(gankBean.getDesc());
            tvUrl.setText(gankBean.getUrl());
        }
    }
}
```

Adapter 的代码没什么好说的，已经写了N多次了

### Presenter 层代码实现

```java
public class MvpPresenter extends BasePresenter<MvpModel, MvpView> {


    private int page = 1; //加载的页数

    public MvpPresenter(MvpView view) {
        super(view);
    }

    @Override
    protected MvpModel createModel() {
        return new MvpModel();  //初始化MOdel
    }

	//加载数据
    public void getGankData(final LoadStatus loadStatus) {
        if (loadStatus == LoadStatus.REFRESH) {
            page = 1;
        } else {
            page++;
        }
        addRx(mModel.getGankData(page)
                .subscribe(new Consumer<List<GankBean>>() {
                    @Override
                    public void accept(@NonNull List<GankBean> gankBeen) throws Exception {
                        if (loadStatus == LoadStatus.REFRESH) {
                            mView.refreshSuccess(gankBeen);
                        } else {
                            mView.loadMoreSuccess(gankBeen);
                        }
                    }
                }, new RxExceptionHandler<Throwable>(new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {

                        if (loadStatus == LoadStatus.REFRESH) {
                            mView.refreshFailed();
                        } else {
                            mView.loadMoreFailed();
                        }
                    }
                })));
    }
}
```

Presenter主要做了以下几件事情：

1、初始化了View。

2、通过View传递过来的指令向Model层请求数据。

3、监听Model层的状态，并将结果刷新到View上。

### 总结分析

Mvp注重的是接口的使用，体现了Java类的继承性，多态性，很重要的一点就是方法的重写。

MVP把activity作为了view层，通过代码也可以看到，整个activity没有任何和model层相关的逻辑代码，取而代之的是把代码放到了presenter层中，presenter获取了model层的数据之后，通过接口的形式将view层需要的数据返回给它就OK了，activity 完全不用理会数据是如何处理的。这样的好处是什么呢？

1、学习过设计模式的人都知道，这样做基本符合了单一职责原则。activity的代码逻辑减少了view层和model层完全解耦。

2、符合单一职责原则后，逻辑十分清晰。

3、View层与Model层交互需要通过Presenter层进行，这样v与m层级间的耦合性降低。

4、通过这种分层处理，每一层的测试也相对简单，维护性更高。如果你需要测试一个http请求是否顺利，你不需要写一个activity，只需要写一个java类，实现对应的接口，presenter获取了数据自然会调用相应的方法，相应的，你也可以自己在presenter中mock数据，分发给view层，用来测试布局是否正确。