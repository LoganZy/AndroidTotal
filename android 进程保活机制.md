---


---

<h2 id="保活手段">保活手段</h2>
<p>当前业界的Android进程保活手段主要分为** 黑、白、灰 **三种，其大致的实现思路如下：</p>
<p><strong>黑色保活</strong>：不同的app进程，用广播相互唤醒（包括利用系统提供的广播进行唤醒）</p>
<p><strong>白色保活</strong>：启动前台Service</p>
<p><strong>灰色保活</strong>：利用<strong>系统的漏洞</strong>启动前台Service</p>
<h2 id="黑色保活">黑色保活</h2>
<p>所谓黑色保活，就是利用不同的app进程使用广播来进行相互唤醒。举个3个比较常见的场景：</p>
<p><strong>场景1</strong>：开机，网络切换、拍照、拍视频时候，利用系统产生的广播唤醒app</p>
<p><strong>场景2</strong>：接入第三方SDK也会唤醒相应的app进程，如微信sdk会唤醒微信，支付宝sdk会唤醒支付宝。由此发散开去，就会直接触发了下面的 <strong>场景3</strong></p>
<p><strong>场景3</strong>：假如你手机里装了支付宝、淘宝、天猫、UC等阿里系的app，那么你打开任意一个阿里系的app后，有可能就顺便把其他阿里系的app给唤醒了。（只是拿阿里打个比方，其实BAT系都差不多）</p>
<p><strong>没错，我们的Android手机就是一步一步的被上面这些场景给拖卡机的。</strong></p>
<p>针对<strong>场景1</strong>，估计Google已经开始意识到这些问题，所以在最新的Android N取消了 ACTION_NEW_PICTURE（拍照），ACTION_NEW_VIDEO（拍视频），CONNECTIVITY_ACTION（网络切换）等三种广播，无疑给了很多app沉重的打击。我猜他们的心情是下面这样的</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-2904dab833d2f189.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/260/format/webp" alt=""></p>
<p>而开机广播的话，记得有一些定制ROM的厂商早已经将其去掉。</p>
<p>针对<strong>场景2</strong>和<strong>场景3</strong>，因为调用SDK唤醒app进程属于正常行为，此处不讨论。但是在借助LBE分析app之间的唤醒路径的时候，发现了两个问题：</p>
<ol>
<li>很多推送SDK也存在唤醒app的功能</li>
<li>app之间的唤醒路径真是多，且错综复杂</li>
</ol>
<p>我把自己使用的手机测试结果给大家围观一下（<strong>我的手机是小米4C，刷了原生的Android5.1系统，且已经获得Root权限才能查看这些唤醒路径</strong>）</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-93b6d8dd13047ef9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/392/format/webp" alt=""></p>
<p>15组相互唤醒路径</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-82fba2826b224df0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/393/format/webp" alt=""></p>
<p>全部唤醒路径</p>
<p>我们直接点开 <strong>简书</strong> 的唤醒路径进行查看</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-4a8629fbce1a5aed.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/391/format/webp" alt=""></p>
<p>简书唤醒路径</p>
<p>可以看到以上3条唤醒路径，但是涵盖的唤醒应用总数却达到了23+43+28款，数目真心惊人。请注意，这只是我手机上一款app的唤醒路径而已，到了这里是不是有点细思极恐。</p>
<p>当然，这里依然存在一个疑问，就是LBE分析这些唤醒路径和互相唤醒的应用是基于什么思路，我们不得而知。所以我们也无法确定其分析结果是否准确，如果有LBE的童鞋看到此文章，不知可否告知一下思路呢？但是，手机打开一个app就唤醒一大批，我自己可是亲身体验到这种酸爽的…</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-93e0875de1708ac5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/559/format/webp" alt=""></p>
<h2 id="白色保活">白色保活</h2>
<p>白色保活手段非常简单，就是调用系统api启动一个前台的Service进程，这样会在系统的通知栏生成一个Notification，用来让用户知道有这样一个app在运行着，哪怕当前的app退到了后台。如下方的LBE和QQ音乐这样：</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-3cc2af629b5f9827.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/389/format/webp" alt=""></p>
<h2 id="灰色保活">灰色保活</h2>
<p>灰色保活，这种保活手段是应用范围最广泛。它是利用系统的漏洞来启动一个前台的Service进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个Notification，看起来就如同运行着一个后台Service进程一样。这样做带来的好处就是，用户无法察觉到你运行着一个前台进程（因为看不到Notification）,但你的进程优先级又是高于普通后台进程的。那么如何利用系统的漏洞呢，大致的实现思路和代码如下：</p>
<ul>
<li>思路一：API &lt; 18，启动前台Service时直接传入new Notification()；</li>
<li>思路二：API &gt;= 18，同时启动两个id相同的前台Service，然后再将后启动的Service做stop处理；</li>
</ul>
<pre><code>
public class GrayService extends Service {

