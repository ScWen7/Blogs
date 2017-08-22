本篇作为 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)的第四篇。【LoadMoreRecyclerView扩展之添加Header和Footer】

 [从0开始开发一款应用市场APP系列](https://github.com/ScWen7/Blogs/blob/master/hwappstore1.md)连载记录项目[仿华为应用市场](https://github.com/ScWen7/HWAPPStore)的开发过程。

本系列的代码同步更新到[https://github.com/ScWen7/HWAPPStore](https://github.com/ScWen7/HWAPPStore) 。

---------------------------------------------------------------------------------------------------

# 前言

​     	在上一篇[封装加载更多的RecyclerView](http://www.jianshu.com/p/ceb6f4f64c99)中对RecylerView加载更多的封装进行了分析和实现，本篇是在上一篇的基础上对LoadMoreRecyclerView 进行进一步的扩展**支持添加Header和Footer View**

### 实现分析

在上一篇[封装加载更多的RecyclerView](http://www.jianshu.com/p/ceb6f4f64c99)中分析道RecyclerView 实现上拉刷新是采用封装Adapter的方式来实现的。而添加HeaderView 和FooterView 与其道理相同保留上拉加载将上拉加载更多的视图作为一个单独的Type 视图放在RecyclerView列表的底部，HeaderView作为一系列的类型放在列表的首端，FooterView作为一系列类型放在列表尾端，在上拉加载Type之前。各种类型示意图如下：

![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/loadrecy3.png)

**关于Header和Footer 本篇里面每添加一个Header或者Footer相当于添加以一个新的类型，如果要添加很多相同类型的Header或者Footer ，则需要进行修改，否则该类型的View的重用次数很少，影响RecyclerView 的性能**

#### Header
#### Footer
#### 封装完成代码

```java

public class LoadRecyclerView extends RecyclerView {

    //下面的ItemViewType是保留值(ReservedItemViewType),
    // 如果用户的adapter与它们重复将会强制抛出异常。不过为了简化,我们检测到重复时对用户的提示是ItemViewType必须小于10000
    //设置一个很大的数字,尽可能避免和用户的adapter冲突
    private static final int TYPE_LOADING = 10000;

    private static final int HEADER_INIT_INDEX = 10001;

    private static final int FOOTER_INIT_INDEX = 20000;

    //头布局的集合
    private SparseArrayCompat<View> mHeaderViews = new SparseArrayCompat<>();
    //尾部布局的集合
    private SparseArrayCompat<View> mFootViews = new SparseArrayCompat<>();


    private View mLoadingView; //加载更多的尾布局

    private boolean isLoadingData = false;
    private boolean isNoMore = false;

    private LoadingListener mLoadingListener;

    private boolean loadingMoreEnabled = true;


    //包装Adapter
    private WrapAdapter mWrapAdapter;


    //数据观察者
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

    private void init() {
        LoadingMoreFooter footView = new LoadingMoreFooter(getContext());
        mLoadingView = footView;
        mLoadingView.setVisibility(GONE);
    }

    /**
     * 设置加载中和加载完成的提示
     *
     * @param loading
     * @param noMore
     */
    public void setLoodingViewText(String loading, String noMore) {
        if (mLoadingView instanceof LoadingMoreFooter) {
            ((LoadingMoreFooter) mLoadingView).setLoadingHint(loading);
            ((LoadingMoreFooter) mLoadingView).setNoMoreHint(noMore);
        }
    }

    /**
     * 添加头布局
     *
     * @param view
     */
    public void addHeaderView(View view) {
        mHeaderViews.put(mHeaderViews.size() + HEADER_INIT_INDEX, view);
        if (mWrapAdapter != null) {
            mWrapAdapter.notifyItemRangeInserted(mHeaderViews.size() - 1, 1);
        }
    }

    //根据 itemType获取HeaderView
    private View getHeaderViewByType(int itemType) {
        View view = mHeaderViews.get(itemType);
        if (view == null) {
            throw new IllegalArgumentException(
                    "You May Have Not Add a Header Which Type Is "
                            + itemType);
        }
        return view;
    }


    public void addFooterView(View view) {
        mFootViews.put(mFootViews.size() + FOOTER_INIT_INDEX, view);
        if (mWrapAdapter != null) {
            int index = mWrapAdapter.getItemCount() - 1;
            if (loadingMoreEnabled) {
                index--;
            }
            mWrapAdapter.notifyItemRangeInserted(index, 1);
        }
    }

    //根据 itemType获取FooterView
    private View getFooterViewByType(int itemType) {
        View view = mFootViews.get(itemType);
        if (view == null) {
            throw new IllegalArgumentException(
                    "You May Have Not Add a Footer Which Type Is "
                            + itemType);
        }
        return view;
    }

    //判断是否是XRecyclerView保留的itemViewType
    private boolean isReservedItemViewType(int itemViewType) {
        return itemViewType == TYPE_LOADING || (mHeaderViews.indexOfKey(itemViewType) >= 0) || (mHeaderViews.indexOfKey(itemViewType) >= 0);
    }


    /**
     * 加载完成
     */
    public void loadMoreComplete() {
        isLoadingData = false;
        if (mLoadingView instanceof LoadingMoreFooter) {
            ((LoadingMoreFooter) mLoadingView).setState(LoadingMoreFooter.STATE_COMPLETE);
        } else {
            mLoadingView.setVisibility(View.GONE);
        }
    }

    /**
     * @param noMore
     */
    public void setNoMore(boolean noMore) {
        isLoadingData = false;
        isNoMore = noMore;
        if (mLoadingView instanceof LoadingMoreFooter) {
            ((LoadingMoreFooter) mLoadingView).setState(isNoMore ? LoadingMoreFooter.STATE_NOMORE : LoadingMoreFooter.STATE_COMPLETE);
        } else {
            mLoadingView.setVisibility(View.GONE);
        }
    }

    public void reset() {
        setNoMore(false);
        loadMoreComplete();
    }


    public void setLoadingMoreEnabled(boolean enabled) {
        loadingMoreEnabled = enabled;
        if (!enabled) {
            if (mLoadingView instanceof LoadingMoreFooter) {
                ((LoadingMoreFooter) mLoadingView).setState(LoadingMoreFooter.STATE_COMPLETE);
            }
        }
    }


    @Override
    public void setAdapter(Adapter adapter) {
        mWrapAdapter = new WrapAdapter(adapter);
        super.setAdapter(mWrapAdapter);
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
                        return (mWrapAdapter.isHeader(position) || mWrapAdapter.isFooter(position))
                                ? gridManager.getSpanCount() : 1;
                    }
                });

            }
        }
    }

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
                if (mLoadingView instanceof LoadingMoreFooter) {
                    ((LoadingMoreFooter) mLoadingView).setState(LoadingMoreFooter.STATE_LOADING);
                } else {
                    mLoadingView.setVisibility(View.VISIBLE);
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

    private class WrapAdapter extends RecyclerView.Adapter<ViewHolder> {

        private RecyclerView.Adapter adapter;

        public WrapAdapter(RecyclerView.Adapter adapter) {
            this.adapter = adapter;
        }

        public RecyclerView.Adapter getOriginalAdapter() {
            return this.adapter;
        }

        //判断当前位置是否是 Header
        public boolean isHeader(int position) {
            return position < mHeaderViews.size();
        }

        //判断当前位置是否是 Footer
        public boolean isFooter(int position) {
            boolean a = position >= getHeadersCount() + adapter.getItemCount();
            if (loadingMoreEnabled) {
                a = a && (position < getItemCount());
            }
            return a;
        }

        //判断当前位置是否是 Looding
        public boolean isLoading(int position) {
            return loadingMoreEnabled && position == getItemCount() - 1;
        }

        //Header 长度
        public int getHeadersCount() {
            return mHeaderViews.size();
        }


        //Footer　长度
        public int getFootersCount() {
            return mFootViews.size();
        }


        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            if (isHeaderType(viewType)) {
                return new SimpleViewHolder(getHeaderViewByType(viewType));
            } else if (isFootererType(viewType)) {
                return new SimpleViewHolder(getFooterViewByType(viewType));
            } else if (viewType == TYPE_LOADING) {
                return new SimpleViewHolder(mLoadingView);
            }
            return adapter.onCreateViewHolder(parent, viewType);
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
            if (isHeader(position) || isFooter(position)) {
                return;
            }
            int adjPosition = position - getHeadersCount();
            int adapterCount;
            if (adapter != null) {
                adapterCount = adapter.getItemCount();
                if (adjPosition < adapterCount) {
                    adapter.onBindViewHolder(holder, adjPosition);
                }
            }
        }

        // some times we need to override this
        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position, List<Object> payloads) {
            if (isHeader(position) || isFooter(position)) {
                return;
            }

            int adjPosition = position - getHeadersCount();
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
            int count = getHeadersCount() + getFootersCount();
            if (adapter != null) {
                count += adapter.getItemCount();
            }
            if (loadingMoreEnabled) {
                count++;
            }
            return count;
        }

        @Override
        public int getItemViewType(int position) {
            int adjPosition = position - (getHeadersCount());
            if (isHeader(position)) {
                return mHeaderViews.keyAt(position);
            } else if (isFooter(position)) {
                return mFootViews.keyAt(position);
            } else if (isLoading(position)) {
                return TYPE_LOADING;
            }

            int adapterCount;
            if (adapter != null) {
                adapterCount = adapter.getItemCount();
                if (adjPosition < adapterCount) {
                    int type = adapter.getItemViewType(adjPosition);
                    if (isReservedItemViewType(type)) {
                        throw new IllegalStateException("LoadRecycler require itemViewType in adapter should be less than 10000 ");
                    }
                    return type;
                }
            }
            return 0;
        }

        @Override
        public long getItemId(int position) {
            if (adapter != null && position >= getHeadersCount()) {
                int adjPosition = position - getHeadersCount();
                if (adjPosition < adapter.getItemCount()) {
                    return adapter.getItemId(adjPosition);
                }
            }
            return -1;
        }

        @Override
        public void onAttachedToRecyclerView(RecyclerView recyclerView) {
            super.onAttachedToRecyclerView(recyclerView);
            RecyclerView.LayoutManager manager = recyclerView.getLayoutManager();
            if (manager instanceof GridLayoutManager) {
                final GridLayoutManager gridManager = ((GridLayoutManager) manager);
                gridManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                    @Override
                    public int getSpanSize(int position) {
                        return (isHeader(position) || isFooter(position)) || isLoading(position)
                                ? gridManager.getSpanCount() : 1;
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
                    && (isHeader(holder.getLayoutPosition()) || isFooter(holder.getLayoutPosition()) || isLoading(holder.getLayoutPosition()))) {
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

    private boolean isHeaderType(int viewType) {
        return mHeaderViews.size() > 0 && (mHeaderViews.indexOfKey(viewType) >= 0);
    }

    private boolean isFootererType(int viewType) {
        return mFootViews.size() > 0 && (mFootViews.indexOfKey(viewType) >= 0);
    }

    private boolean isLoadingType(int viewType) {
        return viewType == TYPE_LOADING;
    }


    public void setLoadingListener(LoadingListener listener) {
        mLoadingListener = listener;
    }

    /**
     * 加载更多回调接口
     */

    public interface LoadingListener {

        void onLoadMore();
    }


}
```

