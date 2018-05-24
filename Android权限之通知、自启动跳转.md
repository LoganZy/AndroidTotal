# Android权限之通知、自启动跳转
>关于极光推送，由于Android手机的特性，用户在关闭app的情境下，无法继续再接收到极光推送的消息；对于我们开发人员来说，需要解决的问题就是怎么样最大化的保证，能接收到推送通知的用户足够的多。

## 解决方案
解决方案分为三种情况，下面给大家分享下，一些解决的心得体会：

- 通过监听系统广播+Service的形式，拉起app程序保活，这里需要保证Service的活性，不容易被杀掉：1、将service优先级调到最大；2、在onDestroy()中自启。3.将优先级设置最大；在AndroidManifest.xml文件里将persistent设置为true。**适用于放在/system/app下的app**
   ```
   <intent-filter  android:priority="1000">
   ```
- 从推送的SDK寻找解决的方案，可以尝试的接入不同厂商的系统通道的推送（小米、华为），或是在接入极光推送的基础上，再接入它们的vip通道（目前有小米、华为、魅族的FCM），不过石收费的。详细的可以联系它们的客服。
- 引导用户手动开启通知、将app加入开启自启动白名单、或是将app加到多任务锁。具体引导流程，不妨参考企业微信的消息、通知开启的引导方式。

