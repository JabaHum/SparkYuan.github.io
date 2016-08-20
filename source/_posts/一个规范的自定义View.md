title: 一个规范的自定义View
date: 2016/3/11 20:46:25
categories:
- Android
- Android开发艺术探索笔记
tags:
- Android
- View
- 自定义View
---
一个规范的自定义View
<!-- more -->

# 一个不规范的自定义View

这个自定义的View很简单，就是画一个圆，实现一个圆形效果的自定义View。

先看一个不规范的自定义View是怎么做的

```java
public class CircleView extends View {

    private int mColor = Color.RED;
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    public CircleView(Context context) {
        super(context);
        init();
    }

    public CircleView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mPaint.setColor(mColor);
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        int width = getWidth();
        int height = getHeight();
        int radius = Math.min(width, height) / 2;
        canvas.drawCircle(width / 2, height / 2, radius, mPaint);
    }
}

```

对应的xml

```xml
<com.ryg.chapter_4.ui.CircleView
    android:id="@+id/circleView1"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:layout_margin="20dp"
    android:background="#000000"
    />
```

这样虽然也能画出一个圆来，但是这并不是一个规范的自定义View，主要存在以下问题：

- android:padding属性是不能使用的
- 使用wrap_content就相当于使用match_partent

# 一个规范的自定义View
为了解决以上问题需要重写View的onMeasure和onDraw方法。

完整代码如下：

```java
public class CircleView extends View {

    private int mColor = Color.RED;
    private Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    public CircleView(Context context) {
        super(context);
        init();
    }

    public CircleView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        mColor = a.getColor(R.styleable.CircleView_circle_color, Color.RED);
        a.recycle();
        init();
    }

    private void init() {
        mPaint.setColor(mColor);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
        if (widthSpecMode == MeasureSpec.AT_MOST
                && heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, 200);
        } else if (widthSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(200, heightSpecSize);
        } else if (heightSpecMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSpecSize, 200);
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        final int paddingLeft = getPaddingLeft();
        final int paddingRight = getPaddingRight();
        final int paddingTop = getPaddingTop();
        final int paddingBottom = getPaddingBottom();
        int width = getWidth() - paddingLeft - paddingRight;
        int height = getHeight() - paddingTop - paddingBottom;
        int radius = Math.min(width, height) / 2;
        canvas.drawCircle(paddingLeft + width / 2, paddingTop + height / 2,
                radius, mPaint);
    }
}
```

# 添加自定义属性
1. 在values文件夹下添加attrs.xml 
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleView">
        <attr name="circle_color" format="color" />
    </declare-styleable>
</resources>
```
自定义的属性集合CircleView，在这个属性集合里只定义了一个格式为color的属性circle_color。
2. 在View的构造函数中解析自定义的属性
```java
 public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        mColor = a.getColor(R.styleable.CircleView_circle_color, Color.RED);
        a.recycle();
        init();
    }
```
3. 在布局文件中使用自定义属性
```xml
   <com.ryg.chapter_4.ui.CircleView
        android:id="@+id/circleView1"
        android:layout_width="wrap_content"
        android:layout_height="100dp"
        android:layout_margin="20dp"
        android:background="#000000"
        android:padding="20dp"
        app:circle_color="@color/light_green" />
```
在使用自定义的属性时，要在schemas声明：xmlns:app="http://schemas.android.com/apk/res-auto"，使用时与普通属性类似，app:circle_color="@color/light_green" 。

# 自定义View须知
- 自定义的View中margin属性可以使用，因为它是由父容器控制的
- 直接继承View或ViewGroup的需要自己处理wrap_content
- View要在onDraw方法中要处理padding，而ViewGroup要在onMeasure和onLayout中处理padding和margin
- View中的post方法可以取代handler
- 在View的onDetachedFromWindow中停止动画，防止内存泄露
- 有滑动嵌套情形时，注意滑动冲突处理
- 关于上面涉及到的一些类和方法的详细解释请参考[http://blog.csdn.net/l664675249/article/details/50774617](http://blog.csdn.net/l664675249/article/details/50774617)

想要自定义出漂亮的View并不容易，只有多读，多写，多测，才能更好的掌握。自己造一个轮子，然后再对比成熟的轮子去找差距和不足。

