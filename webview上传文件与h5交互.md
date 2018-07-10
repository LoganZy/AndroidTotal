##  WebChromeClient之onShowFileChooser或openFileChooser使用及注意事项
`Android `开发使用WebView控件加载包含表单的H5网页，点击上传文件按钮，弹出对话框，选择从相册获取照片、拍照或打开手机文件管理器，从Android手机选取一张图片或一个文件，然后通过ValueCallback接口传递，在WebView加载的H5网页显示。这里有一个问题，点击“取消”或返回按钮，无法重复回调onShowFileChooser或openFileChooser方法。
控制台打印：**Attempted to finish an input event but the input event receiver has already been disposed**
		
	 /**
	  * Tell the client to show a file chooser.
	  * This is called to handle HTML forms with 'file' input 		type, in response to the
	  * user pressing the "Select File" button.
	  * To cancel the request, call <code>filePathCallback.onReceiveValue(null)</code> and
	  * return true.
	  * @param webView The WebView instance that is initiating the request.
	  * @param filePathCallback Invoke this callback to supply the list of paths to files to upload,
	  * or NULL to cancel. Must only be called if the
	  * <code>showFileChooser</code> implementations returns true.
	  * @param fileChooserParams Describes the mode of file chooser to be opened, and options to be
      * used with it.
	  * @return true if filePathCallback will be invoked, false to use default handling.
	  * @see FileChooserParams
	  */
	 public  boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback,
	  FileChooserParams fileChooserParams) {
			return  false;
	 }

该方法的作用，告诉当前APP，打开一个文件选择器，比如：打开相册、启动拍照或打开本地文件管理器，实际上更好的理解，WebView加载包含上传文件的表单按钮，HTML定义了input标签，同时input的type类型为file，手指点击该按钮，回调onShowFileChooser这个方法，在这个重写的方法里面打开相册、启动照片或打开本地文件管理器，甚至做其他任何的逻辑处理，点击一次回调一次的前提是请求被取消，而取消该请求回调的方法：给ValueCallback接口的onReceiveValue抽象方法传入null，同时onShowFileChooser方法返回true；

ValueCallback的抽象方法被回调onShowFileChooser方法返回true；反之返回false；再来看一下openFileChooser的源码，如下：

	  /**
	   * Tell the client to open a file chooser.
	   * @param uploadFile A ValueCallback to set the URI of the file to upload.
	   * onReceiveValue must be called to wake up the thread.a
	   * @param acceptType The value of the 'accept' attribute of the input tag
	   * associated with this file picker.
	   * @param capture The value of the 'capture' attribute of the input tag
	   * associated with this file picker.
	   *
	   * @deprecated Use {@link #showFileChooser} instead.
	   * @hide This method was not published in any SDK version.
	   */
	   @SystemApi
	   @Deprecated
	   public  void openFileChooser(ValueCallback<Uri> uploadFile, String acceptType, String capture) {
          uploadFile.onReceiveValue(null);
	   }
在所有发布的SDK版本中，openFileChooser是一个隐藏的方法，使用onShowFileChooser代替，但是最好同时重写showFileChooser和openFileChooser方法，**Android 4.4.X以上**的系统回调onShowFileChooser方法，**低于或等于Android 4.4.X**的系统回调openFileChooser方法，只重写onShowFileChooser或openFileChooser造成在有的系统可以正常回调，在有的系统点击没有反应。

仔细分析onShowFileChooser和openFileChooser回调方法，这两个方法之间的区别，

第一个区别：前者ValueCallback接口回传一个Uri数组，后者回传一个Uri对象，在onActivityResult回调方法中调用ValueCallback接口方法onReceiveValue传入参数特别注意；	
		
	/**
	 *回调onShowFileChooser方法，onReceiveValue传入Uri对象数组
	 */
	 mFilePathCallback.onReceiveValue(new Uri[]{uri});

	/**
	 *回调openFileChooser方法，onReceiveValue传入一个Uri对象
	 */
	 mFilePathCallback4.onReceiveValue(uri);
	
H5表单写入两个上传文件的按钮，点击其中一个从底部弹出对话框，选择相册文件或拍照，点击“取消”按钮，再次点击“上传文件”按钮能够再次回调onShowFileChooser或openFileChooser方法。

在之前的理解中，误解onShowFileChooser或openFileChooser只能打开相册或启动相机拍照，其实不仅仅是这样，onShowFileChooser或openFileChooser既然是一个回调的方法，可以重复执行各种逻辑代码，比如：启动另一个Activity、弹窗对话框、录制视频或录音等

在上面的例子中，执行弹窗操作，将弹窗的处理代码放置onShowFileChooser或openFileChooser方法体，如下：

	@Override
	public  boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
		super.onShowFileChooser(webView, filePathCallback, fileChooserParams);
		popupDialog();
		PickPhotoUtil.mFilePathCallback = filePathCallback;
		//返回true，如果filePathCallback被调用；返回false，如果忽略处理
		return  true;
	}

或

	public  void openFileChooser(ValueCallback<Uri> filePathCallback, String acceptType, String capture) {
		popupDialog();
		String title = acceptType;
		PickPhotoUtil.mFilePathCallback4 = filePathCallback;
	}

点击弹窗取消按钮、点击打开相册取消操作或取消拍照，可能无法再次回调onShowFileChooser或openFileChooser方法，如果你没有在点击弹窗取消方法中或onActivityResult回调方法resultCode==RESULT_CANCELED处理，再次点击上传按钮，打印出log：

**Attempted to finish an input event but the input event receiver has already been disposed**

同时，点击没有效果

	 /**
	  * 弹窗，启动拍照或打开相册
	  */
	 public  void popupDialog() {
		 ActionSheetDialog actionSheetDialog= new ActionSheetDialog(activity).builder()
			 .setCancelable(false)
			 .setCanceledOnTouchOutside(false)
			 .addSheetItem("手机拍照", ActionSheetDialog.SheetItemColor.Blue,
				 new ActionSheetDialog.OnSheetItemClickListener() {
					 @Override
					 public  void onClick(int which) {
						 goToTakePhoto();
					 }
				  })
			  .addSheetItem("手机相册", ActionSheetDialog.SheetItemColor.Blue,
				 new ActionSheetDialog.OnSheetItemClickListener() {
					 @Override
					 public  void onClick(int which) {
						  goForPicFile();
					 }
				 });
		actionSheetDialog.show();
	/**
	 * 设置点击“取消”按钮监听，目的取消mFilePathCallback回调，可以重复调起弹窗
	 */
		actionSheetDialog.setOnClickListener(new View.OnClickListener() {
			@Override
			public  void onClick(View v) {
				cancelFilePathCallback();
			}
		 });
	  }
	  
	/**
	 *取消mFilePathCallback回调
	 */
	private  void cancelFilePathCallback() {
		if (PickPhotoUtil.mFilePathCallback4 != null) {
			PickPhotoUtil.mFilePathCallback4.onReceiveValue(null);
			PickPhotoUtil.mFilePathCallback4 = null;
		} else  if (PickPhotoUtil.mFilePathCallback != null) {
			PickPhotoUtil.mFilePathCallback.onReceiveValue(null);
			PickPhotoUtil.mFilePathCallback = null;
		}
	}

在不期待回调mFilePathCallback的onReceiveValue方法时，调用**cancelFilePathCallback()**，解决点击上传按钮无法重复回调的问题。
