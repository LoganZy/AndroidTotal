---


---

<h2 id="mac-os-配置java的环境变量">Mac OS 配置java的环境变量</h2>
<ol>
<li>下载java的jdk安装包，前往<a href="https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html">Oracle</a>下载</li>
<li>下载完成后进行安装,Mac会默认安装到:资料库/Java/JavaVirtualMachines/jdk1.8.0_212.jdk，找打jdk的根目录，并复制路径<code>/Library/Java/JavaVirtualMachines/jdk1.8.0_212.jdk/Contents/Home</code></li>
<li>打开terminal，运行如下命令<pre><code>//切换到home顶级目录
1.cd ~
//如果你是第一次配置环境变量，创建一个.bash_profile的隐藏配置文件
2.touch .bash_profile
//如果是已经创建过了，直接打开并编辑
3.open .bash_profile
</code></pre>
<blockquote>
<p>在新打开的界面中添加配置信息如下</p>
</blockquote>
<pre><code>JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_212.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPATH
</code></pre>
保存编辑完的文件，在terminal中输入命令，使配置生效<pre><code>4. source .bash_profile
</code></pre>
之后运行java -verison 查看版本号和其他信息，成功信息如下：<pre><code>java version "1.8.0_212"
Java(TM) SE Runtime Environment (build 1.8.0_212-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.212-b10, mixed mode)
</code></pre>
</li>
</ol>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

