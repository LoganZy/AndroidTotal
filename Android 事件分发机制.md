---


---

<h1 id="前言">前言</h1>
<p>文章转载自<a href="https://www.jianshu.com/p/38015afcdb58">https://www.jianshu.com/p/38015afcdb58</a></p>
<ul>
<li><code>Android</code>事件分发机制是<code>Android</code>开发者必须了解的基础</li>
<li>网上有大量关于<code>Android</code>事件分发机制的文章，但存在一些问题：<strong>内容不全、思路不清晰、无源码分析、简单问题复杂化等等</strong></li>
<li>今天，我将全面总结<code>Android</code>的事件分发机制，我能保证这是<strong>市面上的最全面、最清晰、最易懂的</strong></li>
</ul>
<blockquote>
<ol>
<li>本文秉着“结论先行、详细分析在后”的原则，即先让大家感性认识，再通过理性分析从而理解问题；</li>
<li>所以，请各位读者先记住结论，再往下继续看分析；</li>
</ol>
</blockquote>
<ul>
<li>文章较长，阅读需要较长时间，建议收藏等充足时间再进行阅读</li>
</ul>
<hr>
<h1 id="目录">目录</h1>
<p><img src="//upload-images.jianshu.io/upload_images/944365-e7baca065f885271.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<hr>
<h1 id="基础认知">1. 基础认知</h1>
<h3 id="事件分发的对象是谁？">1.1 事件分发的对象是谁？</h3>
<p><strong>答：点击事件（<code>Touch</code>事件）</strong></p>
<ul>
<li>定义<br>
当用户触摸屏幕时（<code>View</code> 或 <code>ViewGroup</code>派生的控件），将产生点击事件（<code>Touch</code>事件）</li>
</ul>
<blockquote>
<p><code>Touch</code>事件的相关细节（发生触摸的位置、时间等）被封装成<code>MotionEvent</code>对象</p>
</blockquote>
<ul>
<li>事件类型（4种）</li>
</ul>
<p>事件类型</p>
<p>具体动作</p>
<p>MotionEvent.ACTION_DOWN</p>
<p>按下View（所有事件的开始）</p>
<p>MotionEvent.ACTION_UP</p>
<p>抬起View（与DOWN对应）</p>
<p>MotionEvent.ACTION_MOVE</p>
<p>滑动View</p>
<p>MotionEvent.ACTION_CANCEL</p>
<p>结束事件（非人为原因）</p>
<ul>
<li>特别说明：事件列<br>
从手指接触屏幕 至 手指离开屏幕，这个过程产生的一系列事件</li>
</ul>
<blockquote>
<p>注：一般情况下，事件列都是以<code>DOWN</code>事件开始、<code>UP</code>事件结束，中间有无数的MOVE事件，如下图：</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-79b1e86793514e99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/990/format/webp" alt=""></p>
<p>事件列</p>
</blockquote>
<p>即当一个点击事件（<code>MotionEvent</code> ）产生后，系统需把这个事件传递给一个具体的 <code>View</code> 去处理。</p>
<h3 id="事件分发的本质">1.2 事件分发的本质</h3>
<p><strong>答：将点击事件（MotionEvent）传递到某个具体的<code>View</code> &amp; 处理的整个过程</strong></p>
<blockquote>
<p>即 事件传递的过程 = 分发过程。</p>
</blockquote>
<h3 id="事件在哪些对象之间进行传递？">1.3 事件在哪些对象之间进行传递？</h3>
<p><strong>答：Activity、ViewGroup、View</strong></p>
<ul>
<li>
<p><code>Android</code>的<code>UI</code>界面由<code>Activity</code>、<code>ViewGroup</code>、<code>View</code> 及其派生类组成</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-ece40d4524784ffa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/256/format/webp" alt=""></p>
<p>UI界面</p>
</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-02c588300f6ad741.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/590/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="事件分发的顺序">1.4 事件分发的顺序</h3>
<p>即 事件传递的顺序：<code>Activity</code> -&gt; <code>ViewGroup</code> -&gt; <code>View</code></p>
<blockquote>
<p>即：1个点击事件发生后，事件先传到<code>Activity</code>、再传到<code>ViewGroup</code>、最终再传到 <code>View</code></p>
</blockquote>
<p><img src="//upload-images.jianshu.io/upload_images/944365-7fee82bba19a3821.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/505/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="事件分发过程由哪些方法协作完成？">1.5 事件分发过程由哪些方法协作完成？</h3>
<p><strong>答：dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-7c6642f518ffa3d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/675/format/webp" alt=""></p>
<p>示意图</p>
<blockquote>
<p>下文会对这3个方法进行详细介绍</p>
</blockquote>
<h3 id="总结">1.6 总结</h3>
<p><img src="//upload-images.jianshu.io/upload_images/944365-d0a7e6f3c2bbefcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>至此，相信大家已经对 <code>Android</code>的事件分发有了感性的认知</li>
<li>下面，我将详细介绍<code>Android</code>事件分发机制</li>
</ul>
<hr>
<h1 id="事件分发机制-源码分析">2. 事件分发机制 源码分析</h1>
<ul>
<li>请谨记：<code>Android</code>事件分发流程 = <strong>Activity -&gt; ViewGroup -&gt; View</strong></li>
</ul>
<blockquote>
<p>即：1个点击事件发生后，事件先传到<code>Activity</code>、再传到<code>ViewGroup</code>、最终再传到 <code>View</code></p>
</blockquote>
<p><img src="//upload-images.jianshu.io/upload_images/944365-2064dcb69200fc6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/505/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>从上可知，要想充分理解Android分发机制，本质上是要理解：
<ol>
<li><code>Activity</code>对点击事件的分发机制</li>
<li><code>ViewGroup</code>对点击事件的分发机制</li>
<li><code>View</code>对点击事件的分发机制</li>
</ol>
</li>
<li>下面，我将通过源码，全面解析 <strong>事件分发机制</strong></li>
</ul>
<blockquote>
<p>即按顺序讲解：<code>Activity</code>事件分发机制、<code>ViewGroup</code>事件分发机制、<code>View</code>事件分发机制</p>
</blockquote>
<h3 id="activity的事件分发机制">2.1 Activity的事件分发机制</h3>
<p>当一个点击事件发生时，事件最先传到<code>Activity</code>的<code>dispatchTouchEvent()</code>进行事件分发</p>
<h3 id="源码分析">2.1.1 源码分析</h3>
<pre><code>/**
  * 源码分析：Activity.dispatchTouchEvent（）
  */ 
    public boolean dispatchTouchEvent(MotionEvent ev) {

            // 一般事件列开始都是DOWN事件 = 按下事件，故此处基本是true
            if (ev.getAction() == MotionEvent.ACTION_DOWN) {

                onUserInteraction();
                // -&gt;&gt;分析1

            }

            // -&gt;&gt;分析2
            if (getWindow().superDispatchTouchEvent(ev)) {

                return true;
                // 若getWindow().superDispatchTouchEvent(ev)的返回true
                // 则Activity.dispatchTouchEvent（）就返回true，则方法结束。即 ：该点击事件停止往下传递 &amp; 事件传递过程结束
                // 否则：继续往下调用Activity.onTouchEvent

            }
            // -&gt;&gt;分析4
            return onTouchEvent(ev);
        }


