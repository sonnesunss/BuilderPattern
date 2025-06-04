# 构建者模式

一句话，所谓构建者模式就是类型实例构建的一种方式，它与初始化语法、new、default的目的是一致的.


给定一个如下的结构体:

```rust
struct Message {
    from: String,
    content: String,
    attachment: Option<Striing>
}
```

使用rust的结构体初始化语法构建此结构体如下所示:

```rust
Message {
    from: "Sun".into(),
    content: "Hello".into(),
    attachment: None
}
```

此外，我们还可以使用rust的习惯性关联方法new来构建一个结构体，如下:

```rust
impl Message {
    pub fn new(from: String, content: String, attachment: Option<String>)
        -> Self
    {
        Self {
            from,
            content,
            attachment,
        }
    }
}
```

或者在一些不能提供new关联方法的类型上面实现Default trait、通过default派生宏的方式构建一个类型实例, 如下:

```rust
impl Default for Message {
    fn default() -> Self {
        Self {
            from: "sun".into(),
            content: "Hello".into(),
            attachment: None,
        }
    }
}
```

再者, 使用一个专门设计的一系列方法通过chain的方式构建出一个类型实例:

```
Message::builder()
    .from("Sun".into())
    .content("Hello".into())
    .build()
```

将这些放在一起，观察一下就可以发现这种模式可以提供更清晰易读、易理解的初始化过程.


> 虽然例子举得是结构体的，但是对于enum类型也是可以的，不要着相


## Building our own builder pattern

设计模式的引入势必会造成代码量的膨胀，因为添加了一些中间层，提供了原本之外的一些能力.

具体的实现需要方式根据具体的场景去实现就好，这并没有统一的实现模板代码，下面给几个例子说明一下：


### 例子-1



### 例子-2

### 例子-3


