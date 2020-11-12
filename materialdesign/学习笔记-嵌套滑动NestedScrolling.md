### NestedScrollingParent 和 NestedScrollingChild 调用流程

![b0e5ee6d786a4dba86fb4a0a655c59e2](../assets/学习笔记-嵌套滑动NestedScrolling/b0e5ee6d786a4dba86fb4a0a655c59e2.png)

![07bc90cffc0e4aceaf3fc294fbcefb61](../assets/学习笔记-嵌套滑动NestedScrolling/07bc90cffc0e4aceaf3fc294fbcefb61.png)

### 触摸事件调用流程图  
![bd71cc7ab5cb4e1f9d6a6164f0100e85](../assets/学习笔记-嵌套滑动NestedScrolling/bd71cc7ab5cb4e1f9d6a6164f0100e85.png)





###  NestedScrollingChild

```java
public interface NestedScrollingChild {
    /**
    * 启用或禁用嵌套滚动的方法，设置为true，并且当前界面的View的层次结构是支持嵌套滚动的
    * (也就是需要NestedScrollingParent嵌套NestedScrollingChild)，才会触发嵌套滚动。
    * 一般这个方法内部都是直接代理给NestedScrollingChildHelper的同名方法即可
    */
    void setNestedScrollingEnabled(boolean enabled);
    
    /**
    * 判断当前View是否支持嵌套滑动。一般也是直接代理给NestedScrollingChildHelper的同名方法即可
    */
    boolean isNestedScrollingEnabled();
    
    /**
    * 表示view开始滚动了,一般是在ACTION_DOWN中调用，如果返回true则表示父布局支持嵌套滚动。
    * 一般也是直接代理给NestedScrollingChildHelper的同名方法即可。这个时候正常情况会触发Parent的onStartNestedScroll()方法
    */
    boolean startNestedScroll(@ScrollAxis int axes);
    
    /**
    * 一般是在事件结束比如ACTION_UP或者ACTION_CANCLE中调用,告诉父布局滚动结束。一般也是直接代理给NestedScrollingChildHelper的同名方法即可
    */
    void stopNestedScroll();
    
    /**
    * 判断当前View是否有嵌套滑动的Parent。一般也是直接代理给NestedScrollingChildHelper的同名方法即可
    */
    boolean hasNestedScrollingParent();
    
    /**
    * 在当前View消费滚动距离之后。通过调用该方法，把剩下的滚动距离传给父布局。如果当前没有发生嵌套滚动，或者不支持嵌套滚动，调用该方法也没啥用。
    * 内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可
    * dxConsumed：被当前View消费了的水平方向滑动距离
    * dyConsumed：被当前View消费了的垂直方向滑动距离
    * dxUnconsumed：未被消费的水平滑动距离
    * dyUnconsumed：未被消费的垂直滑动距离
    * offsetInWindow：输出可选参数。如果不是null，该方法完成返回时，
    * 会将该视图从该操作之前到该操作完成之后的本地视图坐标中的偏移量封装进该参数中，offsetInWindow[0]水平方向，offsetInWindow[1]垂直方向
    * @return true：表示滚动事件分发成功,fasle: 分发失败
    */
    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow);
    
    /**
    * 在当前View消费滚动距离之前把滑动距离传给父布局。相当于把优先处理权交给Parent
    * 内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可。
	* dx：当前水平方向滑动的距离
	* dy：当前垂直方向滑动的距离
	* consumed：输出参数，会将Parent消费掉的距离封装进该参数consumed[0]代表水平方向，consumed[1]代表垂直方向
	* @return true：代表Parent消费了滚动距离
    */
    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow);
    
    /**
    *将惯性滑动的速度分发给Parent。内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可
	* velocityX：表示水平滑动速度
	* velocityY：垂直滑动速度
	* consumed：true：表示当前View消费了滑动事件，否则传入false
	* @return true：表示Parent处理了滑动事件
	*/
    boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
    
    /**
    * 在当前View自己处理惯性滑动前，先将滑动事件分发给Parent,一般来说如果想自己处理惯性的滑动事件，
    * 就不应该调用该方法给Parent处理。如果给了Parent并且返回true，那表示Parent已经处理了，自己就不应该再做处理。
    * 返回false，代表Parent没有处理，但是不代表Parent后面就不用处理了
    * @return true：表示Parent处理了滑动事件
    */
    boolean dispatchNestedPreFling(float velocityX, float velocityY);
}

```

