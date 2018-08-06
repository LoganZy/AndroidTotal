## Android 中通过adb命令专区logcat日志

>  Android开发中，我们需要打印、抓取log，来判断、定位问题，使用Android Studio开发工具，在debug模式下我们可以很容易的抓取、捕获日志信息，快速的定位为题；但是当我们打Release包的时候，没有办法在编译工具上直观的查看log，所以就需要通过abd命令的形式实现。

1. 首先，需要`abd.exe`、`AbdWinApi.dll`、`AbdWinUsbApi.dll`，三个文件，这个在Android的sdk的platform_tools文件夹下面是会有的；若是没有，自行下载拷贝到相应目录即可。
2. 在Android Studio中，打开terminal窗口，或是在 abd.exe所在的文件夹中，打开cmd窗口。
3. 在terminal窗口中，输入`abd devices`命令，查看手机是否连接电脑。
	
		E:\kotlin_demo>adb devices
		List of devices attached  
		988b5a333851304d34      device

	**List of devices attached** -->表示设备已经连接。

 4. 继续在terminal中，输入`abd shell`命令。

		E:\kotlin_demo>adb shell
		dreamqltechn:/ $

 5. 继续输入`logcat -f /mnt/sdcard/demo_log.log`，`-f`是指写入文件到路径。
 
	    E:\kotlin_demo>adb shell
		dreamqltechn:/ $ logcat -f /mnt/sdcard/demo_log.log

 6. 复现问题，出现问题后按下Ctrl+C来取消抓取日志，再按下 Ctrl+d 退出adb shell模式
 7. 输入指令 `adb pull /mnt/sdcard/demo_log.log E:\kotlin_demo` ,开始把手机根目录下的demo_log.log文件复制到计算机上的`E:\kotlin_demo`文件路径下。
 
		E:\kotlin_demo>adb pull /mnt/sdcard/demo_log.log E:\kotlin_demo
8. 或是在terminal中输入命令直接将log打印出来，命令如下，logcat | grep `TAG`
	
	   E:\kotlin_demo>adb shell
	   dreamqltechn:/ $ logcat | grep kotlin_demo
	
