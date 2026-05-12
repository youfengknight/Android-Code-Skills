---
name: "backup-consistency-check"
description: "检查备份系统中选择器、备份逻辑、恢复逻辑、备份信息显示之间的数据一致性。当用户问备份对不对、备份选漏了没、恢复和备份匹配吗、备份信息准不准时调用。"
---

# 备份一致性检查（Backup Consistency Checker）

自动对比备份系统中多个位置定义的备份项，找出不匹配、遗漏、多余的问题。

## 核心原理

备份系统有 **5 个关键位置** 定义了"要备份/恢复/显示哪些东西"，它们之间必须保持一致：

| 编号 | 位置 | 文件 | 作用 |
|------|------|------|------|
| ① | 实际备份逻辑 | `app/src/main/java/io/legado/app/help/storage/Backup.kt` | 真正执行备份，导出哪些文件 |
| ② | 备份选择器 | `app/src/main/java/io/legado/app/help/storage/BackupSelectorConfig.kt` | 用户在 UI 上可以选择备份哪些项 |
| ③ | 安卓端备份信息 | `app/src/main/java/io/legado/app/help/storage/BackupInfoHelper.kt` | 安卓端"查看备份信息"弹窗展示 |
| ④ | Web端备份信息 | `app/src/main/java/io/legado/app/api/controller/BackupController.kt` | Web端备份概览页面展示 |
| ⑤ | 恢复逻辑 | `app/src/main/java/io/legado/app/help/storage/Restore.kt` | 从备份中恢复哪些文件 |

## 执行步骤

### Step 1：提取各位置的备份项清单

逐一读取以下 5 个文件，提取每个文件中定义的备份项列表（文件名 + 显示名）：

#### ① Backup.kt — 实际备份的文件名列表

读取 `backupFileNames` 数组（约第 105-130 行），提取所有文件名字符串。

同时读取 `backup()` 方法（约第 351-562 行），检查：
- 每个 `selectedFiles.contains(xxx)` 的条件分支，确认实际写了哪些文件
- 是否有文件在 `backupFileNames` 中但 `backup()` 里没有导出逻辑
- 是否有文件在 `backup()` 里导出了但不在 `backupFileNames` 中

#### ② BackupSelectorConfig.kt — 选择器的备份项

读取 `allItems` 列表（约第 20-44 行），提取每个 `BackupItem` 的：
- `key`（唯一标识）
- `fileName`（文件名）
- `title`（显示名）
- `group`（分组）

#### ③ BackupInfoHelper.kt — 安卓端备份信息

读取 `displayNameMap`（约第 51-73 行），提取文件名到显示名的映射。

读取 `getBackupOverview()` 方法（约第 81-156 行），提取：
- `dbItems` 列表中的所有文件名
- `configFiles` 列表中的所有文件名
- 单独处理的文件（如 `DirectLinkUpload.ruleFileName`）
- 背景图片处理

#### ④ BackupController.kt — Web端备份信息

读取 `generateBackupOverview()` 方法（约第 301-415 行），提取：
- `backupItems` 列表中的所有 `BackupItemDef`
- `configItems` 列表中的所有 `ConfigItemDef`
- 背景图片处理

#### ⑤ Restore.kt — 恢复逻辑

读取 `restoreSelectedFiles()` 方法（约第 188-456 行）和 `restore()` 方法（约第 469-700+ 行），提取：
- 所有 `if ("xxx.json" in selectedSet)` 或 `if ("xxx.xml" in selectedSet)` 的条件
- 无条件恢复的文件（如 readRecordSession.json）
- 解压后尝试读取的所有文件名

### Step 2：构建对比矩阵

将提取到的 5 个列表汇总成一张对比表，格式如下：

```
| 文件名 | ① 备份逻辑 | ② 选择器 | ③ 安卓信息 | ④ Web信息 | ⑤ 恢复逻辑 | 状态 |
|--------|:---------:|:-------:|:---------:|:--------:|:---------:|:----:|
| bookshelf.json | ✅ | ✅ | ✅ | ✅ | ✅ | OK |
| xxx.json | ✅ | ❌ | ✅ | ❌ | ✅ | 不一致 |
```

### Step 3：逐项对比并分类问题

对每个文件名检查它在 5 个位置是否都出现，按以下规则分类：

#### 问题类型 A：选择器/信息展示缺少实际备份的项
- ① 有，② 无 → 选择器中用户无法控制此项的备份
- ① 有，③ 无 → 安卓端备份信息不显示此项
- ① 有，④ 无 → Web端备份信息不显示此项

#### 问题类型 B：选择器/信息展示多了实际未备份的项
- ① 无，② 有 → 选择器中有选项但实际不会备份（误导用户）
- ① 无，③ 有 → 信息展示了实际不会备份的项
- ① 无，④ 有 → Web端展示了实际不会备份的项

#### 问题类型 C：恢复与备份不匹配
- ① 有，⑤ 无 → 备份了但恢复时不会读取（数据丢失风险）
- ① 无，⑤ 有 → 恢复时会尝试读取不存在的文件（虽然通常无害，但逻辑不一致）

#### 问题类型 D：显示名不一致
- 同一个文件在 ②③④ 中的显示名不统一

### Step 4：检查 displayNameMap 完整性

单独检查 BackupInfoHelper.kt 的 `displayNameMap`：
- 是否覆盖了所有在 `getBackupOverview()` 中出现的文件名
- 未覆盖的文件名在 UI 上会显示原始文件名，影响用户体验

### Step 5：检查分类匹配

检查 BackupInfoHelper.kt 的 `categoryConfig` 分类规则：
- 关键词是否能正确匹配到所有备份项的文件名
- 是否有备份项无法被任何分类匹配到（会被归入"其他"）

### Step 6：输出分析报告

按照以下格式输出最终报告：

```markdown
## 备份一致性检查报告

### 对比矩阵
（完整的 5 列对比表）

### 发现的问题

#### ❌ 严重问题
（功能上会导致数据丢失或误导用户的问题）

#### ⚠️ 一般问题
（逻辑不一致但不影响功能的问题）

#### 💡 建议
（可优化的点）

### 修复建议
（针对每个问题给出具体的修复方案）
```

## 关键文件路径速查

```
app/src/main/java/io/legado/app/help/storage/Backup.kt
app/src/main/java/io/legado/app/help/storage/BackupSelectorConfig.kt
app/src/main/java/io/legado/app/help/storage/BackupInfoHelper.kt
app/src/main/java/io/legado/app/help/storage/Restore.kt
app/src/main/java/io/legado/app/api/controller/BackupController.kt
```

## 注意事项

1. `DirectLinkUpload.ruleFileName` 是动态值（运行时常量），不是字符串字面量，提取时需注意
2. `ReadBookConfig.configFileName`、`ReadBookConfig.shareConfigFileName`、`ThemeConfig.configFileName`、`BookCover.configFileName` 同理，需在代码中找到实际值
3. `readRecordSession.json` 是一个特殊的灰色地带：Restore 会尝试读取它，但 Backup 从不导出它
4. 背景图片（`bg` 目录）是文件级备份，不是 JSON，需要单独关注
5. `runtimeSourceCache.json` 只在 `BackupConfig.fullBackup` 为 true 时才会备份
6. 对比时要注意 Backup.kt 中 `backupFileNames` 和 `backup()` 方法可能不完全同步——两个地方都需检查