```java
public interface NestedScrollingChild2 extends NestedScrollingChild {

    boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type);

    void stopNestedScroll(@NestedScrollType int type);

    boolean hasNestedScrollingParent(@NestedScrollType int type);

    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
            @NestedScrollType int type);

    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow, @NestedScrollType int type);
}
```

```java
public interface NestedScrollingChild3 extends NestedScrollingChild2 {

    void dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed,
            @Nullable int[] offsetInWindow, @ViewCompat.NestedScrollType int type,
            @NonNull int[] consumed);
}

```

###  NestedScrollingParent

```java
public interface NestedScrollingParent {
    /**
    * 当NestedScrollingChild调用方法startNestedScroll()时,会调用该方法。主要就是通过返回值告诉系统是否需要对后续的滚动进行处理
    * child：该ViewParen的包含NestedScrollingChild的直接子View，如果只有一层嵌套，和target是同一个View
    * target：本次嵌套滚动的NestedScrollingChild
    * nestedScrollAxes：滚动方向
    * @return 
    * true:表示我需要进行处理，后续的滚动会触发相应的回到
    * false: 我不需要处理，后面也就不会进行相应的回调了
    */
    //child和target的区别，如果是嵌套两层如:Parent包含一个LinearLayout，LinearLayout里面才是NestedScrollingChild类型的View。这个时候，
    //child指向LinearLayout，target指向NestedScrollingChild；如果Parent直接就包含了NestedScrollingChild，
    //这个时候target和child都指向NestedScrollingChild
    boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes);

    /**
        * 如果onStartNestedScroll()方法返回的是true的话,那么紧接着就会调用该方法.它是让嵌套滚动在开始滚动之前,
    * 让布局容器(viewGroup)或者它的父类执行一些配置的初始化的
    */
    void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes);

    /**
    * 停止滚动了,当子view调用stopNestedScroll()时会调用该方法
    */
    void onStopNestedScroll(@NonNull View target);

    /**
    * 当子view调用dispatchNestedScroll()方法时,会调用该方法。也就是开始分发处理嵌套滑动了
    * dxConsumed：已经被target消费掉的水平方向的滑动距离
    * dyConsumed：已经被target消费掉的垂直方向的滑动距离
    * dxUnconsumed：未被tagert消费掉的水平方向的滑动距离
    * dyUnconsumed：未被tagert消费掉的垂直方向的滑动距离
    */
    void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed);

    /**
    * 当子view调用dispatchNestedPreScroll()方法是,会调用该方法。也就是在NestedScrollingChild在处理滑动之前，
    * 会先将机会给Parent处理。如果Parent想先消费部分滚动距离，将消费的距离放入consumed
    * dx：水平滑动距离
    * dy：处置滑动距离
    * consumed：表示Parent要消费的滚动距离,consumed[0]和consumed[1]分别表示父布局在x和y方向上消费的距离.
    */
    void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed);

    /**
    * 你可以捕获对内部NestedScrollingChild的fling事件
    * velocityX：水平方向的滑动速度
    * velocityY：垂直方向的滑动速度
    * consumed：是否被child消费了
    * @return
    * true:则表示消费了滑动事件
    */
    boolean onNestedFling(@NonNull View target, float velocityX, float velocityY, boolean consumed);

    /**
    * 在惯性滑动距离处理之前，会调用该方法，同onNestedPreScroll()一样，也是给Parent优先处理的权利
    * target：本次嵌套滚动的NestedScrollingChild
    * velocityX：水平方向的滑动速度
    * velocityY：垂直方向的滑动速度
    * @return
    * true：表示Parent要处理本次滑动事件，Child就不要处理了
    */
    boolean onNestedPreFling(@NonNull View target, float velocityX, float velocityY);

    /**
    * 返回当前滑动的方向，一般直接通过NestedScrollingParentHelper.getNestedScrollAxes()返回即可
    */
    @ScrollAxis
    int getNestedScrollAxes();
}
```

