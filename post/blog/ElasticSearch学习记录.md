# ElasticSearch学习记录

## 一、下载6.5.4版本的ElasticSearch :

 https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.5.4.zip 

下载后解压缩，并执行bin/elasticsearch.bat即可启动ES

用浏览器打开 http://localhost:9200/ 如无意外，可看到类似以下信息

```json
{
  "name" : "O-7kAYF",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "bpZQHGOIQ4WZCsIRYK1Q9g",
  "version" : {
    "number" : "6.5.4",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "d2ef93d",
    "build_date" : "2018-12-17T21:17:40.758843Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

**下载elasticsearch-analysis-ik插件**

https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.5.4/elasticsearch-analysis-ik-6.5.4.zip

解压后获得elasticsearch-analysis-ik-6.5.4目录，把该目录复制到ES安装路径下的plugs目录

准备工作好后  开始开发



## 二、创建elasticsearchdemo

### 			1.项目结构如下：

![image-20200923095139389](C:\Users\tticar\AppData\Roaming\Typora\typora-user-images\image-20200923095139389.png)

### 				2.在pom.xml文件中引入依赖：

```XMl
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>elasticsearchdemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>elasticsearchdemo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.querydsl/querydsl-core -->
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-core</artifactId>
            <version>4.2.1</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.1</version>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**注意spring-boot-starter-parent的版本！**不同的版本可能elasticsearch的api差别很大。

### 			3.创建ElasticSearchClientConfig配置类

```java
@Configuration
public class ElasticSearchClientConfig {

    @Bean
    public RestHighLevelClient restHighLevelClient() {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1", 9200, "http")
                )
        );
        return client;
    }
}
```

### 			4.创建配件实体bean：

```java
/**
 * @Description 汽车配件
 * @Author WanJianTao
 * @Date 2020/9/18 14:12
 * @Version 1.0
 **/
@Data
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName = "car", type = "parts")
public class Parts {

    @Id
    private Long id;

    // 配件UID
    @Field(type = FieldType.Text)
    private String partsId;

    // 配件名称
    @Field(type = FieldType.Text)
    private String name;

    // 配件说明
    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String text;

    // 配件点赞
    private int likes;

    // 上架时间

    @Field(type = FieldType.Date)
    private Date date;
}
```

### 			5.创建ElasticsearchTest测试类：

```java
@SpringBootTest
public class ElasticsearchTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @Test
    public void contextLoads() {
//        Parts parts= new Parts(1l,"P123456789","huohuasai","qichedianhuozhuangzhe",1,new Date());
//        IndexQuery indexQuery = new IndexQueryBuilder().withObject(parts).build();
//        elasticsearchTemplate.index(indexQuery);
        // 删除索引
        elasticsearchTemplate.deleteIndex("car");

        List<IndexQuery> indexQueries = Arrays.asList(
                new IndexQueryBuilder().withObject(new Parts(1l,"P123456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(2l,"P223456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(3l,"P323456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(4l,"P423456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(5l,"P523456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(6l,"P623456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(7l,"P723456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(8l,"P823456789","huohuasai","qichedianhuozhuangzhe",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(9l,"P923456789","火花塞","汽车点火装置",1,new Date())).build(),
                new IndexQueryBuilder().withObject(new Parts(10l,"P1023456789","点火","中文分词测试",1,new Date())).build()
        );
        elasticsearchTemplate.bulkIndex(indexQueries);


//        GetQuery getQuery = new GetQuery();
//        getQuery.setId("1");
//        Parts parts1 = elasticsearchTemplate.queryForObject(getQuery, Parts.class);
//        System.out.println(ToStringBuilder.reflectionToString(parts1));
    }
}
```

### 			6.创建PageHelper<T>类

