---


---

<h2 id="android-broadcastreceiver再理解">Android BroadCastReceiver再理解</h2>
<h3 id="什么是broadcastreceiver？、">什么是BroadcastReceiver？、</h3>
<p>广播（Broadcast）是在组件之间传播数据的一种机制，这些组件可以位于不同的进程中，起到进程间通信的作用</p>
<p><strong>BroadcastReceiver</strong> 是对发送出来的 <strong>Broadcast</strong> 进行过滤、接受和响应的组件。首先将要发送的消息和用于过滤的信息（Action，Category）装入一个 <strong>Intent</strong> 对象，然后通过调用 <strong>Context.sendBroadcast()</strong> 、 <strong>sendOrderBroadcast()</strong> 方法把 Intent 对象以广播形式发送出去。 广播发送出去后，所以已注册的 BroadcastReceiver 会检查注册时的 <strong>IntentFilter</strong> 是否与发送的 Intent 相匹配，若匹配则会调用 BroadcastReceiver 的 <strong>onReceiver()</strong> 方法</p>
<p>所以当我们定义一个 BroadcastReceiver 的时候，都需要实现 onReceiver() 方法。BroadcastReceiver 的生命周期很短，在执行 onReceiver() 方法时才有效，一旦执行完毕，该Receiver 的生命周期就结束了</p>
<h3 id="broadcastreceiver的种类">BroadcastReceiver的种类</h3>
<ul>
<li>标准广播<br>
标准广播是一种完全异步执行的广播，在广播发出后所有的广播接收器会在同一时间接收到这条广播，之间没有先后顺序，效率比较高，且无法被截断</li>
<li>有序广播<br>
有序广播是一种同步执行的广播，在广播发出后同一时刻只有一个广播接收器能够接收到， 优先级高的广播接收器会优先接收，当优先级高的广播接收器的 onReceiver() 方法运行结束后，广播才会继续传递，且前面的广播接收器可以选择截断广播，这样后面的广播接收器就无法接收到这条广播了</li>
</ul>
<h3 id="broadcastreceiver的注册">BroadcastReceiver的注册</h3>
<p><strong>静态注册</strong></p>
<blockquote>
<p>静态注册即在<strong>清单文件</strong>中为 <strong>BroadcastReceiver</strong> 进行注册，使用**&lt; receiver &gt;**标签声明，并在标签内用 <strong>&lt; intent-filter &gt;</strong> 标签设置过滤器。这种形式的 BroadcastReceiver 的生命周期伴随着整个应用，如果这种方式处理的是系统广播，那么不管应用是否在运行，该广播接收器都能接收到该广播。</p>
</blockquote>
<pre><code>&lt;receiver 
    android:enabled=["true" | "false"]
	//此broadcastReceiver能否接收其他App的发出的广播
	//默认值是由receiver中有无intent-filter决定的：如果有intent-filter，默认值为true，否则为false
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
	//继承BroadcastReceiver子类的类名
    android:name=".mBroadcastReceiver"
	//具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
    android:permission="string"
	//BroadcastReceiver运行所处的进程
	//默认为app的进程，可以指定独立的进程
	//注：Android四大基本组件都可以通过此属性指定自己的独立进程
    android:process="string" &gt;

	//用于指定此广播接收器将接收的广播类型
	//本示例中给出的是用于接收网络状态改变时发出的广播
	 &lt;intent-filter&gt;
		&lt;action android:name="android.net.conn.CONNECTIVITY_CHANGE" /&gt;
    &lt;/intent-filter&gt;
