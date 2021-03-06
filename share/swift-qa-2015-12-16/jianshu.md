[原创] 每周 Swift 社区问答 2015-12-16"



本周整理问题如下：

* [Unwrapping NSNumber works fine in iOS Simulator but unexpectedly found nil on iPhone](#Q1)
* [Why my code is working in playground but not in my project?](#Q2)
* [Failable initialisers and unbound instance vars](#Q3)
* [Read-only property](#Q4)
* [Why? insert a new element into array and it always crash!](#Q5)
* [binary operator '??' cannot be applied to functions?](#Q6)
* [Filter array on type](#Q7) 
* [Numbers in swift](#Q8)

对应的代码都放到了 github 上，有兴趣的同学可以下载下来研究：[点击下载](https://github.com/SwiftGGTeam/SwiftCommunityWeeklyQA/tree/master/20151216/%E6%AF%8F%E5%91%A8%20Swift%20%E7%A4%BE%E5%8C%BA%E9%97%AE%E7%AD%9420151216.playground)



<a name="Q1"></a>

## Q1: Unwrapping NSNumber works fine in iOS Simulator but unexpectedly found nil on iPhone

[Q1链接地址](https://forums.developer.apple.com/thread/27726)



### 问题描述

    
    var stack = Array<String>()  
    stack.append("2.3")  
    let lastElement = stack.popLast()!  
    print("Popped last element: \(lastElement)")  
    let number = NSNumberFormatter().numberFromString(lastElement)  
    print("NSNumber gives us: \(lastElement)")  
    let doubleValue = number!.doubleValue  
    print("Double value of this element is: \(doubleValue)")

上述代码在Playground 以及iOS 模拟器中执行结果如下：

    
    Popped last element: 2.3  
    NSNumber gives us: 2.3  
    Double value of this element is: 2.3

但是在真机里是这样的：

    
    Popped last element: 2.3  
    NSNumber gives us: 2.3  
    fatal error: unexpectedly found nil while unwrapping an Optional value  
    (lldb)

所以提问者修改了一行代码

    
    let doubleValue = number?.doubleValue

再次执行：

    
    Popped last element: 2.3  
    NSNumber gives us: 2.3  
    Double value of this element is: nil

发现解包失败，值为 nil，那么问题出在那里呢？

### 解答

由[junkpile](https://forums.developer.apple.com/people/junkpile)解答：

你的手机所处国家可能对小数分隔符的定义是一个逗号‘,’，而不是句号‘.’ 看到这个回答我也是醉了，最后提问者现身说法，确实他设置的德国是使用逗号作为小数分隔符的，所以解包失败。

junkpile 还给出了一个小建议：

在需要对数字字符串进行格式化的地方，比如输入数字的用户控件，你就需要显式的指定数字格式的本地化属性。反之在接收用户输入的数字时，你应该判断本地化属性，让一切尽在掌握中。



<a name="Q2"></a>

## Q2:Why my code is working in playground but not in my project?

[Q2链接地址](https://forums.developer.apple.com/thread/27764)



### 问题描述

使用 NSDateFormater 解析一个字符串日期，代码如下：

    
    import UIKit  
    let lTs = String("Mon, 07 Dec 2015 3:58 pm EST")  
    let dateFormatter = NSDateFormatter()  
    dateFormatter.dateFormat = "EEE, dd MMM yyyy h:mm a zzz"  
    let lDate = dateFormatter.dateFromString(lTs)  
    print("\(lDate!)")
    // 输出The result is "2015-12-07 20:58:00 +0000\n"

不过当提问者复制这段代码到项目中时（原来在playground）,居然crash掉了，问题出在对`lDate!`解包过程。



### 问题解答

由苹果员工[eskimo](https://forums.developer.apple.com/people/eskimo)解答：

如果是对固定格式的字符串日期解析，你需要确定用户所在地域的local。

增加一行代码：

    
    dateFormatter.locale = NSLocale(localeIdentifier: "en_US_POSIX")

最近 swift.gg 也有一篇文章详解了 NSDate 的正确使用姿势，包含了日期格式的一些知识点，有兴趣的同学可以看看：[Swift 的 NSDate 初学者指南](http://swift.gg/2015/12/14/a-beginners-guide-to-nsdate-in-swift/)

<a name="Q3"></a>

## Q3:Failable initialisers and unbound instance vars



[Q3链接地址](https://forums.developer.apple.com/thread/27743)

### 问题描述

    
    class Foo {  
        let a :Int  
        let b : Int  
    
        init?(name: String, m: Int, n: Int){  
            if name != "fistro" {  
                a = m  
                b = n  
            }else{  
                return nil  
            }  
        }  
    }

编译器报错在 else{} 内并未对 a,b 进行变量初始化，但是其实提问者是想说，既然是一个可失败的构造器，为什么一定要对 a,b进行赋值才能返回 nil 呢？

### 问题解答

[Jessy](https://forums.developer.apple.com/people/Jessy)提供了这么一个方法：

    
    
    class Foo {
    	let a :Int
    	let b : Int
    	
    	init?(name: String, m: Int, n: Int){
    		a = m
    		b = n
    		guard name == "fistro" else { return nil }
    	}
    }

然后[ChrisLattner](https://forums.developer.apple.com/people/ChrisLattner)大神(swift之父)站出来了，明确说了：

这是 Swift 2.1 版本的限制，在即将发布的 Swift 2.2 中已经修复啦。



这里给个stackoverflow类似的问题链接：[http://stackoverflow.com/questions/26495586/best-practice-to-implement-a-failable-initializer-in-swift](http://stackoverflow.com/questions/26495586/best-practice-to-implement-a-failable-initializer-in-swift)

<a name="Q4"></a>

## Q4:Read-only property

### 问题链接

[Q4链接地址](https://forums.developer.apple.com/thread/27892)

### 问题描述

日常项目开发中，我们会遇到一些 Access Control 的问题。譬如，我想要在类中实现一个属性对外是readonly(可读)，对内是write and read(可读写)。那么如何实现是最好的呢？下面提供一个简单的思路。

### 代码实现

    
    class MyClass{
    // 对内可修改属性
    private var gip: Bool = false  
    // 这是一个对外的可读属性
    var gameInProgress: Bool {  
         return gip  
      } 
    }

不过这并不是一个最佳的选择，希望有知道的小伙伴可以私信咱们。

<a name="Q5"></a>

## Q5: Why? insert a new element into array and it always crash!

### 问题链接

[Q5链接地址](https://forums.developer.apple.com/thread/27803)

### 问题描述

以下代码会在第三行崩溃：

    
    let oldNums: [Int] = [1, 2, 3, 4, 5 ,6 , 7, 8, 9, 10]
    var newArray = oldNums[1..<4]
    newArray.insert(99, atIndex: 0) // <-- crash here
    newArray.insert(99, atIndex: 1) // <-- work very well

### 问题解答

先看下 newArray 的类型以及其他一些属性。

    
    print(newArray.dynamicType) //->ArraySlice<Int>  
    print(newArray.indices) //->1..<4

显然 newArray 并不是一个数组，而是一个SliceArray，它的 StartIndex 是从 1 开始的，也就是通过 [1..<4] 截取的下标开始的，所以插入下标为 0 的位置，就会报错。

修改如下：

    
    newArray.insert(99, atIndex: newArray.startIndex)

当然如果你还是偏执地想要从0开始 那么不妨重新搞一个数组喽，要知道 Array 有个sliceArray 的构造方法。所以改动如下：

    
    let oldNums: [Int] = [1, 2, 3, 4, 5 ,6 , 7, 8, 9, 10]
    var sliceArray = oldNums[1..<4]
    var newArray = Array(oldNums[1..<4])
    
    print(newArray.dynamicType)	//array
    print(newArray.indices)			//0..<3 数组下标为0 1 2
    newArray.insert(99, atIndex: 0) //在0位置插入一个元素
    print(newArray.dynamicType)          //array
    print(newArray.indices)                  //0..<4 数组下标为0 1 2 3
    newArray.insert(99, atIndex: 1) //在1位置插入一个元素
    print(newArray.dynamicType)
    print(newArray.indices)                  //0..<5 数组下标为0 1 2 3 4

<a name="Q6"></a>

## Q6:binary operator '??' cannot be applied to functions?

### 问题链接

[Q6链接地址](https://forums.developer.apple.com/thread/28099)



### 问题描述

在 Playground 中运行以下代码：

    
    func f1() {  
        return  
    }  
    var f2:(()->())?  
    
    let f3 = f2 ?? f1

直接报错：binary operator '??' cannot be applied to operands of type '(() -> ())?' and '() -> ()'



### 问题解答

其实Chris Lattner 大神说了：这就是个已知的 BUG ! 处理 autoclosure 和 function 时已经有人 report 过了。

不过呢[OOPer](https://forums.developer.apple.com/people/OOPer)还是提供了他自己的解决方式。

    
    var f = f1  
    let f3 = f2 ?? f  //不再报错

注意倘若把 var 变成 let 常量定义的话就报错了!

    
    let f = f1  
    let f3 = f2 ?? f  //报错

亲测在 swift2 中依然存在这个BUG！希望大家引起注意。

<a name="Q7"></a>

## Q7: Filter array on type

### 问题链接

[Q7链接地址](https://forums.developer.apple.com/thread/28185)

### 问题描述

请看下面问题：

    
    // 声明了一个类
    class X{
    	var v:Int
    	init(_ v:Int){self.v = v}
    }
    
    // 继承自X
    class X1:X{}
    // 继承自X
    class X2:X{}
    
    var a :[X]
    var a1:[X1]
    // 注意这里有些是用X1初始化 有些是用X2初始化
    // 但是a数组的类型切记是[X],之所以能这么干的原因在于
    // X1 X2都是X的子类，严格意义上来说，说X1 X2是X也是OK的
    a=[X1(1),X1(2),X2(3),X2(4),X1(5)]
    a[0].v  //输出1
    
    // 错误来了
    a1 = a.filter { $0 is X1 } // ERROR
    a1[2].v

> is 关键字就是用来判断某个实例的所属类，注意说的是实例。

报错：    

`Playground execution failed: playground78.swift:15:8: error: cannot invoke 'filter' with an argument list of type '(@noescape (X) throws -> Bool)'`

然后提问者就想可能是自己闭包格式没写全，于是又这么改：

    
      a1 = a.filter { (p:X) -> Bool in p is X1 }

不出意外，还是挂了。



### 问题解答

首先我们要明白 `filter` 方法的用法，filter 函数接收一个闭包作为筛选数组元素的过滤器，闭包一次处理一个元素，符合返回`true`，反之`false`。只有那些`true`的元素才会被`append`到结果数组中返回。更多filter函数请点击[这里](http://www.jianshu.com/notebooks/1752038/latest)。



现在来看看问题代码：

    
    a1 = a.filter { $0 is X1 }

先看式子右边`a.filter { $0 is X1 } `传入了一个简化版闭包`$0 is X1`，其实就是作为筛选条件，一旦a中元素的类型为`X1`，即我们想要的元素，不过这里的元素类型依旧是`X`，而非`X1`，不难得出最后返回的是`[X]`结果数组； 在看式子左边`a1`，这货的类型是`[X1]`。原因找到了！就是因为`[X1]≠[X]`造成的，修改方式嘛，自然就是`as`喽。所以最后修改代码如下

    
    a1 = a.filter{ $0 is X1} as! [X1]
    a1.map{print("\($0.v)")}

其实吧，我更推荐第二种方式，使用 flatMap 实现：

    
    a1 = a.flatMap{$0 as? X1}

我们对a数组中的元素进行遍历，每个都执行`$0 as? X1`类型转换，倘若成功就将元素转换为`X1`类型，失败则返回`nil`，最后`flapMap`会为我们剔除`nil`值。

### 思考

现在有个问题:倘若我们使用面向对象编程呢？上述两种方法还适用吗？



    
    protocol X{
    	var v:Int{get}
    }
    
    // 继承自X
    class X1:X{
    	var v:Int
    	init(_ v:Int){self.v = v}
    }
    // 继承自X
    class X2:X{
    	var v:Int
    	init(_ v:Int){self.v = v}
    }
    
    var a :[X]
    var a1:[X1]
    // 注意这里有些是用X1初始化 有些是用X2初始化
    a=[X1(1),X1(2),X2(3),X2(4),X1(5)]
    a[0].v
    a1 = a.filter{ $0 is X1} as! [X1] //报错：fatal error: array element cannot be bridged to Objective-C
    
    var a2 = a.flatMap{ $0 as? X1}

看来只有`flatMap`依旧坚挺！如果想要使用`filter`的话，可以这么实现：

    
    (a.filter { $0 is XValue }).map { $0 as! XValue }

画蛇添足的赶脚。有木有更好的方法呢？报错说我们没有桥接到OC,让我想到了`@objc`，于是我尝试了下：

    
    
    @objc protocol X{
    	var v:Int{get}
    }
    
    // 继承自X
    class X1:X{
    	@objc var v:Int
    	init(_ v:Int){self.v = v}
    }
    // 继承自X
    class X2:X{
    	@objc var v:Int
    	init(_ v:Int){self.v = v}
    }
    
    var a :[X]
    var a1:[X1]
    // 注意这里有些是用X1初始化 有些是用X2初始化
    a=[X1(1),X1(2),X2(3),X2(4),X1(5)]
    a[0].v
    var a2 = a.filter{ $0 is X1} as! [X1]

这样是ok的。



## Q8、Numbers in swift

### 问题链接

[Q8链接地址](https://forums.developer.apple.com/thread/28056)

### 问题描述

提问者吐槽，以下代码会出现编译错误：

    var x: Int = 2
    var y: Double = 2.0
    var z: Double = y / x

然后又表示，下面代码也是错的，让人难以接受这是 Swift 干的事情：

    var k: Double
    k = Double(x)
    k = Double(x.value)
    k = (Double) x
    k = (Double) x.value



### 问题解答

[OOPer](https://forums.developer.apple.com/people/OOPer) 大神在回帖中写到，其实`k = Double(x)` 是可以执行的， Swift 是强调强类型的语言，不过也提供了不同类型的转换方式。对不同类型进行运算, Swift 是不允许的。以下代码是可以运行的：

    var x: Int = 2
    var y: Double = 2.0
    var z: Double = y / Double(x)



另外，回帖中还提供了一种使用协议和代码的方式来解决这个问题：

    protocol DoubleBOWEMOJI {  
      var Double: Swift.Double {get}  
    }  
    extension Int: DoubleBOWEMOJI {  
       var Double: Swift.Double {return Swift.Double(self)}  
    }  
    
    var z = y / x.Double