/**
  * 分析1：onUserInteraction()
  * 作用：实现屏保功能
  * 注：
  *    a. 该方法为空方法
  *    b. 当此activity在栈顶时，触屏点击按home，back，menu键等都会触发此方法
  */
      public void onUserInteraction() { 

      }
      // 回到最初的调用原处

/**
  * 分析2：getWindow().superDispatchTouchEvent(ev)
  * 说明：
  *     a. getWindow() = 获取Window类的对象
  *     b. Window类是抽象类，其唯一实现类 = PhoneWindow类；即此处的Window类对象 = PhoneWindow类对象
  *     c. Window类的superDispatchTouchEvent() = 1个抽象方法，由子类PhoneWindow类实现
  */
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {

        return mDecor.superDispatchTouchEvent(event);
        // mDecor = 顶层View（DecorView）的实例对象
        // -&gt;&gt; 分析3
    }

/**
  * 分析3：mDecor.superDispatchTouchEvent(event)
  * 定义：属于顶层View（DecorView）
  * 说明：
  *     a. DecorView类是PhoneWindow类的一个内部类
  *     b. DecorView继承自FrameLayout，是所有界面的父类
  *     c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup
  */
    public boolean superDispatchTouchEvent(MotionEvent event) {

        return super.dispatchTouchEvent(event);
        // 调用父类的方法 = ViewGroup的dispatchTouchEvent()
        // 即 将事件传递到ViewGroup去处理，详细请看ViewGroup的事件分发机制

    }
    // 回到最初的调用原处

/**
  * 分析4：Activity.onTouchEvent（）
  * 定义：属于顶层View（DecorView）
  * 说明：
  *     a. DecorView类是PhoneWindow类的一个内部类
  *     b. DecorView继承自FrameLayout，是所有界面的父类
  *     c. FrameLayout是ViewGroup的子类，故DecorView的间接父类 = ViewGroup
  */
  public boolean onTouchEvent(MotionEvent event) {

        // 当一个点击事件未被Activity下任何一个View接收 / 处理时
        // 应用场景：处理发生在Window边界外的触摸事件
        // -&gt;&gt; 分析5
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }
        
        return false;
        // 即 只有在点击事件在Window边界外才会返回true，一般情况都返回false，分析完毕
    }

/**
  * 分析5：mWindow.shouldCloseOnTouch(this, event)
  */
    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    // 主要是对于处理边界外点击事件的判断：是否是DOWN事件，event的坐标是否在边界内等
    if (mCloseOnTouchOutside &amp;&amp; event.getAction() == MotionEvent.ACTION_DOWN
            &amp;&amp; isOutOfBounds(context, event) &amp;&amp; peekDecorView() != null) {
        return true;
    }
    return false;
    // 返回true：说明事件在边界外，即 消费事件
    // 返回false：未消费（默认）
}
// 回到分析4调用原处

