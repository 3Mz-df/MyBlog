---
title: Java基础
published: 2026-10-26
description: Java基础
image: ''
tags: []
category: '技术学习'
draft: false
lang: ''
slug: java基础
---
## commit规范
`feat` 新功能 `fix` 修 bug `docs` 文档 `chore` 杂活（依赖、配置、构建） `refactor` 重构（不改功能） `style` 格式（空格、缩进，不动逻辑）

## 业务异常
```plaintext
@Getter
public class BizException extends RuntimeException{
    //编号
    private final int code;
    //消息消息
    private final String message;

    //直传一句话
    public BizException(String message){
        this(600,message); //下面接
    }

    //指定编号+指定内容
    public BizException(int code, String message){
        super(message); //把message交给父类RuntimeException
        this.code = code; //编号
        this.message = message; //消息内容
    }

       //指定枚举
    public BizException(ResultCode resultCode){
        this(resultCode.getCode(), resultCode.getMessage()); //交给上面的构造方法
    }

    //用枚举编号+指定内容
    public BizException(ResultCode resultCode, String message){
        this(resultCode.getCode(), message); //一样
    }
}
```

`this.code = code` 里的`this.` 就是为了在"字段名和参数名同名"时，指代"这个对象自己的字段"。

> [!NOTE]
> 这里都是构造方法，方法名必须跟类名相同，不能有返回值。
## 构造方法是干嘛的
造对象用的，当你写`new BizException("余额不足")` 时，Java 就会去执行`BizException(String msg)` 这个构造方法（调构造器），把对象"初始化"好（把 code、message 填进去）。

## 为什么有 4 个名字都一样的构造方法？
因为它们的参数不一样 ，Java 允许"名字相同、参数不同"的方法共存，这叫 <span style="color:#ea580c">**重载**</span> 。

`this(600, msg)` ，意思是： 在这个构造方法里，去调用本类另一个构造方法 （带括号的 this = 调构造器）。作用是"偷懒"，把活儿转给更全的那个构造器去干。


## 场景假设
将来某处代码写了这么一句：

`throw new BizException(502, "余额不足");`

意思就是：造一个异常对象，编号是 502，内容写"余额不足"。这时 Java 会自动跑你这个构造方法，把`502` 和`"余额不足"` 传进去。

## 现在跟着代码逐行走

`public BizException(int code, String message) {`

这一行 = 声明构造方法的门面，意思是"谁想造 BizException 对象，就得给我传一个`int` 和一个`String` "。传进来的值，Java 用 两个临时盒子 接住，盒子分别叫`code` 和`message` 。

执行到这里时，两个盒子里装的是：

```plaintext
code    = 502
message = "余额不足"
```

然后

`    super(message);`

`super` 是"我爹"（父类 RuntimeException）。这句话 = "爹，我这儿有个消息，你先帮我存一份"。Java 规定异常类必须先跟父类报备消息，这一行 <span style="color:#ea580c">**必须写在第一句**</span> 。

执行后：父类 RuntimeException 内部也存了一份`"余额不足"` （将来打日志、打印错误堆栈时要用它）。

然后

`    this.code = code;`

拆开看：

- `code` （右边的）= 传进来的临时盒子，值`502`
- `this.code` （左边的）= 类自己那个正式的字段`private final int code`
这句话 = "把传进来的 502，存进对象自己的 code 字段里"。

执行后：

`对象自己的 code 字段 = 502     ← 现在有值了`

然后

`    this.message = message;`

同理

## 全部执行完，你得到了一个对象
```plaintext
BizException 对象
├── code    = 502
├── message = "余额不足"
└── (父类 RuntimeException 里也存了 "余额不足")
```

这就是`new BizException(502, "余额不足")` 造出来的完整东西。

## 那 get 是在干嘛
`getCode()` 和`getMessage()` 是`@Getter` 自动生成的两个方法，作用就是" 从包裹(传进来的枚举对象)里把值取出来 "：

