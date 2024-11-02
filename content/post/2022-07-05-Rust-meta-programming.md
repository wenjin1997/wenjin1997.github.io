---
layout:     post

title:        "Rust元编程"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt:    "Java并发编程的艺术笔记"
author:       "谢文进"
date:         2022-07-05
# description:  "记录了学习《Programming Rust》笔记。"
# image:      "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published:    2022-07-05 
tags:
     - Rust
     - 元编程

categories:   [ Tech ]
URL:          "/2022/07/05/Rust-meta-programming/"
---
元编程技术大概分为：

* 简单文本替换
* 类型模板
* 反射
* 语法扩展
* 代码自动生成

Rust语言通过反射和AST语法扩展两种手段来支持元编程。

## 反射

反射机制一般是指程序自我访问、检测和修改其自身状态或行为的能力。Rust标准库提供了[std::any::Any](https://rustwiki.org/zh-CN/std/any/trait.Any.html)来支持运行时反射。可以先看下代码结构：

![any-struct](/img/2022-07-05-Rust-meta-programming/any-struct.png)

`Any`定义：

```rust
#[stable(feature = "rust1", since = "1.0.0")]
#[cfg_attr(not(test), rustc_diagnostic_item = "Any")]
pub trait Any: 'static {
    /// Gets the `TypeId` of `self`.
    ///
    /// # Examples
    ///
    /// ```
    /// use std::any::{Any, TypeId};
    ///
    /// fn is_string(s: &dyn Any) -> bool {
    ///     TypeId::of::<String>() == s.type_id()
    /// }
    ///
    /// assert_eq!(is_string(&0), false);
    /// assert_eq!(is_string(&"cookie monster".to_string()), true);
    /// ```
    #[stable(feature = "get_type_id", since = "1.34.0")]
    fn type_id(&self) -> TypeId;
}

#[stable(feature = "rust1", since = "1.0.0")]
impl<T: 'static + ?Sized> Any for T {
    fn type_id(&self) -> TypeId {
        TypeId::of::<T>()
    }
}
```

* 该 trait 加上了`'static` 生命周期限定，意味着该 trait 不能被非静态生命周期的类型实现。 `impl<T: 'static + ?Sized> Any for T` 表示 Rust 中满足 `'static` 生命周期的类型均实现了它。
* `type_id`方法返回 `TypeId` 类型，代表 Rust 中某个类型的全局唯一标识，它是在编译时生成的。

```rust
#[derive(Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Debug, Hash)]
#[stable(feature = "rust1", since = "1.0.0")]
pub struct TypeId {
    t: u64,
}

impl TypeId {
    /// Returns the `TypeId` of the type this generic function has been
    /// instantiated with.
    ///
    /// # Examples
    ///
    /// ```
    /// use std::any::{Any, TypeId};
    ///
    /// fn is_string<T: ?Sized + Any>(_s: &T) -> bool {
    ///     TypeId::of::<String>() == TypeId::of::<T>()
    /// }
    ///
    /// assert_eq!(is_string(&0), false);
    /// assert_eq!(is_string(&"cookie monster".to_string()), true);
    /// ```
    #[must_use]
    #[stable(feature = "rust1", since = "1.0.0")]
    #[rustc_const_unstable(feature = "const_type_id", issue = "77125")]
    pub const fn of<T: ?Sized + 'static>() -> TypeId {
        TypeId { t: intrinsics::type_id::<T>() }
    }
}
```

* 每个 `TypeId` 都是一个“黑盒”，不能检查其内部内容，但是允许复制、比较、打印等操作。`TypeId` 当前仅适用于归因于 `'static` 的类型，但是可以在 future 中消除此限制。

### 通过 is 函数判断类型

```rust
impl dyn Any {
    /// Returns `true` if the inner type is the same as `T`.
    ///
    /// # Examples
    ///
    /// ```
    /// use std::any::Any;
    ///
    /// fn is_string(s: &dyn Any) {
    ///     if s.is::<String>() {
    ///         println!("It's a string!");
    ///     } else {
    ///         println!("Not a string...");
    ///     }
    /// }
    ///
    /// is_string(&0);
    /// is_string(&"cookie monster".to_string());
    /// ```
    #[stable(feature = "rust1", since = "1.0.0")]
    #[inline]
    pub fn is<T: Any>(&self) -> bool {
        // Get `TypeId` of the type this function is instantiated with.
        let t = TypeId::of::<T>();

        // Get `TypeId` of the type in the trait object (`self`).
        let concrete = self.type_id();

        // Compare both `TypeId`s on equality.
        t == concrete
  }
}
```

* 因为 `Any` 是一个 trait ，所以这里的 `is`方法的 `&self` 必然是一个 trait 对象。
* `TypeId::of` 函数用来获取类型 `T` 的全局唯一标识符 `t` 。

```rust
//! 代码清单 12-3: Any 中实现 is 方法源码示意
//! 《Rust编程之道》P436 

