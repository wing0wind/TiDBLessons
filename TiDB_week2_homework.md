---


---

<h1 id="【high-performance-tidb】lesson-02：对-tidb-进行基准测试_homework">【High Performance TiDB】Lesson 02：对 TiDB 进行基准测试_Homework</h1>
<pre><code>分值：300  
  
题目描述：  
  
使用 sysbench、go-ycsb 和 go-tpc 分别对  
TiDB 进行测试并且产出测试报告。  
  
测试报告需要包括以下内容：  
  
* 部署环境的机器配置(CPU、内存、磁盘规格型号)，拓扑结构(TiDB、TiKV 各部署于哪些节点)  
* 调整过后的 TiDB 和 TiKV 配置  
* 测试输出结果  
* 关键指标的监控截图  
* TiDB Query Summary 中的 qps 与 duration  
* TiKV Details 面板中 Cluster 中各 server 的 CPU 以及 QPS 指标  
* TiKV Details 面板中 grpc 的 qps 以及 duration  
  
输出：写出你对该配置与拓扑环境和 workload 下 TiDB 集群负载的分析，提出你认为的 TiDB 的性能的瓶颈所在(能提出大致在哪个模块即 可)  
  
截止时间：下周二（8.25）24:00:00(逾期提交不给分)
</code></pre>
<h2 id="部署">部署</h2>
<p>由于条件有限，只能在一台虚拟机上进行测试。<br>
顺便吐槽一下tiup cluster对于mac的支持非常不行，最终只能使用tiup在虚拟机里进行部署。</p>
<h3 id="测试环境配置：">测试环境配置：</h3>
<p>CentOS8, 4Core CPU, 16GB Memory, 512GB SATA Hard Disk.</p>
<h3 id="拓扑结构：">拓扑结构：</h3>
<pre><code>Starting component cluster:  display tidb-test3node
tidb Cluster: tidb-test3node
tidb Version: v4.0.0
ID               Role          Host       Ports        OS/Arch       Status      Data Dir                      Deploy Dir
--               ----          ----       -----        -------       ------      --------                      ----------
127.0.0.1:9093   alertmanager  127.0.0.1  9093/9094    linux/x86_64  activating  /tidb-data/alertmanager-9093  /tidb-deploy/alertmanager-9093
127.0.0.1:3000   grafana       127.0.0.1  3000         linux/x86_64  activating  -                             /tidb-deploy/grafana-3000
127.0.0.1:2379   pd            127.0.0.1  2379/2380    linux/x86_64  Up          /tidb-data/pd-2379            /tidb-deploy/pd-2379
127.0.0.1:9090   prometheus    127.0.0.1  9090         linux/x86_64  activating  /tidb-data/prometheus-9090    /tidb-deploy/prometheus-9090
127.0.0.1:4000   tidb          127.0.0.1  4000/10080   linux/x86_64  Up          -                             /tidb-deploy/tidb-4000
127.0.0.1:4001   tidb          127.0.0.1  4001/10081   linux/x86_64  Up          -                             /tidb-deploy/tidb-4001
127.0.0.1:4002   tidb          127.0.0.1  4002/10082   linux/x86_64  Up          -                             /tidb-deploy/tidb-4002
127.0.0.1:20160  tikv          127.0.0.1  20160/20180  linux/x86_64  Up          /tidb-data/tikv-20160         /tidb-deploy/tikv-20160
127.0.0.1:20161  tikv          127.0.0.1  20161/20181  linux/x86_64  Up          /tidb-data/tikv-20161         /tidb-deploy/tikv-20161
127.0.0.1:20162  tikv          127.0.0.1  20162/20182  linux/x86_64  Up          /tidb-data/tikv-20162         /tidb-deploy/tikv-20162
</code></pre>
<h2 id="测试">测试</h2>
<h3 id="go-ycsb-测试">go-ycsb 测试</h3>
<p>准备数据：</p>
<pre><code>./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 16

Run finished, takes 3m25.649780058s
INSERT - Takes(s): 205.3, Count: 10000, OPS: 48.7, Avg(us): 111192, Min(us): 22262, Max(us): 358679, 99th(us): 228000, 99.9th(us): 349000, 99.99th(us): 350000
</code></pre>
<p>测试结果：</p>
<pre><code>./bin/go-ycsb run mysql -P workloads/workloada -p recordcount=10000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 16

Run finished, takes 3.164680955s
READ   - Takes(s): 3.2, Count: 484, OPS: 153.2, Avg(us): 2628, Min(us): 328, Max(us): 174433, 99th(us): 15000, 99.9th(us): 175000, 99.99th(us): 175000
UPDATE - Takes(s): 3.0, Count: 508, OPS: 172.0, Avg(us): 83150, Min(us): 12893, Max(us): 217830, 99th(us): 206000, 99.9th(us): 218000, 99.99th(us): 218000
</code></pre>
<p>关键指标截图：</p>
<ul>
<li>TiDB Query Summary 中的 qps 与 duration</li>
<li></li>
<li>TiKV Details 面板中 Cluster 中各server 的 CPU 以及 QPS 指标</li>
<li></li>
<li>TiKV Details 面板中 grpc 的 qps 以及 duration</li>
<li>Dashboard面板</li>
</ul>
<h3 id="go-tpc-测试">go-tpc 测试</h3>
<p>准备数据：</p>
<pre><code> ./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 10 prepare
</code></pre>
<p>测试结果：</p>
<pre><code> ./bin/go-tpc tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 10 run --time 5m

Finished
[Summary] DELIVERY - Takes(s): 338.2, Count: 24, TPM: 4.3, Sum(ms): 47810, Avg(ms): 1992, 90th(ms): 4000, 99th(ms): 4000, 99.9th(ms): 4000
[Summary] NEW_ORDER - Takes(s): 356.3, Count: 258, TPM: 43.4, Sum(ms): 181665, Avg(ms): 704, 90th(ms): 1000, 99th(ms): 1500, 99.9th(ms): 1500
[Summary] NEW_ORDER_ERR - Takes(s): 356.3, Count: 1, TPM: 0.2, Sum(ms): 115, Avg(ms): 115, 90th(ms): 128, 99th(ms): 128, 99.9th(ms): 128
[Summary] ORDER_STATUS - Takes(s): 329.4, Count: 29, TPM: 5.3, Sum(ms): 1665, Avg(ms): 57, 90th(ms): 192, 99th(ms): 256, 99.9th(ms): 256
[Summary] PAYMENT - Takes(s): 358.8, Count: 225, TPM: 37.6, Sum(ms): 121556, Avg(ms): 540, 90th(ms): 1000, 99th(ms): 1000, 99.9th(ms): 1000
[Summary] STOCK_LEVEL - Takes(s): 357.7, Count: 25, TPM: 4.2, Sum(ms): 6453, Avg(ms): 258, 90th(ms): 1000, 99th(ms): 1500, 99.9th(ms): 1500
tpmC: 43.4
</code></pre>
<p>关键指标截图：</p>
<ul>
<li>TiDB Query Summary 中的 qps 与 duration</li>
<li></li>
<li>TiKV Details 面板中 Cluster 中各server 的 CPU 以及 QPS 指标</li>
<li></li>
<li>TiKV Details 面板中 grpc 的 qps 以及 duration</li>
</ul>
<h2 id="结论">结论</h2>
<p>目前环境下瓶颈不处于TiDB上。TiKV层的IO中，特别是插入相关耗时较多，推测大多由锁导致，涉及模块 raftstore ，Scheduler。</p>

