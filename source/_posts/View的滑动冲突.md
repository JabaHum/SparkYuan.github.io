title: View的滑动冲突
categories:
- Android
- Android开发艺术探索笔记
tags:
- Android
- View
- 滑动冲突
---
相信开发Android的人都会有这种体会：从网上下载的demo运行的好好的，但是只要出现了滑动冲突，Demo就无法正常工作了。但是不用担心，解决滑动冲突有固定的模式，常见的有内部拦截和外部拦截两种，只要按照这个模式来就可以顺利解决。
<!-- more -->
#介绍
相信开发Android的人都会有这种体会：从网上下载的demo运行的好好的，但是只要出现了滑动冲突，Demo就无法正常工作了。但是不用担心，解决滑动冲突有固定的模式，常见的有内部拦截和外部拦截两种，只要按照这个模式来就可以顺利解决。本文会涉及到View事件分发的相关知识，关于事件分发请参考[http://blog.csdn.net/l664675249/article/details/50738102](http://blog.csdn.net/l664675249/article/details/50738102)

#常见的滑动冲突场景

[图片](http://)

#示例


#处理规则
对于场景1，处理规则为：当用户左右滑动时，让外部的View拦截点击事件，当用户上下滑动时，让内部的View拦截点击事件。当产生滑动时，根据滑动的起始点与终点坐标位置，如果垂直方向滑动距离大，就判断为垂直滑动，否则判断为水平滑动。其他两种情况处理方法相似，**都是从业务需求上得出相应的规则。**

#解决方法
##外部拦截发
所有的点击事件都先经过父容器拦截处理，如果父容器需要拦截就拦截，不需要就传给内部的View。伪代码如下
```java
  public boolean onInterceptTouchEvent(MotionEvent event) {
        boolean intercepted = false;

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
            	intercepted = false;
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                if (满足父容器的拦截要求) {
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                intercepted = false;
                break;
            }
            default:
                break;
        }
        mLastXIntercept = x;
        mLastYIntercept = y;
        return intercepted;
    }
```

**注：**
ACTION_DOWN这个事件是不能拦截的，因为一旦拦截后续的事件都会由父容器处理了。

##内部拦截法
父容器不拦截任何事件，所有事件都传给子元素。如果子元素需要此事件就直接消耗，否则就交给父容器进行处理。完成这个功能需要配合requestDisallowInterceptTouchEvent()方法才可。这个方法表示是否让父容器拦截事件。伪代码如下：

```java
 public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN: {
            parent.requestDisallowInterceptTouchEvent(true);
            break;
        }
        case MotionEvent.ACTION_MOVE: {
            if (满足父容器的拦截要求) {
                parent.requestDisallowInterceptTouchEvent(false);
            }
            break;
        }
        case MotionEvent.ACTION_UP: {
            break;
        }
        default:
            break;
        }
        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(event);
    }
```

**注**
父容器默认拦截除了ACTION_DOWN以外的其他事件，这样子当元素调用parent.requestDisallowInterceptTouchEvent(false)时，父元素才能拦截所需的事件。

#总结

- 解决滑动冲突有两种方法，推荐外部拦截法，实现起来简单。
- 本文以场景1为例做了讲解，场景2,3的做法与1类似，都是根据业务需要制定处理规则。


#示例效果的源码
##Activity
```java
public class DemoActivity_1 extends Activity {
    private static final String TAG = "DemoActivity_1";

    private HorizontalScrollViewEx mListContainer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.demo_1);
        Log.d(TAG, "onCreate");
        initView();
    }

    private void initView() {
        LayoutInflater inflater = getLayoutInflater();
        mListContainer = (HorizontalScrollViewEx) findViewById(R.id.container);
        final int screenWidth = MyUtils.getScreenMetrics(this).widthPixels;
        final int screenHeight = MyUtils.getScreenMetrics(this).heightPixels;
        for (int i = 0; i < 3; i++) {
            ViewGroup layout = (ViewGroup) inflater.inflate(
                    R.layout.content_layout, mListContainer, false);
            layout.getLayoutParams().width = screenWidth;
            TextView textView = (TextView) layout.findViewById(R.id.title);
            textView.setText("page " + (i + 1));
            layout.setBackgroundColor(Color.rgb(255 / (i + 1), 255 / (i + 1), 0));
            createList(layout);
            mListContainer.addView(layout);
        }
    }

    private void createList(ViewGroup layout) {
        ListView listView = (ListView) layout.findViewById(R.id.list);
        ArrayList<String> datas = new ArrayList<String>();
        for (int i = 0; i < 50; i++) {
            datas.add("name " + i);
        }

        ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
                R.layout.content_list_item, R.id.name, datas);
        listView.setAdapter(adapter);
        listView.setOnItemClickListener(new OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view,
                    int position, long id) {
                Toast.makeText(DemoActivity_1.this, "click item",
                        Toast.LENGTH_SHORT).show();

            }
        });
    }
}
```
##水平滑动的View
```java
public class HorizontalScrollViewEx extends ViewGroup {
    private static final String TAG = "HorizontalScrollViewEx";

    private int mChildrenSize;
    private int mChildWidth;
    private int mChildIndex;

    // 分别记录上次滑动的坐标
    private int mLastX = 0;
    private int mLastY = 0;
    // 分别记录上次滑动的坐标(onInterceptTouchEvent)
    private int mLastXIntercept = 0;
    private int mLastYIntercept = 0;

    private Scroller mScroller;
    private VelocityTracker mVelocityTracker;

    public HorizontalScrollViewEx(Context context) {
        super(context);
        init();
    }

    public HorizontalScrollViewEx(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public HorizontalScrollViewEx(Context context, AttributeSet attrs,
                                  int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    private void init() {
        mScroller = new Scroller(getContext());
        mVelocityTracker = VelocityTracker.obtain();
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        boolean intercepted = false;
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                Log.d(TAG, "onInterceptTouchEvent: ACTION_DOWN");
                intercepted = false;
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                    intercepted = true;
                }
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                Log.d(TAG, "onInterceptTouchEvent: ACTION_MOVE");
                int deltaX = x - mLastXIntercept;
                int deltaY = y - mLastYIntercept;
                if (Math.abs(deltaX) > Math.abs(deltaY)) {
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                intercepted = false;
                break;
            }
            default:
                break;
        }

        Log.d(TAG, "intercepted=" + intercepted);
        mLastX = x;
        mLastY = y;
        mLastXIntercept = x;
        mLastYIntercept = y;

        return intercepted;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mVelocityTracker.addMovement(event);
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                Log.d(TAG, "onTouchEvent: ACTION_DOWN");
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                Log.d(TAG, "onTouchEvent: ACTION_MOVE");
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                Log.d(TAG, "onTouchEvent: deltaX" + deltaX);
                scrollBy(-deltaX, 0);
                break;
            }
            case MotionEvent.ACTION_UP: {

                int scrollX = getScrollX();
                int scrollToChildIndex = scrollX / mChildWidth;
                mVelocityTracker.computeCurrentVelocity(1000);
                float xVelocity = mVelocityTracker.getXVelocity();

                //滑的速度到达阈值就认为需要进入下一页
                if (Math.abs(xVelocity) >= 100) {
                    mChildIndex = xVelocity > 0 ? mChildIndex - 1 : mChildIndex + 1;
                } else {
                    //滑动的距离超过一半，就进入下一页
                    mChildIndex = (scrollX + mChildWidth / 2) / mChildWidth;
                }
                //保证在0页和最后一页滑动时不会越界
                mChildIndex = Math.max(0, Math.min(mChildIndex, mChildrenSize - 1));
                //没有达到进入下一页的要求，恢复原样
                int dx = mChildIndex * mChildWidth - scrollX;
                smoothScrollBy(dx, 0);
                Log.d(TAG, "onTouchEvent: dx = " + dx);
                mVelocityTracker.clear();
                break;
            }
            default:
                break;
        }

        mLastX = x;
        mLastY = y;
        return true;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int measuredWidth = 0;
        int measuredHeight = 0;
        final int childCount = getChildCount();
        measureChildren(widthMeasureSpec, heightMeasureSpec);

        int widthSpaceSize = MeasureSpec.getSize(widthMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightSpaceSize = MeasureSpec.getSize(heightMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        if (childCount == 0) {
            setMeasuredDimension(0, 0);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredHeight = childView.getMeasuredHeight();
            setMeasuredDimension(widthSpaceSize, childView.getMeasuredHeight());
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            final View childView = getChildAt(0);
            measuredWidth = childView.getMeasuredWidth() * childCount;
            setMeasuredDimension(measuredWidth, heightSpaceSize);
        } else {
            final View childView = getChildAt(0);
            measuredWidth = childView.getMeasuredWidth() * childCount;
            measuredHeight = childView.getMeasuredHeight();
            setMeasuredDimension(measuredWidth, measuredHeight);
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childLeft = 0;
        final int childCount = getChildCount();
        mChildrenSize = childCount;

        for (int i = 0; i < childCount; i++) {
            final View childView = getChildAt(i);
            if (childView.getVisibility() != View.GONE) {
                final int childWidth = childView.getMeasuredWidth();
                mChildWidth = childWidth;
                childView.layout(childLeft, 0, childLeft + childWidth,
                        childView.getMeasuredHeight());
                childLeft += childWidth;
            }
        }
    }

    private void smoothScrollBy(int dx, int dy) {
        mScroller.startScroll(getScrollX(), 0, dx, 0, 500);
        invalidate();
    }

    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        mVelocityTracker.recycle();
        super.onDetachedFromWindow();
    }
}
```

欢迎转载，转载请注明出处[]()