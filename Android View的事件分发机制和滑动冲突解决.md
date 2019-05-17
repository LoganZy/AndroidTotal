---


---

<h1 id="android-view的事件分发机制和滑动冲突解决">Android View的事件分发机制和滑动冲突解决</h1>
<p>文章转载自<a href="https://blog.csdn.net/qian520ao/article/details/77429593">https://blog.csdn.net/qian520ao/article/details/77429593</a></p>
<h2 id="初探view事件">初探View事件</h2>
<p>前言View的事件分发和滑动冲突处理是老生常谈的知识了，因为最近撸了一个仿QQ侧滑删除，所以对该View事件有了更深入的总结。老铁们是时候走一波star了。<br>
我们常说的View事件是指： 从手指亲密接触屏幕的那一刻到手指离开屏幕的这个过程，该事件序列以down事件为起点，move事件为过程，up事件为终点。<br>
一次down-move-up这一个事件过程我们称为一个事件序列。所以我们今天研究的对象就是MotionEvent。</p>
<h3 id="事件分发">事件分发</h3>
<h3 id="理论知识">理论知识</h3>
<ul>
<li>
<p>public boolean dispatchTouchEvent(MotionEvent ev)<br>
用来分发事件，即事件序列的大门，如果事件传递到当前View的onTouchEvent或者是子View的dispatchTouchEvent，即该方法被调用了。<br>
return true: 表示消耗了当前事件，有可能是当前View的onTouchEvent或者是子View的dispatchTouchEvent消费了，事件终止，不再传递。<br>
return false: 调用父ViewGroup或则Activity的onTouchEvent。 （不再往下传）。①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件都与该View无关，交由父类处理(父类的onTouchEvent方法)<br>
return super.dispatherTouchEvent: 则继续往下(子View)传递，或者是调用当前View的onTouchEvent方法;</p>
</li>
<li>
<p>public boolean onInterceptTouchEvent(MotionEvent ev)<br>
在dispatchTouchEvent内部调用，顾名思义就是判断是否拦截某个事件。(注：ViewGroup才有的方法，View因为没有子View了，所以不需要也没有该方法)<br>
return true: ViewGroup将该事件拦截，交给自己的onTouchEvent处理。②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该onInterceptTouchEvent方法去询问是否拦截。<br>
return false: 继续传递给子元素的dispatchTouchEvent处理。<br>
return super.dispatherTouchEvent: 事件默认不会被拦截。</p>
</li>
<li>
<p>public boolean onTouchEvent(MotionEvent ev)<br>
在dispatchTouchEvent内部调用<br>
return true: 事件消费，当前事件终止。<br>
return false: 交给父View的onTouchEvent。<br>
return super.dispatherTouchEvent: 默认处理事件的逻辑和返回 false 时相同。</p>
</li>
</ul>
<p>其实上面的关系可以用以下代码简单描述。</p>
<pre><code>public boolean dispatchTouchEvent(MotionEvent ev){
    boolean consume = false;//是否消费事件
    if(onInterceptTouchEvent(ev)){//是否拦截事件
        consume = onTouchEvent(ev);//拦截了，交给自己的View处理
    }else{
        consume = child.dispatchTouchEvent(ev);//不拦截，就交给子View处理
    }

    return consume;//true：消费事件，终止。false:交给父onTouchEvent处理。并不再往下传递当前事件。
}
</code></pre>
<p>有图有真相</p>
<p><a href="https://i.loli.net/2019/05/17/5cde989015ba618757.jpg"><img src="https://i.loli.net/2019/05/17/5cde989015ba618757.jpg" alt="android view-dispatch.jpg"></a></p>
<h2 id="实战讲解">实战讲解</h2>
<h3 id="验证view的事件分发">验证View的事件分发</h3>
<ul>
<li>创建CustomViewGroup继承FrameLayout</li>
<li>创建CustomView继承View<br>
xml文件</li>
</ul>
<pre><code>&lt;FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="qdx.viewtouchevent.MainActivity"&gt;

//最外层为activity（白色背景）
    &lt;qdx.viewtouchevent.CustomViewGroup
        android:layout_width="300dp"
        android:layout_height="400dp"
        android:layout_gravity="right"
        android:background="#84bf96"&gt;
//CustomViewGroup（绿色背景）包含CustomView（黄色背景）
        &lt;qdx.viewtouchevent.CustomView
            android:layout_width="150dp"
            android:layout_height="300dp"
            android:layout_gravity="right"
            android:background="#f2eada" /&gt;

    &lt;/qdx.viewtouchevent.CustomViewGroup&gt;

&lt;/FrameLayout&gt;

