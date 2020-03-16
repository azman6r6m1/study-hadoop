##############################
#       Install HBase        #
##############################

Prerequisites: hadoop

1.  Login as root

2.  Create user hbase with group hdpadm
    useradd -m -s /bin/bash hbase
    usermod -g hdpadm hbase
    passwd hbase

3.  Create directories
    mkdir -p /app/hbase
    chown hbase:hdpadm /app/hbase
    mkdir -p /data/hbase/zookeeper
    chown -R hbase:hdpadm /data/hbase

4.  Add firewall exception
    firewall-cmd --zone=public --permanent --add-port=16010/tcp
    firewall-cmd --zone=public --permanent --add-port=16011/tcp
    firewall-cmd --zone=public --permanent --add-port=16030/tcp
    firewall-cmd --zone=public --permanent --add-port=16031/tcp
    firewall-cmd --reload
    firewall-cmd --list-ports

5.  Create HBase path in HDFS
    su - hadoop
    hdfs dfs -mkdir -p /env/dev/hbase
    hdfs dfs -chown hbase /env/dev/hbase
    hdfs dfs -chgrp hdpadm /env/dev/hbase
    hdfs dfs -chmod 750 /env/dev/hbase
    exit

6.  Switch user to hbase
    su - hbase

7.  Download and setup HBase
    cd /app/hbase
    wget http://mirror.apache-kr.org/hbase/stable/hbase-2.2.3-bin.tar.gz
    tar -xz --no-same-owner -f hbase-2.2.3-bin.tar.gz
    ln -s /app/hbase/hbase-2.2.3 default

8.  Repeat step 1-5 in datanode01

9.  Generate SSH keys
    mkdir ~/.ssh
    cd .ssh
    ssh-keygen -t rsa -b 4096 -C "hbase@namenode01" -f "hbase@namenode01" -N ''
    cat hbase@namenode01.pub >> authorized_keys
    echo '' >> ~/.ssh/config
    echo 'Host localhost' >> ~/.ssh/config
    echo '  IdentityFile ~/.ssh/hbase@namenode01' >> ~/.ssh/config
    echo '' >> ~/.ssh/config
    echo 'Host namenode01' >> ~/.ssh/config
    echo '  IdentityFile ~/.ssh/hbase@namenode01' >> ~/.ssh/config
    echo '' >> ~/.ssh/config
    echo 'Host datanode01' >> ~/.ssh/config
    echo '  IdentityFile ~/.ssh/hbase@namenode01' >> ~/.ssh/config
    chmod 700 .
    chmod 600 *

10. Set environment variables
    echo '' >> ~/.bashrc
    echo 'export HADOOP_HOME=/app/hadoop/default' >> ~/.bashrc
    echo 'export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"' >> ~/.bashrc
    echo 'export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH' >> ~/.bashrc
    echo '' >> ~/.bashrc
    echo 'export HBASE_HOME=/app/hbase/default' >> ~/.bashrc
    echo 'export PATH=$HBASE_HOME/bin:$PATH' >> ~/.bashrc

11. Set configuration
    vi /app/hbase/default/conf/hbase-site.xml
    vi /app/hbase/default/conf/regionservers

12. Copy to namenodes
    scp -r ~/.ssh hbase@datanode01:/home/hbase
    scp ~/.bashrc hbase@datanode01:/home/hbase
    scp /app/hbase/default/conf/hbase-site.xml hbase@datanode01:/app/hbase/default/conf/
    scp /app/hbase/default/conf/regionservers hbase@datanode01:/app/hbase/default/conf/

13. source ~/.bashrc

14. Run
    start-hbase.sh

15. Test CLI
    hbase shell
    status 'simple'
    exit

16. Start backup master
    local-master-backup.sh start 1

17. Start local regionserver
    local-regionservers.sh start 1

18. Go to http://namenode01:16010 for master console

19. Go to http://namenode01:16011 for backup master console

20. Go to http://namenode01:16030 for all regionserver console

21. Go to http://namenode01:16031 for all regionserver console