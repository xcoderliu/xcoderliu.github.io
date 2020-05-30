---
layout: post
title: Flutter InheritedWidget
tag: Flutter
date: 2020-05-30 20:45:00.000000000 +09:00
---

### 赘述

学习 Flutter 之后有两个很重要的概念，StatefulWidget 和 StatelessWidget，其中 Stateful 其实指的就是需要要数据的 UI 并且会根据数据进行调整，也是程序开发中最为常见的状态，比较数据驱动 UI 显示。Stateless 是不需要记录状态的 Widget，一个静态的不需要根据数据做调整的 UI。而今天讲到另一个类型的 Widget，Inherited Widget 他并不能以大多数面向对象编程语言中的继承来理解。

### InheritedWidget 解决了什么？

当我们写好了一个父 StatefulWidget 的容器时，我们的子 Widget 很可能需要根据父视图的一些数据做出不同的渲染，这时候我们需要通过 InheritedWidget 将我们的数据传递到下层 Widget。总而言之，InheritedWidget 允许在 widget 树中有效地向下传播（和共享）信息。

### 实现 & 用法

InheritedWidget 通常需要名为 of 的静态方法，让所以有子 Widget 通过包含的 context 获得最近的 InheritedWidget 实例。updateShouldNotify 当我们的数据变化是否需要通知到子 Widget。子 Widget 中用法 ：InheritedWidget.of(context).xxx 。需要注意的是许多 InheritedWidget 是经常在整个 app 生命周期中保持不变的，这本身没什么问题，实际上我们定义的使用用 final 关键字，仅仅以为着这个值不可被重新赋值，但并不是说内部不可以改变。你可以定义一个 final 的 service 。举个例子，它可以是你封装好的数据库service、一个 web API 的代理、或者 assets 的 provider 等等。这些 service 可以包含自己内部的改变，所以这些子控件在可以通过 of 访问到 service 这样一个内部是动态的 object。