</code></pre>
<h3 id="总结-1">2.1.2 总结</h3>
<ul>
<li>当一个点击事件发生时，从<code>Activity</code>的事件分发开始（<code>Activity.dispatchTouchEvent()</code>）</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-f8fda76bbdad7b96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/590/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>方法总结</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-e186b0edcb590546.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p>那么，<code>ViewGroup</code>的<code>dispatchTouchEvent()</code>什么时候返回<code>true</code> / <code>false</code>？请继续往下看<strong>ViewGroup事件的分发机制</strong></p>
<hr>
<h1 id="viewgroup事件的分发机制">2.2 ViewGroup事件的分发机制</h1>
<p>从上面<code>Activity</code>事件分发机制可知，<code>ViewGroup</code>事件分发机制从<code>dispatchTouchEvent()</code>开始</p>
<h3 id="源码分析-1">2.2.1 源码分析</h3>
<blockquote>
<ol>
<li><code>Android 5.0</code>后，<code>ViewGroup.dispatchTouchEvent()</code>的源码发生了变化（更加复杂），但原理相同；</li>
<li>本文为了让读者容易理解，故采用<code>Android 5.0</code>前的版本</li>
</ol>
</blockquote>
<pre><code>/**
  * 源码分析：ViewGroup.dispatchTouchEvent（）
  */ 
    public boolean dispatchTouchEvent(MotionEvent ev) { 

    ... // 仅贴出关键代码

        // 重点分析1：ViewGroup每次事件分发时，都需调用onInterceptTouchEvent()询问是否拦截事件
            if (disallowIntercept || !onInterceptTouchEvent(ev)) {  

            // 判断值1：disallowIntercept = 是否禁用事件拦截的功能(默认是false)，可通过调用requestDisallowInterceptTouchEvent（）修改
            // 判断值2： !onInterceptTouchEvent(ev) = 对onInterceptTouchEvent()返回值取反
                    // a. 若在onInterceptTouchEvent()中返回false（即不拦截事件），就会让第二个值为true，从而进入到条件判断的内部
                    // b. 若在onInterceptTouchEvent()中返回true（即拦截事件），就会让第二个值为false，从而跳出了这个条件判断
                    // c. 关于onInterceptTouchEvent() -&gt;&gt;分析1

                ev.setAction(MotionEvent.ACTION_DOWN);  
                final int scrolledXInt = (int) scrolledXFloat;  
                final int scrolledYInt = (int) scrolledYFloat;  
                final View[] children = mChildren;  
                final int count = mChildrenCount;  

        // 重点分析2
            // 通过for循环，遍历了当前ViewGroup下的所有子View
            for (int i = count - 1; i &gt;= 0; i--) {  
                final View child = children[i];  
                if ((child.mViewFlags &amp; VISIBILITY_MASK) == VISIBLE  
                        || child.getAnimation() != null) {  
                    child.getHitRect(frame);  

                    // 判断当前遍历的View是不是正在点击的View，从而找到当前被点击的View
                    // 若是，则进入条件判断内部
                    if (frame.contains(scrolledXInt, scrolledYInt)) {  
                        final float xc = scrolledXFloat - child.mLeft;  
                        final float yc = scrolledYFloat - child.mTop;  
                        ev.setLocation(xc, yc);  
                        child.mPrivateFlags &amp;= ~CANCEL_NEXT_UP_EVENT;  

                        // 条件判断的内部调用了该View的dispatchTouchEvent()
                        // 即 实现了点击事件从ViewGroup到子View的传递（具体请看下面的View事件分发机制）
                        if (child.dispatchTouchEvent(ev))  { 

                        mMotionTarget = child;  
                        return true; 
                        // 调用子View的dispatchTouchEvent后是有返回值的
                        // 若该控件可点击，那么点击时，dispatchTouchEvent的返回值必定是true，因此会导致条件判断成立
                        // 于是给ViewGroup的dispatchTouchEvent（）直接返回了true，即直接跳出
                        // 即把ViewGroup的点击事件拦截掉

                                }  
                            }  
                        }  
                    }  
                }  
            }  
            boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||  
                    (action == MotionEvent.ACTION_CANCEL);  
            if (isUpOrCancel) {  
                mGroupFlags &amp;= ~FLAG_DISALLOW_INTERCEPT;  
            }  
            final View target = mMotionTarget;  

        // 重点分析3
        // 若点击的是空白处（即无任何View接收事件） / 拦截事件（手动复写onInterceptTouchEvent（），从而让其返回true）
        if (target == null) {  
            ev.setLocation(xf, yf);  
            if ((mPrivateFlags &amp; CANCEL_NEXT_UP_EVENT) != 0) {  
                ev.setAction(MotionEvent.ACTION_CANCEL);  
                mPrivateFlags &amp;= ~CANCEL_NEXT_UP_EVENT;  
            }  
            
            return super.dispatchTouchEvent(ev);
            // 调用ViewGroup父类的dispatchTouchEvent()，即View.dispatchTouchEvent()
            // 因此会执行ViewGroup的onTouch() -&gt;&gt; onTouchEvent() -&gt;&gt; performClick（） -&gt;&gt; onClick()，即自己处理该事件，事件不会往下传递（具体请参考View事件的分发机制中的View.dispatchTouchEvent（））
            // 此处需与上面区别：子View的dispatchTouchEvent（）
        } 

        ... 

}
/**
  * 分析1：ViewGroup.onInterceptTouchEvent()
  * 作用：是否拦截事件
  * 说明：
  *     a. 返回true = 拦截，即事件停止往下传递（需手动设置，即复写onInterceptTouchEvent（），从而让其返回true）
  *     b. 返回false = 不拦截（默认）
  */
  public boolean onInterceptTouchEvent(MotionEvent ev) {  
    
    return false;

  } 
  // 回到调用原处

</code></pre>
<h3 id="总结-2">2.2.2 总结</h3>
<ul>
<li>结论：<code>Android</code>事件分发总是先传递到<code>ViewGroup</code>、再传递到<code>View</code></li>
<li>过程：当点击了某个控件时</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-6ec2e864af7ffd37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/713/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>核心方法总结</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-ff627fea1a2244ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="demo讲解">2.2.3 Demo讲解</h3>
<ul>
<li>
<p>布局如下</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-b0bf3dd7ad41b335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/210/format/webp" alt=""></p>
<p>布局层次</p>
</li>
<li>
<p>测试代码</p>
</li>
</ul>
<p>布局文件：<em>activity_main.xml</em></p>
<pre><code>&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/my_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:focusableInTouchMode="true"
    android:orientation="vertical"&gt;

    &lt;Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮1" /&gt;

    &lt;Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮2" /&gt;