```java
@Data
public class PageHelper<T> {

    private long total;
    private List<T> list;
    private int pageNum;
    private int pageSize;
    private int pages;
    private int previousPageNum;
    private int nextPageNum;
    private boolean previousPage;
    private boolean nextPage;
    private int[] navigatePageNums;

    public PageHelper(Page<T> page, int pageNum, int pageSize) {

        this.list = page.getContent();
        this.total = page.getTotalElements();
        this.pageNum = pageNum;
        this.pageSize = pageSize;
        this.pages = page.getTotalPages();
        this.previousPage = false;
        this.nextPage = false;

        int startNum = 1;
        int endNum = pages;
        int navigateNums = 10; //导航栏显示数字个数
        if (pages <= navigateNums) {
        } else if ((pageNum + (navigateNums + 1) / 2) > pages) {
            startNum = pages - (navigateNums - 1);
            endNum = pages;
        } else if (pageNum <= (navigateNums + 1) / 2) {
            endNum = navigateNums;
        } else {
            if (navigateNums % 2 == 0) {
                startNum = pageNum - navigateNums / 2;
                endNum = pageNum + (navigateNums - 1) / 2;
            } else {
                startNum = pageNum - navigateNums / 2;
                endNum = pageNum + navigateNums / 2;
            }
        }
        this.navigatePageNums = new int[endNum - startNum + 1];
        for (int i = startNum; i <= endNum; i++) {
            this.navigatePageNums[i - startNum] = i;
        }

        for (int i = 0; i < this.pages; ++i) {
            this.navigatePageNums[i] = i + 1;
        }
        if (this.pageNum > 1) {
            this.previousPage = true;
            this.previousPageNum = this.pageNum - 1;
        }

        if (this.pageNum < this.pages) {
            this.nextPage = true;
            this.nextPageNum = this.pageNum + 1;
        }
    }
}
```

### 			7.创建PostsRequest类

```java
@Data
public class PostsRequest {

    private String keywords;
    private Integer pageNum;
    private Integer pageSize;
    private int type;
    private String name;
    private String startDate;
    private String endDate;
    private String minLikes;
    private String maxLikes;
}
```

### 			8.创建HighlightResultMapper类

```java
public class HighlightResultMapper implements SearchResultMapper {
    @Override
    public <T> AggregatedPage<T> mapResults(SearchResponse searchResponse, Class<T> clazz, Pageable pageable) {
        long totalHits = searchResponse.getHits().getTotalHits();
        List<T> list = new ArrayList<>();
        SearchHits hits = searchResponse.getHits();
        if (hits.getHits().length> 0) {
            for (SearchHit searchHit : hits) {
                Map<String, HighlightField> highlightFields = searchHit.getHighlightFields();
                T item = JSON.parseObject(searchHit.getSourceAsString(), clazz);
                Field[] fields = clazz.getDeclaredFields();
                for (Field field : fields) {
                    field.setAccessible(true);
                    if (highlightFields.containsKey(field.getName())) {
                        try {
                            field.set(item, highlightFields.get(field.getName()).fragments()[0].toString());
                        } catch (IllegalAccessException e) {
                            e.printStackTrace();
                        }
                    }
                }
                list.add(item);
            }
        }
        return new AggregatedPageImpl<>(list, pageable, totalHits);
    }

    @Override
    public <T> T mapSearchHit(SearchHit searchHit, Class<T> aClass) {
        return null;
    }
}
```

### 			9.创建PartsController

