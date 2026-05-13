---
name: room-schema-fix
description: 通过对比 entities、schema JSON 和迁移方案，诊断并修复 Android Room 数据库在 entity 或 database 变更后的 schema 校验崩溃；增加数据库版本号，添加重命名/新增/删除列的迁移，并验证生成的 Room 代码。
metadata:
  short-description: 修复 Room schema 崩溃
---

# Room Schema 修复

当 Android 应用在启动时因 Room schema 校验错误而崩溃时使用此技能，尤其适用于修改了 `@Entity`、`@Database`、`@DatabaseView` 或 DAO 查询之后。

## 工作流程

1. 识别不匹配之处。
   - 检查当前的 database 类、受影响的 entities，以及 `schemas/<db-name>/<version>.json` 下的导出 schema JSON。
   - 查找变更的列名、默认值、可空性、索引、主键、视图或表名。
   - 搜索迁移钩子：`Migration`、`AutoMigration`、`AutoMigrationSpec`、`@RenameColumn`、`@DeleteColumn`。

2. 分类 schema 变更类型。
   - 仅重命名：添加重命名迁移，不要依赖简单的 entity 编辑。
   - 添加可空列或带默认值列：提升数据库版本并添加迁移路径。
   - 删除列或表：如果自动迁移不够，使用删除迁移或手动 SQL。
   - 类型变更、主键变更或数据重构：使用带 SQL 复制/重建的手动 `Migration`。

3. 修复数据源。
   - 每当导出的 schema 变更时，提升 `@Database(version = ...)` 版本号。
   - 保持之前的 schema JSON 不变，添加新版本的 JSON。
   - 将迁移添加到 database builder 的 `autoMigrations` 列表或手动迁移数组中。
   - 优先保留用户数据。只有在应用可以安全丢弃数据库时，才使用破坏性迁移。

4. 验证修复。
   - 运行特定 flavor 的 KSP 任务和编译任务。
   - 如果某个 flavor 被无关配置阻塞，选择另一个能到达 Room 处理阶段的 flavor。
   - 确认生成的 `AppDatabase_*_Impl` 迁移代码与预期的 SQL 或列映射匹配。

## 实用检查

- 如果在最近的 entity 重命名后启动崩溃，假设旧安装的数据库仍然存在。
- 如果 schema JSON 变更但版本号没变，先修复版本号。
- 如果新 entity 字段名与旧列名不同，确保迁移实际将旧数据映射到新列。
- 如果 Room 报告 identity hash 不匹配，比较生成的 schema JSON 和生成的 `AppDatabase_Impl` 输出，而不仅仅是 Kotlin 源码。

## 有效的触发短语

- `Room 数据库 Schema 校验失败`
- `RoomOpenHelper$OpenHelperDelegate.validateMigration`
- `A migration from X to Y was required but not found`
- `unexpected column`
- `missing column`
- `identity hash mismatch`
- `数据库打开闪退`

## 输出格式

应用此技能时，报告以下内容：
- 具体的错误信息
- 涉及的文件
- 版本升级和迁移策略
- 运行的验证命令
