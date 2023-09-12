

## Docker-compose安装

    version: '3.3'
    services:
      elasticsearch:
        image: elasticsearch:6.4.3
        container_name: elasticsearch
        restart: always
        ports:
          - 9200:9200
          - 9300:9300
        environment:
          cluster.name: elasticsearch
        volumes:
          - /data/docker/elasticsearch/data:/usr/share/elasticsearch/data
          - /data/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins

**集成IK分词器**

    wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.3/elasticsearch-analysis-ik-6.4.3.zip

解压：

    unzip elasticsearch-analysis-ik-6.4.3 -d  analysis-ik

拷贝到插件目录

    cp -r analysis-ik plugins

**集成拼音分词器**

    wget https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.4.3/elasticsearch-analysis-pinyin-6.4.3.zip

解压：

    unzip elasticsearch-analysis-pinyin-6.4.3 -d  analysis-pinyin

拷贝到插件目录

    cp -r analysis-pinyin plugins
