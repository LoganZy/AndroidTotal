## Android Stduio不能正常打开，报错
**报错信息**
```
 Internal error. Please report to http://code.google.com/p/android/issues

java.lang.RuntimeException: com.intellij.ide.plugins.PluginManager$StartupAbortedException: Fatal error initializing 'com.intellij.util.indexing.FileBasedIndex'
    at com.intellij.idea.IdeaApplication.run(IdeaApplication.java:159)
    at com.intellij.idea.MainImpl$1$1$1.run(MainImpl.java:46)
    at java.awt.event.InvocationEvent.dispatch(InvocationEvent.java:311)
    at java.awt.EventQueue.dispatchEventImpl(EventQueue.java:744)
    at java.awt.EventQueue.access$400(EventQueue.java:97)
    at java.awt.EventQueue$3.run(EventQueue.java:697)
    at java.awt.EventQueue$3.run(EventQueue.java:691)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.security.ProtectionDomain$1.doIntersectionPrivilege(ProtectionDomain.java:75)
    at java.awt.EventQueue.dispatchEvent(EventQueue.java:714)
    at com.intellij.ide.IdeEventQueue.defaultDispatchEvent(IdeEventQueue.java:697)
    at com.intellij.ide.IdeEventQueue._dispatchEvent(IdeEventQueue.java:524)
    at com.intellij.ide.IdeEventQueue.dispatchEvent(IdeEventQueue.java:335)
    at java.awt.EventDispatchThread.pumpOneEventForFilters(EventDispatchThread.java:201)
    at java.awt.EventDispatchThread.pumpEventsForFilter(EventDispatchThread.java:116)
    at java.awt.EventDispatchThread.pumpEventsForHierarchy(EventDispatchThread.java:105)
    at java.awt.EventDispatchThread.pumpEvents(EventDispatchThread.java:101)
    at java.awt.EventDispatchThread.pumpEvents(EventDispatchThread.java:93)
    at java.awt.EventDispatchThread.run(EventDispatchThread.java:82)
Caused by: com.intellij.ide.plugins.PluginManager$StartupAbortedException: Fatal error initializing 'com.intellij.util.indexing.FileBasedIndex'
    at com.intellij.ide.plugins.PluginManager.handleComponentError(PluginManager.java:244)
    at com.intellij.openapi.components.impl.PlatformComponentManagerImpl.handleInitComponentError(PlatformComponentManagerImpl.java:39)
    at com.intellij.openapi.components.impl.ComponentManagerImpl$ComponentConfigComponentAdapter$1.getComponentInstance(ComponentManagerImpl.java:570)
    at com.intellij.openapi.components.impl.ComponentManagerImpl$ComponentConfigComponentAdapter.getComponentInstance(ComponentManagerImpl.java:590)
    at com.intellij.util.pico.DefaultPicoContainer.getLocalInstance(DefaultPicoContainer.java:225)
    at com.intellij.util.pico.DefaultPicoContainer.getInstance(DefaultPicoContainer.java:212)
    at com.intellij.util.pico.DefaultPicoContainer.getComponentInstance(DefaultPicoContainer.java:199)
    at org.picocontainer.alternatives.AbstractDelegatingMutablePicoContainer.getComponentInstance(AbstractDelegatingMutablePicoContainer.java:75)
    at com.intellij.openapi.components.impl.ComponentManagerImpl.createComponent(ComponentManagerImpl.java:121)
    at com.intellij.openapi.application.impl.ApplicationImpl.createComponent(ApplicationImpl.java:371)
    at com.intellij.openapi.components.impl.ComponentManagerImpl.createComponents(ComponentManagerImpl.java:112)
    at com.intellij.openapi.components.impl.ComponentManagerImpl.init(ComponentManagerImpl.java:89)
    at com.intellij.openapi.components.impl.stores.ApplicationStoreImpl.load(ApplicationStoreImpl.java:87)
    at com.intellij.openapi.application.impl.ApplicationImpl.load(ApplicationImpl.java:508)
    at com.intellij.idea.IdeaApplication.run(IdeaApplication.java:151)
    ... 18 more
Caused by: java.lang.RuntimeException: java.io.FileNotFoundException: C:\Users\UserName\.AndroidStudio\system\index\todoindex\TodoIndex.ver (The system cannot find the path specified)
    at com.intellij.util.indexing.FileBasedIndexImpl.initExtensions(FileBasedIndexImpl.java:332)
    at com.intellij.util.indexing.FileBasedIndexImpl.initComponent(FileBasedIndexImpl.java:359)
    at com.intellij.openapi.components.impl.ComponentManagerImpl$ComponentConfigComponentAdapter$1.getComponentInstance(ComponentManagerImpl.java:548)
    ... 30 more
Caused by: java.io.FileNotFoundException: C:\Users\UserName\.AndroidStudio\system\index\todoindex\TodoIndex.ver (The system cannot find the path specified)
    at java.io.FileOutputStream.open(Native Method)
    at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
    at java.io.FileOutputStream.<init>(FileOutputStream.java:162)
    at com.intellij.util.indexing.IndexInfrastructure$1.execute(IndexInfrastructure.java:95)
    at com.intellij.util.indexing.IndexInfrastructure$1.execute(IndexInfrastructure.java:90)
    at com.intellij.openapi.util.io.FileUtilRt.doIOOperation(FileUtilRt.java:517)
    at com.intellij.util.indexing.IndexInfrastructure.rewriteVersion(IndexInfrastructure.java:90)
    at com.intellij.util.indexing.FileBasedIndexImpl.registerIndexer(FileBasedIndexImpl.java:390)
    at com.intellij.util.indexing.FileBasedIndexImpl.initExtensions(FileBasedIndexImpl.java:290)
    ... 32 more
```
这种错误情况，还是比较罕见的，我也是在一次运行app崩溃后，导致AS自动关闭，再次打开就出现这个错误。

**解决方案：**
这个bug在`Androids open source bug tracker`上，已经有解决方案，[https://code.google.com/p/android/issues/detail?id=74458](https://code.google.com/p/android/issues/detail?id=74458) 在这里有详细的介绍。下面贴出被采取最多的意见：

	This Works without the loss any settings or project. It will take you to your previous state at the time of editing an open file.  
  
	1. go to your home directory. ie /home/XXXXXX/.AndroidStudio.X.X  
	2. rename the .AndroidStudio.X.X to any thing else i.e. back_up  
	3. run your android studio.  
	4. it will prompt you to import current setting or create a new version  
	5. choose the import setting and sellect the back_up directory.  
	6. Bravo You are good to go.
	
如果这种方法不能解决问题，可以参考这里 [StackOverFlow](https://stackoverflow.com/questions/28003717/android-studio-not-starting-fatal-error-initializing-com-intellij-util-indexin)