&lt;/receiver&gt;
</code></pre>
<p><strong>动态注册</strong></p>
<blockquote>
<p>动态注册 BroadcastReceiver 是在代码中定义并设置好一个 <strong>IntentFilter</strong> 对象，然后在需要注册的地方调用 <strong>Context.registerReceiver()</strong> 方法，调用 <strong>Context.unregisterReceiver()</strong> 方法取消注册，此时就不需要在清单文件中注册 Receiver 了</p>
</blockquote>
<pre><code>  // 选择在Activity生命周期方法中的onResume()中注册
  @Override
  protected void onResume(){
      super.onResume();
	  // 1. 实例化BroadcastReceiver子类 &amp;  IntentFilter
      mBroadcastReceiver mBroadcastReceiver = new mBroadcastReceiver();
      IntentFilter intentFilter = new IntentFilter();
      // 2. 设置接收广播的类型
      intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
      // 3. 动态注册：调用Context的registerReceiver（）方法
      registerReceiver(mBroadcastReceiver, intentFilter);
  }
  // 注册广播后，要在相应位置记得销毁广播
  // 即在onPause() 中unregisterReceiver(mBroadcastReceiver)
  // 当此Activity实例化时，会动态将MyBroadcastReceiver注册到系统中
  // 当此Activity销毁时，动态注册的MyBroadcastReceiver将不再接收到相应的广播。
  @Override
  protected void onPause() {
      super.onPause();
      //销毁在onResume()方法中的广播
      unregisterReceiver(mBroadcastReceiver);
  }
</code></pre>
<p><code>在onResume()注册、onPause()注销是因为onPause()在App死亡前一定会被执行，从而保证广播在App死亡前一定会被注销，从而防止内存泄露。</code></p>
<blockquote>
<p>不在onCreate() &amp; onDestory() 或 onStart() &amp; onStop()注册、注销是因为：<br>
当系统因为内存不足（优先级更高的应用需要内存，请看上图红框）要回收Activity占用的资源时，Activity在执行完onPause()方法后就会被销毁，有些生命周期方法onStop()，onDestory()就不会执行。当再回到此Activity时，是从onCreate方法开始执行。<br>
假设我们将广播的注销放在onStop()，onDestory()方法里的话，有可能在Activity被销毁后还未执行onStop()，onDestory()方法，即广播仍还未注销，从而导致内存泄露。<br>
但是，onPause()一定会被执行，从而保证了广播在App死亡前一定会被注销，从而防止内存泄露。</p>
</blockquote>
<h4 id="两种注册方式的区别">两种注册方式的区别</h4>
<p><a href="https://i.loli.net/2019/05/13/5cd937c21865a83355.png"><img src="https://i.loli.net/2019/05/13/5cd937c21865a83355.png" alt="broadcastreceiver.png"></a></p>
<ul>
<li>模型中有3个角色：</li>
</ul>
<ol>
<li>消息订阅者（广播接收者）</li>
<li>消息发布者（广播发布者）</li>
<li>消息中心（<code>AMS</code>，即<code>Activity Manager Service</code>）</li>
</ol>
<p><a href="https://i.loli.net/2019/05/13/5cd9392e0f97f45116.png"><img src="https://i.loli.net/2019/05/13/5cd9392e0f97f45116.png" alt="shiyitu.png"></a></p>
<h3 id="广播的类型">广播的类型</h3>
<ul>
<li>
<p>广播的类型主要分为5类：</p>
</li>
<li>
<p>普通广播（Normal Broadcast）</p>
</li>
<li>
<p>系统广播（System Broadcast）</p>
</li>
<li>
<p>有序广播（Ordered Broadcast）</p>
</li>
<li>
<p>粘性广播（Sticky Broadcast）</p>
</li>
<li>
<p>App应用内广播（Local Broadcast）</p>
</li>
</ul>
<p>具体说明如下：</p>
<p>普通广播（Normal Broadcast）<br>
即 开发者自身定义 intent的广播（最常用）。发送广播使用如下：</p>
<pre><code>　　Intent intent = new Intent();
　　//对应BroadcastReceiver中intentFilter的action
　　intent.setAction(BROADCAST_ACTION);
　　//发送广播
　　sendBroadcast(intent);
</code></pre>
<p>若被注册了的广播接收者中注册时intentFilter的action与上述匹配，则会接收此广播（即进行回调onReceive()）。如下mBroadcastReceiver则会接收上述广播</p>
<pre><code>&lt;receiver 
    //此广播接收者类是mBroadcastReceiver
    android:name=".mBroadcastReceiver" &gt;
    //用于接收网络状态改变时发出的广播
    &lt;intent-filter&gt;
        &lt;action android:name="BROADCAST_ACTION" /&gt;
    &lt;/intent-filter&gt;
