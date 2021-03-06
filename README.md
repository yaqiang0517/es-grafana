一：ES引入springboot
	1.引入jar
		此jar包含elasticsearch工具类ESUtil，包含两个方法：
		createIndex：创建索引以及类型(index/type)
			参数：CreateIndexRequest
				e.g.{
						"indexName":"monitor_index",//索引名
						"typeName":"monitor_type",//类型名
						"fieldInfo":[//字段
							{
								"field":"productId",//字段名
								"type":"long",//字段类型，参照FieldType里边罗列的类型
								"format":"",//日期类型格式，默认是epoch_millis
								"index":true,//是否索引
								"store":false,//否在 _source 之外在独立存储一份
								"analyer":""//分词器，参照Analyer里边罗列的类型
							},
							{
								"field":"productName",
								"type":"text",
								"format":"",
								"index":true,
								"store":false,
								"analyer":"ik_max_word"
							},
							{
								"field":"stock",
								"type":"integer",
								"format":"",
								"index":true,
								"store":false,
								"analyer":""
							},
							{
								"field":"updateTime",
								"type":"date",
								"format":"epoch_millis",
								"index":true,
								"store":false,
								"analyer":""
							}
						]
					}
			返回值：CommResult
				e.g.{ 
						"status": 0,//0表示成功，-1表示失败
						"message": "创建映射成功"//提示信息
					}
				e.g.{
						"status": -1,
						"message": "已经有同名的索引或者类型"
					}	
		index：索引单条doc数据
			参数：IndexRequest
				e.g.{
						"indexName":"monitor_index",//索引名
						"typeName":"monitor_type",//类型名
						"data": //要索引的数据
							{
								"productId":"1001",//已经定义过的字段名
								"productName":"联想笔记本",//已经定义过的字段名
								"stock":"980",//已经定义过的字段名
								"updateTime":"1546502675000"//已经定义过的字段名[必须定义时间字段，使用默认格式，epoch_millis]
							}
					}
			返回值：CommResult
				e.g.{ 
						"status": 0,//0表示成功，-1表示失败
						"message": "请求成功"//提示信息
					}
					
	2.添加maven依赖
	...
    <properties>
        <elasticsearch.version>5.5.0</elasticsearch.version>
    </properties>
	...
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>transport</artifactId>
            <version>${elasticsearch.version}</version>
        </dependency>
	...
	3.添加es配置文件：target/classes/es.properties[配置文件名和位置不能更改]
		es.number_of_shards:1//数据分片数，默认是1，如果只有一台机器，设置为0
		es.number_of_replicas:5//数据备份数，默认是5
		es.cluster.name=search//es集群名
		es.addr=xx.xx.xx.xx//es地址
		es.port=19100//es端口
	4.调用ESUtil
		创建索引:
			@PostMapping("/demoCreate")
			public CommResult demoCreate(@RequestBody CreateIndexRequest request){
				return ESUtil.createIndex(request);
			}
		索引数据：
			@PostMapping("/demoIndex")
			public CommResult demoIndex(@RequestBody IndexRequest request) {
				return ESUtil.index(request);
			}
二：Eagle配置监控
1.	添加数据源
 
设置-名称：自定义数据源名
HTTP-URL：http://ES ip:port
Elasticsearch详情-索引名称：索引名称
Elasticsearch详情-时间字段名称：type中时间字段的名称
Elasticsearch详情-版本：es的版本
2.	添加仪表盘和面板
e.g.一分钟内商品被购买的次数
 
设置指标：
数据源选择刚才配置的数据源：monitor-es
查询：Lucene 查询语句
变量：给指标起的别名
指标：对指标的聚合函数，此处使用count
分组：选择Date Historgram 并且选择时间字段 
Interval：指标采集时间间隔，如1m
	e.g. 库存
 
这里的指标使用average，即一分钟内库存的平均值

