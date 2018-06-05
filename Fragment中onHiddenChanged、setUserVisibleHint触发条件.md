
## Fragment中onHiddenChanged、setUserVisibleHint触发条件
> 在Fragment使用的过程，根据不能的业务需求，我们使用Fragment的情景。多个fragment+viewpage使用；或是多个fragment的add()、hide()或replace()使用。

### Fragment的onHiddenChanged()触发条件
```
/**  
 * Called when the hidden state (as returned by {@link #isHidden()} of  
 * the fragment has changed.  Fragments start out not hidden; this will 
 * be called whenever the fragment changes state from that. 
 * @param hidden True if the fragment is now hidden, false otherwise.  
 */
 public void onHiddenChanged(boolean hidden) { 
 }
```
查看onHidden Change()源码，可以看出，触发执行条件是fragment状态改变，是否可见。由isHidden()方法决定。

```
/**  
 * Return true if the fragment has been hidden.  By default fragments 
 * are shown.  You can find out about changes to this state with 
 * {@link #onHiddenChanged}.  Note that the hidden state is orthogonal  
 * to other states -- that is, to be visible to the user, a fragment 
 * must be both started and not hidden.
 */
 final public boolean isHidden() {  
     return mHidden;  
 }
```
如果该Fragment对象已经被隐藏，那么它返回true。默认情况下，Fragment是被显示的。能够用onHiddenChanged(boolean)回调方法获取该Fragment对象状态的改变，要注意的是隐藏状态与其他状态是正交的---也就是说，要把该Fragment对象显示给用户，Fragment对象必须是被启动并不被隐藏(显示)。 
> 值得我们注意的是 这里的隐藏或者显示是指Fragment调用show或者hider的时候才会改变mHidden的值得。
 * 当点击Fragment A的内容跳转到Activity B；包括从Activity B再次返回到Fragment A 。这些情况下Fragment A的onHiddenChange()是不会执行的。
 * 当前界面为Fragment B，点击Home键回到主屏幕，这个时候Fragment B在手机屏幕上是不可见的，这种情况下监听onHiddenChange()方法来判断fragment的显隐状态是不可行。这种情况下，会响应fragmnet 的onPause()方法。
 * 将onHiddenChange()与onPause()、onResume()搭配使用，才能更完美的解决问题。
 #### 情景一：
 从其他 Activity 返回这个 Activity 时，当前 Fragment 不会调用 onHiddenChanged() 方法，需要在 onResume() 方法中请求服务器
解决方案：为了实现切换刷新操作，必须在 onResume() 和 onHiddenChanged() 方法中请求服务器。但是还得避免重复多次请求服务器操作，必须在两个方法中添加状态判断，只有对用户可见时，才刷新界面。
```
@Override
public void onResume() {
    super.onResume();
    if (isVisible()){
        // 发起网络请求, 刷新界面数据
        requestData();
    }
}
@Override
public void onHiddenChanged(boolean hidden) {
    super.onHiddenChanged(hidden);
    // 这里的 isResumed() 判断就是为了避免与 onResume() 方法重复发起网络请求
    if (isVisible() && isResumed()){
        requestData();
    }
}
```
还有一种特殊情况需要处理，就是系统由于内存不足时杀掉 App 的情况。如果当前显示的不是第一个 Fragment，App 被杀掉再次重启时，显示这个 Fragment 时，isVisible() 的判断始终为 false，这种情况下刷新数据的操作，还要额外处理。
```
@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    if (savedInstanceState!=null){
        requestData();
    }
}
```

### Fragment的setUserVisibleHint()触发条件
多数UI界面和业务，需要我们使用ViewPager+TabLayout+Fragment来实现。我们需要借助 PagerAdapter（FragmentPagerAdapter和FragmentStatePagerAdapter）来实现多个fragment与viewpager的绑定。这种情况下，fragment的setUserVisableHint()方法才会被触发响应。

#### FragmentPagerAdapter
![FragmentPagerAdapter部分代码截图](http://p981u1am0.bkt.clouddn.com/18-6-5/43162487.jpg)
> FragmentPagerAdapter实际上是使用add()，attach()和detach()来管理Fragment的。
> 需要注意的是所有实例化过的Fragment实例都会保存在内存中，所以适合页面数量不多且固定的app首页等情况。

#### FragmentStatePagerAdapter
![FragmentStatePagerAdapter部分代码截图](http://p981u1am0.bkt.clouddn.com/18-6-5/27859897.jpg)
> FragmentStatePagerAdapter使用add()和remove()管理Fragment，所以缓存外的Fragment的实例不会保存在内存中，适合分页多，数据动态的情况

####  显示和隐藏
对干缓存数量外的Fragment会被detach或remove，我们可以根据其常规生命周期进行开发，但是缓存数量内的显隐并不会影响生命周期，我们可以通过setUserVisableHint()方法来拍段某个Fragment是否显示。

![Fragment+PagerAdapter，生命周期变化](http://p981u1am0.bkt.clouddn.com/18-6-5/42571706.jpg)

从日志中我们可以看出，缓存数量内的Fragment0和Fragment1的setUserVisableHint()方法的isVisibleToUser首先会被设置成false，然后分别进行onAttach() - onResume()的生命周期，其中需要显示的Fragment在onCreate()之后，会将isVisibleToUser置为true，然后显示出来。
注意：**由此可见setUserVisibleHint()有可能在onAttach()之前调用，并且到显示前可能调用2次。**

再之后的显隐就是设置isVisibleToUser并回调通知了。和上文中的onHiddenChanged()一样，显隐状态没有变化时，也是不会回调的。
