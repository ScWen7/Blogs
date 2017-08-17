# RecyclerView Adapter封装2——多类型Item 的Adapter封装

在上一篇[RecyclerView Adapter封装1—单类型Item 的Adapter封装](http://www.jianshu.com/p/ae17c86b734b)中分析了RecyclerView 的Adapter,并对单类型Item的Adapter进行了封装

这一篇继续对多类型Item 的Adapter进行封装。

### 多类型Adapter代码分析

下面是一个很简单的多类型Adapter，分析代码找出共同点进行抽取封装

```java
public class MultiItemTypeAdapter extends RecyclerView.Adapter {

    private Context mContext;
    private LayoutInflater mLayoutInflater;//布局渲染器

    public static final int TYPE_1 = 0; //类型1

    public static final int TYPE_2 = 1; //类型2

    private List<MultiItem> mDatas;

    public MultiItemTypeAdapter(Context context, List<MultiItem> datas) {
        this.mContext = context;
        mLayoutInflater = LayoutInflater.from(context);
        this.mDatas = datas;
    }


    @Override
    public int getItemViewType(int position) {
        int itemType = 0;
        int type = mDatas.get(position).getType(); //判断类型
        switch (type) {
            case TYPE_1:
                itemType = TYPE_1;
                break;
            case TYPE_2:
                itemType = TYPE_2;
                break;
        }
        return itemType;

    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
      	//根据不同的类型来创建 ViewHolder
        if (viewType == TYPE_1) {
            return new Type1Holder(mLayoutInflater.inflate(R.layout.item_type1, parent, false));
        } else if (viewType == TYPE_2) {
            return new Type2Holder(mLayoutInflater.inflate(R.layout.item_type2, parent, false));
        }
        return null;
    }


    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
      //根据类型来区分绑定数据
        if (getItemViewType(position) == TYPE_1) {
            Type1Holder type1Holder = (Type1Holder) holder;
            type1Holder.tv_type1.setText(mDatas.get(position).getContent());
        } else if (getItemViewType(position) == TYPE_2) {
            Type2Holder type2Holder = (Type2Holder) holder;
            type2Holder.tv_type2.setText(mDatas.get(position).getContent());
        }

    }
	
  //类型1 对应的ViewHolder
    public class Type1Holder extends RecyclerView.ViewHolder {

        public TextView tv_type1;

        public Type1Holder(View itemView) {
            super(itemView);
            tv_type1 = (TextView) itemView.findViewById(R.id.tv_type1);
        }
    }
	//类型1 对应的ViewHolder
    public class Type2Holder extends RecyclerView.ViewHolder {

        public TextView tv_type2;

        public Type2Holder(View itemView) {
            super(itemView);
            tv_type2 = (TextView) itemView.findViewById(R.id.tv_type2);
        }
    }


    @Override
    public int getItemCount() {
        return mDatas == null ? 0 : mDatas.size();
    }

    @Override
    public void onAttachedToRecyclerView(RecyclerView recyclerView) {
        super.onAttachedToRecyclerView(recyclerView);
        RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            final GridLayoutManager gridManager = ((GridLayoutManager) layoutManager);
            gridManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                @Override
                public int getSpanSize(int position) {
                  //类型1需要在网格布局时 占据全行
                    int type = getItemViewType(position);
                    return type == TYPE_1 ? 1 : gridManager.getSpanCount();
                }
            });
        }
    }

}
```

以上代码中

 **getItemType() **方法很重要，这是分类型的前提，RecyclerView进行展示的时候回来调用这个方法确定Item 的类型

**onCreateViewHolder**方法，参方法的参数**viewType**即为getItemType()方法返回的类型，在此方法中我们可以根据类型创建不同的ViewHolder保存View

**onBindViewHolder**方法，需要在此方法中判断类型，并给不同类型的ItemView 绑定数据

**onAttachedToRecyclerView**方法,这个方法不是必要的 

![](http://oqe10cpgp.bkt.clouddn.com/image/hwappstore/recyadapter_1.png)

但是当某个类型占据一整行，另一个类型只需要一列的情况，这个方法就可以起作用了。

分析MultiItemTypeAdapter的代码发现，我们重点关心的是 

1、每一个类型的Type,什么时候绑定当前的Type

4、每个类型对应layoutId

3、每个类型都有不同的ViewHolder，对于这个问题，我们已经在[RecyclerView Adapter封装1—单类型Item 的Adapter封装](http://www.jianshu.com/p/ae17c86b734b)封装了通用的ViewHolder

4、每个类型如何绑定数据

### MultiItemTypeAdapter代码封装

经过以上的分析，明确了四个要素点 

① 类型的Type  ②类型何时需要被绑定  ③类型的layoutId   ④如何绑定数据

##### 1、关于Type和类型何时需要被绑定

在最初的时候采用了拓展Bean类来实现

```java
public interface MultiType {
    int getItemType();
}
```

由Bean 实现此接口来提供ItemType,使用泛型限定类型为MultiItemTypeAdapter<? extends MultiType>代码就可以写为：

```java
 @Override
    public int getItemViewType(int position) {
       	
        return mDats.getItemType();

    }
```

之后觉得这样对Bean类的侵入性较强，需要处理数据，就放弃了这种写法。

#### 2、通过分析，发现每一种类型相对独立，那么可以按照封装性的原则将每一种类型封装成一个Delegate，然后统一进行管理

```java
public interface ItemViewDelegate<T>
{
	//获取Item类型
  	int getItemType();
  	//是否绑定数据
  	boolean isForViewType(T item, int position);
  	//类型的Item
    int getItemViewLayoutId();
	//绑定数据
    void convert(ViewHolder holder, T t, int position);

}
```

针对四个要素抽象出四个接口由子类去提供，接下来重点就在于Delegate 的管理类  **ItemViewDelegateManager**

对于MultiItemTypeAdapter，可见的只有ItemViewDelegateManager类，通过ItemViewDelegateManager来管理多种类型，具体的类型提供和绑定由ItemViewDelegateManager内部来完成

ItemViewDelegateManager的职责：

1、通过分析getItemViewType(int position) 方法 该类需要提供 **根据 position 来获取ItemType** 的方法

2、通过分析onCreateViewHolder() 方法，该类需要提供**根据itemType来获取layoutId **的方法，用来创建通用的ViewHolder

3、通过分析onBindViewHolder()方法，该类需要提供 **根据position 来绑定数据** 的方法

#### 3、ItemViewDelegateManager代码

```java

public class ItemViewDelegateManager<T> {
  	//存储类型的容器
    SparseArray<ItemViewDelegate<T>> delegates = new SparseArray();

    public int getItemViewDelegateCount() {
        return delegates.size();
    }

    public ItemViewDelegateManager<T> addDelegate(ItemViewDelegate<T> delegate) {
		int viewType = delegate.getItemType();
      //判断之前是否已经添加过
        if (delegates.get(viewType) != null) {
            throw new IllegalArgumentException(
                    "An ItemViewDelegate is already registered for the viewType = "
                            + viewType
                            + ". Already registered ItemViewDelegate is "
                            + delegates.get(viewType));
        }
        delegates.put(viewType, delegate);
        return this;
    }

    public ItemViewDelegateManager<T> removeDelegate(ItemViewDelegate<T> delegate) {
      
        int indexToRemove = delegates.indexOfValue(delegate);

        if (indexToRemove >= 0) {
            delegates.removeAt(indexToRemove);
        }
        return this;
    }

    public ItemViewDelegateManager<T> removeDelegate(int itemType) {
        int indexToRemove = delegates.indexOfKey(itemType);
        if (indexToRemove >= 0) {
            delegates.removeAt(indexToRemove);
        }
        return this;
    }

  //根据position 来获取类型
    public int getItemViewType(T item, int position) {
        int delegatesCount = delegates.size();
        for (int i = 0; i <delegatesCount; i++) {
            ItemViewDelegate<T> delegate = delegates.valueAt(i);
            if (delegate.isForViewType(item, position)) {
                return delegates.keyAt(i);
            }
        }
      //没有添加过抛出异常
        throw new IllegalArgumentException(
                "No ItemViewDelegate added that matches position=" + position);
    }
	//根据类型来绑定数据
    public void convert(ViewHolder holder, T item, int position) {
        int delegatesCount = delegates.size();
        for (int i = 0; i < delegatesCount; i++) {
            ItemViewDelegate<T> delegate = delegates.valueAt(i);
            if (delegate.isForViewType(item, position)) {
                delegate.convert(holder, item, position);
                return;
            }
        }
        throw new IllegalArgumentException(
                "No ItemViewDelegateManager added that matches position=" + position + " in data source");
    }

	//根据type获取getItemViewDelegate对象
    public ItemViewDelegate getItemViewDelegate(int viewType) {
        return delegates.get(viewType);
    }
	//根据viewType获取对应的layoutId
    public int getItemViewLayoutId(int viewType) {
        return getItemViewDelegate(viewType).getItemViewLayoutId();
    }
	//获取指定ItemViewDelegate对应的类型
    public int getItemViewType(ItemViewDelegate itemViewDelegate) {
        return delegates.indexOfValue(itemViewDelegate);
    }
}

```

存储的容器还是采用SparseArray，SparseArray在这样保存的key为 int型itemType的场景下非常好用。

- addDelegate(ItemViewDelegate<T> delegate)方法 ，将ItemViewDelegate添加到容器中，key 为 ItemViewDelegate对应的itemType


- 着重分析一下getItemViewType()方法

```java

    public int getItemViewType(T item, int position) {
        int delegatesCount = delegates.size();
        for (int i = 0; i <delegatesCount; i++) {
            ItemViewDelegate<T> delegate = delegates.valueAt(i);
            if (delegate.isForViewType(item, position)) {
                return delegates.keyAt(i);
            }
        }
      //没有添加过抛出异常
        throw new IllegalArgumentException(
                "No ItemViewDelegate added that matches position=" + position);
    }
```

对于一种类型，你不能确定什么时候来绑定这种类型，可能是1、3、5，也可能是2、4、6，所以才有了isForViewType(T item, int position)来判断某个位置是否需要绑定，而类型现在存储在SparseArray中，所以需要通过循环来判定 某个position 需要哪一个ItemViewDelegate来进行绑定

- convert(ViewHolder holder, T item, int position)方法与getItemType同理，判断哪一个类型需要绑定，直接进行绑定
- getItemViewLayoutId() 直接根据type 在SparseArray匹配ItemViewDelegate获取layoutId

#### 4、MultiItemTypeAdapter实现

```java

public class MultiItemTypeAdapter<T> extends RecyclerView.Adapter<ViewHolder> {
    protected Context mContext;
  //数据集合
    protected List<T> mDatas;
	//类型管理器
    protected ItemViewDelegateManager mItemViewDelegateManager;

    protected OnItemClickListener mOnItemClickListener;
  
   private SparseArray<Integer> fullTypes = new SparseArray<>();


    public MultiItemTypeAdapter(Context context, List<T> datas) {
        mContext = context;
        mDatas = datas;
        mItemViewDelegateManager = new ItemViewDelegateManager();
    }

    @Override
    public int getItemViewType(int position) {
     
        return mItemViewDelegateManager.getItemViewType(mDatas.get(position), position);
    }

    public void clearData() {
        mDatas.clear();
        notifyItemRangeRemoved(0, mDatas.size());
    }

    public void removeData(int position) {
        mDatas.remove(position);
        notifyItemRemoved(position);
        if (position != mDatas.size())
            notifyItemRangeChanged(position, mDatas.size() - position);
    }

    public void remove(int position) {
        mDatas.remove(position);
        notifyDataSetChanged();
    }


    public void addData(List<T> list) {
        addData(0, list);
    }

    public void addData(int position, List<T> data) {
        if (data != null && data.size() > 0) {
            mDatas.addAll(position, data);
            notifyItemRangeInserted(position, data.size());
        }
    }

    public void setData(List<T> list) {
        if (list != null) {
            mDatas = list;
            notifyDataSetChanged();
        }
    }

    public void refresh(List<T> data) {
        clearData();
        addData(data);
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        ItemViewDelegate itemViewDelegate = mItemViewDelegateManager.getItemViewDelegate(viewType);
        int layoutId = itemViewDelegate.getItemViewLayoutId();
        ViewHolder holder = ViewHolder.createViewHolder(mContext, parent, layoutId);
        setListener(parent, holder, viewType);
        return holder;
    }


    protected boolean isEnabled(int viewType) {
        return true;
    }


    protected void setListener(final ViewGroup parent, final ViewHolder viewHolder, int viewType) {
        if (!isEnabled(viewType)) return;
        viewHolder.getConvertView().setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mOnItemClickListener != null) {
                    int position = viewHolder.position;
                    mOnItemClickListener.onItemClick(v, viewHolder, position);
                }
            }
        });

        viewHolder.getConvertView().setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                if (mOnItemClickListener != null) {
                    int position = viewHolder.position;
                    return mOnItemClickListener.onItemLongClick(v, viewHolder, position);
                }
                return false;
            }
        });
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        holder.position = position;
        mItemViewDelegateManager.convert(holder, mDatas.get(position), position);
    }

    @Override
    public int getItemCount() {

        return mDatas==null? 0: mDatas.size();
    }


    public List<T> getDatas() {
        return mDatas;
    }

    public MultiItemTypeAdapter addItemViewDelegate(ItemViewDelegate<T> itemViewDelegate) {
        mItemViewDelegateManager.addDelegate(itemViewDelegate);
        return this;
    }

    public interface OnItemClickListener {
        void onItemClick(View view, RecyclerView.ViewHolder holder, int position);

        boolean onItemLongClick(View view, RecyclerView.ViewHolder holder, int position);
    }

    public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
        this.mOnItemClickListener = onItemClickListener;
    }

    @Override
    public void onAttachedToRecyclerView(RecyclerView recyclerView) {
        super.onAttachedToRecyclerView(recyclerView);
        RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            final GridLayoutManager gridManager = ((GridLayoutManager) layoutManager);
            gridManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                @Override
                public int getSpanSize(int position) {
                    if (position >= mDatas.size()) {
                        return gridManager.getSpanCount();
                    }
                    int type = getItemViewType(position);
                    return isFullSpan(type) ? gridManager.getSpanCount() : 1;

                }
            });
        }
    }

    private boolean isFullSpan(int type) {
        return fullTypes.indexOfKey(type) >= 0;
    }
    public void setFullSpan(int... types) {
        for (int type : types) {
            fullTypes.put(type, type);
        }
    }
}

```

经过了以上的分析MultiItemTypeAdapter的代码就简单易懂了

这里特地说明以下代码：

```java
private SparseArray<Integer> fullTypes = new SparseArray<>();
 private boolean isFullSpan(int type) {
        return fullTypes.indexOfKey(type) >= 0;
    }
    public void setFullSpan(int... types) {
        for (int type : types) {
            fullTypes.put(type, type);
        }
    }
 @Override
    public void onAttachedToRecyclerView(RecyclerView recyclerView) {
        super.onAttachedToRecyclerView(recyclerView);
        RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            final GridLayoutManager gridManager = ((GridLayoutManager) layoutManager);
            gridManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                @Override
                public int getSpanSize(int position) {
                    if (position >= mDatas.size()) {
                        return gridManager.getSpanCount();
                    }
                    int type = getItemViewType(position);
                    return isFullSpan(type) ? gridManager.getSpanCount() : 1;

                }
            });
        }
    }
```

这里保存了一个占据全行type 的集合，在需要设置全行的时候调用 setFullSpan(int ...types)方法，将类型传入

并且在onAttachedToRecyclerView方法的代码中进行了判断调整为全行显示。