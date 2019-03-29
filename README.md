# Android Load more on scroll

1. InfiniteScrollPorvider.java

```java

import android.content.*;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.StaggeredGridLayoutManager;
import android.util.Log;
import android.widget.*;

import static android.support.v4.view.ViewPager.SCROLL_STATE_DRAGGING;
import static android.widget.NumberPicker.OnScrollListener.SCROLL_STATE_IDLE;


public class InfiniteScrollProvider
{
    Context context;

    private RecyclerView recyclerView;
    private boolean isLoading = false;
    private OnLoadMoreListener onLoadMoreListener;
    private ScrollPosChanged onScrollPosChanged;
    private RecyclerView.LayoutManager layoutManager;
    private int lastVisibleItem;
    private int totalItemCount;
    private int previousItemCount = 0;
    private static final int THRESHOLD = 3;

    private int scrollStart = 0;


    public void attach(RecyclerView recyclerView, OnLoadMoreListener onLoadMoreListener) {
        this.recyclerView = recyclerView;
        this.onLoadMoreListener = onLoadMoreListener;
        layoutManager =recyclerView.getLayoutManager();
        setInfiniteScrollGrid(layoutManager);
    }


    public void attach(RecyclerView recyclerView, ScrollPosChanged ScrollPosChanged) {
        this.recyclerView = recyclerView;
        this.onScrollPosChanged = ScrollPosChanged;
        layoutManager =recyclerView.getLayoutManager();
        setInfiniteScrollGrid(layoutManager);
    }

    private void setInfiniteScrollGrid(final RecyclerView.LayoutManager layoutManager) {
        recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy)
            {


//                if(scrollStart <= 50){
//                    scrollStart++;
//                    onScrollPosChanged.ScrollPosChanged(0);
//                }

                onScrollPosChanged.ScrollPosChanged(0,false);

                totalItemCount = layoutManager.getItemCount();

                if (previousItemCount > totalItemCount)
                {
                    previousItemCount = totalItemCount - THRESHOLD;
                }
                if (layoutManager instanceof GridLayoutManager)
                {
                    lastVisibleItem = ((GridLayoutManager)layoutManager).findLastVisibleItemPosition();
                }else if (layoutManager instanceof LinearLayoutManager)
                {
                    lastVisibleItem = ((LinearLayoutManager)layoutManager).findLastVisibleItemPosition();
                }else if (layoutManager instanceof StaggeredGridLayoutManager)
                {
                    StaggeredGridLayoutManager staggeredGridLayoutManager=(StaggeredGridLayoutManager) layoutManager;
                    int spanCount=staggeredGridLayoutManager.getSpanCount();
                    int[] ids=new int[spanCount];
                    staggeredGridLayoutManager.findLastVisibleItemPositions(ids);
                    int max=ids[0];
                    for (int i = 1; i < ids.length; i++) {
                        if (ids[1]>max){
                            max=ids[1];
                        }
                    }
                    lastVisibleItem=max;
                }
                if (totalItemCount > THRESHOLD) {
                    if (previousItemCount <= totalItemCount && isLoading) {
                        isLoading = false;
                        Log.i("InfiniteScroll", "Data fetched");
                    }
                    if (!isLoading && (lastVisibleItem > (totalItemCount - THRESHOLD)) && totalItemCount > previousItemCount) {
                        if (onLoadMoreListener != null) {
                            onLoadMoreListener.onLoadMore();
                        }
                        Log.i("InfiniteScroll", "End Of List");
                        isLoading = true;
                        previousItemCount = totalItemCount;
                    }
                }
                super.onScrolled(recyclerView, dx, dy);


               // super.onScrollStateChanged(recyclerView, );
            }


            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);

                 if(newState == SCROLL_STATE_IDLE){
                     scrollStart = 0;
                     onScrollPosChanged.ScrollPosChanged(0, true);
                 }

            }
        });
    }

}
```

2. OnLoadMoreListener.java

```java

public interface OnLoadMoreListener {
    void onLoadMore();

}
```

3. ScrollPosChanged.java

```java
public interface ScrollPosChanged
{
        void ScrollPosChanged(int pos,boolean show);

}
```

4. RecyclerViewPositionHelper.java

