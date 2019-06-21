---


---

<h2 id="android-stduio-instant-run-install-app-报错">Android Stduio Instant Run install app 报错</h2>
<p>Android stduio通过instant app 方式，安装app到手机，发生错误，报错信息如下：</p>
<pre><code>* What went wrong:
Execution failed for task ':app:transformDexWithInstantRunSlicesApkForChexiaoduoDebug'.
&gt; java.lang.RuntimeException: java.io.FileNotFoundException: /Users/loganzy/AndroidStudioProjects/cdd-last-app/app/build/intermediates/instant_run_split_apk_resources/chexiaoduoDebug/instantRunSplitApkResourcesChexiaoduoDebug/out/slice_3/resources_ap
</code></pre>
<p>在build中点击Run with --stacktrace，gradle向下执行，不会报错，执行通过，拿不到错误信息。</p>
<blockquote>
<p>Bug 出现的条件：安装方式 instant run 勾选中 ，Gradle的版本是5.1.1</p>
</blockquote>
<p><strong>解决方案：</strong></p>
<blockquote>
<p>方式一：</p>
</blockquote>
<ul>
<li>打开 gradle/gradle-wrapper.properties 文件。</li>
<li>更新gradle版本为5.4<br>
<code>distributionUrl=https\://services.gradle.org/distributions/gradle-5.4-all.zip</code></li>
<li>再次构建项目并运行</li>
</ul>
<blockquote>
<p>方式二:</p>
</blockquote>
<ul>
<li>在设置中将<code>instant app</code> 勾选去掉，构建项目并运行就可以成功安装app。</li>
</ul>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

