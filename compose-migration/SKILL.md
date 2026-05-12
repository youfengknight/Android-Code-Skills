---
name: compose-migration
description: 将Android View系统界面重构为Jetpack Compose框架。当用户要求将某个界面改为Compose、重构界面、或提到Compose迁移时调用此技能。
---

# View系统重构为Jetpack Compose指南

本技能提供将传统View系统（XML布局 + RecyclerView + Adapter）重构为Jetpack Compose的完整流程。

## 适用场景

- 用户说"把XX界面改成Compose"
- 用户说"重构XX界面"
- 用户说"用Compose重写XX"
- 需要将XML布局转换为Kotlin代码

## 重构前检查清单

### 1. 分析目标界面

```
需要收集的信息：
├── Activity/Fragment文件路径
├── XML布局文件路径
├── Adapter文件路径（如果有列表）
├── ViewModel文件路径
├── 是否有菜单
├── 是否有对话框
└── 是否有搜索功能
```

### 2. 检查项目Compose配置

```bash
# 检查build.gradle是否有Compose依赖
grep -n "compose" build.gradle
grep -n "compose" libs.versions.toml

# 检查是否已有Compose主题
find . -name "*Theme*.kt" | head -5
```

## 重构步骤

### Step 1: 创建Screen组件文件

文件命名规范：`XxxScreen.kt`

```kotlin
/**
 * XXX界面 - Jetpack Compose实现
 */
package io.legado.app.ui.xxx

// 基础导入
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun XxxScreen(
    viewModel: XxxViewModel = viewModel(),
    onBackClick: () -> Unit
) {
    // 状态管理
    val uiState by viewModel.uiState.collectAsState()
    
    // 页面骨架
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("标题") },
                navigationIcon = {
                    IconButton(onClick = onBackClick) {
                        Icon(Icons.Default.ArrowBack, "返回")
                    }
                },
                actions = {
                    // 菜单按钮
                }
            )
        }
    ) { paddingValues ->
        // 主内容
        Column(modifier = Modifier.padding(paddingValues)) {
            // UI内容
        }
    }
}
```

### Step 2: 修改Activity使用setContent

```kotlin
class XxxActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        // 初始化主题
        initTheme()
        super.onCreate(savedInstanceState)
        
        // 使用setContent替代setContentView
        setContent {
            XxxContent(
                onBackClick = { finish() }
            )
        }
    }
}
```

### Step 3: 状态管理改造

**ViewModel改造：**

```kotlin
class XxxViewModel : BaseViewModel() {
    // 将普通属性改为StateFlow
    private val _uiState = MutableStateFlow<XxxUIState>(XxxUIState.Loading)
    val uiState: StateFlow<XxxUIState> = _uiState.asStateFlow()
    
    // 如果有开关类状态，也要用StateFlow
    private val _isEnabled = MutableStateFlow(AppConfig.isEnabled)
    val isEnabled: StateFlow<Boolean> = _isEnabled.asStateFlow()
    
    fun setEnabled(enabled: Boolean) {
        AppConfig.isEnabled = enabled
        _isEnabled.value = enabled  // 更新StateFlow
    }
}
```

**UIState定义：**

```kotlin
sealed class XxxUIState {
    object Loading : XxxUIState()
    data class Success(val data: List<Item>) : XxxUIState()
    data class Error(val message: String) : XxxUIState()
    object Empty : XxxUIState()
}
```

### Step 4: 列表迁移

**RecyclerView → LazyColumn：**

```kotlin
// ❌ 旧代码：RecyclerView + Adapter
recyclerView.adapter = MyAdapter(items)

// ✅ 新代码：LazyColumn
LazyColumn(
    modifier = Modifier.fillMaxSize(),
    contentPadding = PaddingValues(vertical = 8.dp)
) {
    items(items, key = { it.id }) { item ->
        ItemCard(item = item)
    }
}
```

**列表项组件：**

```kotlin
@Composable
private fun ItemCard(item: Item) {
    Surface(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 12.dp, vertical = 4.dp),
        shape = RoundedCornerShape(12.dp)
    ) {
        Column(modifier = Modifier.padding(12.dp)) {
            Text(text = item.title, style = MaterialTheme.typography.titleMedium)
            Text(text = item.subtitle, style = MaterialTheme.typography.bodySmall)
        }
    }
}
```

### Step 5: 菜单迁移

**onCreateOptionsMenu → DropdownMenu：**

```kotlin
// ❌ 旧代码
override fun onCreateOptionsMenu(menu: Menu): Boolean {
    menuInflater.inflate(R.menu.xxx, menu)
    return true
}

// ✅ 新代码
var showMenu by remember { mutableStateOf(false) }

IconButton(onClick = { showMenu = true }) {
    Icon(Icons.Default.MoreVert, "更多")
}

DropdownMenu(
    expanded = showMenu,
    onDismissRequest = { showMenu = false }
) {
    DropdownMenuItem(
        text = { Text("选项1") },
        onClick = { /* 处理点击 */ }
    )
}
```

### Step 6: 对话框迁移

**AlertDialog.Builder → AlertDialog组件：**

```kotlin
// ❌ 旧代码
AlertDialog.Builder(this)
    .setTitle("标题")
    .setMessage("内容")
    .setPositiveButton("确定") { _, _ -> }
    .show()

// ✅ 新代码
var showDialog by remember { mutableStateOf(false) }

if (showDialog) {
    AlertDialog(
        onDismissRequest = { showDialog = false },
        title = { Text("标题") },
        text = { Text("内容") },
        confirmButton = {
            TextButton(onClick = { showDialog = false }) {
                Text("确定")
            }
        }
    )
}
```

