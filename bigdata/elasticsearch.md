### 学习网站
*  ElasticSearch 6.x 学习笔记：4.IK分词器插件 https://blog.csdn.net/chengyuqiang/article/details/78991570
* Elasticsearch 的 NGram 分词器处理模糊匹配 https://www.imooc.com/article/18578?block_id=tuijian_wz
* https://es.xiaoleilu.com/040_Distributed_CRUD/15_Create_index_delete.html

* elasticsearch install
* https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-targz.html

#### 安装elasticsearch
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.zip
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.zip.sha512
shasum -a 512 -c elasticsearch-6.4.0.zip.sha512 
unzip elasticsearch-6.4.0.zip
cd elasticsearch-6.4.0/ 
```
#### 配置
* elasticsearch-6.4.0/config/elasticsearch.yml
```
#network.host: 192.168.0.1
network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
#http.port: 9200
http.port: 9201
#
# For more information, consult the network module documentation.
```
#### 启动
```
cd elasticsearch-6.4.0/bin
./elasticsearch -d
```

#### .bashrc 环境变量
```
[charmi@dev ~]$ cat .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
export JAVA_HOME=/usr/local/jdk1.8.0_181
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:$PATH 
export ES_HOME=/home/charmi/elasticsearch-6.4.0
[charmi@dev ~]$ 

```

### 安装 elasticsearch-analysis-ik 中文分词
直接下载，展开在your-es-root/plugins/ik这个目录下
* https://github.com/medcl/elasticsearch-analysis-ik/
```
1.download or compile
optional 1 - download pre-build package from here: https://github.com/medcl/elasticsearch-analysis-ik/releases
create plugin folder cd your-es-root/plugins/ && mkdir ik
unzip plugin to folder your-es-root/plugins/ik
```

### 问题处理
```
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```
```
#ftp             hard    nproc           0
#@student        -       maxlogins       4
naxxm hard nofile 65536
naxxm soft nofile 65536
* soft nofile 65536
* hard nofile 65536
# End of file
"/etc/security/limits.conf" 64L, 2509C written
[root@dev ~]#
```

### 案例
```json
curl -XPUT http://localhost:9201/index

curl -XPOST http://localhost:9201/index/fulltext/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            }
        }

}'

curl -XPOST http://localhost:9201/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"id":1,"content":"美国留给伊拉克的是个烂摊子吗"}
'
curl -XPOST http://localhost:9201/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"id":2,"content":"美国留给伊拉克的是个烂摊子吗"}
'
curl -XPOST http://localhost:9201/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"id":3,"content":"美国留给伊拉克的是个烂摊子吗"}
'
curl -XPOST http://localhost:9201/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"id":7,"content":"因为我们只需要按照词典分词，所以这边只有一种最大分词模式，test_max_word。接下来就是Analyzer 和Tokenizor。默认"}
'

curl -XPOST http://localhost:9201/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"id":5,"content":"默认是custom/mydict.dic;custom/single_word_low_freq.dic，   我这里改为我自己的了。   （自定义热更新词库）   custom/mydict.dic;custom/single_word_low_freq.dic;custom/zhouls.dic"}
'

curl -XPOST http://localhost:9201/index/fulltext/_search  -H 'Content-Type:application/json' -d'
{
    "query" : { "match" : { "content" : "默认" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}
'
```
### 案例
```go


curl -XPOST http://localhost:9201/index/fulltext/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            }
        }

}'

curl -XPOST http://localhost:9201/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}
'
curl -XPOST http://localhost:9201/index/fulltext/2 -H 'Content-Type:application/json' -d'
{"content":"公安部：各地校车将享最高路权"}
'
curl -XPOST http://localhost:9201/index/fulltext/3 -H 'Content-Type:application/json' -d'
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
'
curl -XPOST http://localhost:9201/index/fulltext/4 -H 'Content-Type:application/json' -d'
{"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
'

curl -XPOST http://localhost:9201/index/fulltext/_search  -H 'Content-Type:application/json' -d'
{
    "query" : { "match" : { "content" : "美国个烂摊子自首" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}
'
curl -XPOST http://localhost:9201/index/fulltext/5 -H 'Content-Type:application/json' -d'
{"content":"留给伊拉克的是个烂摊子吗，留给伊拉克的是个烂摊子吗"}
'




```