&lt;/LinearLayout&gt;


</code></pre>
<p>核心代码：<em>MainActivity.java</em></p>
<pre><code>/**
  * ViewGroup布局（myLayout）中有2个子View = 2个按钮
  */
    public class MainActivity extends AppCompatActivity {

    Button button1,button2;
    ViewGroup myLayout;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button1 = (Button)findViewById(R.id.button1);
        button2 = (Button)findViewById(R.id.button2);
        myLayout = (LinearLayout)findViewById(R.id.my_layout);

        // 1.为ViewGroup布局设置监听事件
        myLayout.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("TAG", "点击了ViewGroup");
            }
        });

        // 2. 为按钮1设置监听事件
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("TAG", "点击了button1");
            }
        });

        // 3. 为按钮2设置监听事件
        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d("TAG", "点击了button2");
            }
        });

    }
}

</code></pre>
<ul>
<li>
<p>结果测试</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-a9c45aa25d12b589.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/987/format/webp" alt=""></p>
<p>示意图</p>
</li>
</ul>
<p>从上面的测试结果发现：</p>
<ul>
<li>点击<code>Button</code>时，执行<code>Button.onClick()</code>，但<code>ViewGroupLayout</code>注册的<code>onTouch（）</code>不会执行</li>
<li>只有点击空白区域时，才会执行<code>ViewGroupLayout</code>的<code>onTouch（）</code></li>
<li>结论：<code>Button</code>的<code>onClick()</code>将事件消费掉了，因此事件不会再继续向下传递。</li>
</ul>
<hr>
<h1 id="view事件的分发机制">2.3 View事件的分发机制</h1>
<p>从上面<code>ViewGroup</code>事件分发机制知道，<code>View</code>事件分发机制从<code>dispatchTouchEvent()</code>开始</p>
<h3 id="源码分析-2">2.3.1 源码分析</h3>
<pre><code>/**
  * 源码分析：View.dispatchTouchEvent（）
  */
  public boolean dispatchTouchEvent(MotionEvent event) {  

        if (mOnTouchListener != null &amp;&amp; (mViewFlags &amp; ENABLED_MASK) == ENABLED &amp;&amp;  
                mOnTouchListener.onTouch(this, event)) {  
            return true;  
        } 
        return onTouchEvent(event);  
  }
  // 说明：只有以下3个条件都为真，dispatchTouchEvent()才返回true；否则执行onTouchEvent()
  //     1. mOnTouchListener != null
  //     2. (mViewFlags &amp; ENABLED_MASK) == ENABLED
  //     3. mOnTouchListener.onTouch(this, event)
  // 下面对这3个条件逐个分析


/**
  * 条件1：mOnTouchListener != null
  * 说明：mOnTouchListener变量在View.setOnTouchListener（）方法里赋值
  */
  public void setOnTouchListener(OnTouchListener l) { 

    mOnTouchListener = l;  
    // 即只要我们给控件注册了Touch事件，mOnTouchListener就一定被赋值（不为空）
        
} 

/**
  * 条件2：(mViewFlags &amp; ENABLED_MASK) == ENABLED
  * 说明：
  *     a. 该条件是判断当前点击的控件是否enable
  *     b. 由于很多View默认enable，故该条件恒定为true
  */

/**
  * 条件3：mOnTouchListener.onTouch(this, event)
  * 说明：即 回调控件注册Touch事件时的onTouch（）；需手动复写设置，具体如下（以按钮Button为例）
  */
    button.setOnTouchListener(new OnTouchListener() {  
        @Override  
        public boolean onTouch(View v, MotionEvent event) {  
     
            return false;  
        }  
    });
    // 若在onTouch（）返回true，就会让上述三个条件全部成立，从而使得View.dispatchTouchEvent（）直接返回true，事件分发结束
    // 若在onTouch（）返回false，就会使得上述三个条件不全部成立，从而使得View.dispatchTouchEvent（）中跳出If，执行onTouchEvent(event)

