---
name: restful-api-standards
description: "基于《一汽云原生API接口规范》的RESTful API设计检查助手。审查API接口设计、URL命名、参数规范、Swagger注解、返回值规范等，帮助开发者设计规范的RESTful API。"
metadata:
  {
    "builtin_skill_version": "1.0",
    "copaw":
      {
        "emoji": "🔌",
        "requires": {}
      }
  }
---

# RESTful API 接口规范 Skill

## 描述

基于《一汽云原生API接口规范》的API设计检查助手。帮助开发者：
- 审查API接口设计是否符合规范
- 检查Swagger注解使用是否正确
- 验证请求/返回参数命名规范
- 解答RESTful API设计相关问题

## 规范层级说明

- **【强制】**：必须遵守，违反可能导致接口不可用或对接困难
- **【推荐】**：建议遵守，有利于API可维护性和一致性

---

## 一、编写注解规范

### 1.1 参数/返回值自定义类对象

| 规则 | 级别 | 说明 |
|------|------|------|
| 自定义类必须添加@ApiModel注解 | 强制 | 说明该类的作用 |
| 属性名统一使用驼峰命名法 | 强制 | 小写字母开头，如 `userName` |
| 禁止使用`is`前缀命名布尔属性 | 强制 | 否则序列化会有问题 |

**正例：**
```java
@ApiModel("用户信息")
public class UserDTO {
    @ApiModelProperty("用户姓名")
    private String userName;
    
    @ApiModelProperty("是否有效")
    private Boolean valid;
}
```

**反例：**
```java
public class UserDTO {
    private String UserName;  // ❌ 首字母大写
    private Boolean isValid;  // ❌ 布尔属性用is前缀
}
```

### 1.2 控制器类

| 规则 | 级别 | 说明 |
|------|------|------|
| 控制器类必须添加@Api注解 | 强制 | 说明该控制器的功能模块 |
| 控制器类方法必须添加@ApiOperation注解 | 强制 | 说明该方法的功能 |

**正例：**
```java
@Api(tags = "用户管理")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @ApiOperation("获取用户详情")
    @GetMapping("/{id}")
    public Result<UserDTO> getUser(@PathVariable Long id) {
        // ...
    }
}
```

---

## 二、注解规范

### 2.1 @Api

| 属性 | 要求 | 说明 |
|------|------|------|
| tags | 必填 | 控制器功能模块名称 |
| value | 可选 | 控制器描述 |

**示例：**
```java
@Api(tags = "订单管理")
@Api(tags = {"订单管理", "交易模块"})  // 多标签
```

### 2.2 @ApiOperation

| 属性 | 要求 | 说明 |
|------|------|------|
| value | 必填 | 方法功能描述 |
| notes | 推荐 | 详细说明、注意事项 |
| httpMethod | 可选 | HTTP方法，如GET、POST |

**示例：**
```java
@ApiOperation(value = "创建订单", notes = "创建订单后需要调用支付接口完成支付")
@PostMapping
public Result<OrderDTO> createOrder(@RequestBody OrderDTO order) {
    // ...
}
```

### 2.3 @ApiModel

| 属性 | 要求 | 说明 |
|------|------|------|
| value | 必填 | 模型名称/描述 |
| description | 可选 | 详细描述 |

**示例：**
```java
@ApiModel(value = "用户DTO", description = "用户信息传输对象")
public class UserDTO {
    // ...
}
```

### 2.4 @ApiModelProperty

| 属性 | 要求 | 说明 |
|------|------|------|
| value | 必填 | 属性描述 |
| required | 推荐 | 是否必填，默认false |
| example | 推荐 | 示例值 |
| hidden | 可选 | 是否隐藏，默认false |

**示例：**
```java
@ApiModelProperty(value = "用户ID", required = true, example = "10001")
private Long userId;

@ApiModelProperty(value = "创建时间", example = "2024-01-01 12:00:00")
private Date createTime;
```

---

## 三、请求参数规范

### 3.1 请求参数对象属性

| 规则 | 级别 | 说明 |
|------|------|------|
| 属性名统一使用驼峰命名法 | 强制 | 小写字母开头 |
| 必填参数必须添加验证注解 | 强制 | 如@NotNull、@NotBlank |
| 参数必须添加@ApiModelProperty注解 | 强制 | 说明参数含义 |
| 禁止使用Java关键字命名 | 强制 | 避免序列化问题 |