&lt;/receiver&gt;
</code></pre>
<p>若发送广播有相应权限，那么广播接收者也需要相应权限<br>
2. 系统广播（System Broadcast）</p>
<p>Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播<br>
每个广播都有特定的Intent - Filter（包括具体的action），Android系统广播action如下：<br>
系统操作	action<br>
监听网络变化	android.net.conn.CONNECTIVITY_CHANGE<br>
关闭或打开飞行模式	Intent.ACTION_AIRPLANE_MODE_CHANGED<br>
充电时或电量发生变化	Intent.ACTION_BATTERY_CHANGED<br>
电池电量低	Intent.ACTION_BATTERY_LOW<br>
电池电量充足（即从电量低变化到饱满时会发出广播	Intent.ACTION_BATTERY_OKAY<br>
系统启动完成后(仅广播一次)	Intent.ACTION_BOOT_COMPLETED<br>
按下照相时的拍照按键(硬件按键)时	Intent.ACTION_CAMERA_BUTTON<br>
屏幕锁屏	Intent.ACTION_CLOSE_SYSTEM_DIALOGS<br>
设备当前设置被改变时(界面语言、设备方向等)	Intent.ACTION_CONFIGURATION_CHANGED<br>
插入耳机时	Intent.ACTION_HEADSET_PLUG<br>
未正确移除SD卡但已取出来时(正确移除方法:设置–SD卡和设备内存–卸载SD卡)	Intent.ACTION_MEDIA_BAD_REMOVAL<br>
插入外部储存装置（如SD卡）	Intent.ACTION_MEDIA_CHECKING<br>
成功安装APK	Intent.ACTION_PACKAGE_ADDED<br>
成功删除APK	Intent.ACTION_PACKAGE_REMOVED<br>
重启设备	Intent.ACTION_REBOOT<br>
屏幕被关闭	Intent.ACTION_SCREEN_OFF<br>
屏幕被打开	Intent.ACTION_SCREEN_ON<br>
关闭系统时	Intent.ACTION_SHUTDOWN<br>
重启设备	Intent.ACTION_REBOOT<br>
注：当使用系统广播时，只需要在注册广播接收者时定义相关的action即可，并不需要手动发送广播，当系统有相关操作时会自动进行系统广播</p>
<ol start="3">
<li>有序广播（Ordered Broadcast）</li>
</ol>
<p>定义<br>
发送出去的广播被广播接收者按照先后顺序接收<br>
有序是针对广播接收者而言的</p>
<p>广播接受者接收广播的顺序规则（同时面向静态和动态注册的广播接受者）</p>
<p>按照Priority属性值从大-小排序；<br>
Priority属性相同者，动态注册的广播优先；<br>
特点</p>
<p>接收广播按顺序接收<br>
先接收的广播接收者可以对广播进行截断，即后接收的广播接收者不再接收到此广播；<br>
先接收的广播接收者可以对广播进行修改，那么后接收的广播接收者将接收到被修改后的广播<br>
具体使用<br>
有序广播的使用过程与普通广播非常类似，差异仅在于广播的发送方式：</p>
<p><code>sendOrderedBroadcast(intent);</code><br>
4. App应用内广播（Local Broadcast）</p>
<ul>
<li>
<p>背景<br>
Android中的广播可以跨App直接通信（exported对于有intent-filter情况下默认值为true）</p>
</li>
<li>
<p>冲突<br>
可能出现的问题：</p>
</li>
</ul>
<p>其他App针对性发出与当前App intent-filter相匹配的广播，由此导致当前App不断接收广播并处理；<br>
其他App注册与当前App一致的intent-filter用于接收广播，获取广播具体信息；<br>
即会出现安全性 &amp; 效率性的问题。</p>
<blockquote>
<p>解决方案</p>
</blockquote>
<ul>
<li>使用App应用内广播（Local Broadcast）</li>
</ul>
<blockquote>
<p>1.App应用内广播可理解为一种局部广播，广播的发送者和接收者都同属于一个App。<br>
2.相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 &amp; 效率高</p>
</blockquote>
<ul>
<li>具体使用1 - 将全局广播设置成局部广播</li>
</ul>
<p>1.注册广播时将exported属性设置为false，使得非本App内部发出的此广播不被接收；<br>
2.在广播发送和接收时，增设相应权限permission，用于权限验证；<br>
3.发送广播时指定该广播接收器所在的包名，此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。</p>
<blockquote>
<p>通过**intent.setPackage(packageName)**指定报名</p>
</blockquote>
<ul>
<li>具体使用2 - 使用封装好的LocalBroadcastManager类<br>
使用方式上与全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将参数的context变成了LocalBroadcastManager的单一实例</li>
</ul>
<blockquote>
<p>注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册，不能静态注册</p>
</blockquote>
<pre><code>//注册应用内广播接收器
//步骤1：实例化BroadcastReceiver子类 &amp; IntentFilter mBroadcastReceiver 
mBroadcastReceiver = new mBroadcastReceiver(); 
IntentFilter intentFilter = new IntentFilter(); 
//步骤2：实例化LocalBroadcastManager的实例
localBroadcastManager = LocalBroadcastManager.getInstance(this);
//步骤3：设置接收广播的类型 
intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
//步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
//取消注册应用内广播接收器
localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
//发送应用内广播
Intent intent = new Intent();
intent.setAction(BROADCAST_ACTION);
localBroadcastManager.sendBroadcast(intent);
</code></pre>
<h3 id="粘性广播（sticky-broadcast）">5. 粘性广播（Sticky Broadcast）</h3>
<p>由于在Android5.0 &amp; API 21中已经失效，所以不建议使用，在这里也不作过多的总结。</p>
<h2 id="使用私有权限"><strong>使用私有权限</strong></h2>
<p>使用动态注册广播接收器存在一个问题，即系统内的任何应用均可监听并触发我们的 Receiver 。通常情况下我们是不希望如此的</p>
<p>解决办法之一是在清单文件中为 <strong>&lt; receiver &gt;</strong> 标签添加一个 <strong>android:exported=“false”</strong> 属性，标明该 Receiver 仅限应用内部使用。这样，系统中的其他应用就无法接触到该 Receiver 了</p>
<p>此外，也可以选择创建自己的使用权限，即在清单文件中添加一个 <strong>&lt; permission &gt;</strong> 标签来声明自定义权限</p>
<pre><code>    &lt;permission
        android:name="com.example.permission.receiver"
        android:protectionLevel="signature" /&gt;

