---
name: "feature-architecture-planning"
description: "Comprehensive architecture planning for new features. Includes architecture design, tech stack selection, file structure planning, implementation roadmap, and risk assessment. Invoke when adding new features, modules, or when user asks for architecture planning."
---

# 功能架构规划

## 概述

这是一个完整的功能架构规划流程，用于在开发新功能前进行系统性的设计和规划。确保功能开发有清晰的架构、合理的技术选型、明确的实施计划和可控的风险。

## 触发条件

当出现以下情况时调用此 Skill：
- 用户要求添加新功能
- 用户要求开发新模块
- 用户询问"如何实现某功能"
- 用户要求"规划一下架构"
- 用户要求"设计一下方案"
- 用户要求"评估一下技术方案"

## 规划流程

### 第一步：需求分析与现状调研

#### 1.1 理解需求
- 明确功能目标和核心需求
- 识别功能边界和约束条件
- 确定优先级和里程碑

#### 1.2 调研现状
- 搜索项目中相关的现有代码
- 了解项目的技术栈和架构模式
- 分析类似功能的实现方式
- 识别可复用的组件和模块

**工具使用：**
```kotlin
// 搜索相关代码
SearchCodebase("关键词")
Grep("pattern")
Glob("pattern")

// 阅读关键文件
Read("关键文件路径")

// 查看项目结构
LS("目录路径")
```

---

### 第二步：架构设计

#### 2.1 整体架构图

绘制清晰的分层架构图，包含：

```
┌─────────────────────────────────────────────────────────────┐
│                      表现层                  │
├─────────────────────────────────────────────────────────────┤
│  Activity/Fragment/Screen                                    │
│  └── UI组件                                                  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      业务层                  │
├─────────────────────────────────────────────────────────────┤
│  ViewModel                                                   │
│  ├── 状态管理                                                │
│  └── 业务逻辑                                                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      数据层                    │
├─────────────────────────────────────────────────────────────┤
│  Repository                                                  │
│  ├── Dao (数据访问对象)                                      │
│  └── DataSource (数据源)                                     │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      数据存储                   │
├─────────────────────────────────────────────────────────────┤
│  Room Database / SharedPreferences / File / Network         │
└─────────────────────────────────────────────────────────────┘
```

#### 2.2 模块划分

- **表现层**：UI展示、用户交互
- **业务层**：业务逻辑、状态管理
- **数据层**：数据访问、缓存策略
- **基础设施层**：工具类、扩展函数

#### 2.3 依赖关系

明确各层之间的依赖方向（单向依赖，避免循环依赖）

---

### 第三步：技术栈选择

#### 3.1 核心技术评估

| 技术 | 用途 | 选择理由 | 是否已有 |
|------|------|----------|----------|
| 技术A | 用途A | 理由A | ✅/❌ |
| 技术B | 用途B | 理由B | ✅/❌ |

#### 3.2 新增依赖

```gradle
// 需要添加的依赖
implementation 'xxx:xxx:version'
```

#### 3.3 技术选型原则

1. **优先使用项目已有技术**：降低学习成本，保持一致性
2. **选择成熟稳定的技术**：避免踩坑，降低风险
3. **考虑团队技术栈**：便于维护和协作
4. **评估性能影响**：避免引入性能瓶颈

---

### 第四步：数据模型设计

#### 4.1 实体类设计

```kotlin
@Entity(tableName = "table_name")
data class EntityName(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    
    // 业务字段
    val field1: String,
    val field2: Int,
    
    // 管理字段
    val createTime: Long = System.currentTimeMillis(),
    val updateTime: Long = System.currentTimeMillis()
)
```

#### 4.2 数据访问对象（Dao）

```kotlin
@Dao
interface EntityDao {
    @Query("SELECT * FROM table_name")
    fun flowAll(): Flow<List<EntityName>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(entity: EntityName)
    
    @Delete
    suspend fun delete(entity: EntityName)
}
```

#### 4.3 数据库迁移

```kotlin
val MIGRATION_X_Y = object : Migration(X, Y) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // 创建表
        database.execSQL("""
            CREATE TABLE IF NOT EXISTS table_name (...)
        """)
        
        // 创建索引
        database.execSQL("CREATE INDEX IF NOT EXISTS ...")
    }
}
```

---

### 第五步：文件结构规划

#### 5.1 需要新增的文件

```
项目根目录/
├── 数据层/
│   ├── entities/
│   │   └── NewEntity.kt                    ✨ 新增
│   └── dao/
│       └── NewDao.kt                       ✨ 新增
│
├── 业务层/
│   └── NewViewModel.kt                     ✨ 新增
│
└── 表现层/
    ├── NewActivity.kt                      ✨ 新增
    ├── NewScreen.kt                        ✨ 新增
    └── components/
        └── NewComponent.kt                 ✨ 新增
```

#### 5.2 需要修改的文件

```
项目根目录/
├── 配置文件/
│   └── AppDatabase.kt                      📝 修改
│
└── 入口文件/
    └── MainActivity.kt                     📝 修改
```

#### 5.3 需要删除的文件

```
项目根目录/
└── 旧文件/
    └── OldFile.kt                          ❌ 删除
```

