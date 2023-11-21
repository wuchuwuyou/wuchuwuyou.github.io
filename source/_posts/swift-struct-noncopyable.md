---
title: Swift中的不可复制的结构体
published: true
categories:
  - technology
date: 2023-11-21 17:52:13
tags:
keywords:
---

## 值类型
在讨论什么是所有权之前，我们先来说说值类型。我们先看一段代码, 这里声明一个名为User的结构体
```swift
struct User {
	var name: String
}
func create() {
	var user = User(name: "1")
	user.name = "user"
	var copy = user
    copy.name = "copy"
	print(user.name)						// print user
	print(copy.name)						// print copy user
	print(MemoryLayout<User>.size)			// 16
	withUnsafePointer(to: &user) { pointer in
		print("Address of user: \(pointer)") 	// Address of user: 0x000000016b3faea0
	}
	withUnsafePointer(to: &copy) { pointer in
		print("Address of copy: \(pointer)") 	// Address of copy: 0x000000016b3fae90
	}
}
```
<!--More-->
在 `Swift` 中 `struct` 是值类型的，而 `class` 是引用类型的，值类型的每个变量都是独立的值，它们会在赋值的时候发生拷贝。上面的代码，`user` 的内容会被拷贝到 `copy`，修改 `copy.name` 的值并不影响 `user.name` 的值，它们表示的就是不同的东西了, 从上面打印的内存地址我们可以看到这两个变量是不同的内存地址。


## [Swift Noncopyable](https://github.com/apple/swift-evolution/blob/main/proposals/0390-noncopyable-structs-and-enums.md#noncopyable-structs-and-enums)

WWDC2023 中介绍了 Swift 5.9，带来了许多新特性。其中的所有权的特性我觉得是个非常重要的特性，有了这个特性之后 Swift 将更具表达力，对代码有更准确的控制。其中的 `Noncopyable` 类型的实例始终具有唯一的所有权。 

现在让我们把这个结构体改成不可复制类型，只需要添加一个 `~Copyable` 标记
```Swift
struct User: ~Copyable {
	var name: String	
}
```

然后我们再次编译的时候我们得到了错误 如下的错误:
``` Swift
func create() {
	var user = User(name: "1") 		// 'user' used after consume
	var copy = user 				// 1. Consumed here
	print(user.name) 				// 2. Used here
	print(copy.name)
}
```

![](/images/WX20231114-115432.png)

我们显式的声明了该结构体是不可复制类型，当我们把 `user` 赋值给 `copy`, 会导致原有的 `user` 值被消耗, 所有权发生了转移（也称为“仅移动”类型），这意味着 `user` 不能再使用了，因为现在所有权属于 `copy`。与可以自由复制的普通 Swift 类型不同，和 Rust中的所有权概念类似。

# Rust 所有权
在我们继续介绍Swift 所有权之前，让我们先简单了解一下 Rust 中的所有权概念，这样方便我们理解。Rust 是通过所有权来管理内存，编译器在编译时会根据一系列规则进行检查。

首先，让我们看一下Rust所有权的规则：
> 1. Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
> 2. 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
> 3. 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

下面代码我们声明了一个 String 类型的变量 s1, 并将 s1 赋值给变量 s2， 然后我们尝试输出 s1 的值
```Rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}, world!", s1);
}
```
当我们编译的时候会得到如下的错误， s1 被借用后又使用了

```Shell
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:4:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
  |
```

当 s1 赋予 s2 后，Rust 认为 s1 不再有效，因此也无需在 s1 离开作用域后 drop 任何东西，这就是把所有权从 s1 转移给了 s2，s1 在被赋予 s2 后就马上失效了。

我们简单的了解了下 Rust 中的所有权概念，现在让我们回到 Swift 中来。
# Why
已经有了 `class` 类型，为什么还需要 `NonCopyable` 类型？
`class` 代表了唯一的资源，一个 `class` 初始化后，指向这个对象的引用持有这份资源。然后引用这个对象的引用依然是`copyable`，所以类所代表的资源是共享所有权的，这会导致一系列复杂的问题，如堆分配、引用计数的复杂性和共享所有权的复杂性。

