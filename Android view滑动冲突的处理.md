---


---

<h1 id="滑动冲突的常见场景与处理思路">滑动冲突的常见场景与处理思路</h1>
<p>文章转载自<a href="https://www.jianshu.com/p/982a83271327">https://www.jianshu.com/p/982a83271327  </a></p>
<p>当我们内外两层View都可以滑动时候，就会产生滑动冲突！</p>
<blockquote>
<p><strong>常见的滑动冲突场景：</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-91541788260cb2de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/466/format/webp" alt=""></p>
<p>滑动冲突.png</p>
</blockquote>
<blockquote>
<ul>
<li>1.<strong>外层与内层滑动方向不一致</strong>，外层ViewGroup是可以横向滑动的，内层View是可以竖向滑动的（类似ViewPager，每个页面里面是ListView）</li>
</ul>
</blockquote>
<blockquote>
<ul>
<li>2.<strong>外层与内层滑动方向一致</strong>，外层ViewGroup是可以竖向滑动的，内层View同样也是竖向滑动的（类似ScrollView包裹ListView）</li>
</ul>
</blockquote>
<blockquote>
<p>当然还有上面两种组合起来，三层或者多层嵌套产生的冲突，然而不管是多么复杂，解决的思路都是一模一样。所以遇到多层嵌套的小伙伴也不用惊慌，一层一层处理即可。</p>
</blockquote>
<p><strong>有小伙伴肯定有疑问，ViewPager带ListView并没有出现滑动冲突啊。<br>
那是因为ViewPager已经为我们处理了滑动冲突！如果我们自己定义一个水平滑动的ViewGroup内部再使用ListView，那么是一定需要处理滑动冲突的。</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-f76927fdd7f7e332.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/196/format/webp" alt=""></p>
<p>针对上面第一种场景，由于外部与内部的滑动方向不一致，那么**我们可以根据当前滑动方向，水平还是垂直来判断这个事件到底该交给谁来处理。**至于如何获得滑动方向，我们可以得到滑动过程中的两个点的坐标。一般情况下根据水平和竖直方向滑动的距离差就可以判断方向，当然也可以根据滑动路径形成的夹角（或者说是斜率如下图）、水平和竖直方向滑动速度差来判断。</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-b1115e7d6a60a8ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/553/format/webp" alt=""></p>
<p>ViewPager当斜率小于0.5时判断为横向滑动，拦截事件</p>
<p>有兴趣的小伙伴可以看<a href="https://link.jianshu.com?t=http://blog.csdn.net/huachao1001/article/details/51654692">ViewPager源码分析（2）：滑动及冲突处理</a></p>
<p>针对第二种场景，由于外部与内部的滑动方向一致，那么不能根据滑动角度、距离差或者速度差来判断。**这种情况下必需通过业务逻辑来进行判断。**比较常见ScrollView嵌套了ListView。虽然需求不同，业务逻辑自然也不同，但是解决滑动冲突的方式都是一样的。下面为大家截取了微博和天猫当中的同方向滑动冲突场景，方便大家更直观的感受这个场景。</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-155e49411adadae0.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/270/format/webp" alt=""></p>
<p>同方向，竖向滑动冲突</p>
<p>微博的这个是同方向，竖向滑动冲突的场景，可以看到发现布局整体是可以滚动的，而且下方的微博列表也是可以滚动的。根据业务逻辑，当热门，榜单…这一行标签栏滑动到顶部的时候微博列表才可以滚动。否则就是发现布局的整体滚动。这个场景是不是在很多app里面都能够见到呢！</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-6e1e6b34fc820d32.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/270/format/webp" alt=""></p>
<p>同方向，横向滑动冲突</p>
<p>天猫的这个是同方向，横向滑动冲突的场景，内外两层都是可以横向滚动的。它的处理逻辑也很明显，根据用户滑动的位置来判断到底是那个View需要响应滑动。</p>
<p><strong>上述两种滑动冲突的场景区别只是在于拦截的逻辑处理上。第一种是根据水平还是竖直滑动来判断谁来处理滑动，第二种是根据业务逻辑来判断谁来处理滑动，但是处理的套路都是一样的</strong></p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-da70dfd63a38432f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/440/format/webp" alt=""></p>
<h1 id="滑动冲突解决套路">滑动冲突解决套路</h1>
<hr>
<h4 id="套路一-外部拦截法：">套路一 外部拦截法：</h4>
<p>即父View根据需要对事件进行拦截。逻辑处理放在父View的onInterceptTouchEvent方法中。我们只需要重写父View的onInterceptTouchEvent方法，并根据逻辑需要做相应的拦截即可。</p>
<pre><code>    public boolean onInterceptTouchEvent(MotionEvent event) {
        boolean intercepted = false;
        int x = (int) event.getX();
        int y = (int) event.getY();
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

</code></pre>
<p><strong>上面伪代码表示外部拦截法的处理思路，需要注意下面几点</strong></p>
<blockquote>
<ul>
<li>根据业务逻辑需要，在ACTION_MOVE方法中进行判断，如果需要父View处理则返回true，否则返回false，事件分发给子View去处理。</li>
</ul>
</blockquote>
<ul>
<li>ACTION_DOWN 一定返回false，不要拦截它，否则根据View事件分发机制，后续ACTION_MOVE 与 ACTION_UP事件都将默认交给父View去处理！</li>
<li>原则上ACTION_UP也需要返回false，如果返回true，并且滑动事件交给子View处理，那么子View将接收不到ACTION_UP事件，子View的onClick事件也无法触发。而父View不一样，如果父View在ACTION_MOVE中开始拦截事件，那么后续ACTION_UP也将默认交给父View处理！</li>
</ul>
<h4 id="套路二-内部拦截法：">套路二 内部拦截法：</h4>
<p>即父View不拦截任何事件，所有事件都传递给子View，子View根据需要决定是自己消费事件还是给父View处理。这需要子View使用requestDisallowInterceptTouchEvent方法才能正常工作。下面是子View的dispatchTouchEvent方法的伪代码：</p>
<pre><code>    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                parent.requestDisallowInterceptTouchEvent(true);
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                if (父容器需要此类点击事件) {
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

</code></pre>
<p>父View需要重写onInterceptTouchEvent方法：</p>
<pre><code>    public boolean onInterceptTouchEvent(MotionEvent event) {

        int action = event.getAction();
        if (action == MotionEvent.ACTION_DOWN) {
            return false;
        } else {
            return true;
        }
    }

</code></pre>
<blockquote>
<p><strong>使用内部拦截法需要注意：</strong></p>
<ul>
<li>内部拦截法要求父View不能拦截ACTION_DOWN事件，由于ACTION_DOWN不受FLAG_DISALLOW_INTERCEPT标志位控制，一旦父容器拦截ACTION_DOWN那么所有的事件都不会传递给子View。</li>
<li>滑动策略的逻辑放在子View的dispatchTouchEvent方法的ACTION_MOVE中，如果父容器需要获取点击事件则调用 parent.requestDisallowInterceptTouchEvent(false)方法，让父容器去拦截事件。</li>
</ul>
</blockquote>
<h1 id="滑动冲突解决示例代码">滑动冲突解决示例代码</h1>
<hr>
<p>理论最终的落脚是在实践，下面我通过一个例子来演示外部解决法和内部解决法解决滑动冲突，大家只要get到了精髓，那么今后遇到滑动冲突问题都将迎刃而解，不再是开发拦路虎！</p>
<p>我们一开始说过ViewPager已经默认给我们处理了滑动冲突，而它作为ViewGroup使用的是外部拦截法解决的冲突，即在onInterceptTouchEvent方法中进行判断的。为了造成滑动冲突场景，那么我们自定义一个ViewPager，重写onInterceptTouchEvent方法并默认返回false，如下所示：</p>
<pre><code>public class BadViewPager extends ViewPager {
    public BadViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return false;
    }
}

</code></pre>
<p>这样，一个好好的ViewPager就被我们玩坏了！</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-d02de28e8e802c97.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/198/format/webp" alt=""></p>
<p>接下来新建一个ScrollConflicActivity用来测试滑动冲突。</p>
<pre><code>public class ScrollConflictActivity extends BaseActivity {
    private BadViewPager mViewPager;
    private List&lt;View&gt; mViews;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_scroll_conflict);
        initViews();
        initData(false);
    }

    protected void initViews() {
        mViewPager = findAviewById(R.id.viewpager);
        mViews = new ArrayList&lt;&gt;();
    }

    protected void initData(final boolean isListView) {
        //初始化mViews列表
        Flowable.just("view1", "view2", "view3", "view4").subscribe(new Consumer&lt;String&gt;() {
            @Override
            public void accept(String s) throws Exception {
                //当前View
                View view;
                if (isListView) {
                    //初始化ListView
                    ListView listView = new ListView(mContext);
                    final ArrayList&lt;String&gt; datas = new ArrayList&lt;&gt;();
                    //初始化数据，datas ,data0 ...data49
                    Flowable.range(0, 50).subscribe(new Consumer&lt;Integer&gt;() {
                        @Override
                        public void accept(Integer integer) throws Exception {
                            datas.add("data" + integer);
                        }
                    });
                    //初始化adapter
                    ArrayAdapter&lt;String&gt; adapter = new ArrayAdapter&lt;&gt;
                            (mContext, android.R.layout.simple_list_item_1, datas);
                    //设置adapter
                    listView.setAdapter(adapter);
                    //将ListView赋值给当前View
                    view = listView;
                } else {
                    //初始化TextView
                    TextView textView = new TextView(mContext);
                    textView.setGravity(Gravity.CENTER);
                    textView.setText(s);
                    //将TextView赋值给当前View
                    view = textView;
                }
                //将当前View添加到ViewPager的ViewList中去
                mViews.add(view);
            }
        });
        //设置ViewPager的Adapter
        mViewPager.setAdapter(new BasePagerAdapter&lt;&gt;(mViews));
    }
}

