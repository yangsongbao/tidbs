# 环境
个人笔记本，本地部署，采用默认配置，部署指令
```bash
tiup playground v4.0.0 --db 1 --pd 3 --kv 3 --monitor
```
笔记本软硬件环境
 ```bash
Darwin yangsongbaodeMacBook-Pro.local 18.7.0 Darwin Kernel Version 18.7.0: Thu Jun 20 18:42:21 PDT 2019; root:xnu-4903.270.47~4/RELEASE_X86_64 x86_64

//cpu+内存
  处理器名称：	Intel Core i7
  处理器速度：	2.5 GHz
  处理器数目：	1
  核总数：	2
  L2 缓存（每个核）：	256 KB
  L3 缓存：	4 MB
  超线程技术：	已启用
  内存：	16 GB

//磁盘
  可用：	35.69 GB（35,687,268,352 字节）
  容量：	250.69 GB（250,685,575,168 字节）
  装载点：	/
  文件系统：	APFS
 ```

 # sysbench
 ## 压测配置文件
```bash
#sysbench-thread-8.cfg 

mysql-host=127.0.0.1
mysql-port=4000
mysql-user=root
mysql-password=
mysql-db=test
time=60
threads=8
report-interval=10
db-driver=mysql
```
## 准备测试数据
```bash
sysbench --config-file=sysbench-thread-8.cfg oltp_point_select --tables=32 --table-size=100000 prepare
```
## 开始压测
```bash
sysbench --config-file=sysbench-thread-n.cfg oltp_update_index --tables=32 --table-size=100000 run
```
## 压测结果
### 结果数据
|  type  | thread  | tps  | qps  | min latency  |avg latency|95th latency	|max latency
|  ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  |
| oltp_point_select  | 4 |64066.56 |64066.56 |0.20 |0.37 |0.59 |90.18 |
| oltp_point_select  | 8 |66561.91 |66561.91 |0.21 |0.72 |1.39 |81.87 | 
| oltp_point_select  | 16 |87891.83 |87891.83 |0.22 |1.09 |1.89 | 22.03|
| oltp_point_select  | 32 |91984.33 |91984.33 |0.24 |2.09 |4.18 |  40.11|
| oltp_point_select  | 64 |99126.33 |99126.33 |0.26 |3.87 |7.84 | 58.94|
| oltp_point_select  | 128 |102587.22 |102587.22 |0.29 |7.49 |14.46 | 177.60|

### 监控
#### TiDB Query Summary
![thread-4](./tidb-thread-4.png)
![thread-8](./tidb-thread-8.png)
![thread-16](./tidb-thread-16.png)
![thread-32](./tidb-thread-32.png)
![thread-64](./tidb-thread-64.png)
![thread-128](./tidb-thread-128.png)

#### TiKV Details Cluster

![thread-4](./tikv-cluster-4.png)
![thread-8](./tikv-cluster-8.png)
![thread-16](./tikv-cluster-16.png)
![thread-32](./tikv-cluster-32.png)
![thread-64](./tikv-cluster-64.png)
![thread-128](./tikv-cluster-128.png)

#### TiKV Details grpc
![thread-4](./tikv-grpc-4.png)
![thread-8](./tikv-grpc-8.png)
![thread-16](./tikv-grpc-16.png)
![thread-32](./tikv-grpc-32.png)
![thread-64](./tikv-grpc-64.png)
![thread-128](./tikv-grpc-128.png)

 # go-ycsb
 ## workloada
 ### 指令
 ```bash
 ./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=100000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 128
 ./bin/go-ycsb run mysql -P workloads/workloada -p operationcount=100000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 128
 ***************** properties *****************
"workload"="core"
"readallfields"="true"
"scanproportion"="0"
"recordcount"="1000"
"mysql.host"="127.0.0.1"
"insertproportion"="0"
"operationcount"="100000"
"requestdistribution"="uniform"
"readproportion"="0.5"
"dotransactions"="true"
"threadcount"="128"
"mysql.port"="4000"
"updateproportion"="0.5"
**********************************************
 ```
 ### 结果
|type| Takes(s)|Count|OPS|Avg(us)|Min(us)|Max(us)|99th(us)|99.9th(us)|99.99th(us)|
|  ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  |
|READ|9.9|26729|2689.4|8845|647|100538|31000|54000|81000|
|UPDATE|9.9|26851|2700.8|38401|707|440953|222000|286000|402000|
|READ|19.9|48380|2426.6|9712|626|141063|41000|86000|136000|
|UPDATE|19.9|48720|2443.2|42490|707|562110|228000|320000|447000|
|READ|21.1|49783|2364.1|9683|626|141063|42000|93000|136000|
|UPDATE|21.1|50185|2382.8|42318|707|562110|227000|322000|447000|
 ## workloadb
 ### 指令
 ```bash
 ./bin/go-ycsb load mysql -P workloads/workloadb -p recordcount=100000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 128
 ./bin/go-ycsb run mysql -P workloads/workloadb -p operationcount=100000 -p mysql.host=127.0.0.1 -p mysql.port=4000 --threads 128

***************** properties *****************
"insertproportion"="0"
"operationcount"="100000"
"mysql.port"="4000"
"readallfields"="true"
"threadcount"="128"
"scanproportion"="0"
"requestdistribution"="uniform"
"mysql.host"="127.0.0.1"
"recordcount"="1000"
"readproportion"="0.95"
"updateproportion"="0.05"
"dotransactions"="true"
"workload"="core"
**********************************************
```
|type| Takes(s)|Count|OPS|Avg(us)|Min(us)|Max(us)|99th(us)|99.9th(us)|99.99th(us)|
|  ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  | ----  |
|READ|10.0|79596|7991.8|13088|2000|105034|39000|65000|85000|
|UPDATE|9.9|4278|431.1|53740|7361|277117|105000|262000|278000|
|READ|12.9|94890|7361.6|14037|516|132244|47000|74000|93000|
|UPDATE|12.9|5078|395.1|56075|2310|357372|124000|262000|358000|

# 总结
没有部署集群，没有性能分析，仅熟悉了测试工具的使用，忘酌情给分。