这些则是 `NonCopyable` 处理的场景。 `NonCopyable` 只有单一所有权并且不会被复制。


一个实际的例子，关于操作文件的实际的例子：

```Swift
struct FileDescriptor {
  private var fd: CInt
  init(descriptor: CInt) { self.fd = descriptor }

  func write(buffer: [UInt8]) throws {
    let written = buffer.withUnsafeBufferPointer {
      Darwin.write(fd, $0.baseAddress, $0.count)
    }
    // ...
  }
  func close() {
    Darwin.close(fd)
  }
}
```
每次当我们操作完文件以后需要调用 close 方法来关闭文件, 这样当这个值的生命周期结束后，可以安全的释放内存。

```Swift
let handleToFile = FileDescriptor(descriptor: 1)
let anotherhandleToFile = handleToFile
anotherhandleToFile.close()
try! handleToFile.write(buffer: [1, 2, 3]) // BOOM!
```

如上面代码 `handleToFile` 的值赋予给了 `anotherhandleToFile` ，由于 `anotherhandleToFile` 调用了 `close` 的方法，导致 `handleToFile` 的写入操作失败，这种情况就是如果你拷贝了这个值，我们对资源就完全失去控制了，这样的错误随着代码越来越复杂很难观察到，并且不能在编译期间显示出来，所以不可复制的类型就可以满足我们的需求。

```Swift
struct FileDescriptor: ~Copyable {
  private var fd: CInt
  init(descriptor: CInt) { self.fd = descriptor }
  func write(buffer: [UInt8]) throws {
    let written = buffer.withUnsafeBufferPointer {
      Darwin.write(fd, $0.baseAddress, $0.count)
    }
    // ...
  }
  func close() {
    Darwin.close(fd)
  }
  deinit {
    Darwin.close(fd)
  }
}
```

上面的代码中我们将 `FileDescriptor` 声明为 `noncopyable` 类型. 不可复制类型的值始终具有唯一的所有权，并且永远不能被复制。由于不可复制的结构体的值具有唯一的标识，因此它们有 `deinit` 声明（如类），这些声明在唯一实例的生命周期结束时自动运行, 会自动释放文件资源以及变量。

当我们再次编译的时候会得到一个错误，提示我们 `handleToFileA` 在被消耗后又被使用了。这种编译期间的错误信息可以让我们避免很多莫名其妙的问题。

![](/images/WX20231115-164601.png)


还有另外一种情况,  `handleToFileA` 调用 `close` 关闭文件后，我们继续使用会导致程序异常，这样的代码也是可以正常编译的但是在运行时是会有问题的。

```Swift
  let handleToFile = FileDescriptor(descriptor: 1)
  handleToFile.close()
  try! handleToFile.write(buffer: [1, 2, 3]) // BOOM!
```

现在我们给 `close` 方法加上 `consuming` 修饰符

```Swift
  consuming func close() {
    Darwin.close(fd)
  }
```

close 方法被标记为消耗性（consuming）的，调用一个消耗性方法，是将所有权交给这个方法，由于我们的类型是不可复制的，放弃所有权意味着我们不能再使用这个值，默认的情况下 Swift 中的方法会借用（borrowing）其参数包括其自身，那么下面的代码就是无效的了，并且我们再次编译会得到一个错误。

![](/images/WX20231115-171105.png)

至此我们简单的了解了 Swift 所有权中关于不可复制的结构体的部分内容。

## 总结

所有权是一个非常棒的特性，它能够让我们写出更加内存安全的代码，同时又不需要引入额外的开销。我们称这类特性为零开销抽象，它们在系统编程中至关重要。尽管 Swift 没有强制要求我们使用所有权特性，但我们仍然应该了解它，并尽可能利用它写出更好的代码。

## 更多内容

- https://github.com/apple/swift/blob/main/docs/OwnershipManifesto.md#moveonly-contexts
- https://github.com/apple/swift-evolution/blob/main/proposals/0390-noncopyable-structs-and-enums.md
- https://github.com/apple/swift-evolution/blob/main/proposals/0377-parameter-ownership-modifiers.md
- https://developer.apple.com/videos/play/wwdc2023/10164
- https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html