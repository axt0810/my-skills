# Coding Standards Skills

> Java 编码规范与 API 接口设计规范检查助手

## 📚 技能列表

| Skill | 图标 | 描述 |
|-------|------|------|
| [java-coding-standards](./java-coding-standards/) | ☕ | 阿里巴巴 Java 开发手册（嵩山版）|
| [restful-api-standards](./restful-api-standards/) | 🔌 | 云原生 API 接口规范 |

---

## ☕ Java 编码规范 (java-coding-standards)

基于《阿里巴巴 Java 开发手册（嵩山版）》，涵盖七大核心维度。

### 规范层级

| 标识 | 含义 |
|------|------|
| **【强制】** | 必须遵守，违反可能导致严重问题 |
| **【推荐】** | 建议遵守，有利于代码质量 |
| **【参考】** | 供参考，根据实际情况选择 |

### 核心规范速查

#### 命名风格
```java
// ✅ 正确
public class UserService { }
private String userName;
public static final int MAX_COUNT = 100;

// ❌ 错误
public class userservice { }      // 类名大驼峰
private String is_valid;          // 禁用下划线
public static final int maxCount; // 常量全大写
```

#### 并发处理
```java
// ✅ 正确 - 线程池创建
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize, maximumPoolSize, keepAliveTime,
    TimeUnit.SECONDS, new LinkedBlockingQueue<>(),
    new ThreadFactoryBuilder().setNameFormat("my-pool-%d").build()
);

// ❌ 错误 - Executors 创建
ExecutorService executor = Executors.newFixedThreadPool(10);
```

#### 集合处理
```java
// ✅ 正确 - 遍历时删除
list.removeIf(item -> item.getValue() > 100);

// ❌ 错误 - foreach 中删除
for (String item : list) {
    list.remove(item);  // ConcurrentModificationException
}
```

### 涵盖范围

| 维度 | 核心内容 |
|------|----------|
| **编程规约** | 命名风格、常量定义、代码格式、OOP规约、日期时间、集合处理、并发处理、控制语句、注释规约 |
| **异常日志** | 错误码定义、异常处理、日志规约 |
| **单元测试** | AIR原则、BCDE原则、覆盖率要求 |
| **安全规约** | SQL注入防护、XSS防护、CSRF防护、敏感数据处理 |
| **MySQL数据库** | 建表规约、索引规约、SQL语句、ORM映射 |
| **工程结构** | 应用分层、二方库依赖、服务器配置 |
| **设计规约** | 单一职责、避免循环依赖、组合优于继承 |

---

## 🔌 RESTful API 接口规范 (restful-api-standards)

基于《云原生 API 接口规范》，专注于 API 设计与 Swagger 注解规范。

### URL 设计规范

#### 命名规则
```
✅ 正确
GET    /api/v1/users           # 查询列表
GET    /api/v1/users/{id}      # 查询详情
POST   /api/v1/users           # 创建
PUT    /api/v1/users/{id}      # 全量更新
PATCH  /api/v1/users/{id}      # 部分更新
DELETE /api/v1/users/{id}      # 删除

❌ 错误
GET    /getUsers              # URL中使用动词
GET    /user_info             # 使用下划线
GET    /Users/                # 大写字母+末尾斜杠
GET    /api/users             # 缺少版本号
```

#### HTTP 方法规范

| 方法 | 用途 | 幂等性 |
|------|------|--------|
| **GET** | 查询资源 | ✅ 幂等 |
| **POST** | 创建资源 | ❌ 需幂等参数 |
| **PUT** | 全量更新 | ✅ 幂等 |
| **PATCH** | 部分更新 | ✅ 幂等 |
| **DELETE** | 删除资源 | ✅ 幂等 |

### Swagger 注解规范

#### Controller 注解
```java
@Api(tags = "用户管理")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @ApiOperation(value = "获取用户详情", notes = "根据ID获取用户信息")
    @GetMapping("/{id}")
    public Result<UserDTO> getUser(
        @ApiParam(value = "用户ID", required = true, example = "10001")
        @PathVariable Long id) {
        // ...
    }
}
```

#### 请求/响应对象
```java
@ApiModel("用户请求")
public class UserRequest {
    @ApiModelProperty(value = "用户姓名", required = true, example = "张三")
    @NotBlank(message = "用户姓名不能为空")
    private String userName;
}

@ApiModel("统一响应结果")
public class Result<T> {
    @ApiModelProperty(value = "响应码", example = "200")
    private Integer code;
    
    @ApiModelProperty(value = "响应消息", example = "操作成功")
    private String message;
    
    @ApiModelProperty(value = "响应数据")
    private T data;
}
```

### 核心规范速查

| 类别 | 规范要点 |
|------|----------|
| **URL命名** | 小写字母 + 连字符，名词复数，禁止下划线和动词 |
| **版本管理** | 必须包含版本号 `/api/v1/` |
| **请求参数** | 驼峰命名，必填参数加验证注解，复杂参数封装Request对象 |
| **返回参数** | Result包裹，敏感信息脱敏，时间格式统一 |
| **幂等设计** | POST接口必须支持幂等参数，幂等键有效期24小时 |
| **性能要求** | TP50 < 200ms，TP99 < 1000ms |

---

## 🚀 使用示例

### 代码审查

```
请帮我审查这段 Java 代码：

public class userService {
    public void getInfo(String id) {
        List<String> list = new ArrayList<>();
        for (String item : list) {
            if (item.equals("test")) {
                list.remove(item);
            }
        }
    }
}
```

**响应示例：**
> 发现以下问题：
> 
> ❌ **【强制】** 类名使用 UpperCamelCase 风格
> - 问题：`userService` 应改为 `UserService`
> 
> ❌ **【强制】** 禁止在 foreach 循环里进行 remove 操作
> - 问题：会产生 `ConcurrentModificationException`
> - 正例：`list.removeIf(item -> "test".equals(item))`

### API 设计审查

```
请审查这个 API 接口：

@RestController
@RequestMapping("/api/Users")
public class userController {
    @GetMapping("/getUserById")
    public User getInfo(String id) { }
}
```

**响应示例：**
> 发现以下问题：
> 
> ❌ **【强制】** URL 统一使用小写字母
> - 问题：`/api/Users` 应改为 `/api/v1/users`
> 
> ❌ **【强制】** URL 中使用名词而非动词
> - 问题：`/getUserById` 应改为 `/{id}`
> 
> ❌ **【强制】** 控制器类必须添加 @Api 注解
> - 正例：`@Api(tags = "用户管理")`

### 规范查询

```
线程池应该怎么创建？
```

```
POST 接口如何实现幂等？
```

```
数据库表设计有哪些强制规范？
```

---

## 📂 目录结构

```
skills/
├── README.md                    # 本文档（技能总览）
├── java-coding-standards/       # Java 编码规范
│   ├── README.md               # 使用说明
│   └── SKILL.md                # 完整规范
└── restful-api-standards/       # API 接口规范
    ├── README.md               # 使用说明
    └── SKILL.md                # 完整规范
```

---

## 📝 版本信息

| Skill | 版本 | 来源 |
|-------|------|------|
| java-coding-standards | 1.0.0 | 阿里巴巴 Java 开发手册（嵩山版）|
| restful-api-standards | 1.0.0 | 云原生 API 接口规范 |

---

> 💡 **核心理念**：规范的目的是提升协作效率、降低沟通成本，而非消灭代码的创造性。好的代码应该是自解释的、一致的、易于理解的！