- `resultCode.getCode()` → 从包裹里取出`401`
- `resultCode.getMessage()` → 从包裹里取出`"未登录"`
---------------------------------------------------------------------------------------------
## 当前登录用户上下文
```plaintext
/**
 * 当前登录用户上下文：用户信息挂在当前线程上
 */
public final class UserContext {
    private UserContext(){}

    //线程存库
    private static final ThreadLocal<UserInfo> USER_HOLDER = new ThreadLocal<UserInfo>();

    //放进
    public static void set(UserInfo user){
        USER_HOLDER.set(user);
    }

    //取出
    public static UserInfo get(){
        return USER_HOLDER.get();
    }

    //只取 userId 没登陆返回null
    public static Long getUserId(){
        UserInfo user = USER_HOLDER.get();
        return user == null ? null : user.getUserId();
    }

    //强校验 没登录抛401
    public static UserInfo requireLogin(){
        UserInfo user = USER_HOLDER.get();
        if (user == null){
            throw new BizException(ResultCode.UNAUTHORIZED);
        }
        return user;
    }

    //清空 请求结束时调用
    public static void remove(){
        USER_HOLDER.remove();
    }

    //内嵌类（装用户信息）
    @Data
    public static class UserInfo{
        private Long userId; //用户ID
        private String username; //用户名
        private String realName; //真实姓名
        private Integer userType; //用户类型（1 2 3 4）
        private Long institutionId; //所属机构ID（给B端用）
    }
}
```
---------------------------------------------------------------------------------------------
## 工具类 
```plaintext
public final class ***{
    private ***(){}
}
private static final ThreadLocal<UserInfo> USER_HOLDER = new ThreadLocal<UserInfo>();
//这样线程存储库就建好了 下面再写方法就好；
```


```plaintext
   public static void set(UserInfo user){
        USER_HOLDER.set(user);
    }
```
外面调用时，传进来UserInfo类型的对象，在这个方法里取个名字叫user，让后.set（user）调用ThreadLocal自带的set方法把user放进存储库；


```plaintext
    public static UserInfo get(){
        return USER_HOLDER.get();
    }
```
return就是交结果 从左往右看好理解一点 .get()取完值后用return返回给调用这个方法的人 并且返回值类型一定是UserInfo类型的；


```plaintext
  public static Long getUserId(){
        UserInfo user = USER_HOLDER.get();
        return user == null ? null : user.getUserId();
    }
```
￥ ？ 1 ： 2；意思就是在￥条件下成功返回1，否则返回2；

> [!IMPORTANT]
> 这里的第一行的getUserId是static方法，给外部直接类名.方法名调用的；第三行的getUserId是@Data自动生成的实例方法，需要先创建一个UserInfo对象（user），才能调用；

关于@Data
Lombok 的「自动生成规则」写的确实是字段`userId` ，Lombok 就是 根据这个字段名自动拼出 getter 方法名 的。
规则：get + 字段名首字母大写
Lombok 看到字段：
`private Long userId;`
就自动在编译时生成：
```plaintext
public Long getUserId(){
    return this.userId;
}
```


```plaintext
    public static UserInfo requireLogin(){
        UserInfo user = USER_HOLDER.get();
        if (user == null){
            throw new BizException(ResultCode.UNAUTHORIZED);
        }
        return user;
    }
```
```plaintext
        public static void remove(){
        USER_HOLDER.remove();
    }
```

> [!TIP]
> “手环检验”在真检验（token相关）后再检验，确保用户已登录，防止未登录者修改业务。token是门票，requireLogin是手环（进场上线到线程池），进场看门票和手环，出场撕掉手环（下线，后面的remov方法），避免任务完成的线程信息残留在线程池里。
---------------------------------------------------------------------------------------------
## 全局异常处理器
```plaintext
/**
 * 全局异常处理器 接所有异常 转成Result结构返回
 */
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) 
//Web专用 只在 Servlet 类型的 Web 项目里生效
@RestControllerAdvice
//"全局异常处理中心"，能拦截异常并给前端返回 JSON
@Slf4j
public class GlobalExceptionHandler {
        //业务异常（BizException）
    @ExceptionHandler(BizException.class)
    public Result<Void> handleBizException(BizException e){
    //异常对象打包成e 返回一个Result，没有业务数据所以<Void>返回值自动变成json给前端
        log.warn("业务异常:{}", e.getMessage());
        //e就是BizException的对象名，对象名.方法名() 来调用方法
        return Result.failed(e.getCode(), e.getMessage());
        //.failed用到Result类的静态工厂方法，返回一个失败的Result对象
    }
//前端收到{ "code": 401, "message": "未登录或登录已过期", "data": null, "timestamp": 1730000000000 }

//后面的常规异常都是spring自带的异常，直接用即可（ MethodArgumentNotValidException.class BindException.class......... ）

//Stream流处理
    //@RequestBody参数的@Valid校验失败
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleMethodArgumentNotValid(MethodArgumentNotValidException e){
        String message = e.getBindingResult().getFieldErrors().stream()
        //链式调用
        e.getBindingResult()          // 返回 BindingResult 对象
        .getFieldErrors()           // BindingResult 有 getFieldErrors() 方法 继续调用
        .stream()                   // 前面的方法返回的 List 有 stream() 方法 
                .map(FieldError::getDefaultMessage)
                //对流水线上的 每一个 FieldError，调用它的`getDefaultMessage()` 方法
                .findFirst()
                .orElse(ResultCode.VALIDATE_FAILED.getMessage());
        log.warn("参数校验失败:{}", message);
        return Result.failed(ResultCode.VALIDATE_FAILED.getCode(), message);
    }

//spring自己调用，拿到返回的Result自动转成json给前端。spring框架替我调用，我只管写逻辑，这就是Spring的控制反转（Spring IOC）
① 参数校验失败 → Spring 自动 throw MethodArgumentNotValidException
② Spring 捕获异常，找到 @ExceptionHandler 对应的这个方法
③ Spring 自动调用 handleMethodArgumentNotValid(e)   ← 调用者是 Spring
④ 方法里 return Result.failed(...)                    ← 结果还给 Spring
⑤ Spring 拿到这个 Result，转成 JSON                     ← Spring 再转给前端
⑥ 前端收到 {"code":400, "message":"...", ...}
}
```
## 网关
网关做的事就叫 路由转发 ：`http://localhost:8080/system/xxx` → 自动转发到`eams-system` 服务。

