## bool查询

### **`bool` 查询简介**

在 Elasticsearch 中，`bool` 查询是一种复合查询，它允许将多个查询条件组合起来，并通过逻辑运算（`must`、`must_not`、`should` 和 `filter`）进行灵活筛选。

**用途**：
 `bool` 查询用于构建更复杂的查询逻辑，类似于 SQL 中的 `AND`、`OR` 和 `NOT` 逻辑。

------

### **`bool` 查询的基本结构**

```json
{
  "query": {
    "bool": {
      "must": [ ... ],        // 必须满足的条件（相当于 AND）
      "must_not": [ ... ],    // 不能满足的条件（相当于 NOT）
      "should": [ ... ],      // 可以满足的条件（相当于 OR）
      "filter": [ ... ]       // 不影响得分的过滤条件
    }
  }
}
```

------

### **`bool` 查询内部的逻辑操作**

1. **`must`（类似 SQL 中的 `AND`）**

   - **作用**：文档必须满足指定条件。

   - **影响得分**：会影响文档的 `_score`（相关性得分）。

   - 示例

     ：

     ```json
     {
       "bool": {
         "must": [
           { "match": { "field1": "value1" }},
           { "match": { "field2": "value2" }}
         ]
       }
     }
     ```

     - **解释**：文档必须满足 `field1` 包含 `value1` 且 `field2` 包含 `value2`。

2. **`must_not`（类似 SQL 中的 `NOT`）**

   - **作用**：文档不能满足指定条件。

   - **影响得分**：不影响得分，纯粹用于排除文档。

   - 示例

     ：

     ```json
     {
       "bool": {
         "must_not": [
           { "term": { "field": "value" }}
         ]
       }
     }
     ```

     - **解释**：排除所有 `field` 为 `value` 的文档。

3. **`should`（类似 SQL 中的 `OR`）**

   - **作用**：条件是可选的，但满足条件会提高文档的 `_score`。

   - **最低匹配数**：可通过 `minimum_should_match` 控制最少满足的条件数。

   - 示例

     ：

     ```json
     {
       "bool": {
         "should": [
           { "match": { "field1": "value1" }},
           { "match": { "field2": "value2" }}
         ],
         "minimum_should_match": 1
       }
     }
     ```

     - **解释**：文档满足至少一个条件即可被返回。

4. **`filter`（用于过滤，不影响得分）**

   - **作用**：类似于 `must`，但不参与相关性得分计算，性能更高。

   - **用途**：适用于固定过滤条件（例如时间范围、字段值等）。

   - 示例

     ：

     ```json
     {
       "bool": {
         "filter": [
           { "term": { "status": "active" }},
           { "range": { "date": { "gte": "2024-01-01" }}}
         ]
       }
     }
     ```

     - **解释**：过滤 `status` 为 `active` 且 `date` 大于等于 `2024-01-01` 的文档。

------

## **其他查询类型**

除了 `bool` 查询，还有以下常见查询类型：

1. ### **`match` 查询**

   - 用于全文检索，将查询词分词后匹配。

   ```json
   { "match": { "field": "value" }}
   ```

2. ### **`term` 查询**

   - 用于精确匹配，不进行分词。

   ```json
   { "term": { "field": "value" }}
   ```

3. **`range` 查询**

   - 用于范围查询，例如时间范围、数值范围等。

   ```json
   { "range": { "field": { "gte": 10, "lte": 20 }}}
   ```

4. **`wildcard` 查询**

   - 支持通配符查询（`*` 和 `?`）。

   ```json
   { "wildcard": { "field": "val*" }}
   ```

5. **`prefix` 查询**

   - 查询字段值以特定前缀开头的文档。

   ```json
   { "prefix": { "field": "val" }}
   ```

6. **`exists` 查询**

   - 查询某个字段存在的文档。

   ```json
   { "exists": { "field": "field_name" }}
   ```

7. **`fuzzy` 查询**

   - 支持模糊匹配，适用于拼写错误的搜索场景。

   ```json
   { "fuzzy": { "field": "valeu" }}
   ```

8. **`match_phrase` 查询**

   - 短语匹配，查询精确顺序的词组。

   ```json
   { "match_phrase": { "field": "quick fox" }}
   ```

9. **`multi_match` 查询**

   - 对多个字段进行匹配查询。

   ```json
   {
     "multi_match": {
       "query": "value",
       "fields": ["field1", "field2"]
     }
   }
   ```

10. **`constant_score` 查询**

    - 不计算相关性得分，所有匹配文档的得分相同。

    ```json
    {
      "constant_score": {
        "filter": { "term": { "field": "value" }}
      }
    }
    ```

11. **`function_score` 查询**

    - 使用自定义函数对文档进行评分调整。

------

### **总结**

- **`bool` 查询**：是用于组合多个条件的核心查询，支持 `must`、`must_not`、`should` 和 `filter`。
- **其他查询类型**：如 `match`、`term`、`range`、`exists` 等，用于实现不同的匹配逻辑。
- **灵活组合**：可以嵌套使用各种查询来实现复杂的搜索需求。

通过这些查询，Elasticsearch 能够高效地处理简单到复杂的搜索场景。