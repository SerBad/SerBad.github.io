---
title: RecyclerView
date: 2016-04-20 21:13:42
tags: Android
comments: false
---
最新在项目中使用了Android L出现的新控件RecyclerView，把学习的过程都贴出来，便于日后查询。

![Android L](http://img.blog.csdn.net/20160416104439646)

<!--more-->

RecyclerView提供了四种配置：
> - RecyclerView.LayoutManager
> - RecyclerView.Adapter< VH extends RecyclerView.ViewHolder >
> - RecyclerView.ItemDecoration
> - RecyclerView.ItemAnimator

##  **RecyclerView.LayoutManager**
一共有三种布局，分别是：
> - GridLayoutManager      网格布局
> - LinearLayoutManager   线性布局
> - StaggeredGridLayoutManager     瀑布流布局

##  **RecyclerView.ItemDecoration**
可以绘制特殊的RecyclerView的视图，放一篇参考文章<http://blog.csdn.net/lmj623565791/article/details/45059587>

## **RecyclerView.ItemAnimator**
动画效果，可以指定RecyclerView在添加删除Item时的动画效果。同样，放一篇参考效果
<https://github.com/wasabeef/recyclerview-animators>

## **RecyclerView.Adapter< VH extends RecyclerView.ViewHolder >**
RecyclerView的Adapter相比较listview的adapter，强制把ViewHolder分离了出来，有几个方法需要重写：
```java
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder> {
    private Context context;
    private List<String> list;

    public RankRecyclerViewAdpter(Context context, List<String> list) {
        this.context = context;
        this.list = list;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_layout, parent, false);
        return new ViewHolder(v);
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, int position) {
        viewHolder.textView.setText(list.get(position));
    }

    @Override
    public int getItemCount() {
        return list.size();
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        public TextView textView;
        public ViewHolder(View View) {
            super(view);
            textView= (TextView) itemView.findViewById(R.id.textView);
        }
    }
}
```

有几个问题需要注意：
> - layout中放的应该是`android.support.v7.widget.RecyclerView` 不然会报错。
> - `getItemCount()` 的返回值不能为空，不然不会执行`onBindViewHolder`
> - RecyclerView必须指定LayoutManger，不然同样会报错。

## **RecyclerView的点击事件**
官方文档中并没有给我们类似ListView的`OnItemClickListener`回调方法，由于RecyclerView比ListView更高级，所以它并没有行或者列的概念，子View可以任意布局，每个子View处理自己的onClick事件，也就是说在Adapter中给子view的rootview设置点击回调。我们今天所要实现的是另外一种方式，类似ListView的`OnItemClickListener`的方式。通过文档我们知道RecyclerView留给开发者一个`RecyclerView.OnItemTouchListener`接口，我们要做的就是实现它，实现点击的回调和长按回调。当然了，这种方式只是一个开始，我们还可以拓展为各种复杂的手势操作的回调。
参考自：<http://blog.csdn.net/hlglinglong/article/details/44981889>
```java
public class RecyclerItemClickListener implements RecyclerView.OnItemTouchListener{  
    private View childView;  
    private RecyclerView touchView;  
    public RecyclerItemClickListener(Context context, final OnItemClickListener mListener) {  
        mGestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener(){  
            @Override  
            public boolean onSingleTapUp(MotionEvent ev) {  
                if (childView != null && mListener != null) {  
                    mListener.onItemClick(childView, touchView.getChildPosition(childView));  
                }  
                return true;  
            }  
            @Override  
            public void onLongPress(MotionEvent ev) {  
                if (childView != null && mListener != null) {  
                    mListener.onLongClick(childView, touchView.getChildPosition(childView));  
                }  
            }  
        });  
    }  

    GestureDetector mGestureDetector;  

    public interface OnItemClickListener {  
        public void onItemClick(View view, int position);  
        public void onLongClick(View view, int posotion);  
    }  

    @Override  
    public boolean onInterceptTouchEvent(RecyclerView recyclerView, MotionEvent motionEvent) {  
        mGestureDetector.onTouchEvent(motionEvent);  
        childView = recyclerView.findChildViewUnder(motionEvent.getX(), motionEvent.getY());  
        touchView = recyclerView;  
        return false;  
    }  

    @Override  
    public void onTouchEvent(RecyclerView recyclerView, MotionEvent motionEvent) {  

    }  
}
```

## **类似listview的`addHeaderView`和`addFooterView`**

RecyclerView同样没有提供类似listview的`addHeaderView`和`addFooterView` 方法，不过我们可以自定义，参考文章：
<https://github.com/dingdingyr/HeaderAndFooterRecyclerView->
原理就是：
```java
		GridLayoutManager manager = new GridLayoutManager(this, 2);
        manager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                return adapter.getItemViewType()? manager.getSpanCount() : 1;
            }
        });
```
GirdLayout可以设置显示的行数，我们可以根据我们要显示的Item类型来指定这个值。

**还有一个问题，那就是在更新apdater的时候不推荐使用`notifyDataSetChanged()` ，因为`notifyDataSetChanged()` 会强制刷新整个视图，如果插入新的数据可以使用 `public final void notifyItemRangeInserted(int positionStart, int itemCount);`。**