</code></pre>
<p>自定义权限时必须同时指定 <strong>protectionLevel</strong> 属性值，系统根据该属性值确定自定义权限的使用方式</p>

<table>
<thead>
<tr>
<th>属性值</th>
<th>限定方式</th>
</tr>
</thead>
<tbody>
<tr>
<td>normal</td>
<td>默认值。较低风险的权限，对其他应用，系统和用户来说风险最小。系统在安装应用时会自动批准授予应用该类型的权限，不要求用户明确批准（虽然用户在安装之前总是可以选择查看这些权限）</td>
</tr>
<tr>
<td>dangerous</td>
<td>较高风险的权限，请求该类型权限的应用程序会访问用户私有数据或对设备进行控制，从而可能对用户造成负面影响。因为这种类型的许可引入了潜在风险，所以系统可能不会自动将其授予请求的应用。例如，系统可以向用户显示由应用请求的任何危险许可，并且在继续之前需要确认，或者可以采取一些其他方法来避免用户自动允许</td>
</tr>
<tr>
<td>signature</td>
<td>只有在请求该权限的应用与声明权限的应用使用相同的证书签名时，系统才会授予权限。如果证书匹配，系统会自动授予权限而不通知用户或要求用户的明确批准</td>
</tr>
<tr>
<td>signatureOrSystem</td>
<td>系统仅授予Android系统映像中与声明权限的应用使用相同的证书签名的应用。请避免使用此选项，“signature”级别足以满足大多数需求，“signatureOrSystem”权限用于某些特殊情况</td>
</tr>
</tbody>
</table><p>首先，新建一个新的工程，在它的清单文件中创建一个自定义权限，并声明该权限。<strong>protectionLevel</strong> 属性值设为“<strong>signature</strong>”</p>
<pre><code>    &lt;permission
        android:name="com.example.permission.receiver"
        android:protectionLevel="signature" /&gt;
    &lt;uses-permission android:name="com.example.permission.receiver" /&gt;
