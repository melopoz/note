## 安装

安装好jdk1.8的linux系统下

1. 创建hadoop用户

   > $ sudo useradd -m hadoop -s /bin/bash  #创建hadoop用户，并使用/bin/bash作为shell
   > $ sudo passwd hadoop                   #为hadoop用户设置密码，之后需要连续输入两次密码
   > $ sudo adduser hadoop sudo             #为hadoop用户增加管理员权限
   > $ su - hadoop                          #切换当前用户为用户hadoop
   > $ sudo apt-get update                  #更新hadoop用户的apt,方便后面的安装

2. 设置ssh和无密码登陆

   > $ sudo apt-get install openssh-server   #安装SSH server
   >
   > #需要版本对应，安装报依赖不合格的话就apt install 对应版本的client即可
   >
   > $ ssh localhost                         #登陆SSH，第一次登陆输入yes
   > $ exit                                  #退出登录的ssh localhost
   > $ cd ~/.ssh/                            #如果没法进入该目录，执行一次ssh localhost
   > $ ssh-keygen -t rsa　#然后的三次输入都确定即可 直接回车
   >
   > $ cat ./id_rsa.pub >> ./authorized_keys #加入授权
   > $ ssh localhost                         #此时已不需密码即可登录localhost，并可见下图。如果失败则可以搜索SSH免密码登录来寻求答案

3. 下载hadoop-xxx.tar.gz解压到/usr/local/hadoop...

   > sudo tar -zxvf hadoop-xxx.tar.gz -C /usr/local
   >
   > cd /usr/local

   （可选）将/usr/local/hadoop-xxx 改名为 /usr/local/hadoop

   > sudo chown -R hadoop ./hadoop-xxx #授权

   如果没有将hadoop-xxx改名为hadoop 或者 将配置文件中的/usr/local/hadoop改为/usr/local/hadoop-xxx

   就需要给这个目录也授权

   > sudo chown -R hadoop ./hadoop #授权  

4. 配置环境变量 (已配置JAVA的环境变量)

   > sudo vim /etc/profile

   > export HADOOP_HOME=/usr/local/hadoop-xxx
   > export CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath):$CLASSPATH
   > export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
   > export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   >
   > #下边这个为了不报错
   >
   > export  HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib:$HADOOP_COMMON_LIB_NATIVE_DIR"

   > source /etc/profile #使配置文件生效

5. hadoop version即可查看版本

## 伪分布式配置

1. 在/usr/local/hadoop-xxx/etc/hadoop目录下

   > vim hadoop-env.sh
   >
   > #如果没有JAVA_HOME，就在底部添加 export JAVA_HOME=${JAVA_HOME} #已经配置好javahome

2. 修改core-site.xml文件

   > <configuration>
   >         <property>
   >              <name>hadoop.tmp.dir</name>
   >              <value>file:/usr/local/hadoop/tmp</value>
   >              <description>Abase for other temporary directories.</description>
   >         </property>
   >         <property>
   >              <name>fs.defaultFS</name>
   >              <value>hdfs://localhost:9000</value>
   >         </property>
   > </configuration>

3. 修改hdfs-site.xml

   > <configuration>
   >         <property>
   >              <name>dfs.replication</name>
   >              <value>1</value>
   >         </property>
   >         <property>
   >              <name>dfs.namenode.name.dir</name>
   >              <value>file:/usr/local/hadoop/tmp/dfs/name</value>
   >         </property>
   >         <property>
   >              <name>dfs.datanode.data.dir</name>
   >              <value>file:/usr/local/hadoop/tmp/dfs/data</value>
   >         </property>
   > </configuration>

4. 格式化NameNode

   > ./bin/hdfs namenode -format

5. 启动

   > /usr/local/hadoop-xxx/sbin/start-dfs.sh

   启动完成可以使用jps查看进程