---


---

<h1 id="activity生命周期再学习">Activity生命周期再学习</h1>
<h2 id="activity生命周期活动图">activity生命周期活动图</h2>
<p><a href="https://i.loli.net/2019/05/12/5cd7c1844d4bb.jpg"><img src="https://i.loli.net/2019/05/12/5cd7c1844d4bb.jpg" alt="activity生命周期.jpg"></a></p>
<h2 id="生命周期中各个方法的含义和作用">生命周期中各个方法的含义和作用</h2>
<ul>
<li>
<p><strong>onCreate:</strong> create表示创建，这是Activity生命周期的第一个方法，也是我们在android开发中接触的最多的生命周期方法。它本身的作用是进行Activity的一些初始化工作，比如使用setContentView加载布局，对一些控件和变量进行初始化等。但也有很多人将很多与初始化无关的代码放在这，其实这是不规范的。此时<strong>Activity还在后台，不可见</strong>。所以动画不应该在这里初始化，因为看不到……</p>
</li>
<li>
<p><strong>onStart:</strong> start表示启动，这是Activity生命周期的第二个方法。此时Activity已经<strong>可见</strong>了，但是还没出现在前台，我们还看不到，无法与Activity交互。其实将Activity的初始化工作放在这也没有什么问题，放在onCreate中是由于官方推荐的以及我们开发的习惯。</p>
</li>
<li>
<p><strong>onResume:</strong> resume表示继续、重新开始，这名字和它的职责也相同。此时Activity经过前两个阶段的初始化已经蓄势待发。Activity在这个阶段已经出现在前台并且<strong>可见</strong>了。这个阶段可以打开独占设备，Activity已经是处在栈顶了。</p>
</li>
<li>
<p><strong>onPause:</strong> pause表示暂停，当Activity要跳到另一个Activity或应用正常退出时都会执行这个方法。此时Activity在前台并<strong>可见</strong>，我们可以进行一些轻量级的存储数据和去初始化的工作，不能太耗时，因为在跳转Activity时只有当一个Activity执行完了onPause方法后另一个Activity才会启动，而且android中指定如果onPause在500ms即0.5秒内没有执行完毕的话就会强制关闭Activity。从生命周期图中发现可以在这快速重启，但这种情况其实很罕见，比如用户切到下一个Activity的途中按back键快速得切回来。</p>
</li>
<li>
<p><strong>onStop:</strong> stop表示停止，此时Activity已经<strong>不可见</strong>了，但是Activity对象还在内存中，没有被销毁。这个阶段的主要工作也是做一些资源的回收工作。</p>
</li>
<li>
<p><strong>onDestroy:</strong> destroy表示毁灭，这个阶段Activity被销毁，<strong>不可见</strong>，我们可以将还没释放的资源释放，以及进行一些回收工作。</p>
</li>
<li>
<p><strong>onRestart:</strong> restart表示重新开始，Activity在这时<strong>可见</strong>，当用户按Home键切换到桌面后又切回来或者从后一个Activity切回前一个Activity就会触发这个方法。这里一般不做什么操作。</p>
</li>
</ul>
<h2 id="activity各个生命周期中方法的对比">activity各个生命周期中方法的对比</h2>
<h3 id="oncreate和onstart之间有什么区别？">onCreate和onStart之间有什么区别？</h3>
<ul>
<li>(1) 可见与不可见的区别。前者不可见，后者可见。</li>
<li>(2) 执行次数的区别。onCreate方法只在Activity创建时执行一次，而onStart方法在Activity的切换以及按Home键返回桌面再切回应用的过程中被多次调用。因此Bundle数据的恢复在onStart中进行比onCreate中执行更合适。</li>
<li>(3) onCreate能做的事onStart其实都能做，但是onstart能做的事onCreate却未必适合做。如前文所说的，setContentView和资源初始化在两者都能做，然而想动画的初始化在onStart中做比较好。</li>
</ul>
<h3 id="oncreate方法和onrestoreinstancestate方法都有bundle参数，可以用来恢复数据，有什么区别？">onCreate方法和onRestoreInstanceState方法都有Bundle参数，可以用来恢复数据，有什么区别？</h3>
<p>因为onSaveInstanceState 不一定会被调用，所以onCreate()里的Bundle参数可能为空，如果使用onCreate()来恢复数据，一定要做非空判断。</p>
<p>而onRestoreInstanceState的Bundle参数一定不会是空值，因为它只有在上次activity被回收了才会调用。</p>
<p>而且onRestoreInstanceState是在onStart()之后被调用的。有时候我们需要onCreate()中做的一些初始化完成之后再恢复数据，用onRestoreInstanceState会比较方便。下面是官方文档对onRestoreInstanceState的说明：</p>
<p>This method is called after onStart() when the activity is being re-initialized from a previously saved state, given here in savedInstanceState. Most implementations will simply use onCreate(Bundle) to restore their state, but it is sometimes convenient to do it here after all of the initialization has been done or to allow subclasses to decide whether to use your default implementation.</p>
<p>注意这个说明的最后一句是什么意思？<br>
<code>to allow subclasses to decide whether to use your default implementation.</code></p>
<p>它是说，用onRestoreInstanceState方法恢复数据，你可以决定是否在方法里调用父类的onRestoreInstanceState方法，即是否调用super.onRestoreInstanceState(savedInstanceState);<br>
而用onCreate()恢复数据，你必须调用super.onCreate(savedInstanceState); 否则运行会报错误。</p>
<h3 id="onstart方法和onresume方法有什么区别？">onStart方法和onResume方法有什么区别？</h3>
<ul>
<li>(1) 是否在前台。onStart方法中Activity可见但不在前台，不可交互，而在onResume中在前台。</li>
<li>(2) 职责不同，onStart方法中主要还是进行初始化工作，而onResume方法，根据官方的建议，可以做开启动画和独占设备的操作。</li>
</ul>
<h3 id="onpause方法和onstop方法有什么区别？">onPause方法和onStop方法有什么区别？</h3>
<ul>
<li>(1) 是否可见。onPause时Activity可见，onStop时Activity不可见，但Activity对象还在内存中。</li>
<li>(2) 在系统内存不足的时候可能不会执行onStop方法，因此程序状态的保存、独占设备和动画的关闭、以及一些数据的保存最好在onPause中进行，但要注意不能太耗时。</li>
</ul>
<h3 id="onstop方法和ondestroy方法有什么区别？">onStop方法和onDestroy方法有什么区别？</h3>
<ul>
<li>onStop阶段Activity还没有被销毁，对象还在内存中，此时可以通过切换Activity再次回到该Activity，而onDestroy阶段Acivity被销毁</li>
</ul>
<h2 id="activity一些情景下生命周期的变化">activity一些情景下生命周期的变化</h2>
<blockquote>
<p><strong>情景一:</strong> FirstActivity(简称<strong>F</strong>)启动了SecondActivity(简称<strong>S</strong>)，之后按back键回退到FirstActivity，整个流程是怎样的？</p>
</blockquote>
<p>这里要分两种情况，第一种情况为，SecondActivity为透明状态的;第二种情况是正常的Activity。</p>
<ul>
<li>
<p>SecondActivity主题为透明或是非全屏的，从FirstActivity跳转到SecondActivity，因为SecondActivity为透明的，即FirstActivity在下层，仍然是可见，所以FirstActivity不会调用onStop()方法，只会调用onPause()方法， 按back退出SecondActivity，回到FirstActivity，整个流程生命周期变化如下：</p>
<blockquote>
<p>onCreate(F)–&gt;onStart(F)–&gt;onResume(F)–&gt;跳转到SecondActivity(S)–&gt;onPause(F)–&gt;onCreate(S)–&gt;onStart(S)–&gt;onResume(S)–&gt;B显示为透明，同时A在屏幕试可见的–&gt;back返回F–&gt;onPause(S)–&gt;onResume(F)–&gt;onStop(S)–&gt;onDestroy(S)</p>
</blockquote>
</li>
<li>
<p>SecondActivity主题为非透明且全屏的，即我们一般使用的默认主题，从FirstActivity跳转到SecondActivity，SecondActivity显示可见，再按back键返回FirstActivity,整个流程生命周期变化如下：</p>
<blockquote>
<p>正确的流程： onCreate(F)–&gt;onStart(F)–&gt;onResume(F)–&gt;跳转到SecondActivity(S)–&gt;onPause(F)–&gt;onWindowFocusChanged()–&gt;onCreate(S)–&gt;onStart(S)–&gt;onResume(S)–&gt;onWindowFocusChanged()–&gt;B显示，同时A在屏幕上不可见–&gt;onStop(F)–&gt;back返回F–&gt;onPause(S)–&gt;onRestart(F)–&gt;onStart(F)–&gt;onResume(F)–&gt;onStop(S)–&gt;onDestroy(S)</p>
</blockquote>
</li>
<li>
<p>(1) 一个Activity或多或少会占有系统资源，而在官方的建议中，onPause方法将会释放掉很多系统资源，为切换Activity提供流畅性的保障，而不需要再等多两个阶段，这样做切换更快。<strong>不要在onPause()中做CPU密集型操作（如文件读写），否则可能会影响下一步的操作</strong>（Activity B的创建和显示)。</p>
</li>
<li>
<p>(2) 按照生命周期图的表示，如果用户在切换Activity的过程中再次切回原Activity，是在onPause方法后直接调用onResume方法的，这样比onPause→onStop→onRestart→onStart→onResume要快得多。</p>
</li>
</ul>
<blockquote>
<p><strong>情景二:</strong> 横竖屏切换时Activity的生命周期变化?</p>
</blockquote>
<ul>
<li>如果自己没有配置android:ConfigChanges，这时默认让系统处理，就会重建Activity，此时Activity的生命周期会走一遍。整个过程生命周期的变化为：onCreate()—&gt;onStart()—&gt;onResume()—&gt;旋转屏幕—&gt;onPause()—&gt;onSaveInstanceState()—&gt;onStop()—&gt;onDestory()—&gt;onCreate()—&gt;onStart()—&gt;onRestoreInstanceState()—&gt;onResume()</li>
<li>如果配置<code>android:configChanges="orientation|keyboardHidden|screenSize"&gt;</code>,此时activity的生命周期不会重建，会调用onConfigurationChanged(),生命周期变化如下：onCreate()—&gt;onStart()—&gt;onResume()—&gt;旋转屏幕—&gt;onConfigurationChanged()</li>
</ul>
<blockquote>
<p><strong>情景三:</strong> 一个Activity从创建到销毁，是不是一定会得到这些全部生命周期的回调？</p>
</blockquote>
<p>并不一定，在两种情况下，Activity就不会得到完整的生命周期回调：</p>
<ul>
<li><strong>在onCreate()中调用Activity的finish()方法</strong>，这时候会直接进入onDestory()，而不会产生其他中间过程的回调，即onCreate()-&gt;onDestory()。这个我们可以应用到类似跳转功能的Activity中，在onCreate()进行逻辑处理后，打开目标Activity，然后finish()。这种情况下，用户不会看到这个Activity，这个Activity充当一个分发者的角色。</li>
<li>某个生命周期发生了crash，如在onCreate()中就发生了crash，就不会有下面生命周期的回调了。</li>
</ul>
<blockquote>
<p><strong>情景四：</strong> 弹窗、toast、键盘、下拉任务栏等情况，生命周期的变化。</p>
</blockquote>
<ul>
<li><code>弹窗</code>：Activity在前台状态，弹出一个dialog，activity生命周期的状态无影响；activity的样式设置为dialog，弹窗弹出，activity调用onPause(),弹窗消失，activity调用onResume()。</li>
<li><code>键盘</code>: Android下拉通知栏不会影响Activity的生命周期方法。</li>
<li><code>下拉任务栏</code>:Android下拉通知栏不会影响Activity的生命周期方法。</li>
<li><code>Toast</code>:Android下拉通知栏不会影响Activity的生命周期方法。</li>
</ul>
<h2 id="activity一些值得注意的方法的生命周期">activity一些值得注意的方法的生命周期</h2>
<p><strong>onSaveInstanceState()和onRestoreInstanceState()</strong><br>
<strong>onRestoreInstanceState()调用的时机</strong></p>
<p>只有在activity确实是被系统回收，重新创建activity的情况下才会被调用。这个方法的主要作用，是恢复在onSaveInstanceState()中保存的状态。它的调用时机在onStart()和onPostCreate()之间。</p>
<p><strong>onSaveInstanceState()调用的时机</strong></p>
<p>当activity有可能被系统回收的情况下，注意是有可能，如果是已经确定会被销毁，比如用户按下了返回键，或者调用了finish()方法销毁activity，则onSaveInstanceState不会被调用。总结下，onSaveInstanceState(Bundle outState)会在以下情况被调用：</p>
<blockquote>
<p>1、当用户按下HOME键时。<br>
2、从最近应用中选择运行其他的程序时。<br>
3、按下电源按键（关闭屏幕显示）时。<br>
4、从当前activity启动一个新的activity时。<br>
5、屏幕方向切换时(无论竖屏切横屏还是横屏切竖屏都会调用)。</p>
</blockquote>
<p>官方文档中介绍–这个方法的调用时机，在不同的系统中不同，从Android P开始，它是在onStop之后调用的，在之前的系统中，则是在onStop之前调用的。但是是否发生在onPause前后，则看具体情况。(目前为止，我遇到的都是在onPause()之后调用的)。</p>
<blockquote>
<p>事例一：activity的<code>android:configChanges</code>属性没有配置，发生屏幕方向切换时，activity生命周期如下：<br>
<strong>onPause()-&gt; onSaveInstanceState() -&gt; onStop() -&gt; onDestroy() -&gt; onCreate() -&gt; onStart() -&gt; onRestoreInstanceState() -&gt; onResume()</strong><br>
事例二：按HOME键返回桌面，又马上点击应用图标回到原来页面时，activity生命周期如下：<br>
<strong>onPause() -&gt; onSaveInstanceState() -&gt; onStop() -&gt; onRestart() -&gt; onStart()-&gt; onResume()</strong></p>
</blockquote>
<p>从上面的两个事例中，我们可以看出<code>onSaveInstanceState()和onRestoreInstanceState()</code>不一定是成对被调用的，即onRestoreInstanceState()被调用了，则onSaveInstanceState()一定被调用过；反之则不然。</p>
<p>这两个生命周期只是在<strong>Activity被异常终止</strong>时成对出现（6.0以上在动态权限申请时也会出现）。Activity的异常终止大概有以下两种情况：</p>
<ul>
<li>系统配置发生改变</li>
<li>内存不足，优先级低的Activity资源被回收</li>
</ul>
<p>其中<strong>系统配置改变</strong>是指系统需要重新加载一些资源以适应某些配置的改变，如旋转屏幕、语言或地区改变、键盘改变等。当发生这种改变的时候，系统会重新创建这个Activity，在销毁和重新创建的过程中，这两个生命周期将会被回调。在这种情况下，系统会自动保存当前view的一些状态，然后在重新创建时恢复数据。当然，我们也可以通过onSaveInstanceState()做一些自定义的保存。<strong>过滤不想监听的配置变化</strong></p>
<p><strong>onSaveInstanceState()方法的默认实现</strong></p>
<p>如果我们没有覆写onSaveInstanceState()方法, 此方法的默认实现会自动保存activity中的某些状态数据, 比如activity中各种UI控件的状态.。android应用框架中定义的几乎所有UI控件都恰当的实现了onSaveInstanceState()方法,因此当activity被摧毁和重建时, 这些UI控件会自动保存和恢复状态数据. 比如EditText控件会自动保存和恢复输入的数据,而CheckBox控件会自动保存和恢复选中状态.开发者只需要为这些控件指定一个唯一的ID(通过设置android:id属性即可), 剩余的事情就可以自动完成了.如果没有为控件指定ID, 则这个控件就不会进行自动的数据保存和恢复操作。</p>
<p>由上所述, 如果我们需要覆写onSaveInstanceState()方法, 一般会在第一行代码中调用该方法的默认实现:super.onSaveInstanceState(outState)。</p>
<p><strong>是否需要重写onSaveInstanceState()方法</strong></p>
<p>既然该方法的默认实现可以自动的保存UI控件的状态数据, 那什么时候需要覆写该方法呢?</p>
<p>如果需要保存额外的数据时, 就需要覆写onSaveInstanceState()方法。大家需要注意的是：onSaveInstanceState()方法只适合保存瞬态数据, 比如UI控件的状态, 成员变量的值等，而不应该用来保存持久化数据，持久化数据应该当用户离开当前的 activity时，在 onPause() 中保存（比如将数据保存到数据库或文件中）。说到这里，还要说一点的就是在onPause()中不适合用来保存比较费时的数据，所以这点要理解。</p>
<p>由于onSaveInstanceState()方法方法不一定会被调用, 因此不适合在该方法中保存持久化数据, 例如向数据库中插入记录等. 保存持久化数据的操作应该放在onPause()中。若是永久性值，则在onPause()中保存；若大量，则另开线程吧，别阻塞UI线程。</p>
<p><strong>onPostCreate()</strong><br>
这个回调发生在onStart()之后，onResume()之前，此时onCreate()已经执行完了，因此我们可以在其中做一些依赖于onCreate()中状态的操作；有时还会在这个函数里面做一些优先级稍低的事情，比如侧边栏的初始化，因为侧边栏的特性是需要进一步的用户交互来展现，因此我们可以减缓它初始化的优先级。对onPostCreate()的正确使用，可以使代码层次化更清晰，结构更合理。</p>
<p><strong>onLowMemory()</strong><br>
该生命周期出现在当系统内存不足的，低优先级的Activity的资源即将被回收时收到的回调，在这个回调中，我们可以做一些数据持久化工作、保存一些关键状态。在执行完这个生命周期后，系统将立即执行gc。</p>
<h2 id="activity生命周期中的一些问题">activity生命周期中的一些问题</h2>
<blockquote>
<p>setContentView如果放在onStart或者onResume中，会有什么问题吗？</p>
</blockquote>
<p>从官方文档来看，也只是说应该把setContentView放在onCreate中，并没有说必须放在这里，所以应该也不会有显示以及调用的问题吧。还是跑一下程序看看，分别将setContentView以及设置点击事件放在onStart以及onResume中，跑了下程序，没有出现显示问题。但是这个仅仅是显示上的问题，会不会存在效率的问题呢？我们来打印一下时间，以调用onCreate到onWindowFocusChanged之间的时间，作为Activity加载时间，来进行对比</p>
<pre><code>SecondActivity: =====SecondActivity=====Load Time:56
SecondActivity: =====SecondActivity=====Load Time:57
SecondActivity: =====SecondActivity=====Load Time:57
</code></pre>
<p>三次时间几乎一样的，也就是说，如果只是单丛初次启动的效率来说，在三个地方去进行setContentView是没有任何差别的。但是为甚么官方说应该放在onCreate里面去处理了，这是因为，onCreate在正常状况下，只会被调用一次，而onStart以及onResume都会被调用多次，放在这里面去做的话，在onResume的过程，会增加额外的耗时。</p>
<p>另外，由于onRestoreInstanceState是在onStart之后才调用的，如果将setContentView放在onResume的话，可能会产生问题。</p>
<blockquote>
<p>onPause中可以保存状态，为什么还要onSaveInstanceState，onCreate中有恢复机制，为什么还需要onRestoreInstanceState？</p>
</blockquote>
<p>首先，onSaveInstanceState以及onRestoreInstanceState是Android进行UI状态保存与恢复的一套单独的机制。说是单独的机制，是因为每个view里面，都会有onSaveInstanceState去进行一些默认的状态保存操作，常规状态下不需要用户去干预，比方说编辑框中输入的文本信息，这样做为开发者省了很多事。</p>
<p>根据官方文档来看，onPause主要的作用是停止动画以及一些耗CPU的操作，可以用于保存状态。onSaveInstanceState主要作用是对UI进行状态保存。两者的侧重点不同，onPause也可以保存UI的状态，但是，onPause设计的主要目的是和onStart配对，停止耗时操作。</p>
<p>onCreate中也可以恢复状态，但是onRestoreInstanceState的触发时间滞后于onCreate，在onStart之后执行，它设计的目的是用来恢复onSaveInstanceState中保存的状态。</p>
<p>onPause以及onCreate可以干这些事情，但是它们当初不是设计出来，专门干这个事情的。</p>
<blockquote>
<p>如何判断一个Activity真正可见?</p>
</blockquote>
<p>对于这个问题，其实没有严格的说法，onResume以及onWindowFocusChanged中都可以做判断，但是官方文档上说，onWindowFocusChanged是Activity对用户是否可见最好的指示器。</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