</code></pre>
<p>然后，发送含有该权限声明的 Broadcast 。这样，只有使用相同证书签名且声明该权限的应用才能接收到该 Broadcast 了</p>
<pre><code>    private final String PERMISSION_PRIVATE = "com.example.permission.receiver";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    public void sendPermissionBroadcast(View view) {
        sendBroadcast(new Intent("Hi"), PERMISSION_PRIVATE);
    }
</code></pre>
<p>回到之前的工程<br>
首先在清单文件中声明权限</p>
<pre><code>&lt;uses-permission android:name="com.example.permission.receiver" /&gt;
</code></pre>
<p>创建一个 BroadcastReceiver</p>
<pre><code>public class PermissionReceiver extends BroadcastReceiver {
    private final String TAG = "PermissionReceiver";
    public PermissionReceiver() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e(TAG, "接收到了私有权限广播");
    }
}
</code></pre>
<p>然后注册广播接收器。因为 Android Studio 在调试的时候会使用相同的证书为每个应用签名，所以，在之前新安装的App发送出广播后，PermissionReceiver 就会输出 Log 日志</p>
<pre><code>    private final String PERMISSION_PRIVATE = "com.example.permission.receiver";

    private PermissionReceiver permissionReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        IntentFilter intentFilter1 = new IntentFilter("Hi");
        permissionReceiver = new PermissionReceiver();
        registerReceiver(permissionReceiver, intentFilter1, PERMISSION_PRIVATE, null);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(permissionReceiver);
    }
</code></pre>
<h3 id="特别注意">特别注意</h3>
<p>对于不同注册方式的广播接收器回调OnReceive（Context context，Intent intent）中的context返回值是不一样的：</p>
<ul>
<li>对于静态注册（全局+应用内广播），回调onReceive(context, intent)中的context返回值是：ReceiverRestrictedContext；</li>
<li>对于全局广播的动态注册，回调onReceive(context, intent)中的context返回值是：Activity Context；</li>
<li>对于应用内广播的动态注册（LocalBroadcastManager方式），回调onReceive(context, intent)中的context返回值是：Application Context。</li>
<li>对于应用内广播的动态注册（非LocalBroadcastManager方式），回调onReceive(context, intent)中的context返回值是：Activity Context；</li>
</ul>
<h2 id="broadcastreceiver一些使用场景">BroadcastReceiver一些使用场景</h2>
<ul>
<li><code>Android</code>不同组件间的通信（含 ：应用内 / 不同应用之间）</li>
<li>多线程通信</li>
<li>与  <code>Android</code>  系统在特定情况下的通信</li>
<li>监听网络状态、电量的变化，应用的安装、更新、及卸载</li>
</ul>

