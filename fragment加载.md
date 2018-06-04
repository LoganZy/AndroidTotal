---


---

<h2 id="fragment中onhiddenchanged、setuservisiblehint触发条件">Fragment中onHiddenChanged、setUserVisibleHint触发条件</h2>
<blockquote>
<p>在Fragment使用的过程，根据不能的业务需求，我们使用Fragment的情景。多个fragment+viewpage使用；或是多个fragment的add()、hide()或replace()使用。</p>
</blockquote>
<h3 id="fragment的onhiddenchanged触发条件">Fragment的onHiddenChanged()触发条件</h3>
<pre><code>/** 
* Return true if the fragment has been hidden. By default fragments 
* are shown.You can find out about changes to this state with 
* {@link #onHiddenChanged}. Note that the hidden state is orthogonal
* to other states -- that is, to be visible to the user, a fragment
* must be both started and not hidden. 
* /
</code></pre>
<p>如果该Fragment对象已经被隐藏，那么它返回true。默认情况下，Fragment是被显示的。能够用onHiddenChanged(boolean)回调方法获取该Fragment对象状态的改变，要注意的是隐藏状态与其他状态是正交的—也就是说，要把该Fragment对象显示给用户，Fragment对象必须是被启动并被隐藏。<br>
值得我们注意的是 这里的隐藏或者显示是指Fragment调用show或者hider的时候才会改变mHidden的值得。</p>