## 开启通知- - -跳转app权限设置界面
开启通知权限，区别于存储、拍照、电话、定位的权限。这些权限需要动态获取的，而且调用后，就会有交互的弹窗用于拒绝或是授权，通知权限需要先判断，若没有授权，再跳转到系统界面授权。判断及跳转的代码如下。
```
    /**
     * 判断允许通知，是否已经授权
     * 返回值为true时，通知栏打开，false未打开。
     *@param context 上下文
     */
    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    private boolean isNotificationEnabled(Context context) {

        String CHECK_OP_NO_THROW = "checkOpNoThrow";
        String OP_POST_NOTIFICATION = "OP_POST_NOTIFICATION";

        AppOpsManager mAppOps = (AppOpsManager) context.getSystemService(Context.APP_OPS_SERVICE);
        ApplicationInfo appInfo = context.getApplicationInfo();
        String pkg = context.getApplicationContext().getPackageName();
        int uid = appInfo.uid;

        Class appOpsClass = null;
        /* Context.APP_OPS_MANAGER */
        try {
            appOpsClass = Class.forName(AppOpsManager.class.getName());
            Method checkOpNoThrowMethod = appOpsClass.getMethod(CHECK_OP_NO_THROW, Integer.TYPE, Integer.TYPE,
                    String.class);
            Field opPostNotificationValue = appOpsClass.getDeclaredField(OP_POST_NOTIFICATION);

            int value = (Integer) opPostNotificationValue.get(Integer.class);
            return ((Integer) checkOpNoThrowMethod.invoke(mAppOps, value, uid, pkg) == AppOpsManager.MODE_ALLOWED);

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return false;
    }
    
    /**  
     * 跳转到app的设置界面--开启通知  
     * @param context  
     */
     private void goToNotificationSetting(Context context) {  
	    Intent intent = new Intent();  
	    if (Build.VERSION.SDK_INT >= 26) {  
	        // android 8.0引导  
	        intent.setAction("android.settings.APP_NOTIFICATION_SETTINGS");  
	        intent.putExtra("android.provider.extra.APP_PACKAGE", context.getPackageName());  
	    } else if (Build.VERSION.SDK_INT >= 21) {  
	        // android 5.0-7.0  
	        intent.setAction("android.settings.APP_NOTIFICATION_SETTINGS");  
	        intent.putExtra("app_package", context.getPackageName());  
	        intent.putExtra("app_uid", context.getApplicationInfo().uid);  
	    } else {  
	        // 其他  
	        intent.setAction("android.settings.APPLICATION_DETAILS_SETTINGS");  
	        intent.setData(Uri.fromParts("package", context.getPackageName(), null));  
	    }  
	    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);  
	    context.startActivity(intent);  
	}  

```
## 自启动- - -跳转系统自启动界面
跳转到手机系统的自启动界面，自行勾选自启动功能。这里不同手机厂商的rom不同，这个功能页面也不是一致的、这个功能界面在有的系统中石不允许外部Intent直接跳转的，所以我们只能跳转到该子页面的前一级界面；我们尽可能多的做主流界面的兼容。
>首先，自启动界面，在不同手机上的UI入口是不同的，有的是在系统设置中，有的是在手机自带的安全管家中设置的，需要我们自行的获取当前activity的名称，之后指定ComponentName来跳转。
- 打开手机相应的自启动界面，在terminal中运行，adb名称获取当前activity的全称。
	```
	adb shell dumpsys activity top
	```
  Samsung手机的结果如下，以供参考：
  
  ![](http://p981u1am0.bkt.clouddn.com/18-5-24/92317295.jpg)
- 将上面的目标Activity 的路径copy，通过Intent 的ComponentName形式来跳转打开，并进行下一步的自启动操作的设置。因为手机厂商目标Activity的路径需要相应的适配。附上我适配的代码：
	```
    /**
     * 跳转到手机系统的自启动界面或是手机管家界面
     * adb shell dumpsys activity top
     * @param context
     */
    public static void jumpStartManager(Context context) {
        Intent intent = new Intent();
        try {
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            Log.e("StartUtils", "当前手机型号为：" + getMobileType());
            ComponentName componentName = null;
            if (getMobileType().equals("Xiaomi")) { 
				// 小米 5s、红米note 4s 
                componentName = new ComponentName("com.miui.securitycenter", "com.miui.permcenter.autostart.AutoStartManagementActivity");
            } else if (getMobileType().equals("Letv")) { 
				// 乐视2 
                intent.setAction("com.letv.android.permissionautoboot");
            } else if (getMobileType().equals("samsung")) { 
                //samsung Galaxy s7 
                componentName = ComponentName.unflattenFromString("com.samsung.android.sm_cn/com.samsung.android.sm.ui.cstyleboard.SmartManagerDashBoardActivity");
				//componentName = new ComponentName("com.samsung.android.sm_cn", "com.samsung.android.sm.ui.ram.AutoRunActivity");
            } else if (getMobileType().equals("HUAWEI")) { 
                //华为 p10 android 8.0
				//componentName = ComponentName.unflattenFromString("com.huawei.systemmanager/.appcontrol.activity.StartupAppControlActivity");
                //荣耀 H60-l03 android 4.4.2 
                componentName = ComponentName.unflattenFromString("com.huawei.systemmanager/.optimize.bootstart.BootStartActivity");
				//componentName = new ComponentName("com.huawei.systemmanager", "com.huawei.systemmanager.optimize.process.ProtectActivity");
            } else if (getMobileType().equals("vivo")) { 
				// vivo x7只能进入安全管家的界面
				componentName = ComponentName.unflattenFromString("com.iqoo.secure/.MainActivity");
				//componentName = ComponentName.unflattenFromString("com.iqoo.secure/.safeguard.PurviewTabActivity");
				// vivo x7 进入手机白名单界面
				//componentName = ComponentName.unflattenFromString("com.iqoo.secure/.ui.phoneoptimize.AddWhiteListActivity");
            } else if (getMobileType().equals("Meizu")) { 
				// 魅族
                componentName = ComponentName.unflattenFromString("com.meizu.safe/.permission.SmartBGActivity");
				//componentName = ComponentName.unflattenFromString("com.meizu.safe/.permission.PermissionMainActivity");
            } else if (getMobileType().equals("OPPO")) { 
				// OPPO R9m 可以
				componentName = ComponentName.unflattenFromString("com.coloros.safecenter/.startupapp.StartupAppListActivity");
				//componentName = ComponentName.unflattenFromString("com.oppo.safe/.permission.startup.StartupAppListActivity");
                //耗电列表
				//componentName = ComponentName.unflattenFromString("com.coloros.oppoguardelf/com.coloros.powermanager.fuelgaue.PowerUsageModelActivity");
            } else if (getMobileType().equals("ulong")) { 
				// 360手机 
                componentName = new ComponentName("com.yulong.android.coolsafe", ".ui.activity.autorun.AutoRunListActivity");
            } else {
                // 将用户引导到系统设置页面
                if (Build.VERSION.SDK_INT >= 9) {
                    intent.setAction("android.settings.APPLICATION_DETAILS_SETTINGS");
                    intent.setData(Uri.fromParts("package", context.getPackageName(), null));
                } else if (Build.VERSION.SDK_INT <= 8) {
                    intent.setAction(Intent.ACTION_VIEW);
                    intent.setClassName("com.android.settings", "com.android.settings.InstalledAppDetails");
                    intent.putExtra("com.android.settings.ApplicationPkgName", context.getPackageName());
                }
            }
            intent.setComponent(componentName);
            context.startActivity(intent);
        } catch (Exception e) {
		    //抛出异常就直接打开设置页面
			Log.e("StartUtils", "异常信息:" + e.getMessage().toString());
            intent = new Intent(Settings.ACTION_SETTINGS);
            context.startActivity(intent);
        }
    }
	```
	
从上面的代码中，我们可以的发现，目标Activity的全称路径。在不同手机厂商中是不同的，在同一手机厂商不同型号、系统版本手机中，也有可能是不一样的，这就需要我们做多类型的兼容。
> 推荐一种实现的思路：
		app端获取到手机型号、版本号等信息并调用后台的接口，以参数的形式，将信息传递给后台，后台根据不同的参数，返回网页链接（这里的链接可能是多个不同的url，根据手机型号、系统版本做匹配，可以参考企业微信的网页链接，适配就做的比较全面），app端打开网页链接，在h5界面中点击前往设置的按钮，调用原生的方法，进而跳转到系统的自启动界面。