</code></pre>
<p>接下来，我们继续看：**onTouchEvent(event)**的源码分析</p>
<blockquote>
<ol>
<li>详情请看注释</li>
<li><code>Android 5.0</code>后 <code>View.onTouchEvent()</code>源码发生了变化（更加复杂），但原理相同；</li>
<li>本文为了让读者更好理解，所以采用<code>Android 5.0</code>前的版本</li>
</ol>
</blockquote>
<pre><code>/**
  * 源码分析：View.onTouchEvent（）
  */
  public boolean onTouchEvent(MotionEvent event) {  
    final int viewFlags = mViewFlags;  

    if ((viewFlags &amp; ENABLED_MASK) == DISABLED) {  
         
        return (((viewFlags &amp; CLICKABLE) == CLICKABLE ||  
                (viewFlags &amp; LONG_CLICKABLE) == LONG_CLICKABLE));  
    }  
    if (mTouchDelegate != null) {  
        if (mTouchDelegate.onTouchEvent(event)) {  
            return true;  
        }  
    }  

    // 若该控件可点击，则进入switch判断中
    if (((viewFlags &amp; CLICKABLE) == CLICKABLE ||  
            (viewFlags &amp; LONG_CLICKABLE) == LONG_CLICKABLE)) {  

                switch (event.getAction()) { 

                    // a. 若当前的事件 = 抬起View（主要分析）
                    case MotionEvent.ACTION_UP:  
                        boolean prepressed = (mPrivateFlags &amp; PREPRESSED) != 0;  

                            ...// 经过种种判断，此处省略

                            // 执行performClick() -&gt;&gt;分析1
                            performClick();  
                            break;  

                    // b. 若当前的事件 = 按下View
                    case MotionEvent.ACTION_DOWN:  
                        if (mPendingCheckForTap == null) {  
                            mPendingCheckForTap = new CheckForTap();  
                        }  
                        mPrivateFlags |= PREPRESSED;  
                        mHasPerformedLongPress = false;  
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                        break;  

                    // c. 若当前的事件 = 结束事件（非人为原因）
                    case MotionEvent.ACTION_CANCEL:  
                        mPrivateFlags &amp;= ~PRESSED;  
                        refreshDrawableState();  
                        removeTapCallback();  
                        break;

                    // d. 若当前的事件 = 滑动View
                    case MotionEvent.ACTION_MOVE:  
                        final int x = (int) event.getX();  
                        final int y = (int) event.getY();  
        
                        int slop = mTouchSlop;  
                        if ((x &lt; 0 - slop) || (x &gt;= getWidth() + slop) ||  
                                (y &lt; 0 - slop) || (y &gt;= getHeight() + slop)) {  
                            // Outside button  
                            removeTapCallback();  
                            if ((mPrivateFlags &amp; PRESSED) != 0) {  
                                // Remove any future long press/tap checks  
                                removeLongPressCallback();  
                                // Need to switch from pressed to not pressed  
                                mPrivateFlags &amp;= ~PRESSED;  
                                refreshDrawableState();  
                            }  
                        }  
                        break;  
                }  
                // 若该控件可点击，就一定返回true
                return true;  
            }  
             // 若该控件不可点击，就一定返回false
            return false;  
        }

/**
  * 分析1：performClick（）
  */  
    public boolean performClick() {  

        if (mOnClickListener != null) {  
            playSoundEffect(SoundEffectConstants.CLICK);  
            mOnClickListener.onClick(this);  
            return true;  
            // 只要我们通过setOnClickListener（）为控件View注册1个点击事件
            // 那么就会给mOnClickListener变量赋值（即不为空）
            // 则会往下回调onClick（） &amp; performClick（）返回true
        }  
        return false;  
    }  

</code></pre>
<h3 id="总结-3">2.3.2 总结</h3>
<ul>
<li>每当控件被点击时：</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-76ce9e8299386729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/590/format/webp" alt=""></p>
<p>示意图</p>
<blockquote>
<p>注：<code>onTouch（）</code>的执行 先于 <code>onClick（）</code></p>
</blockquote>
<ul>
<li>核心方法总结</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-762cf45f36858bbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="demo讲解-1">2.3.3 Demo讲解</h3>
<p>下面我将用<code>Demo</code>验证上述的结论</p>
<pre><code>/**
  * 结论验证1：在回调onTouch()里返回false
  */
   // 1. 通过OnTouchListener()复写onTouch()，从而手动设置返回false
   button.setOnTouchListener(new View.OnTouchListener() {

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("执行了onTouch(), 动作是:" + event.getAction());
           
                return false;
            }
        });

    // 2. 通过 OnClickListener（）为控件设置点击事件，为mOnClickListener变量赋值（即不为空），从而往下回调onClick（）
    button.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                System.out.println("执行了onClick()");
            }

        });

/**
  * 结论验证2：在回调onTouch()里返回true
  */
   // 1. 通过OnTouchListener()复写onTouch()，从而手动设置返回true
   button.setOnTouchListener(new View.OnTouchListener() {

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("执行了onTouch(), 动作是:" + event.getAction());
           
                return true;
            }
        });

    // 2. 通过 OnClickListener（）为控件设置点击事件，为mOnClickListener变量赋值（即不为空）
    // 但由于dispatchTouchEvent（）返回true，即事件不再向下传递，故不调用onClick()）
    button.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                System.out.println("执行了onClick()");
            }
            
        });


