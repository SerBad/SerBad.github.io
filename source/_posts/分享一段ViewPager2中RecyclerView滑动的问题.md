---
title: 分享一段ViewPager2中RecyclerView滑动的问题
tags: Android
comments: false
date: 2021-02-19 11:17:24
---

在**ViewPager2**中插入**RecyclerView**，滑动过于敏感，下面记录一种方法，其实就是重新处理事件分发。
<!--more-->
```kotlin
import android.content.Context
import android.util.AttributeSet
import android.view.MotionEvent
import androidx.recyclerview.widget.RecyclerView
import kotlin.math.abs

class RecyclerViewAtViewPager2 : RecyclerView {
    constructor(context: Context) : super(context) {}
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs) {}
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(
        context,
        attrs,
        defStyleAttr
    ) {
    }

    private var disallowIntercept = false

    private var startX = 0
    private var startY = 0
    var isDispatch: Boolean = true
    override fun dispatchTouchEvent(ev: MotionEvent): Boolean {
        if (isDispatch) {
            when (ev.action) {
                MotionEvent.ACTION_DOWN -> {
                    startX = ev.x.toInt()
                    startY = ev.y.toInt()
                    parent.requestDisallowInterceptTouchEvent(true)
                }
                MotionEvent.ACTION_MOVE -> {
                    val endX = ev.x.toInt()
                    val endY = ev.y.toInt()
                    val disX = abs(endX - startX)
                    val disY = abs(endY - startY)
                    if (disX > disY) {
                        //为了解决RecyclerView嵌套RecyclerView时横向滑动的问题
                        if (disallowIntercept) {
                            parent.requestDisallowInterceptTouchEvent(disallowIntercept)
                        } else {
                            parent.requestDisallowInterceptTouchEvent(canScrollHorizontally(startX - endX))
                        }
                    } else {
                        parent.requestDisallowInterceptTouchEvent(true)
                    }
                }
                MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> parent.requestDisallowInterceptTouchEvent(
                    false
                )
            }
        }

        return super.dispatchTouchEvent(ev)
    }

    override fun requestDisallowInterceptTouchEvent(disallowIntercept: Boolean) {
        this.disallowIntercept = disallowIntercept
        super.requestDisallowInterceptTouchEvent(disallowIntercept)

    }
}
```
