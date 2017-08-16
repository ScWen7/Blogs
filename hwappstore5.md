本篇作为 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)的第三篇。【封装加载更多的RecyclerView】

 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)连载记录项目[仿华为应用市场](https://github.com/ScWen7/HWAPPStore)的开发过程。

本系列的代码同步更新到[https://github.com/ScWen7/HWAPPStore](https://github.com/ScWen7/HWAPPStore) 。

---------------------------------------------------------------------------------------------------

# 前言

​        在一个项目中列表显示是必不可少的功能，现在用的最多的就是RecyclerView了，而上拉加载更多则是非常常见的功能。提到上拉加载，之前可能更多的是采用自定义控件的方式完成下拉刷新与上拉加载更多的功能，但是结合RecyclerView来使用，我决定将上拉加载加载更多的职责放到RecyclerView中。本文就是一个对RecylerView加载更多的一个封装。

### 实现分析

RecyclerView 实现上拉刷新是采用Adapter的方式来实现的。

RecyclerView提供了  **getItemViewType(int position) **来获取视图的类型，可以在这里下点功夫，将上拉加载更多的视图作为一个单独的Type 视图放在RecyclerView列表的底部，当列表滑动到底部的时候，即可显示加载更多的视图，并提供接口进行回调。

![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/loadrecy1.png)

#### Footer

##### LoadMoreFooter类代码：

```java
public class LoadingMoreFooter extends FrameLayout {

  //三种状态
    public final static int STATE_LOADING = 0;  //加载中
    public final static int STATE_COMPLETE = 1; //加载完成
    public final static int STATE_NOMORE = 2;  //没有更多了
  
    private TextView mText;
  //三种状态显示的文本
    private String loadingHint;
    private String noMoreHint;
    private String loadingDoneHint;

    private Context mContext;
    private ProgressBar mProgressBar;

    public LoadingMoreFooter(Context context) {
        super(context);
        this.mContext = context;
        initView();
    }

    /**
     * @param context
     * @param attrs
     */
    public LoadingMoreFooter(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.mContext = context;
        initView();
    }

    public void setLoadingHint(String hint) {
        loadingHint = hint;
    }

    public void setNoMoreHint(String hint) {
        noMoreHint = hint;
    }

    public void setLoadingDoneHint(String hint) {
        loadingDoneHint = hint;
    }

    public void initView() {
		//设置LayoutParams
        RecyclerView.LayoutParams layoutParams = new RecyclerView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        setLayoutParams(layoutParams);

        LayoutInflater layoutInflater = LayoutInflater.from(mContext);
      	//初始化布局
        View loadingView = layoutInflater.inflate(R.layout.loading_footer, this, true);
        mProgressBar = (ProgressBar) loadingView.findViewById(R.id.progress_loading);
        mText = (TextView) loadingView.findViewById(R.id.tv_hint);
        loadingHint = (String) getContext().getText(R.string.listview_loading);
        noMoreHint = (String) getContext().getText(R.string.nomore_loading);
        loadingDoneHint = (String) getContext().getText(R.string.loading_done);
    }

	//根据状态切切换显示
    public void setState(int state) {
        switch (state) {
            case STATE_LOADING:
                mProgressBar.setVisibility(View.VISIBLE);
                mText.setText(loadingHint);
                this.setVisibility(View.VISIBLE);
                break;
            case STATE_COMPLETE:
                mText.setText(loadingDoneHint);
                this.setVisibility(View.GONE);
                break;
            case STATE_NOMORE:
                mText.setText(noMoreHint);
                mProgressBar.setVisibility(View.GONE);
                this.setVisibility(View.VISIBLE);
                break;
        }
    }
}
```

其中用到的布局 xml文件：

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:gravity="center"
              android:padding="8dp"
              android:orientation="horizontal">

    <ProgressBar
        android:id="@+id/progress_loading"
        style="?android:progressBarStyleInverse"
        android:layout_width="35dp"
        android:layout_height="35dp"
        android:layout_gravity="center"
        android:visibility="visible"
        />

    <TextView
        android:id="@+id/tv_hint"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        android:text="数据正在加载中..."
        android:textSize="15sp"/>

</LinearLayout>
```

布局很简单，就一个圆形进度条和 提示文本![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/loadrecy2.png)

LoadMoreFooter 中初始化 加载更多的布局,并且定义了三种加载的状态：**加载中 **  **加载完成**   **没有更多**

并且能够根据三种状态切换视图的显示状态

#### RecyclerView 的代码

```java
public class LoadRecyclerView extends RecyclerView {

  //是否正在加载更多
    private boolean isLoadingData = false; 
  //是否还有更多
    private boolean isNoMore = false;
  	//是否能够加载更多
  private boolean loadingMoreEnabled = true;

    //包装Adapter
    private WrapAdapter mWrapAdapter;

  	//加载更多的监听
    private LoadingListener mLoadingListener;

   

    // 底部加载更多的 itemType
    //设置一个很大的数字,尽可能避免和用户的adapter冲突

    private static final int TYPE_FOOTER = 10001;

    private View mFootView; //尾部局

    //数据观察者
  	//这个很重要
    private final RecyclerView.AdapterDataObserver mDataObserver = new DataObserver();


    public LoadRecyclerView(Context context) {
        this(context, null);
    }

    public LoadRecyclerView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public LoadRecyclerView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }
	/**
	初始化LoadMoreFooter 
	*/
    private void init() {
        LoadingMoreFooter footView = new LoadingMoreFooter(getContext());
        mFootView = footView;
        mFootView.setVisibility(GONE);
    }

    /**
     * 设置加载中和加载完成的提示
     *
     * @param loading
     * @param noMore
     */
    public void setFootViewText(String loading, String noMore) {
        if (mFootView instanceof LoadingMoreFooter) {
            ((LoadingMoreFooter) mFootView).setLoadingHint(loading);
            ((LoadingMoreFooter) mFootView).setNoMoreHint(noMore);
        }
    }

    public void setFootView(final View view) {
        mFootView = view;
    }

    /**
     * 加载完成
     */
    public void loadMoreComplete() {
        isLoadingData = false;
        if (mFootView instanceof LoadingMoreFooter) {
            ((LoadingMoreFooter) mFootView).setState(LoadingMoreFooter.STATE_COMPLETE);
        } else {
            mFootView.setVisibility(View.GONE);
        }
    }

    /**
     * @param noMore   是否还有更多数据
     */
    public void setNoMore(boolean noMore) {
        isLoadingData = false;
        isNoMore = noMore;
        if (mFootView instanceof LoadingMoreFooter) {
            ((LoadingMoreFooter) mFootView).setState(isNoMore ? LoadingMoreFooter.STATE_NOMORE : LoadingMoreFooter.STATE_COMPLETE);
        } else {
            mFootView.setVisibility(View.GONE);
        }
    }
	 /**
     * 重置加载更多的状态
     */
    public void reset() {
        setNoMore(false);
        loadMoreComplete();
    }

	 /**
     * 是否还能够加载更多
     */
    public void setLoadingMoreEnabled(boolean enabled) {
        loadingMoreEnabled = enabled;
        if (!enabled) {
            if (mFootView instanceof LoadingMoreFooter) {
                ((LoadingMoreFooter) mFootView).setState(LoadingMoreFooter.STATE_COMPLETE);
            }
        }
    }


    @Override
    public void setAdapter(Adapter adapter) {
      	//对用户的adapter进行包装
        mWrapAdapter = new WrapAdapter(adapter);
        super.setAdapter(mWrapAdapter);
      	//用户的adapter 注册 观察者
        adapter.registerAdapterDataObserver(mDataObserver);
        mDataObserver.onChanged();
    }

    //避免用户自己调用getAdapter() 引起的ClassCastException
    @Override
    public Adapter getAdapter() {
        if (mWrapAdapter != null)
            return mWrapAdapter.getOriginalAdapter();
        else
            return null;
    }

    @Override
    public void setLayoutManager(LayoutManager layout) {
        super.setLayoutManager(layout);
        if (mWrapAdapter != null) {
            if (layout instanceof GridLayoutManager) {
                final GridLayoutManager gridManager = ((GridLayoutManager) layout);
                gridManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                    @Override
                    public int getSpanSize(int position) {
                      //设置LoadMoreFooter占据的列数，保持填满RecyclerView 的宽度
                        return mWrapAdapter.isFooter(position) ? gridManager.getSpanCount() : 1;
                    }
                });
            }
        }
    }
	/**
	*监听RecyclerView 的滑动状态，判断是否需要执行上拉加载
	*/
    @Override
    public void onScrollStateChanged(int state) {
        super.onScrollStateChanged(state);
        if (state == RecyclerView.SCROLL_STATE_IDLE && mLoadingListener != null && !isLoadingData && loadingMoreEnabled) {
            LayoutManager layoutManager = getLayoutManager();
            int lastVisibleItemPosition;
            if (layoutManager instanceof GridLayoutManager) {
                lastVisibleItemPosition = ((GridLayoutManager) layoutManager).findLastVisibleItemPosition();
            } else if (layoutManager instanceof StaggeredGridLayoutManager) {
                int[] into = new int[((StaggeredGridLayoutManager) layoutManager).getSpanCount()];
                ((StaggeredGridLayoutManager) layoutManager).findLastVisibleItemPositions(into);
                lastVisibleItemPosition = findMax(into);
            } else {
                lastVisibleItemPosition = ((LinearLayoutManager) layoutManager).findLastVisibleItemPosition();
            }
            if (layoutManager.getChildCount() > 0
                    && lastVisibleItemPosition >= layoutManager.getItemCount() - 1 && layoutManager.getItemCount() > layoutManager.getChildCount() && !isNoMore) {
                isLoadingData = true;
                if (mFootView instanceof LoadingMoreFooter) {
                    ((LoadingMoreFooter) mFootView).setState(LoadingMoreFooter.STATE_LOADING);
                } else {
                    mFootView.setVisibility(View.VISIBLE);
                }
                mLoadingListener.onLoadMore();
            }
        }
    }


    private int findMax(int[] lastPositions) {
        int max = lastPositions[0];
        for (int value : lastPositions) {
            if (value > max) {
                max = value;
            }
        }
        return max;
    }

	/**
	* 为用户的adapter 设置观察者  当用户调用相应的 notify方法时，会执行WrapAdapter 的notify方法
	*/
    private class DataObserver extends RecyclerView.AdapterDataObserver {
        @Override
        public void onChanged() {
            if (mWrapAdapter != null) {
                mWrapAdapter.notifyDataSetChanged();
            }
        }

        @Override
        public void onItemRangeInserted(int positionStart, int itemCount) {
            mWrapAdapter.notifyItemRangeInserted(positionStart, itemCount);
        }

        @Override
        public void onItemRangeChanged(int positionStart, int itemCount) {
            mWrapAdapter.notifyItemRangeChanged(positionStart, itemCount);
        }

        @Override
        public void onItemRangeChanged(int positionStart, int itemCount, Object payload) {
            mWrapAdapter.notifyItemRangeChanged(positionStart, itemCount, payload);
        }

        @Override
        public void onItemRangeRemoved(int positionStart, int itemCount) {
            mWrapAdapter.notifyItemRangeRemoved(positionStart, itemCount);
        }

        @Override
        public void onItemRangeMoved(int fromPosition, int toPosition, int itemCount) {
            mWrapAdapter.notifyItemMoved(fromPosition, toPosition);
        }
    }
	//包装的WrapAdapter
    private class WrapAdapter extends RecyclerView.Adapter<ViewHolder> {

      	//用户的adapter
        private RecyclerView.Adapter adapter;

        public WrapAdapter(RecyclerView.Adapter adapter) {
            this.adapter = adapter;
        }
		//对外暴露出去
        public RecyclerView.Adapter getOriginalAdapter() {
            return this.adapter;
        }

		/**
		*判断是否是 尾部
		*/
        public boolean isFooter(int position) {
            return loadingMoreEnabled && position == getItemCount() - 1;
        }


        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            if (viewType == TYPE_FOOTER) { //如果是尾部
                return new SimpleViewHolder(mFootView);
            }
          	//交由用户的adapter 来执行
            return adapter.onCreateViewHolder(parent, viewType);
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {

            int adjPosition = position;
            int adapterCount;
            if (adapter != null) {
                adapterCount = adapter.getItemCount();
                if (adjPosition < adapterCount) {  //类型是正常类型时才执行adapter 的 onBind
                    adapter.onBindViewHolder(holder, adjPosition);
                }
            }
        }

        // some times we need to override this
        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position, List<Object> payloads) {


            int adjPosition = position;
            int adapterCount;
            if (adapter != null) {
                adapterCount = adapter.getItemCount();
                if (adjPosition < adapterCount) {
                    if (payloads.isEmpty()) {
                        adapter.onBindViewHolder(holder, adjPosition);
                    } else {
                        adapter.onBindViewHolder(holder, adjPosition, payloads);
                    }
                }
            }
        }

        @Override
        public int getItemCount() {
            if (loadingMoreEnabled) {
                if (adapter != null) {
                    return adapter.getItemCount() + 1;
                } else {
                    return 1;
                }
            } else {
                if (adapter != null) {
                    return adapter.getItemCount();
                } else {
                    return 0;
                }
            }
        }

        @Override
        public int getItemViewType(int position) {
            int adjPosition = position;
            if (isFooter(position)) {
                return TYPE_FOOTER;
            }
            int adapterCount;
            if (adapter != null) {
                adapterCount = adapter.getItemCount();
                if (adjPosition < adapterCount) {
                    int type = adapter.getItemViewType(adjPosition);
                    return type;
                }
            }
            return 0;
        }

        @Override
        public long getItemId(int position) {
            if (adapter != null) {
                int adjPosition = position;
                if (adjPosition < adapter.getItemCount()) {
                    return adapter.getItemId(adjPosition);
                }
            }
            return -1;
        }

        @Override
        public void onAttachedToRecyclerView(RecyclerView recyclerView) {
            super.onAttachedToRecyclerView(recyclerView);
          //这个方法是设置Footer 占据的列数
            RecyclerView.LayoutManager manager = recyclerView.getLayoutManager();
            if (manager instanceof GridLayoutManager) {
                final GridLayoutManager gridManager = ((GridLayoutManager) manager);
                gridManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                    @Override
                    public int getSpanSize(int position) {
                        return isFooter(position) ? gridManager.getSpanCount() : 1;
                    }
                });
            }
            adapter.onAttachedToRecyclerView(recyclerView);
        }

        @Override
        public void onDetachedFromRecyclerView(RecyclerView recyclerView) {
            adapter.onDetachedFromRecyclerView(recyclerView);
        }

        @Override
        public void onViewAttachedToWindow(RecyclerView.ViewHolder holder) {
            super.onViewAttachedToWindow(holder);
            ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
            if (lp != null
                    && lp instanceof StaggeredGridLayoutManager.LayoutParams
                    && isFooter(holder.getLayoutPosition())) {
                StaggeredGridLayoutManager.LayoutParams p = (StaggeredGridLayoutManager.LayoutParams) lp;
                p.setFullSpan(true);
            }
            adapter.onViewAttachedToWindow(holder);
        }

        @Override
        public void onViewDetachedFromWindow(RecyclerView.ViewHolder holder) {
            adapter.onViewDetachedFromWindow(holder);
        }

        @Override
        public void onViewRecycled(RecyclerView.ViewHolder holder) {
            adapter.onViewRecycled(holder);
        }

        @Override
        public boolean onFailedToRecycleView(RecyclerView.ViewHolder holder) {
            return adapter.onFailedToRecycleView(holder);
        }

        @Override
        public void unregisterAdapterDataObserver(AdapterDataObserver observer) {
            adapter.unregisterAdapterDataObserver(observer);
        }

        @Override
        public void registerAdapterDataObserver(AdapterDataObserver observer) {
            adapter.registerAdapterDataObserver(observer);
        }

        private class SimpleViewHolder extends RecyclerView.ViewHolder {
            public SimpleViewHolder(View itemView) {
                super(itemView);
            }
        }
    }

    public void setLoadingListener(LoadingListener listener) {
        mLoadingListener = listener;
    }

  	//定义加载更多的接口
    public interface LoadingListener {
        void onLoadMore();
    }

}
```



