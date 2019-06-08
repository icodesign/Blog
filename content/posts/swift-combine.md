---
title: "Swift Combine 入门导读"
date: 2019-06-08T23:21:54+08:00
description: "WWDC 2019 毫无疑问是近年来最好的一届。Apple 为了拉拢开发者做出了很多努力，从史上最强工作站 Mac Pro, 到更方便的平台互通 Project Catalyst，再到现代化的界面语言 SwiftUI，当然，以及本文将要介绍的基于 Swift 的 Combine 框架。"
images: 
- /images/swift-combine/op_scan.svg
---

WWDC 2019 毫无疑问是近年来最好的一届。Apple 为了拉拢开发者做出了很多努力，从史上最强工作站 [Mac Pro](https://www.apple.com/mac-pro/), 到更方便的平台互通 [Project Catalyst](https://developer.apple.com/ipad-apps-for-mac/)，再到现代化的界面语言 [SwiftUI](https://developer.apple.com/xcode/swiftui/)，当然，以及本文将要介绍的基于 Swift 的 [Combine](https://developer.apple.com/documentation/combine) 框架。

在具体介绍 Combine 之前，有两个重要的概念需要简要介绍一下：

- 观察者模式
- 响应式编程

## 观察者模式

观察者模式（Observer Pattern）是一种设计模式，用来描述一对多关系：一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。这两类对象分别被称为被 __观察目标（Observable）__和 __观察者（Observer）__，也就是说一个 Observable 可以对应多个 Observer，Observer可以订阅它们感兴趣的内容，当 Observable 内容改变时，它会向这些 Observer 广播通知（调用 Observer 的更新方法）。有一点需要说明的是，Observer 之间彼此时互相独立的，也并不知道对方的存在。

在 Swift 中，一个简单的观察者模式可以被描述为：

```swift
protocol Observable {
    associatedtype T: Observer
    mutating func attach(observer: T)
}

protocol Observer {
    associatedtype State
    func notify(_ state: State)
}

struct AnyObservable<T: Observer>: Observable{

    var state: T.State {
        didSet {
            notifyStateChange()
        }
    }

    var observers: [T] = []

    init(_ state: T.State) {
        self.state = state
    }

    mutating func attach(observer: T) {
        observers.append(observer)
        observer.notify(state)
    }

    private func notifyStateChange() {
        for observer in observers {
            observer.notify(state)
        }
    }
}

struct AnyObserver<S>: Observer {

    private let name: String

    init(name: String) {
        self.name = name
    }

    func notify(_ state: S) {
        print("\(name)'s state updated to \(state)")
    }
}

var observable = AnyObservable<AnyObserver<String>>("hello")
let observer = AnyObserver<String>(name: "My Observer")
observable.attach(observer: observer)
observable.state = "world"

// "My Observer's state updated: hello"
// "My Observer's state updated: world"
```

## 响应式编程

响应式编程（Reactive Programming）是一种编程思想，相对应的有面向过程编程、面向对象编程、函数式编程等等。不同的是，__响应式编程的核心是面向异步数据流和变化的__。

在现在的前端世界中，我们需要处理大量的事件，既有用户的交互，也有不断的网络请求，还有来自系统或者框架的各种通知，因此也无可避免产生纷繁复杂的状态。使用响应式编程后，所有的事件将成为异步的数据流，更加方便的是可以对这些数据流可以进行组合变换，最终我们只需要监听所关心的数据流的变化并做出响应即可。

举个有趣的例子来解释一下：

当你早上起床之后，你需要一边洗漱一边烤个面包，最后吃早饭。

传统的编程方法：

```swift
func goodMorning() {
    wake {
        let group = DispatchGroup()
        group.enter()
        wash {
            group.leave()
        }
        group.enter()
        cook {
            group.leave()
        }
        group.notify(queue: .main) {
            eat {
                print("Finished")
            }
        }
    }
}
```

响应式编程：

```swift
func reactiveGoodMorning() {
    let routine = wake.rx.flapLatest {
        return Observable.zip(wash, cook)
    }.flapLatest {
        return eat.rx
    }
    routine.subsribe(onCompleted: {
        print("Finished")
    })
}
```

不同于传统可以看到 wake/wash/cook/eat 这几个事件通过一些组合转换被串联成一个流，我们也只需要订阅这个流就可以在应该响应的时候得到通知。

### Marble Diagram

为了更方便理解数据流，我们通常用一种叫 Marble Diagram 的图象形式来表示数据流基于时间的变化。

![marble-diagram](/images/swift-combine/marble_diagram.svg)

我们用从左至右的箭头表示时间线，不同的形状代表 Publisher 发出的值（Input），竖线代表正常终止，叉代表发生错误而终止。上图中，上面一条时间线是一个数据流，中间的方块代表组合变换，下方的另外一条时间线代表经过变换后的数据流。

## Combine

现在可以进入正题了，什么是 Combine？

官方文档的介绍是：

> The Combine framework provides a declarative Swift API for processing values over time. These values can represent user interface events, network responses, scheduled events, and many other kinds of asynchronous data.
>
> By adopting Combine, you’ll make your code easier to read and maintain, by centralizing your event-processing code and eliminating troublesome techniques like nested closures and convention-based callbacks.

简而言之，Combine 可以使代码更加简洁、易于维护，也免除了饱受诟病的嵌套闭包和回调地狱。Combine 是 Reactive Programming 在 Swift 中的一个实现，更确切的说是对 [ReactiveX (Reactive Extensions, 简称 Rx)](https://en.wikipedia.org/wiki/Reactive_extensions) 的实现，而这个实现正是基于观察者模式的。

Combine 是基于泛型实现的，是类型安全的。它可以无缝地接入已有的工程，用来处理现有的 `Target/Action`、`Notification`、`KVO`、`callback/closure` 以及各种异步网络请求。

在 Combine 中，有几个重要的组成部分：

- 发布者：Publiser
- 订阅者：Subscriber
- 操作符：Operator

![overview](/images/swift-combine/overview.png)

## Publisher

__在 Combine 中，Publisher 是观察者模式中的 Observable，并且可以通过组合变换（利用 Operator）重新生成新的 Publisher。__

```swift
public protocol Publisher {
    /// The kind of values published by this publisher.
    associatedtype Output

    /// The kind of errors this publisher might publish.
    ///
    /// Use `Never` if this `Publisher` does not publish errors.
    associatedtype Failure : Error

    /// This function is called to attach the specified `Subscriber` to 
    /// this `Publisher` by `subscribe(_:)`
    ///
    /// - SeeAlso: `subscribe(_:)`
    /// - Parameters:
    ///     - subscriber: The subscriber to attach to this `Publisher`.
    ///                   once attached it can begin to receive values.
    func receive<S>(subscriber: S) where S : Subscriber, 
        Self.Failure == S.Failure, Self.Output == S.Input
}
```

在 Publisher 的定义中，`Output` 代表数据流中输出的值，值的更新可能是同步，也可能是异步，`Failure` 代表可能产生的错误，也就是说 Pubslier 最核心的是定义了值与可能的错误。Publisher 通过 `receive(subscriber:)` 用来接受订阅，并且要求 Subscriber 的值和错误类型要一致来保证类型安全。

例如来自官方的一个例子：

```swift
extension NotificationCenter {
		struct Publisher: Combine.Publisher {
				typealias Output = Notification
				typealias Failure = Never
				init(center: NotificationCenter, name: Notification.Name, object: Any? = nil)
		}
}
```

> 注意：在 beta 1 中这个 API 尚未开放，并且所有的和 Foundation framework 的集成都没有开放

这个 Publisher 提供的值就是 Notification 类型，而且永远不会产生错误（`Never`）。这个扩展非常有用，可以很方便地将任何 Notification 转换成 Publisher，便于我们将应用改造成 Reactive。

![Notification Publisher](/images/swift-combine/notification_publisher.svg)

让我们再看一个的例子：

```swift
let justPubliser = Publishers.Just("Hello")
```

`justPubliser` 会给每个订阅者发送一个 `"Hello"` 消息，然后立即结束（这个数据流只包含一个值）。

![Just Publisher](/images/swift-combine/just_publisher.svg)

类似的，Combine 为我们提供了一些便捷的 Publisher 的实现，除了上面的 `Just`，这里也列出一些很有用的 Publisher。

### Empty

`Empty` 不提供任何值的更新，并且可以选择立即正常结束。

![Empty Publisher](/images/swift-combine/empty_publisher.svg)

### Once

`Once` 可以提供以下两种数据流之一：

- 发送一次值更新，然后立即正常结束（和 `Just` 一样）
- 立即因错误而终止

### Fail

`Fail` 和 `Once` 很像，也是提供两种情况之一：

- 发送一次值更新，然后立即因错误而终止
- 立即因错误而终止

![Empty Publisher](/images/swift-combine/fail_publisher.svg)

### Sequence

`Sequence` 将给定的一个序列按序通知到订阅者。

![Empty Publisher](/images/swift-combine/sequence_publisher.svg)

### Future

`Future`  初始化需要提供执行具体操作的 closure，这个操作可以是异步的，并且最终返回一个 `Result`，所以 `Future` 要么发送一个值，然后正常结束，要么因错误而终止。在将一些异步操作转换为 Publisher 时非常有用，尤其是网络请求。例如：

```swift
let apiRequest = Publishers.Future { promise in
    URLSession.shared.dataTask(with: url) { data, _, _ in
        promise(.success(data))
    }.resume()
}
```

![Empty Publisher](/images/swift-combine/future_publisher.svg)

### Deferred

`Deferred` 初始化需要提供一个生成 Publisher 的 closure，只有在有 Subscriber 订阅的时候才会生成指定的 Publisher，并且每个 Subscriber 获得的 Publisher 都是全新的。

![Empty Publisher](/images/swift-combine/deferred_publisher.svg)

那么，在之前的 Just Publisher 例子中，假设我们要实现延迟两秒后再发送消息呢？也很简单。我们前面说到 Publisher 都是可以组合变换的，这些组合变换可以通过操作符来实现的。

上面的例子利用 `delay` 操作符就可以改写为：

```swift
let delayedJustPublisher = justPubliser.delay(for: 2, scheduler: DispatchQueue.main)
```

> 和前面一样，官方目前在 beta 1 中并没有提供有用的 Scheduler 实现，但 session 上的消息来看，RunLoop 和 GCD Queue 都会支持

在文章的后半部分，我们将了解一些常用操作符以及它们的使用场景。

## Subscriber

和 Publisher 相对应的，Subscriber 就是观察者模式中 Observer。

```swift
public protocol Subscriber : CustomCombineIdentifierConvertible {
    /// The kind of values this subscriber receives.
    associatedtype Input

    /// The kind of errors this subscriber might receive.
    ///
    /// Use `Never` if this `Subscriber` cannot receive errors.
    associatedtype Failure : Error

    /// Tells the subscriber that it has successfully subscribed to 
    /// the publisher and may request items.
    ///
    /// Use the received `Subscription` to request items from the publisher.
    /// - Parameter subscription: A subscription that represents the connection 
    /// between publisher and subscriber.
    func receive(subscription: Subscription)

    /// Tells the subscriber that the publisher has produced an element.
    ///
    /// - Parameter input: The published element.
    /// - Returns: A `Demand` instance indicating how many more elements 
    /// the subcriber expects to receive.
    func receive(_ input: Self.Input) -> Subscribers.Demand

    /// Tells the subscriber that the publisher has completed publishing, 
    /// either normally or with an error.
    ///
    /// - Parameter completion: A `Completion` case indicating 
    /// whether publishing completed normally or with an error.
    func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

可以看出，从概念上和我们之前定义的简单观察者模式相差无几。Publisher 在自身状态改变时，调用 Subscriber 的三个不同方法（`receive(subscription)`, `receive(_:Input)`, `receive(completion:)`）来通知 Subscriber。

![subscriber](/images/swift-combine/subscriber.svg)

这里也可以看出，Publisher 发出的通知有三种类型：

- Subscription：Subscriber 成功订阅的消息，只会发送一次
- Value（Subscriber 的 Input，Publisher 中的 Output）：真正的数据，可能发送 0 次或多次
- Completion：数据流终止的消息，包含两种类型：`.finished` 和 `.failure(Error)`，最多发送一次，一旦发送了终止消息，这个数据流就断开了，当然有的数据流可能永远没有终止

大部分场景下我们主要关心的是后两种消息，即数据流的更新和终止。

Combine 内置的 Subscriber 有三种：

- `Sink`
- `Assign`
- `Subject`

`Sink` 是非常通用的 `Subscriber`，我们可以自由的处理数据流的状态。

例如：

```swift
let once: Publishers.Once<Int, Never> = Publishers.Once(100)
let observer: Subscribers.Sink<Publishers.Once<Int, Never>> = Subscribers.Sink(receiveCompletion: {
    print("completed: \($0)")
}, receiveValue: {
    print("received value: \($0)")
})
once.subscribe(observer)

// received value: 100
// completed: finished
```

Publisher 甚至提供了 `sink(receiveCompletion:, receiveValue:)` 方法来直接订阅。

`Assign` 可以很方便地将接收到的值通过 `KeyPath` 设置到指定的 Class 上（不支持 `Struct`），很适合将已有的程序改造成 Reactive。

例如：

```swift
class Student {
    let name: String
    var score: Int

    init(name: String, score: Int) {
        self.name = name
        self.score = score
    }
}

let student = Student(name: "Jack", score: 90)
print(student.score)
let observer = Subscribers.Assign(object: student, keyPath: \Student.score)
let publisher = PassthroughSubject<Int, Never>()
publisher.subscribe(observer)
publisher.send(91)
print(student.score)
publisher.send(100)
print(student.score)

// 90
// 91
// 100
```

在例子中，一旦 publisher 的值发生改变，相应的，`student` 的 `score` 也会被更新。

`PassthroughSubject` 这里是 Combine 内置的一个 Publisher，可能你会好奇：前面不是说 `Subject` 是一种 Observer 吗，这里怎么又是 Pulisher 呢？

## Subject

有些时候我们想随时在 Publisher 插入值来通知订阅者，在 Rx 中也提供了一个 `Subject` 类型来实现。Subject 通常是一个中间代理，即可以作为 Publisher，也可以作为 Observer。`Subject` 的定义如下：

```swift
public protocol Subject : AnyObject, Publisher {

    /// Sends a value to the subscriber.
    ///
    /// - Parameter value: The value to send.
    func send(_ value: Self.Output)

    /// Sends a completion signal to the subscriber.
    ///
    /// - Parameter completion: A `Completion` instance which indicates 
    /// whether publishing has finished normally or failed with an error.
    func send(completion: Subscribers.Completion<Self.Failure>)
}
```

作为 Observer 的时候，可以通过 Publisher 的 `subscribe(_:Subject)` 方法订阅某个 Publisher。

作为 Publisher 的时候，可以主动通过 `Subject` 的两个 `send` 方法，我们可以在数据流中随时插入数据。目前在 Combine 中，有三个已经实现对 `Subject`: `AnySubject`，`CurrentValueSubject` 和 `PassthroughSubject` 。

利用 Subject 可以很轻松的将现在已有代码的一部分转为 Reactive，一个比较典型的使用场景是：

```swift
// Before
class ContentManager {

    var content: [String] {
        didSet {
            delegate?.contentDidChange(content)
        }
    }

    func getContent() {
        content = ["hello", "world"]
    }
}

// After
class RxContentController {

    var content = CurrentValueSubject<[String], NSError>([])

    func getContent() {
        content.value = ["hello", "world"]
    }
}
```

`CurrentValueSubject` 的功能很简单，就是包含一个初始值，并且会在每次值变化的时候发送一个消息，这个值会被保存，可以很方便的用来替代 Property Observer。在上例中，以前需要实现 delegate 来获取变化，现在只需要订阅 content 的变化即可，并且它作为一个 Publisher，可以很方便的利用操作符进行组合变换。

`PassthroughSubject` 和 `CurrentValueSubject` 几乎一样，只是没有初始值，也不会保存任何值。

## AnyPublisher、AnySubscriber、AnySubject

前面说到 Publisher、Subscriber、Subject 是类型安全的，那么在使用中不同类型的 Publisher、Subscriber、Subject 势必会造成一些麻烦。

考虑下面这种情况：

```swift
class StudentManager {

    let namesPublisher: ??? // what's the type?

    func updateStudentsFromLocal() {
        let student1 = Student(name: "Jack", score: 75)
        let student2 = Student(name: "David", score: 80)
        let student3 = Student(name: "Alice", score: 96)
        let namesPublisher: Publishers.Sequence<[String], Never> = Publishers.Sequence<[Student], Never>(sequence: [student1, student2, student3]).map { $0.name }
        self.namesPublisher = namesPublisher
    }

    func updateStudentsFromNetwork() {
        let namesPublisher: Publishers.Future<[String], Never> = Publishers.Future { promise in
            getStudentsFromNetwork {
                let names: [String] = ....
                promise(.success([names]))
            }
        }
        self.namesPublisher = namesPublisher
    }
}
```

这里 `namesPublisher` 究竟该是什么类型呢？第一个方法返回的是 `Publishers.Sequence<[String], Never>`，第二个方法是 `Publishers.Future<[String], Never>`，而我们是无法定义成 `Publisher<[String], Never>` 的。

`AnyPublisher`、`AnySubscriber`、`AnySubject` 正是为这样的场景设计的，它们是通用类型，任意的 Publisher、Subscriber、Subject 都可以通过 `eraseToAnyPublisher()`、`eraseToAnySubscriber()`、`eraceToAnySubject()` 转化为对应的通用类型。

```swift
class StudentManager {

    let namePublisher: AnyPublisher<[String, Never]>

    func updateStudentsFromLocal() {
        let namePublisher: AnyPublisher<[String, Never]> = Publishers.Sequence<[Student], Never>(sequence: students).map { $0.name }.eraseToAnyPublisher()
        self.namePublisher = namePublisher
    }

    func updateStudentsFromNetwork() {
        let namePublisher: AnyPublisher<[String, Never]> = Publishers.Future { promise in
            promise(.success([names]))
        }.eraseToAnyPublisher()
        self.namePublisher = namePublisher
    }
}
```

## 操作符

操作符是 Combine 中非常重要的一部分，通过各式各样的操作符，可以将原来各自不相关的逻辑变成一致的（unified）、声明式的（declarative）的数据流。

转换操作符：

- `map`/`mapError`
- `flatMap`
- `replaceNil`
- `scan`
- `setFailureType`

过滤操作符：

- `filter`
- `compactMap`
- `removeDuplicates`
- `replaceEmpty`/`replaceError`

reduce 操作符：

- `collect`
- `ignoreOutput`
- `reduce`

运算操作符：

- `count`
- `min`/`max`

匹配操作符：

- `contains`
- `allSatisfy`

序列操作符：

- `drop`/`dropFirst`
- `append`/`prepend`
- `prefix`/`first`/`last`/`output`

组合操作符：

- `combineLatest`
- `merge`
- `zip`

错误处理操作符：

- `assertNoFailure`
- `catch`
- `retry`

时间控制操作符：

- `measureTimeInterval`
- `debounce`
- `delay`
- `throttle`
- `timeout`

其他操作符：

- `encode`/`decode`
- `switchToLatest`
- `share`
- `breakpoint`/`breakpointOnError`
- `handleEvents`

> 以下内容仅供参考，目前并未完成所有的操作符的详细介绍

### map/mapError

`map` 将收到的值按照给定的 closure 转换为其他值，`mapError` 则将错误转换为另外一种错误类型。 

```swift
func map<T>(_ transform: (Output) -> T) -> Publishers.Just<T>
```

![subscriber](/images/swift-combine/op_map.svg)

### replaceNil

`replaceNil` 将收到的 `nil` 转换为给定的值。 

```swift
func replaceNil<T>(with output: T) -> Publishers.Map<Self, T> where Self.Output == T?
```

### scan

`scan` 将收到的值与当前的值（第一次使用 `initialResult` ）按照给定的 closure 进行转换。 

```swift
func scan<T>(_ initialResult: T, _ nextPartialResult: @escaping (T, Self.Output) -> T) -> Publishers.Scan<Self, T>
```

![subscriber](/images/swift-combine/op_scan.svg)

### setFailureType

`setFailureType` 强制将上游 Publisher 的错误类型设置为指定类型。这个方法并不是进行错误类型的转换，因为它并没有让我们提供一个 closure，实际上只是为了让不同的 Publisher 的错误类型进行统一，因而这个 Publisher 实际上是不应该发生错误的。

```swift
func scan<T>(_ initialResult: T, _ nextPartialResult: @escaping (T, Self.Output) -> T) -> Publishers.Scan<Self, T>
```

### filter

`filter` 只会让满足条件的值通过。

```swift
func filter(_ isIncluded: @escaping (Self.Output) -> Bool) -> Publishers.Filter<Self>
```

![subscriber](/images/swift-combine/op_filter.svg)

### compactMap

`compactMap` 和 `map` 的功能类似，只是会自动过滤掉空的元素。

```swift
func compactMap<T>(_ transform: @escaping (Self.Output) -> T?) -> Publishers.CompactMap<Self, T>
```

### removeDuplicates

`removeDuplicates` 会跳过在之前已经出现过的值。

```swift
func removeDuplicates(by predicate: @escaping (Self.Output, Self.Output) -> Bool) -> Publishers.RemoveDuplicates<Self>
```

![subscriber](/images/swift-combine/op_removeduplicates.svg)

### replaceEmpty/replaceError

如果上游 Publisher 是个空的数据流，`replaceEmpty` 会发送指定的值，然后正常结束。 

如果上游 Publisher 因错误而终止，`replaceError` 会发送指定的值，然后正常结束。 

```swift
func replaceEmpty(with output: Self.Output) -> Publishers.ReplaceEmpty<Self>
func replaceError(with output: Self.Output) -> Publishers.ReplaceError<Self>
```

![subscriber](/images/swift-combine/op_replaceempty.svg)

### drop

drop` 会一致丢弃收到的值，直到给定的条件得到满足，然后后面的值会正常发送。

```swift
func drop(while predicate: @escaping (Self.Output) -> Bool) -> Publishers.DropWhile<Self>
```

![subscriber](/images/swift-combine/op_drop.svg)

### allSatisfy

`allSatisfy` 会返回一个布尔值来表示所有收到的值是否满足给定的条件。需要说明的是，一旦收到的值不满足条件， `allSatisfy` 会立即发送 `false` 并且正常结束，而如果收到的值满足条件，会一直等到收到上游的 Publisher 发出正常结束的消息之后才发送 `true` 并且正常结束。

```swift
func allSatisfy(_ predicate: @escaping (Self.Output) -> Bool) -> Publishers.AllSatisfy<Self>
```

![subscriber](/images/swift-combine/op_allsatisfy.svg)

### contains
`contains` 会返回一个布尔值来表示收到的值是否包含满足给定的条件的值。需要说明的是，一旦收到的值满足条件， `contains` 会立即发送 `true` 并且正常结束，而如果收到的值不满足条件，会一直等到收到上游的 Publisher 发出正常结束的消息之后才发送 `false` 并且正常结束。

```swift
func contains(_ output: Self.Output) -> Publishers.Contains<Self>
```

![subscriber](/images/swift-combine/op_contains.svg)

### removeDuplicates

`removeDuplicates` 会跳过在之前已经出现过的值。

```swift
func removeDuplicates(by predicate: @escaping (Self.Output, Self.Output) -> Bool) -> Publishers.RemoveDuplicates<Self>
```

![subscriber](/images/swift-combine/op_removeduplicates.svg)

### subscribe(on:)/receiveOn(on:)

在前面的介绍中，我们都没有提及到线程的管理。一种非常常见的场景是我们希望耗时的操作在移步执行，操作完成后在主线程发布值更新的消息，以便于我们直接更新到 UI 控件上。在 Combine 中，这是通过 `Scheduler` 来实现的，`Scheduler` 定义了什么时候以及怎么去执行操作，例如区分后台线程和主线程。

`subscribe(on:)` 指定了 Publisher 在某个 `Scheduler` 执行，而 `receiveOn(on:) ` 指定了 Publisher 在哪个 `Scheduler` 发出通知。 

```swift
let ioPerformingPublisher == // Some publisher.
let uiUpdatingSubscriber == // Some subscriber that updates the UI.
ioPerformingPublisher
    .subscribe(on: backgroundQueue)
    .receiveOn(on: mainQueue)
    .subscribe(uiUpdatingSubscriber)
```

![Scheduler](/images/swift-combine/scheduler.svg)

 `DispatchQueue` 和 `Runloop` 都已经实现了 `Scheduler` 协议，但是在 beta 1 中暂时还不可用。

> 未完待续

## 交流反馈

关于 Combine 的反馈以及其他 iOS 开发技术的交流，欢迎私下联系：

- [Twitter](https://twitter.com/icodesign_me)
- [微博](https://www.weibo.com/jiyise)