mybatis的sql语句为什么自动加了deleted=0

MyBatis自动加上`deleted=0`条件通常是由于使用了逻辑删除（soft delete）功能，这种功能会在查询时自动加上逻辑删除字段的条件。逻辑删除的实现一般通过MyBatis插件或框架扩展来完成，比如MyBatis-Plus。

### 逻辑删除（Soft Delete）概述

逻辑删除是一种不真正删除数据记录的方式，而是通过设置一个标志位（如`deleted`字段）来表示记录已被删除。这种方式可以避免物理删除数据带来的数据丢失问题，同时可以保留删除记录以供审计和恢复。

#### 2. 配置逻辑删除

在配置类中启用MyBatis-Plus的逻辑删除功能。例如，在`application.yml`中进行配置：

```yaml
yaml复制代码mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-not-delete-value: 0
      logic-delete-value: 1
```

如果删除logic-delete-field: deleted 则不会再加