# Android Code Review Skill - Overview

## 技能定位

这是一个专业的 Android 代码审查技能，专注于发现 Kotlin 代码中的**关键问题**和**高优先级问题**，确保应用稳定性、可维护性和架构完整性。

## 核心功能

### 🎯 审查范围

| 优先级 | 问题类型 | 影响 |
|--------|----------|------|
| 🔴 CRITICAL | 内存泄漏、生命周期违规、协程泄漏 | 应用崩溃、ANR |
| 🟠 HIGH | 命名规范、UIState模式、业务逻辑位置、架构违规 | 可维护性差、难测试 |

### 📋 审查清单

#### 1. 命名规范
- **类型**：PascalCase（如 `PaymentViewModel`）
- **变量/函数**：camelCase（如 `paymentAmount`）
- **常量**：UPPER_SNAKE_CASE（如 `MAX_RETRY_COUNT`）
- **布尔值**：`is`/`has`/`should`/`can` 前缀（如 `isLoading`）
- **禁止**：缩写（除 URL、ID、API、HTTP、UI）

#### 2. 内存泄漏
- ❌ ViewModel 中禁止持有 Activity/Fragment/View 引用
- ❌ 禁止使用 GlobalScope
- ✅ 使用 viewModelScope/lifecycleScope
- ✅ Context 使用 ApplicationContext

#### 3. UIState 模式
- ✅ 使用 sealed class 定义单一 UIState
- ✅ 使用 `StateFlow<UIState>` 不可变暴露
- ✅ 覆盖所有状态：Loading、Success、Error、Empty

#### 4. 业务逻辑位置
- ❌ ViewModel 只能编排，不能包含业务逻辑
- ✅ 业务逻辑必须在 UseCase 中
- ✅ Repository 只做数据操作

#### 5. 生命周期管理
- ✅ Fragment 中使用 `viewLifecycleOwner`
- ✅ 使用 `repeatOnLifecycle` 或 `flowWithLifecycle`
- ✅ 在 onCleared() 中清理资源

#### 6. MVVM/MVI 模式
- ViewModel → UseCase → Repository（单向数据流）
- ViewModel 不直接调用 Repository
- View 只负责渲染，不含业务逻辑

## 文档结构

```
py-claude-marketplace-py-plugin-skills-android-code-review/
├── SKILL.md       # 技能主定义（触发词、审查流程、报告格式）
├── examples.md    # 7个真实问题案例（含错误代码和正确代码对比）
└── standards.md  # 8大主题开发标准（命名、Kotlin、协程、架构等）
```

## 示例案例

### 1. 内存泄漏 - Activity 引用
```kotlin
// ❌ 错误：ViewModel 持有 Activity 引用
class PaymentViewModel(private val activity: PaymentActivity) : ViewModel() {
    fun onPaymentComplete() {
        activity.showSuccessDialog() // 内存泄漏！
    }
}

// ✅ 正确：使用事件流
class PaymentViewModel : ViewModel() {
    private val _events = MutableSharedFlow<PaymentEvent>()
    val events: SharedFlow<PaymentEvent> = _events.asSharedFlow()

    fun onPaymentComplete() {
        viewModelScope.launch {
            _events.emit(PaymentEvent.ShowSuccessDialog)
        }
    }
}
```

### 2. 协程作用域
```kotlin
// ❌ 错误：GlobalScope 永不取消
GlobalScope.launch {
    repository.getPayments()
}

// ✅ 正确：使用 viewModelScope
class PaymentViewModel : ViewModel() {
    fun loadPayments() {
        viewModelScope.launch {
            val payments = repository.getPayments()
            _uiState.value = UiState.Success(payments)
        }
    }
}
```

### 3. 架构分层
```kotlin
// ❌ 错误：ViewModel 包含业务逻辑
class PaymentViewModel(private val repository: PaymentRepository) : ViewModel() {
    fun processPayment(amount: Double) {
        viewModelScope.launch {
            if (amount <= 0) return@launch
            val fee = amount * 0.02 // 业务逻辑！
            repository.savePayment(amount + fee)
        }
    }
}

// ✅ 正确：业务逻辑在 UseCase
class ProcessPaymentUseCase(private val repository: PaymentRepository) {
    suspend operator fun invoke(amount: Double): Result<Payment> {
        if (amount <= 0) return Result.failure(Exception("Invalid"))
        val fee = amount * 0.02
        return repository.savePayment(amount + fee)
    }
}
```

## 开发标准（8大主题）

| 主题 | 关键要点 |
|------|----------|
| **命名规范** | PascalCase/camelCase/UPPER_SNAKE_CASE、布尔前缀、无缩写 |
| **Kotlin 最佳实践** | 空安全、数据类、密封类、不可变性、扩展函数 |
| **协程模式** | viewModelScope、lifecycleScope、Dispatchers.IO、错误处理 |
| **Clean Architecture** | Presentation/Domain/Data 分层、UseCase、Repository |
| **生命周期管理** | viewLifecycleOwner、onCleared 清理、Flow collection |
| **安全最佳实践** | EncryptedSharedPreferences、无硬编码密钥、无敏感日志 |
| **性能优化** | 后台线程、DiffUtil、图片缓存、数据库索引 |
| **依赖注入** | Hilt/Dagger、@Module、@Inject、@AndroidEntryPoint |

## 输出格式

审查报告结构：
```
# Android Code Review Report

## Summary
- 🔴 Critical: X issues
- 🟠 High Priority: X issues

## 🔴 CRITICAL ISSUES
### 🔴 Memory Leak - [具体问题]

## 🟠 HIGH PRIORITY ISSUES
### 🟠 Naming Convention - [具体问题]
### 🟠 UIState Pattern - [具体问题]
### 🟠 Business Logic Misplacement - [具体问题]

## ⚠️ MUST FIX
## ✅ Well Done
```

## 触发词

当用户说以下内容时激活此技能：
- "review android code" / "检查 android 代码"
- "check android file" / "检查 android 文件"
- "review android PR" / "审查 android PR"
- "code quality" / "代码质量"
- "best practices" / "最佳实践"
- "MVI/MVVM patterns" / "MVI/MVVM 模式"

## 适用文件类型

- Activity / Fragment
- ViewModel
- UseCase
- Repository
- Kotlin/Java 源文件

## 忽略范围

以下内容不在审查范围内：
- 代码格式和风格（由 linter 处理）
- 文档和注释
- 性能优化（除非关键问题）
- 安全问题（单独审查）
- 测试覆盖
- 依赖注入配置

---

**核心原则**：只报告会导致崩溃、泄漏或违反核心架构的问题，提供具体修复建议和代码示例。