**正例：**
```java
@ApiModel("用户查询请求")
public class UserQueryRequest {
    
    @ApiModelProperty(value = "用户ID", required = true, example = "10001")
    @NotNull(message = "用户ID不能为空")
    private Long userId;
    
    @ApiModelProperty(value = "用户姓名", example = "张三")
    @Size(max = 50, message = "姓名长度不能超过50")
    private String userName;
    
    @ApiModelProperty(value = "页码", example = "1")
    @Min(value = 1, message = "页码最小为1")
    private Integer pageNum;
    
    @ApiModelProperty(value = "每页条数", example = "10")
    @Max(value = 100, message = "每页条数最大为100")
    private Integer pageSize;
}
```

### 3.2 请求参数对象

| 规则 | 级别 | 说明 |
|------|------|------|
| 复杂查询参数封装为对象 | 推荐 | 避免方法参数过多 |
| 对象命名以Request结尾 | 推荐 | 如UserQueryRequest |
| 添加分组验证 | 推荐 | 区分新增/修改场景 |

**示例：**
```java
// 分组验证
public interface CreateGroup {}
public interface UpdateGroup {}

@ApiModel("用户请求")
public class UserRequest {
    
    @ApiModelProperty(value = "用户ID", example = "10001")
    @NotNull(groups = UpdateGroup.class, message = "修改时用户ID不能为空")
    private Long userId;
    
    @ApiModelProperty(value = "用户姓名", required = true)
    @NotBlank(groups = {CreateGroup.class, UpdateGroup.class}, message = "用户姓名不能为空")
    private String userName;
}
```

---

## 四、返回参数规范

### 4.1 返回参数对象属性

| 规则 | 级别 | 说明 |
|------|------|------|
| 属性名统一使用驼峰命名法 | 强制 | 小写字母开头 |
| 返回属性必须添加@ApiModelProperty注解 | 强制 | 说明返回值含义 |
| 敏感信息禁止直接返回 | 强制 | 如密码、身份证号需脱敏 |
| 时间类型统一格式 | 强制 | 使用String类型或配置序列化格式 |

**正例：**
```java
@ApiModel("用户信息响应")
public class UserResponse {
    
    @ApiModelProperty(value = "用户ID", example = "10001")
    private Long userId;
    
    @ApiModelProperty(value = "用户姓名", example = "张三")
    private String userName;
    
    @ApiModelProperty(value = "手机号", example = "138****1234")
    private String phone;  // 脱敏处理
    
    @ApiModelProperty(value = "创建时间", example = "2024-01-01 12:00:00")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;
}
```

### 4.2 返回参数包裹类

| 规则 | 级别 | 说明 |
|------|------|------|
| 统一使用Result/Response包裹返回值 | 强制 | 统一响应格式 |
| 包含code、message、data三个基本字段 | 强制 | 标准响应结构 |
| 提供常用静态工厂方法 | 推荐 | 如success()、error() |

**正例：**
```java
@ApiModel("统一响应结果")
public class Result<T> {
    
    @ApiModelProperty(value = "响应码", example = "200")
    private Integer code;
    
    @ApiModelProperty(value = "响应消息", example = "操作成功")
    private String message;
    
    @ApiModelProperty(value = "响应数据")
    private T data;
    
    // 静态工厂方法
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("操作成功");
        result.setData(data);
        return result;
    }
    
    public static <T> Result<T> error(Integer code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        return result;
    }
}
```

### 4.3 分页返回参数

| 规则 | 级别 | 说明 |
|------|------|------|
| 分页结果必须包含总数 | 强制 | 便于前端分页展示 |
| 统一分页响应结构 | 强制 | 包含total、list、pageNum、pageSize |

**正例：**
```java
@ApiModel("分页响应结果")
public class PageResult<T> {
    
    @ApiModelProperty(value = "总记录数", example = "100")
    private Long total;
    
    @ApiModelProperty(value = "数据列表")
    private List<T> list;
    
    @ApiModelProperty(value = "当前页码", example = "1")
    private Integer pageNum;
    
    @ApiModelProperty(value = "每页条数", example = "10")
    private Integer pageSize;
    
    @ApiModelProperty(value = "总页数", example = "10")
    private Integer pages;
}
```

---

## 五、请求地址规范

### 5.1 URL命名规范