//一个必须记住的坑
网关用的是 WebFlux + Netty （反应式、异步非阻塞），它跟`spring-boot-starter-web` （Servlet + Tomcat 同步模型）是 两套完全不同的引擎 。
用汽油车/柴油车打比方：一辆车只能装一种发动机，两个都塞进去就当场报错。所以：
 网关里绝对不能加`spring-boot-starter-web` ，否则启动直接冲突报错。

## MyBatis-Plus
//继承BaseMapper用

- 数据源 ：怎么告诉 Spring「数据库在哪、账号密码」。
  # 数据源
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: "jdbc:mysql://127.0.0.1:3306/eams?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&useSSL=false&allowPublicKeyRetrieval=true"
    username: root
    password: "****"

- 建表 → 实体类 → Mapper 三层对应关系，以及`@TableName` 、`@TableId` 两个翻译锚点。
  /**
  * 系统用户实体
 */
@Data
@TableName("sys_user")
public class SysUser {

    @TableId(type = IdType.AUTO)
    private Long id;

    private String username;

    private String nickname;

    private LocalDateTime createTime;
}

- MyBatis-Plus 的`BaseMapper` 白送增删改查，全程没手写一句 SQL。
/**
 * 用户Mapper 继承BaseMapper
 */
public interface SysUserMapper extends BaseMapper<SysUser> {
}

- 构造器注入 ：Spring 怎么把 Mapper「塞」进 Controller。返回 JSON ：Java 对象自动序列化给浏览器
/**
 * 检验数据库连通
 */
@RestController
@RequestMapping("/system/user")
public class UserController {
    
    private final SysUserMapper sysUserMapper;
    
    public UserController(SysUserMapper sysUserMapper){
        this.sysUserMapper = sysUserMapper;
    }
    
    @GetMapping("/list")
    public List<SysUser> list(){
        return sysUserMapper.selectList(null);
    }
}

## MyBatis-Plus 的 Service 也是"白送"的，继承两个套件`IService` +`ServiceImpl` ，跟 Mapper 一样自动拥有一堆方法（`list` /`getById` /`save` /`updateById` …）。

## 分页查询\
///**
 * MyBatis plus配置：分页插件
 */
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //分页拦截器
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
//SysUserService 增加 pageQuery 分页方法来用

## 补全增删改
// 新增用户：请求体里塞一段 JSON，Spring 自动翻译成 SysUser 对象
@PostMapping
public Result<Void> add(@RequestBody SysUser user){
    sysUserService.save(user);          // IService 自带的新增方法
    return Result.success();
}

// 修改用户：同样读请求体 JSON，但这次 JSON 里必须带 id
@PutMapping
public Result<Void> update(@RequestBody SysUser user){
    sysUserService.updateById(user);    // 根据 id 更新
    return Result.success();
}

// 删除用户：id 直接写在 URL 里，比如 /system/user/5
@DeleteMapping("/{id}")
public Result<Void> delete(@PathVariable Long id){
    sysUserService.removeById(id);      // 根据 id 删除
    return Result.success();
}


