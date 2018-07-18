## Rocketmq Cluster


# broker镜像文件重构

    # Start from a Java image.
    FROM java:8

    # Rocketmq version
    ENV ROCKETMQ_VERSION 4.2.0

    # Rocketmq home
    ENV ROCKETMQ_HOME  /opt/rocketmq-${ROCKETMQ_VERSION}
    ENV JAVA_OPT_EXT -server -Xms1024m -Xmx1024m -Xmn512m    

    WORKDIR  ${ROCKETMQ_HOME}

    RUN mkdir -p \
            /opt/logs \
            /opt/store

    RUN curl https://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip -o rocketmq.zip \
             && unzip rocketmq.zip \
             && rm rocketmq.zip

    RUN chmod +x bin/mqbroker

    CMD cd ${ROCKETMQ_HOME}/bin && export JAVA_OPT=" -Duser.home=/opt" && sh mqbroker -n namesrv:9876 -c /opt/conf/broker.properties 

    EXPOSE 10909 10911
    VOLUME /opt/logs \
            /opt/store \
            /opt/conf
            
            
# Compose.yml

    version: '2'
    services:
      namesrv:
        imag going/rocketmq-namesrv:4.2.0
        ports:
          - 9876:9876
        volumes:
          - "/rocketmq/namesrv/master/logs:/opt/logs"
          - "/rocketmq/namesrv/master/stor/opt/store"
      broker-a-m:
        imag going/rocketmq-broker:4.2.1
        ports:
          - 11909:10909
          - 11911:10911
        volumes:
          - "/opt/logs/rocketmqlogs:/opt/logs"
          - "/rocketmq/broker/a-s/stor/opt/store"
          - "/opt/rocketmq-4.2.0/conf/2m-2s-sync:/opt/conf"
        links:
          - namesrv:namesrv
        command: cd ${ROCKETMQ_HOME}/bin && export JAVA_OPT=" -Duser.home=/opt" && sh mqbroker -n namesrv:9876 -c /opt/conf/broker-a.properties 
      broker-a-s:
        imag going/rocketmq-broker:4.2.1
        ports:
          - 11909:10909
          - 11911:10911
        volumes:
          - "/opt/logs/rocketmqlogs:/opt/logs"
          - "/rocketmq/broker/a-s/stor/opt/store"
          - "/opt/rocketmq-4.2.0/conf/2m-2s-sync:/opt/conf"
        links:
          - namesrv:namesrv    
        command: cd ${ROCKETMQ_HOME}/bin && export JAVA_OPT=" -Duser.home=/opt" && sh mqbroker -n namesrv:9876 -c /opt/conf/broker-a-s.properties 
      consol
        imag styletang/rocketmq-console-ng:latest
        ports:
         - "8088:8088"
        links:
         - namesrv:namesrv
        environment:
         JAVA_OPTS: -Drocketmq.config.namesrvAddr=namesrv:9876
         
# 配置文件挂载路径 日志挂在路径
    /opt/rocketmq-4.2.0/conf/2m-2s-sync
    
    broker-a-s.properties  broker-b-s.properties
    broker-a.properties    broker-b.properties
    
    /opt/logs/rocketmqlogs
        