</code></pre>
<ul>
<li>
<p>测试结果</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-97959093583369a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/904/format/webp" alt=""></p>
<p>示意图</p>
</li>
</ul>
<h1 id="总结-4">2.4 总结</h1>
<p><img src="//upload-images.jianshu.io/upload_images/944365-eeebede55f55b040.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="若您已经看到此处，那么恭喜你，你已经能非常熟悉掌握android的事件分发机制了">若您已经看到此处，那么恭喜你，你已经能非常熟悉掌握Android的事件分发机制了</h3>
<blockquote>
<p>即：<code>Activity</code>、<code>ViewGroup</code>、<code>View</code> 的事件分发机制</p>
</blockquote>
<hr>
<h1 id="工作流程-总结">3. 工作流程 总结</h1>
<ul>
<li>在本节中，我将结合源码，梳理出1个事件分发的工作流程总结，具体如下：</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-aea821bbb613c195.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<blockquote>
<p>左侧虚线：具备相关性 &amp; 逐层返回</p>
</blockquote>
<ul>
<li>以角色为核心的图解说明</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-bccafd3ff8a880ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>以方法为核心的图解说明</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-9f340a39bdad520e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<hr>
<h1 id="核心方法总结">4. 核心方法总结</h1>
<ul>
<li>
<p>已知事件分发过程的核心方法为：<code>dispatchTouchEvent()</code>、<code>onInterceptTouchEvent()</code> 和 <code>onTouchEvent()</code></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-faaf73d0f3eb870f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
</li>
<li>
<p>下面，我将结合总结的工作流程，再次详细讲解该3个方法</p>
</li>
</ul>
<h3 id="dispatchtouchevent">4.1 dispatchTouchEvent()</h3>
<ul>
<li>简介</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-4fbf11afa24b033b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-f8888a622c255648.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>返回情况说明</li>
</ul>
<p><strong>情况1：默认</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-a4582905b3972904.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/824/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-fee3555bdc6f9524.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><strong>情况2：返回true</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-b00acc9f529f5393.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-6822b1af27852dbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><strong>情况3：返回false</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-962fc440d2fffe0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-0a9ae1bc05f432b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<hr>
<h3 id="onintercepttouchevent">4.2 onInterceptTouchEvent()</h3>
<ul>
<li>简介</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-f4116863606e494e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<blockquote>
<p>注：<code>Activity</code>、<code>View</code>都无该方法</p>
</blockquote>
<p><img src="//upload-images.jianshu.io/upload_images/944365-28d0f5e7a1665148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>返回情况说明</li>
</ul>
<p><strong>情况1：true</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-9a83aed00a8c0a54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/926/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-6889eda6ebda8c40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><strong>情况2：false（默认）</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-ea06029e3176635f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/926/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-299cfcbe3a9c9fd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<hr>
<h3 id="ontouchevent">4.3 onTouchEvent()</h3>
<ul>
<li>简介</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-b5f527027257b98e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-744f5b7e8d413562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>返回情况说明</li>
</ul>
<p><strong>情况1：返回true</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-36af591c11a8e450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/824/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-aef94c6c182353a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p><strong>情况2：返回false（default）</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-efd21a46a9af808e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/824/format/webp" alt=""></p>
<p>示意图</p>
<p><img src="//upload-images.jianshu.io/upload_images/944365-5da9fe2990f75d9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="三者关系">4.4 三者关系</h3>
<p>下面，我用一段伪代码来阐述上述3个方法的关系 &amp; 事件传递规则</p>
<pre><code>/**
  * 点击事件产生后
  */ 
  // 步骤1：调用dispatchTouchEvent（）
  public boolean dispatchTouchEvent(MotionEvent ev) {

    boolean consume = false; //代表 是否会消费事件
    
    // 步骤2：判断是否拦截事件
    if (onInterceptTouchEvent(ev)) {
      // a. 若拦截，则将该事件交给当前View进行处理
      // 即调用onTouchEvent (）方法去处理点击事件
        consume = onTouchEvent (ev) ;

    } else {

      // b. 若不拦截，则将该事件传递到下层
      // 即 下层元素的dispatchTouchEvent（）就会被调用，重复上述过程
      // 直到点击事件被最终处理为止
      consume = child.dispatchTouchEvent (ev) ;
    }

    // 步骤3：最终返回通知 该事件是否被消费（接收 &amp; 处理）
    return consume;

   }

