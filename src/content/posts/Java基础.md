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

`this.code = code` 里的`this.` 就是为了在"字段名和参数名同名"时，指代"这个对象自己的字段"。

这里都是构造方法，方法名必须跟类名相同，不能有返回值。
## 构造方法是干嘛的
造对象用的，当你写`new BizException("余额不足")` 时，Java 就会去执行`BizException(String msg)` 这个构造方法（调构造器），把对象"初始化"好（把 code、message 填进去）。

## 为什么有 4 个名字都一样的构造方法？
因为它们的参数不一样 ，Java 允许"名字相同、参数不同"的方法共存，这叫 重载 。

`this(600, msg)` ，意思是： 在这个构造方法里，去调用本类另一个构造方法 （带括号的 this = 调构造器）。作用是"偷懒"，把活儿转给更全的那个构造器去干。


## 场景假设
将来某处代码写了这么一句：

`throw new BizException(502, "余额不足");`

意思就是：造一个异常对象，编号是 502，内容写"余额不足"。这时 Java 会自动跑你这个构造方法，把`502` 和`"余额不足"` 传进去。

## 现在跟着代码逐行走

`public BizException(int code, String message) {`

这一行 = 声明构造方法的门面，意思是"谁想造 BizException 对象，就得给我传一个`int` 和一个`String` "。传进来的值，Java 用 两个临时盒子 接住，盒子分别叫`code` 和`message` 。

执行到这里时，两个盒子里装的是：

`code    = 502
message = "余额不足"`

然后

`    super(message);`

`super` 是"我爹"（父类 RuntimeException）。这句话 = "爹，我这儿有个消息，你先帮我存一份"。Java 规定异常类必须先跟父类报备消息，这一行 必须写在第一句 。

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
`BizException 对象
├── code    = 502
├── message = "余额不足"
└── (父类 RuntimeException 里也存了 "余额不足")`

这就是`new BizException(502, "余额不足")` 造出来的完整东西。

## 那 get 是在干嘛
`getCode()` 和`getMessage()` 是`@Getter` 自动生成的两个方法，作用就是" 从包裹(传进来的枚举对象)里把值取出来 "：

- `resultCode.getCode()` → 从包裹里取出`401`
- `resultCode.getMessage()` → 从包裹里取出`"未登录"`