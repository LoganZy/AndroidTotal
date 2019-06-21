---


---

<p>ListView、RecycleView的item点击事件无效<br>
产生的原因是应为item的布局中又有CheckBox、Button、RadioButton、EditText等控件的存在，优先获取焦点，导致父布局的点击事件失效。<br>
解决方案：<br>
1.将Button、CheckBox的实现替换为TextView、ImageView。<br>
2.若是不能替换，就讲item布局中这些view的focusable属性设置为false。<br>
3.设置ListView的item的根布局android:descendantFocusability=“blocksDescendants”，一般推荐第三种，意思是ListView的item下边所有的子控件都不能获取焦点。</p>
<p>android:descendantFocusability的值有3种，其中ViewGroup指的是设置这个属性的View，在这里就指的是ListView的item的根布局：</p>
<pre><code>beforeDescendants：ViewGroup会优先其子类控件而获取到焦点
afterDescendants：ViewGroup只有当其子类控件不需要获取焦点时才获取焦点
blocksDescendants：ViewGroup会覆盖子类控件而直接获得焦点
</code></pre>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