---

### 第六步：核心代码实现要点

#### 6.1 ViewModel 设计

```kotlin
class FeatureViewModel(
    application: Application
) : BaseViewModel(application) {
    
    // UI状态
    private val _uiState = MutableStateFlow<UiState>(UiState.Idle)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    // 数据流
    val data: Flow<List<Entity>> = repository.getData()
    
    // 业务方法
    fun doSomething() {
        execute {
            // 业务逻辑
        }.onSuccess {
            // 成功处理
        }.onError {
            // 错误处理
        }
    }
    
    sealed class UiState {
        object Idle : UiState()
        object Loading : UiState()
        data class Error(val message: String) : UiState()
    }
}
```

#### 6.2 Screen/Fragment 设计

```kotlin
@Composable
fun FeatureScreen(
    viewModel: FeatureViewModel = viewModel(),
    onBackClick: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()
    val data by viewModel.data.collectAsState(initial = emptyList())
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("功能标题") },
                navigationIcon = {
                    IconButton(onClick = onBackClick) {
                        Icon(Icons.Default.ArrowBack, "返回")
                    }
                }
            )
        }
    ) { paddingValues ->
        when (uiState) {
            is UiState.Loading -> LoadingComponent()
            is UiState.Error -> ErrorComponent((uiState as UiState.Error).message)
            else -> ContentComponent(data)
        }
    }
}
```

#### 6.3 Repository 设计

```kotlin
class FeatureRepository {
    private val dao = appDb.entityDao()
    
    fun getData(): Flow<List<Entity>> = dao.flowAll()
    
    suspend fun saveData(entity: Entity) {
        dao.insert(entity)
    }
    
    suspend fun deleteData(entity: Entity) {
        dao.delete(entity)
    }
}
```

---

### 第七步：实施计划

#### 阶段一：数据层搭建（X天）

**任务清单：**
- [ ] 创建实体类
- [ ] 创建Dao接口
- [ ] 修改AppDatabase
- [ ] 编写数据库迁移脚本
- [ ] 创建Repository
- [ ] 编写单元测试

**关键文件：**
- Entity.kt
- Dao.kt
- Repository.kt

**验收标准：**
- 数据库表创建成功
- CRUD操作正常
- 单元测试通过

---

#### 阶段二：业务层实现（X天）

**任务清单：**
- [ ] 创建ViewModel
- [ ] 实现状态管理
- [ ] 实现业务逻辑
- [ ] 处理异常情况
- [ ] 编写单元测试

**关键文件：**
- ViewModel.kt

**验收标准：**
- 状态流转正确
- 业务逻辑完整
- 异常处理完善

---

#### 阶段三：UI层开发（X天）

**任务清单：**
- [ ] 创建Activity/Screen
- [ ] 实现UI布局
- [ ] 实现交互逻辑
- [ ] 添加动画效果
- [ ] 适配深色模式
- [ ] UI测试

**关键文件：**
- Activity.kt
- Screen.kt
- Component.kt

**验收标准：**
- UI显示正确
- 交互流畅
- 适配各种屏幕尺寸

---

#### 阶段四：集成与测试（X天）

**任务清单：**
- [ ] 修改入口跳转
- [ ] 删除旧代码
- [ ] 数据迁移
- [ ] 功能测试
- [ ] 性能测试
- [ ] 回归测试

**关键文件：**
- 入口文件修改
- 旧文件删除

**验收标准：**
- 功能完整可用
- 无明显性能问题
- 无回归bug

---

### 第八步：数据迁移策略

#### 8.1 旧数据迁移

```kotlin
object DataMigration {
    fun migrateOldData() {
        // 读取旧数据
        val oldData = getOldData()
        
        // 转换为新格式
        val newData = oldData.map { 
            NewEntity(
                field1 = it.oldField1,
                field2 = it.oldField2
            )
        }
        
        // 保存到新存储
        appDb.entityDao().insert(*newData.toTypedArray())
        
        // 清理旧数据
        clearOldData()
    }
}
```

#### 8.2 默认数据导入

```kotlin
fun importDefaultData() {
    val defaultData = loadDefaultData()
    appDb.entityDao().insert(*defaultData.toTypedArray())
}
```

---

### 第九步：风险与应对

| 风险 | 影响程度 | 发生概率 | 应对措施 |
|------|----------|----------|----------|
| 风险1 | 高/中/低 | 高/中/低 | 具体措施 |
| 风险2 | 高/中/低 | 高/中/低 | 具体措施 |

**常见风险：**

1. **数据库版本冲突**
   - 影响：高
   - 应对：使用fallbackToDestructiveMigrationFrom，提供完整迁移脚本

2. **数据丢失**
   - 影响：高
   - 应对：实现数据迁移工具，首次启动时自动迁移

3. **性能问题**
   - 影响：中
   - 应对：使用Flow响应式更新，避免频繁查询

4. **兼容性问题**
   - 影响：中
   - 应对：充分测试各种场景，提供降级方案

---

### 第十步：预期成果

#### 10.1 功能成果

✅ **核心功能**
- 功能点1
- 功能点2
- 功能点3