</code></pre>
<p><a href="https://i.loli.net/2019/05/17/5cde98dd3653f58674.jpeg"><img src="https://i.loli.net/2019/05/17/5cde98dd3653f58674.jpeg" alt="demo1.jpeg"></a></p>
<p>如上图所示，down事件由activity-&gt;ViewGroup-&gt;View，因为View并没有处理down事件，所以事件消费情况为false，并且最后由View-&gt;ViewGroup-&gt;activity传递。</p>
<h3 id="验证不消耗action_down事件">验证不消耗ACTION_DOWN事件</h3>
<p>我们再来验证①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件系列都与该View无关，交由父类处理(父类的onTouchEvent方法)<br>
根据上面文字描述，因为我们的CustomViewGroup和CustomView都没有去处理任何事件，即当前序列的所有事件都return false，所以我们也无法接收/处理其他事件(move,up)</p>
<p>我们再将customView设置为可点击状态，即消费touch事件。setClickable(true);</p>
<h3 id="验证-viewgroup事件拦截">验证 ViewGroup事件拦截</h3>
<p>viewGroup将事件拦截后，②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该onInterceptTouchEvent方法去询问是否拦截。</p>
<p>通过上面的几个验证，我们越来越接近真相，用通俗的话来解释就是：</p>
<blockquote>
<p>老板发现BUG解决，一开始是由上级往下级问话。（类似责任链设计模式）<br>
例如突然间出现了BUG，老板问小组A有没有空处理一下BUG(即分发ACTION_DOWN)，小组A说没时间(return false)，那么老板就不会把这个序列的BUG（ACTION_MOVE和ACTION_UP）交给小组A。如果再次出现BUG，老板还会再次询问小组A。①<br>
如果你举手揽了这个BUG（即拦截），那么这一事件的BUG都交由你解决，并且相同序列的BUG老板不会问话，直接找你处理。②</p>
</blockquote>
<h2 id="activity的事件分发">Activity的事件分发</h2>
<p>Activity的事件分发还关系到View的绘制和加载机制，等待下一篇来更详细认识这个知识点。</p>
<pre><code>public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }

    //最终获取到顶级View(ViewGroup)分发事件
    //（getWindow().getDecorView().findViewById(android.R.id.Content)）.getChildAt(0)
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }

    //如果所有的View都没有处理事件，则由Activity亲自出马
    return onTouchEvent(ev);
}
</code></pre>
<h2 id="viewgroup的事件拦截">ViewGroup的事件拦截</h2>
<pre><code>        public boolean dispatchTouchEvent(MotionEvent ev) {
        ......

        final int action = ev.getAction();
        final int actionMasked = action &amp; MotionEvent.ACTION_MASK;

        if (actionMasked == MotionEvent.ACTION_DOWN) {
            cancelAndClearTouchTargets(ev);

            //清除FLAG_DISALLOW_INTERCEPT，并且设置mFirstTouchTarget为null
            resetTouchState(){
                if(mFirstTouchTarget!=null){mFirstTouchTarget==null;}
                mGroupFlags &amp;= ~FLAG_DISALLOW_INTERCEPT;
                ......
            };
        }
        final boolean intercepted;//ViewGroup是否拦截事件

        //mFirstTouchTarget是ViewGroup中处理事件(return true)的子View
        //如果没有子View处理则mFirstTouchTarget=null,ViewGroup自己处理
        if (actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null) {
            final boolean disallowIntercept = (mGroupFlags &amp; FLAG_DISALLOW_INTERCEPT) != 0;
            if (!disallowIntercept) {
                intercepted = onInterceptTouchEvent(ev);//onInterceptTouchEvent
                ev.setAction(action);
            } else {
                intercepted = false;

                //如果子类设置requestDisallowInterceptTouchEvent（true）
                //ViewGroup将无法拦截MotionEvent.ACTION_DOWN以外的事件
            }
        } else {
            intercepted = true;

            //actionMasked != MotionEvent.ACTION_DOWN并且没有子View处理事件，则将事件拦截
            //并且不会再调用onInterceptTouchEvent询问是否拦截
        }

        ......
        ......
      }
