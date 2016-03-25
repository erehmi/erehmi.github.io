---
date: "2015-01-16 21:15:00"
description: ""
layout: post
permalink: ancase-login-and-register
categories:
  - blog
  - android
  - case
title: "[Deprecated!] Android开发案例 - 微博正文"
---

**Deprecated!**
**更好的实现方式: 使用 android.support.design.widget.CoordinatorLayout.**


 本文详细介绍如何实现如下图中的[微博](<http://m.weibo.com/web/cellphone.php#android>)正文页面效果,
其中包括:

\>
实现页面滚动时[转发-评论-赞]工具条的[Sticky悬停效果](<https://github.com/emilsjolander/StickyListHeaders>)

\> 实现工具条切换,
并解决[转发-评论-赞]子页面item不足以占满屏幕时切换页面导致屏幕滚动的问题

![](<http://erehmi.github.io/assets/image/weibo-text-detail.png>)

**知识要点:**
1.  ListView
2.  ListView \# HeaderView & FooterView
3.  AbsListView.onScrollListener
4.  ListAdapter

**实现代码:**
**\> 定义**
-   DetailActivity - 正文界面
-   MiddleTab - 正文界面中的工具条

**\> 界面需求-1**
首先实现的是Sticky悬停效果, 基本思路是:
1.  设计正文界面的Layout-XML布局, 并以ListView来展示转发/评论/赞列表的详情.
2.  设计MiddleTab的Layout-XML布局, 以android:visibility="gone"方式添加到上一个Layout-XML中, 这里的MiddleTab, 我们暂且叫它为Main-MiddleTab
3.  设计另外一个布局, 包含正文布局和MiddleTab-Layout-XML, 并把它作为ListView的HeaderView, 我们暂且叫它为HeaderView-MiddlerTab
4.  当ListView\#HeaderView中的MiddleTab滚出屏幕顶部时, 显示Main-MiddleTab

进入正题前, 先介绍以下几个类和变量:
-   [类] ScrollDetector - ListView滚动的辅助类, 它用来监听第一个Item滚动事件以及最后一个Item显示事件, 代码如下.
-   [类] HdrViewHolder - 用保存HeaderView的Holder容器
-   [变量] HdrViewHolder.middleTabs - HeaderView中的工具条
-   [变量] mMiddleTabs - Main-MiddleTab
-   [变量] mListView - 用来展示正文及详情的ListView
-   [变量] mListAdapter - ListItem适配器, 在这里, 我们假设它为DetailsAdapter类型(支持ArrayAdapter的操作), 并假定它能完美支持转发/评论/赞列表(本文中将不实现DetailsAdapter代码, 因为它的代码实现和常规Adapter基本相同).
-   [变量] R.id.\*Tab - MiddleTab中转发/评论/赞所对应的RadioButton-id

> ScrollDetector.java
{% highlight java %}
import ...

/**
 * Detect scroll events of list or grid.
 */
public class ScrollDetector implements OnScrollListener {
    /** @see #onScroll(android.widget.AbsListView, int, int, int) */
    private boolean mFirstItemVisible = false;
    private OnFirstItemScrollListener mFisListener;
    private OnLastItemVisibleListener mLivListener;

    public ScrollDetector(OnFirstItemScrollListener fisListener,
            OnLastItemVisibleListener livListener) {
        mFisListener = fisListener;
        mLivListener = livListener;
    }

    public void setOnFirstItemScrollListener(OnFirstItemScrollListener listener) {
        mFisListener = listener;
    }

    public void setOnLastItemVisibleListener(OnLastItemVisibleListener listener) {
        mLivListener = listener;
    }

    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        if (mLivListener != null) {
            if (triggerLastItemVisible(view, scrollState)) {
                mLivListener.onLastItemVisible();
            }
        }
    }

    private boolean triggerLastItemVisible(AbsListView view, int scrollState) {
        return (scrollState == SCROLL_STATE_IDLE &&
                (view.getLastVisiblePosition() == view.getCount() - 1));
    }

    /**
     * 用超高的初速度滚动AbsListView时, 可能会出现跳过firstVisibleItem=0的情况, 因此,
     * 通过设置mFirstItemVisible来避免在出现上述情况时不会调用onFirstItemScroll的问题
     *
     * @see OnScrollListener#onScroll(android.widget.AbsListView, int, int, int)
     */
    @Override
    public void onScroll(AbsListView view, int firstVisibleItem,
            int visibleItemCount, int totalItemCount) {
        if (mFisListener != null) {
            if (triggerFirstItemScroll(view, firstVisibleItem) || mFirstItemVisible) {
                mFisListener.onFirstItemScroll(view.getChildAt(0));
                if (!mFirstItemVisible) {
                    mFirstItemVisible = true;
                }
            } else {
                if (mFirstItemVisible) {
                    mFirstItemVisible = false;
                }
            }
        }
    }

    private boolean triggerFirstItemScroll(AbsListView view, int firstVisibleItem) {
        return (firstVisibleItem == 0);
    }

    public static interface OnLastItemVisibleListener {
        void onLastItemVisible();
    }

    public static interface OnFirstItemScrollListener {
        void onFirstItemScroll(View itemView);
    }
}
{% endhighlight %}

　　Layout-XML代码略. 需要说明的是, 工具条是以RadioGroup方式实现的. 以下为DetailActivity.java代码:

{% highlight java %}
import ...

public class DetailActivity extends Activity implements onClickListener, OnCheckedChangeListener, OnFirstItemScrollListener {
    ...
    private ListView mListView;
    private DetailsAdapter mListAdapter;
    private RadioGroup mMiddleTabs;
    private HdrViewHolder mHdrViewHolder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);
        ...
        
        //initMiddleTabsStates();    // 暂时不用理会这行代码, 界面需求-2 会使用到它
        initContent();
        initDetails();    
    }

　　private void initContent() {
        final LayoutInflater inflater = getLayoutInflater();
        View hdrView = inflater.inflate(R.layout.item_detail_header, mListView, false);
        mHdrViewHolder = new HdrViewHolder(hdrView);
        mListView.addHeaderView(hdrView);
        
        ...
        mHdrViewHolder.middleTabs.check(R.id.commentTab);
    }

    private void initDetails() {
        mScrollDetector = new ScrollDetector(this, this);
        mListView.setOnScrollListener(mScrollDetector);
        
        mListAdapter = new DetailsAdapter(this);
        mListView.setAdapter(wrapperAdapter);

        mMiddleTabs.check(R.id.commentTab);
    }
    
    @Override
    public void onClick(View v) {
        int id = buttonView.getId();
        switch(id) {
        case R.id.forwardTab:
        case R.id.commentTab:
        case R.id.praiseTab:
            if (!checked) {// 切换TAB前保存当前CommentTab的状态
                updateMiddleTabs(id);
            }
            break;
        }
    }
    
    private void updateMiddleTabs(int id) {
        if (mMiddleTabs.getCheckedRadioButtonId() == id) {
            mHdrViewHolder.middleTabs.check(id);
            //restoreMiddleTabsStates();// 暂时不用理会这行代码, 界面需求-2 会使用到它
        }
    }
    
    @Override
    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
        int id = buttonView.getId();
        switch(id) {
        case R.id.forwardTab:
        case R.id.commentTab:
        case R.id.praiseTab:
            if (!checked) {// 切换TAB前保存当前CommentTab的状态
                //saveMiddleTabsStates(id);    // 暂时不用理会这行代码, 界面需求-2 会使用到它
            }
            break;
        }
    }
    
    @Override
    public void onFirstItemScroll(View itemView) {
        int[] location = new int[2];
        int[] location2 = new int[2];
        mHdrViewHolder.middleTabs.getLocationOnScreen(location);
        mMiddleTabs.getLocationOnScreen(location2);

        boolean visible = (location[1] <= location2[1]);
        mMiddleTabs.setVisibility(visible ? View.VISIBLE : View.GONE);
    }

    static class HdrViewHolder implements onClickListener {
        ...

        @InjectView(R.id.tabs)
        RadioGroup middleTabs;

        HdrViewHolder(View view) {
            ...
            middleTabs.setOnClickListener(this);
        }

        @Override
        public void onClick(View v) {
            int id = buttonView.getId();
            switch(id) {
            case R.id.forwardTab:
            case R.id.commentTab:
            case R.id.praiseTab:
                if (!checked) {// 切换TAB前保存当前CommentTab的状态
                    updateMiddleTabs(id);
                }
                break;
            }
        }
        
        private void updateMiddleTabs(int id) {
            if (middleTabs.getCheckedRadioButtonId() == id) {
                mMiddleTabs.check(id);
                //restoreMiddleTabsStates();// 暂时不用理会这行代码, 界面需求-2 会使用到它
            }
        }
    }
}
{% endhighlight %}

