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

std中的一个线程构建器的例子

```rust
struct Builder {
    name: Option<String>,
    stack_size: Option<usize>,
    no_hooks: bool,
}

impl Builder {
    pub fn new() -> Builder {
        Builder {
            name: None,
            stack_size: None,
            no_hooks: false,
        }
    }

    // 针对每一个结构体属性字段都有一个专属的返回自身的设置实例方法，在其内可以有
    // 简单的设置值，也可以有对设置值的一些专门的处理，例如针对年龄的
    // 或许有检查是否是合法年龄的逻辑等
    pub fn name(mut self, name: String) -> Builder {
        self.name = Some(name);
        self    // 返回自身实例是实现chain-style calling的关键所在
    }

    pub fn stack_size(mut self, size: usize) -> Builder {
        self.stack_size = Some(size);
        self
    }

    pub fn no_hooks(mut self) -> Builder {
        self.no_hooks = true;
        self
    }
}

// 如何使用
let builder = Builder::new()
    .name("sun".into())
    .stack_size(6);

// or
let builder = Builder::new()
    .stack_size(6);
```

### 例子-2

稍微复杂一点的例子

```rust
struct User {
    username: String,
    birthday: NaiveDate,
}

struct UserBuilder {
    username: Option<String>,
    birthday: Option<NaiveDate>
}

struct InvalidUsername;

enum IncompleteUserBuild {
    NoUsername,
    NoCreatedOn,
}

impl UserBuilder {
    fn new() -> Self {
        Self {
            username: None,
            birthday: None,
        }
    }

    // 添加了一些额外的检查逻辑，只有符合这些逻辑才会继续走下去
    fn set_username(&mut self, username: String) -> Result<&mut self. InvalidUsername> {
        let valid = username
            .chars()
            .all(|chr| matches!(chr, 'a'..='z' | '0'..='9'));

        if valid {
            self.username = Some(username);
            Ok(self)
        } else {
            Err(InvalidUsername)
        }
    }

    fn set_birthday(&mut self, data: NaiveDate) -> &mut Self {
        self.birthday = Some(date);
        self
    }

    fn build(&self) -> Result<User, IncompleteUserBuild> {
        if let Some(username) = self.username.clone() {
            if let Some(birthday) = self.birthday.clone() {
                Ok(User { username, birthday })
            } else {
                Err(IncompleteUserBuild::NoCreatedOn)
            }
        } else {
            Err(IncompleteUserBuild::NoUsername)
        }
    }
}

// 简单的使用
let user = UserBuilder::new()
    .set_username("aaaa")
    .set_birthday(NaiveDate::new("1995-06-01 06:01:00"))
    .build();
```


## 使用macro实现自动生成builder pattern

![#builder](https://lib.rs/keywords/builder)

我们还可以使用如Clone、Debug的工作方式来为类型添加builder，无须手动添加相关中间层代码，下面的例子
使用名为TypedBuilder的crate实现:

```rust
use typed_builder::TypedBuilder;

#[derive(TypedBuilder)]
struct Foo {
    x: i32,
    #[builder(default, setter(strip_option))]
    y: Option<i32>,
    #builder[(default=20)]
    z: i32,
}

let foo = Foo::builder()
    .x(1)
    .y(2)
    .z(3)
    .build()
```

> derive_builder crate活跃、使用度也很高


## 最后

构建者模式可以让我们写出更清晰、易读的API.