## 注解
  - `@RequestBody` ：负责"把 JSON 翻译成`SysUser` 对象"；
  - `@Valid` ：负责"翻译完之后，按`SysUser` 里写好的注解（`@NotBlank` 、`@Size` 等校验注解），把这个对象检查一遍"。


## Knife4j/OpenAPI接口测试文档配置
pom:
        <!-- Knife4j -->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
        </dependency>

config:
/**
 * Knife4j/OpenAPI接口测试文档配置
 */
@Configuration
public class Knife4jConfig {

    @Bean
    public OpenAPI openAPI(){
        return new OpenAPI()
                .info(new Info()
                        .title("EAMS 教培管理系统 - 系统服务接口文档")
                        .description("sys_user 用户管理等系统模块接口")
                        .version("1.0.0"));
    }
}

yml：
springdoc:
  api-docs:
    enabled: true #接口数据源
  swagger-ui:
    enabled: true #接口文档界面

knife4j:
  enable: true
  setting:
    language: zh_cn

----------------------------------------------------------
## 关于Spring：
Spring 是地基，Spring Boot 是盖在地基上的“精装脚手架” 。两者并非并列的替代关系，Spring Boot 并不是另一个新框架；它只是对 Spring 的自动化封装。

## Spring 到底是什么
“Spring”这个词其实指一整个家族，最底层的是 Spring Framework 。它提供的是这些“能力零件”：

模块 作用 IoC / DI 容器 帮你 new 对象、管对象、注入依赖 AOP 日志、事务这类横切逻辑抽出去 事务管理 `@Transactional` 背后的实现 Spring MVC 处理 HTTP 请求、路由到 Controller 数据访问 JdbcTemplate、整合 MyBatis / JPA

问题是：这些零件 全靠你手动拼装 。用传统 Spring 写一个 Web 项目，要写`web.xml` 、`applicationContext.xml` ，手动配 DispatcherServlet、数据源、事务管理器……配置量可能比业务代码还多。


## Spring Boot 干了什么
Spring Boot 不做新功能 ，它只做三件事，把上面那套拼装活儿自动化：

1. 起步依赖（starter） ：一个依赖顶一堆。引入`spring-boot-starter-web` ，Spring MVC、Jackson、内嵌 Tomcat 就都齐了，版本还帮你对齐好。
2. 自动配置 ：类路径里有什么 jar，它就按约定帮你配好对应的 Bean。有 Tomcat 就配 Web 环境，有 MySQL 驱动就配数据源——这就是“约定优于配置”。
3. 内嵌服务器 + 启动类 ：打成一个 jar，`java -jar` 直接跑，不用再装外部 Tomcat、部署 war 包。


## 注解
- Spring 容器 = 一个大工具箱 。
- `@Bean` = 往工具箱里"放一件工具"
- `@Configuration` = 告诉 Spring："这个类里放工具了，帮我登记一下。"
好处：以后谁要用这件工具， 不用自己`new` ，写个`@Autowired` ，Spring 就从工具箱里递给你。全项目共用同一个实例，而且以后想换，只改这一个地方。
----------------------------------------------------------
## JWT
//JWT工具类
public class JwtUtil {
    //token
    private static final String SECRET = "eams-jwt-secret-key-please-change-me-2026";

    //token有效期7天
    private static final long EXPIRE = 604800000;

    //转换
    private static final SecretKey KEY = Keys.hmacShaKeyFor(SECRET.getBytes());

    //生成token
    public static String generateToken(Long userId, String username) {
        return Jwts.builder()
                .subject(username) // 设置主题
                .claim("userId", userId) // 添加自定义声明
                .issuedAt(new Date()) // 设置签发时间
                .expiration(new Date(System.currentTimeMillis() + EXPIRE)) //过期时间
                .signWith(KEY)  //防伪
                .compact(); //打包
    }

    //解析token
    public static Claims parseToken(String token) {
        return Jwts.parser()
                .verifyWith(KEY)
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }
}

接口
    /**
     * 登录：校验用户名密码，成功返回 token
     */
    String login(LoginDTO loginDTO);