</code></pre>
<p><strong>注：Flowable是RxJava2的方法，这里其实用for循环也是一样的<br>
BasePagerAdapter是<a href="https://www.jianshu.com/p/d5ad3f127ebf">BaseProject</a>里的方法</strong></p>
<p>上面的代码我们使用了BadViewPager，初始化了BadViewPager里面的子View。<br>
initData(false);方法传false表示里面的子View是一个TextView，传true表示里面的子View是ListView。首先我们看BadViewPager里面子View是TextView是否可以滑动。</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-72dc452b0609fcc3.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/474/format/webp" alt=""></p>
<p>BadViewPager_bad_textview.gif</p>
<p>似乎对BadViewPager的滑动没有任何影响。</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-165116734db0252d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/139/format/webp" alt=""></p>
<p>大家别急，我们来分析一下，BadViewPager的onInterceptTouchEvent默认返回false则所有事件都会给子View去处理。大家是否还记得上一篇说到的View处理事件的原则？</p>
<p><strong>View 的onTouchEvent 方法默认都会消费掉事件（返回true），除非它是不可点击的（clickable和longClickable同时为false），View的longClickable默认为false，clickable需要区分情况，如Button的clickable默认为true，而TextView的clickable默认为false。</strong></p>
<p>所以TextView默认并没有消费事件，因为他是不可点击的。事件会交由父View即BadViewPager的onTouchEvent方法去处理。所以它自然是可以滑动的。</p>
<p>我们将textview的Clickable设置成true，即让它来消费事件。大家再看看呢</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-4c24adb07dcad44a.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/474/format/webp" alt=""></p>
<p>BadViewPager_bad_textview_clickable.gif</p>
<p>所以我们不难推测如果将TextView换成Button，将是一样的无法滑动的效果。虽然这并不是常规的滑动冲突（子View不是滑动的），但是造成的原因其实是一样的，没有做滑动判断导致父View不能正确响应滑动事件。</p>
<p>接下来稍稍修改一下代码 initData(true);传入true，即BadViewPager的子View使用ListView，显然ListView是可以滑动的，BadViewPager是不能滑动的。我们分别通过外部拦截和内部拦截方法来对BadViewPager进行修复。</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-58923f4fc44d0263.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/474/format/webp" alt=""></p>
<p>BadViewPager_bad_listview.gif</p>
<h4 id="外部拦截法fix-badviewpager：">1.外部拦截法Fix BadViewPager：</h4>
<pre><code>public class BadViewPager extends ViewPager {
    private static final String TAG = "BadViewPager";

    private int mLastXIntercept;
    private int mLastYIntercept;

    public BadViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        int x = (int) ev.getX();
        int y = (int) ev.getY();
        final int action = ev.getAction() &amp; MotionEvent.ACTION_MASK;
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                //调用ViewPager的onInterceptTouchEvent方法初始化mActivePointerId
                super.onInterceptTouchEvent(ev);
                break;
            case MotionEvent.ACTION_MOVE:
                //横坐标位移增量
                int deltaX = x - mLastXIntercept;
                //纵坐标位移增量
                int deltaY = y - mLastYIntercept;
                if (Math.abs(deltaX)&gt;Math.abs(deltaY)){
                    intercepted = true;
                }else{
                    intercepted = false;
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
            default:
                break;
        }

        mLastXIntercept = x;
        mLastYIntercept = y;

        LogUtil.e(TAG,"intercepted = "+intercepted);
        return intercepted;
    }
}