　　在上述代码中, 首先用ScrollDetector实现了Sticky悬停效果, 然后就是同步Main-MiddleTab和HeaderView-MiddlerTab的checked状态. 接下来, 再看如何实现界面需求-2. 

**\> 界面需求-2**

　　进入正题前, 我们还得介绍一个辅助类PlaceholderListAdapter, 它虽然有点像android系统的HeaderViewListAdapter, 但它却是我们用来应付Item未能占满ListView的情况的辅助类. 假想下当所有数据都加载到ListView的情况, 如果第一个数据项已经不显示在ListView上, 那么这时我们可以认为ListView已经被Item占满了, 否则, 就需要非数据项视图或者FooterView来占满空余的ListView. 而PlaceholderListAdapter的原理正是这样, 我们先预置一个类似FooterView的View给PlaceholderListAdapter, 并且在getView()时检测ListView是否已经需要显示该View了, 如果是, 则按上述逻辑来处理. 代码如下:
> PlaceholderListAdapter.java
{% highlight java %}
import ...

/**
 * PlaceholderListAdapter可以帮助我们解决这样的问题:<br> <ul><li>当所有Item视图不足以占满ListView时,
 * 用空白视图来填充空白区域.</li></ul><br> 效果图见微博Android客户端的微博正文页面. 该适配器主要是用来提升用户体验的,
 * 尤其是在切换TAB时.
 *
 * @see android.widget.HeaderViewListAdapter
 */
