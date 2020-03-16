##############################
#      Install Hadoop        #
##############################
1.  Login as root
2.  Create group hdpadm
3.  Create user hadoop with group hdpadm
4.  Create folder /app/java
6.  Download JDK from the following URL to /app/java
    https://www.azul.com/downloads/zulu-community/?version=java-8-lts&os=centos&architecture=x86-64-bit&package=jdk
7.  Unpack and rename /app/java/zulu8.xx.x.x-ca-jdk8.0.xxx-linux_x64 to /app/java/zulu-jdk8
8.  Create symbolic link /app/java/jdk to /app/java/zulu-jdk8
9.  Edit /etc/bashrc
    export JAVA_HOME=/app/java/jdk
    export PATH=$JAVA_HOME/bin:$PATH
10. Download hadoop binary from the following URL to /app
    https://downloads.apache.org/hadoop/common/stable/
11. Unpack and rename /app/apache-hadoop-x.x.x to /app/hadoop
12. Change owner and group of /app/hadoop to hadoop and hdpadm
13. Create folder /data/hadoop/namenode and set owner and group to hadoop and hdpadm
14. Create folder /data/hadoop/datanode and set owner and group to hadoop and hdpadm
15. Edit /etc/hosts
    namenode01 x.x.x.x
    datanode01 x.x.x.x
    servnode01 x.x.x.x
16. Repeat step 1-15 in datanode01
17. Switch user to hadoop
18. Edit /home/hadoop/.bashrc
    export HADOOP_HOME=/app/hadoop
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
    export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
19. Edit /app/hadoop/etc/hadoop/core-site.xml. Refer core-site.xml.
20. Edit /app/hadoop/etc/hadoop/hdfs-site.xml. Refer hdfs-site.xml.
21. Edit /app/hadoop/etc/hadoop/mapred-site.xml. Refer mapred-site.xml.
22. Edit /app/hadoop/etc/hadoop/yarn-site.xml. Refer yarn-site.xml.
23. Edit /app/hadoop/etc/hadoop/workers
    namenode01
    datanode01
24. Copy the following files from namenode01 to datanode01
    /etc/bashrc
    /home/hadoop/.bashrc
    /app/hadoop/etc/hadoop/core-site.xml
    /app/hadoop/etc/hadoop/hdfs-site.xml
    /app/hadoop/etc/hadoop/mapred-site.xml
    /app/hadoop/etc/hadoop/yarn-site.xml
    /app/hadoop/etc/hadoop/workers
25. source /home/hadoop/.bashrc
26. start-all.sh