</code></pre>
<p>根据我们的外部拦截法的套路，需要重写BadViewPager 的onInterceptTouchEvent方法，并且ACTION_DOWN 和 ACTION_UP返回为false。处理逻辑在ACTION_MOVE中，Math.abs(deltaX)&gt;Math.abs(deltaY)表示横向位移增量大于竖向位移增量，即水平滑动，则BadViewPager 拦截事件。</p>
<p>这里我们在ACTION_DOWN 当中还调用了super.onInterceptTouchEvent(ev);即ViewPager的onInterceptTouchEvent方法。主要是为了初始化ViewPager的成员变量mActivePointerId。mActivePointerId默认值为-1，在ViewPager的onTouchEvent方法的ACTION_MOVE中有这么一段：</p>
<pre><code>class ViewPager
    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        ...
        switch (action &amp; MotionEventCompat.ACTION_MASK) {
            case MotionEvent.ACTION_MOVE:
                if (!mIsBeingDragged) {
                    final int pointerIndex = ev.findPointerIndex(mActivePointerId);
                    if (pointerIndex == -1) {
                        // A child has consumed some touch events and put us into an inconsistent
                        // state.
                        needsInvalidate = resetTouch();
                        break;
                    }
                    //具体的滑动操作...
                }
                ...
                break;
                ...
        }
        ...
    }