### Step 7: 搜索功能迁移

**SearchView → OutlinedTextField：**

```kotlin
// ❌ 旧代码
searchView.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
    override fun onQueryTextChange(newText: String?): Boolean {
        viewModel.search(newText)
        return true
    }
})

// ✅ 新代码
var searchQuery by remember { mutableStateOf("") }

OutlinedTextField(
    value = searchQuery,
    onValueChange = { 
        searchQuery = it
        viewModel.search(it.ifBlank { null })
    },
    placeholder = { Text("搜索...") },
    leadingIcon = { Icon(Icons.Default.Search, null) }
)
```

### Step 8: 删除旧文件

重构完成后删除：

```
需要删除的文件：
├── res/layout/activity_xxx.xml      # Activity布局
├── res/layout/item_xxx.xml          # 列表项布局
├── res/menu/xxx.xml                 # 菜单文件（可选保留）
└── XxxAdapter.kt                    # RecyclerView适配器
```

## 重构后验证清单

| 检查项 | 命令/方法 |
|--------|----------|
| Activity使用setContent | `grep "setContent" XxxActivity.kt` |
| 无XML布局文件 | `find . -name "activity_xxx.xml"` |
| 无Adapter文件 | `find . -name "*Adapter*.kt"` |
| 组件有@Composable | `grep "@Composable" XxxScreen.kt` |
| 列表用LazyColumn | `grep "LazyColumn" XxxScreen.kt` |
| 状态用collectAsState | `grep "collectAsState" XxxScreen.kt` |
| 编译通过 | `./gradlew compileDebugKotlin` |

## 常见问题处理

### 1. suspend函数调用

```kotlin
// ❌ 错误：直接调用suspend函数
onClick = { viewModel.clearAll() }

// ✅ 正确：在协程中调用
val coroutineScope = rememberCoroutineScope()

onClick = {
    coroutineScope.launch {
        viewModel.clearAll()
    }
}
```

### 2. 状态不更新

```kotlin
// ❌ 错误：普通函数获取，不会更新
val isEnabled = viewModel.isEnabled()

// ✅ 正确：使用collectAsState
val isEnabled by viewModel.isEnabled.collectAsState()
```

### 3. Context获取

```kotlin
// 在Composable中获取Context
val context = LocalContext.current

// 获取Application
val application = context.applicationContext as Application
```

### 4. 资源获取

```kotlin
// 字符串资源
stringResource(R.string.xxx)

// 颜色资源
colorResource(R.color.xxx)

// 尺寸资源
dimensionResource(R.dimen.xxx)
```

## 代码模板

### 完整的Screen模板

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun XxxScreen(
    viewModel: XxxViewModel = viewModel(),
    onBackClick: () -> Unit
) {
    // 状态
    val uiState by viewModel.uiState.collectAsState()
    val items by viewModel.items.collectAsState()
    
    // 本地状态
    var showSearch by remember { mutableStateOf(false) }
    var searchQuery by remember { mutableStateOf("") }
    var showMenu by remember { mutableStateOf(false) }
    val coroutineScope = rememberCoroutineScope()
    
    // 副作用
    LaunchedEffect(searchQuery) {
        viewModel.search(searchQuery.ifBlank { null })
    }
    
    // 页面骨架
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("标题") },
                navigationIcon = {
                    IconButton(onClick = onBackClick) {
                        Icon(Icons.Default.ArrowBack, "返回")
                    }
                },
                actions = {
                    IconButton(onClick = { showSearch = !showSearch }) {
                        Icon(Icons.Default.Search, "搜索")
                    }
                    Box {
                        IconButton(onClick = { showMenu = true }) {
                            Icon(Icons.Default.MoreVert, "更多")
                        }
                        DropdownMenu(
                            expanded = showMenu,
                            onDismissRequest = { showMenu = false }
                        ) {
                            DropdownMenuItem(
                                text = { Text("选项") },
                                onClick = { showMenu = false }
                            )
                        }
                    }
                }
            )
        }
    ) { paddingValues ->
        Column(Modifier.padding(paddingValues)) {
            // 搜索框
            AnimatedVisibility(visible = showSearch) {
                OutlinedTextField(
                    value = searchQuery,
                    onValueChange = { searchQuery = it },
                    modifier = Modifier.fillMaxWidth().padding(16.dp),
                    placeholder = { Text("搜索...") }
                )
            }
            
            // 内容区域
            when (val state = uiState) {
                is XxxUIState.Loading -> {
                    Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
                        CircularProgressIndicator()
                    }
                }
                is XxxUIState.Success -> {
                    LazyColumn(Modifier.fillMaxSize()) {
                        items(state.data, key = { it.id }) { item ->
                            ItemCard(item)
                        }
                    }
                }
                is XxxUIState.Empty -> {
                    Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
                        Text("暂无数据")
                    }
                }
                is XxxUIState.Error -> {
                    Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
                        Text(state.message, color = MaterialTheme.colorScheme.error)
                    }
                }
            }
        }
    }
}
```

## 输出要求

重构完成后，提供以下信息：

1. **文件变更清单**
   - 新增文件列表
   - 修改文件列表
   - 删除文件列表

2. **代码对比**
   - 关键代码的前后对比
   - 功能对应关系

3. **验证结果**
   - 编译是否通过
   - 功能是否完整
