# Android中Gradle的基础构建
# Gradle构建基础

在AndroidStudio创建一个安卓项目时会自动生成三个Gradle文件，其中两个build.gradle和一个settings.gradle文件。他们的后缀都是.gradle，并且如果在项目中创建一个module也会随之生成一个build.gradle文件。初始化后的这三个gradle文件结构如下所示： 

 ![](http://p981u1am0.bkt.clouddn.com/18-6-8/98261915.jpg)

-   根目录下的settings.build文件在初始化阶段被执行，并且定义了那些模块应该包含在构建内
    -   本例中只有一个app模块，故只有一行`include ':app'`
    -   若还有其他一些模块，比如mylibrary库，应该为`include ':app', ':mylibrary'`
-   根目录下的build.gradle文件默认会包含两个代码块，分别如下：

		buildscript {
		
		    repositories {
		    
		        jcenter()
		        
		    }
		    
		    dependencies {
		    
		        classpath 'com.android.tools.build:gradle:2.2.2'
		        
		        // NOTE: Do not place your application dependencies here; they belong
		        
		        // in the individual module build.gradle files
		        
		    }
		    
		}
		
		allprojects {
		
		    repositories {
		    
		        jcenter()
		        
		    }
		    
		}

实际构建配置在`buildscript`代码块中，`repositories`将jcenter配置为一个仓库，和编译有关的依赖库都是从jcenter仓库中拉取；`dependencies`将配置构建过程中的gradle插件的依赖包，默认情况下只定义Android插件（有了`classpath 'com.android.tools.build:gradle:2.2.2'`这句话后随后的app中的build.gradle文件才可以使用android代码块变量以及一些有关Android的属性）。

`allprojects`代码块可用来声明那些需要用于所有模块的属性，甚至可以在该代码块下创建任务，这些任务最终会被运用到所有模块中。

-   app模块下的build.gradle文件所定义的属性或者任务只能应用在该app模块下，它可以覆盖根下的build.gradle文件所声明属性。
	默认的AndroidStudio会生成如下代码：
		
		#Android应用插件，根下的build.gradle文件要配置安卓构建工具`classpath 'com.android.tools.build:gradle:2.2.2'`的依赖才能使用
		apply plugin: 'com.android.application'
		
		android {
		
		    compileSdkVersion 25 #用来编译应用的Android API版本
		    
		    buildToolsVersion "25.0.0" #构建工具和编译器使用的版本号
		    
		    #改代码块下的属性会覆盖AndroidManifest中声明的相关属性
		    
		    defaultConfig { 
		    
		        #覆盖manifest中的packagename，但是它和package name还有不同，它应用在

		        #variants多版本构建时至关重要（多版本要有不同的包名才能安装到同一设备上）

		        applicationId "com.icedcap.myapplication" 

		        minSdkVersion 22 #对应manifest <uses-sdk>

		        targetSdkVersion 25 #对应manifest <uses-sdk>

		        versionCode 1 #和manifest的该属性意义相同

		        versionName "1.0" #和manifest的该属性意义相同

		        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
		        
		    }
		    
		    # 该代码块用来定义如何构建和打包不同类型的应用，这将在后续详细指出
		    buildTypes { 
		    
		        release {
		        
		            minifyEnabled false
		            
		            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
		            
		        }
		        
		    }

		}
	

# app项目所要依赖的库要在该出声明

	dependencies { 

	    compile fileTree(dir: 'libs', include: ['*.jar'])

	    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {

	        exclude group: 'com.android.support', module: 'support-annotations'})

	    compile 'com.android.support:appcompat-v7:25.0.1'

	    testCompile 'junit:junit:4.12'

	}

# Gradle任务和简单的自定义构建