public class PlaceholderListAdapter implements WrapperListAdapter {
    private final ListAdapter mAdapter;

    public class FixedViewInfo {
        public View view;
        public Object data;
        public boolean isSelectable;
    }

    private ArrayList<FixedViewInfo> mFooterViewInfos;
    private View mPinnedHeaderView;

    public PlaceholderListAdapter(Context context, ListAdapter adapter) {
        mAdapter = adapter;
        mFooterViewInfos = new ArrayList<FixedViewInfo>();
        init(context);
    }

    private void init(Context context) {
        View placeholder = new View(context);
        placeholder.setLayoutParams(
                new AbsListView.LayoutParams(
                        LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT));
        addFooterView(placeholder, true);
    }

    public void setPinnedHeaderView(View view) {
        mPinnedHeaderView = view;
    }

    public void addFooterView(View view) {
        addFooterView(view, false);
    }

    private void addFooterView(View view, boolean isPlaceholder) {
        FixedViewInfo info = new FixedViewInfo();
        info.view = view;
        info.data = null;
        info.isSelectable = true;
        if (isPlaceholder) {
            mFooterViewInfos.add(info);
        } else {
            mFooterViewInfos.add(mFooterViewInfos.size() - 1, info);
        }
    }

    public int getPlaceholdersCount() {
        return mFooterViewInfos.size();
    }

    @Override
    public boolean hasStableIds() {
        return (mAdapter != null) ? mAdapter.hasStableIds() : false;
    }

    @Override
    public boolean isEmpty() {
        return mAdapter == null || mAdapter.isEmpty();
    }

    @Override
    public int getCount() {
        int adapterCount = (mAdapter != null) ? mAdapter.getCount() : 0;
        return getPlaceholdersCount() + adapterCount;
    }

    @Override
    public boolean areAllItemsEnabled() {
        if (mAdapter != null) {
            return mAdapter.areAllItemsEnabled();
        } else {
            return true;
        }
    }

    @Override
    public boolean isEnabled(int position) {
        if (mAdapter != null && position < mAdapter.getCount()) {
            return mAdapter.isEnabled(position);
        }
        return false;
    }

    @Override
    public Object getItem(int position) {
        int adapterCount = 0;
        if (mAdapter != null) {
            adapterCount = mAdapter.getCount();
            if (position < adapterCount) {
                return mAdapter.getItem(position);
            }
        }

        return mFooterViewInfos.get(position - adapterCount).data;
    }

    @Override
    public long getItemId(int position) {
        if (mAdapter != null && position < mAdapter.getCount()) {
            return mAdapter.getItemId(position);
        }

        return -1;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        int adapterCount = 0;
        if (mAdapter != null) {
            adapterCount = mAdapter.getCount();
            if (position < adapterCount) {
                return mAdapter.getView(position, convertView, parent);
            }
        }

        View view = mFooterViewInfos.get(position - adapterCount).view;
        if (position == getCount() - 1) {// 当convertView为占位View时
            if (!(parent instanceof ListView)) {
                throw new IllegalArgumentException("the parent is not a ListView.");
            }

            ListView listView = (ListView) parent;
            int startPosition = listView.getHeaderViewsCount();
            int itemsHeight = (mPinnedHeaderView != null) ? mPinnedHeaderView.getHeight() : 0;
            int firstVisiblePos = listView.getFirstVisiblePosition();
            int lastVisiblePos = listView.getLastVisiblePosition();
            if (startPosition >= firstVisiblePos) {// 第一个数据视图还在屏幕上, 此时需要占位视图
                for (int i = startPosition; i <= lastVisiblePos; ++i) {
                    View childView = listView.getChildAt(i - firstVisiblePos);
                    itemsHeight += childView.getHeight();
                }
            } else {// 第一个数据视图已经滚出屏幕, 此时不需要显示占位视图
                itemsHeight = listView.getHeight();
            }

            ViewGroup.LayoutParams params = view.getLayoutParams();
            if (params == null) {
                throw new IllegalArgumentException("the layout parameters is not set.");
            }

            params.height = listView.getHeight() - itemsHeight;
            //view.setLayoutParams(params);
        }

        return view;
    }

