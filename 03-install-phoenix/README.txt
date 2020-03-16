##############################
#      Install Phoenix       #
##############################

Prerequisites: hbase

1.  Login as hbase

2.  Download and unpack phoenix server
    cd /app/hbase
    wget http://mirror.apache-kr.org/phoenix/apache-phoenix-5.0.0-HBase-2.0/bin/apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz
    tar -xz --no-same-owner -f apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz
    cp apache-phoenix-5.0.0-HBase-2.0-bin/phoenix-5.0.0-HBase-2.0-server.jar default/lib
    scp apache-phoenix-5.0.0-HBase-2.0-bin/phoenix-5.0.0-HBase-2.0-server.jar hbase@datanode01:/app/hbase/default/lib

3.  Restart Hbase
    stop-hbase.sh
    start-hbase.sh
    local-master-backup.sh start 1
    local-regionservers.sh start 1