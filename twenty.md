---
title: Thinking in Java 第二十章
date: 2020-03-01 13:38:07
tags:
categories: Java
---
# 第二十章 注解
注解（元数据）为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据。

- @Override，表示当前的方法定义将覆盖超类中的方法。
- @Deprecated（废弃的），如果程序员使用了注解为它的元素，那么编译器会发出警告信息。
- @SuppressWarnings，关闭不当的编译器警告信息。

## 20.1 基本语法

注解的方法与其他方法没有区别，例如public、static、void 从语法的角度来看，注解的使用方式几乎与修饰符的使用一模一样。
```
@Test void testExecute();
```
#### 20.1.1 定义注解

与其他任何Java接口一样，注解也将会编译成class文件。

#### 20.1.2 元注解
元注解专职负责注解其他的注解：

<table>
<thead>
<tr>
<th>元注解</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code><a href="https://github.com/Target" title="@Target" class="at-link" target="_blank">@Target</a></code></td>
<td>表示该注解可以用于什么地方。可能的<code>ElementType</code>包括：<br><code>CONSTRUCTOR</code>: 构造器的声明<br><code>FIELD</code>: 域声明（包括<code>enum</code>实例）<br><code>LOCAL_VARIABLE</code>: 局部变量声明<br><code>METHOD</code>: 方法声明<br><code>PACKAGE</code>: 包声明 <br><code>PARAMETER</code>: 参数声明 <br><code>TYPE</code>: 类、接口（）包括注解类型) 或<code>enum</code>声明</td>
</tr>
<tr>
<td><code><a href="https://github.com/Retention" title="@Retention" class="at-link" target="_blank">@Retention</a></code></td>
<td>表示需要在什么级别保存该注解信息。可选的<code>RetentionPolicy</code>参数包括：<br><code>SOURCE</code>: 注解将被编译器丢弃<br><code>CLASS</code>: 注解在<code>class</code>文件中可用，但会被虚拟机丢弃。<br><code>RUNTIME</code>: 虚拟机将在运行期也保留注解，因此可以通过反射机制读取注解的信息。</td>
</tr>
<tr>
<td><code><a href="https://github.com/Documented" title="@Documented" class="at-link" target="_blank">@Documented</a></code></td>
<td>将此注解包含在Javadoc中。</td>
</tr>
<tr>
<td><code><a href="https://github.com/Inherited" title="@Inherited" class="at-link" target="_blank">@Inherited</a></code></td>
<td>允许子类继承父类的注解。</td>
</tr>
</tbody>
</table>

## 20.2 编写注解处理器

#### 20.2.1 注解元素

注解元素可用的类型：

1. 所有基本数据类型（int, float, boolean 等）
2. String
3. Class
4. enum
5. Annotation
6. 以上类型的数组

使用其他类型编译器就会报错，也不允许使用任何包装类型。
注解也可以作为元素的类型，也就是说注解可以嵌套。
```
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
public class Annotation {
    public static void trackUseCases(List<Integer> useCases, Class<?> cl) {
        for (Method m : cl.getDeclaredMethods()) {
            UseCase uc = m.getAnnotation(UseCase.class);
            if (uc != null) {
                System.out.println("Found Use Case:" + uc.id() + " " + uc.description());
                useCases.remove(new Integer(uc.id()));
            }
        }
        for (int i : useCases) {
            System.out.println("Warning: Missing use case-" + i);
        }
    }
    public static void main(String[] args) {
        List<Integer> useCases = new ArrayList<Integer>();
        Collections.addAll(useCases, 47, 48, 49, 50);
        trackUseCases(useCases, PasswordUtils.class);
      PasswordUtils util=new PasswordUtils();
        try {
            Method m=util.getClass().getMethod("validatePassword", String.class);
            if(!((boolean)m.invoke(util, "test"))){
                UseCase uc=m.getAnnotation(UseCase.class);
                System.out.println(uc.description());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
class PasswordUtils {
    @UseCase(id = 47, description = "Passwords must contain at least one numeric")
    public boolean validatePassword(String password) {
        return (password.matches("\\w*\\d\\w*"));
    }
    @UseCase(id = 48)
    public String encryptPassword(String password) {
        return new StringBuilder(password).reverse().toString();
    }
    @UseCase(id = 49, description = "New passwords can’t equal previously used ones")
    public boolean checkForNewPassword(List<String> prevPasswords,
            String password) {
        return !prevPasswords.contains(password);
    }
}
```
```
import java.lang.annotation.*;
import java.lang.annotation.Target;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    public int id() default -1;
    public String description() default "default description";
}
```

#### 20.2.2 默认值限制

元素必须要么具有默认值，要么使用注解时提供的值。

对于非基本类型的元素，无论是在源代码中声明时，或是在注解接口中定义默认值，都不能以null作为其值。

注解快捷方式：如果注解元素声明为value()，则在使用注解时如果只声明value，可以只写值，不必写名值对。例如可写为@Test(10)

#### 20.2.3 生成外部文件

变通之道
同时使用两个注解类型来注解一个域。 

#### 20.2.4 注解不支持继承

不能使用extends来继承某个@interface。算是遗憾，可以嵌套解决。

#### 20.2.5 实现处理器

在编译时通过注解获取相关数据，然后处理操作。

## 20.3 使用apt处理注解

自定义的每个注解都需要自己的处理器，而apt工具能很容易地将多个注解处理器组合在一起。
（java提供了一个名为AbstractProcessor.java的抽象类，我们只要继承该类，就实现自己的注解处理器，来处理自定义的注解，书中是自己实现了一个apt）

## 20.4 将观察者模式用于apt

用观察者模式 一对多处理多个注解。
## 20.5 基于注解的单元测试

有了注解，可以直接在要验证的类里面编写测试,方便快捷。

#### 20.5.1 将@Unit用于泛型

测试类继承自泛型，来测试堆栈里的数据和类型。唯一缺点是继承让我们无法访问private方法，要么修改他的访问权限修饰符。要么就在类中添加@TestProperty 由他来调用方法。例如：
```
public class Ex8 {
	private String methodOne() { return "methodOne"; }
	@TestProperty protected String methodOneCall() {
         return this.methodOne(); }
 }

```
#### 20.5.2 不需要任何套件

@Uint 只是简单的搜索类文件，检查是否具有恰当的注解，然后运行@Test方法。
在@JUnit中，程序员必须告诉测试工具你打算测试什么，这就要求用套件来组织测试，以便JUnit能够找到它们，并运行包含的测试。

##### 20.5.3 实现@Unit

##### 20.5.4 移除测试代码

开源Javassist工具库。

## 20.6 总结
注解是Java引入的一项非常受欢迎的补充，从而似的程序员为代码加入元数据，不会导致代码杂乱。而Javadoc 中的@deprecated被@Deprecated注解取代的事实也说明，与注释文字相比，注解更适合买哦书类的相关信息。
有了apt工具可以简化构建过程，Javassist能操作字节码的工具。
不过现在的framework一定会将注解包含在其提供的工具集内，注解会将Java编程体验提升。