实现
    private final PasswordEncoder passwordEncoder;

    //构造器注入 让Spring传入 BCryptPasswordEncoder
    public SysUserServiceImpl(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }
        @Override
    public String login(LoginDTO loginDTO) {
        //根据用户名查用户
        SysUser user = this.getOne(new LambdaQueryWrapper<SysUser>()
                .eq(SysUser::getUsername, loginDTO.getUsername()), false);
        //用户不存在
        if (user == null || !passwordEncoder.matches(loginDTO.getPassword(), user.getPassword())) {
            throw new BizException(ResultCode.UNAUTHORIZED, "用户名或密码错误");
         }

        //登陆成功
        return JwtUtil.generateToken(user.getId(), user.getUsername());
        }
- `LambdaQueryWrapper` ：MyBatis-Plus 的"链式查询条件"，`.eq(字段, 值)` = "字段等于值"，不用手写 SQL。`getOne(..., false)` 里的`false` 意思是"如果查到多条不抛异常，取第一条"。
- `passwordEncoder.matches(明文, 密文)` ：BCrypt 是单向的，只能"比对"不能"解密"，返回 true/false。
- 为什么统一提示"用户名或密码错误" ：不告诉攻击者"到底是账号不存在还是密码错"，防止恶意试探出哪些用户名存在。
- 失败时抛`BizException(ResultCode.UNAUTHORIZED, ...)` ，会被 GlobalExceptionHandler ` 返回。这就是项目里"业务异常统一抛出"的用法。

## 密码加密方式
BCrypt 每次加密同一份明文，结果都不一样，因为他会再密文里加一个随机的盐（随机字符）
比如同样123的密码，每次加密出的结果都不同就是因为盐不一样。这样数据库里存的每个人的密码密文都是独立的。
这里用 matches(明文, 密文) 去检验那个数据库里存好的盐；
`matches` 内部干的事是这样：
1. 从数据库的密文里取出盐。
2. 用这个 同一把盐 + 你输入的明文，重新算一遍哈希。
3. 比"算出来的结果"和"数据库里存的"是不是一致。
if (user == null || !passwordEncoder.matches(loginDTO.getPassword(), user.getPassword())) {
            throw new BizException(ResultCode.UNAUTHORIZED, "用户名或密码错误");
         }
tips：`!` 是取反，意思是" 只要比对不过（false），就走进 if 里抛异常 "。

## 盐
`salt` （盐值）。它是密码学里一个通用概念，不光 BCrypt 用，SHA-256 加盐、PBKDF2、Argon2 这些加密方案全都用。
盐就是一段随机字符串 。加密前，把"密码 + 盐"混在一起再哈希。
不加盐：  hash("123456")                 → 所有人结果都一样
加盐：    hash("123456" + "随机盐A")      → 张三的结果 ≠ 李四的结果


## DTO
`password` 加的`@JsonIgnore` ，其实会 同时屏蔽两个方向 ：
- 输出 （返回给前端）：密码被隐藏 ✅ 这是我们要的。
- 输入 （前端传进来）：密码也读不进来 ❌ 这是副作用。
所以`SysUser` 不能 用来当登录的请求体（否则 password 根本收不到）。而且登录只需要`username + password` 两个字段，没必要塞整个实体。

结论：建一个专门的`LoginDTO` 来收登录参数。 DTO = 专门给前端"传进/传出"用的对象 ，跟数据库实体分开，这就是之前说的"彻底隔离"。
----------------------------------------------------------
## JWT 拦截器 + 注册放行路径
/**
 * JWT 登录拦截器：校验请求头里的 token，不合法就拦截返回 401
 */
@Component
public class JwtInterceptor implements HandlerInterceptor {

    private final ObjectMapper objectMapper;

    // 注入 Spring 的 ObjectMapper（把 Result 转成 JSON 写回给前端）
    public JwtInterceptor(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //从请求头里拿 token
        String token = request.getHeader("Authorization");

        //没带 token，或格式不对 拦截
        if (token == null || !token.startsWith("Bearer ")) {
            return reject(response);
        }

        //去掉 "Bearer " 前缀，拿到真正的 token
        token = token.substring(7);

        try {
            //解析 token，过期/伪造/被篡改都会抛异常
            JwtUtil.parseToken(token);
        } catch (Exception e) {
            // 解析失败  拦截
            return reject(response);
        }

        //token 合法 → 放行
        return true;
    }

    // 拦截并返回 401（把错误信息写成 JSON 写回给前端）
    private boolean reject(HttpServletResponse response) throws Exception {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // HTTP 状态码 401
        response.setContentType("application/json;charset=UTF-8");
        Result<Void> result = Result.failed(ResultCode.UNAUTHORIZED);
        response.getWriter().write(objectMapper.writeValueAsString(result));
        return false; // 返回 false = 拦截
    }
}

/**
 * Web 配置：注册 JWT 拦截器，并配置哪些路径需要登录、哪些放行
 */
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    private final JwtInterceptor jwtInterceptor;

    public WebMvcConfig(JwtInterceptor jwtInterceptor) {
        this.jwtInterceptor = jwtInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtInterceptor)
                .addPathPatterns("/**")           // 拦所有请求
                .excludePathPatterns(             // 但这些路径放行（免登录）
                        "/system/user/login",     // 登录接口本身（否则没法登录）
                        "/doc.html",              // Knife4j 文档页面
                        "/webjars/**",            // Knife4j 静态资源
                        "/v3/api-docs/**",        // 接口文档 JSON
                        "/swagger-ui/**",         // swagger-ui 页面
                        "/favicon.ico",
                        "/error"
                );
    }
}

1.`Authorization: Bearer xxx` 是啥？

这是 HTTP 行业约定：token 放在请求头`Authorization` 里，格式`Bearer 空格 token` 。`Bearer` 是"持有者"的意思，可以理解为"我拿这个凭证证明我是谁"。前端调接口时要在请求头带上它。

2. 为什么拦截器里要"自己写 JSON"，而不是抛异常？

还记得我们之前的`GlobalExceptionHandler` 吗？它其实 管不到拦截器 。因为：

- 拦截器的`preHandle` 在 Controller 之前 执行。
- `@RestControllerAdvice` 只能捕获 Controller 里 抛出的异常。
- 拦截器里抛异常，会直接变成 500 错误页，不会变成我们想要的 JSON。
所以拦截器里要 手动 把 401 和错误 JSON 写回`response` ，然后`return false` 拦下。

3.`addPathPatterns("/**")` +`excludePathPatterns(...)` 就是"看门规则"

- `addPathPatterns` ：大爷站哪些门（所有门`/**` ）。
- `excludePathPatterns` ：大爷手里的"免检名单"（登录接口、文档页面不用查证件）。
注意： 登录接口`login` 必须放行 ，否则你连登录都登录不了，就死锁了。


## 拦截器类里的线程 用上UserContext
  JwtInterceptor类下

try {
            Claims claims = JwtUtil.parseToken(token);

            //从 token 里取出 userId 和 username
            //用 toString 统一转
            Object userIdObj = claims.get("userId");
            Long userId = userIdObj == null ? null : Long.valueOf(userIdObj.toString());
            String username = claims.getSubject();

            // 装进 UserInfo，挂到当前线程的抽屉里
            UserContext.UserInfo userInfo = new UserContext.UserInfo();
            userInfo.setUserId(userId);
            userInfo.setUsername(username);
            UserContext.set(userInfo);
        } catch (Exception e) {
            //解析失败  拦截
            return reject(response);
        }

        //token 合法 → 放行
        return true;
    }

    // 请求结束时调用：清空线程池
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        UserContext.remove();
    }

1. 为什么要`afterCompletion` 里`UserContext.remove()` ？

Tomcat 用 线程池 复用线程。一个请求处理完，线程会被拿去处理下一个请求。如果不清空抽屉：

- 下一个请求可能拿到 上一个请求留下的 userId → 严重的话会"串号"（A 的操作被当成 B 的）。
- 抽屉里的对象引用一直不被释放 → 内存泄漏。
所以"用完必须清"，这是 ThreadLocal 的铁律。

2.`userIdObj.toString()` 这个写法是为什么？

我们存 token 时`userId` 是`Long` 。但 token 本质是 JSON，JSON 里`1` 这种小数字，解析回来时 可能是`Integer` 也可能是`Long` 。如果直接`(Long) claims.get("userId")` 强转，遇到`Integer` 就会 ClassCastException。用`Long.valueOf(toString())` 两种都能安全转。

3.`get()` vs`requireLogin()`

`UserContext` 里有两个取法（你已经写好了）：

- `get()` → 可能返回 null（宽松）。
- `requireLogin()` → 没登录就抛 401（严格）。

## JWT至此；
- JWT：签 token、验 token、Header 里怎么传。
- BCrypt：盐、单向哈希、`encode` /`matches` 。
- Spring DI：`@Component` 、`@Bean` 、构造器注入、按类型注入。
- 拦截器：`HandlerInterceptor` 、放行规则、为什么拦截器里要手动写 JSON。
- ThreadLocal +`UserContext` ：上下文传递、为什么用完必须`remove()` 。