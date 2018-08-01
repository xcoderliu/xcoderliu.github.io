---
layout: post
title: CALayer 动画学习（译）
tag: iOS translate
date: 2016-08-18 15:04:24.000000000 +09:00
---

首先贴上原文地址：https://medium.com/ios-os-x-development/effervescent-calayer-transfigurations-on-ios-c5e78781db17#.o654x27jz

然后是对应的ios code：https://github.com/xcoderliu/CALayerAnimationStudy.git

差不多一个星期之前, 我随意浏览了一下 [Dribbble](https://dribbble.com/) （碰巧是我最爱的网站之一）。我有时候会到这里浏览，当我百般聊奈的等着持续集成的成功 (或者失败), 就在这一天我看到一个很讨人喜欢的 gif 形式的加载指示器动画， 在当前的网页中非常显眼。 它看起来就是这个样子的:

 ![CALayerAnimation](https://cdn-images-1.medium.com/max/800/1*qBpM9ACmF4pRp5z-UiDb_A.gif) 旋转 漂亮的旋转

可惜的是，在接下来的日子里我一次都找不到像这样让我陷入震撼的动画原型。 它对我的好奇心有着非常深刻的影响, 我可以在我脑海中清晰地就好像我盯着眼前浏览器中的图片一样构建出它。在有一次我差点找不到它的时候，我决定将我大脑中存储的不论是以前想象中或者是亲眼所见的（我现在都不确定我亲眼见过），转化成一个教程给大家。

不用说，本教程中有很多这样甜蜜的GUI糖， 它应该被认为是某种危险的 - 或者至少是不安全的**放纵**。 👍

## Part 1 — CALayer的不足

接下来创建一个新的 Xcode 项目。 应该选择 Single-view application 模板, 同时记得选择 **swift** 作为开发语言。 不要在意那些什么选择是否使用 Core Data 之类乱七八糟的复选框, 因为我们现在不需要这些。

现在, 创建一个新的 Swift 文件。按照你喜欢的名字命名。 这个文件应该是一个 **CALayer** 的**子类**。

首先我将要添加一个自定义的初始化函数。 这是因为我们要为这个堆积的效果提供一系列的物体, 所以最好添加这类的东西在初始化函数中。 此外，我将添加一个变量来保存底色，和一个容纳每个堆栈物体的大小。

每当你将子 layer 插入到父 layer 中，想一下它们作为子视图或者反过来看如果这对你来说更容易理解，- layer 将会在动画中被裁减因为父 layer 只有一定规模的大小。这看起来不是很好，它破坏了很多动画，所以我也将它禁用了。

为了让这些看起来更加简洁明了 — 说真的，我可以花一整天的时间 — 我将会添加一个循环在刚写的 init 方法中。而不是花6行代码来描述形状, 我会将它放入到一个方法中让我的循环看起来更具有可读性。
这是我一个早期的例子:

```objc
//
//  LZMLayer.swift
//  CALayerTest
//
//  Created by liuzhimin on 17/08/2016.
//  Copyright © 2016 liuzhimin. All rights reserved.
//

import UIKit

class LZMLayer: CATransformLayer {
    
    var color: UIColor = UIColor.white {
        didSet {
            guard let sublayers = sublayers , sublayers.count > 0 else { return }
            for (index, layer) in sublayers.enumerated() {
                (layer as? CAShapeLayer)?.fillColor = color.set(hueSaturationOrBrightness: .Brightness, percentage: 1.0-(0.1*CGFloat(index))).cgColor
            }
        }
    }
    
    /* 之后如果调整layer的大小，堆栈将会重新绘制.
     *
     * 假设长宽相等，圆角的半径被计算为宽度的四分之一
     * 默认大小是100X100
     */
    
    var size: CGSize = CGSize(width: 100, height: 100) {
        didSet {
            sublayers?.forEach({
                ($0 as? CAShapeLayer)?.path = UIBezierPath(roundedRect: CGRect(x: 0, y: 0, width: size.width, height: size.height), cornerRadius: size.width/4).cgPath
                ($0 as? CAShapeLayer)?.frame = (($0 as? CAShapeLayer)?.path)!.boundingBox
                setAnchorPoint(anchorPoint: CGPoint(x: 0.5, y: 0.5), forLayer: $0)
            })
        }
    }
    
    convenience init(withNumberOfItems items: Int) {
        self.init()
        masksToBounds = false
        
        /* 循环添加子图层 */
        
        for i in 0..<items {
            let layer = generateLayer(withSize: size, withIndex: i)
            insertSublayer(layer, at: 0)
            setZPosition(ofShape: layer, z: CGFloat(i))
        }
        
        /*为了颜色是自上而下变得更深*/
        
        sublayers = sublayers?.reversed()
        
        /*居中图层*/
        centerInSuperlayer()
        
        /*旋转自身图层3D z轴*/
        rotateParentLayer(toDegree : 60.0)
    }
    
    
    private func generateLayer(withSize size: CGSize, withIndex index: Int) -> CAShapeLayer {
        let square = CAShapeLayer()
        square.path = UIBezierPath(roundedRect: CGRect(x: 0, y: 0, width: size.width, height: size.height), cornerRadius: size.width/4).cgPath
        square.frame = square.path!.boundingBox
        /*设置中心点为锚点 同时计算出新的位置*/
        setAnchorPoint(anchorPoint: CGPoint(x: 0.5, y: 0.5), forLayer: square)
        return square
    }
    
    // Because adjusting the anchorPoint itself adjusts the frame, this is needed to avoid it, and keep the layer stationary.
    
    private func setAnchorPoint(anchorPoint: CGPoint, forLayer layer: CALayer) {
        var newPoint = CGPoint(x: layer.bounds.size.width * anchorPoint.x, y: layer.bounds.size.height * anchorPoint.y)
        var oldPoint = CGPoint(x: layer.bounds.size.width * layer.anchorPoint.x, y: layer.bounds.size.height * layer.anchorPoint.y)
        newPoint = newPoint.applying(layer.affineTransform())
        oldPoint = oldPoint.applying(layer.affineTransform())
        
        var position = layer.position
        position.x -= oldPoint.x
        position.x += newPoint.x
        position.y -= oldPoint.y
        position.y += newPoint.y
        
        layer.position = position
        layer.anchorPoint = anchorPoint
    }
    
    private func setZPosition(ofShape shape: CAShapeLayer, z: CGFloat) {
        shape.zPosition = z*(-20)
    }
    
    private func centerInSuperlayer() {
        frame = CGRect(x: getX(), y: getY(), width: size.width, height: size.height)
    }
    
    private func getX() -> CGFloat {
        let screenWidth = UIScreen.main.bounds.size.width
        return (screenWidth/2)-(size.width/2)
    }
    
    private func getY() -> CGFloat {
        let screenHeight = UIScreen.main.bounds.size.height
        return (screenHeight/2)-(2*(size.height/2))
    }
    
    // When the time comes to animate, we'll need this. It converts...well...degrees into radians..
    
    private func degreesToRadians(degrees: CGFloat) -> CGFloat {
        return ((CGFloat(M_PI) * degrees) / 180.0)
    }
}

```

是否你注意到了，在刚添加的颜色变量中我们调用了一个还没有实现的方法。 因此我们要稍稍修改一些 **UIColor** 通过色调或者饱和度来设置亮度。不过这都不是重点，重点是我们要形成这样一个形状。保存下面的文件等下我们会用到:

```objc
//
//  UIColor+HexHSB.swift
//  CALayerTest
//
//  Created by liuzhimin on 17/08/2016.
//  Copyright © 2016 liuzhimin. All rights reserved.
//

import UIKit

enum UIColorInputError : Error {
    case MissingHashMarkAsPrefix,
    UnableToScanHexValue,
    MismatchedHexStringLength
}

extension UIColor {
    
    convenience init(hex: UInt) {
        self.init(
            red: CGFloat((hex & 0xFF0000) >> 16) / 255.0,
            green: CGFloat((hex & 0x00FF00) >> 8) / 255.0,
            blue: CGFloat(hex & 0x0000FF) / 255.0,
            alpha: CGFloat(1.0)
        )
    }
    
    convenience init(rgba: String, defaultColor: UIColor = UIColor.clear) {
        guard let color = try? UIColor(rgba_throws: rgba) else {
            self.init(cgColor :defaultColor.cgColor)
            return
        }
        self.init(cgColor: color.cgColor)
    }
    
    convenience init(rgba_throws rgba: String) throws {
        guard rgba.hasPrefix("#") else {
            throw UIColorInputError.MissingHashMarkAsPrefix
        }
        guard let hexString: String = rgba.substring(to: rgba.index(rgba.startIndex, offsetBy: 1)),
            var   hexValue:  UInt32 = 0
            , Scanner(string: hexString).scanHexInt32(&hexValue) else {
                throw UIColorInputError.UnableToScanHexValue
        }
        
        guard hexString.characters.count  == 3
            || hexString.characters.count == 4
            || hexString.characters.count == 6
            || hexString.characters.count == 8 else {
                throw UIColorInputError.MismatchedHexStringLength
        }
        
        switch (hexString.characters.count) {
        case 6:
            self.init(hex6: hexValue)
        default:
            self.init(hex6: hexValue)
        }
    }
    
    convenience init(hex6: UInt32, alpha: CGFloat = 1) {
        let divisor = CGFloat(255)
        let red     = CGFloat((hex6 & 0xFF0000) >> 16) / divisor
        let green   = CGFloat((hex6 & 0x00FF00) >>  8) / divisor
        let blue    = CGFloat( hex6 & 0x0000FF       ) / divisor
        self.init(red: red, green: green, blue: blue, alpha: alpha)
    }
    
    func set(hueSaturationOrBrightness hsb: HSBA, percentage: CGFloat) -> UIColor {
        var hueValue : CGFloat = 0.0, saturationValue : CGFloat = 0.0, brightnessValue : CGFloat = 0.0, alphaValue : CGFloat = 0.0
        self.getHue(&hueValue, saturation: &saturationValue, brightness: &brightnessValue, alpha: &alphaValue)
        
        switch hsb {
        case .Hue:
            print(hueValue)
            return UIColor(hue: hueValue * percentage, saturation: saturationValue, brightness: brightnessValue, alpha: alphaValue)
        case .Saturation:
            return UIColor(hue: hueValue, saturation: saturationValue * percentage, brightness: brightnessValue, alpha: alphaValue)
        case .Brightness:
            return UIColor(hue: hueValue, saturation: saturationValue, brightness: brightnessValue * percentage, alpha: alphaValue)
        case .Alpha:
            return UIColor(hue: hueValue, saturation: saturationValue, brightness: brightnessValue, alpha: alphaValue * percentage)
        }
    }
}

enum HSBA {
    case Hue
    case Saturation
    case Brightness
    case Alpha
}

```

UIColor+HexHSB — 用十六进制来设定你的颜色以及色彩饱和度。

尝试着在色板中找到同一种颜色的不同渐变 — 坦白说 — 太蛋疼了，除非你是一个有双氪金狗眼的设计师。 有了这么一个小小的扩展之后, 你可以利用它轻松的在你的app需求下创建一个基于某个颜色较暗或者较亮的渐变。

好像对我来说好像一直挺有用?

## 打断一下

好了, **CALayer** and **CAShapeLayer** 是两个非常强大而且令人迷惑的 [CoreAnimation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html) 的组成部分。接下来我会给一个到目前为止实现的一个简单整理:

- 创建一个 **CALayer** 的子类
- 写一个 **convenience initialiser** 包含绘制一些形状
- 添加一个变量保存一个基础颜色, 物体 以及父 layer 的大小
- 写一个循环在 **parent layer** 中创建摆放和移动每一个 **shape layer** 
- 在 layer 中填充颜色
- sublayer 数组逆序
- 在父视图中居住 parent layer
- 每个 shape layer 都调整锚点
- 转化角度为弧度

逆序子图层这一点是非常重要的, 由于我们这种呈现栈的方式。 任何在运行中的修改, 比如调整每个 Layer 在 Z 轴之间的距离，这个效果将会被破坏如果我们没有将数组逆序。

**CAShapeLayer** 是 **CALayer** 的一种它需要填充色和路径。 我们通过修改这些属性来绘制我们想要在屏幕上显示的样子 。

每当我们修改一个 layer 的锚点, layer 的 frame 将会有变化。它不总是能达到我们想要实现的效果, 如果你也尝试修改这个属性, 没准你也得不到你想要的结果。

![anchor point](https://cdn-images-1.medium.com/max/800/1*hkoQA_TnJvYgkplsMBvekw.png)
Home is where the anchor drops（好像是一个电影或者一本书之类的 不知道和插图有啥关系）

默认的情况下，一个 layer 的锚点是（0，0）, 这意味所以的动画执行将要围绕着这个点进行。 为了达到我们想要实现的效果，我们应该将锚点居中。

我希望能居中一个 layer 就像居中一个 UIView 那么简单，哎 事实上我不能。

## Part 2 — Stamp your hind legs, get behind me! Animate!

继续，复制下面的内容到你的类中:

```objc
extension LZMLayer {
    func startAnimating() {
        var offsetTime = 0.0
        var transform = CATransform3DIdentity
        transform.m34 = 1.0 / -500.0
        transform = CATransform3DRotate(transform, CGFloat(M_PI), 0, 0, 1)
        
        /*动画开始*/
        CATransaction.begin()
        sublayers?.forEach({
            let basic = getSpin(forTransform: transform)
            basic.beginTime = $0.convertTime(CACurrentMediaTime(), to: nil) + offsetTime
            $0.add(basic, forKey: nil)
            /*按照index更新启动时间*/
            offsetTime += 0.1
        })
        CATransaction.commit()
    }
    
    func stopAnimating() {
        sublayers?.forEach({ $0.removeAllAnimations() })
    }
    
    /*创建动画*/
    private func getSpin(forTransform transform: CATransform3D) -> CABasicAnimation {
        let basic = CABasicAnimation(keyPath: "transform")
        basic.fromValue = NSValue(caTransform3D: CATransform3DIdentity)
        basic.toValue = NSValue(caTransform3D: transform)
        basic.duration = 1.0
        basic.fillMode = kCAFillModeForwards
        basic.repeatCount = HUGE
        basic.autoreverses = true
        basic.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
        basic.isRemovedOnCompletion = false
        return basic
    }
}

extension FloatingPoint {
    var degreesToRadians: Self { return self * .pi / 180 }
    var radiansToDegrees: Self { return self * 180 / .pi }
}

```

你刚刚扩展了 **LZMLayer**， 顺带一提 — 类带有额外的函数性。 一个函数开始动画, 一个结束动画, 还要一个函数在循环中给每一个 layer 生成 **CABasicAnimation** 。🎉

## 预备

在我们运行这个 app 来演示动画效果之前。 — 给你的主视图控制器添加下面的代码:

```objc
import UIKit

class ViewController: UIViewController {
    let layer = LZMLayer(withNumberOfItems: 6)
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        view.backgroundColor = UIColor.darkGray
        view.layer.addSublayer(layer)
        layer.color = UIColor.white
        spin(sender: nil)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
    }
    
    // Wire these up to two UIButtons on your Storyboard. But for now I'll just call the first above.
    
     func spin(sender: AnyObject?) {
        layer.startAnimating()
    }
    
     func halt(sender: AnyObject) {
        layer.stopAnimating()
    }
}

```

## 打断一下

如果我们运行代码将会看到动画顺利的进行了。🎉

每个 layer 都有着不同的颜色, 制造出来一个非常不错的效果。 但是这个图片好像有些不对劲?

![2d animation](https://cdn-images-1.medium.com/max/800/1*HpFwrkddwcikb8Gu5R-ruA.gif)
哎呦..

你可以看到这些图层的位置是对的, 但是角度却错了。 现在让我们来纠正它， 添加以下的函数:

```objc
extension LZMLayer {
    private func rotateParentLayer(toDegree degree: CGFloat) {
        var transform = CATransform3DIdentity
        transform.m34 = 1.0 / -500.0
        transform = CATransform3DRotate(transform, degree.degreesToRadians, 1, 0, 0)
        self.transform = transform
    }
}
```

在初始化函数的 centerInSuperlayer() 之后调用它, 如下:

```objc
rotateParentLayer(toDegree: 60)
```

它将会把 parent layer 在 x 轴旋转60°,  再一次运行, 我们将会看到…


## Part 3— 牛逼闪闪的 CATransformLayer 

如果你按照上面的修改再运行一次, 最后会变成这个样子:

![stillwrong](https://cdn-images-1.medium.com/max/800/1*r306UUZTvCXPLXMTLP9MNQ.gif)
艹 还是没用

这是因为 CALayer 本身并不能够呈现出深度。 我们已经正确的修改了他们在 z 轴中的位置,但是等到你旋转并且期待着想象中的效果时，你会非常的失望。

不管你怎么去更改栈中这些 layer 的位置，动画始终还是如上图所示。这是因为任何一个在顶部的新的变换都会代替旧的。

这时候就需要引入 **CATransformLayer** 。 这种漂亮的 layer 类型 支持深度的呈现。

为了实现我们最终想要的效果, 仅仅需要改变继承的类 从 **CALayer** 变为 **CATransformLayer**, 然后运行 app。 🎉

![animtionRight](https://cdn-images-1.medium.com/max/800/1*PM8u7LoShI62XroODTyjnQ.gif)