```java
public interface NestedScrollingParent2 extends NestedScrollingParent {
    boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes,
            @NestedScrollType int type);

    void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes,
            @NestedScrollType int type);

    void onStopNestedScroll(@NonNull View target, @NestedScrollType int type);

    void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);

    void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed,
            @NestedScrollType int type);

}
```

```java
public interface NestedScrollingParent3 extends NestedScrollingParent2 {

    void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed,
            int dyUnconsumed, @ViewCompat.NestedScrollType int type, @NonNull int[] consumed);

}
```

### NestedScrollingParentHelper

```java
public class NestedScrollingParentHelper {
//兼容NestedScrollingParent2的NestedScrollType
    private int mNestedScrollAxesTouch;
    private int mNestedScrollAxesNonTouch;

  
    public NestedScrollingParentHelper(@NonNull ViewGroup viewGroup) {
    }

    public void onNestedScrollAccepted(@NonNull View child, @NonNull View target,
            @ScrollAxis int axes) {
            //默认就是TYPE_TOUCH
        onNestedScrollAccepted(child, target, axes, ViewCompat.TYPE_TOUCH);
    }


    public void onNestedScrollAccepted(@NonNull View child, @NonNull View target,
            @ScrollAxis int axes, @NestedScrollType int type) {
        if (type == ViewCompat.TYPE_NON_TOUCH) {
            mNestedScrollAxesNonTouch = axes;
        } else {
            mNestedScrollAxesTouch = axes;
        }
    }


    @ScrollAxis
    public int getNestedScrollAxes() {
        return mNestedScrollAxesTouch | mNestedScrollAxesNonTouch;
    }

    public void onStopNestedScroll(@NonNull View target) {
        onStopNestedScroll(target, ViewCompat.TYPE_TOUCH);
    }


    public void onStopNestedScroll(@NonNull View target, @NestedScrollType int type) {
        if (type == ViewCompat.TYPE_NON_TOUCH) {
            mNestedScrollAxesNonTouch = ViewGroup.SCROLL_AXIS_NONE;
        } else {
            mNestedScrollAxesTouch = ViewGroup.SCROLL_AXIS_NONE;
        }
    }
}
```

### NestedScrollingChildHelper