| 规则 | 级别 | 说明 |
|------|------|------|
| URL统一使用小写字母 | 强制 | 避免大小写敏感问题 |
| 单词之间使用连字符(-)分隔 | 强制 | 如 `/user-info` |
| 禁止使用下划线(_) | 强制 | 与连字符统一 |
| 禁止在URL末尾加斜杠(/) | 强制 | 保持URL一致性 |
| 使用名词而非动词 | 强制 | RESTful风格 |
| 使用复数形式表示资源 | 推荐 | 如 `/users`、`/orders` |

**正例：**
```
GET    /api/v1/users           # 获取用户列表
GET    /api/v1/users/{id}      # 获取单个用户
POST   /api/v1/users           # 创建用户
PUT    /api/v1/users/{id}      # 更新用户
DELETE /api/v1/users/{id}      # 删除用户
GET    /api/v1/user-orders     # 用户订单列表
```

**反例：**
```
GET    /api/v1/getUsers        # ❌ URL中使用动词
GET    /api/v1/user_info       # ❌ 使用下划线
GET    /api/v1/Users           # ❌ 大写字母
GET    /api/v1/users/         # ❌ 末尾斜杠
```

### 5.2 版本管理规范

| 规则 | 级别 | 说明 |
|------|------|------|
| API必须包含版本号 | 强制 | 便于接口升级和维护 |
| 版本号格式为v+数字 | 强制 | 如v1、v2 |
| 版本号放在URL中 | 推荐 | 如 `/api/v1/users` |
| 大版本变更时升级版本号 | 强制 | 不兼容修改需升级大版本 |

**示例：**
```
/api/v1/users    # 版本1
/api/v2/users    # 版本2（不兼容升级）
```

### 5.3 HTTP方法规范

| HTTP方法 | 用途 | 是否幂等 |
|----------|------|----------|
| GET | 查询资源 | 是 |
| POST | 创建资源 | 否 |
| PUT | 全量更新资源 | 是 |
| PATCH | 部分更新资源 | 是 |
| DELETE | 删除资源 | 是 |

**示例：**
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @GetMapping              // 查询列表
    public Result<PageResult<UserDTO>> list(UserQueryRequest request) { }
    
    @GetMapping("/{id}")     // 查询详情
    public Result<UserDTO> get(@PathVariable Long id) { }
    
    @PostMapping             // 创建
    public Result<UserDTO> create(@RequestBody @Validated UserRequest request) { }
    
    @PutMapping("/{id}")     // 全量更新
    public Result<UserDTO> update(@PathVariable Long id, @RequestBody @Validated UserRequest request) { }
    
    @PatchMapping("/{id}")    // 部分更新
    public Result<UserDTO> patch(@PathVariable Long id, @RequestBody Map<String, Object> updates) { }
    
    @DeleteMapping("/{id}")   // 删除
    public Result<Void> delete(@PathVariable Long id) { }
}
```

---

## 六、性能规范

### 6.1 响应时间要求

| 指标 | 说明 | 要求 |
|------|------|------|
| TP50 | 50%请求的响应时间 | < 200ms |
| TP99 | 99%请求的响应时间 | < 1000ms |
| OPS | 每秒操作数 | 根据业务场景定义 |

### 6.2 幂等性要求

| 操作类型 | 是否幂等 | 实现方式 |
|----------|----------|----------|
| GET | 是 | 天然幂等 |
| PUT | 是 | 全量更新幂等 |
| DELETE | 是 | 多次删除结果相同 |
| POST | 否 | 需要幂等参数 |

**POST幂等实现：**
```java
@ApiModel("创建订单请求")
public class OrderCreateRequest {
    
    @ApiModelProperty(value = "幂等键", required = true, example = "uuid-12345")
    @NotBlank(message = "幂等键不能为空")
    private String idempotentKey;  // 客户端生成的唯一标识
    
