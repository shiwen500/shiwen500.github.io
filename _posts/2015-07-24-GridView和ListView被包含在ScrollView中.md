---
layout: post
title:  "GridView 和 ListView 被包含在 ScrollView 中"
date:   2015-07-24 14:06:05
categories: Android
excerpt: ScrollView GridView ListView
---

## 问题

很多时候，我们需要在`ScrollView`中包含一个`ListView`或者`GridView`。

现在，公司的项目遇到这种需求，但是我发现直接在`ScrollView`中包含一个

`ListView`或者`GridView`，`ListView`的内容并没有像预想那样全部显示，

而是只显示部分内容。

## 解决方法

重写ListView或者GridView

这是[stackoverflow](http://stackoverflow.com/questions/8481844/gridview-height-gets-cut)上得一个解决方法：

{%  highlight java %}

public class ExpandableHeightGridView extends GridView
{

    boolean expanded = false;

    public ExpandableHeightGridView(Context context)
    {
        super(context);
    }

    public ExpandableHeightGridView(Context context, AttributeSet attrs)
    {
        super(context, attrs);
    }

    public ExpandableHeightGridView(Context context, AttributeSet attrs,
            int defStyle)
    {
        super(context, attrs, defStyle);
    }

    public boolean isExpanded()
    {
        return expanded;
    }

    @Override
    public void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
    {
        if (isExpanded())
        {
            
            // AT_MOST 模式会让GridView计算出所有child出现时的高度
            int expandSpec = MeasureSpec.makeMeasureSpec(MEASURED_SIZE_MASK,
                    MeasureSpec.AT_MOST);
            super.onMeasure(widthMeasureSpec, expandSpec);

            ViewGroup.LayoutParams params = getLayoutParams();
            params.height = getMeasuredHeight();
        }
        else
        {
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        }
    }

    public void setExpanded(boolean expanded)
    {
        this.expanded = expanded;
    }
}

{%  endhighlight %}

正常情况下，这是一个正常的`GirdView` 只有将`expanded`设置为true时才会全部展示。

如果ListView想要同样地效果，修改的代码是一样的。

## 使用

* 布局的例子

{% highlight ruby %}

<com.example.ExpandableHeightGridView
    android:id="@+id/myId"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:horizontalSpacing="2dp"
    android:isScrollContainer="false"
    android:numColumns="4"
    android:stretchMode="columnWidth"
    android:verticalSpacing="20dp" />

{% endhighlight %}

* java代码

{%  highlight java %}

mAppsGrid = (ExpandableHeightGridView) findViewById(R.id.myId);
mAppsGrid.setExpanded(true);

{%  endhighlight %}