```java
/**
* 一个标准的嵌套滚动的框架策略,即如何控制了嵌套滚动的事件分发和一些逻辑处理
* 就是在当前Child的所有的祖辈ViewParent中勋在一个实现了NestedScroolingParent接口，并且支持嵌套滚动(onStartNestedScroll()返回true)的。找到之后，在对应的分发方法中，将相关参数分发到ViewParent中与之对应的处理方法中。而且为了兼容性，都是通过ViewParentCompat进行转发操作的
*/
public class NestedScrollingChildHelper {
    private ViewParent mNestedScrollingParentTouch;
    private ViewParent mNestedScrollingParentNonTouch;
    private final View mView;
    private boolean mIsNestedScrollingEnabled;
    private int[] mTempNestedScrollConsumed;

 
	//就是传一个View进来，该View就是实现了NestedScrollingChild接口的View类型
    public NestedScrollingChildHelper(@NonNull View view) {
        mView = view;
    }

    public void setNestedScrollingEnabled(boolean enabled) {
    //主要用于是给变量mIsNestedScrollingEnabled进行赋值，
    //记录否可以支持嵌套滚动的方式。可以看到，如果之前是支持嵌套滚动的话，
    //会先调用ViewCompat.stopNestedScroll(mView)停止当前滚动，然后进行赋值操作
        if (mIsNestedScrollingEnabled) {
            ViewCompat.stopNestedScroll(mView);
        }
        mIsNestedScrollingEnabled = enabled;
    }

    public boolean isNestedScrollingEnabled() {
    //判断当前View是否支持嵌套滚动
        return mIsNestedScrollingEnabled;
    }


    public boolean hasNestedScrollingParent() {
        return hasNestedScrollingParent(TYPE_TOUCH);
    }

    public boolean hasNestedScrollingParent(@NestedScrollType int type) {
    //获取嵌套滚动的Parent，也就是实现了NestedScrollingParent的ViewGroup
        return getNestedScrollingParentForType(type) != null;
    }

    public boolean startNestedScroll(@ScrollAxis int axes) {
        return startNestedScroll(axes, TYPE_TOUCH);
    }

    public boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type) {
        if (hasNestedScrollingParent(type)) {
            // 先判断是否已经在嵌套滑动中，是的话，不处理
            return true;
        }
        if (isNestedScrollingEnabled()) {// 判断是否支持嵌套滑动
            ViewParent p = mView.getParent();
            View child = mView;
// 这里就是利用循环一层一层的往上取出ParentView，直到该ParentView是支持嵌套滑动或者为null的时候
            while (p != null) {
//就是判断当前ViewParent是否支持嵌套滑动。同时如果ViewParent是NestedScrollingParent的子类的话，会调用onStartNestedScroll()判断当前ViewParent是否需要嵌套滑动
                //如果外层View有一个CoordinatorLayout，则这个NestedScrollView就能关联上CoordinatorLayout了
                if (ViewParentCompat.onStartNestedScroll(p, child, mView, axes, type)) {//1
// 给mNestedScrollingParentTouch赋值，后面就可以直接获取有效ViewParent
                    setNestedScrollingParentForType(type, p);
                    //注释1 返回true，注释②里面就会调用NestedScrollingParent的onNestedScrollAccepted()方法,也就是说如果要处理嵌套滑动，onStartNestedScroll必须返回true
                    ViewParentCompat.onNestedScrollAccepted(p, child, mView, axes, type);
                    return true;
                }
                if (p instanceof View) {
                    child = (View) p;
                }
                p = p.getParent();
            }
        }
        return false;
    }

    public void stopNestedScroll() {
        stopNestedScroll(TYPE_TOUCH);
    }

    public void stopNestedScroll(@NestedScrollType int type) {
        ViewParent parent = getNestedScrollingParentForType(type);
        if (parent != null) {
        // 这里面就会调用ViewParent的onStopNestedScroll(target, type)方法
            ViewParentCompat.onStopNestedScroll(parent, mView, type);
            // 该次滑动结束 将给mNestedScrollingParentTouch置空
            setNestedScrollingParentForType(type, null);
        }
    }

    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow) {
        return dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                offsetInWindow, TYPE_TOUCH, null);
    }


    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
            int dyUnconsumed, @Nullable int[] offsetInWindow, @NestedScrollType int type) {
        return dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                offsetInWindow, type, null);
    }


    public void dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
            int dyUnconsumed, @Nullable int[] offsetInWindow, @NestedScrollType int type,
            @Nullable int[] consumed) {
        dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                offsetInWindow, type, consumed);
    }

    private boolean dispatchNestedScrollInternal(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
            @NestedScrollType int type, @Nullable int[] consumed) {
        if (isNestedScrollingEnabled()) {
        // 在startNestedScroll()中进行了赋值操作，所以这里可以直接获取ViewParent了
            final ViewParent parent = getNestedScrollingParentForType(type);
            if (parent == null) {
                return false;
            }
 // 判断是否是有效的嵌套滑动
            if (dxConsumed != 0 || dyConsumed != 0 || dxUnconsumed != 0 || dyUnconsumed != 0) {
                int startX = 0;
                int startY = 0;
                if (offsetInWindow != null) {
                    mView.getLocationInWindow(offsetInWindow);
                    startX = offsetInWindow[0];
                    startY = offsetInWindow[1];
                }

                if (consumed == null) {
                    consumed = getTempNestedScrollConsumed();
                    consumed[0] = 0;
                    consumed[1] = 0;
                }
 // 这里就会调用ViewParent的onNestedScroll()方法
                ViewParentCompat.onNestedScroll(parent, mView,
                        dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, type, consumed);

                if (offsetInWindow != null) {
                    mView.getLocationInWindow(offsetInWindow);
                    offsetInWindow[0] -= startX;
                    offsetInWindow[1] -= startY;
                }
                return true;
            } else if (offsetInWindow != null) {
                offsetInWindow[0] = 0;
                offsetInWindow[1] = 0;
            }
        }
        return false;
    }


    public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow) {
        return dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow, TYPE_TOUCH);
    }

    public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow, @NestedScrollType int type) {
        if (isNestedScrollingEnabled()) {
            final ViewParent parent = getNestedScrollingParentForType(type);
            if (parent == null) {
                return false;
            }

            if (dx != 0 || dy != 0) {
                int startX = 0;
                int startY = 0;
                if (offsetInWindow != null) {
                    mView.getLocationInWindow(offsetInWindow);
                    startX = offsetInWindow[0];
                    startY = offsetInWindow[1];
                }

                if (consumed == null) {
                    consumed = getTempNestedScrollConsumed();
                }
                consumed[0] = 0;
                consumed[1] = 0;
                // 这里会调用ViewParent的onNestedPreScroll()方法 Parent消费的数据会缝在consumed变量中
                ViewParentCompat.onNestedPreScroll(parent, mView, dx, dy, consumed, type);

                if (offsetInWindow != null) {
                    mView.getLocationInWindow(offsetInWindow);
                    offsetInWindow[0] -= startX;
                    offsetInWindow[1] -= startY;
                }
                return consumed[0] != 0 || consumed[1] != 0;
            } else if (offsetInWindow != null) {
                offsetInWindow[0] = 0;
                offsetInWindow[1] = 0;
            }
        }
        return false;
    }

    public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
        if (isNestedScrollingEnabled()) {
            ViewParent parent = getNestedScrollingParentForType(TYPE_TOUCH);
            if (parent != null) {
            // 这里会调用ViewParent的onNestedFling()方法
                return ViewParentCompat.onNestedFling(parent, mView, velocityX,
                        velocityY, consumed);
            }
        }
        return false;
    }

    public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
        if (isNestedScrollingEnabled()) {
            ViewParent parent = getNestedScrollingParentForType(TYPE_TOUCH);
            if (parent != null) {
                return ViewParentCompat.onNestedPreFling(parent, mView, velocityX,
                        velocityY);
            }
        }
        return false;
    }

    public void onDetachedFromWindow() {
        ViewCompat.stopNestedScroll(mView);
    }

    public void onStopNestedScroll(@NonNull View child) {
        ViewCompat.stopNestedScroll(mView);
    }

    private ViewParent getNestedScrollingParentForType(@NestedScrollType int type) {
        switch (type) {
            case TYPE_TOUCH:
                return mNestedScrollingParentTouch;
            case TYPE_NON_TOUCH:
                return mNestedScrollingParentNonTouch;
        }
        return null;
    }

    private void setNestedScrollingParentForType(@NestedScrollType int type, ViewParent p) {
        switch (type) {
            case TYPE_TOUCH:
                mNestedScrollingParentTouch = p;
                break;
            case TYPE_NON_TOUCH:
                mNestedScrollingParentNonTouch = p;
                break;
        }
    }

    private int[] getTempNestedScrollConsumed() {
        if (mTempNestedScrollConsumed == null) {
            mTempNestedScrollConsumed = new int[2];
        }
        return mTempNestedScrollConsumed;
    }
}
```

### 