---


---

<h1 id="【high-performance-tidb】lesson-01：tidb-整体架构_homework"># 【High Performance TiDB】Lesson 01：TiDB 整体架构_Homework</h1>
<p>Author: y-bi<br>
Date:2020.08.16</p>
<p>（这段内容可略过）<br>
作为一个本业移动端实际啥都干的码农，一直很想体验后端的<s>从删库到跑路</s>让自己的代码承受着巨大流量轰炸的刺激感。而想成为后端工程师，数据库相关是永远绕不过去的，最重要的一座坚城，也是个人知识最欠缺的领域。<br>
希望通过这段时间的学习能让自己在成为想要成为的，真正意义上的全栈工程师路上，离目标更近一点。</p>
<h1 id="题目描述">题目描述</h1>
<blockquote>
<p>本地下载 TiDB，TiKV，PD 源代码，改写源码并编译部署以下环境：</p>
<ul>
<li>1 TiDB</li>
<li>1 PD</li>
<li>3 TiKV<br>
改写后：使得 TiDB 启动事务时，能打印出一个 “hello transaction” 的日志</li>
</ul>
<p>输出：一篇文章介绍以上过程</p>
</blockquote>
<h1 id="编译阶段">编译阶段</h1>
<p>本次的Homework基于以下环境</p>
<pre><code>OS: macOS Catalina 10.15.5
Memory: 32GB
Xcode: 11.6 with CLT
</code></pre>
<p>首先从github上clone TiDB，TiKV，PD源码。为了方便今后的操作，我新建了一个目录 <code>TiDB/src/github.com/pingcap</code>并将它们放置在此目录下。之后的所有操作将以此目录为根目录。</p>
<pre><code>git clone https://github.com/pingcap/tidb
git clone https://github.com/tikv/tikv
git clone https://github.com/pingcap/pd
</code></pre>
<p>之后便是分别对这三个源码进行编译。其中TiDB需要go环境支持，而TiKV则复杂得多（后述）。</p>
<p>虽然三者编译顺序并无影响，但我们还是按照从上往下的顺序来编译把。</p>
<h2 id="tidb的编译">TiDB的编译</h2>
<p>在安装好go环境支持后，在根目录执行以下命令：</p>
<pre><code>cd tidb
make
</code></pre>
<p>如果一切顺利，你就可以看到<code>./tidb/bin/</code>中生成了一个tidb-server文件。</p>
<h2 id="tikv编译">TiKV编译</h2>
<p>参照TiKV的<a href="https://github.com/tikv/tikv/blob/master/CONTRIBUTING.md">文档</a>，可以看出需要以下组件：</p>
<blockquote>
<p>To build TiKV you’ll need to at least have the following installed:</p>
<ul>
<li><code>git</code>  - Version control</li>
<li><a href="https://rustup.rs/"><code>rustup</code></a>  - Rust installer and toolchain manager</li>
<li><code>make</code>  - Build tool (run common workflows)</li>
<li><code>cmake</code>  - Build tool (required for gRPC)</li>
<li><code>awk</code>  - Pattern scanning/processing language</li>
</ul>
</blockquote>
<p>在mac上通过homebrew配置这些环境很简单，不再赘述。唯一需要注意的一点是请提前安装好XCode Commandline Tool，不然在安装需要gcc的部件时会遇上<a href="https://stackoverflow.com/questions/24966404/brew-install-gcc-too-time-consuming">这样的麻烦</a>。</p>
<p>在安装好环境支持后，在根目录执行以下命令：</p>
<pre><code>cd tikv
make build
</code></pre>
<p>如果一切顺利，你就可以看到<code>./tikv/target/debug/</code>中生成了一系列文件。其中我们需要的是tikv-server和tikv-ctl文件。</p>
<h2 id="pd的编译">PD的编译</h2>
<p>在安装好TiDB编译所需go环境后，PD无需多余的环境配置即可编译。</p>
<p>在根目录执行以下命令：</p>
<pre><code>cd pd
make
</code></pre>
<p>如果一切顺利，你就可以看到<code>./pd/bin/</code>中生成了一系列文件。其中我们需要的是pd-server和pd-ctl文件。</p>
<h1 id="部署阶段">部署阶段</h1>
<p>为了方便操作，我在根目录下建立了一个<code>./bin/</code>目录，将所需的文件全部copy到其中。在根目录执行以下命令：</p>
<pre><code>cp ./pd/bin/pd-server ./bin/
cp ./pd/bin/pd-ctl ./bin/
cp ./tikv/target/debug/tikv-server ./bin/
cp ./tidb/bin/tidb-server ./bin/
</code></pre>
<p>接着按照题目要求，我们需要启动1个TiDB，1个PD，3个TiKV组成一个最小的集群。</p>
<h2 id="pd的部署">PD的部署</h2>
<p>由于TiKV应该以cluster的形式运行，而扮演cluster的管理者角色的就是PD，所以首先我们应该启动PD</p>
<pre><code>./bin/pd-server --name=pd1 \
            --data-dir=pd1 \
            --client-urls="http://127.0.0.1:2379" \
            --peer-urls="http://127.0.0.1:2380" \
            --initial-cluster="pd1=http://127.0.0.1:2380" \
            --log-file=pd1.log