use std::any::{Any, TypeId};
enum E { H, He, Li }
struct S { x: u8, y: u8, z: u16 }
#[test]
fn main() {
    let v1 = 0xc0ffee_u32;
    let v2 = E::He;
    let v3 = S { x: 0xde, y: 0xad, z: 0xbeef };
    let v4 = "rust";
    let mut a: &dyn Any;
    a = &v1;
    assert!(a.is::<u32>());
    println!("{:?}", TypeId::of::<u32>());
    a = &v2;
    assert!(a.is::<E>());
    println!("{:?}", TypeId::of::<E>());
    a = &v3;
    assert!(a.is::<S>());
    println!("{:?}", TypeId::of::<S>());
    a = &v4;
    assert!(a.is::<&str>());
    println!("{:?}", TypeId::of::<&str>());
}
```

输出结果：

```bash
TypeId { t: 18349839772473174998 }
TypeId { t: 5500635625788377815 }
TypeId { t: 4485408004367722735 }
TypeId { t: 13307641874416792075 }
```

* `TypeId` 是一个结构体，其字段 `t` 存储了一串数字，这就是全局唯一类型标识符，实际上是 u64 类型。代表唯一标识符的这串数字，在不同的编译环境中，产生的结果是不同的。所以在实际开发中，最好不要将 `TypeId` 暴露到外部接口中被当作依赖。

### 转换到具体类型

Any 提供了 `downcast_ref` 和 `downcast_mut` 两个成对的泛型方法，用于将泛型T向下转换为具体的类型，返回值分别是 `Option<&T>` 和 `Option<&mut T>` 类型。

```rust
impl dyn Any {
    /// Returns some reference to the inner value if it is of type `T`, or
    /// `None` if it isn't.
    ///
    /// # Examples
    ///
    /// ```
    /// use std::any::Any;
    ///
    /// fn print_if_string(s: &dyn Any) {
    ///     if let Some(string) = s.downcast_ref::<String>() {
    ///         println!("It's a string({}): '{}'", string.len(), string);
    ///     } else {
    ///         println!("Not a string...");
    ///     }
    /// }
    ///
    /// print_if_string(&0);
    /// print_if_string(&"cookie monster".to_string());
    /// ```
    #[stable(feature = "rust1", since = "1.0.0")]
    #[inline]
    pub fn downcast_ref<T: Any>(&self) -> Option<&T> {
        if self.is::<T>() {
            // SAFETY: just checked whether we are pointing to the correct type, and we can rely on
            // that check for memory safety because we have implemented Any for all types; no other
            // impls can exist as they would conflict with our impl.
            unsafe { Some(self.downcast_ref_unchecked()) }
        } else {
            None
        }
    }

    /// Returns some mutable reference to the inner value if it is of type `T`, or
    /// `None` if it isn't.
    ///
    /// # Examples
    ///
    /// ```
    /// use std::any::Any;
    ///
    /// fn modify_if_u32(s: &mut dyn Any) {
    ///     if let Some(num) = s.downcast_mut::<u32>() {
    ///         *num = 42;
    ///     }
    /// }
    ///
    /// let mut x = 10u32;
    /// let mut s = "starlord".to_string();
    ///
    /// modify_if_u32(&mut x);
    /// modify_if_u32(&mut s);
    ///
    /// assert_eq!(x, 42);
    /// assert_eq!(&s, "starlord");
    /// ```
    #[stable(feature = "rust1", since = "1.0.0")]
    #[inline]
    pub fn downcast_mut<T: Any>(&mut self) -> Option<&mut T> {
        if self.is::<T>() {
            // SAFETY: just checked whether we are pointing to the correct type, and we can rely on
            // that check for memory safety because we have implemented Any for all types; no other
            // impls can exist as they would conflict with our impl.
            unsafe { Some(self.downcast_mut_unchecked()) }
        } else {
            None
        }
    }
}
```

### 非静态生命周期类型

```rust
//! 代码清单 12-9: 非静态生命周期类型没有实现 Any
use std::any::Any;
struct Unstatic<'a> { x: &'a i32 }
#[test]
fn main() {
    let a = 42;
    let v = Unstatic { x: &a };
    let mut any: &dyn Any;
    // any = &v; // 编译错误
}
```

* 带引用字段的结构体 `Unstatic<'a>` ，其生命周期不是静态的，编译会报错。

