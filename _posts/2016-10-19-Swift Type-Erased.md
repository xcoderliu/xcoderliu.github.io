---
layout: post
title: Swift Type-Erased(类型擦除)
tag: iOS
date: 2016-10-19 18:19:00.000000000 +09:00
---

泛型编程对现代软件开发有着不可忽视的作用，我们希望在 protocol 中加入 **associated types** ，但是令人不愉快的是编译器似乎总是闹别扭。 在 swift 中 **类型擦除** 是一种不错地将 **associated types** 转化为   **generic constraints** 的方法。

面向协议的编程在 swift 中是非常提倡的对吧？ 我的意思是甚至是在 WWDC 会议期间，人们呐喊出："Protocols are awesome!"。他们让你定义接口，允许你忽略那些接口的实现。在 Swift 的世界中，如果我们将协议称之为国王，那么泛型则可以视作皇后，所谓一山不容二虎，当我们把这两者结合起来使用的时候，似乎会遇到极大的困难。那么是否有一种方法，能够将这两个概念结合在一起，以便让它们成为我们前进道路上的垫脚石，而不是碍手碍脚的呢？答案是有的，这里我们将会使用到类型擦除 (Type Erasure) 这个强大的特性。

### 具体类型与抽象类型

当你开始学习一门编程语言的时候，你很有可能是从 string、int 或者 float 开始学习的：这些都是所谓的「具体类型 (concrete type)」，也就是编译器能够在编译时确定此类型所占的空间大小——这使得编译器对具体类型的处理非常友好。如果某个类型可以被初始化——也就是说您可以调用它的初始化方法——那么就说明它是一个具体类型。

另外一种类型就是「抽象类型 (abstract type)」( 也被称为存在类型 (existential) )。对于抽象类型来说，编译器无法知道这个类型的确切功能。当编译器处理抽象类型的时候，它无法知晓其所占的空间大小；甚至可能会认为这个类型是不存在的。事实上，您很可能见到过这样的错误描述「我无法为您找寻到此类型」。

在 Swift 当中，抽象类型的普遍表述方式使用 associatedType 来描述的，之前我们是使用 typealias 来定义的，此外泛型 <T> 同样也是一个抽象类型。如果编译器无法在编译时知晓类型的所占空间大小，或者您无法将其初始化，也就是说您没办法调用它的 init 的方法：那么就说明这个类型很可能是抽象类型。

### 什么是 Swift 中的类型擦除？

有很多地方会用到类型擦除，并且它们的作用的各不相同。在 Java 当中，它用来让编译器知晓其相关内容，它会抹除 <T> 的存在，然后用一个具体类型将其替代。在 Swift 当中，我们不必处理这些琐事，我们可以自行实现——有很多您可以选择的实现方式。对于类型擦除来说，绝大多数语言当中的基本概念是相同的，但是在 Swift 当中，它解决了这个问题。

### 遇到的问题

假设我们简单的定义一个口袋妖怪的协议：

```objc

protocol Pokemon {
    associatedtype Power //每个口袋妖怪都有自己的属性 但是属性是一个抽象的类 （可以是🔥 ⚡️...）
    func attack() -> Power //攻击
}

//电属性的皮卡丘
struct Pikachu: Pokemon {
    func attack() -> 🌩 {
        return 🌩()
    }
}

//火属性的喷火龙
struct Charmander: Pokemon {
    func attack() -> 🔥 {
        return 🔥()
    }
}

// 属性类型
struct 🔥 { }
struct 🌩 { }

```
看起来好像没啥问题，接下来我们开始让口袋妖怪攻击吧，

```objc
let pokemon: Pokemon = Pikachu()
pokemon.attack()
```

Oh,fuck 编译器报错了！

![failedImg](https://www.natashatherobot.com/wp-content/uploads/Screen-Shot-2016-05-30-at-12.30.25-PM-768x50.png)

### 解决办法

这时候就需要 **类型擦除** 了，我们想用一个抽象的 Pokemon 类型 来代替 Pikachu 类型：

```objc
struct AnyPokemon<Power>: Pokemon {
    
    // 我们需要保存 properties 和 functions
    // Pokemon protocol 将会用到
    private let _attack: () -> Power
    
    init<P: Pokemon where P.Power == Power>(_ pokemon: P) {
        _attack = pokemon.attack
    }
    
    func attack() -> Power {
        // using the original Pokemon's attack implementation
        return _attack()
    }
}
 
let firePowerPokemon = AnyPokemon<🔥>(Charmander())
// 现在我们丢失了喷火龙的属性信息
// firePowerPokemon 可以是任何火属性的口袋妖怪
firePowerPokemon.attack() //🔥

```

### 解决方法升级

现在还有一个更简单的方式来解决 **associated types** 的问题：

```objc
struct AnyPokemon<P: Pokemon>: Pokemon {
    
    private let pokemon: P
    
    init(_ pokemon: P) {
        self.pokemon = pokemon
    }
    
    func attack() -> P.Power {
        return pokemon.attack()
    }
}
 
// 使用
let pokemon = AnyPokemon(Pikachu())
pokemon.attack()
```
但很可惜，这不是 **类型擦除** 。

参考文章地址：https://realm.io/news/altconf-hector-matos-type-erasure-magic/

http://krakendev.io/blog/generic-protocols-and-their-shortcomings