</code></pre>
<p>我们将上面的结论再次写下来，方便对照。<br>
①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件都与该View无关，交由父类处理(父类的onTouchEvent方法)（dispatchTouchEvent）<br>
②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该onInterceptTouchEvent方法去询问是否拦截。（onInterceptTouchEvent return true）</p>
<ul>
<li>首先我们分析上面第21行代码： ViewGroup在两种情况下会拦截事件（ACTION_DOWN || mFirstTouchTarget != null）所以反过来也就是说 I : 当ACTION_MOVE和ACTION_UP事件到来时，如果没有子元素处理事件（mFirstTouchTarget==null），则ViewGroup的onInterceptTouchEvent不会再被调用，而且同一序列中的其它事件都会默认交给它处理（第34行 intercepted=true）；与上面所说的①②呼应。</li>
<li>紧接着22行： ViewGroupdisallowIntercept（不拦截）的判定是FLAG_DISALLOW_INTERCEPT标记位，这个标记是通过子ViewrequestDisallowInterceptTouchEvent方法设置的。所以我们可以得出这么一个结论II : 当子View处理了ACTION_DOWN事件(mFirstTouchTarget =该子View)，而且设置了FLAG_DISALLOW_INTERCEPT标记位，那么ViewGroup将无法拦截除了ACTION_DOWN以外的其它事件。（在11行代码ACTION_DOWN时清除了FLAG_DISALLOW_INTERCEPT标记位，所以ViewGroup无论如何都可以选择是否拦截处理ACTION_DOWN）<br>
上面变着花样的又一次验证了①②个知识点，不得不说read the fuck source code让我们可以找到一个处理滑动冲突的方法：子View处理DOWN事件并且设置FLAG_DISALLOW_INTERCEPT标记位，就可以不让ViewGroup拦截DOWN以外的事件。</li>
</ul>
<h2 id="viewgroup的事件分发">ViewGroup的事件分发</h2>
<pre><code>    public boolean dispatchTouchEvent(MotionEvent ev) {
    final View[] children = mChildren;

    for (int i = childrenCount - 1; i &gt;= 0; i--) {
        final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
        final View child = getAndVerifyPreorderedView(preorderedList, children, childIndex);

        ......

        if (!canViewReceivePointerEvents(child) || !isTransformedTouchPointInView(x, y, child, null)) 
        {
            ev.setTargetAccessibilityFocus(false);
            //如果子View没有播放动画，而且点击事件的坐标在子View的区域内，继续下面的判断
            continue;
        }
        //判断是否有子View处理了事件
        newTouchTarget = getTouchTarget(child);

        if (newTouchTarget != null) {
            //如果已经有子View处理了事件，即mFirstTouchTarget!=null，终止循环。
            newTouchTarget.pointerIdBits |= idBitsToAssign;
            break;
        }

        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
            //点击dispatchTransformedTouchEvent代码发现其执行方法实际为
            //return child.dispatchTouchEvent(event); （因为child!=null）
            //所以如果有子View处理了事件，我们就进行下一步：赋值

            ......

            newTouchTarget = addTouchTarget(child, idBitsToAssign);
            //addTouchTarget方法里完成了对mFirstTouchTarget的赋值
            alreadyDispatchedToNewTouchTarget = true;

            break;
        }
    }
}

private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
    final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
    target.next = mFirstTouchTarget;
    mFirstTouchTarget = target;
    return target;
}

