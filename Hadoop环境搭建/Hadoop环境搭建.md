1.安装centos 7(先暂时安装一台master,后面两台采取clone模式)  
2.安装成功后 执行 yum install net-tools:安装网络工具 输入两个yes（可以先添加镜像源，更新库）
3.使用ifconfig命令查询虚拟机ipv4地址(查询不出进行附件1) ping www.baidu.com
4.克隆master虚拟机  查看ifconfig ipv4地址 如果出现 127.0.0.1 重复附件1操作 将ip地址最后一位改掉

-------以下步骤可使用xshell连接后执行----------> 

（如果安装ssh服务后再克隆，会克隆出一模一样的网卡，导致xshell无法连接等问题）

5.安装ssh服务(all)   yum -y install openssh-server
                    yum -y install openssh-clients

（Secure Shell（缩写：*SSH*），即“安全壳协议”，建立安全的网络连接需要）

6.修改主机名 vim /etc/hostname  将其修改成 master slave01 slave02 重启输入hostname生效(reboot重启)
7.关闭防火墙(all)、禁止开启自启(centos 7 采用下面方式关闭,7以下见附件2)
    centos7开始默认用的是firewall 7
  	1、关闭防火墙  systemctl stop firewalld.service
      2、禁止firewall开机启动  systemctl disable firewalld.service

8.配置 /etc/hosts文件，追加以下内容(all) :（注意写自己的ip)

	192.168.230.131 master
	192.168.230.129 slave01
	192.168.230.130 slave02

9.生成公钥和私钥(all)并设置无密码登录
    1.生成公钥和私钥(all)   ssh-keygen -t rsa  一路回车即可
    2.将公钥复制到authorized_keys并赋予authorized_keys600权限(all) 
        cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	    chmod 600 ~/.ssh/authorized_keys
    3.将master节点上的authoized_keys远程传输到slave1和slave2的~/.ssh/目录下
        scp /root/.ssh/authorized_keys root@slave02:/root/.ssh/
    4.验证无密码登录 ： ssh slave01 不需要输入密码即可登录 exit:退出

10.安装jdk和hadoop  使用 filezilla（或其他软件如xftp） 进行传输（在master主机上）
   1.tar zxvf jdkxxx 解压jdk(在opt目录下新建SoftWare文件夹,将其解压到该目录)
   2.配置jdk环境变量: vim /etc/profile 在末尾追加：（最后两行是Hadoop的环境变量，Linux的环境变量都/etc/profile文件中）
    	export JAVA_HOME=/opt/SoftWare/jdk-12.0.1
		export JRE_HOME=/opt/SoftWare/jdk-12.0.1/jre
		CLASSPATH=.:$JRE_HOME/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
		PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
		export HADOOP_HOME=/opt/SoftWare/hadoop-2.7.7
		export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

   3.source /etc/profile：刷新环境变量
     验证是否安装成功: java -version
   4.上传并解压hadoop
      1.tar axvf hadoopxxx(解压到SoftWare文件夹下)
      2.解压后在hadoop文件夹下创建tmp、logs、hdfs、hdfs/name、hdfs/data五个文件夹
      3.使用ftp软件进行配置文件的更改:都在hadoop/etc/hadoop下
      		 1.hadoop-env.sh 
        	 修改第25行的 ${JAVA_HOME}为自己的jdk安装目录(/opt/SoftWare/jdkxxx)
       		2.yarn-env.sh 
         	修改第23行, 解注释(export JAVA_HOME=/home/y/libexec/jdk1.6.0/)
        	 修改为自己的jdk安装目录(/opt/SoftWare/jdkxxx)
       		3.slaves  修改localhost 为slave01和slave02（回车换行或空格）
       4.在master命令行中执行  重命名操作
       	(该文件在opt/SoftWare/hadoop-2.7.7/etc/hadoop/下)
       	mv mapred-site.xml.template mapred-site.xml
       5.配置 core-site.xml文件 (可通过ftp直接覆盖，以下同)

 	  6.配置hdfs-site.xml文件

​	   7.配置yarn-site.xml

​       8.配置mapred-site.xml

​       9.将SoftWare文件夹拷贝到slave01和slave02的opt目录下
​		  scp -r /opt/SoftWare/ root@slave01:/opt/
​		  scp -r /opt/SoftWare/ root@slave02:/opt/

​       10.将/etc/profile文件拷贝到slave01heslave02的/etc目录下
​			scp /etc/profile root@slave01:/etc/
​			scp /etc/profile root@slave01:/etc/
   	 11.刷新slave01和slave02中的环境变量 验证java是否配置成功
 			  source /etc/profile   java -version
​		12.锁定时间同步(all)
​    		安装时间同步软件 ntpdate yum -y install ntp ntpdate
​			和网络时间进行同步 ntpdate cn.pool.ntp.org
​			把时间写入硬件进行锁定: hwclock --systohc
​		13.进入hadoop-2.7.7/bin目录 
​			使用 ./hdfs namenode -format进行格式化
​			出现successfully formatted 表示成功

​			不允许多次格式化,会导致集群无法启动
​			 如果出错: 1.检查并修改配置文件
​		           2.删除三个主机上的hdfs中的name和data文件夹
​	  	         3.重新修改 并发送至slave01和slave02
​		           4.重新格式化
​		14.进入 hadoop-2.7.7/sbin目录 启动hdfs服务
​	    		./start-dfs.sh
​		15.分别执行jps命令 出现以下结果 代表配置成功
​			    master: NameNode SecondaryNameNode

​	  		  访问:http://192.168.147.129:50070（注意修改为自己的master ip地址，端口号不变）
​		16.在sbin目录中启动yarn服务 ./start-yarn.sh
​				分别执行jps命令 ResourceManager  NodeManager

​				访问http://192.168.230.131:8088（同上）

​		17.创建目录：hadoop fs -mkdir -p /test/input（测试是否成功）
​			查看创建的目录： hadoop fs -ls /
​		上传 hadoop fs -put ./111.txt /test/input
​	    	 hdfs dfs -put ./111.txt /test
​		下载文件 
​		hadoop fs -get /test/input/111.txt /home/hadoop/       		   

------------------------------------------------------------->

附件1.配置ifcfg-ens33文件 vi /etc/sysconfig/network-scripts/ifcfg-ens33
      修改 BOOTPROTO="static" | dhcp
      修改后执行 service network restart  重启网络服务

附件2. 7以下版本采用如下方式:
	  关闭防火墙(all)  service iptables stop
      分别禁止防火墙开机启动  chkconfig iptables off

------------------------------------------------------------->
		ifcfg-ens33文件 key值解释
		ONBOOT=yes：网卡设备自动启动
		BOOTPROTO=static 设置静态ip
		GATWAY ：默认网关
		IPADDR：ip地址
		NETMASK：子网掩码
		DNS：dns服务器