    @ApiModelProperty(value = "订单信息")
    private OrderInfo orderInfo;
}
```

### 6.3 幂等参数要求

| 规则 | 级别 | 说明 |
|------|------|------|
| POST接口必须支持幂等参数 | 强制 | 防止重复提交 |
| 幂等键由客户端生成 | 强制 | 使用UUID或业务唯一标识 |
| 服务端缓存幂等键 | 强制 | 使用Redis等缓存 |
| 幂等键有效期设置 | 推荐 | 建议24小时 |

**幂等处理示例：**
```java
@PostMapping
@ApiOperation("创建订单")
public Result<OrderDTO> createOrder(
    @RequestHeader("X-Idempotent-Key") String idempotentKey,
    @RequestBody @Validated OrderCreateRequest request) {
    
    // 1. 检查幂等键是否存在
    if (redisTemplate.hasKey(idempotentKey)) {
        // 返回之前的结果
        return redisTemplate.opsForValue().get(idempotentKey);
    }
    
    // 2. 执行业务逻辑
    OrderDTO result = orderService.createOrder(request);
    
    // 3. 缓存结果
    redisTemplate.opsForValue().set(idempotentKey, Result.success(result), 24, TimeUnit.HOURS);
    
    return Result.success(result);
}
```

---

## 七、Swagger配置规范

### 7.1 开启Swagger

| 规则 | 级别 | 说明 |
|------|------|------|
| 开发/测试环境开启Swagger | 推荐 | 便于接口调试 |
| 生产环境关闭Swagger | 强制 | 安全考虑 |
| 配置API文档基本信息 | 强制 | 标题、描述、版本 |

**配置示例：**
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    
    @Value("${swagger.enabled:false}")
    private Boolean enabled;
    
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
            .enable(enabled)
            .apiInfo(apiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
            .paths(PathSelectors.any())
            .build();
    }
    
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
            .title("XX系统API文档")
            .description("XX系统接口说明文档")
            .version("v1.0.0")
            .contact(new Contact("开发团队", "", "dev@example.com"))
            .build();
    }
}
```

**application.yml配置：**
```yaml
swagger:
  enabled: true  # 生产环境设为false
```

### 7.2 Controller注解示例

```java
@Api(tags = "用户管理")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @ApiOperation(value = "获取用户列表", notes = "支持分页查询，按创建时间倒序排列")
    @GetMapping
    public Result<PageResult<UserResponse>> list(
        @ApiParam(value = "查询条件") @Validated UserQueryRequest request) {
        // ...
    }
    
    @ApiOperation(value = "获取用户详情", notes = "根据用户ID获取用户详细信息")
    @GetMapping("/{id}")
    public Result<UserResponse> get(
        @ApiParam(value = "用户ID", required = true, example = "10001") 
        @PathVariable Long id) {
        // ...
    }
    
    @ApiOperation(value = "创建用户", notes = "创建新用户，返回用户详情")
    @PostMapping
    public Result<UserResponse> create(
        @ApiParam(value = "用户信息", required = true) 
        @RequestBody @Validated(CreateGroup.class) UserRequest request) {
        // ...
    }
}
```

---

## 八、常见错误示例

### 8.1 URL命名错误

| 错误示例 | 正确示例 | 问题 |
|----------|----------|------|
| `/getUsers` | `/users` | URL中使用动词 |
| `/user_info` | `/user-info` | 使用下划线 |
| `/Users` | `/users` | 大写字母 |
| `/users/` | `/users` | 末尾斜杠 |
| `/api/users` | `/api/v1/users` | 缺少版本号 |

### 8.2 参数命名错误

| 错误示例 | 正确示例 | 问题 |
|----------|----------|------|
| `private String UserName;` | `private String userName;` | 首字母大写 |
| `private Boolean isValid;` | `private Boolean valid;` | 布尔属性is前缀 |
| `private String user_name;` | `private String userName;` | 下划线命名 |

### 8.3 Swagger注解缺失

| 问题 | 影响 | 解决 |
|------|------|------|
| Controller无@Api注解 | Swagger文档分组混乱 | 添加@Api(tags = "模块名") |
| 方法无@ApiOperation注解 | 接口功能不明确 | 添加@ApiOperation(value = "功能描述") |
| 参数无@ApiModelProperty注解 | 参数含义不明确 | 添加@ApiModelProperty注解 |
| 返回值无@ApiModel注解 | 响应结构不清晰 | 添加@ApiModel注解 |

---

## 使用方法

### 代码审查场景

当用户要求审查API接口时：

1. 检查URL命名是否符合规范
2. 检查HTTP方法使用是否正确
3. 检查Swagger注解是否完整
4. 检查参数/返回值命名规范
5. 检查幂等性设计

### 规范咨询场景

当用户询问API设计问题时：

1. 给出明确的规范要求
2. 说明规范级别
3. 提供正例和反例

### 学习场景

当用户学习RESTful API规范时：

1. 按章节讲解
2. 结合实际代码示例
3. 说明规范的背景和意义