private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
        View child, int desiredPointerIdBits) {
        ......

        if (child == null) {
        //如果没有子View处理事件，就自己处理
            handled = super.dispatchTouchEvent(event);
        } else {
       //有子View，调用子View的dispatchTouchEvent方法
            handled = child.dispatchTouchEvent(event);

        ......

        return handled;
}
</code></pre>
<p>上面为ViewGroup对事件的分发，主要有2点</p>
<p>如果有子View，则调用子View的dispatchTouchEvent方法判断是否处理了事件，如果处理了便赋值mFirstTouchTarget，赋值成功则跳出循环。<br>
ViewGroup的事件分发最终还是调用View的dispatchTouchEvent方法，具体如上代码所述。<br>
至此View的事件分发机制已经演练完毕，如果事件分发机制理解深入的话，那么处理滑动冲突便是手到擒来了。</p>
<h2 id="view的滑动冲突">View的滑动冲突</h2>
<p>关于View的滑动冲突我们就开门见山吧，因为上述的事件分发已经有足够的理论知识了，我们可以单刀赴会了。</p>
<p>针对上图，这个是比较普遍的滑动冲突事件，我们先拿它来开刀。</p>
<p>好记性不如烂笔头，我们再次把结论搬到战场上<br>
①另外如果不消耗ACTION_DOWN事件，那么down,move,up事件都与该View无关，交由父类处理(父类的onTouchEvent方法)（dispatchTouchEvent）<br>
②而且这一个事件序列（当前和其它事件）都只能由该ViewGroup处理，并且不会再调用该onInterceptTouchEvent方法去询问是否拦截。（onInterceptTouchEvent return true）</p>
<p>I : 当ACTION_MOVE和ACTION_UP事件到来时，如果没有子元素处理事件（mFirstTouchTarget==null），则ViewGroup的onInterceptTouchEvent不会再被调用，而且同一序列中的其它事件都会默认交给它处理（第34行 intercepted=true）；</p>
<h3 id="外部拦截">外部拦截</h3>
<p>外部拦截顾名思义就是由父ViewGroup对事件拦截处理（所以重写onInterceptTouchEvent方法即可），子View只能眼巴巴的处理父View“吃剩”的事件。主要有以下几点。</p>
<p>父类不能拦截ACTION_DOWN，也就是说必须返回false，根据上述①②和 I 可得。<br>
父类在ACTION_MOVE的时候根据需求，判断是否拦截。<br>
ACTION_UP事件建议返回false或者super.onInterceptTouchEvent，因为如果已经拦截的话，那么并不会调用onInterceptTouchEvent方法再次询问。如果不拦截，而且返回true，子View可能就无法触发onClick等相关事件。<br>
ViewGroup ： 需要重写onInterceptTouchEvent，判断是否拦截即可。<br>
但是有一种情况：用户正在水平滑动（事件已拦截给ViewGroup），但是水平滑动停止前用户再进行竖直滑动，下面代码我用isSolve进行简单的处理。</p>
<pre><code>private boolean isIntercept;
private boolean isSolve;//是否完成了拦截判断，如果决定拦截，那么同系列事件就不能设置为不拦截

@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mPointGapF.x = ev.getX();
            mPointGapF.y = ev.getY();
            return false;//down的时候拦截后，就只能交给自己处理了

        case MotionEvent.ACTION_MOVE:
            if (!isSolve) {//是否已经决定拦截/不拦截？
                isIntercept = (Math.abs(ev.getX() - mPointGapF.x) &gt; Math.abs(ev.getY() - mPointGapF.y)*2);//如果是左右滑动，且水平角度小于30°，就拦截
                isSolve = true;
            }
            return isIntercept;//如果是左右滑动，就拦截
    }
    return super.onInterceptTouchEvent(ev);
}

@Override
public boolean onTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_MOVE:
            scrollBy((int) (mPointGapF.x - ev.getX()), 0);

            mPointGapF.x = ev.getX();
            mPointGapF.y = ev.getY();
            break;
    }
    return super.onTouchEvent(ev);
}
</code></pre>
<p>子View ： 和子View没有多大关系，只需要处理自身的移动操作即可。</p>
<pre><code>    public boolean onTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                scrollBy(0, (int) (mPointGapF.y - ev.getY()));
                mPointGapF.x = ev.getX();
                mPointGapF.y = ev.getY();
                break;
        }
        return true;
    }
</code></pre>
<h3 id="内部拦截">内部拦截</h3>
<p>II : 当子View处理了ACTION_DOWN事件(mFirstTouchTarget =该子View)，而且设置了FLAG_DISALLOW_INTERCEPT标记位，那么ViewGroup将无法拦截除了ACTION_DOWN以外的其它事件。</p>
<p>ViewGroup ： 只需在onInterceptTouchEventMotionEvent.ACTION_DOWN时候不拦截，其他时候都需要拦截，否则父类的onTouchEvent就不能处理任何事件了。</p>
<pre><code>public boolean onInterceptTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            return false;//down的时候拦截后，就只能交给自己处理了
    }
    return true;//如果不拦截，父类的onTouchEvent方法就无事件可以处理。
}

@Override
public boolean onTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_MOVE:
            scrollBy((int) (mPointGapF.x - ev.getX()), 0);

            mPointGapF.x = ev.getX();
            mPointGapF.y = ev.getY();
            break;
    }
    return super.onTouchEvent(ev);
}
</code></pre>
<p>子View ： 需要在ACTION_DOWN事件设置getParent().requestDisallowInterceptTouchEvent(true)，并且在ACTION_MOVE的时候通过判断是否禁止父类的拦截。</p>
<pre><code>private boolean isSolve;
private boolean isIntercept;

@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            isIntercept = false;
            isSolve = false;
            mPointGapF.x = ev.getX();
            mPointGapF.y = ev.getY();
            getParent().requestDisallowInterceptTouchEvent(true);
            break;

        case MotionEvent.ACTION_MOVE:
            if (!isSolve) {
                isSolve = true;
                isIntercept = (Math.abs(ev.getX() - mPointGapF.x) &lt; Math.abs(ev.getY() - mPointGapF.y) * 2);
                getParent().requestDisallowInterceptTouchEvent(isIntercept);
            }
            break;
    }
    return super.dispatchTouchEvent(ev);
}
</code></pre>
<p>最终的效果图和外部拦截的效果一致，这里就不再次贴出来了。</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

