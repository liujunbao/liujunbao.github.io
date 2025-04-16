---
author:
  name: Hugo Authors
date: 2025-04-16
linktitle: Migrating from Jekyll
title: mybatis-plus自定义通用List类型处理器
type:
  - post
  - posts
weight: 10
series:
  - Hugo 101
aliases:
  - /blog/migrate-from-jekyll/
---
### 1. 定义

```java
package nxcloud.middle.activity.domain;

import com.baomidou.mybatisplus.extension.handlers.AbstractJsonTypeHandler;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.type.TypeFactory;

import java.util.List;

/**
 * @author xiaobao
 */
public class JacksonListTypeHandler <T> extends AbstractJsonTypeHandler<List<T>> {

    private static final ObjectMapper objectMapper = new ObjectMapper();

    private final Class<T> clazz;

    public JacksonListTypeHandler(Class<T> clazz) {
        this.clazz = clazz;
    }

    @Override
    protected List<T> parse(String json) {
        try {
            JavaType javaType = TypeFactory.defaultInstance().constructCollectionType(List.class, clazz);
            return objectMapper.readValue(json, javaType);
        } catch (Exception e) {
            throw new RuntimeException("Failed to parse JSON to List<" + clazz.getName() + ">", e);
        }
    }

    @Override
    protected String toJson(List<T> obj) {
        try {
            return objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            throw new RuntimeException("Failed to convert List<" + clazz.getName() + "> to JSON", e);
        }
    }
}
```

### 使用
你在实体类中需要声明的时候，要指定具体类型的 TypeHandler：

❗必须给每种类型写一个子类（因为 MyBatis 无法传泛型）

```java
@MappedTypes(List.class)
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ActivityRulesListTypeHandler extends JacksonListTypeHandler<ActivityRules> {
    public ActivityRulesListTypeHandler() {
        super(ActivityRules.class);
    }
}
```

```java
/**
     * 优惠规则
     */
    @TableField(typeHandler = ActivityRulesListTypeHandler.class)
    private List<ActivityRules> rules;

    /**
     * 适用商品
     */
    @TableField(typeHandler = JacksonTypeHandler.class)
    private ActivityApplyGoods applyGoods;
```