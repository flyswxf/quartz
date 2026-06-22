1. 先下载Maven[Download Apache Maven – Maven](https://maven.apache.org/download.cgi)
	- 任选一个版本,建议使用zip版本
	- 下载完解压
2. 配置环境变量
	- `Maven_Home`: 设置为maven文件夹位置
	- Path添加一个`%Maven_Home\bin`
3. 打开cmd,输入`mvn -v`查看是否配置成功
	输出结果应类似
```
Apache Maven 3.9.11 (3e54c93a704957b63ee3494413a2b544fd3d825b)
Maven home: /opt/apache-maven-3.9.11
Java version: 1.8.0_45, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.5", arch: "x86_64", family: "mac"
```