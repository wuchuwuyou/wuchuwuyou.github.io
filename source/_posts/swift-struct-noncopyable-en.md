---
title: Noncopyable Struct in Swift
published: false
categories:
  - technology
date: 2023-11-21 17:56:11
tags:
keywords:
---

## Value Types
Before discussing ownership, let’s first talk about value types.

Let’s look at this code first. Here we declare a structure named User.
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
In `Swift`, `struct` is a value type, and `class` is a reference type. Each variable of a value type is an independent value, and they will be copied during assignment. In the above code, the content of `user` will be copied to `copy`. Modifying the value of `copy.name` does not affect the value of `user.name`. They represent different things. From the memory address printed, We can see that these two variables are different memory addresses.

## [Swift Noncopyable](https://github.com/apple/swift-evolution/blob/main/proposals/0390-noncopyable-structs-and-enums.md#noncopyable-structs-and-enums)

Swift 5.9 was introduced at WWDC2023, bringing many new features. I think the ownership feature is a very important feature. With this feature, Swift will be more expressive and have more accurate control over the code. Instances of `Noncopyable` types always have unique ownership.

Now let's change this structure to a non-copyable type, just add a `~Copyable` suppression annotation:

```Swift
struct User: ~Copyable {
	var name: String	
}
```

Then when we compiled again we got the following error:
``` Swift
func create() {
	var user = User(name: "1") 		// 'user' used after consume
	var copy = user 				// 1. Consumed here
	print(user.name) 				// 2. Used here
	print(copy.name)
}
```

![](/images/WX20231114-115432.png)

We explicitly declared that the structure is a non-copyable type. When we assign `user` to `copy`, the original `user` value will be consumed and the ownership will be transferred (also called "move only" type).
This means that `user` can no longer be used, as ownership now belongs to `copy`. Unlike normal Swift types which can be freely copied, similar to the concept of ownership in Rust.

# Rust ownership

Before we move on to Swift ownership, let’s take a brief look at the concept of ownership in Rust so that we can understand it more easily. Rust manages memory through ownership, which the compiler checks against a series of rules at compile time.

First, let’s take a look at the ownership rules:

> - Each value in Rust has an owner.
> - There can only be one owner at a time.
> - When the owner goes out of scope, the value will be dropped.

In the following code, we declare a variable s1 of type String, assign s1 to variable s2, and then we try to output the value of s1.

```Rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}, world!", s1);
}
```
When we compile this code, we will get the following error. s1 is used after being borrowed.

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

When s1 is assigned to s2, Rust considers that s1 is no longer valid, so there is no need to drop anything after s1 leaves the scope. This is to transfer ownership from s1 to s2, and s1 becomes invalid immediately after being assigned to s2.

We briefly looked at the concept of ownership in Rust, now let’s get back to Swift.

# Why

There is already a `class` type, why do we need a `NonCopyable` type structure?

`class` represents a unique resource. After a `class` is initialized, the reference pointing to this object holds this resource. Then the reference that refers to this object is still `copyable`, so the resource represented by the class is shared ownership, which leads to a series of complex problems, such as heap allocation, reference counting complexity, and shared ownership complexity. 

These complex problems are scenarios that the `NonCopyable` type structure can handle. Structures declared as `NonCopyable` have single ownership and will not be copied.


An example:

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
Every time we finish operating the file, we need to call the close method to close the file, so that when the life cycle of this value ends, the memory can be safely released.

```Swift
let handleToFile = FileDescriptor(descriptor: 1)
let anotherhandleToFile = handleToFile
anotherhandleToFile.close()
try! handleToFile.write(buffer: [1, 2, 3]) // BOOM!
```

For example, in the above code the value of `handleToFile` is assigned to `anotherhandleToFile`. Since `anotherhandleToFile` calls the `close` method, the writing operation of `handleToFile` fails. In this case, if you copy this value, we will Resources are completely out of control. Such errors are difficult to observe as the code becomes more complex, and cannot be displayed during compilation, so non-copyable types can meet our needs.

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

In the above code we declare `FileDescriptor` as a `noncopyable` type. Values of non-copyable types always have unique ownership and can never be copied. Since the values of non-copyable structures have unique identities, they have `deinit` declarations (like classes), which run automatically at the end of the lifetime of the unique instance, automatically releasing file resources and variables.

When we compile again we will get an error, prompting us that `handleToFileA` is used after being consumed. This kind of error message during compilation can help us avoid many inexplicable problems.

![](/images/WX20231115-164601.png)


There is another situation. After `handleToFileA` calls `close` to close the file, if we continue to use it, it will cause a program exception. Such code can be compiled normally but there will be problems at runtime.

```Swift
  let handleToFile = FileDescriptor(descriptor: 1)
  handleToFile.close()
  try! handleToFile.write(buffer: [1, 2, 3]) // BOOM!
```

Now we add the `consuming` modifier to the `close` method

```Swift
  consuming func close() {
    Darwin.close(fd)
  }
```

The close method is marked as consuming. Calling a consuming method transfers ownership to this method. Since our type is not copyable, giving up ownership means that we can no longer use this value. By default Methods in Swift will borrow (borrowing) their parameters including themselves, so the following code is invalid, and we will get an error when compiling again.

![](/images/WX20231115-171105.png)

So far we have briefly understood the aspects of Swift ownership regarding non-copyable structures.

## Closing up

Ownership is a great feature that allows us to write more memory-safe code without introducing additional overhead. We call such features zero-overhead abstractions, and they are crucial in systems programming. Even though Swift doesn’t force us to use the ownership feature, we should still be aware of it and take advantage of it whenever possible to write better code.

## Reference

- https://github.com/apple/swift/blob/main/docs/OwnershipManifesto.md#moveonly-contexts
- https://github.com/apple/swift-evolution/blob/main/proposals/0390-noncopyable-structs-and-enums.md
- https://github.com/apple/swift-evolution/blob/main/proposals/0377-parameter-ownership-modifiers.md
- https://developer.apple.com/videos/play/wwdc2023/10164
- https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html