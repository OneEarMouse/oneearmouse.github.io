---
title: 在SpringBoot中整合ElasticSearch
date: 2023-09-14 23:27:44
categories: ElasticSearch
tags:
description: 简单记录如何在SpringBoot项目中引入ElasticSearch搜索引擎
---

# 依赖及相关配置
1. 在pom.xml中引入依赖
```xml
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    </dependency>
```
2. application.yml配置
```yml
spring:
  elasticsearch:
    rest:
      # 如果是集群，用逗号隔开
      uris: http://ip:port
      username: xxx
      password: yyy
      connection-timeout: 3000
      read-timeout: 1000
```

注：在ES7.1.5版本后，使用RestHighLevelClient已经被标为过时并放弃，因此这里不再介绍。本文主要介绍使用ElasticsearchRestTemplate和使用ElasticsearchRepository的方法

# 创建对应实体类
创建一个实体类，例子如下：
```java
@Data
@Document(indexName = "file_record", shards = 2, replicas = 0)
@Setting(settingPath = "ElasticSearch/settings.json")
public class FileRecordESInfo implements Serializable {

    @Id
    @Field(name = "id", type = FieldType.Keyword)
    private String id;

    @Field(name = "task_uuid", type = FieldType.Keyword)
    private String taskUuid;

    @Field(name = "file_name", type = FieldType.Text, analyzer = "custom_analyzer")
    private String fileName;

    @Field(name = "file_size", type = FieldType.Long, index = false)
    private Long fileSize;

    @Field(name = "file_type", type = FieldType.Integer)
    private Integer fileType;

    @Field(name = "file_create_time", type = FieldType.Date, format = DateFormat.epoch_millis)
    private Date fileCreateTime;

    @Field(name = "tag_list", type = FieldType.Nested)
    private List<Tag> tagList;

    @Data
    public static class Tag {
        @Field(name = "tag_id", type = FieldType.Keyword)
        private String tagId;

        @Field(name = "tag_name", type = FieldType.Keyword)
        private String tagName;

        @Field(name = "tag_type", type = FieldType.Integer, index = false)
        private Integer tagType;

        @Field(name = "tag_set_time", type = FieldType.Date, format = DateFormat.epoch_millis, index = false)
        private Date tagSetTime;
    }
}
```

其中：
- @Document()注解可指定对应index，分片数量等
- @Field()注解可指定字段在es索引中的名称及属性等，并指定是否需要被索引
- 注解中的字段属性、索引等值的选取，需仔细思考后赋值

# 使用ElasticsearchRepository
ElasticsearchRepository接口封装了Document的CRUD操作。对于需要在es中操作的实体类，如果在application.yml中已经配置了```spring.elasticsearch```相关属性，则可以直接通过ElasticsearchRepository实现CRUD操作

```java
public interface ESFileRecordRepository extends ElasticsearchRepository<FileRecordESInfo, String> {
  // add other methods here
}
```

常见的可调用接口如下：
```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S entity);

    <S extends T> Iterable<S> saveAll(Iterable<S> entities);

    Optional<T> findById(ID id);

    boolean existsById(ID id);

    Iterable<T> findAll();

    Iterable<T> findAllById(Iterable<ID> ids);

    long count();

    void deleteById(ID id);

    void delete(T entity);

    void deleteAllById(Iterable<? extends ID> ids);

    void deleteAll(Iterable<? extends T> entities);

    void deleteAll();
}
```

因此，我们可以这样调用。以下是调用批量保存的例子。其他方法的逻辑类似
```java
@Slf4j
@Service
public class ESService {
    @Autowired
    private ESCollectRecordRepository esCollectRecordRepository;

    public void saveRecordESInfoList(List<RecordESInfo> recordESInfoList) {
        esCollectRecordRepository.saveAll(recordESInfoList);
    }
}
```

## 注意：
### 索引初始化问题
如果连接到的ES中不存在对应实体类的索引，那么Spring会在启动时，将连接的所有ES实例中按照所有ElasticsearchRepository创建空索引。在实际生产环境中部署时，需要注意这点！

### 查询问题
该CURD接口无法使用复杂的查询，我们需要另外配置查询相关内容


# 使用ElasticsearchRestTemplate
## 引入
如果在application.yml中已经配置了```spring.elasticsearch```相关属性。则可以在service中直接引入，例如批量：
```java
@Service
public class ESService{
  @Autowired
  private ElasticsearchRestTemplate elasticsearchRestTemplate;
}
```

## 使用
elasticsearchRestTemplate中也对应了许多预定义操作，可以直接通过```org.springframework.data.elasticsearch.core```中的源码查看，这里仅介绍查询相关，其他功能不再赘述。

