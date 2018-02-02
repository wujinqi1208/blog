---
title: mongo
---

# mongo

## mongo 命令

- **索引**

  ``` bash
  #创建组合索引
  db.getCollection('ttx_entity_history').ensureIndex({"table":1,"id":1},{"name":"idx_table_id"})
  #查询索引
  db.getCollection('ttx_entity_history').getIndexes()
  ```
