# 项目开发规范

## 角色定位

你是一位资深 Java 工程师，熟悉 Java 17+、Spring Boot 3.x、Maven/Gradle。
代码风格严格遵循**阿里巴巴 Java 开发手册**规范。

---

## 一、Java 编码规范

### 基本原则
- 使用 Java 17+ 特性（Record、sealed class、Pattern Matching、Stream 等）
- 依赖注入统一使用**构造器注入**，禁止 Field 注入（`@Autowired` 直接加字段）
- 所有对外接口返回统一包装类（如 `Result<T>`）
- 统一异常处理由 `GlobalExceptionHandler` 负责，业务层不要吞异常或只打印堆栈

### 命名规范（阿里规范）
- 类名：UpperCamelCase，如 `UserService`
- 方法名 / 变量名：lowerCamelCase，如 `getUserById`
- 常量：全大写下划线，如 `MAX_RETRY_COUNT`
- 包名：全小写，如 `com.example.service`

### 异常处理
- 禁止 `catch (Exception e) { e.printStackTrace(); }`
- 捕获异常必须记录上下文信息（入参、业务场景等）
- SQL 异常不得暴露给业务层，由 `GlobalExceptionHandler` 统一转换：
  - `handleSQLException`
  - `handleMyBatisException`
  - `handleSpringSQLException`

### 其他约定
- 善用 `Optional` 避免 NPE，不允许直接返回 `null` 集合（返回空集合代替）
- 日志使用 SLF4J，禁止使用 `System.out.println`

---

## 二、数据库设计规范

### 基础规范
- **命名**：库、表、字段、索引统一使用**全小写下划线**命名，如 `user_order_tab`
- **引擎**：统一使用 InnoDB（支持事务、行级锁）
- **字符集**：统一 `utf8mb4`，排序规则 `utf8mb4_general_ci` 或 `utf8mb4_bin`
- **主键**：明确选择自增主键或雪花算法 ID（分布式场景必须用雪花 ID 保证唯一性，避免 UUID 作聚簇索引）
- **冗余字段**：为减少联表操作，允许适当引入冗余字段

### 字段规范
- 所有字段尽可能设置 `NOT NULL` + 默认值（NULL 影响索引效率，Java 侧容易空指针）
- **数字类型**：
  - 主键、时间戳 → `bigint`
  - 状态/标识 → `tinyint`
- **字符串类型**：
  - 尽可能避免 `text`
  - 使用 `varchar` 时根据字段含义设置合理长度（ID、Code 等短字段不要设 255）
- **日期类型**：优先使用 `datetime`（Java 端处理更方便）

### 索引规范
- 主键使用自增 ID 或雪花 ID（保持递增趋势），**禁止用 UUID 做聚簇索引**
- 尽量使用**覆盖索引**，避免回表
- 组合索引：
  - 高频查询、区分度高的字段放最左边（遵循最左匹配原则）
  - 注意索引失效场景：不遵循最左匹配、中间字段断档、范围查询字段后的列、`LIKE %xxx` 模糊查询
- 业务排序字段（如时间）尽量建索引；既有 `WHERE` 又有 `ORDER BY` 时，需综合评估索引策略

---

## 三、SQL 编写规范（DML）

- **禁止 `SELECT *`**：明确列出所需字段，利用覆盖索引减少数据量
- **禁止对索引列做计算**：会导致索引失效；将计算结果预处理后传入，或将计算移到等号右侧
- **避免 JOIN**：尽可能在内存中完成联表逻辑（小数据量场景可评估）
- **深分页优化**：禁止 `LIMIT 1000000, 10`，改用游标分页：
  ```sql
  WHERE id > last_id LIMIT 10
  ```
- **批量插入**：禁止在 XML 中写超长批量 `INSERT VALUES`（存在内存溢出风险），统一使用 MyBatis Batch 模式：
  ```java
  try (SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH, false)) {
      XxxMapper mapper = sqlSession.getMapper(XxxMapper.class);
      for (int i = 0; i < list.size(); i++) {
          mapper.insert(list.get(i));
          if (i % 100 == 99) {
              sqlSession.flushStatements();
          }
      }
      sqlSession.commit();
  }
  ```

---

## 四、安全规范

- **防 SQL 注入**：MyBatis 中一律使用 `#{}` 占位符，**禁止使用 `${}`**（后者直接字符串替换，存在注入风险）
- **SQL 异常隔离**：SQL 异常不得透传到业务响应，必须由 `GlobalExceptionHandler` 捕获并转换为友好提示

---

## 五、禁止事项（Hard Rules）

| 禁止行为 | 原因 |
|---------|------|
| Field 注入（`@Autowired` 加字段）| 破坏可测试性，隐藏依赖 |
| `catch` 后只做 `e.printStackTrace()` | 吞异常，线上排查困难 |
| `SELECT *` | 浪费带宽，破坏覆盖索引 |
| `${}` 拼接 SQL 参数 | SQL 注入风险 |
| UUID 做主键 | 随机写入破坏 B+ 树，性能差 |
| XML 中超长批量 INSERT VALUES | 内存溢出风险 |
| 对索引列做函数/计算 | 索引失效 |
| `LIMIT offset, n` 深分页 | 全表扫描，性能灾难 |

---

## 六、gstack

使用 `/browse` skill 进行所有网页浏览，**禁止使用** `mcp__claude-in-chrome__*` 工具。

可用 skills：
/office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /design-shotgun, /design-html, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse, /connect-chrome, /qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /retro, /investigate, /document-release, /codex, /cso, /autoplan, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade, /learn