</code></pre>
<hr>
<h1 id="常见的事件分发场景">5. 常见的事件分发场景</h1>
<p>下面，我将通过实例说明<strong>常见的事件传递情况 &amp; 流程</strong></p>
<h3 id="背景描述">5.1 背景描述</h3>
<ul>
<li>讨论的布局如下：</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-e0f526dd1b5731be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/438/format/webp" alt=""></p>
<p>示意图</p>
<ul>
<li>
<p>情景</p>
<ol>
<li>用户先触摸到屏幕上<code>View C</code>上的某个点（图中黄区）</li>
</ol>
<blockquote>
<p><code>Action_DOWN</code>事件在此处产生</p>
</blockquote>
<ol start="2">
<li>用户移动手指</li>
<li>最后离开屏幕</li>
</ol>
</li>
</ul>
<h3 id="一般的事件传递情况">5.2 一般的事件传递情况</h3>
<p>一般的事件传递场景有：</p>
<ul>
<li>默认情况</li>
<li>处理事件</li>
<li>拦截<code>DOWN</code>事件</li>
<li>拦截后续事件（<code>MOVE</code>、<code>UP</code>）</li>
</ul>
<h3 id="场景1：默认">场景1：默认</h3>
<ul>
<li>
<p>即不对控件里的方法（<code>dispatchTouchEvent()</code>、<code>onTouchEvent()</code>、<code>onInterceptTouchEvent()</code>）进行重写 或 更改返回值</p>
</li>
<li>
<p>那么调用的是这3个方法的默认实现：调用下层的方法 &amp; 逐层返回</p>
</li>
<li>
<p>事件传递情况：（呈<code>U</code>型）</p>
<ol>
<li>从上往下调用dispatchTouchEvent()</li>
</ol>
<blockquote>
<p>Activity A -&gt;&gt; ViewGroup B -&gt;&gt; View C</p>
</blockquote>
<ol start="2">
<li>从下往上调用onTouchEvent()</li>
</ol>
<blockquote>
<p>View C -&gt;&gt; ViewGroup B -&gt;&gt; Activity A</p>
</blockquote>
</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-161a6e6fc8723248.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<blockquote>
<p>注：虽然<code>ViewGroup B</code>的<code>onInterceptTouchEvent</code>（）对<code>DOWN</code>事件返回了<code>false</code>，但后续的事件<code>（MOVE、UP）</code>依然会传递给它的<code>onInterceptTouchEvent()</code><br>
这一点与<code>onTouchEvent（）</code>的行为是不一样的：不再传递 &amp; 接收该事件列的其他事件</p>
</blockquote>
<h3 id="场景2：处理事件">场景2：处理事件</h3>
<p>设<code>View C</code>希望处理该点击事件，即：设置<code>View C</code>为可点击的<code>（Clickable）</code> 或 复写其<code>onTouchEvent（）</code>返回<code>true</code></p>
<blockquote>
<p>最常见的：设置<code>Button</code>按钮来响应点击事件</p>
</blockquote>
<p>事件传递情况：（如下图）</p>
<ul>
<li><code>DOWN</code>事件被传递给C的<code>onTouchEvent</code>方法，该方法返回<code>true</code>，表示处理该事件</li>
<li>因为<code>View C</code>正在处理该事件，那么<code>DOWN</code>事件将不再往上传递给ViewGroup B 和 <code>Activity A</code>的<code>onTouchEvent()</code>；</li>
<li>该事件列的其他事件<code>（Move、Up）</code>也将传递给<code>View C</code>的<code>onTouchEvent()</code></li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-77e933eb44682777.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<blockquote>
<p>会逐层往<code>dispatchTouchEvent()</code> 返回，最终事件分发结束</p>
</blockquote>
<h3 id="场景3：拦截down事件">场景3：拦截DOWN事件</h3>
<p>假设<code>ViewGroup B</code>希望处理该点击事件，即<code>ViewGroup B</code>复写了<code>onInterceptTouchEvent()</code>返回<code>true</code>、<code>onTouchEvent()</code>返回<code>true</code><br>
事件传递情况：（如下图）</p>
<ul>
<li>
<p><code>DOWN</code>事件被传递给<code>ViewGroup B</code>的<code>onInterceptTouchEvent()</code>，该方法返回<code>true</code>，表示拦截该事件，即自己处理该事件（事件不再往下传递）</p>
</li>
<li>
<p>调用自身的<code>onTouchEvent()</code>处理事件（<code>DOWN</code>事件将不再往上传递给<code>Activity A</code>的<code>onTouchEvent()</code>）</p>
</li>
<li>
<p>该事件列的其他事件<code>（Move、Up）</code>将直接传递给<code>ViewGroup B</code>的<code>onTouchEvent()</code></p>
</li>
</ul>
<blockquote>
<p>注：</p>
<ol>
<li>该事件列的其他事件<code>（Move、Up）</code>将不会再传递给<code>ViewGroup B</code>的<code>onInterceptTouchEvent</code>（）；因：该方法一旦返回一次<code>true</code>，就再也不会被调用</li>
<li>逐层往<code>dispatchTouchEvent()</code> 返回，最终事件分发结束</li>
</ol>
</blockquote>
<p><img src="//upload-images.jianshu.io/upload_images/944365-a5e7cfed2cba02c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<h3 id="场景4：拦截down的后续事件">场景4：拦截DOWN的后续事件</h3>
<p><strong>结论</strong></p>
<ul>
<li>若 <code>ViewGroup</code> 拦截了一个半路的事件（如<code>MOVE</code>），该事件将会被系统变成一个<code>CANCEL</code>事件 &amp; 传递给之前处理该事件的子<code>View</code>；</li>
<li>该事件不会再传递给<code>ViewGroup</code> 的<code>onTouchEvent()</code></li>
<li>只有再到来的事件才会传递到<code>ViewGroup</code>的<code>onTouchEvent()</code></li>
</ul>
<p><strong>场景描述</strong><br>
<code>ViewGroup B</code> 无拦截<code>DOWN</code>事件（还是<code>View C</code>来处理<code>DOWN</code>事件），但它拦截了接下来的<code>MOVE</code>事件</p>
<blockquote>
<p>即 <code>DOWN</code>事件传递到<code>View C</code>的<code>onTouchEvent（）</code>，返回了<code>true</code></p>
</blockquote>
<p><strong>实例讲解</strong></p>
<ul>
<li>在后续到来的MOVE事件，<code>ViewGroup B</code> 的<code>onInterceptTouchEvent（）</code>返回<code>true</code>拦截该<code>MOVE</code>事件，但该事件并没有传递给<code>ViewGroup B</code> ；这个<code>MOVE</code>事件将会被系统变成一个<code>CANCEL</code>事件传递给<code>View C</code>的<code>onTouchEvent（）</code></li>
<li>后续又来了一个<code>MOVE</code>事件，该<code>MOVE</code>事件才会直接传递给<code>ViewGroup B</code> 的<code>onTouchEvent()</code></li>
</ul>
<blockquote>
<p>后续事件将直接传递给<code>ViewGroup B</code> 的<code>onTouchEvent()</code>处理，而不会再传递给<code>ViewGroup B</code> 的<code>onInterceptTouchEvent（）</code>，因该方法一旦返回一次true，就再也不会被调用了。</p>
</blockquote>
<ul>
<li><code>View C</code>再也不会收到该事件列产生的后续事件</li>
</ul>
<p><img src="//upload-images.jianshu.io/upload_images/944365-1599f532038686cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>示意图</p>
<p>至此，关于<code>Android</code>常见的事件传递情况 &amp; 流程已经讲解完毕。</p>
<hr>
<h1 id="额外知识">6. 额外知识</h1>
<h3 id="touch事件的后续事件（move、up）层级传递">6.1 Touch事件的后续事件（MOVE、UP）层级传递</h3>
<ul>
<li>若给控件注册了<code>Touch</code>事件，每次点击都会触发一系列<code>action</code>事件（ACTION_DOWN，ACTION_MOVE，ACTION_UP等）</li>
<li>当<code>dispatchTouchEvent（）</code>事件分发时，只有前一个事件（如ACTION_DOWN）返回true，才会收到后一个事件（ACTION_MOVE和ACTION_UP）</li>
</ul>
<blockquote>
<p>即如果在执行ACTION_DOWN时返回false，后面一系列的ACTION_MOVE、ACTION_UP事件都不会执行</p>
</blockquote>
<p>从上面对事件分发机制分析知：</p>
<ul>
<li>dispatchTouchEvent()、 onTouchEvent() 消费事件、终结事件传递（返回true）</li>
<li>而onInterceptTouchEvent 并不能消费事件，它相当于是一个分叉口起到分流导流的作用，对后续的ACTION_MOVE和ACTION_UP事件接收起到非常大的作用</li>
</ul>
<blockquote>
<p>请记住：接收了ACTION_DOWN事件的函数不一定能收到后续事件（ACTION_MOVE、ACTION_UP）</p>
</blockquote>
<p><strong>这里给出ACTION_MOVE和ACTION_UP事件的传递结论</strong>：</p>
<ul>
<li>结论1<br>
若对象（Activity、ViewGroup、View）的dispatchTouchEvent()分发事件后消费了事件（返回true），那么收到ACTION_DOWN的函数也能收到ACTION_MOVE和ACTION_UP</li>
</ul>
<blockquote>
<p>黑线：ACTION_DOWN事件传递方向<br>
红线：ACTION_MOVE 、 ACTION_UP事件传递方向</p>
</blockquote>
<p><img src="//upload-images.jianshu.io/upload_images/944365-93d0b1496e9e6ca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>流程讲解</p>
<ul>
<li>结论2<br>
若对象（Activity、ViewGroup、View）的onTouchEvent()处理了事件（返回true），那么ACTION_MOVE、ACTION_UP的事件从上往下传到该<code>View</code>后就不再往下传递，而是直接传给自己的<code>onTouchEvent()</code>&amp; 结束本次事件传递过程。</li>
</ul>
<blockquote>
<p>黑线：ACTION_DOWN事件传递方向<br>
红线：ACTION_MOVE、ACTION_UP事件传递方向</p>
</blockquote>
<p><img src="//upload-images.jianshu.io/upload_images/944365-9d639a0b9ebf7b4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp" alt=""></p>
<p>流程讲解</p>
<h3 id="ontouch和ontouchevent的区别">6.2 onTouch()和onTouchEvent()的区别</h3>
<ul>
<li>该2个方法都是在<code>View.dispatchTouchEvent（）</code>中调用</li>
<li>但<code>onTouch（）</code>优先于<code>onTouchEvent</code>执行；若手动复写在<code>onTouch（）</code>中返回<code>true</code>（即 将事件消费掉），将不会再执行<code>onTouchEvent（）</code></li>
</ul>
<blockquote>
<p>注：若1个控件不可点击（即非<code>enable</code>），那么给它注册<code>onTouch</code>事件将永远得不到执行，具体原因看如下代码</p>
</blockquote>
<pre><code>// &amp;&amp;为短路与，即如果前面条件为false，将不再往下执行
//  故：onTouch（）能够得到执行需2个前提条件：
     // 1. mOnTouchListener的值不能为空
     // 2. 当前点击的控件必须是enable的
