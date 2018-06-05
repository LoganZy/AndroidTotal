---


---

<h2 id="fragment中onhiddenchanged、setuservisiblehint触发条件">Fragment中onHiddenChanged、setUserVisibleHint触发条件</h2>
<blockquote>
<p>在Fragment使用的过程，根据不能的业务需求，我们使用Fragment的情景。多个fragment+viewpage使用；或是多个fragment的add()、hide()或replace()使用。</p>
</blockquote>
<h3 id="fragment的onhiddenchanged触发条件">Fragment的onHiddenChanged()触发条件</h3>
<pre><code>/**  
 * Called when the hidden state (as returned by {@link #isHidden()} of  
 * the fragment has changed.  Fragments start out not hidden; this will 
 * be called whenever the fragment changes state from that. 
 * @param hidden True if the fragment is now hidden, false otherwise.  
 */
 public void onHiddenChanged(boolean hidden) {  
 }
</code></pre>
<p>查看onHidden Change()源码，可以看出，触发执行条件是fragment状态改变，是否可见。由isHidden()方法决定。</p>
<pre><code>/**  
 * Return true if the fragment has been hidden.  By default fragments 
 * are shown.  You can find out about changes to this state with 
 * {@link #onHiddenChanged}.  Note that the hidden state is orthogonal  
 * to other states -- that is, to be visible to the user, a fragment 
 * must be both started and not hidden.
 */
 final public boolean isHidden() {  
  return mHidden;  
 }
</code></pre>
<p>如果该Fragment对象已经被隐藏，那么它返回true。默认情况下，Fragment是被显示的。能够用onHiddenChanged(boolean)回调方法获取该Fragment对象状态的改变，要注意的是隐藏状态与其他状态是正交的—也就是说，要把该Fragment对象显示给用户，Fragment对象必须是被启动并不被隐藏(显示)。</p>
<blockquote>
<p>值得我们注意的是 这里的隐藏或者显示是指Fragment调用show或者hider的时候才会改变mHidden的值得。</p>
</blockquote>
<ul>
<li>当点击Fragment A的内容跳转到Activity B；包括从Activity B再次返回到Fragment A 。这些情况下Fragment A的onHiddenChange()是不会执行的。</li>
<li>当前界面为Fragment B，点击Home键回到主屏幕，这个时候Fragment B在手机屏幕上是不可见的，这种情况下监听onHiddenChange()方法来判断fragment的显隐状态是不可行。这种情况下，会响应fragmnet 的onPause()方法。</li>
<li>将onHiddenChange()与onPause()、onResume()搭配使用，才能更完美的解决问题。</li>
</ul>
<h3 id="fragment的setuservisiblehint触发条件">Fragment的setUserVisibleHint()触发条件</h3>
<p>多数UI界面和业务，需要我们使用ViewPager+TabLayout+Fragment来实现。我们需要借助 PagerAdapter（FragmentPagerAdapter和FragmentStatePagerAdapter）来实现多个fragment与viewpager的绑定。这种情况下，fragment的setUserVisableHint()方法才会被触发响应。</p>
<h4 id="fragmentpageradapter">FragmentPagerAdapter</h4>
<p><img src="!%5B%5D%28http://p981u1am0.bkt.clouddn.com/18-6-5/43162487.jpg%29" alt="FragmentPagerAdapter部分代码截图"></p>
<h4 id="fragmentstatepageradapter">FragmentStatePagerAdapter</h4>