</code></pre>
<h2 id="tikv的部署">TiKV的部署</h2>
<p>接着我们在另外终端里开启三个TiKV，将其连接到PD上</p>
<pre><code>./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                --addr="127.0.0.1:20160" \
                --data-dir=tikv1 \
                --log-file=tikv1.log

./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                --addr="127.0.0.1:20161" \
                --data-dir=tikv2 \
                --log-file=tikv2.log

./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                --addr="127.0.0.1:20162" \
                --data-dir=tikv3 \
                --log-file=tikv3.log
</code></pre>
<p>当PD和TiKV部署完成后，可执行以下命令来测试是否正常工作</p>
<pre><code>./bin/pd-ctl store -d -u http://127.0.0.1:2379
</code></pre>
<p>终端会输出类似于如下内容的json</p>
<pre><code>{
  "count": 3,
  "stores": [
    {
      "store": {
        "id": 1,
        "address": "127.0.0.1:20160",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "ae7a6ecee6e3367da016df0293a9ffe9cc2b5705",
        "start_timestamp": 1597547759,
        "deploy_path": "/Users/y-bi/TiDB/src/github.com/pingcap/./bin",
        "last_heartbeat": 1597547890050963000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "465.6GiB",
        "available": "141.1GiB",
        "used_size": "28.81MiB",
        "leader_count": 1,
        "leader_weight": 1,
        "leader_score": 1,
        "leader_size": 1,
        "region_count": 1,
        "region_weight": 1,
        "region_score": 1,
        "region_size": 1,
        "start_ts": "2020-08-16T12:15:59+09:00",
        "last_heartbeat_ts": "2020-08-16T12:18:10.050963+09:00",
        "uptime": "2m11.050963s"
      }
    },
    {
      "store": {
        "id": 1001,
        "address": "127.0.0.1:20161",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "ae7a6ecee6e3367da016df0293a9ffe9cc2b5705",
        "start_timestamp": 1597547868,
        "deploy_path": "/Users/y-bi/TiDB/src/github.com/pingcap/./bin",
        "last_heartbeat": 1597547888454864000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "465.6GiB",
        "available": "141.1GiB",
        "used_size": "28.8MiB",
        "leader_count": 0,
        "leader_weight": 1,
        "leader_score": 0,
        "leader_size": 0,
        "region_count": 1,
        "region_weight": 1,
        "region_score": 1,
        "region_size": 1,
        "start_ts": "2020-08-16T12:17:48+09:00",
        "last_heartbeat_ts": "2020-08-16T12:18:08.454864+09:00",
        "uptime": "20.454864s"
      }
    },
    {
      "store": {
        "id": 1003,
        "address": "127.0.0.1:20162",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "ae7a6ecee6e3367da016df0293a9ffe9cc2b5705",
        "start_timestamp": 1597547888,
        "deploy_path": "/Users/y-bi/TiDB/src/github.com/pingcap/./bin",
        "state_name": "Up"
      },
      "status": {
        "capacity": "0B",
        "available": "0B",
        "used_size": "0B",
        "leader_count": 0,
        "leader_weight": 1,
        "leader_score": 0,
        "leader_size": 0,
        "region_count": 0,
        "region_weight": 1,
        "region_score": 0,
        "region_size": 0,
        "start_ts": "2020-08-16T12:18:08+09:00",
        "last_heartbeat_ts": "1970-01-01T09:00:00+09:00"
      }
    }
  ]
}
</code></pre>
<p>我们可以看到三个TiKV都以连接上，且<code>state_name</code>都是UP，说明工作的很好。</p>
<h2 id="tidb的部署">TiDB的部署</h2>
<p>在根目录通过以下命令进行部署：</p>
<pre><code>./bin/tidb-server --store=tikv \
                  --path="127.0.0.1:2379" \
                  --log-file=tidb.log