## 查询
在```org.springframework.data.elasticsearch.core.query```中实现了Query类及其子类。我们可以通过其中的类组装ES的查询参数，再通过```elasticsearchRestTemplate.search```进行es的查询。以下是一个分页查询及不同子类型查询的例子
```java
public PageVo<RecordESInfo> searchRecordESInfo(PageVo<RecordESInfo> pageVo, RecordQueryParam recordQueryParam) {
        // 创建布尔查询构造器，对应ES中布尔查询（各个子句之间的逻辑关系是与的查询）
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        /*
        * 根据recordQueryParam中的查询参数，给布尔查询中添加查询参数。
        * 以下是一些查询的列子
        */
        // matchQuery类型查询，会走倒查索引
        if (StringUtils.isNotBlank(recordQueryParam.getFileNamePattern())) {
            boolQueryBuilder.must(QueryBuilders.matchQuery("file_name", recordQueryParam.getFileNamePattern()));
        }
        // 时间类型查询
        if (recordQueryParam.getStartTime() != null) {
            String startDate = String.valueOf(recordQueryParam.getStartTime().getTime());
            boolQueryBuilder.must(QueryBuilders.rangeQuery("file_create_time").gte(startDate));
        }
        if (recordQueryParam.getEndTime() != null) {
            String endDate = String.valueOf(recordQueryParam.getEndTime().getTime());
            boolQueryBuilder.must(QueryBuilders.rangeQuery("file_create_time").lte(endDate));
        }
        // 完全匹配查询
        if (recordQueryParam.getFileType() != null) {
            boolQueryBuilder.must(QueryBuilders.termQuery("file_type", recordQueryParam.getFileType()));
        }
        // nest类型查询，对应实体类中的子类
        if (StringUtils.isNotBlank(recordQueryParam.getTagId())) {
            boolQueryBuilder.must(QueryBuilders.nestedQuery("tag_list",
                    QueryBuilders.matchPhraseQuery("tag_list.tag_id", recordQueryParam.getTagId()), ScoreMode.None));
        }
        // 同样是nest类型查询
        // 字符查询可通过使用wildcard进行类似MySQl的LIKE查询
        if (StringUtils.isNotBlank(recordQueryParam.getTagNamePattern())) {
            boolQueryBuilder.must(QueryBuilders.nestedQuery("tag_list",
                    QueryBuilders.wildcardQuery("tag_list.tag_name", "*" + recordQueryParam.getTagNamePattern() + "*"), ScoreMode.None));
        }

        // 创建 NativeSearchQuery 对象并设置排序与分页参数
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder().withQuery(boolQueryBuilder)
                .withSorts(SortBuilders.fieldSort("file_create_time").order(SortOrder.DESC))
                .withPageable(PageRequest.of(pageVo.getCurrent(), pageVo.getSize()))
                .withTrackTotalHits(true)
                .build();

        try {
            // 执行查询
            SearchHits<RecordESInfo> searchHits = elasticsearchRestTemplate.search(searchQuery, RecordESInfo.class);
            // 获取查询中的结果及分页参数
            if (searchHits.getTotalHits() > 0) {
                List<RecordESInfo> searchProductList = searchHits.stream().map(SearchHit::getContent).collect(Collectors.toList());
                pageVo.setTotal(searchHits.getTotalHits());
                pageVo.setPages((int) Math.ceil((double) pageVo.getTotal() / pageVo.getSize()));
                pageVo.setRecords(searchProductList);
            } else {
                pageVo.setTotal(0L);
                pageVo.setPages(0);
                pageVo.setRecords(new ArrayList<>());
            }
        } catch (Exception e) {
            // todo 处理查询中异常
        }
        return pageVo;
    }
```

传入参数，调用searchRecordESInfo，即可完成一次分页查询

# 自定义索引Setting
1. 创建setting.json，放到resources目录下，用于存放配置
例如这里创建一个自定义分词器，放在resources目录中的ElasticSearch目录下
```json
{
	  "analysis": {
	    "analyzer": {
	      "url_analyzer": {
	        "tokenizer": "standard",
	        "char_filter": [
	          "url_char_filter"
	        ]
	      }
	    },
	    "char_filter": {
	      "url_char_filter": {
	        "type": "pattern_replace",
	        "pattern": "\\.",
	        "replacement": "-"
	      }
	    }
	  }
	}
```
2. 在对应ES的实体类上加入```@Setting```注解，填写对应路径。这里使用刚才第一步中创建的```ElasticSearch/setting.json```

```java
@Data
@Document(indexName = "es_info")
@Setting(settingPath = "ElasticSearch/settings.json")
public class ESInfo{
    // data
}
```

3. 使用类似如下语句创建索引
```java
elasticsearchRestTemplate.indexOps(RecordESInfo.class).create(); // 创建实体类对应索引
elasticsearchRestTemplate.indexOps(RecordESInfo.class).putMapping(); // 创建实体类对应索引mapping
elasticsearchRestTemplate.indexOps(RecordESInfo.class).createSettings(); // 创建对应索引设置
```

