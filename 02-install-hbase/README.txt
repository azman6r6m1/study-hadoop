##############################
#       Install Hive         #
##############################
1.  Login as root
2.  Create user hbase with group hdpadm
3.  Download hbase binary from the following URL to /app
    https://downloads.apache.org/hbase/stable/
4.  Unpack and rename /app/hbase-x.x.x to /app/hbase
5.  Change owner and group of /app/hbase to hbase and hdpadm
6.  Create folder /data/hbase and set owner and group to hbase and hdpadm
7.  Create folder /data/hbase/zookeeper and set owner and group to hbase and hdpadm
8.  Switch user to hbase
9.  Edit /home/hbase/.bashrc
    export HADOOP_HOME=/app/hadoop
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
    export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
    export HBASE_HOME=/app/hbase
    export PATH=$HBASE_HOME/bin:$PATH
10. Edit /app/hbase/conf/hbase-site.xml
11. source /home/hbase/.bashrc
12. start-hbase.sh
13. Test CLI
    "hbase shell"