</code></pre>
<p>之后就可以看到tidb.log文件中输出了。</p>
<pre><code>["server is running MySQL protocol"]
</code></pre>
<p>我们再启动SQL看看。执行</p>
<pre><code>mysql -h 127.0.0.1 -P 4000 -u root -D test
</code></pre>
<p>终端会输出：</p>
<pre><code>Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 1

Server version: 5.7.25-TiDB-v4.0.0-beta.2-960-g5184a0d70-dirty TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible
...
</code></pre>
<p>OK, MySQL客户端连接上TiDB了。</p>
<p>我们再查看一下集群，mysql下执行：</p>
<pre><code>select * from information_schema.cluster_info;
</code></pre>
<p>终端会输出：</p>
<pre><code>+------+-----------------+-----------------+--------------+------------------------------------------+---------------------------+---------------+
| TYPE | INSTANCE        | STATUS_ADDRESS  | VERSION      | GIT_HASH                                 | START_TIME                | UPTIME        |
+------+-----------------+-----------------+--------------+------------------------------------------+---------------------------+---------------+
| tidb | 0.0.0.0:4000    | 0.0.0.0:10080   | 4.0.0-beta.2 | 5184a0d7060906e2022d18f11532f119f5df3f39 | 2020-08-16T12:20:53+09:00 | 12m9.240076s  |
| pd   | 127.0.0.1:2379  | 127.0.0.1:2379  | 4.1.0-alpha  | 865fbd82a028aecfb875a20b932fee3ba4b8c73c | 2020-08-16T12:15:37+09:00 | 17m25.24008s  |
| tikv | 127.0.0.1:20160 | 127.0.0.1:20180 | 4.1.0-alpha  | ae7a6ecee6e3367da016df0293a9ffe9cc2b5705 | 2020-08-16T12:15:59+09:00 | 17m3.240082s  |
| tikv | 127.0.0.1:20161 | 127.0.0.1:20180 | 4.1.0-alpha  | ae7a6ecee6e3367da016df0293a9ffe9cc2b5705 | 2020-08-16T12:17:48+09:00 | 15m14.240084s |
| tikv | 127.0.0.1:20162 | 127.0.0.1:20180 | 4.1.0-alpha  | ae7a6ecee6e3367da016df0293a9ffe9cc2b5705 | 2020-08-16T12:18:08+09:00 | 14m54.240085s |
+------+-----------------+-----------------+--------------+------------------------------------------+---------------------------+---------------+
5 rows in set (0.01 sec)
</code></pre>
<p>部署完成！</p>
<h1 id="改写源码阶段">改写源码阶段</h1>
<p>由于自己对于数据库相关经验基本为零，这个阶段会记录很多思路，可能会比较啰嗦。</p>
<h2 id="审题（收集信息）">审题（收集信息）</h2>
<p>题目要求的是“使得 TiDB 启动事务时，能打印出一个 ‘hello transaction’ 的日志”。那么首先，我们得先明确什么事事务，摘抄一下WIKI：</p>
<blockquote>
<p>数据库事务通常包含了一个序列的对数据库的读/写操作。包含有以下两个目的：</p>
<ol>
<li>为数据库操作序列提供了一个从失败中恢复到正常状态的方法，同时提供了数据库即使在异常状态下仍能保持一致性的方法。</li>
<li>当多个<a href="https://zh.wikipedia.org/wiki/%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F" title="应用程序">应用程序</a>在<a href="https://zh.wikipedia.org/wiki/%E5%B9%B6%E5%8F%91" title="并发">并发</a>访问数据库时，可以在这些应用程序之间提供一个隔离方法，以防止彼此的操作互相干扰。</li>
</ol>
<p>并非任意的对数据库的操作序列都是数据库事务。数据库事务拥有以下四个特性，习惯上被称之为**<a href="https://zh.wikipedia.org/wiki/ACID" title="ACID">ACID特性</a>**。</p>
<ul>
<li><strong>原子性（Atomicity）</strong>：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行<a href="https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1#cite_note-acid-3">[3]</a>。</li>
<li><strong>一致性（Consistency）</strong>：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。_一致状态_的含义是数据库中的数据应满足完整性约束<a href="https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1#cite_note-acid-3">[3]</a>。</li>
<li><strong>隔离性（Isolation）</strong>：多个事务并发执行时，一个事务的执行不应影响其他事务的执行<a href="https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1#cite_note-acid-3">[3]</a>。</li>
<li><strong>持久性（Durability）</strong>：已被提交的事务对数据库的修改应该永久保存在数据库中<a href="https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%8B%E5%8A%A1#cite_note-acid-3">[3]</a>。</li>
</ul>
<p><a href="https://zh.wikipedia.org/wiki/SQL" title="SQL">SQL</a>国际标准使用<code>START TRANSACTION</code>开始一个事务（也可以用方言命令<code>BEGIN</code>）。<code>[COMMIT](https://zh.wikipedia.org/w/index.php?title=Commit_(SQL)&amp;action=edit&amp;redlink=1 "Commit (SQL)（页面不存在）")</code>语句使事务成功完成。<code>[ROLLBACK](https://zh.wikipedia.org/wiki/%E5%9B%9E%E6%BB%9A_(%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86) "回滚 (数据管理)")</code>语句结束事务，放弃从<code>BEGIN TRANSACTION</code>開始的一切变更。若<a href="https://zh.wikipedia.org/wiki/Autocommit" title="Autocommit">autocommit</a>被<code>START TRANSACTION</code>的使用禁止，在事务结束时autocommit會重新啟用。</p>
</blockquote>
<p>此外，对于单条SQL语句，数据库系统自动将其作为一个事务执行，这种事务被称为隐式事务。</p>
<h2 id="推理">推理</h2>
<p>从审题阶段我们可以提取出解题需要的几个关键字：</p>
<p>启动（ <code>START TRANSACTION</code>，<code>BEGIN</code>），<br>
事务（<code>transaction</code>），<br>
打印日志。</p>
<p>那么对于一个对于数据库一窍不通，没有读过源码的人，要怎么解决这个问题呢？</p>
<p>首先，这次我们有三个程序的源码，必须先确定需要修改的程序是哪一个。根据课程视频，TiKV实现存储，PD实现集群管理。而事务本质上，是由一个有限的数据库操作组成的集合。因此很明显，数据库操作，不应该属于TiKV和PD，而应属于TiDB的范畴。因此可以确定我们需要修改的代码应该在TiDB代码中。</p>
<p>之后自然是在官方文档中搜索“事务”，很快就找到了 <a href="https://docs.pingcap.com/zh/tidb/stable/transaction-overview">TiDB事务概览</a>，这里面的信息和从wiki收集的信息以及上面的推断互相映照。那么可以确定，我们需要修改的地方一定是在 事务（<code>transaction</code>）的启动（ <code>START TRANSACTION</code>，<code>BEGIN</code>）代码周边的范围。</p>
<p>再详细的搜索一下，我们就可以发现 <code>tidb/kv/kv.go</code>中，定义了事务相关的接口，尤其store中定义了开启事务相关的接口：</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token comment">// Storage defines the interface for storage.</span>
<span class="token comment">// Isolation should be at least SI(SNAPSHOT ISOLATION)</span>
<span class="token keyword">type</span> Storage <span class="token keyword">interface</span> <span class="token punctuation">{</span>
    <span class="token comment">// Begin transaction</span>
    <span class="token function">Begin</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">(</span>Transaction<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span>
    <span class="token comment">// BeginWithStartTS begins transaction with startTS.</span>
    <span class="token function">BeginWithStartTS</span><span class="token punctuation">(</span>startTS <span class="token builtin">uint64</span><span class="token punctuation">)</span> <span class="token punctuation">(</span>Transaction<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span>
    <span class="token operator">...</span>
<span class="token punctuation">}</span>
</code></pre>
<p>因为我们这次使用的是TiKV进行存储，我们可以在 <code>./store/tikv/kv.go</code> 找到具体实现代码。如下：</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token keyword">func</span> <span class="token punctuation">(</span>s <span class="token operator">*</span>tikvStore<span class="token punctuation">)</span> <span class="token function">Begin</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">(</span>kv<span class="token punctuation">.</span>Transaction<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	txn<span class="token punctuation">,</span> err <span class="token operator">:=</span> <span class="token function">newTiKVTxn</span><span class="token punctuation">(</span>s<span class="token punctuation">)</span>
	<span class="token keyword">if</span> err <span class="token operator">!=</span> <span class="token boolean">nil</span> <span class="token punctuation">{</span>
		<span class="token keyword">return</span> <span class="token boolean">nil</span><span class="token punctuation">,</span> errors<span class="token punctuation">.</span><span class="token function">Trace</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">return</span> txn<span class="token punctuation">,</span> <span class="token boolean">nil</span>
<span class="token punctuation">}</span>

<span class="token comment">// BeginWithStartTS begins a transaction with startTS.</span>
<span class="token keyword">func</span> <span class="token punctuation">(</span>s <span class="token operator">*</span>tikvStore<span class="token punctuation">)</span> <span class="token function">BeginWithStartTS</span><span class="token punctuation">(</span>startTS <span class="token builtin">uint64</span><span class="token punctuation">)</span> <span class="token punctuation">(</span>kv<span class="token punctuation">.</span>Transaction<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	txn<span class="token punctuation">,</span> err <span class="token operator">:=</span> <span class="token function">newTikvTxnWithStartTS</span><span class="token punctuation">(</span>s<span class="token punctuation">,</span> startTS<span class="token punctuation">,</span> s<span class="token punctuation">.</span><span class="token function">nextReplicaReadSeed</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
	<span class="token keyword">if</span> err <span class="token operator">!=</span> <span class="token boolean">nil</span> <span class="token punctuation">{</span>
		<span class="token keyword">return</span> <span class="token boolean">nil</span><span class="token punctuation">,</span> errors<span class="token punctuation">.</span><span class="token function">Trace</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">return</span> txn<span class="token punctuation">,</span> <span class="token boolean">nil</span>
<span class="token punctuation">}</span>
</code></pre>
<p>在这里我们可以看出，这里的开启事务实质上就是获取一个新的<code>TiKVTxn</code>，进入 <code>./store/tikv/txn.go</code>查看更深一步的定义：</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token keyword">func</span> <span class="token function">newTiKVTxn</span><span class="token punctuation">(</span>store <span class="token operator">*</span>tikvStore<span class="token punctuation">)</span> <span class="token punctuation">(</span><span class="token operator">*</span>tikvTxn<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	bo <span class="token operator">:=</span> <span class="token function">NewBackofferWithVars</span><span class="token punctuation">(</span>context<span class="token punctuation">.</span><span class="token function">Background</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> tsoMaxBackoff<span class="token punctuation">,</span> <span class="token boolean">nil</span><span class="token punctuation">)</span>
	startTS<span class="token punctuation">,</span> err <span class="token operator">:=</span> store<span class="token punctuation">.</span><span class="token function">getTimestampWithRetry</span><span class="token punctuation">(</span>bo<span class="token punctuation">)</span>
	<span class="token keyword">if</span> err <span class="token operator">!=</span> <span class="token boolean">nil</span> <span class="token punctuation">{</span>
		<span class="token keyword">return</span> <span class="token boolean">nil</span><span class="token punctuation">,</span> errors<span class="token punctuation">.</span><span class="token function">Trace</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">return</span> <span class="token function">newTikvTxnWithStartTS</span><span class="token punctuation">(</span>store<span class="token punctuation">,</span> startTS<span class="token punctuation">,</span> store<span class="token punctuation">.</span><span class="token function">nextReplicaReadSeed</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token punctuation">}</span>

<span class="token comment">// newTikvTxnWithStartTS creates a txn with startTS.</span>
<span class="token keyword">func</span> <span class="token function">newTikvTxnWithStartTS</span><span class="token punctuation">(</span>store <span class="token operator">*</span>tikvStore<span class="token punctuation">,</span> startTS <span class="token builtin">uint64</span><span class="token punctuation">,</span> replicaReadSeed <span class="token builtin">uint32</span><span class="token punctuation">)</span> <span class="token punctuation">(</span><span class="token operator">*</span>tikvTxn<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	ver <span class="token operator">:=</span> kv<span class="token punctuation">.</span><span class="token function">NewVersion</span><span class="token punctuation">(</span>startTS<span class="token punctuation">)</span>
	snapshot <span class="token operator">:=</span> <span class="token function">newTiKVSnapshot</span><span class="token punctuation">(</span>store<span class="token punctuation">,</span> ver<span class="token punctuation">,</span> replicaReadSeed<span class="token punctuation">)</span>
	<span class="token keyword">return</span> <span class="token operator">&amp;</span>tikvTxn<span class="token punctuation">{</span>
		snapshot<span class="token punctuation">:</span>  snapshot<span class="token punctuation">,</span>
		us<span class="token punctuation">:</span>        kv<span class="token punctuation">.</span><span class="token function">NewUnionStore</span><span class="token punctuation">(</span>snapshot<span class="token punctuation">)</span><span class="token punctuation">,</span>
		store<span class="token punctuation">:</span>     store<span class="token punctuation">,</span>
		startTS<span class="token punctuation">:</span>   startTS<span class="token punctuation">,</span>
		startTime<span class="token punctuation">:</span> time<span class="token punctuation">.</span><span class="token function">Now</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
		valid<span class="token punctuation">:</span>     <span class="token boolean">true</span><span class="token punctuation">,</span>
		vars<span class="token punctuation">:</span>      kv<span class="token punctuation">.</span>DefaultVars<span class="token punctuation">,</span>
	<span class="token punctuation">}</span><span class="token punctuation">,</span> <span class="token boolean">nil</span>
<span class="token punctuation">}</span>
</code></pre>
<p>可以看到，最终代码都归结到了<code>func newTikvTxnWithStartTS(store *tikvStore, startTS uint64, replicaReadSeed uint32) (*tikvTxn, error)</code>这个方法签名上。那么在这里增加输出日志的语句，应该就能达到我们想要的效果。</p>
<p>改写如下：</p>
<pre class=" language-go"><code class="prism  language-go"><span class="token comment">// newTikvTxnWithStartTS creates a txn with startTS.</span>
<span class="token keyword">func</span> <span class="token function">newTikvTxnWithStartTS</span><span class="token punctuation">(</span>store <span class="token operator">*</span>tikvStore<span class="token punctuation">,</span> startTS <span class="token builtin">uint64</span><span class="token punctuation">,</span> replicaReadSeed <span class="token builtin">uint32</span><span class="token punctuation">)</span> <span class="token punctuation">(</span><span class="token operator">*</span>tikvTxn<span class="token punctuation">,</span> <span class="token builtin">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	ver <span class="token operator">:=</span> kv<span class="token punctuation">.</span><span class="token function">NewVersion</span><span class="token punctuation">(</span>startTS<span class="token punctuation">)</span>
	snapshot <span class="token operator">:=</span> <span class="token function">newTiKVSnapshot</span><span class="token punctuation">(</span>store<span class="token punctuation">,</span> ver<span class="token punctuation">,</span> replicaReadSeed<span class="token punctuation">)</span>
	<span class="token comment">// Print a "hello transaction" when begin a transaction</span>
	logutil<span class="token punctuation">.</span><span class="token function">BgLogger</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">Info</span><span class="token punctuation">(</span><span class="token string">"hello transaction"</span><span class="token punctuation">,</span> zap<span class="token punctuation">.</span><span class="token function">Uint64</span><span class="token punctuation">(</span><span class="token string">"startTS"</span><span class="token punctuation">,</span> startTS<span class="token punctuation">)</span><span class="token punctuation">)</span>
	<span class="token keyword">return</span> <span class="token operator">&amp;</span>tikvTxn<span class="token punctuation">{</span>
		snapshot<span class="token punctuation">:</span>  snapshot<span class="token punctuation">,</span>
		us<span class="token punctuation">:</span>        kv<span class="token punctuation">.</span><span class="token function">NewUnionStore</span><span class="token punctuation">(</span>snapshot<span class="token punctuation">)</span><span class="token punctuation">,</span>
		store<span class="token punctuation">:</span>     store<span class="token punctuation">,</span>
		startTS<span class="token punctuation">:</span>   startTS<span class="token punctuation">,</span>
		startTime<span class="token punctuation">:</span> time<span class="token punctuation">.</span><span class="token function">Now</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
		valid<span class="token punctuation">:</span>     <span class="token boolean">true</span><span class="token punctuation">,</span>
		vars<span class="token punctuation">:</span>      kv<span class="token punctuation">.</span>DefaultVars<span class="token punctuation">,</span>
	<span class="token punctuation">}</span><span class="token punctuation">,</span> <span class="token boolean">nil</span>
<span class="token punctuation">}</span>
</code></pre>
<p>重新编译TiDB并运行，测试一些sql查询。OK可以在<code>TiDB.log</code>中看到<code>hello transaction</code>了！不过好像数量有点多是怎么回事？还一直在增长…会不会是课程视频上说的ddl worker和gc worker造成的？那log level调整为debug看看吧。</p>
<pre><code>./bin/tidb-server -L debug --store=tikv \
                  --path="127.0.0.1:2379" \
                  --log-file=tidb.log
</code></pre>
<p>重启TiDB后发现果然如此，log文件中部分内容如下：</p>
<pre><code>[2020/08/16 14:46:18.268 +09:00] [DEBUG] [ddl.go:210] ["[ddl] check whether is the DDL owner"] [isOwner=true] [selfID=9fb0292d-3ce5-4136-93d2-553dfd694c7a]
[2020/08/16 14:46:19.266 +09:00] [DEBUG] [ddl_worker.go:148] ["[ddl] wait to check DDL status again"] [worker="worker 2, tp add index"] [interval=1s]
[2020/08/16 14:46:19.266 +09:00] [DEBUG] [ddl_worker.go:148] ["[ddl] wait to check DDL status again"] [worker="worker 1, tp general"] [interval=1s]
[2020/08/16 14:46:19.267 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924340498433]
[2020/08/16 14:46:19.267 +09:00] [DEBUG] [ddl.go:210] ["[ddl] check whether is the DDL owner"] [isOwner=true] [selfID=9fb0292d-3ce5-4136-93d2-553dfd694c7a]
[2020/08/16 14:46:19.267 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924340498434]
[2020/08/16 14:46:19.267 +09:00] [DEBUG] [ddl.go:210] ["[ddl] check whether is the DDL owner"] [isOwner=true] [selfID=9fb0292d-3ce5-4136-93d2-553dfd694c7a]
[2020/08/16 14:46:19.430 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924380082177]
[2020/08/16 14:46:19.461 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924393189377]
[2020/08/16 14:46:19.461 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924393189378]
[2020/08/16 14:46:19.463 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924393189379]
[2020/08/16 14:46:20.266 +09:00] [DEBUG] [ddl_worker.go:148] ["[ddl] wait to check DDL status again"] [worker="worker 2, tp add index"] [interval=1s]
[2020/08/16 14:46:20.266 +09:00] [DEBUG] [ddl_worker.go:148] ["[ddl] wait to check DDL status again"] [worker="worker 1, tp general"] [interval=1s]
[2020/08/16 14:46:20.267 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924602904578]
[2020/08/16 14:46:20.267 +09:00] [DEBUG] [ddl.go:210] ["[ddl] check whether is the DDL owner"] [isOwner=true] [selfID=9fb0292d-3ce5-4136-93d2-553dfd694c7a]
[2020/08/16 14:46:20.267 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924602904579]
[2020/08/16 14:46:20.267 +09:00] [DEBUG] [ddl.go:210] ["[ddl] check whether is the DDL owner"] [isOwner=true] [selfID=9fb0292d-3ce5-4136-93d2-553dfd694c7a]
[2020/08/16 14:46:21.266 +09:00] [DEBUG] [ddl_worker.go:148] ["[ddl] wait to check DDL status again"] [worker="worker 1, tp general"] [interval=1s]
[2020/08/16 14:46:21.267 +09:00] [DEBUG] [ddl_worker.go:148] ["[ddl] wait to check DDL status again"] [worker="worker 2, tp add index"] [interval=1s]
[2020/08/16 14:46:21.267 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924866097153]
[2020/08/16 14:46:21.267 +09:00] [DEBUG] [ddl.go:210] ["[ddl] check whether is the DDL owner"] [isOwner=true] [selfID=9fb0292d-3ce5-4136-93d2-553dfd694c7a]
[2020/08/16 14:46:21.267 +09:00] [INFO] [txn.go:99] ["hello transaction"] [startTS=418789924866097154]
</code></pre>
<p>OK，这题到这里应该算是解答完成了。</p>
<h2 id="思考">思考</h2>
<p>这次的修改直接针对KV API Layer接口针对TiKV的具体实现中进行了修改，但如果切换底层存储引擎，修改就没用了。而go不支持interface中默认实现，不支持重载重写，暂时没有想出如何在 不增加或改变现有接口&amp;不在每个具体实现中写同样的代码 的情况下做到输出同样的日志。</p>

