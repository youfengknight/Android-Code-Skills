---
name: "compose-reuse-checker"
description: "分析 Compose 界面是否使用了项目中可复用的组件和主题系统。适用于检查 Compose 屏幕是否符合项目规范或需要重构以使用共享组件。"
---

# Compose 复用检查器

分析 Compose UI 文件，判断其是否正确使用了项目的可复用组件和主题系统。

## 使用场景

- 用户要求检查 Compose 屏幕是否符合项目规范
- 用户想要改进/重构现有的 Compose 界面
- 用户询问"这个界面是否使用了共享组件"或类似问题
- 在重构 Compose 界面以提高代码复用性之前

## 分析步骤

### 1. 识别目标文件

接受以下任一形式：
- 包含 `@Composable` 函数的 `.kt` 文件的完整路径
- 用户直接提供的文件内容

### 2. 检查主题系统使用

在文件中查找以下导入和使用：

**必要的主题导入：**
```kotlin
import io.legado.app.ui.theme.LegadoTheme
import io.legado.app.ui.theme.*
```

**必要的颜色函数**（应替代硬编码颜色使用）：
```kotlin
// 来自 CommonPageColors.kt - 页面级颜色
pageTopBarContainerColor()       // TopBar/Header 背景色
pageCardContainerColor()         // 卡片容器背景色
pageCardElevatedContainerColor() // 悬浮卡片背景色
pageHeaderContainerColor()       // Header 背景色
pageSecondaryTextColor()         // 次要文字颜色
pageAccentColor()                // 强调/高亮颜色
pageSurfaceVariantColor()         // Surface 变体颜色
pageMutedIconTint()              // 弱化图标色调

// 来自 LegadoTheme.kt - 应包装内容
LegadoTheme { content() }
```

### 3. 检查可复用组件

查找以下组件的导入和使用：

**卡片组件：**
```kotlin
import io.legado.app.ui.widget.components.card.GlassCard
import io.legado.app.ui.widget.components.card.TextCard

// 使用方式
GlassCard(modifier, containerColor, cornerRadius) { ... }
TextCard(text, backgroundColor, contentColor)
```

**模态组件：**
```kotlin
import io.legado.app.ui.widget.components.modalBottomSheet.AppModalBottomSheet

// 使用方式
AppModalBottomSheet(show, onDismissRequest, title) { ... }
```

**列表组件：**
```kotlin
import io.legado.app.ui.widget.components.list.TopFloatingStickyItem

// 使用方式
TopFloatingStickyItem(item) { item -> ... }
```

**复选框组件：**
```kotlin
import io.legado.app.ui.widget.components.checkBox.CheckboxItem

// 使用方式
CheckboxItem(title, checked, onCheckedChange)
```

### 4. 检查 MaterialTheme 使用

查找以下用法：
```kotlin
MaterialTheme.colorScheme.primary
MaterialTheme.colorScheme.surface
MaterialTheme.colorScheme.onSurface
MaterialTheme.typography.bodyMedium
```

这些应该尽可能替代原始的 `Color` 对象使用。

## 输出格式

分析完成后，提供：

### 总结部分
```
## 分析结果

### ✅ 已正确使用
- [正确使用的组件/函数列表]

### ❌ 未使用/可优化
- [缺失的可复用组件或硬编码值列表]

### 🔧 建议修改
```

### 详细建议

针对每个发现的问题，提供：
1. **当前代码模式**（正在使用的替代方案）
2. **推荐修改**（应使用哪个可复用组件/函数）
3. **预期收益**（一致性、可维护性等）

### 迁移模板

如果需要修改，提供具体的迁移计划：

```
## 迁移计划

### 1. 添加必要的导入
```kotlin
import io.legado.app.ui.theme.pageTopBarContainerColor
import io.legado.app.ui.widget.components.card.GlassCard
// ... 其他导入
```

### 2. 替换硬编码颜色
**修改前：**
```kotlin
backgroundColor = Color(0xFFE0E0E0)
```
**修改后：**
```kotlin
backgroundColor = pageCardContainerColor()
```

### 3. 替换自定义组件
**修改前：**
```kotlin
Card(modifier, colors = CardDefaults.cardColors(...)) { ... }
```
**修改后：**
```kotlin
GlassCard(modifier) { ... }
```

### 4. 包装主题
**修改前：**
```kotlin
@Composable
fun MyScreen() {
    // ...
}
```
**修改后：**
```kotlin
@Composable
fun MyScreen() {
    LegadoTheme {
        // ...
    }
}
```
```

## 示例

### 示例 1：良好用法

正确使用主题系统的文件：

```kotlin
import io.legado.app.ui.theme.LegadoTheme
import io.legado.app.ui.theme.pageTopBarContainerColor
import io.legado.app.ui.theme.pageCardContainerColor
import io.legado.app.ui.widget.components.card.GlassCard

@Composable
fun GoodScreen() {
    LegadoTheme {
        val topBarColor = pageTopBarContainerColor()
        val cardColor = pageCardContainerColor()

        GlassCard(containerColor = cardColor) {
            Text("Content")
        }
    }
}
```

### 示例 2：需要改进

使用硬编码颜色的文件：

```kotlin
// ❌ 缺少导入
// ❌ 硬编码背景颜色
// ❌ 缺少 LegadoTheme 包装

@Composable
fun BadScreen() {
    Column(modifier = Modifier.background(Color(0xFFF5F5F5))) {
        Card(modifier = Modifier.background(Color.White)) {
            Text("Content")
        }
    }
}
```

## 实现注意事项

- 页面级样式始终使用 `CommonPageColors.kt` 中的 `pageTopBarContainerColor()` 等函数
- 适当创建页面专属颜色封装（如 `ReadRecordColors.kt` 中的做法）
- 优先使用 `GlassCard` 而不是原始 `Card` 以实现玻璃效果
- 始终用 `LegadoTheme` 包装内容以保持主题一致性
- 使用 `MaterialTheme.colorScheme.*` 替代硬编码颜色来处理 Material 组件