✅ **性能指标**
- 启动时间 < X秒
- 内存占用 < XMB
- 响应时间 < X毫秒

#### 10.2 代码成果

- 📁 新增文件：**X个**
- 📝 修改文件：**X个**
- ❌ 删除文件：**X个**
- 📊 代码行数预估：**约X行**

#### 10.3 文档成果

- 功能说明文档
- 架构设计文档
- API文档
- 使用指南

---

### 第十一步：后续扩展方向

#### 11.1 短期扩展（1-2周）

- [ ] 扩展功能1
- [ ] 扩展功能2

#### 11.2 中期扩展（1-2月）

- [ ] 扩展功能3
- [ ] 扩展功能4

#### 11.3 长期扩展（3-6月）

- [ ] 扩展功能5
- [ ] 扩展功能6

---

## 验证清单

### 架构验证

- [ ] 分层是否清晰？
- [ ] 职责是否单一？
- [ ] 依赖是否单向？
- [ ] 扩展性如何？

### 技术验证

- [ ] 技术选型是否合理？
- [ ] 是否有更好的替代方案？
- [ ] 是否引入不必要的依赖？
- [ ] 性能是否满足要求？

### 实现验证

- [ ] 文件结构是否合理？
- [ ] 命名是否规范？
- [ ] 代码是否可测试？
- [ ] 是否易于维护？

### 测试验证

- [ ] 单元测试覆盖是否充分？
- [ ] 集成测试是否完整？
- [ ] UI测试是否全面？
- [ ] 性能测试是否通过？

### 文档验证

- [ ] 架构文档是否完整？
- [ ] API文档是否清晰？
- [ ] 使用文档是否易懂？
- [ ] 注释是否充分？

---

## 输出模板

```markdown
## 📐 [功能名称] 架构规划

### 一、架构设计

#### 1.1 整体架构图
[架构图]

#### 1.2 模块划分
[模块说明]

---

### 二、技术栈选择

#### 2.1 核心技术
[技术表格]

#### 2.2 新增依赖
[依赖列表]

---

### 三、数据模型设计

#### 3.1 实体类
[实体类定义]

#### 3.2 数据访问对象
[Dao定义]

---

### 四、文件结构规划

#### 4.1 新增文件
[文件树]

#### 4.2 修改文件
[文件列表]

#### 4.3 删除文件
[文件列表]

---

### 五、核心代码实现要点

#### 5.1 ViewModel设计
[代码示例]

#### 5.2 Screen设计
[代码示例]

---

### 六、实施计划

#### 阶段一：数据层搭建（X天）
[任务清单]

#### 阶段二：业务层实现（X天）
[任务清单]

#### 阶段三：UI层开发（X天）
[任务清单]

#### 阶段四：集成与测试（X天）
[任务清单]

---

### 七、数据迁移策略

#### 7.1 旧数据迁移
[迁移代码]

#### 7.2 默认数据导入
[导入代码]

---

### 八、风险与应对

| 风险 | 影响 | 应对措施 |
|------|------|----------|
| ... | ... | ... |

---

### 九、预期成果

#### 9.1 功能成果
[功能列表]

#### 9.2 代码成果
[统计数据]

---

### 十、后续扩展方向

#### 10.1 短期扩展
[扩展计划]

#### 10.2 中期扩展
[扩展计划]

#### 10.3 长期扩展
[扩展计划]

---

## 📋 总结

[总结说明]

**预计总开发时间：X-Y天**
```

---

## 注意事项

1. **充分调研**：在规划前必须充分调研现有代码，避免重复造轮子
2. **遵循规范**：遵循项目现有的架构规范和编码风格
3. **渐进式开发**：分阶段实施，每个阶段都要有可验证的成果
4. **风险意识**：提前识别风险，准备好应对措施
5. **文档先行**：先设计后编码，文档和代码同步更新
6. **可测试性**：设计时考虑测试，确保代码可测试
7. **性能优先**：关注性能指标，避免性能瓶颈
8. **用户体验**：从用户角度思考，提供良好的用户体验

---

## 示例参考

参考实际案例：直链上传配置功能架构规划

该案例完整展示了：
- 从需求分析到架构设计的完整流程
- 数据层、业务层、UI层的详细规划
- 技术选型和风险评估
- 分阶段实施计划
- 数据迁移策略
- 预期成果和后续扩展

---

## 快速使用

当用户要求规划新功能时，按照以下步骤快速输出：

1. **第一步**：调研现状（使用搜索工具）
2. **第二步**：设计架构（绘制架构图）
3. **第三步**：选择技术（评估技术栈）
4. **第四步**：设计数据模型（定义实体和Dao）
5. **第五步**：规划文件结构（列出新增/修改/删除文件）
6. **第六步**：编写核心代码要点（ViewModel、Screen等）
7. **第七步**：制定实施计划（分阶段任务）
8. **第八步**：设计迁移策略（数据迁移方案）
9. **第九步**：评估风险（风险列表和应对）
10. **第十步**：定义预期成果（功能和代码成果）
11. **第十一步**：规划扩展方向（短中长期）

输出完整的架构规划文档，供用户确认后开始实施。
