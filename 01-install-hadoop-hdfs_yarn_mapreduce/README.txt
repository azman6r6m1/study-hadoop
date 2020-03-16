##############################
#      Install Hadoop        #
##############################

1.  Login as root

2.  Download and setup JDK 8
    mkdir -p /app/java
    cd /app/java
    wget https://cdn.azul.com/zulu/bin/zulu8.44.0.11-ca-jdk8.0.242-linux_x64.tar.gz
    tar xzf zulu8.44.0.11-ca-jdk8.0.242-linux_x64.tar.gz
    ln -s /app/java/zulu8.44.0.11-ca-jdk8.0.242-linux_x64 default
    echo '' >> /etc/bashrc
    echo 'export JAVA_HOME=/app/java/default' >> /etc/bashrc
    echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /etc/bashrc

3.  Edit /etc/hosts
    x.x.x.x namenode01
    x.x.x.x datanode01
    x.x.x.x servnode01

4.  Create group hdpadm
    groupadd hdpadm

5.  Create user hadoop with group hdpadm
    useradd -m -s /bin/bash hadoop
    usermod -g hdpadm hadoop
    passwd hadoop

6.  Create directories
    mkdir -p /app/hadoop
    chown hadoop:hdpadm /app/hadoop
    mkdir -p /data/hadoop/namenode
    mkdir -p /data/hadoop/datanode
    chown -R hadoop:hdpadm /data/hadoop

7.  Add firewall exception
    firewall-cmd --zone=public --permanent --add-port=9000/tcp
    firewall-cmd --zone=public --permanent --add-port=9870/tcp
    firewall-cmd --zone=public --permanent --add-port=8088/tcp
    firewall-cmd --reload
    firewall-cmd --list-ports

8.  Switch user to hadoop
    su - hadoop

9.  Download and setup Hadoop
    cd /app/hadoop
    wget http://mirror.apache-kr.org/hadoop/common/stable/hadoop-3.2.1.tar.gz
    tar -xz --no-same-owner -f hadoop-3.2.1.tar.gz
    ln -s /app/hadoop/hadoop-3.2.1 default

10. Repeat step 1-9 in datanode01

11. Generate SSH keys
    mkdir ~/.ssh
    cd .ssh
    ssh-keygen -t rsa -b 4096 -C "hadoop@namenode01" -f "hadoop@namenode01" -N ''
    cat hadoop@namenode01.pub >> authorized_keys
    echo '' >> ~/.ssh/config
    echo 'Host namenode01' >> ~/.ssh/config
    echo '  IdentityFile ~/.ssh/hadoop@namenode01' >> ~/.ssh/config
    echo '' >> ~/.ssh/config
    echo 'Host datanode01' >> ~/.ssh/config
    echo '  IdentityFile ~/.ssh/hadoop@namenode01' >> ~/.ssh/config
    chmod 700 .
    chmod 600 *

12.  Set environment variables
    echo '' >> ~/.bashrc
    echo 'export HADOOP_HOME=/app/hadoop/default' >> ~/.bashrc
    echo 'export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"' >> ~/.bashrc
    echo 'export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH' >> ~/.bashrc

13. Set configuration
    vi /app/hadoop/etc/hadoop/core-site.xml
    vi /app/hadoop/etc/hadoop/hdfs-site.xml
    vi /app/hadoop/etc/hadoop/mapred-site.xml
    vi /app/hadoop/etc/hadoop/yarn-site.xml
    vi /app/hadoop/etc/hadoop/workers

14. Copy to datanodes
    scp -r ~/.ssh hadoop@datanode01:/home/hadoop
    scp ~/.bashrc hadoop@datanode01:/home/hadoop
    scp /app/hadoop/default/etc/hadoop/core-site.xml hadoop@datanode01:/app/hadoop/default/etc/hadoop
    scp /app/hadoop/default/etc/hadoop/hdfs-site.xml hadoop@datanode01:/app/hadoop/default/etc/hadoop
    scp /app/hadoop/default/etc/hadoop/mapred-site.xml hadoop@datanode01:/app/hadoop/default/etc/hadoop
    scp /app/hadoop/default/etc/hadoop/yarn-site.xml hadoop@datanode01:/app/hadoop/default/etc/hadoop
    scp /app/hadoop/default/etc/hadoop/workers hadoop@datanode01:/app/hadoop/default/etc/hadoop

15. source ~/.bashrc

16. Format HDFS
    hdfs namenode -format

17. Start HDFS
    start-dfs.sh

18. Create basic path in HDFS
    su - hadoop
    hdfs dfs -mkdir -p /env/dev
    hdfs dfs -chgrp -R hdpadm /env
    hdfs dfs -chmod -R 750 /env
    hdfs dfs -mkdir /tmp
    hdfs dfs -chgrp hdpadm /tmp
    hdfs dfs -chmod 1777 /tmp
    exit

19. Get HDFS report
    hdfs dfsadmin -report

20. Go to http://namenode01:9870 for resource console

21. Start YARN
    start-yarn.sh

22. Get YARN Nodes and Applications
    yarn node -list
    yarn application -list

23. Go to http://namenode01:8088 for yarn console