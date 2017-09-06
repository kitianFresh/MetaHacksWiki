---
title: "hadoop-install"
date: 2017-05-01 11:06
---
## hadoop download & checksum
proxychains wget http://www-eu.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3-src.tar.gz

proxychains wget http://www-eu.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3-src.tar.gz.mds

proxychains wget http://www-eu.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz

proxychains wget http://www-eu.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz.mds

## hadoop require accounts
可以进入另一个ubuntu 账号， 从这个账号通过ssh登录到 Hadoop。 Hadoop和Ubuntu一样拥有多账户管理， 这就是为什么 安装Hadoop的时候默认要求设置ssh。

例如， 我可以进入我的Ubuntu 的kinny 账户， 然后执行 `ssh hadoop@localhost` 进入本主机上的Hadoop账户(前提是 安装了 openssh-client 和 openssh-server, Ubuntu默认有客户端没有服务端。然后启动了ssh服务， 打开 20 端口。)， Hadoop账户拥有对Hadoop的管理权限， 实际上， Hadoop软件就安装在这同一台主机上， 不过你的 kinny账户没有对 Hadoop软件目录的管理权限而已。 但是你可以使用ssh 登录到 Hadoop账户即可。

## hdfs
```
./bin/hdfs dfs -mkdir -p /user/hadoop
```
接着将 `./etc/hadoop` 中的 xml 文件作为输入文件复制到分布式文件系统中，即将 `/usr/local/hadoop/etc/hadoop` 复制到分布式文件系统中的 `/user/hadoop/input` 中。我们使用的是 hadoop 用户，并且已创建相应的用户目录 `/user/hadoop` ，因此在命令中就可以使用相对路径如 input，其对应的绝对路径就是 `/user/hadoop/input`:

```
./bin/hdfs dfs -mkdir input
./bin/hdfs dfs -put ./etc/hadoop/*.xml input
```
复制完成后，可以通过如下命令查看文件列表：
```
./bin/hdfs dfs -ls input
```
伪分布式运行 MapReduce 作业的方式跟单机模式相同，区别在于伪分布式读取的是HDFS中的文件（可以将单机步骤中创建的本地 input 文件夹，输出结果 output 文件夹都删掉来验证这一点）。

```
./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep input output 'dfs[a-z.]+'
```
查看运行结果的命令（查看的是位于 HDFS 中的输出结果）：
```
./bin/hdfs dfs -cat output/*
```

## YARN
通过 `./sbin/start-dfs.sh` 启动 Hadoop，仅仅是启动了 MapReduce 环境，我们可以启动 YARN ，让 YARN 来负责资源管理与任务调度。

首先修改配置文件 `mapred-site.xml`，这边需要先进行重命名：
```
mv ./etc/hadoop/mapred-site.xml.template ./etc/hadoop/mapred-site.xml
```
然后再进行编辑，同样使用 gedit 编辑会比较方便些 `gedit ./etc/hadoop/mapred-site.xml`：

```xml
<configuration>
        <property>
             <name>mapreduce.framework.name</name>
             <value>yarn</value>
        </property>
</configuration>
```
接着修改配置文件 `yarn-site.xml`：
```xml
<configuration>
        <property>
             <name>yarn.nodemanager.aux-services</name>
             <value>mapreduce_shuffle</value>
            </property>
</configuration>
```
然后就可以启动 YARN 了（需要先执行过 `./sbin/start-dfs.sh`）：
```
./sbin/start-yarn.sh      # 启动YARN
./sbin/mr-jobhistory-daemon.sh start historyserver  # 开启历史服务器，才能在Web中查看任务运行情况
```
**但 YARN 主要是为集群提供更好的资源管理与任务调度，然而这在单机上体现不出价值，反而会使程序跑得稍慢些。因此在单机上是否开启 YARN 就看实际情况了。**


**不启动 YARN 需重命名 `mapred-site.xml`**
如果不想启动 YARN，务必把配置文件 `mapred-site.xml` 重命名，改成 `mapred-site.xml.template`，需要用时改回来就行。否则在该配置文件存在，而未开启 YARN 的情况下，运行程序会提示 “Retrying connect to server: 0.0.0.0/0.0.0.0:8032” 的错误，这也是为何该配置文件初始文件名为 `mapred-site.xml.template`。

同样的，关闭 YARN 的脚本如下：
```
./sbin/stop-yarn.sh
./sbin/mr-jobhistory-daemon.sh stop historyserver
```
# 参考
 - [Hadoop: Setting up a Single Node Cluster.](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

# 错误
Library you are using is compiled for 32 bit and you are using 64 bit version. so open your .bashrc file where configuration for hadoop exists. Go to this line

export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
and replace it with

export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib/native"

- [Unable to load native-hadoop library for your platform(已解决)](http://www.cnblogs.com/kevinq/p/5103653.html)


```java
package multicore;

public class ArraySumerDc extends java.lang.Thread{
	int[] a;
	int low;
	int high;
	int result = 0;
	ArraySumerDc(int[] a, int low, int high) {
		this.a = a;
		this.low = low;
		this.high = high;
	}
	
	public void run() {
		if (this.low == this.high) { // 只剩下一个结果的时候
			this.result = this.a[this.low];
		}
		else {// 还可以继续分解
			int mid = (this.low + this.high) / 2;
			ArraySumerDc left = new ArraySumerDc(this.a, this.low, mid);
			ArraySumerDc right = new ArraySumerDc(this.a, mid + 1, this.high);
			left.start();
			right.start();
			try {
				left.join();
				right.join();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			this.result = left.result + right.result;
		}
	}
	
	public static int sum(int[] arr) throws InterruptedException {
		ArraySumerDc asdc = new ArraySumerDc(arr, 0, arr.length-1);
		asdc.start();
		asdc.join();
		return asdc.result;
	}
	public static void main(String[] args) throws InterruptedException {
		// size = 400000, 树的高度为 log400000 = 18, 线程个数即这棵平衡二叉树的节点数目, 2^19 = 524288... 
		// 显然, 这么创建线程来解决这个求和问题是为了并行而并行.JVM的线程个数是有限制的. 即使调整为 40000, log40000 = 15, 2^16 = 65536都是线程数目比数组规模还大.
		// size = 4000, log4000 = 12. 2^13 = 8096 个线程, 这个时候 JVM 没有发生 outofmemory error.
		// 很简单, 因为cut的条件是每一个线程处理一个数, 即每个叶子处理一个数, 而中间的非叶子节点都在等待左右孩子做完后在相加.
		int size = 4000;
		int[] a = new int[size];
		for (int i = 0; i < size; i++) {
			a[i] = i;
		}
		int sum = 0;
		long start, end;
		start = System.currentTimeMillis();
		sum = ArraySumerDc.sum(a);
		end = System.currentTimeMillis();
		System.out.println(sum);
	}
}
```