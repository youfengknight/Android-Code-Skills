---
name: dev-learning-record
description: "开发学习记录生成器。每次添加新功能或修改代码后，从Kotlin语法、Android架构、UI开发、数据库等角度分析学习点，生成结构化的学习文档。适用于代码学习、知识总结、开发复盘场景。"
---

# 开发学习记录生成器

帮助Kotlin和Android开发初学者理解每次代码修改背后的知识点。

## 触发场景

- 用户说"记录学习点"、"总结一下学到了什么"
- 完成一个功能开发后
- 用户想了解某段代码的知识点
- 代码review后需要总结

## 分析维度

### 1. Kotlin语法层面

分析涉及的Kotlin语法知识：

| 语法特性 | 说明 | 示例 |
|---------|------|------|
| 数据类 | `data class` 自动生成equals/hashCode/toString | `data class UrlRecord(...)` |
| 密封类 | `sealed class` 限制继承层次，用于状态管理 | `sealed class UIState` |
| 扩展函数 | 为现有类添加新方法 | `fun Context.toast()` |
| 高阶函数 | 函数作为参数或返回值 | `collect { }` |
| 协程 | 异步编程 | `viewModelScope.launch { }` |
| Flow | 响应式数据流 | `flowAll(): Flow<List<T>>` |
| 空安全 | `?`、`?.`、`?:`、`!!` 操作符 | `record.sourceName?.let { }` |
| 属性委托 | `by lazy`、`by viewModels` | `val viewModel by viewModels()` |
| 单例对象 | `object` 声明单例 | `object UrlRecordInterceptor` |

### 2. Android架构层面

分析架构模式和组件使用：

| 架构概念 | 说明 | 本项目应用 |
|---------|------|-----------|
| MVVM | Model-View-ViewModel架构 | Activity观察ViewModel状态 |
| 单向数据流 | StateFlow驱动UI | `_uiState -> uiState` |
| Repository模式 | 数据访问抽象层 | `appDb.urlRecordDao` |
| 依赖注入 | Hilt/Koin管理依赖 | `by viewModels()` |
| 生命周期感知 | Lifecycle-aware组件 | `lifecycleScope.launch` |

### 3. UI开发层面

分析界面开发技术：

| UI技术 | 说明 | 示例 |
|--------|------|------|
| ViewBinding | 视图绑定，替代findViewById | `ActivityUrlRecordBinding` |
| RecyclerView | 列表展示 | `UrlRecordAdapter` |
| SearchView | 搜索框组件 | `OnQueryTextListener` |
| Menu | 菜单系统 | `onCreateOptionsMenu` |
| Dialog | 对话框 | `alert { }` |

### 4. 数据库层面

分析Room数据库使用：

| Room组件 | 说明 | 示例 |
|---------|------|------|
| Entity | 实体类，对应表 | `@Entity tableName = "url_records"` |
| DAO | 数据访问对象 | `@Dao interface UrlRecordDao` |
| Database | 数据库类 | `@Database version = 93` |
| Migration | 数据库迁移 | `AutoMigration 92→93` |
| Flow查询 | 响应式查询 | `fun flowAll(): Flow<List<T>>` |

### 5. 网络层面

分析网络请求相关：

| 网络组件 | 说明 | 示例 |
|---------|------|------|
| OkHttp | HTTP客户端 | `OkHttpClient` |
| Interceptor | 拦截器 | `UrlRecordInterceptor` |
| 协程异步 | 非阻塞请求 | `scope.launch { }` |

## 输出格式

```markdown
# 开发学习记录 - [功能名称]

## 📚 Kotlin语法学习

### [语法点1]
**代码示例**：
```kotlin
// 代码片段
```

**知识点**：
- 要点1
- 要点2

**为什么这样写**：
解释原因

---

## 🏗️ Android架构学习

### [架构点1]
...

---

## 🎨 UI开发学习

### [UI点1]
...

---

## 💾 数据库学习

### [数据库点1]
...

---

## 🔗 知识关联

| 本次学习 | 相关知识 | 扩展阅读 |
|---------|---------|---------|
| ... | ... | ... |

---

## 💡 实践建议

1. 建议1
2. 建议2
```

