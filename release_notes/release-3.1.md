# StarRocks version 3.1

## 3.1.0-RC02

发布日期：2023 年 7 月 21 日

### 新增特性

#### 存算分离架构

支持[存储卷（Storage Volume）抽象](../deployment/deploy_shared_data.md#创建默认存储卷)，方便在存算分离架构中配置存储位置及鉴权等相关信息。后续创建库表时可以直接引用，提升易用性。

#### 存储、导入与查询

- 自动创建分区功能升级为表达式分区，建表时只需要使用简单的分区表达式（时间函数表达式或列表达式）即可配置需要的分区方式，并且数据导入时 StarRocks 会根据数据和分区表达式的定义规则自动创建分区。这种创建分区的方式，更加灵活易用，能满足大部分场景。
- 支持 LIST 分区方式。数据按照分区列枚举值列表进行分区，可以加速查询和高效管理分类明确的数据。

#### SQL 语句和函数

- 增加 Storage Volume 相关 SQL 语句：[CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE%20STORAGE%20VOLUME.md)、[ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER%20STORAGE%20VOLUME.md)、[DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP%20STORAGE%20VOLUME.md)、[SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET%20DEFAULT%20STORAGE%20VOLUME.md)、[DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC%20STORAGE%20VOLUME.md)、[SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW%20STORAGE%20VOLUMES.md)。
- 原表函数（Table function）TABLE() 更名为 [FILES()](../sql-reference/sql-functions/table-functions/files.md)，用以读取外部数据源文件以供查询或导入。

#### 权限与安全

增加 Storage Volume 相关[权限项](../administration/privilege_item.md#存储卷权限-storage-volume)，支持通过 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) 语句进行相关权限的赋予和撤销。

### 功能优化

#### 存储、导入与查询

- 优化统计信息收集，以降低对导入影响，提高收集性能。
- 优化并行 Merge 算法，在全排序场景下整体性能最高可提升 2 倍。
- 优化查询逻辑以不再依赖 DB 锁。

## 3.1.0-RC01

发布日期：2023 年 7 月 7 日

### 新增特性

#### 存算分离架构

- 新增支持主键模型（Primary Key）表，暂不支持持久化索引。
- 支持自增列属性 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)，提供表内全局唯一 ID，简化数据管理。
- 支持[导入时自动创建分区和使用分区表达式定义分区规则](../table_design/automatic_partitioning.md)，提高了分区创建的易用性和灵活性。

#### 数据湖分析

- 支持访问 Parquet 格式的 Iceberg v2 数据表。
<!--- 【公测中】支持写出数据到 Parquet 格式的 Iceberg 表。-->
- 【公测中】支持通过外部 [Elasticsearch catalog](../data_source/catalog/elasticsearch_catalog.md）访问 Elasticsearch，简化外表创建等过程。
- 【公测中】支持 [Paimon catalog](../data_source/catalog/paimon_catalog.md)，让用户也能用 StarRocks 对流式数据进行湖分析。

#### 存储、导入与查询

- 支持[随机分桶（Random Bucketing）](../table_design/Data_distribution.md#设置分桶)功能，建表时无需选择分桶键，StarRocks 将导入数据随机分发到各个分桶中，同时配合使用 2.5.7 版本起支持的自动设置分桶数量 (`BUCKETS`) 功能，用户可以不再关心分桶配置，大大简化建表语句。不过，在大数据、高性能要求场景中，建议继续使用 Hash 分桶方式，借助分桶裁剪来加速查询。
- 支持在 [INSERT INTO](../loading/InsertInto.md) 语句中使用表函数 TABLE()，从 AWS S3 或 HDFS 直接导入 Parquet 或 ORC 格式文件的数据，TABLE() 函数会还自动进行表结构 (Table Schema) 推断，不再需要提前创建 External Catalog 或文件外部表，大大简化导入过程。
- 支持[生成列（Generated Column）](../sql-reference/sql-statements/generated_columns.md)功能，自动计算生成列表达式的值并存储，且在查询时可自动改写，以提升查询性能。
- 支持通过 [Spark connector](../loading/Spark-connector-starrocks.md) 导入 Spark 数据至 StarRocks。相较于 [Spark Load](../loading/SparkLoad.md)，Spark connector 能力更完善。您可以自定义 Spark 作业，对数据进行 ETL 操作，Spark connector 只作为 Spark 作业中的 sink。
- 支持导入数据到 [MAP](../sql-reference/sql-statements/data-types/Map.md)、[STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md) 类型的字段，并且在 ARRAY、MAP、STRUCT 类型中支持了 Fast Decimal 类型。

#### SQL 语句和函数

- 支持通过 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER%20TABLE.md) 修改表的注释。[#21035](https://github.com/StarRocks/starrocks/pull/21035)

- 增加如下函数：

  - Struct 函数：[struct (row)](../sql-reference/sql-functions/struct-functions/row.md)、[named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
  - Map 函数：[str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md)、[map_concat](../sql-reference/sql-functions/map-functions/map_concat.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、[element_at](../sql-reference/sql-functions/map-functions/element_at.md)、[distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md)、[cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - Map 高阶函数：[map_filter](../sql-reference/sql-functions/map-functions/map_filter.md)、[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md)、[transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - Array 函数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) 支持 `ORDER BY`、[array_generate](../sql-reference/sql-functions/array-functions/array_generate.md)、[element_at](../sql-reference/sql-functions/array-functions/element_at.md)、[cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - Array 高阶函数：[all_match](../sql-reference/sql-functions/array-functions/all_match.md)、[any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - 聚合函数：[min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md)、[percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - 表函数 (Table function)：[generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - 日期函数：[next_day](../sql-reference/sql-functions/date-time-functions/next_day.md)、[previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md)、[last_day](../sql-reference/sql-functions/date-time-functions/last_day.md)、[makedate](../sql-reference/sql-functions/date-time-functions/makedate.md)、[date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - Bitmap 函数：[bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md)、[bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)

### 功能优化

#### 存算分离架构

优化存算分离架构中的数据缓存功能，可以指定热数据的范围，并防止冷数据查询占用 Local Disk Cache、进而影响热数据查询效率。

#### 物化视图

- 优化异步物化视图的创建：
  - 支持随机分桶 (Random Bucketing)，在创建物化视图时不指定分桶列 (Bucketing Column) 时默认采用随机分桶。
  - 支持通过 `ORDER BY` 指定排序键。
  - 支持使用 `colocate_group`、`storage_medium` 、`storage_cooldown_time` 等属性。
  - 支持使用会话变量 (Session Variable)，通过 `properties("session.<variable_name>" = "<value>")` 设置，灵活调整视图刷新的执行策略。
  - 支持基于视图（View）创建物化视图，优化数据建模场景的易用性，可以灵活使用视图和物化视图进行分层建模。
- 优化异步物化视图的查询改写：
  - 支持 Stale Rewrite，即允许指定时间内未刷新的物化视图直接用于查询改写，无论其对应基表数据是否更新。用户可在创建物化视图时通过 `mv_rewrite_staleness_second` 属性配置可容忍未刷新时间。
  - 基于 Hive Catalog 外表创建的物化视图支持 View Delta Join 场景的查询改写（需要定义主键和外键约束）。
  - 支持 Join 派生改写、Count Distinct、time_slice 函数等场景改写，优化 Union 改写。
- 优化异步物化视图的刷新：
  - 优化 Hive Catalog 外表物化视图的刷新机制，StarRocks 可以感知到分区级别的数据变更，自动刷新时仅刷新有数据变更的分区。
  - 支持通过 `REFRESH MATERIALIZED VIEW WITH SYNC MODE` 同步调用物化视图刷新任务。
- 增强异步物化视图的使用：
  - 支持通过 `ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}`  语句启用或禁用物化视图，已禁用（处于 `INACTIVE` 状态）的物化视图不会被刷新或用于查询改写，但是仍然可以直接查询。
  - 支持通过 `ALTER MATERIALIZED VIEW SWAP WITH` 替换物化视图，可以通过新建一个物化视图并进行原子替换来实现物化视图的 Schema Change。
- 优化同步物化视图：
  - 支持通过 SQL Hint `[_SYNC_MV_]` 直接查询同步物化视图，规避少量无法自动改写的场景。
  - 同步物化视图支持更多表达式，现可使用 `CASE-WHEN`、`CAST`、数学运算等表达式，扩展其使用场景。

#### 数据湖分析

- 优化 Iceberg 元数据缓存与访问，提升查询性能。
- 优化湖分析的数据缓存（Data Cache）功能，进一步提升湖分析性能。

#### 存储、导入与查询

执行 [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) 语句对主键模型表进行部分更新时支持启用列模式，适用于更新少部分列但是大量行的场景，更新性能可提升十倍。

#### SQL 语句和函数

- 条件函数 case、coalesce、if、ifnull、nullif 支持 ARRAY、MAP、STRUCT、JSON 类型。
- 以下 Array 函数支持嵌套结构类型 MAP、STRUCT、ARRAY：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 以下 Array 函数支持 Fast Decimal 类型：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### 问题修复

修复了如下问题：

- 执行 Routine Load 时，无法正常处理重连 Kafka 的请求。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- SQL 查询中涉及多张表、并且含有 `WHERE` 子句时，如果这些 SQL 查询的语义相同但给定表顺序不同，则有些 SQL 查询不能改写成对相关物化视图的使用。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- 查询包含 `GROUP BY` 子句时，会返回重复的数据结果。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- 调用 lead() 或 lag() 函数可能会导致 BE 意外退出。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 根据 External Catalog 外表物化视图重写部分分区查询失败。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- SQL 语句同时包含反斜线 (`\`) 和分号 (`;`) 时解析报错。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- 物化视图删除后，其基表数据无法清空 (Truncate)。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 行为变更

存算分离架构下删除建表时的 `storage_cache_ttl` 参数，Cache 写满后按 LRU 算法进行淘汰。