mOnTouchListener != null &amp;&amp; (mViewFlags &amp; ENABLED_MASK) == ENABLED &amp;&amp;  
            mOnTouchListener.onTouch(this, event)

// 对于该类控件，若需监听它的touch事件，就必须通过在该控件中重写onTouchEvent（）来实现

</code></pre>
<p>-------------------------------分割线------------------------------------------</p>
<ul>
<li>怎么拦截事件？</li>
</ul>
<blockquote>
<p>ViewGroup通过onInterceptTouchEvent()方法拦截。<br>
返回true：拦截事件，不再向下传递；<br>
返回false：不拦截，继续向下传递；</p>
</blockquote>
<ul>
<li>子View对事件的处理会有什么影响?</li>
</ul>
<blockquote>
<p>子View消费事件：不会再向ViewGroup通知；<br>
子View没有消费事件：事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent()函数；</p>
</blockquote>
<ul>
<li>子View如何防止被父View拦截?</li>
</ul>
<blockquote>
<p>子View可调用requestDisallowInterceptTouchEvent方法，来设置disallowIntercept=true，从而阻止父ViewGroup的onInterceptTouchEvent拦截操作；<br>
如果View没有消费ACTION_DOWN事件，则之后的ACTION_MOVE等事件都不会再接收。</p>
</blockquote>
<ul>
<li>onTouch与onTouchEvent区别？</li>
</ul>
<blockquote>
<p>onTouch能够得到执行需要两个前提条件，第一mOnTouchListener的值不能为空，第二当前点击的控件必须是enable的。</p>
</blockquote>
<blockquote>
<p>onTouch优先于onTouchEvent执行。如果在onTouch方法中通过返回true将事件消费掉，onTouchEvent将不会再执行。</p>
</blockquote>
<ul>
<li>onTouch与onClick关系</li>
</ul>
<blockquote>
<p>onClick事件藏在onTouchEvent事件的ACTION_UP中，也就是标示的performClick，这样结合上面onTouch与onTouchEvent事件的关系，可以很容易得到：<br>
① 触摸事件先执行（onTouch&gt;onClick）；<br>
② 触摸事件返回值影响点击事件（前者影响后者，而后者不影响前者）；<br>
③ onTouch方法的返回值改为true时，只执行onTouch事件，不执行onClick事件，当然也不执行onTouchEvent事件。</p>
</blockquote>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

