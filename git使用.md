---


---

<h2 id="git的使用与错误">git的使用与错误</h2>
<ol>
<li>
<p>使用Git pull文件时，出现"error: RPC failed; curl 18 transfer closed with outstanding read data remaining<br>
错误信息如下：</p>
<pre><code>error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
</code></pre>
<p>1.缓存区溢出curl的postBuffer的默认值太小，需要增加缓存</p>
<p>使用git命令增大缓存（<strong>单位是b</strong>，524288000B也就500M左右）</p>
<p><strong>git config --global http.postBuffer 524288000</strong></p>
<p>使用<strong>git config --list</strong>查看是否生效</p>
<p>此时重新克隆即可</p>
<p>2.网络下载速度缓慢</p>
<p>修改下载速度</p>
<pre><code>git config --global http.lowSpeedLimit 0 git config --global http.lowSpeedTime 999999
</code></pre>
<p>3.以上两种方式依旧无法clone下，尝试以浅层clone，然后更新远程库到本地</p>
<pre><code>git clone --depth=1 http://xxx.git
git fetch --unshallow
</code></pre>
</li>
</ol>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

