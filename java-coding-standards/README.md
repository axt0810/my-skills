# Java 编码规范检查 Skill

> 基于《阿里巴巴 Java 开发手册（嵩山版）》的代码规范检查助手

## 📖 简介

本 Skill 整理了阿里巴巴 Java 开发手册的核心规范要点，帮助开发者：

- ✅ **代码审查** — 快速检查代码是否符合规范
- 📚 **规范学习** — 系统学习 Java 编码最佳实践
- 💡 **问题解答** — 咨询具体规范问题和解决方案

## 🎯 规范层级

| 标识 | 含义 | 违反后果 |
|------|------|----------|
| **【强制】** | 必须遵守 | 可能导致严重 bug 或系统故障 |
| **【推荐】** | 建议遵守 | 影响代码质量和可维护性 |
| **【参考】** | 供参考 | 根据实际情况选择 |

## 📚 规范覆盖范围

### 一、编程规约

| 子章节 | 核心要点 |
|--------|----------|
| **命名风格** | 类名 UpperCamelCase、方法名 lowerCamelCase、常量全大写下划线分隔、禁用拼音命名 |
| **常量定义** | 禁止魔法值、Long 类型使用大写 L、常量分层管理 |
| **代码格式** | 大括号规范、4空格缩进、单行120字符限制、方法参数不超过5个 |
| **OOP 规约** | @Override 注解、equals/hashCode、BigDecimal 使用、包装类使用规范 |
| **日期时间** | yyyy 与 YYYY 区别、JDK8 使用 Instant/LocalDateTime |
| **集合处理** | subList 陷阱、toArray 规范、foreach 中 remove/add、泛型通配符 PECS |
| **并发处理** | 线程池创建、SimpleDateFormat 线程安全、锁的使用规范、ThreadLocal 回收 |
| **控制语句** | switch 必须有 default、必须使用大括号、卫语句优先 |
| **注释规约** | Javadoc 规范、枚举注释、TODO 格式 |

### 二、异常日志

| 子章节 | 核心要点 |
|--------|----------|
| **错误码** | A-B-C 格式、分层管理、不直接输出给用户 |
| **异常处理** | 禁止空 catch、禁止 Exception 作为 throws 类型、NPE 防护 |
| **日志规约** | SLF4J 框架、日志级别判断、禁止 System.out、TraceId 链路追踪 |

### 三、单元测试

- **AIR 原则** — Automatic（自动）、Independent（独立）、Repeatable（可重复）
- **BCDE 原则** — Border（边界）、Correct（正确）、Design（设计）、Error（错误）
- **覆盖率要求** — 语句覆盖 70%+，核心模块 100%

### 四、安全规约

| 风险类型 | 防护措施 |
|----------|----------|
| **越权访问** | 权限校验拦截 |
| **SQL 注入** | 参数过滤、#{} 替代 ${} |
| **XSS 攻击** | HTML 转义 |
| **CSRF 攻击** | Token 验证 |
| **敏感数据** | 脱敏展示、加密存储 |

### 五、MySQL 数据库

| 子章节 | 核心要点 |
|--------|----------|
| **建表规约** | is_xxx 命名、小写表名、decimal 存小数、必备三字段、utf8mb4 字符集 |
| **索引规约** | 唯一索引、三表禁 join、varchar 索引长度、覆盖索引 |
| **SQL 语句** | count(*)、ISNULL() 判断、禁用存储过程/触发器、禁 NOT IN |
| **ORM 映射** | 字段映射、禁用 * 查询、#{} 防注入 |

### 六、工程结构

- **应用分层** — DO/DTO/VO/AO 模型定义与转换
- **二方库依赖** — 版本号规范、SNAPSHOT 禁用
- **服务器配置** — TCP time_wait、文件句柄、JVM HeapDump

### 七、设计规约

- 单一职责原则
- 避免循环依赖
- 组合优于继承
- 预留扩展点

## 🚀 使用方法

### 1. 代码审查

发送 Java 代码片段，按规范逐项检查：

```
请帮我审查这段代码：

public class userService {
    public void getInfo(String id) {
        // ...
    }
}
```

**输出示例**：
> ❌ 【强制】类名使用 UpperCamelCase 风格
> - 问题：`userService` 应改为 `UserService`
> - 正例：`public class UserService { }`

### 2. 规范查询

询问具体规范问题：

```
线程池应该怎么创建？
```

**输出示例**：
> 【强制】线程池不允许使用 Executors 创建，应通过 ThreadPoolExecutor 创建
> 
> **原因**：Executors 创建的线程池存在 OOM 风险
> - `FixedThreadPool` 和 `SingleThreadPool` 允许的请求队列长度为 Integer.MAX_VALUE
> - `CachedThreadPool` 允许的创建线程数量为 Integer.MAX_VALUE

### 3. 分类学习

按章节学习规范：

```
讲讲集合处理的规范
```

## 📂 文件结构

```
skills/java-coding-standards/
├── README.md      # 本文档
└── SKILL.md       # 完整规范内容
```

## 🔗 参考资料

- 《阿里巴巴 Java 开发手册（嵩山版）》
- 配套图书《码出高效》

## 📝 版本信息

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0.0 | 2024-04 | 基于嵩山版整理 |

---

> 💡 **提示**：规范的目的是提升协作效率、降低沟通成本，而非消灭代码的创造性。代码的字里行间流淌的是软件系统的血液，码出质量，码出高效！