修改：

```rust
#![allow(unused)]
use std::any::Any;
struct UnStatic<'a> { x: &'a i32 }
static ANSWER: i32 = 42;
fn main() {
    let v = UnStatic { x: &ANSWER };
    let mut a: &dyn Any;
    a = &v;
    assert!(a.is::<UnStatic>());
}
```

### Any 的应用

* [oso库应用](https://github.com/osohq/oso/blob/main/languages/rust/oso/src/host/class.rs)
* [bevy_reflect库应用](https://github.com/bevyengine/bevy/blob/main/crates/bevy_reflect/src/lib.rs)

### Remark

在2018 Edition 中，以下语句不会报错：

```rust
let mut a: &Any;   			// 2018 Edition Ok
let mut a: &dyn Any;   	// 2018 Edition Ok
```

但是在2021 Edition中，必须加上 `dyn` 。

```rust
let mut a: &Any; 			// 2021 Edition Error: trait objects must include the `dyn` keyword
let mut a: &dyn Any; 	// 2021 Edition Ok
```

## 宏系统
Rust编译过程：

![compile-process](/img/2022-07-05-Rust-meta-programming/compile-process.png)

### 声明宏

宏展开命令： cargo rustc – -Z unstable-options –pretty=expanded



示例1:

```rust
#![allow(unused)]
macro_rules! unless {
      ($arg:expr, $branch:expr) => ( if !$arg { $branch };);
  }
  fn cmp(a: i32, b: i32) {
      unless!( a > b, {
          println!("{} < {}", a, b);
      });
  }
  fn main() {
      let (a, b) = (1, 2);
      cmp(a, b);
  }
```

支持token类型：

```bash
item — an item, like a function, struct, module, etc.
block — a block (i.e. a block of statements and/or an expression, surrounded by braces)
stmt — a statement
pat — a pattern
expr — an expression
ty — a type
ident — an identifier 标识符
path — a path (e.g., foo, ::std::mem::replace, transmute::<_, int>, …)
meta — a meta item; the things that go inside #[...] and #![...] attributes
tt — a single token tree
vis — a possibly empty Visibility qualifier
lifetime — 指代声明周期参数
```

hashmap 1.0

```rust
macro_rules! hashmap {
    ($($key:expr => $value:expr), *) => {
        {
            let mut _map = ::std::collections::HashMap::new();
            $(
                _map.insert($key, $value);
            )*
            _map
        }
    };
}
#[test]
fn main() {
    let map = hashmap!{
        "a" => 1,
        "b" => 2
        // "c" => 3, // 会报错，不支持结尾有逗号
    };
    assert_eq!(map["a"], 1);
}
```

hashmap 2.0：可以匹配结尾有逗号

```rust
macro_rules! hashmap {
    ($($key:expr => $value:expr,)*) => 
        { hashmap!($($key => $value),*) };
    ($($key:expr => $value:expr),*) => {
        {
            let mut _map = ::std::collections::HashMap::new();
            $(
                _map.insert($key, $value);
            )*
            _map
        }
    };
}
#[test]
fn main() {
    let map = hashmap!{
        "a" => 1,
        "b" => 2,
        "c" => 3,
    };
    assert_eq!(map["a"], 1);
}
```

hashmap 3.0

```rust
macro_rules! hashmap {
    ($($key:expr => $value:expr),* $(,)*) => {
        {
            let mut _map = ::std::collections::HashMap::new();
            $(
                _map.insert($key, $value);
            )*
            _map
        }
    };
}
#[test]
fn main() {
    let map = hashmap!{
        "a" => 1,
        "b" => 2,
        "c" => 3,
    };
    assert_eq!(map["a"], 1);
}
```

hashmap 4.0 ： 节省存储空间

```rust
macro_rules! unit {
    ($($x:tt)*) => (());
}
macro_rules! count {
    ($($key:expr),*) => (<[()]>::len(&[$(unit!($key)),*]));
}
macro_rules! hashmap {
    ($($key:expr => $value:expr),* $(,)*) => {
        {
            let _cap = count!($($key),*);
            let mut _map 
                = ::std::collections::HashMap::with_capacity(_cap);
            $(
                _map.insert($key, $value);
            )*
            _map
        }
    };
}
#[test]
fn main() {
    let map = hashmap!{
        "a" => 1,
        "b" => 2,
        "c" => 3,
    };
    assert_eq!(map["a"], 1);
}
```

hashmap 5.0：将声明宏放在一起

```rust
macro_rules! hashmap {
    (@unit $($x:tt)*) => (());
    (@count $($rest:expr),*) => (<[()]>::len(&[$(hashmap!(@unit $rest)),*]));
    ($($key:expr => $value:expr),* $(,)*) => {
        {
            let _cap = hashmap!(@count $($key),*);
            let mut _map 
                = ::std::collections::HashMap::with_capacity(_cap);
            $(
                _map.insert($key, $value);
            )*
            _map
        }
    };
}
#[test]
fn main() {
    let map = hashmap!{
        "a" => 1,
        "b" => 2,
        "c" => 3,
    };
    assert_eq!(map["a"], 1);
}
```

示例：

```rust
  macro_rules! sum {
      ($e:expr) => ({
          let a = 2;
          $e + a
      })
  }
  fn main(){
      // error[E0425]: cannot find value `a` in this scope
      let four = sum!(a);
  }
```

* 注意这里会报错，宏里面的 `a` 变量是局部的。

### 过程宏

过程宏三件套：

- [syn](https://github.com/dtolnay/syn)
- [quote](https://github.com/dtolnay/quote)
- [proc-macro2](https://github.com/alexcrichton/proc-macro2)

介绍：

* Syn is a parsing library for parsing a stream of Rust tokens into a syntax tree of Rust source code. 将 token 流转换成语法树。
* quote: This crate provides the [`quote!`](https://docs.rs/quote/1.0/quote/macro.quote.html) macro for turning Rust syntax tree data structures into tokens of source code. 将语法树转换成 token 源码。
* proc-macro2 比 proc-macro 更加的灵活。TokenStream API接口。

```rust
 // find_by_or!{ Person -> people::[name:String || company_name:String]   }

    use super::*;

    pub struct DbOpByOrBy {
        pub model: Type,
        pub table: Ident,
        pub bracket_token: token::Bracket,
        pub content: FieldContentOr,
    }

    pub struct FieldContentOr {
        pub name1: Ident,
        pub ty1: Type,
        pub name2: Ident,
        pub ty2: Type,
    }

    impl Parse for DbOpByOrBy {
        fn parse(input: ParseStream) -> Result<Self> {
            let content;
            let model: Type = input.parse()?;
            input.parse::<Token![->]>()?;
            let table: Ident = input.parse()?;
            input.parse::<Token![::]>()?;
            let bracket_token = bracketed!(content in input);
            let content = content.parse()?;
            Ok(DbOpByOrBy {
                model,
                table,
                bracket_token,
                content,
            })
        }
    }

    impl Parse for FieldContentOr {
        fn parse(input: ParseStream) -> Result<Self> {
            let name1: Ident = input.parse()?;
            input.parse::<Token![:]>()?;
            let ty1: Type = input.parse()?;
            input.parse::<Token![||]>()?;
            let name2: Ident = input.parse()?;
            input.parse::<Token![:]>()?;
            let ty2: Type = input.parse()?;
            Ok(FieldContentOr {
                name1,
                ty1,
                name2,
                ty2,
            })
        }
    }

    // in lib.rs

    // find_by_or!{ Person -> people::[name:String || company_name:String]   }

    #[proc_macro]
    pub fn find_by_or(input: TokenStream) -> TokenStream {
        let DbOpByOrBy {
            model,
            table,
            bracket_token,
            content,
        } = parse_macro_input!(input as DbOpByOrBy);
        let (name1, name2) = (content.name1, content.name2);
        let (ty1, ty2) = (content.ty1, content.ty2);
        let fn_name = format!("find_by_{}_or_{}", name1, name2);
        let fn_name = Ident::new(&fn_name, proc_macro2::Span::call_site());

        let expanded = quote! {
            impl #model {
                pub fn #fn_name(conn: &PgConnection, #name1: #ty1, #name2: #ty2) -> QueryResult<#model> {
                    #table::table
                    .filter(#table::dsl::#name1.eq(#name1))
                    .or_filter(#table::dsl::#name2.eq(#name2))
                    .get_result(conn)
                }
            }
        };
        TokenStream::from(expanded)
    }
```



## 参考资料

* 《Rust编程之道》