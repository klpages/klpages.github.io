---
layout:     post
title:      完美解决ScrollView嵌套可滑动EditText滑动冲突的问题 
subtitle:   
date:       2019-12-21
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Armory
---

最近有个项目，ScrollView中嵌套嵌套了一个可以上下滑动的多行EditText,而且EditText滑动到顶部或者顶部时需要将滑动事件交还给ScrollView,
具体效果如下(图片中可能有些卡顿，但是实际非常流畅)：   
<img src="/img/article/EditTextWithScrollView.gif" width="350" height="500"/>    

好了，话不多说，直接上代码：   
```java
public class EditTextWithScrollView extends AppCompatEditText {

    //滑动距离的最大边界
    private int mOffsetHeight;

    //是否滚动到顶或者到底的标志
    private boolean mBoundaryFlag = false;
    private boolean mCanVerticalScroll;

    public EditTextWithScrollView(Context context) {
        super(context);

    }

    public EditTextWithScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);

    }

    public EditTextWithScrollView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        if (canVerticalScroll()) {
            //获得内容面板
            Layout layout = getLayout();
            //获得内容面板的总高度
            int layoutHeight = layout.getHeight();
            //获取上内边距
            int paddingTop = getTotalPaddingTop();
            //获取下内边距
            int paddingBottom = getTotalPaddingBottom();

            //控件实际显示的高度
            int height = getHeight();
            //控件内容总高度与实际显示高度的差值(可以滚动的距离)
            mOffsetHeight = layoutHeight + paddingTop + paddingBottom - height;


           /* //控件内容的总高度
            int scrollRange = getLayout().getHeight();
            //控件实际显示的高度
            int scrollExtent = getHeight() - getCompoundPaddingTop() - getCompoundPaddingBottom();
            //控件内容总高度与实际显示高度的差值
            mOffsetHeight = scrollRange - scrollExtent;*/

        }
    }

   @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_DOWN)
            //如果是新的按下事件，则对mBottomFlag重新初始化
            mBoundaryFlag = false;

        return super.dispatchTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (mCanVerticalScroll) {
            //没有滚动到边界时不允许父布局拦截事件，这个方法会在onScrollChanged方法之后再调用一次
            if (!mBoundaryFlag)
                getParent().requestDisallowInterceptTouchEvent(true);
        } else {
            getParent().requestDisallowInterceptTouchEvent(false);
        }
        return super.onTouchEvent(event);
    }

    @Override
    protected void onScrollChanged(int horiz, int vert, int oldHoriz, int oldVert) {
        super.onScrollChanged(horiz, vert, oldHoriz, oldVert);
        //vert=0表示滑倒了顶部
        if (vert == mOffsetHeight || vert == 0) {
            //滚动到边界后触发父布局或祖父布局的滑动事件
            getParent().requestDisallowInterceptTouchEvent(false);
            mBoundaryFlag = true;
        }
    }

    /**
     * EditText竖直方向是否可以滚动
     *
     * @return true：可以滚动   false：不可以滚动
     */
    private boolean canVerticalScroll() {
        mCanVerticalScroll = getLineCount() > getMaxLines();
        return mCanVerticalScroll;
    }
}
```


如果不能理解上述内容，建议先了解一下事件分发原理：<a href="http://rjgc.cn/2019/12/21/Android事件分发详解/">Android事件分发机制详解</a>

参考链接：     
https://blog.csdn.net/sahadev_/article/details/51207068