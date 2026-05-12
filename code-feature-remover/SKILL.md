---
name: "code-feature-remover"
description: "安全删除代码功能的完整流程。当用户要求删除某部分功能、移除特性、清理无用代码时调用此技能。"
---

# 代码功能删除流程

安全删除代码功能需要系统性地分析和处理，避免遗漏或引入错误。

## 删除前的检查清单

### 1. 功能依赖分析

删除前需要识别所有相关代码：

```
功能依赖树：
├── 状态变量
│   └── remember { mutableStateOf() }
├── UI组件
│   ├── 按钮/输入框/对话框
│   └── 条件渲染块 (if/when)
├── 事件处理
│   └── onClick/onChange 回调
├── 辅助函数
│   └── 被删除功能调用的函数
├── 导入语句
│   └── 仅被此功能使用的导入
└── 资源文件
    └── 字符串/颜色/布局等
```

### 2. 删除顺序

**从叶子节点到根节点**（自底向上）：

1. **UI组件** → 2. **事件处理** → 3. **状态变量** → 4. **辅助函数** → 5. **导入语句** → 6. **资源文件**

## 删除流程

### Step 1: 标记目标代码

```bash
# 使用注释标记要删除的代码
// TODO: DELETE - 搜索功能开始
...要删除的代码...
// TODO: DELETE - 搜索功能结束
```

### Step 2: 删除UI组件

```kotlin
// ❌ 删除条件渲染块
if (showSearch) {
    OutlinedTextField(...)
}

// ❌ 删除触发按钮
IconButton(onClick = { showSearch = !showSearch }) {
    Icon(Icons.Default.Search, ...)
}
```

### Step 3: 删除事件处理

```kotlin
// ❌ 删除回调中的相关逻辑
Button(onClick = {
    // 删除这部分
    if (searchQuery.isNotEmpty()) { ... }
})
```

### Step 4: 删除状态变量

```kotlin
// ❌ 删除 remember 状态
var searchQuery by remember { mutableStateOf("") }
var showSearch by remember { mutableStateOf(false) }
```

### Step 5: 删除辅助函数

```kotlin
// ❌ 删除仅被此功能使用的函数
private fun highlightText(query: String) { ... }
```

### Step 6: 清理导入语句

```kotlin
// ❌ 删除不再使用的导入
import androidx.compose.ui.text.AnnotatedString
import androidx.compose.ui.text.SpanStyle
```

### Step 7: 清理资源文件（可选）

```xml
<!-- ❌ 删除不再使用的字符串资源 -->
<string name="search_hint">搜索...</string>
```

## 验证清单

删除完成后，执行以下验证：

| 验证项 | 命令/方法 |
|--------|----------|
| 编译通过 | `./gradlew compileDebugKotlin` |
| 无未使用导入 | IDE 警告检查 |
| 无未使用变量 | IDE 警告检查 |
| 功能完整 | 运行应用测试相关流程 |

## 常见问题处理

### 1. 状态变量被多处引用

```kotlin
// 如果状态被多处使用，检查是否可以安全删除
// 使用 Grep 搜索变量名
Grep pattern: "searchQuery"
```

### 2. 导入语句不确定是否使用

```kotlin
// 删除后编译，如果报错则恢复
// 或使用 IDE 的 "Optimize Imports" 功能
```

### 3. 函数被其他地方调用

```kotlin
// 使用 Grep 搜索函数名确认
Grep pattern: "highlightText"
```

## 删除模板

```markdown
## 删除 [功能名称]

### 删除内容清单
- [ ] UI组件: [列出组件]
- [ ] 状态变量: [列出变量]
- [ ] 事件处理: [列出回调]
- [ ] 辅助函数: [列出函数]
- [ ] 导入语句: [列出导入]
- [ ] 资源文件: [列出资源]

### 验证结果
- [ ] 编译通过
- [ ] 无警告
- [ ] 功能正常
```

## 示例：删除搜索功能

```kotlin
// === 删除前 ===
var searchQuery by remember { mutableStateOf("") }
var showSearch by remember { mutableStateOf(false) }

IconButton(onClick = { showSearch = !showSearch }) {
    Icon(Icons.Default.Search, contentDescription = "搜索")
}

if (showSearch) {
    OutlinedTextField(
        value = searchQuery,
        onValueChange = { searchQuery = it },
        ...
    )
}

val annotatedText = if (searchQuery.isNotEmpty()) {
    buildAnnotatedString { ... }
} else {
    AnnotatedString(text)
}

// === 删除后 ===
// 状态变量已删除
// 搜索按钮已删除
// 搜索框已删除
// 高亮逻辑已删除

Text(text = text)  // 直接显示文本
```

## 输出要求

删除完成后，提供以下信息：

1. **删除清单** - 列出所有删除的代码
2. **代码对比** - 关键代码的前后对比
3. **验证结果** - 编译是否通过
4. **影响范围** - 是否影响其他功能