```java
/**
 * @Description 汽车配件Api
 * @Author WanJianTao
 * @Date 2020/9/18 16:20
 * @Version 1.0
 **/
@RestController
@RequestMapping("/parts")
public class PartsController {


    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @RequestMapping("/partsPage")
    public  PageHelper<Parts> query(@RequestBody  PostsRequest postsRequest) {
        Integer pageNum = postsRequest.getPageNum();
        Integer pageSize = postsRequest.getPageSize();
        if(pageNum == null || pageNum<= 0)
            pageNum = 1;
        if(pageSize == null || pageSize <= 0)
            pageSize = 5;

        String nameFieldName = "name";
        String textFieldName = "text";
        String preTags = "<span style=\"color:#F56C6C\">";
        String postTags = "</span>";
        HighlightBuilder.Field nameField = new HighlightBuilder.Field(nameFieldName).preTags(preTags).postTags(postTags);
        HighlightBuilder.Field textField = new HighlightBuilder.Field(textFieldName).preTags(preTags).postTags(postTags);


         /*  // 空查询
        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchAllQuery())
                .build();*/


        if(postsRequest.getType() == 0) {

            SearchQuery searchQuery;
            // 有关键字就查关键字
            if(StringUtils.isNotBlank(postsRequest.getKeywords())) {
                searchQuery = new NativeSearchQueryBuilder()
                        .withQuery(QueryBuilders.multiMatchQuery(postsRequest.getKeywords(), "name", "text"))
                        // 根据时间排序（最新时间在前面）
                        .withSort(SortBuilders.fieldSort("date").order(SortOrder.DESC))
                        .withPageable(QPageRequest.of(pageNum - 1, pageSize))
                        .withHighlightFields(nameField, textField)
                        .build();
            }
            // 没有就空查询
            else {
                searchQuery = new NativeSearchQueryBuilder()
                        .withQuery(QueryBuilders.matchAllQuery())
                        // 根据时间排序（最新时间在前面）
                        .withSort(SortBuilders.fieldSort("date").order(SortOrder.DESC))
                        .withPageable(QPageRequest.of(pageNum - 1, pageSize))
                        .withHighlightFields(nameField, textField)
                        .build();
            }
            Page<Parts> parts = elasticsearchTemplate.queryForPage(searchQuery, Parts.class,new HighlightResultMapper());
            PageHelper<Parts> partsPageHelper = new PageHelper<>(parts,pageNum,pageSize);
        /*List<Parts> list = parts.getContent().stream().collect(Collectors.toList());
        System.out.println(partsPageHelper.toString());*/
            return partsPageHelper;
        }else{
            SearchQuery searchQuery;
            BoolQueryBuilder queryBuilder = QueryBuilders.boolQuery();
            if(StringUtils.isNotBlank(postsRequest.getName())) {
                queryBuilder.must(QueryBuilders.termQuery("name.keyword", postsRequest.getName()));
            }
            if(StringUtils.isNotBlank(postsRequest.getStartDate()) || StringUtils.isNotBlank(postsRequest.getEndDate())) {
                RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("date");
                if(StringUtils.isNotBlank(postsRequest.getStartDate())) {
                    rangeQueryBuilder = rangeQueryBuilder.gte(parseDate(postsRequest.getStartDate()).getTime());
                }
                if(StringUtils.isNotBlank(postsRequest.getEndDate())) {
                    rangeQueryBuilder = rangeQueryBuilder.lte(parseDate(postsRequest.getEndDate()).getTime());
                }
                queryBuilder.must(rangeQueryBuilder);
            }
            if(StringUtils.isNotBlank(postsRequest.getMinLikes()) || StringUtils.isNotBlank(postsRequest.getMaxLikes())) {
                RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("likes");
                if(StringUtils.isNotBlank(postsRequest.getMinLikes())) {
                    int minLikes = NumberUtils.toInt(postsRequest.getMinLikes());
                    rangeQueryBuilder = rangeQueryBuilder.gte(minLikes);
                }
                if(StringUtils.isNotBlank(postsRequest.getMaxLikes())) {
                    int maxLikes = NumberUtils.toInt(postsRequest.getMaxLikes());
                    rangeQueryBuilder = rangeQueryBuilder.lte(maxLikes);
                }
                queryBuilder.must(rangeQueryBuilder);
            }
            searchQuery = new NativeSearchQueryBuilder()
                    .withPageable(QPageRequest.of(pageNum - 1, pageSize))
                    .withQuery(queryBuilder)
                    .withSort(SortBuilders.fieldSort("date").order(SortOrder.DESC))
                    .build();
            Page<Parts> parts = elasticsearchTemplate.queryForPage(searchQuery, Parts.class,new HighlightResultMapper());
            PageHelper<Parts> partsPageHelper = new PageHelper<>(parts,pageNum,pageSize);
            return partsPageHelper;
        }
    }
}
```

## 三、测试

- 此次项目的是要完成对于配件搜索的支持中文分词、模糊查询、范围查找、精确查找、高亮显示和分页

测试结果如下：

1.

![1](C:\Users\tticar\Desktop\elsearchDemo\e1.png)

2.

![2](C:\Users\tticar\Desktop\elsearchDemo\e2.png)

3.

![3](C:\Users\tticar\Desktop\elsearchDemo\e3.png)

4.

![4](C:\Users\tticar\Desktop\elsearchDemo\e4.png)

# 总结

因为此次代码中都有详细的解释，因此我就不一一介绍了，仅以此来记录此次学习的结果，以提供之后的借鉴。