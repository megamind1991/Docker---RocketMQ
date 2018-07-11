# Docker---RocketMQ

## 参考地址 https://www.jianshu.com/p/139a52a90d61

### docker-compose.yml
    version: '2'
    services:
      namesrv:
        image: rocketmq-namesrv:4.2.0
        ports:
          - 9876:9876
        volumes:
          - "/rocketmq/namesrv/master/logs:/opt/logs"
          - "/rocketmq/namesrv/master/store:/opt/store"
      broker:
        image: rocketmq-broker:4.2.0
        ports:
          - 10909:10909
          - 10911:10911
        volumes:
          - "/rocketmq/broker/a-m/logs:/opt/logs"
          - "/rocketmq/broker/a-m/stor/opt/store"
          - "/rocketmq/broker/a-m/conf:/opt/conf"
        links:
          - namesrv:namesrv
      console:
        image: styletang/rocketmq-console-ng:latest
        ports:
         - "8088:8080"
        links:
         - namesrv:namesrv
        environment:
         JAVA_OPTS: -Drocketmq.config.namesrvAddr=namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false
         
### nameserver镜像生成脚本
       
    # Start from a Java image.
    FROM java:8
    ARG version
    # Rocketmq version
    ENV ROCKETMQ_VERSION 4.2.0
    # Rocketmq home
    ENV ROCKETMQ_HOME  /opt/rocketmq-${ROCKETMQ_VERSION}
    ENV JAVA_OPT_EXT -server -Xms256m -Xmx256m -Xmn128m

    WORKDIR  ${ROCKETMQ_HOME}
    RUN mkdir -p \
            /opt/logs \
            /opt/store
    RUN curl https://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip -o rocketmq.zip \
              && unzip rocketmq.zip \
              && rm rocketmq.zip

    RUN chmod +x bin/mqnamesrv
    CMD cd ${ROCKETMQ_HOME}/bin && export JAVA_OPT=" -Duser.home=/opt" && sh mqnamesrv
    EXPOSE 9876
    VOLUME /opt/logs \
            /opt/store

### broker镜像生成脚本

    FROM java:8
    # Rocketmq version
    ENV ROCKETMQ_VERSION 4.2.0
    # Rocketmq home
    ENV ROCKETMQ_HOME  /opt/rocketmq-${ROCKETMQ_VERSION}
    ENV JAVA_OPT_EXT -server -Xms128m -Xmx128m -Xmn128m

    WORKDIR  ${ROCKETMQ_HOME}
    RUN mkdir -p \
            /opt/logs \
            /opt/store
    RUN curl https://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/${ROCKETMQ_VERSION}/rocketmq-all-${ROCKETMQ_VERSION}-bin-release.zip -o rocketmq.zip \
                  && unzip rocketmq.zip \
                  && rm rocketmq.zip
    RUN chmod +x bin/mqbroker
    CMD cd ${ROCKETMQ_HOME}/bin && export JAVA_OPT=" -Duser.home=/opt" && sh mqbroker -n namesrv:9876
    EXPOSE 10909 10911
    VOLUME /opt/logs \
            /opt/store
       
       
