eclipse 配置hadoop环境:

1.将Linux master配置好的hadoop-2.7.7下载到本地(建议先压缩再下载) 
  将插件中的bin目录的所有文件放入hadoop-2.7.7的bin目录中(插件在\BigData\配置文件\hadoop-eclipse-plugin\目录下)
2.将bin目录下的hadoop.dll文件放到C:\Windows\System32目录下
    配置环境变量 HADOOP_USER_NAME：root（值为root)

​	配置环境变量HADOOP_HOME （值为到bin目录前）

​	配置hadoop的 \bin \sbin 目录到环境变量

  修改本机的hosts C:\Windows\System32\drivers\etc\hosts
    192.168.30.1     localhost
	192.168.147.129  master
	192.168.147.130  slave01
	192.168.147.131  slave02
3.将hadoop-eclipse-plugin-2.7.3.jar放入eclipse的dropins目录下(\BigData\配置文件\hadoop-eclipse-plugin)
4.重启eclipse 点击window-->Preferences 出现Hadoop Map/Reduce选项
5.点击Browse 将hadoop路径配置进去
6.点击右上角 Perspective 出现小蓝象图标,点击open 
7.eclipse下方出现Map/Reduce locations选项卡,点击右上角小蓝象 新建一个Hadoop locations
8.在新窗口填入一下信息： 之后 点击finish
	location name：任意 eg：Lichtr
	Host:master ip地址 eg：192.168.147.129
	Port：9001
	右面Port：9000
	User name：root
9.左边出现DFS Locations 选项,点击新建的小蓝象。出现文件夹,文件夹中即为HDFS内容。


wordcount 代码步骤：

1.设置conf对象，并配置hdfs路径
2.设置job对象，在job对象中设置 Jar map reduce reduce的输出类型(key value) 输出格式化方式
   文件的输入和输出路径  系统退出

3.pom.xml添加依赖
	<dependency>
	    <groupId>org.slf4j</groupId>
	    <artifactId>slf4j-api</artifactId>
	    <version>1.7.26</version>
	</dependency>

	<dependency>
	    <groupId>org.slf4j</groupId>
	    <artifactId>slf4j-simple</artifactId>
	    <version>1.6.4</version>
	 </dependency>
4.在resource目录下添加log4j.properties文件





问题一：Permission denied
解决：https://blog.csdn.net/wang7807564/article/details/74627138
问题二：HADOOP_HOME or hadoop.home.dir are not set.
解决：配置HADOOP_HOME环境变量 并在conf对象前加如下代码：
System.setProperty("hadoop.home.dir", "D:\\teach\\Java\\eclipse\\hadoop-2.7.7"); 
问题三：Could not locate executable D:\teach\Java\eclipse\hadoop-2.7.7\bin\winutils.exe in the Hadoop binaries.
解决：
 https://blog.csdn.net/congcong68/article/details/42043093
 https://github.com/4ttty/winutils  相应版本的winutils.exe到hadoop\bin  地址

问题四：ExitCodeException exitCode=-1073741515:
解决：https://blog.csdn.net/u013303361/article/details/88853684


删除Hadoop的文件 hadoop fs -rmr /home/mm/lily2(要求是你把Hadoop的bin加到PATH中,并开启Hadoop)



 ----------------------------------->
创建maven工程
1.解压maven压缩包
2.进入eclipse 点击window-->Preferences-->Maven-->Installations-->add-->将maven目录添加进去。
3.点击window-->Preferences-->Maven-->User Settings-->Browse--将settings.xml导入 更改maven的下载网址
<mirrors>
	<mirror>
		<id>alimaven</id>
		<name>aliyun maven</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		<mirrorOf>central</mirrorOf> 
	</mirror>
</mirrors>
4.新建 maven project 