```java

import android.support.v7.widget.*;
import android.view.*;

public class RecyclerViewPositionHelper {
    final RecyclerView recyclerView;
    final RecyclerView.LayoutManager layoutManager;

    RecyclerViewPositionHelper(RecyclerView recyclerView) {
        this.recyclerView = recyclerView;
        this.layoutManager = recyclerView.getLayoutManager();
    }

    public static RecyclerViewPositionHelper createHelper(RecyclerView recyclerView) {
        if (recyclerView == null) {
            throw new NullPointerException("Recycler View is null");
        }
        return new RecyclerViewPositionHelper(recyclerView);
    }

    /**
     * Returns the adapter item count.
     *
     * @return The total number on items in a layout manager
     */
    public int getItemCount() {
        return layoutManager == null ? 0 : layoutManager.getItemCount();
    }

    /**
     * Returns the adapter position of the first visible view. This position does not include
     * adapter changes that were dispatched after the last layout pass.
     *
     * @return The adapter position of the first visible item or {@link RecyclerView#NO_POSITION} if
     * there aren't any visible items.
     */
    public int findFirstVisibleItemPosition() {
        final View child = findOneVisibleChild(0, layoutManager.getChildCount(), false, true);
        return child == null ? RecyclerView.NO_POSITION : recyclerView.getChildAdapterPosition(child);
    }

    /**
     * Returns the adapter position of the first fully visible view. This position does not include
     * adapter changes that were dispatched after the last layout pass.
     *
     * @return The adapter position of the first fully visible item or
     * {@link RecyclerView#NO_POSITION} if there aren't any visible items.
     */
    public int findFirstCompletelyVisibleItemPosition() {
        final View child = findOneVisibleChild(0, layoutManager.getChildCount(), true, false);
        return child == null ? RecyclerView.NO_POSITION : recyclerView.getChildAdapterPosition(child);
    }

    /**
     * Returns the adapter position of the last visible view. This position does not include
     * adapter changes that were dispatched after the last layout pass.
     *
     * @return The adapter position of the last visible view or {@link RecyclerView#NO_POSITION} if
     * there aren't any visible items
     */
    public int findLastVisibleItemPosition() {
        final View child = findOneVisibleChild(layoutManager.getChildCount() - 1, -1, false, true);
        return child == null ? RecyclerView.NO_POSITION : recyclerView.getChildAdapterPosition(child);
    }

    /**
     * Returns the adapter position of the last fully visible view. This position does not include
     * adapter changes that were dispatched after the last layout pass.
     *
     * @return The adapter position of the last fully visible view or
     * {@link RecyclerView#NO_POSITION} if there aren't any visible items.
     */
    public int findLastCompletelyVisibleItemPosition() {
        final View child = findOneVisibleChild(layoutManager.getChildCount() - 1, -1, true, false);
        return child == null ? RecyclerView.NO_POSITION : recyclerView.getChildAdapterPosition(child);
    }

    View findOneVisibleChild(int fromIndex,int toIndex,boolean completelyVisible,
                             boolean acceptPartiallyVisible) {
        OrientationHelper helper;
        if (layoutManager.canScrollVertically()) {
            helper = OrientationHelper.createVerticalHelper(layoutManager);
        } else {
            helper = OrientationHelper.createHorizontalHelper(layoutManager);
        }

        final int start = helper.getStartAfterPadding();
        final int end = helper.getEndAfterPadding();
        final int next = toIndex > fromIndex ? 1 : -1;
        View partiallyVisible = null;
        for (int i = fromIndex; i != toIndex; i += next) {
            final View child = layoutManager.getChildAt(i);
            final int childStart = helper.getDecoratedStart(child);
            final int childEnd = helper.getDecoratedEnd(child);
            if (childStart < end && childEnd > start) {
                if (completelyVisible) {
                    if (childStart >= start && childEnd <= end) {
                        return child;
                    } else if (acceptPartiallyVisible && partiallyVisible == null) {
                        partiallyVisible = child;
                    }
                } else {
                    return child;
                }
            }
        }
        return partiallyVisible;
    }
}

```

5. MainActivity.java

```java

 InfiniteScrollProvider infiniteScrollProvider = new InfiniteScrollProvider();
        infiniteScrollProvider.attach(book_recyclerView,new OnLoadMoreListener() {
            @Override
            public void onLoadMore()
            {
               // Do something when scroll comes to an end;
            }
        });

        infiniteScrollProvider.attach(book_recyclerView,new ScrollPosChanged() {
            @Override
            public void ScrollPosChanged(int pos, boolean show)
            {
                int position = mRecyclerViewHelper.findFirstVisibleItemPosition();
//                Log.i("InfiniteScroll", String.valueOf(position));
                if(position <= 1) {
                    if(show == true) {
                        // Do something when true
                    }
                }
                else{
                    if(show == false) {
                        // Do something when false

                    }
                }
            }
        });

```
