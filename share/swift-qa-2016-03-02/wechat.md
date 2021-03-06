每周 Swift 社区问答 2016-03-02"


作者：[shanks](http://codebuild.me)

本周共整理了 5 个问题。涉及问题有：浅拷贝问题，Any类型问题，数组初始化问题，泛型类型问题和有继承关系的协议实现问题。


本周整理问题如下:

* [How to make shallow copy of array in swift](#Q1)
* [swift closure cannot override Any](#Q2)
* [Swift Define Array with more than one Integer Range one liner](#Q3)
* [Strange generics behavior in swift](#Q4)
* [Adoption of Protocol in Swift](#Q5)




对应的代码都放到了 github 上，有兴趣的同学可以下载下来研究：[点击下载](https://github.com/SwiftGGTeam/SwiftCommunityWeeklyQA/tree/master/20160302)

<!--more-->

<a name="Q1"></a>

## Question1: How to make shallow copy of array in swift
[点击打开问题原地址](http://stackoverflow.com/questions/35716175/how-to-make-shallow-copy-of-array-in-swift)

### 问题描述

此问题是与 Swift 中的值类型和引用类型相关的。楼主提问：以下代码中，数组之间的赋值不是浅拷贝（arr2 只是指向 arr1，arr1内部值变动，arr2访问也会跟着变化）的？

    class Person
    {
        var name:String = ""
    }
    
    var arr1:[Person] = [Person]()
    
    let p1 = Person()
    p1.name = "Umair"
    
    let p2 = Person()
    p2.name = "Ali"
    
    arr1.append(p1)
    arr1.append(p2)
    var arr2 = arr1
    
    print("\(arr1.count)")    //"2\n"
    print("\(arr2.count)")    //"2\n"
    
    arr1.removeFirst()
    
    print("\(arr1.count)")    //"1\n"
    print("\(arr2.count)")    //"2\n"


### 问题解答

事实上，Array 在 Swift 中，是作为值类型存在的。而值类型的赋值，默认是深拷贝的。只有类在 Swift 中是浅拷贝的。这篇[文章](http://letsswift.com/2014/08/value-type-and-reference-type/)可以看到更多的细节。值类型和引用类型作为 Swift 中非常基础并且重要的概念，理解了值类型，写 Swift 代码就会更加清晰。
如果要在 Swift 中实现浅拷贝数组咋办？那就要借助一个类来实现了，用一个独立的类把数组包装起来，拷贝这个类的实例是浅拷贝的。见以下代码：


    class Person
    {
        var name: String
        
        init(_ name: String) {
            self.name = name
        }
    }
    
    let p1 = Person("Umair")
    let p2 = Person("Ali")
    
    
    class Container {
        var people = [Person]()
        
        init(people: [Person]) {
            self.people = people
        }
    }
    
    let arr1 = Container(people: [p1, p2])
    let arr2 = arr1
    
    print(arr1.people)
    print(arr2.people)
    
    arr1.people.removeFirst()
    
    print(arr1.people)
    print(arr2.people)

<a name="Q2"></a>

## Question2: swift closure cannot override Any
[点击打开问题原地址](http://stackoverflow.com/questions/35697569/swift-closure-cannot-override-any)

### 问题描述

楼主定义了一个类的方法`on`，参数中包含一个含有`Any?`参数的闭包。见以下代码：

    mutating func on(eventName:String, action:((Any?)->())) {
        
    }

调用此方法时，以下代码会报错：

    appSessionHadler.on("login") { (weak data: String?) in
        //...
    }

以下是正确的调用方式：

    appSessionHadler.on("login") { (weak data: Any?) in
        //...
    }
那么问题来了，既然Any?是表示所有的类型，那么应该也包括了String?,在第一次调用时，传入String?为什么会报错？

### 问题解答


此问题和问题5有点类似，Any是可以包含所有类型的值，但是这仅仅是表示，内部数据可以是任何值。在定义方法参数指定为Any?后，调用这个方法，也必须传入Any?的变量作为入参。所以以上把data变成String?当然会报错。
此问题更好的方法是使用泛型来解决，可以在调用时候指定参数的类型：

    mutating func on<T>(eventName:String, action:((T?)->()))
    
    
    appSessionHadler.on("login") { (weak data: String?) in
        //...
    }








<a name="Q3"></a>


## Question3: Swift Define Array with more than one Integer Range one liner

[点击打开问题原地址](http://stackoverflow.com/questions/35717863/swift-define-array-with-more-than-one-integer-range-one-liner)
### 问题描述

可以用以下代码定义一个数组，然后插入一个元素到头部：

    var array: [Int] = Array(1...24)
    array.insert(9999, atIndex: 0)

楼主的问题是，能否把这2句代码写成一句代码，类似以下这样的代码。。虽然是错的，想法倒是挺好的：

    var array: [Int] = Array(9999...9999,1...24)


### 问题解答

楼下提供了以下几种方式任君选择：

    let arr = [1...5, 11...15].reduce([]) { $0.0 + Array($0.1) }
    
    arr
    
    var arr1 = Array([1...5, 11...15].flatten())
    arr1
    
    let arr2 = Array(10 ... 14) + Array(1 ... 24)
    
    arr2
    
    let arr3 = [10 ... 14, 1 ... 4].flatMap { $0 }
    
    arr3

## Question4: Strange generics behavior in swift
[点击打开问题原地址](http://stackoverflow.com/questions/35658177/strange-generics-behavior-in-swift)
### 问题描述

楼主定义了一个泛型函数，然后调用时候发现一个奇怪问题，第一次调用还好，数组元素和第二个参数的类型都是Int的。而第二次调用时，数组元素类型是字符串，而第二个参数类型是Double，为神马不会报错？

    import Foundation
    
    func findAll<T: Equatable>(arr: [T], _ elem: T) -> [Int] {
        var indexesArr = [Int]()
        var counter = 0
        
        for i in arr {
            if i == elem {
                indexesArr.append(counter)
            }
            counter++
        }
        
        return indexesArr
    }
    
    findAll([5, 3, 7, 3, 9], 3)
    findAll(["a", "b", "c", "b", "d"], 2.0) //这样调用也可以？？




### 问题解答

问题出在`import Foundation`上面，引入了`Foundation`以后，数组会映射为`[AnyObject]`,而值类型会映射了对应的`Foundation`类型（比如`NSString`，`NSNumber`等），也就是AnyObject了。这样其实，第二个调用对应的函数原型是：`func findAll(arr: [AnyObject], _ elem: AnyObject) -> [Int] `，传入`[String]`和`Double`是可以通过的。

可以把`import Foundation`去掉，就可以查看到错误。

那么正确的定义应该怎么样呢？二楼给出了答案：

    public protocol NotAnyObject {}
    extension Int: NotAnyObject {}
    extension String: NotAnyObject {}
    extension Double: NotAnyObject {}
    func findAll<T: Equatable where T: NotAnyObject>(arr: [T], _ elem: T) -> [Int] {
        var indexesArr = [Int]()
        var counter = 0
        
        for i in arr {
            if i == elem {
                indexesArr.append(counter)
            }
            counter++
        }
        
        return indexesArr
    }
    
    
    findAll([5, 3, 7, 3, 9], 3)
    findAll(["a", "b", "c", "b", "d"], 2.0) //错误

另外增加了一个空的协议，让需要用到的类型去扩展此协议，这样做的目的就是强制把泛型限制在一致的类型上去。而不能使用不同的类型。



<a name="Q5"></a>

## Question5: Adoption of Protocol in Swift
[点击打开问题原地址](http://stackoverflow.com/questions/35656667/adoption-of-protocol-in-swift)
### 问题描述

以下代码会报错，代码中， 类 Y 遵循协议 X，赋值时候，把A的子类B的实例赋值给了协议X中的存储属性。楼主认为既然A和B有继承关系，那么为啥会报错了？

    protocol A {
        
    }
    
    class B : A {
        var foo = "foo"
    }
    
    protocol X {
        var someA : A {get set}
    }
    
    class Y : X {  //错误: Type Y does not conform to protocol X
       var someA = B()
    }
### 问题解答

编译器只能根据协议X的情况，去要求someA必须是类A的一个实例，B是继承自A的，它有自己的存储属性foo。当然不满足协议X了。如果真的需要声明一个B的实例，那么必须显式指定someA类型为A。见以下代码：

    class Y : X {
        var someA: A = B()
    }

以上的代码虽然可以运行，但是someA已经不能调用B的存储属性了。如果需要调用，必须做一下显式转换：`someA as! B`。一般使用as语句的地方，目前的主流做法都不太推荐。楼下举了一个例子，某个协议要求一个存储属性去存储一辆车的信息，这个时候某人有一辆保时捷，赋值给了这个存储属性，然后另外一个人有一辆马自达，同样也可以赋值给这个存储属性，但是我们看不出来这车是保时捷还是马自达，其实他们差别还是蛮大的。
关于这个话题，有以下2篇文章可以进一步解答：

* [Friday Q&A 2015-11-20: Covariance and Contravariance](https://mikeash.com/pyblog/friday-qa-2015-11-20-covariance-and-contravariance.html)，中文版本，咱们翻译组翻译的：[Friday Q&A 2015-11-20：协变与逆变](http://swift.gg/2015/12/24/friday-qa-2015-11-20-covariance-and-contravariance/)
* [Type Variance in Swift](http://nomothetis.svbtle.com/type-variance-in-swift)








