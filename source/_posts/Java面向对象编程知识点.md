---
title: Java面向对象编程知识点
date: 2019-02-20 10:40:58
tags: 
    - Java
    - 面向对象
categories: 面试
---

## 面向对象的三个基本特征和五种设计原则

### 三个基本特征

#### 封装
封装就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏；
#### 继承
继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重写原有类的情况下对这些功能进行扩展。通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“父类”或“基类”。继承的过程就是从一般到特殊的过程。  
要实现继承可以通过“继承”和“组合”两种方式实现。在某些OOP语言中，一个子类可以继承多个基类，但一般情况下（Java语言），一个子类只有一个基类，实现多重继承可以通过多级继承来实现。 
<!-- more --> 
继承概念的实现方式有三类：  

- 实现继承：指使用基类的属性和方法而无需额外编码的能力；
- 接口继承：指仅使用基类的属性和方法名称，但子类必须提供实现的能力；
- 可视继承：指子类使用基类的外观和实现代码的能力。
#### 多态

多态是允许你将父对象设置成为和一个或多个它的子对象相等的技术，赋值之后，父对象就可以根据当前赋值给它的子对象的特性以不同方式运作。简单地说就是：允许将子类类型的指针赋值给父类类型的指针。  
实现多态有两种方式：覆盖和重载：  

- 覆盖是指子类重新定义父类的虚函数的做法；
- 重载是指允许多个同名函数，这些函数的参数不同（参数个数不同，或参数类型不同，或两者都不同）。

### 五种设计原则

#### 单一职责原则(Single Responsibility Principle，SRP)

单一职责原则是指一个类的功能要单一，不能包罗万象。一个类应该只有一个引起它变化的原因。

#### 开放封闭原则(Open－Close Principle，OCP)

 开放封闭原则指的是对扩展性的开放和对修改的封闭：
 - 对扩展性的开放：模块的行为应该是可扩展的，从而该模块可表现出行的行为以满足需求的变化；
 - 对修改的封闭：模块自身的代码时不应该被修改的，扩展模块的一般途径是修改内部实现。

 #### 里氏替换原则(the Liskov Substitution Principle，LSP) 

 子类型必须能够替换掉它们的父类型，并出现在父类能够出现的任何地方。


#### 依赖倒置原则(the Dependency Inversion Principle，PIP)

具体依赖抽象，上层依赖下层。

#### 接口隔离原则(the Interface Segregation Principle，ISP)

模块间要通过抽象接口隔离开，而不是通过具体的类强耦合起来。


## `Class.java`类文件下`getMethods()`和`getDeclaredMethods()`的区别

```
@CallerSensitive
public Method[] getMethods() throws SecurityException {
    checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
    return copyMethods(privateGetPublicMethods());
}

@CallerSensitive
public Method[] getDeclaredMethods() throws SecurityException {
    checkMemberAccess(Member.DECLARED, Reflection.getCallerClass(), true);
    return copyMethods(privateGetDeclaredMethods(false));
}
```
`getMethods()`是获取对应类及其所有父类中的共有（public）方法；`getDeclaredMethods()`获取的是当前类中的所有方法。
