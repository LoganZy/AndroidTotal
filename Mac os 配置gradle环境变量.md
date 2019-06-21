---


---

<p>Mac OS 配置 Android stduio 中的gradle 环境变量</p>
<p>1.找到已经安装的gradle的文件路径</p>
<ul>
<li>打开<code>访达</code>–&gt;在左侧任务栏中选择 <code>应用程序</code>–&gt;找到Android stduio的图标并<code>右键</code>打开<code>显示包内容</code>–&gt;Contents–&gt;gradle–&gt;gradle-xxx版本–&gt;bin 路径。</li>
<li>复制bin下gradle文件的路径为<code>/Applications/Android Studio.app/Contents/gradle/gradle-5.4/bin</code><br>
其中<code>/Applications/Android</code>后面手动输入<code>\</code>，后面留有空格，最终如下：<br>
<code>/Applications/Android\ Studio.app/Contents/gradle/gradle-5.1.1/bin</code><br>
2.配置gradle路径</li>
<li>打开terminal，执行以下命令。<pre><code>1. cd ~
//到home目录下
2. touch .bash_profile
//在没有.bash_profile时会重新创建
3. open -e .bash_profile
//会以文本的形式打开文件
</code></pre>
在打开的文件中添加如下：<pre><code>export GRADLE_HOME=/Applications/Android\ Studio.app/Contents/gradle/gradle-5.1.1
export PATH=${PATH}:${GRADLE_HOME}/bin
</code></pre>
注：因为链接中<code>Android Studio.app</code>中间有空格，路径中不能带有空格之类的特殊字符，所以需要在空格前加\进行转义。<pre><code>4. source .bash_profile
//使修改生效
</code></pre>
在terminal中输入gradle -v查看是否出现版本号及其他信息,出现如下信息，则成功:<pre><code>------------------------------------------------------------
Gradle 5.1.1
------------------------------------------------------------
Build time:   2019-01-10 23:05:02 UTC
Revision:     3c9abb645fb83932c44e8610642393ad62116807

Kotlin DSL:   1.1.1
Kotlin:       1.3.11
Groovy:       2.5.4
Ant:          Apache Ant(TM) version 1.9.13 compiled on July 10 2018
JVM:          1.8.0_212 (Oracle Corporation 25.212-b10)
OS:           Mac OS X 10.14.2 x86_64
</code></pre>
</li>
<li>如果没有出现上图这种情况，是因为<code>gradle</code>和<code>gradle.bat</code>执行权限不够，需进行权限修改，执行如下命令<code>5</code>，进入到刚才的<code>bin</code>目录下，并使用命令<code>6</code>查看权限,如下：<pre><code>5. cd /Applications/Android\ Studio.app/Contents/gradle/gradle-5.1.1/bin
6. ls -l
7. -rwxr-xr-x  1 loganzy  admin  5297  6  3 11:27 gradle
8. -rwxr-xr-x  1 loganzy  admin  2261  6  3 11:27 gradle.bat
</code></pre>
<code>-rwxr-xr-x</code>若是没有x，说明没有可执行权限，因为我已经添加过权限，所以有可执行权限。依次执行如下命令，增加执行权限。<pre><code>9. chmod +x gradle
10. chmod +x gradle.bat
</code></pre>
重新打开<code>Mac</code>终端或者在<code>Android Studio</code>的<code>Terminal</code>中输入<code>gradle -v</code>，有输出说明配置成功。若是弹窗提示需要安装java的jdk才能使用此功能，那就需要安装并配置java环境变量。java环境变量配置完成后，运行gadle -v就能看到成功的配置信息了。</li>
</ul>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