</code></pre>
<p>假如mActivePointerId不进行初始化，ViewPager会认为这个事件已经被子View给消费了，然后break掉，接下来的滑动操作也就不执行了。</p>
<p><img src="//upload-images.jianshu.io/upload_images/2839355-99676fc25f07fd00.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/474/format/webp" alt=""></p>
<p>BadViewPager_fixed_listview.gif</p>
<h4 id="内部拦截法fix-badviewpager：">2.内部拦截法Fix BadViewPager：</h4>
<p>内部拦截法需要重写ListView的dispatchTouchEvent方法，所以我们自定义一个ListView：</p>
<pre><code>public class FixListView extends ListView {
    private static final String TAG = "FixListView";

    private int mLastX;
    private int mLastY;

    public FixListView(Context context) {
        super(context);
    }

    public FixListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        int x = (int) ev.getX();
        int y = (int) ev.getY();

        final int action = ev.getAction() &amp; MotionEvent.ACTION_MASK;
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                //水平移动的增量
                int deltaX = x - mLastX;
                //竖直移动的增量
                int deltaY = y - mLastY;
                //当水平增量大于竖直增量时，表示水平滑动，此时需要父View去处理事件
                if (Math.abs(deltaX) &gt; Math.abs(deltaY)){
                    getParent().requestDisallowInterceptTouchEvent(false);
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
            default:
                break;
        }
        mLastX = x;
        mLastY = y;
        return super.dispatchTouchEvent(ev);
    }
}

</code></pre>
<p>再看BadViewPager，需要重写拦截方法</p>
<pre><code>public class BadViewPager extends ViewPager {
    private static final String TAG = "BadViewPager";

    public BadViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        final int action = ev.getAction() &amp; MotionEvent.ACTION_MASK;
        if (action == MotionEvent.ACTION_DOWN){
            super.onInterceptTouchEvent(ev);
            return false;
        }
        return true;
    }
}

</code></pre>
<p>可以看到和我们的套路代码基本上一样，只是ACTION_MOVE中有我们自己的逻辑处理，处理的方式与外部拦截法也是一致的，并且效果也一样，在此不作赘述。</p>
<p><strong>大家只用掌握上述滑动冲突的解决套路，不论场景是不同方向，还是同方向，还是乱七八糟的堆加在一起，就用套路去解决，万变不离其宗。根据上述的外部拦截和内部拦截法，可以看出外部拦截法实现起来更加简单，而且也符合View的正常事件分发机制，所以推荐使用外部拦截法（重写父View的onInterceptTouchEvent，父View决定是否拦截）来处理滑动冲突</strong></p>
<p>-------------------第二篇文章----------------------</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

