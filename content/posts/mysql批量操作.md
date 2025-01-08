---
author:
  name: Hugo Authors
date: 2025-01-08
linktitle: Migrating from Jekyll
title: mysql批量操作
type:
  - post
  - posts
weight: 10
series:
  - Hugo 101
aliases:
  - /blog/migrate-from-jekyll/
---
### 1. mybatis-plus的批量操作

有id的可以用mybatis-plus的批量操作方法。

### 2. mybatis的批量操作

没有id的也可以

```java
@Component  
@RequiredArgsConstructor  
public class MyBatisBatchExecutor {  
  
    private final SqlSessionFactory sqlSessionFactory;  
  
    public <T> void executeBatch(Class<T> mapperClass, Consumer<T> consumer) {  
        try (SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)) {  
            T mapper = sqlSession.getMapper(mapperClass);  
            consumer.accept(mapper);    // 接受mapper参数，并执行一些操作  
            sqlSession.flushStatements();   // 显式刷新批量操作  
        }  
    }  
}
```

使用

```java
@Slf4j  
@Service  
public class PointUserDetailService extends ServiceImpl<PointUserDetailMapper, PointUserDetailPO> {  
  
    @Autowired  
    private MyBatisBatchExecutor myBatisBatchExecutor;  
  
    @Transactional(rollbackFor = Exception.class)  
    public void expireJob() {  
        // something
        // mybatis-plus，批量修改，需要id 
        updateBatchById(list.stream()  
                .map(x -> {  
                    PointUserDetailPO map = BeanMapper.map(x, PointUserDetailPO.class);  
                    map.setPointNumber(0);  
                    return map;  
                }).collect(Collectors.toList()));  
		// mybatis，批量修改
        myBatisBatchExecutor.executeBatch(PointUserMapper.class, mapper -> list.stream()  
                .collect(Collectors.groupingBy(PointUserDetailPO::getUserId,  
                        Collectors.summingInt(PointUserDetailPO::getPointNumber)))  
                .forEach(mapper::updatePointByUserId));  
  
        // something
    }  
}
```

### 3. 批量sql语句

```sql
<insert id="insertOrUpdateBatch" keyProperty="id" useGeneratedKeys="true">  
    insert into ch_qxj.early_warning(warning_id, station_number, publish_place)  
    values  
    <foreach collection="entities" item="entity" separator=",">  
        (#{entity.warningId}, #{entity.stationNumber}, #{entity.publishPlace})  
    </foreach>  
    on duplicate key update  
     publish_place = values(publish_place)    
</insert>
```
1. keyProperty="id" useGeneratedKeys="true"表示会把自动生成的主键回填到实体中
2. on duplicate key update表示如果插入时主键或唯一索引冲突，则会更新后面的字段

### 总结

个人感觉1和2结合着用比较简单，3比较麻烦，还得写sql语句。