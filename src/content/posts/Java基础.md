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

这里都是构造方法，方法名必须跟类名相同，不能有返回值。
## 构造方法是干嘛的
造对象用的，当你写`new BizException("余额不足")` 时，Java 就会去执行`BizException(String msg)` 这个构造方法（调构造器），把对象"初始化"好（把 code、message 填进去）。

## 为什么有 4 个名字都一样的构造方法？
因为它们的参数不一样 ，Java 允许"名字相同、参数不同"的方法共存，这叫 **重载** 。

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

`super` 是"我爹"（父类 RuntimeException）。这句话 = "爹，我这儿有个消息，你先帮我存一份"。Java 规定异常类必须先跟父类报备消息，这一行 **必须写在第一句** 。

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
---------------------------------------------------------------------------------------------
## 工具类 
public final class ***{
    private ***(){}
}
private static final ThreadLocal<UserInfo> USER_HOLDER = new ThreadLocal<UserInfo>();
//这样线程存储库就建好了 下面再写方法就好；


   public static void set(UserInfo user){
        USER_HOLDER.set(user);
    }
外面调用时，传进来UserInfo类型的对象，在这个方法里取个名字叫user，让后.set（user）调用ThreadLocal自带的set方法把user放进存储库；


    public static UserInfo get(){
        return USER_HOLDER.get();
    }
return就是交结果 从左往右看好理解一点 .get()取完值后用return返回给调用这个方法的人 并且返回值类型一定是UserInfo类型的；


  public static Long getUserId(){
        UserInfo user = USER_HOLDER.get();
        return user == null ? null : user.getUserId();
    }
￥ ？ 1 ： 2；意思就是在￥条件下成功返回1，否则返回2；

这里的第一行的getUserId是static方法，给外部直接类名.方法名调用的；第三行的getUserId是@Data自动生成的实例方法，需要先创建一个UserInfo对象（user），才能调用；

关于@Data
Lombok 的「自动生成规则」写的确实是字段`userId` ，Lombok 就是 根据这个字段名自动拼出 getter 方法名 的。
规则：get + 字段名首字母大写
Lombok 看到字段：
`private Long userId;`
就自动在编译时生成：
`public Long getUserId(){
    return this.userId;
}`


    public static UserInfo requireLogin(){
        UserInfo user = USER_HOLDER.get();
        if (user == null){
            throw new BizException(ResultCode.UNAUTHORIZED);
        }
        return user;
    }
        public static void remove(){
        USER_HOLDER.remove();
    }

“手环检验”在真检验（token相关）后再检验，确保用户已登录，防止未登录者修改业务。token是门票，requireLogin是手环（进场上线到线程池），进场看门票和手环，出场撕掉手环（下线，后面的remov方法），避免任务完成的线程信息残留在线程池里。
---------------------------------------------------------------------------------------------
## 全局异常处理器
/**
 * 全局异常处理器 接所有异常 转成Result结构返回
 */
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) //Web专用
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

}