    private final static int GRAY_SERVICE_ID = 1001;

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        if (Build.VERSION.SDK_INT &lt; 18) {
            startForeground(GRAY_SERVICE_ID, new Notification());//API &lt; 18 ，此方法能有效隐藏Notification上的图标
        } else {
            Intent innerIntent = new Intent(this, GrayInnerService.class);
            startService(innerIntent);
            startForeground(GRAY_SERVICE_ID, new Notification());
        }

        return super.onStartCommand(intent, flags, startId);
    }

    ...
    ...

    /**
     * 给 API &gt;= 18 的平台上用的灰色保活手段
     */
    public static class GrayInnerService extends Service {

        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            startForeground(GRAY_SERVICE_ID, new Notification());
            stopForeground(true);
            stopSelf();
            return super.onStartCommand(intent, flags, startId);
        }

    }
}


</code></pre>
<p>代码大致就是这样，能让你神不知鬼不觉的启动着一个前台Service。其实市面上很多app都用着这种灰色保活的手段，什么？你不信？好吧，我们来验证一下。流程很简单，打开一个app，看下系统通知栏有没有一个 Notification，如果没有，我们就进入手机的adb shell模式，然后输入下面的shell命令</p>
<pre><code>dumpsys activity services PackageName

</code></pre>
<p>打印出指定包名的所有进程中的Service信息，看下有没有 <strong>isForeground=true</strong> 的关键信息。如果通知栏没有看到属于app的 Notification 且又看到 <strong>isForeground=true</strong> 则说明了，此app利用了这种灰色保活的手段。</p>
<p>下面分别是我手机上微信、qq、支付宝、陌陌的测试结果，大家有兴趣也可以自己验证一下。</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-01201b49f73277e4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/876/format/webp" alt=""></p>
<p>微信</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-9aaa4e98a2d19dda.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/802/format/webp" alt=""></p>
<p>手Q</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-e7cb3aad386c6ef5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/950/format/webp" alt=""></p>
<p>支付宝</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-7d4685e4cb88d6d6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/947/format/webp" alt=""></p>
<p>陌陌</p>
<p>其实Google察觉到了此漏洞的存在，并逐步进行封堵。这就是为什么这种保活方式分 API &gt;= 18 和 API &lt; 18 两种情况，从Android5.0的ServiceRecord类的postNotification函数源代码中可以看到这样的一行注释</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-886c3f690e4f06d6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/624/format/webp" alt=""></p>
<p>当某一天 API &gt;= 18 的方案也失效的时候，我们就又要另谋出路了。需要注意的是，**使用灰色保活并不代表着你的Service就永生不死了，只能说是提高了进程的优先级。如果你的app进程占用了大量的内存，按照回收进程的策略，同样会干掉你的app。**感兴趣于灰色保活是如何利用系统漏洞不显示 Notification 的童鞋，可以研究一下系统的 ServiceRecord、NotificationManagerService 等相关源代码，因为不是本文的重点，所以不做详述。</p>
<h2 id="唠叨的分割线">唠叨的分割线</h2>
<p>到这里基本就介绍完了** 黑、白、灰 **三种实现方式，仅仅从代码层面去讲保活是不够的，我希望能够通过系统的进程回收机制来理解保活，这样能够让我们更好的避免踩到进程被杀的坑。</p>
<h2 id="进程回收机制">进程回收机制</h2>
<p>熟悉Android系统的童鞋都知道，系统出于体验和性能上的考虑，app在退到后台时系统并不会真正的kill掉这个进程，而是将其缓存起来。打开的应用越多，后台缓存的进程也越多。在系统内存不足的情况下，系统开始依据自身的一套进程回收机制来判断要kill掉哪些进程，以腾出内存来供给需要的app。这套杀进程回收内存的机制就叫 <strong>Low Memory Killer</strong> ，它是基于Linux内核的 **OOM Killer（Out-Of-Memory killer）**机制诞生。</p>
<p>了解完 <strong>Low Memory Killer</strong>，再科普一下<strong>oom_adj</strong>。什么是<strong>oom_adj</strong>？它是linux内核分配给每个系统进程的一个值，代表进程的优先级，进程回收机制就是根据这个优先级来决定是否进行回收。对于<strong>oom_adj</strong>的作用，你只需要记住以下几点即可：</p>
<ul>
<li><strong>进程的oom_adj越大，表示此进程优先级越低，越容易被杀回收；越小，表示进程优先级越高，越不容易被杀回收</strong></li>
<li><strong>普通app进程的oom_adj&gt;=0,系统进程的oom_adj才可能&lt;0</strong></li>
</ul>
<p>那么我们如何查看进程的<strong>oom_adj</strong>值呢，需要用到下面的两个shell命令</p>
<pre><code>ps | grep PackageName //获取你指定的进程信息