谷歌的工程师在开发Android的Gradle插件时也是利用依赖进行开发的。首先Gradle的Android插件使用了Java的基础插件，而Java基础插件又使用了Gradle基础插件，基础插件中定义了任务的标准生命周期和一些共同约定的属性。基础插件中定义了`assemble`和`clean`任务，Java插件定义了`check`和`build`任务。在基础插件中`assemble`和`clean`任务既不被实现，也不执行任何操作，它类似于编程语言中的接口或者是抽象类由依赖该任务的其他插件来具体实现，例如在[Gradle 基础介绍](https://github.com/zylaoshi/AndroidTotal/blob/master/Android%E4%B8%ADGradle%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D.md)列出的所有Gradle任务以及它的依赖，其中`app:assemble`要依赖`app:assembleDebug`和`app:assembleRelease`而`app:installDebug`要依赖`app:assembleDebug`。

对于底层插件定义的任务约定如下：

-   assemble：集合项目的输出
-   clean：清理项目的输出
-   check：运行所有的检查，通常是单元测试和集成测试
-   build：同时运行assemble和check

而对于安卓Gradle插件来说是在上述约定的基础扩展而来，具体任务约定如下：

-   assemble：为每一个构建版本创建一个APK文件
-   clean：删除所有的构建内容，例如APK文件以及编译时产生的资源R文件等
-   check：运行Lint检查，如果Lint发现问题则停止构建
-   build：同时运行assemble和check任务

Android的Gradle插件除了扩展了这四个基础任务之外，还自定义了很多有用的任务，在[Gradle 基础介绍](https://github.com/zylaoshi/AndroidTotal/blob/master/Android%E4%B8%ADGradle%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D.md)中也列出了一个Android项目所有的任务以及详细描述和依赖情况。

这里列出几个平时用的比较多的任务：

-   connectedCheck：在连接设备或者模拟器上运行测试
-   deviceCheck：一个占位任务，专为其他插件在远端设备上运行测试
-   installDebug和installRelease：在连接设备上或模拟器上安装特定版本
-   uninstallDebug和uninstallRelease：卸载相应的版本

好了，说完了基本的构建任务外再来谈谈简单的自定义构建。

简单的自定义构建也是平时项目中常常用到的比较有灵气的Gradle小技巧，主要包括以下几块内容：

-   BuildConfig设置
-   项目范围设置
-   项目属性设置
-   默认任务设置

## BuildConfig设置

从SDK工具升级到17以后，构建工具会在项目中生成一个叫`BuildConfig`的类，默认的使用`generateDebugSources`（点击AndroidStudio的Gradle同步按钮也会执行该任务）构建时，会生成如下信息：

	/**

	 * Automatically generated file. DO NOT MODIFY

	 */

	package com.icedcap.myapplication;

	public final class BuildConfig {

		  public static final boolean DEBUG = Boolean.parseBoolean("true");

		  public static final String APPLICATION_ID = "com.icedcap.myapplication";

		  public static final String BUILD_TYPE = "debug";

		  public static final String FLAVOR = "";

		  public static final int VERSION_CODE = 1;

		  public static final String VERSION_NAME = "1.0";

	}

包括`DEBUG`  `APPLICATION_ID`  `BUILD_TYPE`  `FLAVOR`  `VERSION_CODE`  `VERSION_NAME`所以我们在使用Debug版本时可以引用到该类中的属性。应用在实际场景中，比如构建debug版应用时项目内访问测试服务器，当发布正式版本时要访问正式的服务器，这时候就可以定义两个版本的服务器URL从而在不同版本上使用不同的URL，同理，对于http请求的日志只想在debug版本中看到。

	buildTypes {

        debug {

            buildConfigField "String", "APP_URL", "\"http://debug.test.com\""

            buildConfigField "boolean", "LOG_HTTP_CALLS", "true"

            ...

        }

        release {

            buildConfigField "String", "APP_URL", "\"http://release.test.com\""

            buildConfigField "boolean", "LOG_HTTP_CALLS", "false"

            ...

        }

    }

**注意：** 这里的url地址要使用转移符号将双引号连带地址一并传入BuildConfig类中。

可以看到分别使用assembleDebug和assembleRelease构架时生成的BuildConfig类内容分别如下：

	/**

	 * Automatically generated file. DO NOT MODIFY

	 */

	package com.icedcap.myapplication;

	public final class BuildConfig {

		  public static final boolean DEBUG = Boolean.parseBoolean("true");

		  public static final String APPLICATION_ID = "com.icedcap.myapplication";

		  public static final String BUILD_TYPE = "debug";

		  public static final String FLAVOR = "";

		  public static final int VERSION_CODE = 1;

		  public static final String VERSION_NAME = "1.0";

		  // Fields from build type: debug

		  public static final String APP_URL = "http://debug.test.com";

		  public static final boolean LOG_HTTP_CALLS = true;

	}



	/**

	 * Automatically generated file. DO NOT MODIFY

	 */

	package com.icedcap.myapplication;

	public final class BuildConfig {

		  public static final boolean DEBUG = false;

		  public static final String APPLICATION_ID = "com.icedcap.myapplication";

		  public static final String BUILD_TYPE = "release";

		  public static final String FLAVOR = "";

		  public static final int VERSION_CODE = 1;

		  public static final String VERSION_NAME = "1.0";

		  // Fields from build type: release

		  public static final String APP_URL = "http://release.test.com";

		  public static final boolean LOG_HTTP_CALLS = false;

	}

## 项目范围设置

针对多模块的Android项目，通常在根目录下的build.gradle文件的`allprojects`代码块进行配置时，它将应用在各个子模块中。

	allprojects {

	    apply plugin: 'com.android.application'

	    android {

	        compileSdkVersion 23

	        buildToolsVersion "25.0.0"
	
	    }

	}

因为Gradle允许在Project对象上添加额外的属性，这就意味着任何gradle.build文件都能定义额外的属性，添加额外的属性需要使用`ext`代码块

	ext {

	    local = "hello world, from buid.gradle"

	    compileSdkVersion = 22

	    buildToolsVersion = "22.0.1"

	}

在各个模块的build.gradle文件中可以引用根下build.gradle使用ext声明的变量：

	android {

	    compileSdkVersion rootProject.ext.compileSdkVersion

	    buildToolsVersion rootProject.ext.buildToolsVersion

	}

## 项目属性

当然上述在根下的build.gradle文件下使用`ext`声明变量外，还有一些其他方法声明属性变量为模块中的build.gradle来引用：

-   ext代码块
-   gradle.properties文件
-   -P命令行参数

ext在前文中声明了一个local变量，同理在`gradle.properties`文件下也声明一个`propertiesFile`变量并赋值，`propertiesFile=hello world from properties`然后我们在根下的build.gradle文件中定义一个打印任务来把这些变量输出出来，具体如下代码

	task printProperties << {

	    println local

	    println propertiesFile

	    if (project.hasProperty('cmd')) {

	        println cmd

	    }

	}

使用命令`gradle printProperties -P cmd="hello world from cmd"`输出如下：  

	:printProperties

	hello world, from buid.gradle

	hello world from properties

	hello world from cmd

## 默认任务配置

通常在根下的build.gradle文件中使用`defaultTask`来定义默认任务：

	defaultTasks 'clean', 'iD'

这里我配置了`clean`和`installDebug`为默认的任务，每次命令行输入gradle或者./gradlew不加任何任务名时会默认构建`clean`和`installDebug`两个任务。

# 依赖管理

依赖通常指的是外部依赖，例如其他开发者提供的依赖库，谷歌官方提供的support库等等。当不使用依赖仓库的时候，通常是先下载jar或者aar包放到项目中手动去引用它。而加入了依赖管理机制会很方便的使用到这些依赖库包括该依赖库的名称、版本号等信息。

在[Gradle 基础介绍](https://github.com/zylaoshi/AndroidTotal/blob/master/Android%E4%B8%ADGradle%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D.md)已经介绍了`repositories`代码块，在该代码块下添加仓库，默认的只有jcenter仓库：

	repositories {

	    jcenter()

	}

Gradle支持三种不同的依赖仓库`Maven`  `Ivy`以及静态文件或文件夹（JCenter是Maven的超集）。而通常一个依赖由三种元素定义它们分别是`group`  `name`  和`version`，`group`指的是创建该依赖库的组织通常是反向域名例如io.reactivex，`name`是该依赖的唯一标示例如rxjava，`version`则为依赖库的版本号，使用这三个元素就可以在`dependencies`中声明一个依赖了：

	dependencies {

	    compile fileTree(dir: 'libs', include: ['*.jar'])

	    testCompile 'junit:junit:4.12'

	    compile 'com.android.support:appcompat-v7:24.1.0'

	    compile 'com.android.support:cardview-v7:24.1.0'

	    compile 'com.android.support:design:24.1.0'

	    compile 'com.android.support:support-v13:24.1.0'

	    compile 'com.android.support:recyclerview-v7:24.1.0'

	    compile 'com.jakewharton:butterknife:7.0.1'

	    compile 'com.squareup.okhttp3:okhttp:3.3.1'

	    compile 'com.squareup.okhttp3:okhttp-urlconnection:3.3.1'

	    compile 'io.reactivex:rxandroid:1.2.1'

	    compile 'io.reactivex:rxjava:1.1.6'

	    compile 'com.google.code.gson:gson:2.7'

	    compile 'com.squareup.retrofit2:retrofit:2.1.0'

	    compile 'com.squareup.retrofit2:converter-gson:2.1.0'

	    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'

	    compile 'org.apache.httpcomponents:httpcore:4.3.2'

	    compile 'com.android.support:support-v4:24.1.0'

	    compile 'com.github.bumptech.glide:glide:3.7.0'

	}

## 远程仓库依赖

远程仓库依赖就是指在指定的仓库中拉取要使用的依赖库，通常Gradle预定义了三个Maven仓库：`JCenter`  `Maven Central`和本地`Maven Local`仓库，为了在构建脚本中使用他们所以要在`repositories`代码块中包含它们：

	repositories {

	    jcenter()

	    mavenCentral()

	    mavenLocal()

	}

`jcenter`和`mavenCentral`是两个有名的远程仓库，一般不同时使用他们，通常推荐使用`jcenter`。`jcenter`是`Maven Central`的超集，而且它支持HTTPS传输。`mavenLocal`是已经使用过的所有依赖的本地缓存，当然也可以自己向本地添加依赖。默认的它会以隐藏文件存储在home目录下的.m2文件夹下。

很多大厂或者有实力的开发者开发的插件或者依赖库，并且被其很多开发者使用的时候。他更喜欢把该库放在自己的Maven或者Ivy服务器上，而不是统一发布在Jcenter或者Maven Central，为了在自己的项目中使用这个依赖库，这时要在maven代码块中添加url

	repositories {

	  maven {

	    url "http://repo.example.com/maven2"

	  }

	  ivy {

	    url "http://repo.example.com/repo"

	  }

	  //自己团队使用的自己的远程仓库

	  maven {

	    url "http://repo.mycompany.com/maven2"

	    credentials {

	      username 'user'

	      password 'password'

	    }

	  }

	  //本机仓库,把路径作为url

	  maven {

	    url "../repo"

	  }

	  //除了把路径作为url还可以使用flatDir代码块

	  flatDir {

	    dirs 'aars'
	
	  }

	}

**注意：**不建议在配置文件中存储任何密码或者凭证，因为构建配置文件是纯文本它会被迁入代码控制系统中，从而导致密码泄露。最好的办法是使用前文提到的单独的属性文件定义这些变量并且赋值。

## 本地文件（夹）依赖

把单独文件作为依赖库可以使用`files`：  

	dependencies {

	    compile files('libs/xxx.jar')

	    ...

	  }

把一个目录下单所以jar文件添加到依赖：  

	dependencies {

	    compile fileTree(dir: 'libs', include: ['*.jar'])

	    ...

	}

对于原生依赖库可以使用`sourceSet`资源集来定义它，默认的原生库文件夹命名为jniLibs，并且在该文件夹下创建每一个平台如`armeabi`  `armeabi-v7a`  `mips`以及`x86`，将每一个平台下的so文件放在相应的文件夹下，在gradle配置如下：

	android {

	    sourceSets.main {

	        jniLibs.srcDir `src/main/libs`

	    }

	}

## 项目依赖以及依赖类型

当我们在项目中新建一个模块用于库项目时，我们要在该模块的gradle配置一句`apply plugin: 'com.android.library'`这句话的意思是为该模块加入library插件以至于该模块被构建为共享库并且生成jar或者aar包。当然除了定义该插件外还要记得在[Gradle 基础介绍](https://github.com/zylaoshi/AndroidTotal/blob/master/Android%E4%B8%ADGradle%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D.md)经过的在setting.gradle文件中加入该模块`include ':app', ':library'`。

当我们在app下使用该库的话可以在`dependencies`代码块下定义它的依赖：  

	dependencies {

	    compile(name:'libraryname', ext:'aar')

	}

上述代码定义的依赖，构建app时会去`libraryname`项目下找aar包。

当我们使用AndroidStudio创建一个新的安卓项目时在`dependencies`代码块下包括以下几个库

	dependencies {

	    compile fileTree(dir: 'libs', include: ['*.jar'])

	    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {

	        exclude group: 'com.android.support', module: 'support-annotations'})

	    compile 'com.android.support:appcompat-v7:25.0.1'

	    testCompile 'junit:junit:4.12'

	}

我们可以看到除了`compile`命令外还有`testCompile`和`androidTestCompile`，这些命令统称为scope也可以通过IDE的Project Structure可以看到。  
除了上述AndroidStudio创建项目时默认添加的三个外完整的scope依赖类型包括如下：

-   compile 默认的配置，在编译主应用时包含所有的依赖。该配置不仅会将依赖添加到类路径，还会生成对应的apk
-   apk 该依赖只会被打包进apk，而不会添加到编译类路径
-   provided 该类型的依赖刚好相反，它不会被打包进apk，该类型与apk类型的依赖只适用于JAR依赖，如果试图在依赖项目中添加他们将会出错。
-   testCompile和androidTestCompile类型的依赖会添加用于测试的额外依赖库，在运行测试相关的任务时，这些配置会被使用。

# 多版本构建

当开发一个应用时，通常会有几个不同的版本。最常见的情况是，你有一个手动测试用于保证质量的测试版本和一个生产版本。这些版本通常有不同的配置，例如**BuildConfig设置**小节讲到的几个变量在debug和release两个版本的不同。除此之外，你的应用有可能还有一个免费版和一个额外功能付费版。在这种情况下，你需要处理四中不同的版本，它们分别是：免费测试版、付费测试版、免费正式版、付费正式版。所以涉及到这种多版本的构建也是很伤脑筋的，幸好Gradle支持多版本构建。

在这里我们先把debug和release两个版本称为**构建类型**把类似于测试版和付费版命名为**product flavor（产品定制）**。Gradle在构建含有product flavor的项目时会结合构建类型（debug和release）一起生产每个product flavor 的debug和release版本。

所以下面我们就来探讨下构建类型和product flavor。

## 构建类型

在Gradle的Android插件中，构建类型通常被定义为如何构建一个应用或者依赖库。你可以在`buildTypes`代码块中来定义构建类型：

	buildTypes {

        release {

            minifyEnabled false //清除无用的资源

            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

        }
        
	}

使用AndroidStudio时会自动生成上述的代码，你可以在`release`代码块中定义其他有关release版本的信息。除此之外，Gradle的Android插件其实是默认已经配置好了`debug`的构建类型而且它是默认的构建类型，当然你也可以在上述的代码块中加入`debug`的配置来覆盖默认的配置。

除了`debug`和`release`两种我们常用的构建类型外其实我们可以自定义其他的构建类型。比如：

	buildTypes {

        ...

        myBuildType {

            applicationIdSuffix ".custom"

            versionNameSuffix "-custom"

            buildConfigField "String", "APP_URL","\"http://custom.test.com\""

        }

    }

`myBuildType`构建类型针对`applicationID`定义了一个新的后缀，使其和`debug`和`release`两个版本的`applicationID`不一样从而可以安装在同一台设备上。就像下面三个构建类型的包名一样：

-   **Debug :** com.package
-   **Release :** com.package
-   **myBuildType :** com.package.custom

在定义新的构建类型或者修改已有的构建类型时可以使用已经声明好的构建类型的配置进行初始化，从而减少配置代码的书写：

	buildTypes {

        ...

        staging.initWith(buildTypes.debug)

        staging {

            applicationIdSuffix ".staging"

            versionNameSuffix "-staging"

            debuggable false

        }

    }

`initWith()`方法创建一个新的构建类型，并且复制了一个已经存在的构建类型的所有属性到新的构建类型中。

无论是多构建类型还是多product flavor的定义都会产生多个sourceSet来存放这些不同构建类型和不同产品定制之间的不同资源文件，例如，一个_Constants.java_类文件在不同构建类型下肯定是不同的或者每个构建类型或定制产品的首页布局有所不同等等。
![](http://p981u1am0.bkt.clouddn.com/18-6-8/2949562.jpg)

这里提出一个问题当我在定义product flavor时也会定义不同的sourceSets，当在构建不同的的版本时比如付费版release版时这些源集是如何合并的呢？哈哈，先挖个坑，之后再填！

对于依赖来说，比如我只想在debug版本中添加logging框架怎么办？很简单上小节**依赖管理**我们也讲到了使用debugCompile的scope。

## product flavor

与构建类型不同product flavor是用来创建不同的版本，并且使用`productFlavor`代码块来创建不同版本：

	productFlavors {

	    red {

	        applicationId 'com.example.red'

	        versionCode 3

	    }

	    blue {

	        applicationId 'com.example.blue'

	        versionCode 4

	        minSdkVersion 14

	    }

	}

对于product flavor的源集，源集文件夹要以product flavor的名字命名。甚至可以为一个构建类型和一个productflavor的结合创建一个文件夹。比如，blue版本的release构建类型可以创建blueRelease名字的文件夹存放该版本所要使用到不同其他版本的资源文件。

在很多情况下productflavor可以按照维度来划分又可以拆成不同的版本，比如说，可以按照颜色和是否收费又可以划分为红色免费版、红色付费版、蓝色免费版、蓝色付费版等等。对于这种需求的版本划分就要使用`flavorDimensions`：flavorDimensions "color", "price"

    productFlavors {

        red {

            flavorDimension "color"

            applicationId 'com.example.red'

            versionCode 3

        }

        blue {

            flavorDimension "color"

            applicationId 'com.example.blue'

            versionCode 4

            minSdkVersion 14

        }

        free {

            flavorDimension "price"

        }

        paid {

            flavorDimension "price"

        }

    }

有了维度就意味着构建版本又可以组合为新的构建版本对于不同flavor配置了相同的属性那么就会按照维度设置的顺序进行覆盖。例如上面的color维度的属性将会覆盖price维度的属性。

## 构建variant

好了，开始填补上面挖的坑了。  
构建variant就是构建这些构建类型和productflavor的结合体，在我机器上配置了两个维度的productflavor和两个构建类型，然后分别生成如下的构建variant：

  ![](http://p981u1am0.bkt.clouddn.com/18-6-8/6152104.jpg)

有了构建variant就可以使用gradle命令构建了（IDE也有相应的按钮进行构建，不过笔者更喜欢全程使用命令行）。一个Android项目默认有debug和release两种构建类型所以可以使用`gradle assembleDebug`和`gradle assembleRelease`来分别构建两个版本的apk，单独使用`gradle assemble`命令是构建这两个版本的apk。当又定义了productflavor后呢？我们可以使用`gradle assembleRed`来构建red版的debug和release两个apk。使用`gradle assembleDebug`可以生产red版和blue版的测试版本apk，而使用`gradle assembleBlueDebug`只会生成blue的release版本。

**多版本结合如何合并sourceSets中的资源呢？**  
多版本结合后的sourceSet资源是按照一定的优先级进行合并覆盖的。首先Gradle的Android插件在打包应用之前将main文件夹下的代码和构建类型sourceSet下的代码合并在一起，此外library项目也可以提供额外的资源，这些也需要合并。这样同样适用于manifest文件。例如在应用的debug variant中可能需要额外的Android权限来存储log文件，而你不想在main里中声明该权限，因为会吓跑用户。相反可以在debug版本中单独定义sourceSet把该权限放到这个sourceSet中。所以资源以及manifest的有限构建顺序如下：

  
**BUILD TYPE**->**FLAVOR**->**MAIN**->**DEPENDENCIES**  

当然有时候对于不同版本的改动比较小使用sourceSet过于繁重，这时候我们可以直接在构建脚本中声明不同的资源id以及它的值，比如：

	productFlavors {

        red {

            applicationId 'com.example.red'

            versionCode 3

            resValue "color", "flavor_color", "#ff0000"

        }

    }

所以在构建red版本时候代码中使用`flavor_color`颜色的资源引用是`#ff0000`。

最后，所有的variant可以通过过滤来构建。比如如下配置：

	android.variantFilter { variant ->

	    if (variant.buildType.name.equals('release')) {

	        variant.getFlavors().each() { flavor ->

	            if (flavor.name.equals('blue')) {

	                variant.setIgnore(true);

	            }

	        }

	    }

	}

这时候我在查看我的所有variant如下：

  
![](http://p981u1am0.bkt.clouddn.com/18-6-8/80994048.jpg)
  
很显然过滤掉了release和blue的结合variant。

# 签名配置

对于签名配置来说比较简单，代码如下：

	signingConfigs{

	    release {

	        keyAlias 'myapp.keystore'

	        keyPassword 'xxx'

	        storePassword 'xxx'

	        storeFile file('myapp.keystore')

	    }

	}

	buildTypes {

	    ...
	
	    release {

	        signingConfig signingConfigs.release

	        minifyEnabled false

	        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

	    }

	}