## 示例：URL访问记录功能学习记录

### Kotlin语法学习

#### 1. sealed class（密封类）
```kotlin
sealed class UrlRecordUIState {
    object Loading : UrlRecordUIState()
    data class Success(val records: List<UrlRecord>) : UrlRecordUIState()
    data class Error(val message: String) : UrlRecordUIState()
    object Empty : UrlRecordUIState()
}
```

**知识点**：
- 密封类限制继承只能在同一文件中
- 配合`when`表达式可以穷举所有状态
- 编译器会检查是否覆盖所有情况

**为什么用密封类**：
UI状态是有限的几种，用密封类可以确保类型安全，避免遗漏状态处理。

#### 2. StateFlow（状态流）
```kotlin
private val _uiState = MutableStateFlow<UrlRecordUIState>(UrlRecordUIState.Loading)
val uiState: StateFlow<UrlRecordUIState> = _uiState.asStateFlow()
```

**知识点**：
- `MutableStateFlow`是可变的状态流
- `asStateFlow()`暴露为只读的`StateFlow`
- 值变化时会自动通知所有观察者

**为什么这样写**：
遵循单向数据流原则，ViewModel内部可修改，外部只能观察。

#### 3. Flow（数据流）
```kotlin
@Query("SELECT * FROM url_records ORDER BY timestamp DESC")
fun flowAll(): Flow<List<UrlRecord>>
```

**知识点**：
- Room支持返回`Flow`类型
- 数据库数据变化时自动发射新值
- 配合`collect`持续观察数据变化

**与List的区别**：
- `List`：一次性查询，数据变化不会更新
- `Flow`：持续观察，数据变化自动更新

### Android架构学习

#### 1. MVVM模式
```
View (Activity) ← 观察 ← ViewModel ← 查询 ← Model (DAO/Database)
       │                      │
       └──── 用户事件 ────────┘
```

**职责划分**：
- **Activity**：只负责UI渲染和用户交互
- **ViewModel**：管理UI状态，处理业务逻辑
- **Model**：数据存储和查询

#### 2. 生命周期感知
```kotlin
lifecycleScope.launch {
    viewModel.uiState.collectLatest { state ->
        // 更新UI
    }
}
```

**知识点**：
- `lifecycleScope`绑定Activity生命周期
- Activity销毁时自动取消协程
- 避免内存泄漏

### 数据库学习

#### 1. Entity定义
```kotlin
@Entity(tableName = "url_records", indices = [Index("timestamp"), Index("domain")])
data class UrlRecord(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val url: String,
    val domain: String,
    // ...
)
```

**知识点**：
- `@Entity`定义表名
- `@PrimaryKey`定义主键
- `indices`创建索引加速查询

#### 2. 数据库迁移
```kotlin
@Database(
    version = 93,
    entities = [..., UrlRecord::class],
    autoMigrations = [..., AutoMigration(from = 92, to = 93)]
)
```

**知识点**：
- 数据库版本号变化需要迁移
- `AutoMigration`自动处理简单迁移
- 新增表可以自动迁移

### UI开发学习

#### 1. ViewBinding使用
```kotlin
override val binding by viewBinding(ActivityUrlRecordBinding::inflate)
```

**知识点**：
- 替代`findViewById`
- 类型安全
- 自动生成绑定类

#### 2. SearchView集成
```kotlin
class UrlRecordActivity : ..., SearchView.OnQueryTextListener {
    override fun onQueryTextChange(newText: String?): Boolean {
        viewModel.setSearchQuery(newText)
        return false
    }
}
```

**知识点**：
- 实现`OnQueryTextListener`接口
- `onQueryTextChange`输入时触发
- 返回`false`表示不消费事件

## 使用方法

1. 完成功能开发后，调用此skill
2. 提供修改的文件列表或功能描述
3. 生成结构化的学习记录
4. 保存到项目的学习文档目录