</code></pre>
<p><img src="//upload-images.jianshu.io/upload_images/912181-5a244b4256260e76.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/576/format/webp" alt=""></p>
<p>这里是以我写的demo代码为例子，红色圈中部分别为下面三个进程的ID</p>
<p>UI进程：<strong>com.clock.daemon</strong><br>
普通后台进程：<strong>com.clock.daemon:bg</strong><br>
灰色保活进程：<strong>com.clock.daemon:gray</strong></p>
<p>当然，这些进程的id也可以通过AndroidStudio获得</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-5ae05cdd97ded04e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/488/format/webp" alt=""></p>
<p>接着我们来再来获取三个进程的<strong>oom_adj</strong></p>
<pre><code>cat /proc/进程ID/oom_adj

</code></pre>
<p><img src="//upload-images.jianshu.io/upload_images/912181-eb170317ab201e22.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/554/format/webp" alt=""></p>
<p>从上图可以看到UI进程和灰色保活Service进程的<strong>oom_adj=0</strong>，而普通后台进程<strong>oom_adj=15</strong>。到这里估计你也能明白，**为什么普通的后台进程容易被回收，而前台进程则不容易被回收了吧。**但明白这个还不够，接着看下图</p>
<p><img src="//upload-images.jianshu.io/upload_images/912181-b54d0a1eb4da6785.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/556/format/webp" alt=""></p>
<p>上面是我把app切换到后台，再进行一次<strong>oom_adj</strong>的检验，你会发现UI进程的值从0变成了6,而灰色保活的Service进程则从0变成了1。这里可以观察到，<strong>app退到后台时，其所有的进程优先级都会降低。但是UI进程是降低最为明显的，因为它占用的内存资源最多，系统内存不足的时候肯定优先杀这些占用内存高的进程来腾出资源。所以，为了尽量避免后台UI进程被杀，需要尽可能的释放一些不用的资源，尤其是图片、音视频之类的</strong>。</p>
<p>从Android官方文档中，我们也能看到优先级从高到低列出了这些不同类型的进程：<strong>Foreground process</strong>、<strong>Visible process</strong>、<strong>Service process</strong>、<strong>Background process</strong>、<strong>Empty process</strong>。而这些进程的oom_adj分别是多少，又是如何挂钩起来的呢？推荐大家阅读下面这篇文章：</p>
<p><a href="https://link.jianshu.com?t=http://www.cnblogs.com/angeldevil/archive/2013/05/21/3090872.html">http://www.cnblogs.com/angeldevil/archive/2013/05/21/3090872.html</a></p>
<h2 id="总结（文末有福利）">总结（文末有福利）</h2>
<p>絮絮叨叨写完了这么多，最后来做个小小的总结。回归到开篇提到QQ进程不死的问题，我也曾认为存在这样一种技术。可惜我把手机root后，杀掉QQ进程之后就再也起不来了。有些手机厂商把这些知名的app放入了自己的白名单中，保证了进程不死来提高用户体验（如微信、QQ、陌陌都在小米的白名单中）。如果从白名单中移除，他们终究还是和普通app一样躲避不了被杀的命运，为了尽量避免被杀，还是老老实实去做好优化工作吧。</p>
<p><strong>所以，进程保活的根本方案终究还是回到了性能优化上，进程永生不死终究是个彻头彻尾的伪命题！</strong></p>
<h2 id="补充更新-（2016-04-20）">补充更新 （2016-04-20）</h2>
<p><strong>有童鞋问，在华为的机子上发现微信和手Q的UI进程退到后台，oom_adj的值一点都没有变，是不是有什么黑科技在其中。为此，我稍稍验证了一下，验证方式就是把demo工程的包名改成手机QQ的，编译运行在华为的机子上，发现我的进程怎么杀也都是不死的，退到后台oom_adj的值同样不发生变化，而恢复原来的包名就不行了。所以，你懂的，手Q就在华为机子的白名单中。</strong></p>
<hr>
<h2 id="第二种方案">第二种方案</h2>
<p>目前市面上的应用，貌似除了微信和手Q都会比较担心被用户或者系统（厂商）杀死问题。本文对 Android 进程拉活进行一个总结。</p>
<p>Android 进程拉活包括两个层面：</p>
<ol>
<li>
<p>提供进程优先级，降低进程被杀死的概率</p>
</li>
<li>
<p>在进程被杀死后，进行拉活</p>
</li>
</ol>
<p>本文下面就从这两个方面做一下总结。</p>
<h2 id="进程的优先级">1. 进程的优先级</h2>
<p>Android 系统将尽量长时间地保持应用进程，但为了新建进程或运行更重要的进程，最终需要清除旧进程来回收内存。 为了确定保留或终止哪些进程，系统会根据进程中正在运行的组件以及这些组件的状态，将每个进程放入“重要性层次结构”中。 必要时，系统会首先消除重要性最低的进程，然后是清除重要性稍低一级的进程，依此类推，以回收系统资源。</p>
<p>进程的重要性，划分5级：</p>
<ol>
<li>
<p>前台进程 (Foreground process)</p>
</li>
<li>
<p>可见进程 (Visible process)</p>
</li>
<li>
<p>服务进程 (Service process)</p>
</li>
<li>
<p>后台进程 (Background process)</p>
</li>
<li>
<p>空进程 (Empty process)</p>
</li>
</ol>
<p><img src="https://segmentfault.com/img/remote/1460000006252478" alt=""></p>
<p>前台进程的重要性最高，依次递减，空进程的重要性最低，下面分别来阐述每种级别的进程</p>
<h4 id="前台进程-——-foreground-process">1.1 前台进程 —— Foreground process</h4>
<p>用户当前操作所必需的进程。通常在任意给定时间前台进程都为数不多。只有在内存不足以支持它们同时继续运行这一万不得已的情况下，系统才会终止它们。</p>
<ol>
<li>
<p>拥有用户正在交互的 Activity（已调用  <code>onResume()</code>）</p>
</li>
<li>
<p>拥有某个 Service，后者绑定到用户正在交互的 Activity</p>
</li>
<li>
<p>拥有正在“前台”运行的 Service（服务已调用  <code>startForeground()</code>）</p>
</li>
<li>
<p>拥有正执行一个生命周期回调的 Service（<code>onCreate()</code>、<code>onStart()</code>  或  <code>onDestroy()</code>）</p>
</li>
<li>
<p>拥有正执行其  <code>onReceive()</code>  方法的 BroadcastReceiver</p>
</li>
</ol>
<h4 id="可见进程-——-visible-process">1.2 可见进程 —— Visible process</h4>
<p>没有任何前台组件、但仍会影响用户在屏幕上所见内容的进程。可见进程被视为是极其重要的进程，除非为了维持所有前台进程同时运行而必须终止，否则系统不会终止这些进程。</p>
<ol>
<li>
<p>拥有不在前台、但仍对用户可见的 Activity（已调用  <code>onPause()</code>）</p>
</li>
<li>
<p>拥有绑定到可见（或前台）Activity 的 Service</p>
</li>
</ol>
<h4 id="服务进程-——-service-process">1.3 服务进程 —— Service process</h4>
<p>尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。</p>
<p>正在运行  <code>startService()</code>  方法启动的服务，且不属于上述两个更高类别进程的进程。</p>
<h4 id="后台进程-——-background-process">1.4 后台进程 —— Background process</h4>
<p>后台进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU 列表中，以确保包含用户最近查看的  <code>Activity</code>  的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。</p>
<p>对用户不可见的 Activity 的进程（已调用  <code>Activity的onStop()</code>  方法）</p>
<h4 id="空进程-——-empty-process">1.5 空进程 —— Empty process</h4>
<p>保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。</p>
<p>不含任何活动应用组件的进程</p>
<blockquote>
<p>详情参见：<a href="http://developer.android.com/intl/zh-cn/guide/components/processes-and-threads.html">http://developer.android.com/…</a></p>
</blockquote>
<h2 id="android-进程回收策略">2. Android 进程回收策略</h2>
<p>Android 中对于内存的回收，主要依靠 Lowmemorykiller 来完成，是一种根据 OOM_ADJ 阈值级别触发相应力度的内存回收的机制。</p>
<p>关于 OOM_ADJ 的说明如下：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252382" alt=""></p>
<p>其中红色部分代表比较容易被杀死的 Android 进程（OOM_ADJ&gt;=4）,绿色部分表示不容易被杀死的 Android 进程，其他表示非 Android 进程（纯 Linux 进程）。在 Lowmemorykiller 回收内存时会根据进程的级别优先杀死 OOM_ADJ 比较大的进程，对于优先级相同的进程则进一步受到进程所占内存和进程存活时间的影响。</p>
<p>Android 手机中进程被杀死可能有如下情况：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252378" alt=""></p>
<p>综上，可以得出减少进程被杀死概率无非就是想办法提高进程优先级，减少进程在内存不足等情况下被杀死的概率。</p>
<h2 id="提升进程优先级的方案">3. 提升进程优先级的方案</h2>
<h4 id="利用-activity-提升权限">3.1 利用 Activity 提升权限</h4>
<p>方案设计思想：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。注意该 Activity 需设计成用户无感知。</p>
<p>通过该方案，可以使进程的优先级在屏幕锁屏时间由4提升为最高优先级1。</p>
<p>方案适用范围：</p>
<ul>
<li>
<p>适用场景：本方案主要解决第三方应用及系统管理工具在检测到锁屏事件后一段时间（一般为5分钟以内）内会杀死后台进程，已达到省电的目的问题。</p>
</li>
<li>
<p>适用版本：适用于所有的 Android 版本。</p>
</li>
</ul>
<p>方案具体实现：首先定义 Activity，并设置 Activity 的大小为1像素：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006760368" alt=""></p>
<p>其次，从 AndroidManifest 中通过如下属性，排除 Activity 在 RecentTask 中的显示：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252405" alt=""></p>
<p>最后，控制 Activity 为透明：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252455" alt=""></p>
<p>Activity 启动与销毁时机的控制：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252537" alt=""></p>
<h4 id="利用-notification-提升权限">3.2 利用 Notification 提升权限</h4>
<p>方案设计思想：Android 中 Service 的优先级为4，通过 setForeground 接口可以将后台 Service 设置为前台 Service，使进程的优先级由4提升为2，从而使进程的优先级仅仅低于用户当前正在交互的进程，与可见进程优先级一致，使进程被杀死的概率大大降低。</p>
<p>方案实现挑战：从 Android2.3 开始调用 setForeground 将后台 Service 设置为前台 Service 时，必须在系统的通知栏发送一条通知，也就是前台 Service 与一条可见的通知时绑定在一起的。</p>
<p>对于不需要常驻通知栏的应用来说，该方案虽好，但却是用户感知的，无法直接使用。</p>
<p>方案挑战应对措施：通过实现一个内部 Service，在 LiveService 和其内部 Service 中同时发送具有相同 ID 的 Notification，然后将内部 Service 结束掉。随着内部 Service 的结束，Notification 将会消失，但系统优先级依然保持为2。</p>
<p>方案适用范围：适用于目前已知所有版本。</p>
<p>方案具体实现：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252438" alt=""></p>
<p><img src="https://segmentfault.com/img/remote/1460000006252469" alt=""></p>
<h2 id="进程死后拉活的方案">4. 进程死后拉活的方案</h2>
<h4 id="利用系统广播拉活">4.1 利用系统广播拉活</h4>
<p>方案设计思想：在发生特定系统事件时，系统会发出响应的广播，通过在 AndroidManifest 中“静态”注册对应的广播监听器，即可在发生响应事件时拉活。</p>
<p>常用的用于拉活的广播事件包括：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006252358" alt=""></p>
<p>方案适用范围：适用于全部 Android 平台。但存在如下几个缺点：</p>
<ol>
<li>
<p>广播接收器被管理软件、系统软件通过“自启管理”等功能禁用的场景无法接收到广播，从而无法自启。</p>
</li>
<li>
<p>系统广播事件不可控，只能保证发生事件时拉活进程，但无法保证进程挂掉后立即拉活。</p>
</li>
</ol>
<p>因此，该方案主要作为备用手段。</p>
<h4 id="利用第三方应用广播拉活">4.2 利用第三方应用广播拉活</h4>
<p>方案设计思想：该方案总的设计思想与接收系统广播类似，不同的是该方案为接收第三方 Top 应用广播。</p>
<p>通过反编译第三方 Top 应用，如：手机QQ、微信、支付宝、UC浏览器等，以及友盟、信鸽、个推等 SDK，找出它们外发的广播，在应用中进行监听，这样当这些应用发出广播时，就会将我们的应用拉活。</p>
<p>方案适用范围：该方案的有效程度除与系统广播一样的因素外，主要受如下因素限制：</p>
<ol>
<li>
<p>反编译分析过的第三方应用的多少</p>
</li>
<li>
<p>第三方应用的广播属于应用私有，当前版本中有效的广播，在后续版本随时就可能被移除或被改为不外发。</p>
</li>
</ol>
<p>这些因素都影响了拉活的效果。</p>
<h4 id="利用系统service机制拉活">4.3 利用系统Service机制拉活</h4>
<p>方案设计思想：将 Service 设置为 START_STICKY，利用系统机制在 Service 挂掉后自动拉活：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006760369" alt=""></p>
<p>方案适用范围：如下两种情况无法拉活</p>
<ol>
<li>
<p>Service 第一次被异常杀死后会在5秒内重启，第二次被杀死会在10秒内重启，第三次会在20秒内重启，一旦在短时间内 Service 被杀死达到5次，则系统不再拉起。</p>
</li>
<li>
<p>进程被取得 Root 权限的管理工具或系统工具通过 forestop 停止掉，无法重启。</p>
</li>
</ol>
<h4 id="利用native进程拉活">4.4 利用Native进程拉活</h4>
<p>方案设计思想：</p>
<ul>
<li>
<p>主要思想：利用 Linux 中的 fork 机制创建 Native 进程，在 Native 进程中监控主进程的存活，当主进程挂掉后，在 Native 进程中立即对主进程进行拉活。</p>
</li>
<li>
<p>主要原理：在 Android 中所有进程和系统组件的生命周期受 ActivityManagerService 的统一管理。而且，通过 Linux 的 fork 机制创建的进程为纯 Linux 进程，其生命周期不受 Android 的管理。</p>
</li>
</ul>
<p>方案实现挑战：</p>
<ul>
<li><strong>挑战一：在 Native 进程中如何感知主进程死亡。</strong></li>
</ul>
<p>要在 Native 进程中感知主进程是否存活有两种实现方式：</p>
<ol>
<li>
<p>在 Native 进程中通过死循环或定时器，轮训判断主进程是否存活，档主进程不存活时进行拉活。该方案的很大缺点是不停的轮询执行判断逻辑，非常耗电。</p>
</li>
<li>
<p>在主进程中创建一个监控文件，并且在主进程中持有文件锁。在拉活进程启动后申请文件锁将会被堵塞，一旦可以成功获取到锁，说明主进程挂掉，即可进行拉活。由于 Android 中的应用都运行于虚拟机之上，Java 层的文件锁与 Linux 层的文件锁是不同的，要实现该功能需要封装 Linux 层的文件锁供上层调用。</p>
</li>
</ol>
<p>封装 Linux 文件锁的代码如下：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253714" alt=""></p>
<p>Native 层中堵塞申请文件锁的部分代码：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253716" alt=""></p>
<ul>
<li><strong>挑战二：在 Native 进程中如何拉活主进程。</strong></li>
</ul>
<p>通过 Native 进程拉活主进程的部分代码如下，即通过 am 命令进行拉活。通过指定“–include-stopped-packages”参数来拉活主进程处于 forestop 状态的情况。</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253718" alt=""></p>
<ul>
<li><strong>挑战三：如何保证 Native 进程的唯一。</strong></li>
</ul>
<p>从可扩展性和进程唯一等多方面考虑，将 Native 进程设计层 C/S 结构模式，主进程与 Native 进程通过 Localsocket 进行通信。在Native进程中利用 Localsocket 保证 Native 进程的唯一性，不至于出现创建多个 Native 进程以及 Native 进程变成僵尸进程等问题。</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253720" alt=""></p>
<p>方案适用范围：该方案主要适用于 Android5.0 以下版本手机。</p>
<p>该方案不受 forcestop 影响，被强制停止的应用依然可以被拉活，在 Android5.0 以下版本拉活效果非常好。</p>
<p>对于 Android5.0 以上手机，系统虽然会将native进程内的所有进程都杀死，这里其实就是系统“依次”杀死进程时间与拉活逻辑执行时间赛跑的问题，如果可以跑的比系统逻辑快，依然可以有效拉起。记得网上有人做过实验，该结论是成立的，在某些 Android 5.0 以上机型有效。</p>
<h4 id="利用-jobscheduler-机制拉活">4.5 利用 JobScheduler 机制拉活</h4>
<p>方案设计思想：Android5.0 以后系统对 Native 进程等加强了管理，Native 拉活方式失效。系统在 Android5.0 以上版本提供了 JobScheduler 接口，系统会定时调用该进程以使应用进行一些逻辑操作。</p>
<p>在本项目中，我对 JobScheduler 进行了进一步封装，兼容 Android5.0 以下版本。封装后 JobScheduler 接口的使用如下：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253710" alt=""></p>
<p><img src="https://segmentfault.com/img/remote/1460000006253712" alt=""></p>
<p>方案适用范围：该方案主要适用于 Android5.0 以上版本手机。</p>
<p>该方案在 Android5.0 以上版本中不受 forcestop 影响，被强制停止的应用依然可以被拉活，在 Android5.0 以上版本拉活效果非常好。</p>
<p>仅在小米手机可能会出现有时无法拉活的问题。</p>
<h4 id="利用账号同步机制拉活">4.6 利用账号同步机制拉活</h4>
<p>方案设计思想：Android 系统的账号同步机制会定期同步账号进行，该方案目的在于利用同步机制进行进程的拉活。添加账号和设置同步周期的代码如下：</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253814" alt=""></p>
<p>该方案需要在 AndroidManifest 中定义账号授权与同步服务。</p>
<p><img src="https://segmentfault.com/img/remote/1460000006253812" alt=""></p>
<p>方案适用范围：该方案适用于所有的 Android 版本，包括被 forestop 掉的进程也可以进行拉活。</p>
<p>最新 Android 版本（Android N）中系统好像对账户同步这里做了变动，该方法不再有效。</p>
<h2 id="其他有效拉活方案">5. 其他有效拉活方案</h2>
<p>经研究发现还有其他一些系统拉活措施可以使用，但在使用时需要用户授权，用户感知比较强烈。</p>
<p>这些方案包括：</p>
<ol>
<li>
<p>利用系统通知管理权限进行拉活</p>
</li>
<li>
<p>利用辅助功能拉活，将应用加入厂商或管理软件白名单。</p>
</li>
</ol>
<p>这些方案需要结合具体产品特性来搞。</p>
<p>上面所有解释这些方案都是考虑的无 Root 的情况。</p>
<p>其他还有一些技术之外的措施，比如说应用内 Push 通道的选择：</p>
<ol>
<li>
<p>国外版应用：接入 Google 的 GCM。</p>
</li>
<li>
<p>国内版应用：根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。</p>
</li>
</ol>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>