    @Override
    public int getItemViewType(int position) {
        if (mAdapter != null && position < mAdapter.getCount()) {
            return mAdapter.getItemViewType(position);
        }
        return AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER;
    }

    @Override
    public int getViewTypeCount() {
        return (mAdapter != null) ? mAdapter.getViewTypeCount() : 1;
    }

    @Override
    public void registerDataSetObserver(DataSetObserver observer) {
        if (mAdapter != null) {
            mAdapter.registerDataSetObserver(observer);
        }
    }

    @Override
    public void unregisterDataSetObserver(DataSetObserver observer) {
        if (mAdapter != null) {
            mAdapter.unregisterDataSetObserver(observer);
        }
    }

    @Override
    public ListAdapter getWrappedAdapter() {
        return mAdapter;
    }
}
{% endhighlight %}

　　既然有了PlaceholderListAdapter, 那后面就是很简单的事情了,
就只剩下在切换MiddleTab时保存ListView的滚动位置的问题了, 到这里,
就可以用到上面那些默默占位, 却没被理会的代码了. 代码如下: 

{% highlight java %}
　　private static class SavedState {
        private int viewTop;
        private int position;
        private int loadState;
        private List<Object> objects;

        private SavedState() {
            position = -1;
            viewTop = 0;
            objects = new ArrayList<Object>();
            loadState = LOAD_STATE_IDLE;
        }
    }

　　private SparseArray<SavedState> mSavedStates;

   private void initMiddleTabsStates() {
　　　　mSavedStates = new SparseArray<SavedState>();
　　　　int[] ids = {R.id.commentTab, R.id.praiseTab};
　　　　for (int id : ids) {
　　　　　　mSavedStates.put(id, new SavedState());
　　　　}
　　}

　　private void initDetails() {
        mScrollDetector = new ScrollDetector(this, this);
        mListView.setOnScrollListener(mScrollDetector);
        
        mListAdapter = new DetailsAdapter(this);
        //mListView.setAdapter(wrapperAdapter);　　// 这行代码只能满足界面需求-1

        PlaceholderListAdapter wrapperAdapter = new PlaceholderListAdapter(this, mListAdapter);
        wrapperAdapter.setPinnedHeaderView(mHdrViewHolder.middleTabs);

        View footerView = getLayoutInflater().inflate(R.layout.load_more, mListView, false);
        wrapperAdapter.addFooterView(footerView);

        //wrapperAdapter.addPlaceholder(mPlaceholder);
        mListView.setAdapter(wrapperAdapter);
        
        mMiddleTabs.check(R.id.commentTab);
    }

　　/**
     * 还原对应ID的TAB的状态
     *
     * @see #saveMiddleTabsStates(int)
     */
    private void restoreMiddleTabsStates() {
        int id = mMiddleTabs.getCheckedRadioButtonId();
        SavedState state = mSavedStates.get(id);

        mListAdapter.setNotifyOnChange(false);
        mListAdapter.clear();
        mListAdapter.addAll(state.objects);
        mListAdapter.notifyDataSetChanged();

        if (mMiddleTabs.getVisibility() == View.VISIBLE) {
            if (state.position == -1) {
                setPinnedSavedState(state);
            }

            mListView.setSelectionFromTop(state.position, state.viewTop);
        }
    }

　　/**
     * 保存对应ID的TAB的状态, 并在切换回来之后, 还原该TAB的状态
     *
     * @see #restoreMiddleTabsStates()
     */
    private void saveMiddleTabsStates(int id) {
        SavedState state = mSavedStates.get(id);
        if (mMiddleTabs.getVisibility() == View.VISIBLE) {
            View child = mListView.getChildAt(0);
            int viewTop = ((child != null) ? (child.getTop() - mListView.getPaddingTop()) : 0);
            state.position = mListView.getFirstVisiblePosition();
            state.viewTop = viewTop;
        } else {
            setPinnedSavedState(state);
        }
    }
    
    private void setPinnedSavedState(SavedState state) {
        state.position = mListView.getHeaderViewsCount();
        state.viewTop = mMiddleTabs.getHeight() - mListView.getPaddingTop();
    }
{% endhighlight %}

 

　　到这里, 我们已经实现了微博正文界面.
需要说明的是使用PlaceholderListAdapter之后,
建议用PlaceholderListAdapter.addFooterView(View
view)来替代ListView.addFooterView(View view)调用,
否则会在ListView的Item与FooterView之间出现空白区域,
那正是我们用来占满空余ListView的视图.
 

**END. \>\> SEE MORE:**
[http://erehmi.github.io/](<**http://erehmi.github.io/**>)
