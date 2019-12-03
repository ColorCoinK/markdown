---
title: Java设计模式
toc: true
layout: blog
categories:
  - Blog
tags:
  - Blog
date: 2019-09-04 14:45:50
---
设计模式代表有经验的面向对象软件开发人员使用的最佳实践。设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是由许多软件开发人员在相当长的时间内通过试错获得的。设计模式的目的是为了让软件具有更好的代码重用性、可读性、可扩展性、可靠性,同时使程序实现高内聚、低耦合。
<!-- more -->
## 模式类型

* 在《设计模式-可重用的面向对象软件元素》一书中,阐明一共有`23`中设计模式。按三类可分为:创造型模式(Creational Patterns)、结构型模式(Structural Patterns)、行为型模式(Behavioral Patterns)。其中还有另一类设计模式: J2EE设计模式。

| 序号 | 分类名称   | 说明                                                                                                                                            | 包括                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | 创造型模式 | 创造模式提供了一种创建对象而隐藏创建逻辑的方法,并不是直接使用`new`实例化对象。<br/>使程序在决定对于给定的用例需要创建哪些对象时具有更大的灵活性 | 工厂模式(Factory Pattern) <br/>抽象工厂模式(Abstract Factory Pattern) <br/> 单例模式(Singleton Pattern)<br/> 建造者模式(Builder Pattern) <br/>原型模式(Prototype Pattern)                                                                                                                                                                                                                                           |
| 2    | 结构型模式 | 结构型模式关注类和对象的组合。<br/>集成的概念被用来组合接口和定义组合对象获得新的功能                                                           | 适配器模式(Adapter Pattern)<br/> 桥接模式(Bridge Pattern)<br/>过滤器模式(Filter、Criteria Pattern)<br/>组合模式(Composite Pattern)<br/>装饰器模式(Decorator Pattern)<br/>外观模式(Facade Pattern)<br/>享元模式(Flyweight Pattern)<br/>代理模式(Proxy Pattern)                                                                                                                                                       |
| 3    | 行为型模式 | 行为型模式特别关注对象之间的通信                                                                                                                | 责任链模式(Chain of Responsibility Pattern)<br/>命令模式(Command Pattern)<br/>解释器模式(Interpreter Pattern)<br/>迭代器模式(Iterator Pattern)<br/>中介者模式(Mediator Pattern)<br/>备忘录模式(Memento Pattern)<br/>观察者模式(Observer Pattern)<br/>状态模式(State Pattern)<br/>空对象模式(Null Object Pattern)<br/>策略模式(Strategy Pattern)<br/>模板模式(Template Pattern)<br/>访问者模式(Visitor Pattern)<br/> |
| 4    | J2EE模式   | J2EE模式特别关注表示层,是由Sun Java Center 鉴定的。                                                                                             | MVC模式(MVC Pattern)<br/>业务代表模式(Business Delegate Pattern)<br/>组合实体模式(Composite Entity Pattern)<br/>数据访问对象模式(Data Access Object Pattern)<br/>前端控制器模式(Front Controller Pattern)<br/>拦截过滤器模式(Intercepting Filter Pattern)<br/>服务定位器模式(Service Locator Pattern)<br/>传输对象模式(Transfer Object Pattern)                                                                     |

## 设计原则

* 提倡`Design Pattern`原因: 实现代码复用,增加可维护性。

> 单一职责原则(Single Responsibility Principle SRP)

注: 详细解释参照百科<a href="https://baike.baidu.com/item/单一职责原则" target="_blank">单一职责原则</a>

* 说明: 单一职责原则又称单一功能原则,面向对象五个基本原则(SOLID)之一。
* 定义: 一个类,只有一个引起它变化的原因。
* 解释: 所谓职责就是指类变化的原因。如果一个类有多余一个的动机被改变,那么这个类就具有多余一个的职责。而单一职责原则就是指一个类应该有且仅有一个改变的原因。
* 解决的问题:
  * 降低类的复杂度
  * 提高类的可读性,提高系统的可维护性
  * 降低变更引起的风险(降低对其他功能的影响)

> 开闭原则(Open Close Principle OCP)

* 说明: 在面向对象编程领域中，开闭原则规定`软件中的对象(类、模块、函数等)应该对于扩展是开放的,但是对于修改时封闭的。`
* 解释: 在程序需要进行扩展的时候,不能去修改原有的代码,实现一个热插拔的效果。简言之,是为了是程序的扩展性好,易于维护和升级。
* 实现:

> 里氏代换原则(Liskov Substitution Principle LSP)

* 定义: 任何基类可以出现的地方,子类一定可以出现。
* 解释: 
  LSP是继承复用的基石,只有当衍生类可以替换掉基类,软件单位的功能不受到影响时,基类才能真正被复用,而衍生类也能够给在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。
* 解决的问题:
  * 增强程序的健壮性,版本升级时也可以保持非常好的兼容性
  * 提高代码复用率

> 依赖倒转原则(Dependence Inversion Principle DIP)

> 接口隔离原则(Interface Segregation Principle ISP)

> 合成/聚合复用原则(Composite/Aggregate Reuse Principle CARP)

> 最小知识原则(Principle of Least Knowledge PLK,迪米特法则)
