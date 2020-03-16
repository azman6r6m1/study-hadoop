##############################
#      Install Hive       #
##############################

Prerequisites: hadoop

1.  Login as root

2.  Install PostgreSQL
    dnf -y module disable postgresql
    dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    dnf updateinfo
    dnf install postgresql12-server
    /usr/pgsql-12/bin/postgresql-12-setup initdb
    systemctl enable postgresql-12
    sed -i "s/#listen_addresses = 'localhost'/listen_addresses = 'namenode01'/" /var/lib/pgsql/12/data/postgresql.conf
    sed -i 's/#standard_conforming_strings = on/standard_conforming_strings = off/' /var/lib/pgsql/12/data/postgresql.conf
    echo '' >> /var/lib/pgsql/12/data/pg_hba.conf
    echo 'host all all namenode01 md5' >> /var/lib/pgsql/12/data/pg_hba.conf
    systemctl restart postgresql-12
    systemctl status postgresql-12
    
2.  Create hive user and database in PostgreSQL
    su - postgres
    psql -c "create user hive with password 'hive' login nosuperuser nocreatedb noinherit;"
    psql -c "create database hivemetastore with owner = hive;"
    exit

3.  Test connection to PostgreSQL
    psql -h namenode01 -p 5432 -U hive -W -d hivemetastore -c 'select current_timestamp;'
    
3.  Create user hive with group hdpadm
    useradd -m -s /bin/bash hive
    usermod -g hdpadm hive
    passwd hive
    
4.  Create directories
    mkdir -p /app/hive
    chown hive:hdpadm /app/hive

7.  Add firewall exception
    firewall-cmd --zone=public --permanent --add-port=5432/tcp
    firewall-cmd --zone=public --permanent --add-port=9083/tcp
    firewall-cmd --zone=public --permanent --add-port=10000/tcp
    firewall-cmd --reload
    firewall-cmd --list-ports

5.  Create Hive path in HDFS
    su - hadoop
    hdfs dfs -mkdir -p /env/dev/hive/warehouse
    hdfs dfs -chown -R hive /env/dev/hive
    hdfs dfs -chgrp -R hdpadm /env/dev/hive
    hdfs dfs -chmod -R 750 /env/dev/hive
    exit
    
6.  Set hive as Hadoop proxy user
    su - hadoop
    vi /app/hadoop/default/etc/hadoop/core-site.xml
      <property>
        <name>hadoop.proxyuser.hive.hosts</name>
        <value>*</value>
      </property>
      <property>
        <name>hadoop.proxyuser.hive.groups</name>
        <value>*</value>
      </property>
    exit
    
6.  Switch user to hive
    su - hive
    
7.  Download and setup Hive
    cd /app/hive
    wget http://mirror.apache-kr.org/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
    tar -xz --no-same-owner -f apache-hive-3.1.2-bin.tar.gz
    ln -s /app/hive/apache-hive-3.1.2-bin default
    
8.  Download PostgreSQL client library
    cd lib
    wget https://repo1.maven.org/maven2/org/postgresql/postgresql/42.2.11/postgresql-42.2.11.jar
    
8.  Copy Guava library from hadoop installation
    mv guava-19.0.jar guava-19.0.jar.bak
    cp -p /app/hadoop/default/share/hadoop/common/lib/guava-27.0-jre.jar .

8.  Set environment variables
    echo '' >> ~/.bashrc
    echo 'export HADOOP_HOME=/app/hadoop/default' >> ~/.bashrc
    echo 'export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"' >> ~/.bashrc
    echo 'export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH' >> ~/.bashrc
    echo '' >> ~/.bashrc
    echo 'export HBASE_HOME=/app/hbase/default' >> ~/.bashrc
    echo 'export PATH=$HBASE_HOME/bin:$PATH' >> ~/.bashrc
    echo '' >> ~/.bashrc
    echo 'export HIVE_HOME=/app/hive/default' >> ~/.bashrc
    echo 'export PATH=$HIVE_HOME/bin:$PATH' >> ~/.bashrc

9.  Set configuration
    vi /app/hive/default/conf/hive-site.xml
    
10. Initialize schema
    schematool -dbType postgres -initSchema
    schematool -dbType postgres -info
    
11. Start metastore
    mkdir -p /app/hive/default/log
    mkdir -p /app/hive/default/pid
    filetimestamp=$(date +%Y%m%d%H%M%S) && nohup hive --service metastore 1> /app/hive/default/log/metastore.$filetimestamp.out 2> /app/hive/default/log/metastore.$filetimestamp.err &
    echo $! > /app/hive/default/pid/metastore.pid
    
12. Start hive server
    filetimestamp=$(date +%Y%m%d%H%M%S) && nohup hive --service hiveserver2 1> /app/hive/default/log/hiveserver2.$filetimestamp.out 2> /app/hive/default/log/hiveserver2.$filetimestamp.err &
    echo $! > /app/hive/default/pid/hiveserver2.pid