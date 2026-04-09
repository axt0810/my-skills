# RESTful API 接口规范 Skill

> 基于《一汽云原生API接口规范》的RESTful API设计检查助手

## 📖 简介

本 Skill 整理了云原生环境下的 RESTful API 接口规范要点，帮助开发者：

- ✅ **审查API设计** — 检查URL命名、HTTP方法、参数规范是否符合标准
- 📚 **规范学习** — 系统学习 RESTful API 设计最佳实践
- 💡 **问题解答** — 咨询 Swagger 注解、幂等设计、版本管理等具体问题

## 🎯 规范层级

| 标识 | 含义 | 违反后果 |
|------|------|----------|
| **【强制】** | 必须遵守 | 可能导致接口不可用或对接困难 |
| **【推荐】** | 建议遵守 | 影响API可维护性和一致性 |

## 📚 规范覆盖范围

### 一、编写注解规范

| 类别 | 核心要点 |
|------|----------|
| **自定义类** | 必须添加 @ApiModel 注解，说明类作用 |
| **属性命名** | 统一驼峰命名法，布尔属性禁止 `is` 前缀 |
| **控制器类** | 必须添加 @Api 注解，说明功能模块 |
| **控制器方法** | 必须添加 @ApiOperation 注解，说明方法功能 |

### 二、Swagger 注解规范

| 注解 | 必填属性 | 说明 |
|------|----------|------|
| **@Api** | tags | 控制器功能模块名称 |
| **@ApiOperation** | value | 方法功能描述 |
| **@ApiModel** | value | 模型名称/描述 |
| **@ApiModelProperty** | value, required, example | 属性描述、是否必填、示例值 |

### 三、请求参数规范

| 类别 | 核心要点 |
|------|----------|
| **命名规范** | 驼峰命名法，小写字母开头 |
| **验证注解** | 必填参数必须添加 @NotNull、@NotBlank 等 |
| **复杂参数** | 封装为 Request 对象，以 Request 结尾 |
| **分组验证** | 区分 CreateGroup、UpdateGroup 等场景 |

### 四、返回参数规范

| 类别 | 核心要点 |
|------|----------|
| **响应包裹** | 统一使用 Result/Response 包裹，包含 code、message、data |
| **命名规范** | 驼峰命名法，添加 @ApiModelProperty 注解 |
| **敏感信息** | 禁止直接返回密码、身份证号等，需脱敏处理 |
| **时间格式** | 统一格式 yyyy-MM-dd HH:mm:ss |
| **分页响应** | 包含 total、list、pageNum、pageSize、pages |

### 五、请求地址规范

| 类别 | 规则 | 示例 |
|------|------|------|
| **URL命名** | 小写字母 + 连字符 | `/user-info` |
| **禁止** | 下划线、大写字母、末尾斜杠 | ❌ `/user_info/` |
| **资源命名** | 名词复数形式 | `/users`、`/orders` |
| **版本管理** | v + 数字 | `/api/v1/users` |

#### HTTP 方法规范

| 方法 | 用途 | 幂等性 |
|------|------|--------|
| **GET** | 查询资源 | ✅ 幂等 |
| **POST** | 创建资源 | ❌ 非幂等（需幂等参数） |
| **PUT** | 全量更新 | ✅ 幂等 |
| **PATCH** | 部分更新 | ✅ 幂等 |
| **DELETE** | 删除资源 | ✅ 幂等 |

### 六、性能规范

| 指标 | 说明 | 要求 |
|------|------|------|
| **TP50** | 50%请求响应时间 | < 200ms |
| **TP99** | 99%请求响应时间 | < 1000ms |
| **幂等参数** | POST接口必须支持 | 客户端生成唯一幂等键 |
| **幂等键有效期** | Redis缓存 | 建议 24 小时 |

### 七、Swagger 配置规范

| 环境 | 配置 |
|------|------|
| **开发/测试** | `swagger.enabled: true` |
| **生产环境** | `swagger.enabled: false` |

## 🚀 使用方法

### 1. API 审查

发送 Controller 代码，按规范逐项检查：

```
请帮我审查这个API接口：

@RestController
@RequestMapping("/api/Users")
public class userController {
    @GetMapping("/getUser")
    public User getInfo(String id) {
        // ...
    }
}
```

**输出示例**：
> ❌ **【强制】** URL统一使用小写字母
> - 问题：`/api/Users` 应改为 `/api/v1/users`
> 
> ❌ **【强制】** URL中使用名词而非动词
> - 问题：`/getUser` 应改为 `/{id}`
> 
> ❌ **【强制】** 控制器类必须添加 @Api 注解
> - 正例：`@Api(tags = "用户管理")`

### 2. 规范查询

询问具体规范问题：

```
POST接口如何实现幂等？
```

**输出示例**：
> POST 接口非天然幂等，需通过幂等参数实现：
> 
> 1. 客户端生成唯一幂等键（如UUID）
> 2. 请求时携带 `X-Idempotent-Key` 请求头
> 3. 服务端使用 Redis 缓存幂等键，有效期 24 小时
> 4. 重复请求直接返回缓存结果

### 3. 分组验证

```
如何区分新增和修改场景的参数验证？
```

**输出示例**：
> 使用分组验证，定义不同验证组：
> 
> ```java
> public interface CreateGroup {}
> public interface UpdateGroup {}
> 
> public class UserRequest {
>     @NotNull(groups = UpdateGroup.class)
>     private Long userId;  // 修改时必填
>     
>     @NotBlank(groups = {CreateGroup.class, UpdateGroup.class})
>     private String userName;  // 新增和修改都必填
> }
> ```

## 📂 文件结构

```
skills/restful-api-standards/
├── README.md      # 本文档
└── SKILL.md       # 完整规范内容
```

## 📝 常见错误速查

| 错误类型 | 错误示例 | 正确示例 |
|----------|----------|----------|
| URL动词 | `/getUsers` | `/users` |
| URL下划线 | `/user_info` | `/user-info` |
| URL大写 | `/Users` | `/users` |
| 末尾斜杠 | `/users/` | `/users` |
| 缺少版本 | `/api/users` | `/api/v1/users` |
| 布尔is前缀 | `Boolean isValid` | `Boolean valid` |
| 属性大写开头 | `String UserName` | `String userName` |

## 🔗 参考资料

- 《一汽云原生API接口规范》
- RESTful API 设计指南
- Swagger/OpenAPI 规范

## 📝 版本信息

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0.0 | 2024-04 | 基于一汽云原生规范整理 |

---

> 💡 **提示**：规范的目的是统一团队编码风格、提升接口可维护性、降低前后端对接成本。好的 API 设计应该是自解释的、一致的、易于理解的！