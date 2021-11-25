---
layout:     post
title:      Android AOP之AspectJ入门
subtitle:   
date:       2020-05-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
--- 

### 前言

什么是AOP，什么是AspectJ，这里就不过多介绍了，如果有不懂的，请移步[Android AOP从入门到实战](http://rjgc.cn/2020/05/01/Android-AOP%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%AE%9E%E6%88%98/) 进行了解，本文旨在AspectJ使用介绍。



### 一、术语

- **Join Points**

  Join Points(连接点)，简称JPoints，简单点说就是你想在程序的哪个地方插入代码，能插入代码的地方就称为Join Points。

  请注意，并不是所有地方都能插入代码，AspectJ所认可的JoinPoints的类型如下表

  | Join Points           | 说明                                                   |
  | --------------------- | ------------------------------------------------------ |
  | method call           | 方法被调用的地方                                       |
  | method execution      | 方法内部，比如test(){//此处可以看做Join Point}，       |
  | constructor call      | 构造方法被调用的地方                                   |
  | constructor exectuion | 构造方法内部                                           |
  | field get             | 读取属性的地方                                         |
  | field set             | 写入属性的地方                                         |
  | static initialization | static静态代码块内部                                   |
  | handler               | 异常处理，比如try catch(xxx)的catch块可以看到joinPoint |

- **Pointcuts**

  pointcuts(切入点),即用来告诉程序，在哪个Join Points注入哪些代码。

  一个程序会有多个Join Points，即使同一个函数，也还分为call和execution类型的Join Points，但并不是所有的Join Points都是我们关心的，Pointcuts就是提供一种使得开发者能够选择自己需要的JoinPoints的方法。

- **Advice**

  Advice(通知)，即注入到class文件的具体代码，以及注入的具体位置。比如 before表示在目标房前执行前插入代码；after表示在目标方法执行后插入代码，还有其他的后续会介绍到

- **Aspect**

  Apect，即切面，一般负责管理注解的处理和代码织入的类称为切面类

### 二、语法

- **Pointcuts**

  上面有说过，pointcuts是用来捕获符合我们要求的Join Point，那如何捕获呢？pointcuts 遵循特定的语法用于捕获每一种可用的连接点，具体语法如下：

  | Join Points           | 对应的pointcuts语法                               |
  | --------------------- | ------------------------------------------------- |
  | method call           | call(MethodSignature)                             |
  | method execution      | execution(MethodSignature)                        |
  | constructor call      | call(ConstructorSignature)                        |
  | constructor exectuion | execution(ConstructorSignature)                   |
  | field get             | get(FieldSignature)                               |
  | field set             | set(FieldSignature)                               |
  | static initialization | staticinitialization(TypeSignature)               |
  | handler               | handler(TypeSignature)注：仅能与@Before()配合使用 |

  我们可以看到pointcuts语法中包含一些MethodSignature、ConstructorSignature等Signature参数，那他们到底是什么呢？其实就是表示我们需要获取什么样的join point,即join point的特征，请看下表：

  | Signature            | 语法/规则                                                    |
  | -------------------- | ------------------------------------------------------------ |
  | MethodSignature      | @Annotation java修饰符 返回值类型 类名.方法名(参数)          |
  | ConstructorSignature | @Annotation java修饰符 返回值类型 类名.new(参数)             |
  | FieldSignature       | @Annotation java修饰符 属性类型 类名.属性名                  |
  | TypeSignature        | 类名  例：staticinitialization(Person):表示Person类的静态代码块 |

  对于以上的讲解有的朋友可能依然还是搞不懂怎么定义Signature，那我们再详细解释一下，看下表：

  | Signature语法明细 | 说明                                                         |
  | ----------------- | ------------------------------------------------------------ |
  |                   | 每个特征/关键字之间要用空格隔开，切记                        |
  | @Annotation       | 注解类的完整路径，没有则不写                                 |
  | java修饰符        | private、public、protected以及static和final,没有则不写       |
  | 返回值类型        | 如果不限定类型，使用通配符*表示                              |
  | 类名.方法名       | 使用注解时可以省略类名<br/>方法名可使用通配符<br/>ConstructorSignature的方法名只能是new; |
  | 属性类型          | 属性配型，通配符*表示任意类型                                |
  | 类名.属性名       | 可使用通配符                                                 |
  | 参数              | 例:1.(String，..)表示至少有一个参数,第一个类型为String，<br/>后面参数类型及数量不限制，在参数匹配中..表示任意参数类型和数量;<br/>2.(Object...)表示不定数量，且类型为Object<br/>3.(int,String)表示有2个参数，第一个为int类型，第二个为String类型 |

  我们上文中不止一次提到了通配符，那么，我们来看一下通配符的含义：

  | 通配符 | 说明                                  |
  | ------ | ------------------------------------- |
  | *      | 表示除"."以外的任意字符串或类型       |
  | ..     | 表示任意子package或任意参数类型及数量 |
  | +      | 表示匹配指定类型的子类类型            |

  为了加深对pointcuts语法的理解，这里举两个例子：

  execution(public * *(..)  )    表示任何公共方法的执行；

  execution(@java.lang.Deprecated * *(..))  表示任何使用@java.lang.Deprecated注解的方法的执行；

- **Advice**

  Advice(通知)语法比较简单，有5中通知类型请看下表：

  | Advice             | 说明                                                         |
  | ------------------ | ------------------------------------------------------------ |
  | @Before(Pointcut)  | 在JoinPoint方法运行前运行通知(即@Before()修饰的方法）        |
  | @After(Pointcut)   | 在JoinPoint方法运行后运行通知(即@After()修饰的方法)，无论其结果如何 |
  | @Around(Ponintcut) | 替代原来的代码，如果要执行原来的代码需要使用<br/>ProceedingJoinPoint.proceed()<br/>注：不支持和@Before()、@After()等一起使用 |
  | @AfterReturning    | @AfterReturning(pointcut="pointcut语法",returning="retValue")<br/>在JoinPoint方法执行成功后运行通知(@AfterReturning修饰的方法)<br/>returning表示返回的变量 |
  | @AfterThrowing     | @AfterThrowing(pointcut="pointcut语法",throwing="error")<br/>`@AfterThrowing`是一种通知类型，只有在方法通过抛出异常<br/>而退出方法之后才能运行通知(@AfterReturning修饰的方法)。<br/>throwing表示返回的异常信息 |

  

### 三、AspectJ实战

AspectJ有2种实现方式：

一种是入侵式，即引入技术(框架)会改变原有代码的结构(原有代码需要做相应的修改)，比如基于注解实现的AOP框架；

一种是非入侵式，即引入技术(框架)不会改变原有代码的结构(原有代码不需要做任何修改)，比如使用pointcuts语法匹配应用的生命周期方法或系统事件

入侵式会让用户代码产生对框架的依赖，这些代码不能在框架外使用，不利于代码的复用（缺点）。但入侵式可以使用户跟框架更好的结合，更容易更充分的利用框架提供的功能（优点）。 

非入侵式的代码则没有过多的依赖，可以很方便的迁移到其他地方（优点）。但是与用户代码互动的方式可能就比较复杂（缺点）。 

#### 防抖动拦截实战

1. 切面类

   ```java
   @Aspect
   public class AntiShakeAspect {
       private Long lastClickTime = 0L;
       private final Long FILTER_TIMEM = 1000L;
       /**
        * 根据pointcut语法寻找符合要求的jpoint
        */
       //MethodSignature语法规则 @Annotation java修饰符 返回值类型 类名.方法名(参数)
       @Pointcut("execution(@cn.rjgc.aoptest.annotation.AntiShake * *(..))")
       public void findJPoint() {
   
       }
   
       /**
        * 在符合findJPoint()要求的连接点方法运行时运行此通知
        * @param joinPoint 使用@Around注解必须带着此参数
        */
       @Around("findJPoint()")
       public void antiShake(ProceedingJoinPoint joinPoint) {
           if (System.currentTimeMillis() - lastClickTime >= FILTER_TIMEM) {
               lastClickTime = System.currentTimeMillis();
               try {
                   joinPoint.proceed();//执行Join Point方法的代码
               } catch (Throwable throwable) {
                   throwable.printStackTrace();
               }
           } else {
               Log.e("AntiShakeAspect","重复点击,已过滤");
           }
       }
   }
   ```

   

2. 注解

   ```java
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface AntiShake {
   
   }
   ```

   

3. 客户端测试类

   ```java
   public class MainActivity extends AppCompatActivity {
   
       private static final String TAG = "MainActivity";
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
       }
   
       //防抖动测试
       @AntiShake
       public void antiShake(View view) {
           Log.e(TAG, "按钮被点击了" );
       }
   }
   
   ```

源码已上传，欢迎star  [AspectJ防抖动实战](https://github.com/Don-Lee/AopDemo)

