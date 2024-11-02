---
layout:     post

title:        "Programming Rust"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt:    "Java并发编程的艺术笔记"
author:       "谢文进"
date:         2022-07-05
description:  "记录了学习《Programming Rust》笔记。"
# image:      "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published:    2022-07-05 
tags:
     - Rust
     - 笔记

categories:   [ Tech ]
URL:          "/2022/07/05/Programming-Rust/"
---

记录了学习《Programming Rust》笔记。

## Chapter 3. Fundamental Types

* Rust有类型推断
* 函数可以是泛型的

Rust的类型

| Type                                    | Description                     |
| --------------------------------------- | ------------------------------- |
| i8,i16,i32,i64,i128,u8,u16,u32,u64,u128 |                                 |
| isize,usize                             |                                 |
| f32,f64                                 |                                 |
| bool                                    |                                 |
| char                                    |                                 |
| (char, u8, i32)                         | 元组                            |
| ()                                      |                                 |
| `struct S { x: f32, y:f32 }`            | 结构体                          |
| `struct T (i32, char);`                 | Tuple-like struct               |
| `struct E;`                             | Unit-like struct; has no fields |
| `enum Attend { OnTime, Late(u32) }`     |                                 |
| `Box<Attend>`                           |                                 |
| &i32,&mut i32                           |                                 |
| String                                  |                                 |
| &str                                    |                                 |
| [f64; 4], [u8; 256]                     |                                 |
| `Vec<f64>`                              |                                 |
| &[u8], &mut [u8]                        |                                 |
| Option<&str>                            |                                 |
| Result<u64, Error>                      |                                 |
| &dyn Any, &mut dyn Read                 |                                 |
| fn(&str) -> bool                        |                                 |
| (Clousure types have no written form)   | Clousure                        |

### Fixed-Width Numerical Types

如果需要任意精度表示的一些数值，可以查看 *num* 包。

#### Integer Types

不像C和C++，Rust将字符看作是和数值不同的类型，一个 char 不是 u8，也不是 u32（尽管字符是32位长的）。

Rust要求数组的索引必须是 usize 类型的值。

在Rust整数可以带一个后缀来表示它们的类型，比如`42u8`。如果没有给整数指定具体的类型，Rust会将值存放在一个特定的类型中，传给函数期望的参数，比较另一个有别的特定类型的值，最后，如果有多个可能，而 `i32`是其中一个的话，就默认是 `i32`类型，否则就会报错。

前缀`0x`，`0o`和`0b`表示十六进制、八进制和二进制。

为了更清楚地表示很长的数字，还可以给数字加下划线，比如`4_294_967_295`。有前缀和后缀的数字依然可以加下划线。

尽管Rust中数字和字符是不同的类型，但是也提供了 **字节字面量**（ *byte literals*），即用类字符字面量表示u8值：`b'X'`代表的ASCII编码的字符`X`，它是一个u8值。比如，由于字符A的ASCII编码是65，因此`b'A'`等价于`65u8`。字节字面量中只能出现在ASCII编码的字符。

特殊的ACSII编码的字符

| Character | Byte literal |
| --------- | ------------ |
| 单引号    | `b'\''`      |
| 反斜杠    | `b'\\'`      |
| 换行      | `b'\n'`      |
| 回车      | `b'\r'`      |
| 制表符    | `b'\t'`      |

对于一些ASCII编码的字符可以用十六进制来表示，这样更容易阅读。比如`b'\x1b'`。

整数类型之间转换可以用 `as` 操作符。比如 `10_i8 as u16`。

整数的一些方法，参见 [std::i32](https://rustwiki.org/zh-CN/std/primitive.i32.html#)。

```rust
assert_eq!(2_u16.pow(4), 16);
assert_eq!((-4_i32).abs(), 4);
assert_eq!(0b101101_u8.count_ones(), 4);
```

看下面这段代码

```rust
println!("{}", (-4).abs());
```

这里没有给出数值的类型，是不是默认 `i32` 呢？但其实会报错：

```text
error: can't call method `abs` on ambiguous numeric type `{integer}`
```

为什么会这样子呢？因为Rust想要在调用一个类型自己的方法之前明确地知道一个值的整数类型。i32 的默认值仅适用于在所有方法调用都已解决后类型仍然不明确的情况，在这里就太晚了，因此会报错。换句话说，就是这里我要先调用 `abs()`方法，这个时候需要确定类型，还没等推断出默认类型，就已经报错了。解决方案：

```rust
println!("{}", (-4_i32).abs());
println!("{}", i32::abs(-4));
```

注意**方法调用会发生在一元前缀操作符之前**，所以处理负数的时候要当心。比如`-4_i32.abs()`，这里没有括号，会先调用函数`4_i32.abs()`，然后再取负号，最终结果为-4。

#### Checked, Wrapping, Saturating, and Overflowing Arithmetic

在Debug模式下，如果整数数值操作溢出，会报错。但是在发布模式构建的时候，会 *wraps*。

> In a release build, the operation *wraps around*: it produces the value equivalent to the mathematically correct result modulo the range of the value. (In neither case is overflow undefined behavior, as it is in C and C++.)

上述是默认的，如果这不是你想要的，可以调用一些方法，主要有四类。

* ***Checked*** operations return an Option of the result: Some(v) if the mathematically correct result can be represented as a value of that type, or None if it cannot.
* ***Wrapping*** operations return the value equivalent to the mathematically correct result modulo the range of the value.
* ***Saturating*** operations return the representable value that is closest to the mathematically correct result. In other words, the result is “clamped” to the maximum and minimum values the type can represent.
* ***Overflowing*** operations return a tuple (result, overflowed), where result is what the wrapping version of the function would return, and overflowed is a bool indicating whether an overflow occurred.

| Operation           | Name suffix | Example                               |
| ------------------- | ----------- | ------------------------------------- |
| Addition            | add         | 100_i8.checked_add(27) == Some(127)   |
| Subtraction         | sub         | 10_u8.checked_sub(11) == None         |
| Multiplication      | mul         | 128_u8.saturating_mul(3) == 255       |
| Division            | div         | 64_u16.wrapping_div(8) == 8           |
| Remainder           | rem         | (-32768_i16).wrapping_rem(-1) == 0    |
| Negation            | neg         | (-128_i8).checked_neg() == None       |
| Absolute value      | abs         | (-32768_i16).wrapping_abs() == -32768 |
| Exponentiation      | pow         | 3_u8.checked_pow(4) == Some(81)       |
| Bitwise left shift  | shl         | 10_u32.wrapping_shl(34) == 40         |
| Bitwise right shift | shr         | 40_u64.wrapping_shr(66) == 10         |

#### Floating-Point Types

![img](/img/2022-07-05-Programming-Rust/programming-rust-floating-point.png)

整数部分后的分数部分，指数或者类型后缀至少包含一个，来区别于整数字面量。`5.`整数部分之后只有一个小数点，也是一个有效的浮点数常数。

浮点数默认类型是 f64 。

为了类型推断的目的，Rust中整数和浮点数之间不会进行相互推断。

f32和f64都有一些特殊值，如 `INFINITY`，`NEG_INFINITY`，`NAN`（that not-a-number value），`MIN`和`MAX`。

[std::f32](https://rustwiki.org/zh-CN/std/primitive.f32.html)和[std::f64](https://rustwiki.org/zh-CN/std/primitive.f64.html)提供了很多方法。

[std::f32::consts](https://rustwiki.org/zh-CN/std/f32/consts/index.html)和[std::f64::consts](https://rustwiki.org/zh-CN/std/f64/consts/index.html)提供一些数学常量。

Rust是没有数值的隐式转换的，所以如果传入的参数类型和函数的参数类型不一致，这里不会发生隐式转换，因此会报错。想要转换只能显式地用 `as` 操作符。

### The bool Type

在Rust中，bool类型的值为 true 和 false。

Rust中在要求一个Boolean值的上下文中是非常严格的。控制结构`if`和`while`要求条件必须是 bool 表达式， 短路逻辑操作符 `&&` 和 `||` 也要求必须是 bool 表达式。必须 `if x!= 0 { ... }`这样写，而不能简单地写为 `if x { ... }`。

Rust可以用 as 操作符将 bool 值转换为数字，但是不能反过来将数值转换为 bool 值。

```rust
assert_eq!(false as i32, 0);
assert_eq!(true as i32, 1);
```

虽然可以只有一位就表示 bool 值，但是Rust底层用一整个字节来存储 bool 值，这样的话就可以创建一个指针来指向它。

### Characters

char 类型表示单个Unicode字符，占 32 位。

字符可以用十六进制来表示或者`\u{}`来表示。

* 如果字符编码在ASCII编码范围内，可以用十六进制。例如`'\xHH'`。
* 也可以用`\u{}`来表示任何的Unicode编码字符。例如`'\u{HHHHHH}'`。

字符必须在有效的 Unicode 码点范围内。

char 和其他类型之间没有隐式地转换。当然可以显式用 as 操作符将 char 类型转换成一个整数类型，如果类型小于32位，会进行截断。相反地，只有 u8 类型能用 as 操作符显式地转换为 char 类型。这一点也很自然，Rust要保证可靠，其他数值类型转换成 char 类型时，可能是无效的字符。不过`std::char::from_u32`可以将任何 u32 类型的值转换，返回的是 `Option<char>`。

[char(primitive type)](https://rustwiki.org/zh-CN/std/primitive.char.html)文档有相关的方法。

### Tuples

元组可以将不同类型的值组合在一起，可以用在返回多个不同类型的值的函数中。

用 `a.0`可以来获取具体的元素。

元组的索引必须是常数，不可以是变量，比如`t[i]`或者`t.i`就是错误的。

可以用模式匹配得到元组对应的值。在指定变量值或者函数返回多个值时很有用。用元组来表示相关的变量是很清晰的，比如宽和高，可以用一个元组把它们放在一起，语义更加清晰。

unit type `()`可以用于函数的返回类型，表示什么也不返回。

可以在元组的最后加上逗号，容易看出列表删除或添加了元素。如果是单个值，`("lonely hearts",)`相比 `("lonely hearts")`更加清晰，可以判断是元组而不是括号表达式。

### Pointer Types

#### References

`&x` 产生了对 `x` 的一个引用，用Rust的术语来说是向 `x` 借了一个引用。

在Rust中，引用始终不为空。Rust会跟踪值的所有权和生命周期，在编译时期会检查出悬垂指针，二次释放以及指针无效的错误。

* & T：不可变的共享引用
* &mut T：可变的独占引用

将不可变引用和可变引用看作单个读-写者和多个读者。

#### Boxes

用于堆分配的指针类型。

```rust
let t = (12, "eggs");
let b = Box::new(t); // 在堆上分配内存，类型为 Box<(i32, &str)>
```

#### Raw Pointers

* *const T
* *mut T

只能在 unsafe 块中对原始指针解引用。

### Arrays, Vectors, and Slices

有三种类型表示内存中连续的一块数据。

* `[T; N]`表示数组。数组的长度必须要在编译时期就确定。
* `Vec<T>` 表示 T 类型的向量。动态分配的，元素存放在堆上。
* `&[T]`和`&mut [T]` 称为 T 的共享切片或可变切片。可以把切片看作是一个胖指针，指向第一个元素，同时还包含可以访问到的元素的个数。

一些方法：

* `v.len()` 返回元素的个数。
* `v[i]` 表示 v 的第 i 个元素，注意这里索引 i 必须是 usize 类型的。Rust总是会去检查 i 有没有在有效的范围内。

#### Arrays

直接用方括号生成数组，或者[T; N]生成数组，Rust不提供未初始化的数组。

数组的长度必须在编译时就确定，不可以用变量，比如`[true; n]` 这就是错误的。如果想在运行期间获得数组的长度，应该用向量。

在调用一些数组的方法时，Rust会隐式地将一个数组的引用转换为切片，这样可以直接在一个数组上使用任何切片的方法。

```rust
let mut chaos = [3, 5, 4, 1, 2];
chaos.sort(); // 这里隐式地转换为 &mut [i32]，调用了切片的 sort 方法
assert_eq!(chaos, [1, 2, 3, 4, 5]);
```

关于这一点，[官方文档](https://rustwiki.org/zh-CN/std/primitive.array.html)是这样说的。

> 数组强制转换为 `slices ([T]) `，因此可以在数组上调用 slice 方法。实际上，这提供了用于处理数组的大多数 API。 切片具有动态大小，并且不强制转换为数组。

#### Vectors

在堆上分配的容器。

可以用 `vec!`或者`Vec::new()`来生成新的 vector。

可以动态添加元素:

```rust
let mut primes = vec![2, 3, 5, 7];
primes.push(11);
```

可以用重复表达式来创建 vector：

```rust
fn new_pixel_buffer(rows: usize, cols: usize) -> Vec<u8> {
  vec![0; rows * cols]
}
```

迭代器构造：

```rust
let v: Vec<i32> = (0..5).collect(); // 这里必须指定 v 的类型，因为 collect 方法有很多种
assert_eq!(v, [0, 1, 2, 3, 4]);
```

和数组一样，向量也会在调用一些方法时隐式地转换为切片类型。

一个 `Vec<T>` 包含三个值：指向元素的堆 buffer 的指针，这个是被 `Vec<T>` 所创建和拥有的； buffer 中的容量；包含元素的个数，也就是长度。

知道向量的容量，可以用`Vec::with_capacity`来创建。提升效率，减少重新分配空间。

`capacity`方法返回向量不需要再分配时向量的容量。

一些方法：

* insert
* remove
* pop

可以用 for 在向量上进行迭代。

#### Slices

切片，直接指向引用的数据，看作一个胖指针，指向第一个元素，以及包含 len。

### String Types

#### String Literals

字符串字面量在双引号内。

```rust
// 空格和换行照样输出
println!("In the room the women come and go,
  Singing of Mount Abora");

// 去掉换行
println!("It was a bright, cold day in April, and \
      there were four of us—\
      more or less.")

// raw strings
let default_win_install_path = r"C:\Program Files\Gorillas"; 
let pattern = Regex::new(r"\d+(\.\d+)*");

// 包含双引号
println!(r###"
      This raw string started with 'r###"'.
      Therefore it does not end until we reach a quote mark ('"')
      followed immediately by three pound signs ('###'):
"###);
```

#### Byte Strings

带有 b 前缀的是一个字节字面量。

```rust
let method = b"GET"; // &[u8; 3] 类型
assert_eq!(method, &[b'G', b'E', b'T']);
```

#### Strings in Memory

内存中以UTF-8编码存储。

```rust
let noodles = "noodles".to_string(); // String 底层是 Vec<u8>
let oodles = &noodles[1..]; // &str
let poodles = "ಠ_ಠ"; // &str
```

内存布局：

![img](/img/2022-07-05-Programming-Rust/rust-programming-string.png)

#### String

`&str`像`&[T]`，而 `String`像`Vec<T>`。

创建 String:

```rust
// to_string 方法
let error_message = "too many pets".to_string();

// format!
assert_eq!(format!("{}°{:02}′{:02}′′N", 24, 5, 23),
                     "24°05′23′′N".to_string());

// 数组、向量的 concat、join 方法
let bits = vec!["veni", "vidi", "vici"]; 
assert_eq!(bits.concat(), "venividivici"); 
assert_eq!(bits.join(", "), "veni, vidi, vici");
```

#### Using Strings

Strings support the == and != operators. Two strings are equal if they contain the same characters in the same order (regardless of whether they point to the same location in memory).

#### Other String-Like Types

* Stick to **String** and **&str** for Unicode text.
* When working with filenames, use **std::path::PathBuf** and **&Path** instead.
* When working with binary data that isn’t UTF-8 encoded at all, use **`Vec<u8>`** and **&[u8]**.
* When working with environment variable names and command-line arguments in the native form presented by the operating system, use **OsString** and **&OsStr**.
* When interoperating with C libraries that use null-terminated strings, use **std::ffi::CString** and **&CStr**.

### Type Aliases

类型别名用 `type` ：

```rust
type Bytes = Vec<u8>;
```

## Chapter 4. Ownership and Moves

管理内存，想达到的效果：

* 当我们选择一个时间时，内存可以及时释放
* 内存释放后，不再有指针指向它

目前的内存管理方案：

* GC，垃圾回收。解决了悬垂指针的问题，但是会出现世界停时，也就是我们期望释放内存的时候，它还没有释放。
* 自己完全控制内存，比如C和C++。但是对程序员要求高，使用不当有时也会发生错误。

Rust是怎么解决这个问题的呢？秘密武器就是限制你的程序对指针的使用。给的这些限制会保证安全，但也不会丧失自由度。书中这样写道：

> Rust’s radical wager, the claim on which it stakes its success and that forms the root of the language, is that **even with these restrictions in place, you’ll find the language more than flexible enough for almost every task and that the benefits**—the elimination of broad classes of memory management and concurrency bugs—will justify the adaptations you’ll need to make to your style. The authors of this book are bullish on Rust exactly because of our extensive experience with C and C++. For us, Rust’s deal is a no-brainer.

### Ownership

在Rust中，所有权的概念是内建在语言中的，并且会通过编译器检查强制执行。每个值都有一个所有者，这个所有者决定它的生命周期。当所有者被释放时，它所拥有的值也会被释放。

```rust
{
  let point = Box::new((0.625, 0.5)); // point allocated here 
  let label = format!("{:?}", point); // label allocated here , 这里返回 String
  assert_eq!(label, "(0.625, 0.5)");
}									  // both dropped here
```

内存布局：

![image-20220709110854213](/img/2022-07-05-Programming-Rust/rust-programming-fig4-3.png)

像变量会拥有它的值，结构体会拥有它们的字段，元组、数组以及向量会拥有它们的元素。

再看一个复杂的例子：

```rust
struct Person { name: String, birth: i32 }

let mut composers = Vec::new();
composers.push(Person { name: "Palestrina".to_string(),
                        birth: 1525 });
composers.push(Person { name: "Dowland".to_string(),
                        birth: 1563 });
composers.push(Person { name: "Lully".to_string(),
												birth: 1632 }); 
for composer in &composers {
		println!("{}, born {}", composer.name, composer.birth);
}
```

内存布局：

![image-20220709111433484](/img/2022-07-05-Programming-Rust/image-20220709111433484.png)

每个值都有一个单独的所有者，这个所有者很容易来决定什么时候 drop 它所拥有的值。但是一个单独的值可能会拥有很多其他的值，如上面这个例子中的向量 composers。所有者和它们所拥有的值会形成一个**树**。在每个树的根部是一个变量，当这个变量离开它的作用范围，整棵树也会跟着离开。

Rust通常不会显式地 drop 它的值，而是通过：离开变量的作用域，从向量中删除一个元素，或者其他的。

Rust怎么在这些严格的限制下实现灵活性呢？如下列出了一些方式：

* You can **move** values from one owner to another. This allows you to build, rearrange, and tear down the tree.
* Very simple types like integers, floating-point numbers, and characters are excused from the ownership rules. These are called **Copy** types.
* The standard library provides the reference-counted pointer types **Rc** and **Arc**, which allow values to have multiple owners, under some restrictions.
* You can “**borrow a reference**” to a value; references are non-owning pointers, with limited lifetimes.

### Moves

In Rust, for most types, operations like assigning a value to a variable, passing it to a function, or returning it from a function don’t copy the value: they *move* it. **The source** relinquishes ownership of the value to the destination and **becomes uninitialized**; the destination now controls the value’s lifetime. Rust programs build up and tear down complex structures one value at a time, one move at a time.

看看Python中的变量赋值。

```python
s = ['udon', 'ramen', 'soba'] 
t=s
u=s
```

开始的内存布局：

![image-20220709132630820](/img/2022-07-05-Programming-Rust/image-20220709132630820.png)

执行代码后的内存布局：

![image-20220709131847520](/img/2022-07-05-Programming-Rust/image-20220709131847520.png)

Python has copied the pointer from s into t and u and updated the list’s reference count to 3. Assignment in Python is cheap, but because it creates a new reference to the object, we must maintain reference counts to know when we can free the value.

来看看C++中的实现：

```c++
using namespace std;
vector<string> s = { "udon", "ramen", "soba" }; 
vector<string> t = s;
vector<string> u = s;
```

开始的内存布局

![image-20220709132451059](/img/2022-07-05-Programming-Rust/image-20220709132451059.png)

代码执行后的内存布局：

![image-20220709132539575](/img/2022-07-05-Programming-Rust/image-20220709132539575.png)

Depending on the values involved, assignment in C++ can consume unbounded amounts of memory and processor time. The advantage, however, is that it’s easy for the program to decide when to free all this memory: when the variables go out of scope, everything allocated here gets cleaned up automatically.

Rust中的实现：

```rust
let s = vec!["udon".to_string(), "ramen".to_string(), "soba".to_string()]; 
let t = s;
let u = s;
```

开始的内存分布：

![image-20220709132918760](/img/2022-07-05-Programming-Rust/image-20220709132918760.png)

在`let t = s;`之后，s 的所有权就 move 到 t 上了。

![image-20220709133027720](/img/2022-07-05-Programming-Rust/image-20220709133027720.png)

这时 s 变成未初始化状态，再执行 `let u = s;` 就会报错，因为使用了未初始化的变量。

Consider the consequences of Rust’s use of a move here. Like Python, the assignment is cheap: the program simply moves the three-word header of the vector from one spot to another. But like C++, ownership is always clear: the program doesn’t need reference counting or garbage collection to know when to free the vector elements and string contents.

如果想实现Python那样的引用计数，可以用 Rc 和 Arc，如果想实现C++那样的深拷贝，可以显式调用 clone() 方法。

#### More Operations That Move

move语义的发生：

* 给一个函数传递参数，会将所有权给参数
* 从函数返回一个值会将所有权给调用者
* 创建一个元组，会将值的所有权给元组

Moving values around like this may sound inefficient, but there are two things to keep in mind. First, the moves always apply to the value proper, not the heap storage they own. For vectors and strings, the *value proper* is the three-word header alone; the potentially large element arrays and text buffers sit where they are in the heap. Second, the Rust compiler’s code generation is good at “seeing through” all these moves; in practice, the machine code often stores the value directly where it belongs.

#### Moves and Control Flow

 If it’s possible for a variable to have had its value moved away and it hasn’t definitely been given a new value since, it’s considered uninitialized. 

#### Moves and Indexed Content

We’ve mentioned that a move leaves its source uninitialized, as the destination takes ownership of the value. **But not every kind of value owner is prepared to become uninitialized.** 

```rust
// Build a vector of the strings "101", "102", ... "105"
let mut v = Vec::new(); for i in 101 .. 106 {
      v.push(i.to_string());
  }
// Pull out random elements from the vector.
let third = v[2]; // error: Cannot move out of index of Vec 
let fifth = v[4]; // here too
```

要将 vector 看作一个整体，这里不能单独将 v[2] move 出来。如果确实想拿出其中的元素，有下面一些方法：

```rust
// Build a vector of the strings "101", "102", ... "105"
let mut v = Vec::new(); for i in 101 .. 106 {
      v.push(i.to_string());
}

// 1. Pop a value off the end of the vector:
let fifth = v.pop().expect("vector empty!"); 
assert_eq!(fifth, "105"); // ["101", "102", "103", "104"]

// 2. Move a value out of a given index in the vector, 
// and move the last element into its spot:
let second = v.swap_remove(1);
assert_eq!(second, "102"); // ["101", "104", "103"]

// 3. Swap in another value for the one we're taking out:
let third = std::mem::replace(&mut v[2], "substitute".to_string()); 
assert_eq!(third, "103"); // ["101", "104", "substitute"]
  
// Let's see what's left of our vector.
assert_eq!(v, vec!["101", "104", "substitute"]);
```

来看看循环

```rust
let v = vec!["liberté".to_string(), 
  			    "égalité".to_string(),
                "fraternité".to_string()];

for mut s in v { 
    s.push('!');
    println!("{}", s);
}
```

这里在for循环中，v 的所有权给了for循环，v 就变成未初始化的状态，然后 for 循环分离每一个元素，将所有权给每一个。因为 s 有所有权，就可以在内部修改字符串了。

由于在循环中将 v 的所有权给了循环，因此后面就不能再使用 v 了，下面代码就会报错。

```rust
let v = vec!["liberté".to_string(), 
  				"égalité".to_string(),
  				"fraternité".to_string()];

for mut s in v { 
    s.push('!');
    println!("{}", s);
}
let _u = v; // error[E0382]: use of moved value: `v`
```

如果想得到结构体中的元素值，可以调用 `std::mem::replace`方法，将原来的值用`None`来占位。

```rust
struct Person { name: Option<String>, birth: i32 }
let mut composers = Vec::new();
composers.push(Person { name: Some("Palestrina".to_string()),
  birth: 1525 });  

let first_name = std::mem::replace(&mut composers[0].name, None); 
// let first_name = composers[0].name.take(); // 和上述语句达到的效果一样
assert_eq!(first_name, Some("Palestrina".to_string())); 
assert_eq!(composers[0].name, None);
```

### Copy Types: The Exception to Moves

Assigning a value of a Copy type copies the value, rather than moving it. The source of the assignment remains initialized and usable, with the same value it had before. Passing Copy types to functions and constructors behaves similarly.

The standard Copy types include all the machine integer and floating-point numeric types, the char and bool types, and a few others. A tuple or fixed-size array of Copy types is itself a Copy type.

Only types for which a simple bit-for-bit copy suffices can be Copy. As a rule of thumb, any type that needs to do something special when a value is dropped cannot be Copy.

自定义的结构体或者枚举类型可以用属性宏实现 Copy trait。

```rust
#[derive(Copy, Clone)]
struct Label { number: u32 }
```

 In Rust, every move is a byte-for-byte, shallow copy that leaves the source uninitialized. Copies are the same, except that the source remains initialized. 

One of Rust’s principles is that costs should be apparent to the programmer. Basic operations must remain simple. Potentially expensive operations should be explicit, like the calls to `clone` in the earlier example that make deep copies of vectors and the strings they contain.

### Rc and Arc: Shared Ownership

The Rc and Arc types are very similar; the only difference between them is that an Arc is safe to share between threads directly—the name Arc is short for *atomic reference count*—whereas a plain Rc uses faster non-thread-safe code to update its reference count. 

前面的例子用Rust实现Python中引用计数的效果：

```rust
use std::rc::Rc;

// Rust can infer all these types; written out for clarity
let s: Rc<String> = Rc::new("shirataki".to_string()); 
let t: Rc<String> = s.clone();
let u: Rc<String> = s.clone();
```

内存布局：

![image-20220709143228683](/image-20220709143228683.png)

Rust’s memory and thread-safety guarantees depend on ensuring that no value is ever simultaneously shared and mutable. Rust assumes the referent of an Rc pointer might in general be shared, so it must not be mutable.

However, Rust does provide ways to create mutable portions of otherwise immutable values; this is called *interior mutability*. 这可能造成循环引用。

想要用Rc避免循环引用。You can sometimes avoid creating cycles of Rc pointers by using *weak pointers*, **std::rc::Weak**, for some of the links instead. 

## Chapter 5. References

Rust also has non-owning pointer types called *references*, which have no effect on their referents’ lifetimes.

In fact, it’s rather the opposite: **references must never outlive their referents**. You must make it apparent in your code that no reference can possibly outlive the value it points to. To emphasize this, Rust refers to creating a reference to some value as *borrowing* the value: what you have borrowed, you must eventually return to its owner.

### **References to Values**

 A reference lets you access a value without affecting its ownership. References come in two kinds:

* A *shared reference* lets you read but not modify its referent. However, you can have as many shared references to a particular value at a time as you like. The expression `&e `yields a shared reference to e’s value; if e has the type T, then &e has the type &T, pronounced “ref T.” Shared references are Copy.
* If you have a *mutable reference* to a value, you may both read and modify the value. However, **you may not have any other references of any sort to that value active at the same time**. The expression &mut e yields a mutable reference to e’s value; you write its type as &mut T, which is pronounced “ref mute T.” Mutable references are not Copy.

```rust
fn show(table: &Table) {
  for (artist, works) in table {
  	println!("works by {}:", artist); 
    for work in works {
      println!("  {}", work);
    }
  } 
}
```

Iterating over a shared reference to a HashMap is defined to produce shared references to each entry’s key and value: artist has changed from a `String` to a `&String`, and works from a `Vec<String>` to a `&Vec<String>`.

The inner loop is changed similarly. Iterating over a shared reference to a vector is defined to produce shared references to its elements, so work is now a &String. No ownership changes hands anywhere in this function; it’s just passing around non-owning references.

When we pass a value to a function in a way that moves ownership of the value to the function, we say that we have passed it ***by value***. If we instead pass the function a reference to the value, we say that we have passed the value ***by reference***. 

### **Working with References**

#### **Rust References Versus C++ References**

In Rust, references are created explicitly with the `&` operator, and dereferenced explicitly with the `*` operator.

Since references are so widely used in Rust, the . operator implicitly dereferences its left operand, if needed.

```rust
struct Anime { name: &'static str, bechdel_pass: bool };
let aria = Anime { name: "Aria: The Animation", bechdel_pass: true };
let anime_ref = &aria;
assert_eq!(anime_ref.name, "Aria: The Animation"); 
  
// Equivalent to the above, but with the dereference written out:
assert_eq!((*anime_ref).name, "Aria: The Animation");
```

The `println!` macro used in the show function expands to code that uses the `. `operator.

The `. `operator can also implicitly borrow a reference to its left operand, if needed for a method call.

```rust
let mut v = vec![1973, 1968];
v.sort(); // implicitly borrows a mutable reference to v
(&mut v).sort(); // equivalent, but more verbose
```

#### **Assigning References**

```rust
let x = 10; 
let y = 20; 
let mut r = &x;

if b { r = &y; }

assert!(*r == 10 || *r == 20);
```

The reference r initially points to x. But if b is true, the code points it at y instead.

![image-20220711094925795](/img/2022-07-05-Programming-Rust/image-20220711094925795.png)

#### **References to References**

```rust
struct Point { x: i32, y: i32 }
let point = Point { x: 1000, y: 729 }; 
let r: &Point = &point;
let rr: &&Point = &r;
let rrr: &&&Point = &rr;
```

The . operator follows as many references as it takes to find its target.

```rust
assert_eq!(rrr.y, 729);
```

![image-20220711095223237](/img/2022-07-05-Programming-Rust/image-20220711095223237.png)

So the expression `rrr.y`, guided by the type of `rrr`, actually traverses three references to get to the `Point` before fetching its `y` field.

#### **Comparing References**

```rust
let x = 10;
let y = 10; 

let rx = &x;
let ry = &y; 

let rrx = &rx;
let rry = &ry; 

assert!(rrx <= rry);
assert!(rrx == rry);
```

比较操作符和`.`操作符一样可以解引用到最终的目标值。

可以用`std::ptr::eq`来比较地址是否相等：

```rust
assert!(rx == ry); // their referents are equal 
assert!(!std::ptr::eq(rx, ry)); // but occupy different addresses
```

比较操作符两端的类型必须相同：

```rust
assert!(rx == rrx); // error: type mismatch: `&i32` vs `&&i32` 
assert!(rx == *rrx); // this is okay
```

#### **References Are Never Null**

Rust references are never null. There is no default initial value for a reference (you can’t use any variable until it’s been initialized, regardless of its type) and Rust won’t convert integers to references (outside of unsafe code), so you can’t convert zero into a reference.

In Rust, if you need a value that is either a reference to something or not, use the type `Option<&T>`. 

#### **Borrowing References to Arbitrary Expressions**

Rust lets you borrow a reference to the value of any sort of expression at all:

```rust
fn factorial(n: usize) -> usize { 
  (1..n+1).product()
}

let r = &factorial(6); 
// Arithmetic operators can see through one level of references. 
assert_eq!(r + &1009, 1729);
```

Rust simply creates an anonymous variable to hold the expression’s value and makes the reference point to that. The lifetime of this anonymous variable depends on what you do with the reference:

* If you immediately assign the reference to a variable in a let statement (or make it part of some struct or array that is being immediately assigned), then Rust makes the anonymous variable live as long as the variable the let initializes. 
* Otherwise, the anonymous variable lives to the end of the enclosing statement.

#### **References to Slices and Trait Objects**

***fat pointers***:

* **A reference to a slice** is a fat pointer, carrying the starting address of the slice and its length. 
* Rust’s other kind of fat pointer is a ***trait object***, a reference to a value that implements a certain trait. A trait object carries a value’s address and a pointer to the trait’s implementation appropriate to that value, for invoking the trait’s methods.

### **Reference Safety**

#### **Borrowing a Local Variable**

You can’t borrow a reference to a local variable and take it out of the variable’s scope:

```rust
{
	let r;
  {
  	let x = 1;
  	r = &x; 
  }
	assert_eq!(*r, 1); // bad: reads memory `x` used to occupy 
}
```

The variables r and x both have a lifetime, extending from the point at which **they’re initialized until the point that the compiler can prove they are no longer in use**. The third lifetime is that of a reference type: the type of the reference we borrow to x and store in r.

**We say that the variable’s lifetime must *contain* or *enclose* that of the reference borrowed from it.**  变量的生命周期必须包含或涵盖从它那里借来的引用的生命期。

Here’s another kind of constraint: **if you store a reference in a variable r, the reference’s type must be good for the entire lifetime of the variable, from its initialization until its last us**.

**We say that the reference’s lifetime must contain or enclose the variable’s.**  引用的生命期必须包含或涵盖保存它的变量的生命期。

The first kind of constraint limits how large a reference’s lifetime can be, while the second kind limits how small it can be.

下面的生命周期是正确的：

![image-20220711103418336](/img/2022-07-05-Programming-Rust/image-20220711103418336.png)

First, understand the constraints arising from the way the program uses references; then, find lifetimes that satisfy them. 

#### **Receiving References as Function Arguments**

* **Every static must be initialized.**
* Mutable statics are inherently not thread-safe (after all, any thread can access a static at any time), and even in single-threaded programs, they can fall prey to other sorts of reentrancy problems. For these reasons, you may access a mutable static only within an unsafe block. In this example we’re not concerned with those particular problems, so we’ll just throw in an unsafe block and move on.

`fn f<'a>(p: &'a i32)`, we’re defining a function that takes a reference to an i32 with any given lifetime 'a.

In other words, we were unable to write a function that stashed a reference in a global variable without reflecting that intention in the function’s signature. In Rust, a function’s signature always exposes the body’s behavior.

 If we do see a function with a signature like `g(p: &i32)` (or with the lifetimes written out, `g<'a>(p: &'a i32)`), we can tell that it *does not* stash its argument `p` anywhere that will outlive the call.

#### **Passing References to Functions**

```rust
// This could be written more briefly: fn g(p: &i32), 
// but let's write out the lifetimes for now.
fn g<'a>(p: &'a i32) { ... }

let x = 10; 
g(&x);
```

从g的签名看，Rust知道它不会把p保存到超出调用生命周期的变量里；任何涵盖调用的生命周期都满足`'a`。

```rust
fn f(p: &'static i32) { ... }
       
let x = 10; 
f(&x); // error
```

这里 `&x`不能存活得比`x`长，而函数签名要求`&x`活得和`&'static`一样长，这就无法得到满足，因此报错。

#### **Returning References**

```rust
// v should have at least one element.
fn smallest(v: &[i32]) -> &i32 { 
  let mut s = &v[0];
  for r in &v[1..] {
  	if *r < *s { s = r; } 
  }
  s
}
```

如果一个函数只有一个引用作为参数并返回一个引用，则它们拥有相同的生命周期。

#### **Structs Containing References**

当引用类型出现在另一个类型的定义中时，必须写出其生命周期。

```rust
struct S {
	r: &'static i32 // 这里 r 必须写上生命周期
}
```

或者

```rust
struct S<'a>{
  r: &'a i32
}
```

保存在r中的任何引用的生命周期最好包含`'a`，而`'a`也必须比保存在S的任何值都长寿。

 The lifetime of any reference you store in r had better enclose 'a, and 'a must outlast the lifetime of wherever you store the S.

The assignment s = S { ... } stores this S in a variable whose lifetime extends to the end of the example, constraining 'a to outlast the lifetime of s.

And now Rust has arrived at the same contradictory constraints as before: 'a must not outlive x, yet must live at least as long as s.

#### **Distinct Lifetime Parameters**

```rust
struct S<'a> { 
  x: &'a i32,
  y: &'a i32 
}
```

如下代码：

```rust
let x = 10; let r;
{
	let y = 20; 
  {
    let s = S { x: &x, y: &y };
    r = s.x; 
  }
}
println!("{}", r);
```

If you work through the code carefully, you can follow its reasoning:

* Both fields of S are references with the same lifetime 'a, so Rust must find a single lifetime that works for both s.x and s.y.
* We assign r = s.x, requiring 'a to enclose r’s lifetime. (r <= 'a)
* We initialized s.y with &y, requiring 'a to be no longer than y’s lifetime. ('a <= y)

#### **Omitting Lifetime Parameters**

三条规则（见[生命周期与引用有效性](https://rustwiki.org/zh-CN/book/ch10-03-lifetime-syntax.html)）：

* 第一条规则是每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。

* 第二条规则是如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。

* 第三条规则是如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，说明是个对象的方法(method)(译者注： 这里涉及 Rust 的面向对象参见 17 章), 那么所有输出生命周期参数被赋予 `self` 的生命周期。第三条规则使得方法更容易读写，因为只需更少的符号。

### Sharing Versus Mutation
Throughout its lifetime, a shared reference makes its referent read-only: you may not assign to the referent or move its value elsewhere. 

1. *Shared access is read-only access.*

   Values borrowed by shared references are read-only. Across the lifetime of a shared reference, neither its referent, nor anything reachable from that referent, can be changed *by anything*. There exist no live mutable references to anything in that structure, its owner is held read-only, and so on. It’s really frozen.

2. *Mutable access is exclusive access.*

   A value borrowed by a mutable reference is reachable exclusively via that reference. Across the lifetime of a mutable reference, there is no other usable path to its referent or to any value reachable from there. The only references whose lifetimes may overlap with a mutable reference are those you borrow from the mutable reference itself.

### **Taking Arms Against a Sea of Objects**

![image-20220712115540366](/img/2022-07-05-Programming-Rust/image-20220712115540366.png)

![image-20220712115559036](/img/2022-07-05-Programming-Rust/image-20220712115559036.png)

## Chapter 6. Expressions

Rust中绝大多数都是表达式。

### An Expression Language

* 在Rust中，if 和 match 都可以产生值。一个match表达式可以给一个函数或者宏来传参。
* Rust 中没有三元操作符。
* Rust中所有的控制流程工具是表达式。

### Precedence and Associativity

* 闭包的优先级在最后。
* All of the operators that can usually be chained are left-associative. 所有这些操作符在链式操作时都具有左关联性。
* 比较操作符，赋值操作符（assignment iperators）以及范围操作符（`..`range operators）不能链式链接。

### Blocks and Semicolons

* block 的值是最后一个表达式的值。
* **let 声明必须要有分号。**
* **如果 if 表达式没有 else，那么总是返回 ()。**

### Declarations

* let 声明必须要有分号。
* let 声明可以只声明变量而不初始化它。
* 在没有初始化一个变量之前是不能使用它的。
* 变量遮蔽，可以是不同的类型。
* **一个块可以包含*特性项*（ item） declarations**，就是指任何可以在程序或模块的全局中出现的声明，比如 fn，struct 或者 use。
* 嵌套的 fn 不能使用 scope 中的局部变量。
* 块中甚至可以包含一个完整的模块。

### if and match

* **if 表达式的 condition 必须是 bool 类型的**，Rust 不会将数值或者指针隐式转换成 Boolean 类型。
* 在 if 表达式形式中，围绕条件的圆括号不是必需的。事实上，rustc在发现不必要的圆括号时会给出警告。但花括号是必需的。
* 一个只有 if 没有 else 的表达式相当于有一个 空的else 块。因此**如果 if 表达式没有 else，那么总是返回 `()`。**
* 编译器可以用一个跳转表（ jump table ）来优化 match 表达式。如果 match 的每个分支都产生一个常量值，那么也可以应用同样的优化。此时，编译器会构建一个这些值的数组，而 match 会被编译为对数组的访问。除了边界检查，编译后的代码中根本没有分支。
* **Rust 禁止 match 表达式没有覆盖到所有情况。**
* **一个 if 表达式的所有块必须产生相同类型的值，对于 match 表达式也是一样的**，match 表达式的所有分支也都必须返回相同类型的值。

### if let

```rust
if let pattern = expr {
  block1
} else {
  block2
}
```

等价于

```rust
match expr {
  pattern => { block1 }
  _ => { block2 }
}
```

### Loops

有四种 looping 表达式：

```rust
while condition {
  block
}

while let pattern = expr {
  block
}

loop {
  block
}

for pattern in iterable {
  block
}
```

* while 和 for  的值一直都是 ()，循环的值是`()`。
* 用 loop 可以无限循环，除非遇到 break，return或者线程 panics。
* **`for` 循环每迭代一个值就会消耗一个值，有时候可以使用引用。**

### Control Flow in Loops

* break 只能在 loop 中，不能在 match 中。
* **可以给 break 一个表达式，这个表达式的值就是 loop 的值。**
* 自然地，**所有 break 表达式在一个循环中必须是相同类型的**，并且这个类型就是 loop 的类型。
* **循环可以加上生命周期的标签**，然后可以 break 这个标签，break 也可以同时带上值表达式。
* 标签同样可以用于 continue。

### return Expressions

* return 表达式会退出当前函数，并且给调用者返回一个值。
* 可以将函数体看成一个块表达式。
* **return 可以放弃正在运行中的工作。**

### Why Rust Has loop

Rust编译器分析你的程序的控制流程：

* 检查返回类型
* 检查局部变量不会使用未初始化的值
* 警告不可到达的代码

以上这些称为**流敏感**（flow-sensitive）分析。

Rust追求简单，其流敏感分析压根不会检查循环条件，而只是假设程序中的任何条件不是true就是false。

`!` 表示是一个发散函数。比如 break，return，panic!()，无穷循环，std::process::exit() ，它们都不以惯常的方式结束，这些可以跳过之前说的类型一致规则。

### Function and Method Calls

* `.location()` 方法可能取 player 的值或者引用。
* `.` 操作符可以自动的解引用 `player`或者借一个引用。可以接收值或引用。
* 静态方法`Vec::new()` ，非静态方法通过值调用：`my_vec.len()`。
* 对于泛型，函数或者方法调用要用比目鱼 turbofish 操作符，比如：`return Vec::<i32>::with_capcity(1000);`

### Fields and Elements

点操作符或者方括号的左侧是一个引用或者智能指针，那么它可以被自动地解引用，这样的表达式被称为 **`lvalues`**，它们可以出现在一个 assignment 的左侧。

操作符：

```rust
..
a ..
.. b
a .. b
..= b
a ..= b
```

### Reference Operators

* &
* & mut
* `*`操作符可以用来解引用

### Arithmetic, Bitwise, Comparsion, and Logical Operators

* **不能将`-`和`+` 用到无符号的数值类型上。**
* `%`也可以用在浮点数上。
* `!`表示按位非。
* 位操作符的优先级高于比较操作符的优先级。
* 比较操作符两端的类型必须都是 bool 类型。

### Assignment

* Rust中可以有复合操作符：`+=` `-=` `*=`
* Rust**不支持**链式 assignment，比如 `a = b = 3`
* Rust**没有**自增和自减操作符：`++`和`--`。

### Type Casts

可以用 `as` 进行类型转换。几种类型转换是允许的：

* 任何内置的数值之间可以转换。
* bool、char 或者 C-like enum 类型可以转换为整数。u8 可能转换为 char 类型。
* 有些含有 unsafe 的指针类型的转换也是允许的。

一些自动转换，被称为*解引用强制转换*（deref coercions）：

* `&String` -> `&str`
* `&Vec<i32>` -> `&[i32]`
* `&Box[Chessboard]` -> `&Chessboard`

它们实现了内置的 Deref trait。		

### Closures

```rust
let is_even = |x| x % 2 == 0;
```

Rust自动推断类型。调用闭包就像调用一个函数那样：

```rust
assert_eq!(is_even(14), true);
```

### Onward

表达式只是我们思考 running code。

## Chapter 7.  Error Handling

### Panic

panic 是程序本身出现一个bug。比如：

* 数组越界
* 整数除零
* 对于一个 `Result`调用`.expect()`。
* 断言失败

panic时要么展开栈，要么中止程序，展开栈是默认的。

#### Unwinding

panic发生：

* 终端打印错误信息
* 运行 `RUST_BACKTRACE=1`会将栈展开。资源会关闭，会调用drop方法。
* 最后，线程退出。

panic不是崩溃，不是未定义行为。panic像Java中的运行时异常，其行为是明确定义的，只是不应该发生而已。

panic是安全的，它不违反任何Rust的安全规则。不会在内存中导致悬空指针或初始化一半的值，Rust展开栈后，进程的其他部分还可以运行。

panic是线程级别的。

有一种方法可以捕获栈展开，运行panic线程存活并继续运行。使用标准库中的`std::panic::catch_unwind()`函数。

#### Aborting

* 如果`.drop()`方法触发了第二个panic当Rust在第一个panic后尝试 clean up，这会被当作是致命的。Rust会停止展开并且直接终止整个流程。
* 编译加上`-C panic=abort`，第一个 panic 就立马终止程序。

### Result

如果不使用Result值，会得到一个警告。

#### Catching Errors

用 match 来处理 error

```rust
match get_weather(hometown) {
  Ok(report) => {
    display_weather(hometown, &report);
  }
  Err(err) => {
    println!("error querying the weather: {}", err);
    schedule_weather_retry();
  }
}
```

但是上面这种方式太冗长了。这里有一些方法：

```rust
// 1. 返回一个bool值
result.is_ok()
result.is_err()
// 2. 返回成功的值，返回 Option<T>, 成功就返回值，失败就返回 None
result.ok()
// 3. 返回失败的值
result.err()
// 4. 返回成功的值，否则就返回 fallback，抛弃错误的值，只有在存在适当后备值的情况下才可以使用这个方法
result.unwrap_or(fallback)
// 5. 返回成功的值，否则返回 fallback_fn，这里可以是一个函数或者闭包，这个方法适合计算
//    后备值如果用不上会浪费的情况。只有在返回错误结果时才会调用 fallback_fn
result.unwrap_or_else(fallback_fn)
// 6. 返回成功的值，否则就 panic
result.unwrap()
result.expect(message) // panic 时打印信息
// 7. 将 Result<T, E> 转换为 Result<&T, &E>，这对于还要用原来的 result 的值很有用
result.as_ref()
// 8. 将 Result<T, E> 转换为 Result<&mut T, &mut E>
result.as_mut()
```

#### Result Type Aliases

在标准库 std::io 模块中有：

```rust
pub type Result<T> = result::Result<T, Error>;
```

这里用了 type aliases。

#### Printing Errors

`std::io::Error`、`std::fmt::Error`、`std::str::Utf8Error`等等实现了`std::error::Error` trait。一些公用的方法：

* `println!()`

  打印错误，用`{}`可以打印简短的错误信息，用`{:?}`的Debug模式打印出错误。

* `err.to_string()`

  返回错误信息字符串。

* `err.source()`

  返回一个Option，包含潜在的错误。

```rust
use std::error::Error;
use std::io::{Write, stderr};

/// Dump an error message to `stderr`.
///
/// If another error happens while building the error message or 
/// writing to `stderr`, it is ignored.
fn print_error(mut err: &dyn Error) {
  let _ = writeln!(stderr(), "error: {}", err); 
  while let Some(source) = err.source() {
  	let _ = writeln!(stderr(), "caused by: {}", source);
    err = source;
  }
}

```

想要展开错误，在Rust不稳定版本下，可以使用`anyhow`包。

#### Propagating Errors

如果错误发生，我们希望调用者来处理。

```rust
let weather = get_weather(hometown)?;
let weather = try!(get_weather(hometown));
```

`?`操作符的行为取决于这个函数的返回值：

* 如果是成功结果，那么它会打开Result并取出其中的成功值。
* 如果是错误结果，那么它会立即从闭合函数中返回，将错误结果沿调用链向上传播。

`?`操作符一样可以使用match表达式实现。

`?`可以类似地用在`Option`类型的值上面。In a function that returns Option, you can use ? to unwrap a value and return early in the case of None.

#### Working with Multiple Error Types

* `?`不能将`std::num::ParseIntError`转换为`std::io::Error`类型。
* 所有标准库中的错误都可以转成`Box<dyn std::error::Error + Send + Sync + 'static'>`类型。

```rust
type GenericError = Box<dyn std::error::Error + Send + Sync + 'static>;
type GenericResult = Result<T, GenericError>;
```

`?`会调用`GenericError::from()`方法来进行自动转换。

* 如果想要其中一种特别的错误类型进行处理，其它的错误进行传播的话，可以用泛型方法`error.downcast_ref::<ErrorType>()`

#### Dealing with Errors That "Can't Happen"

* 当我们不想处理不会发生的错误时，用`.unwrap()`方法。
* 也可以用`.expect(message)`方法。

#### Ingoring Errors

```rust
let _ = writeln!(stderr(), "error: {}", err);
```

用 `let _ = ...`用来禁止警告。

#### Handling Errors in main()

* 一般情况下在 main 函数中不能用 `?` ，因为 `main()` 函数的返回类型不是 `Result`。
* 最简单的方法是用 `.expect()`方法。发生错误时会 panic 并且返回一个非零的退出代码。
* 或者可以返回一个 `Result` 类型，这样就能用`?`，同时用debug `{:?}`方式打印出错误。

```rust
fn main() -> Result<(), TideCalcError> { }
```

* 或者自己处理相应的错误。用`print_error()`方法结合`if let` 打印简介的错误信息。

#### Declaring a Custom Error Type

* 可以自定义`Error`类型，用 struct 实现。Errors 应该实现`fmt::Display`和 `std::error::Error` trait。
* 或者直接用`thiserror` 中的，加上属性宏`#[derive(Error)]`。

```rust
use thiserror::Error;
#[derive(Error, Debug)]
#[error("{message:} ({line:}, {column})")]
pub struct JsonError {
  message: String,
  line: usize,
  column: usize,
}
```

#### Why Results?

* 在代码中记录错误，对错误做出决定。
* 允许错误传播，传播路径可见。
* 每一个函数都有返回类型，这样更清楚函数可不可能失败。
* Rust 会检查 Result 类型的值是否被使用。
* 因为 Result 是一个数据类型，因此更容易处理一系列成功或者失败的值，也更容易存储。

## Chapter 8. Crates and Modules

### Crates

Rust programs are made of *crates*. Each crate is a complete, cohesive unit: all the source code for a single library or executable, plus any associated tests, examples, tools, configuration, and other junk.

要了解包是什么，以及它们如何协作，可以对一个使用了依赖的已有项目运行 `cargo build` 命令，同时加上 `--verbose` 标记。

Cargo 的依赖会形成  *dependency graph* 。

Cargo 会使用 `--crate-type lib`选项，会告诉 rustc 不要去找 main() 函数，而是生成一个 `.rlib` 文件，其中包含编译后的代码，可以供以后在创建库或者`.rlib`文件使用。

> When compiling libraries, Cargo uses the -- crate-type lib option. This tells rustc not to look for a main() function but instead to produce an *.rlib* file containing compiled code that can be used to create binaries and other *.rlib* files.
>

Cargo使用 `--crate-type bin` 选项，编译结果将是一个针对目标平台的二进制可执行文件。

运行每个 rustc 命令时，Cargo会通过`--extern`选项给出当前包用到的每个库的文件名。这样，当rustc看到`use image::png::PNGEncoder`这行代码时，它就知道到磁盘的什么位置去找这个库编码后的代码了。Rust编译器需要访问`.rlib`文件，因为其中包含第三方库编译后的代码。Rust会将这些代码静态链接到最终的可执行文件上。`.rlib`文件也包含类型信息，Rust可以据此检查我们代码中用到的库特性确实在对应的包里存在，从而保证正确地使用它们。这个文件里还包含包的公共内联函数、泛型和宏的一个副本，这些特性直到Rust遇到调用它们的代码时才会编译为机器码。

`cargo build --release`产生优化代码，优化代码的运行速度更快，但编译时间比较长，而且不会检查整型溢出，还会跳过`debug_assert!()`断言，另外它们针对 panic 生成的栈追踪信息一般不太可靠。

#### Editions

To evolve without breaking existing code, Rust uses *editions*. The 2015 edition of Rust is compatible with Rust 1.0. The 2018 edition changed **async** and **await** into keywords, streamlined the module system, and introduced various other language changes that are incompatible with the 2015 edition. 

Rust promises that the compiler will always accept all extant editions of the language, and programs can freely mix crates written in different editions. It’s even fine for a 2015 edition crate to depend on a 2018 edition crate. In other words, **a crate’s edition only affects how its source code is construed; edition distinctions are gone by the time the code has been compiled**. This means there’s no pressure to update old crates just to continue to participate in the modern Rust ecosystem. Similarly, **there’s no pressure to keep your crate on an older edition to avoid inconveniencing its users**. **You only need to change editions when you want to use new language features in your own code.**

If you have a crate written in an older edition of Rust, the **cargo fix** command may be able to help you automatically upgrade your code to the newer edition. The Rust Edition Guide explains the cargo fix command in detail.

#### Build Profiles

构建分析

| 命令行               | Cargo.toml使用的区块 |
| -------------------- | -------------------- |
| cargo build          | [profile.dev]        |
| cargo build --realse | [profile.realse]     |
| cargo test           | [profile.test]       |

想要同时启用优化和调试，可以这样设置：

```toml
[profile.realse]
debug = true # 在发布构建中启用调试标记
```

这里debug设置控制 rustc 中的 -g 选项。有了这个配置，再执行 `cargo build --realse`，就可以得到一个带有调试符号的二进制文件。优化设置不受影响。

### Modules

```rust
mod spores {
  use cells::{Cell, Gene};

  /// A cell made by an adult fern. It disperses on the wind as part of
  /// the fern life cycle. A spore grows into a prothallus -- a whole
  /// separate organism, up to 5mm across -- which produces the zygote
  /// that grows into a new fern. (Plant sex is complicated.)
  pub struct Spore { 
    ...
  }

  /// Simulate the production of a spore by meiosis.
  pub fn produce_spore(factory: &mut Sporangium) -> Spore { 
    ...
  }

  /// Extract the genes in a particular spore.
  pub(crate) fn genes(spore: &Spore) -> Vec<Gene> { 
    ...
  }
  /// Mix genes to prepare for meiosis (part of interphase).
  fn recombine(parent: &mut Cell) { 
    ...
  }

  ...
}
```

One function is marked pub(crate), **meaning that it is available anywhere inside this crate**, but isn’t exposed as part of the external interface. It can’t be used by other crates, and it won’t show up in this crate’s documentation.

Anything that isn’t marked pub is private and **can only be used in the same module** in which it is defined, or **any child modules**.

#### Nested Modules

mod可以嵌套。

It’s also possible to specify`pub(super)`, making an item visible to the parent module only, and `pub(in <path>)`, which makes it visible in a specific parent module and its descendants.

#### Modules in Separate Files

These three options—modules in their own file, modules in their own directory with a *mod.rs*, and modules in their own file with a supplementary directory containing submodules—give the module system enough flexibility to support almost any project structure you might desire.

#### Paths and Imports

用`::`操作符。

```rust
if s1 > s2 {
	std::mem::swap(&mut s1, &mut s2);
}
```

或者用`use`导入。

```rust
use std::mem;

if s1 > s2 {
	mem::swap(&mut s1, &mut s2);
}
```

一次导入多个模块：

```rust
use std::collections::{HashMap, HashSet}; // import both
use std::fs::{self, File}; // import both `std::fs` and `std::fs::File`. 
use std::io::prelude::*; // import everything
```

导入时起别名

```rust
use std::io::Result as IOResult;

// This return type is just another way to write `std::io::Result<()>`:
fn save_spore(spore: &Spore) -> IOResult<()> 
...
```

模块不会自动从自己的父模块继承名字。关键字super是父模块的一个别名，self则是当前模块的一个别名，关键词 crate 表示包含当前模块的包。

```rust
// proteins/synthesis.rs
use crate::proteins::AminoAcid; // explicitly import relative to crate root

pub fn synthesize(seq: &[AminoAcid]) // ok 
...
```

使用相对于根的路径，这样当模块移动时，也是有效的。

`use super::*`可以让子模块获得父模块的私有项。

绝对路径：

```rust
// 外部 crate
use ::image::Pixels; // the `image` crate's `Pixels`
// 自己定义的模块
use self::image::Sampler; // the `image` module's `Sampler`
```

#### **The Standard Prelude**

Furthermore, a few particularly handy names, like Vec and Result, are included in the *standard prelude* and automatically imported. Rust behaves as though every module, including the root module, started with the following import:

```rust
use std::prelude::v1::*;
```

The standard prelude contains a few dozen commonly used traits and types.

参考[`std::prelude`](https://rustwiki.org/zh-CN/std/prelude/index.html)。

#### **Making use Declarations pub**

```rust
// in plant_structures/mod.rs
...
pub use self::leaves::Leaf; 
pub use self::roots::Root;
```

This means that Leaf and Root are public items of the plant_structures module. They are still simple aliases for `plant_structures::leaves::Leaf` and `plant_structures::roots::Root`.

#### **Making Struct Fields pub**

```rust
pub struct Fern {
	pub roots: RootSet, 
  pub stems: StemSet
}
```

Outside the module, only public fields are accessible.

#### **Statics and Constants**

```rust
pub const ROOM_TEMPERATURE: f64 = 20.0; // degrees Celsius
pub static ROOM_TEMPERATURE: f64 = 68.0; // degrees Fahrenheit
```

A constant is a bit like a C++ #define: the value is compiled into your code every place it’s used. A static is a variable that’s set up before your program starts running and lasts until it exits. Use constants for magic numbers and strings in your code. Use statics for larger amounts of data, or any time you need to borrow a reference to the constant value.

There are no mut constants. 

### **Turning a Program into a Library**

The first step is to factor your existing project into two parts: **a library crate**, which contains all the shared code, and **an executable**, which contains the code that’s only needed for your existing command-line program.

By default, cargo build looks at the files in our source directory and figures out what to build. When it sees the file *src/lib.rs*, it knows to build a library. The code in *src/lib.rs* forms the *root module* of the library. Other crates that use our library can only access the public items of this root module.

### **The src/bin Directory**

We can keep our program and our library in the same crate, too. Put this code into a file named *src/bin/efern.rs*:

```rust
use fern_sim::{Fern, run_simulation};
fn main() {
  let mut fern = Fern {
  	size: 1.0,
    growth_rate: 0.001
   };
  run_simulation(&mut fern, 1000);
  println!("final fern size: {}", fern.size);
}
```

Because we’ve put this file into *src/bin*, Cargo will compile both the fern_sim library and this program the next time we run cargo build. We can run the efern program using cargo run --bin efern. Here’s what it looks like, using --verbose to show the commands Cargo is running.

```toml
[dependencies]
fern_sim = { path = "../fern_sim" }
```

可以将 fern_sim 作为一个单独的项目，然后在 Cargo.toml 上加上依赖。

### Attributes

Rust程序中的任何 item 都可以用属性来修饰。

* `#[allow(non_camel_case_types)]` 可以不使用驼峰命名法。
* `#[cfg]` 条件编译
* `#[inline]` 内联函数。当出现一个函数或者一个方法定义在一个 crate 中，而我们在另一个 crate 中去调用，可以显式使用这个属性。
  * `#[inline(always)]` 要求函数在每一次调用的点都展开内联
  * `#[inline(never)]` 要求一个函数从不进行内联
* `#[cfg]`和`#[allow]`可以用在一整个模块或者里面的任何东西，但是 `#[inline]` 和 `#[test]`只能用在单个项上面。
* 想在整个 crate 上面绑定属性，在main.rs`或者`lib.rs`文件顶部中用`#!。`#!`也可以用在函数，结构体中，但是它只能一贯地用在文件的开头，来绑定给整个模块或者 crate 绑定一个属性。而有的属性总是用 `#!`，因为它只能作用在整个 crate 上，比如 `#![feature]`。
* `#![feature]` 用来表示 Rust 语言不稳定的 features，这些是实验性质的。当后面这个 feature 稳定之后，编译器就会警告，建议移除 `#![feature]`。

### Tests and Documention

* `#[test]` 标记函数，表明这个是测试函数，用`cargo test` 可以测试所有测试函数，如果只想测试某一个，可以用`cargo test name`来测试具体的函数。
* `assert!(expr)`：如果表达式为真就通过测试，否则 panic。
* `assert_eq!(v1, v2)`：判别 v1 和 v2 是否相等。
* 如果只想在 debug 模式下检验是否相等，可以用 `debug_assert!` 和 `debug_assert_eq!`。
* `#[should_panic]` 表示会 panic。或者返回`Result<(), E>`。
* `cargo build`和`cargo build --realse`会跳过`#[test]`测试代码。
* 当运行 `cargo test` ，cargo 会编译两次代码。一次是常规的，还有一次是测试程序并启用测试套件。
* `#[cfg(test)]`标记整个模块。
* 一般 cargo test 会启用多线程来一次运行多个测试，用 `cargo test name` 和 `cargo test -- --test-threads 1`来限制只用一个线程测试。
* `cargo test -- --no-capture`：也输出那些通过的测试。

#### Integration Tests

* 集成测试，可以用来测试一些公共的API，站在用户的角度。
* 单独建立一个 `tests`的文件夹，放在和 `src` 同一个路径下，当运行 `cargo test` 的时候，集成测试和单元测试都会运行。如果只想运行集成测试，可以用命令`cargo test --test unfurl`来运行一个具体的集成测试。

#### Documentation

```rust
cargo doc --no-deps --open
```

* --no-deps ：只生成自己的文档，不生成所有它依赖的 crates。
* --open：在浏览器中打开文档。
* 生成的文档放在 target/doc 目录下。
* `///`的文档注释类似`#[doc]`属性。而`//!`和`#![doc]`一样。
* 文档注释中markdown 的链接也可以链接到相应代码的文档。
* One special feature of doc comments in Rust is that Markdown links can use Rust item paths, like leaves::Leaf, instead of relative URLs, to indicate what they refer to.

```rust
/// Create and return a [`VascularPath`] which represents the path of
/// nutrients from the given [`Root`][r] to the given [`Leaf`](leaves::Leaf). 
///
/// [r]: roots::Root
pub fn trace_path(leaf: &leaves::Leaf, root: &roots::Root) -> VascularPath {
	... 
}
```

You can also add search aliases to make it easier to find things using the built-in search feature.

```rust
#[doc(alias = "route")] 
pub struct VascularPath {
	... 
}
```

文档中的代码：

```rust
/// A block of code in a doc comment:
///
////	if samples::everything().works() {
////	    println!("ok");
////	}
```

或者用Markdown格式：

```rust
  /// Another snippet, the same code, but written differently:
  ///
  /// ```
  /// if samples::everything().works() {
  ///     println!("ok");
  /// }
  /// ```
```

#### Doc-Tests

* Rust 也会测试在文档注释中的代码，会隐式的加在 `fn main()` 函数里面。
* The idea behind doc-tests is not to put all your tests into comments. Rather, you write the best possible documentation, and Rust makes sure the code samples in your documentation actually compile and run.
*  To hide a line of a code sample, put a # followed by a space at the beginning of that line.
*  rustdoc therefore treats any code block containing the exact string fn main as a complete program and doesn’t add anything to it.
* To tell Rust to compile your example, but stop short of actually running it, use a fenced code block with the `no_run` annotation.
* If the code isn’t even expected to compile, use `ignore` instead of `no_run`. Blocks marked with `ignore` don’t show up in the output of cargo run, but no_run tests show up as having passed if they compile.
* If the code block isn’t Rust code at all, use the name of the language, like c++ or sh, or text for plain text. 

### **Specifying Dependencies**

用版本号：

```toml
image = "0.6.1"
```

Git仓库地址和修订版本号：

```toml
image = { git = "https://github.com/Piston/image.git", rev = "528f19c" }
```

可以指定使用哪个 rev、tag 或 branch。

另一种方式是指定包含依赖包源代码的目录：

```toml
image = { path = "vendor/image" }
```

#### **Versions**

对于在 Cargo.toml 中写的 `image = "0.6.1"` ，Cargo 的解释并没有那么严格。它会使用与 0.6.1 版兼容的最新版本的 image 。

兼容性的判断基本上遵循“语义化版本”的思想。

* 以0.0开头的版本过于原始，Cargo不会假设它与任何其他版本兼容。
* 以0.x（基本x不是0）开头的版本，会被认为同其他以0.x开头的版本兼容。
* 如果项目达到1.0版，则只有新的主版本号才会破坏兼容性。

不同项目对依赖和版本有不同的需求。因此，指定版本时可以使用操作符。例如`>`、`>=`、`<=`。

使用通配符`*`表示任何版本都可以，这个并不多见。

#### **Cargo.lock**

在第一次构建项目时，Cargo会输出一个Cargo.lock文件，记录它使用的每个包的确切的版本号。如果手动修改了Cargo.toml文件中的版本号或者运行`cargo update`时，会把新版本号保存到Cargo.lock文件中。

对于保存在Git代码库的依赖也是类似的。

如果你的项目是一个可执行文件，应该把Cargo.lock提交到版本控制系统。如果是一个普通的Rust库，就不用提交Cargo.lock了。如果恰好你的项目是一个共享库，没有这种下游的cargo用户，那就应该提供Cargo.lock。

### **Publishing Crates to crates.io**

`cargo package` 让Cargo打包。会创建一个`.crate`文件，其中包含库的源文件以及Cargo.toml。

`cargo package --list`可以查看其中包含什么文件。

```toml
[package]
name = "fern_sim"
version = "0.1.0"
edition = "2018"
authors = ["You <you@example.com>"]
license = "MIT"
homepage = "https://fernsim.example.com/"
repository = "https://gitlair.com/sporeador/fern_sim"
documentation = "http://fernsim.example.com/docs"
description = """
Fern simulation, from the cellular level up.
"""
```

可以同时指定本地依赖和版本号，不过要保证二者同步：

```toml
image = { path = "vendor/image", version = "0.13.0" }
```

接着就是发布了：

```bash
$ cargo login 5j0dV54BjlXBpUUbfIj7G9DvNl1vsWW1
$ cargo publish
Updating registry `https://github.com/rust-lang/crates.io-index`
     Uploading fern_sim v0.1.0 (file:///.../fern_sim)
```

### Workspace

使用Cargo工作空间可以节省编译时间和磁盘空间。所谓工作空间，就是共享相同构建目录和Cargo.lock文件的一组包。

要使用工作空间，只需在存储库的根目录下创建一个Cargo.toml文件，并把下面几行放进去：

```toml
[workspace]
members = ["fern_sim", "fern_img", "fern_video"]
```

Here fern_sim etc. are the names of the subdirectories containing your crates. Delete any leftover *Cargo.lock* files and *target* directories that exist in those subdirectories.

The command `cargo build --workspace` builds all crates in the current workspace. `cargo test` and `cargo doc` accept the `--workspace` option as well.

## **Chapter 9. Structs**

Rust has three kinds of struct types, ***named-field***, ***tuple-like***, and ***unit-like***, which differ in how you refer to their components: a named-field struct gives a name to each component, whereas a tuple- like struct identifies them by the order in which they appear. Unit-like structs have no components at all; these are not common, but more useful than you might think.

### **Named-Field Structs**

```rust
/// A rectangle of eight-bit grayscale pixels.
struct GrayscaleMap { 
  pixels: Vec<u8>, 
  size: (usize, usize)
}
```

结构体的名字用驼峰命名法，其中的字段用蛇形拼写法。

You can use `key: value` syntax for some fields and shorthand for others in the same struct expression.

To access a struct’s fields, use the familiar `.` operator.

可以将结构体设为pub，但是字段默认私有。这样可以通过一些公有的方法创建结构体或者修改它。

使用`.. EXPR`，任何没有出现的字段都将从`EXPR`中取得自己的值，前提是`EXPR`必须为同一结构体类型的另一个值。

### **Tuple-Like Structs**

类元组结构体：

```rust
struct Bounds(usize, usize);
```

通过`.`操作符来访问。

定义此结构体会隐式定义一个函数：

```rust
fn Bounds(elem0: usize, elem1: usize) -> Bounds { ... }
```

*newtypes*，即只包含一个要经过更严格类型检查的组件的结构体。

```rust
struct Ascii(Vec<u8>);
```

### **Unit-Like Structs**

```rust
struct Onesuch;
```

这种类型的值不占任何内存，像`()`。

> Rust doesn’t bother actually storing unit-like struct values in memory or generating code to operate on them, because it can tell everything it might need to know about the value from its type alone. But logically, an empty struct is a type with values like any other—or more precisely, **a type of which there is only a single value**.

### **Struct Layout**

```rust
struct GrayscaleMap { 
  pixels: Vec<u8>, 
  size: (usize, usize)
}
```

一种可能的内存布局：

![image-20220715110333582](/img/2022-07-05-Programming-Rust/image-20220715110333582.png)

Rust不保证结构体的字段或元素在内存中会以某种顺序存储，但是保证把字段的值直接存储在结构体的内存块中。

### **Defining Methods with impl**

Functions defined in an impl block are called *associated functions*, since they’re associated with a specific type. The opposite of an associated function is a *free function*, one that is not defined as an impl block’s item.

* `self`
* `&self`
* `&mut self`

当调用方法时，`.` 操作符会隐式转换，如果方法定义的是借用，就会隐式转换为引用。注意`self`会移动所有权。

#### **Passing Self as a Box, Rc, or Arc**

A method’s self argument can also be a `Box<Self>`, `Rc<Self>`, or `Arc<Self>`. Such a method can only be called on a value of the given pointer type. Calling the method passes ownership of the pointer to it.

```rust
let mut bq = Box::new(Queue::new());

// `Queue::push` expects a `&mut Queue`, but `bq` is a `Box<Queue>`. 
// This is fine: Rust borrows a `&mut Queue` from the `Box` for the 
// duration of the call.
bq.push('■');
```

Rust automatically borrows a reference from pointer types like Box, Rc, and Arc, so &self and &mut self are almost always the right thing in a method signature, along with the occasional self.

```rust
impl Node {
	fn append_to(self: Rc<Self>, parent: &mut Node) {
    parent.children.push(self);
  }
}
```

If the caller has an `Rc<Node>` at hand, it can call append_to directly, passing the Rc by value:

```rust
let shared_node = Rc::new(Node::new("first")); 
shared_node.append_to(&mut parent);
```

Again, for most methods, &self, &mut self, and self (by value) are all you need. But if a method’s purpose is to affect the ownership of the value, using other pointer types for self can be just the right thing.

#### **Type-Associated Functions**

```rust
impl Queue {
	pub fn new() -> Queue {
    Queue { older: Vec::new(), younger: Vec::new() }
  }
}
```

Separating a type’s methods from its definition may seem unusual, but there are several advantages to doing so:

* It’s always easy to find a type’s data members. 
* Pulling methods out into an impl block allows a single syntax for all three. In fact, Rust uses this same syntax for defining methods on types that are not structs at all, such as enum types and primitive types like i32. 
* The same impl syntax also serves neatly for implementing traits, which we’ll go into in Chapter 11.

### **Associated Consts**

```rust
pub struct Vector2 { 
  x: f32,
  y: f32, 
}
  
impl Vector2 {
  const ZERO: Vector2 = Vector2 { x: 0.0, y: 0.0 }; 
  const UNIT: Vector2 = Vector2 { x: 1.0, y: 0.0 };
}
```

使用：

```rust
let scaled = Vector2::UNIT.scaled_by(2.0); // scaled_by() 方法需要自己定义，这里没有列出
```

### **Generic Structs**

```RUST
pub struct Queue<T> { 
  older: Vec<T>,
  younger: Vec<T>
}
```


相关方法：

```rust
impl<T> Queue<T> {
  pub fn new() -> Queue<T> {
    Queue { older: Vec::new(), younger: Vec::new() }
  }

  pub fn push(&mut self, t: T) { 
    self.younger.push(t);
  }

  pub fn is_empty(&self) -> bool { 
    self.older.is_empty() && self.younger.is_empty()
  }
  ... 
}
```
使用`Self`：
```rust
pub fn new() -> Self {
  Queue { older: Vec::new(), younger: Vec::new() }
}
```

For associated function calls, you can supply the type parameter explicitly using the ::<> (turbofish) notation:

```rust
let mut q = Queue::<char>::new();
```

### **Structs with Lifetime Parameters**

```rust
fn find_extrema<'s>(slice: &'s [i32]) -> Extrema<'s> { 
  let mut greatest = &slice[0];
  let mut least = &slice[0];

  for i in 1..slice.len() {
    if slice[i] < *least    { least     = &slice[i]; } 
    if slice[i] > *greatest { greatest  = &slice[i]; }
  }
  Extrema { greatest, least }
}
```

当然这里生命周期参数也可以省略，因为可以根据三条规则进行推断。 

### **Deriving Common Traits for Struct Types**

```rust
#[derive(Copy, Clone, Debug, PartialEq)]
struct Point { 
  x: f64,
  y: f64 
}
```

### **Interior Mutability**
支持内部可变性，用`Cell<T>`和`RefCell<T>`，定义在`std::cell`模块中。

`Cell<T>`是只包含一个`T`类型私有值的结构体。Cell唯一特别的地方是不需要对其自身的mut引用，你也能取得或设置其私有字段的值。

* `Cell::new(value)`：创建一个新Cell，将value转移到其中。
* `cell.get()`：返回cell中值的副本。要求 cell 实现 Copy trait。
* `cell.set(value)`：把value保存到cell，丢弃之间保存的值。

`RefCell<T>`是只包含一个T类型值的泛型类型。但与Cell不同，`RefCell<T>`支持借用它的T类型值的引用。
* `RefCell::new(value)`：创建一个新RefCell，将value转移到其中。
* `ref_cell.borrow()`：返回一个`Ref<T>`，基本上是对 ref_cell 中值的共享引用。This method panics if the value is already mutably borrowed.
* `ref_cell.borrow_mut()`: Returns a `RefMut<T>`, essentially a mutable reference to the value in ref_cell. This method panics if the value is already borrowed.
* `ref_cell.try_borrow()`, `ref_cell.try_borrow_mut()` : Work just like `borrow()` and `borrow_mut()`, but return a `Result`. Instead of panicking if the value is already mutably borrowed, they return an `Err` value.

This is a lot like how normal references work. The only difference is that normally, when you borrow a reference to a variable, Rust checks at compile time to ensure that you’re using the reference safely. If the checks fail, you get a compiler error. RefCell enforces the same rule using run-time checks. So if you’re breaking the rules, you get a panic (or an Err, for try_borrow and try_borrow_mut).

The other drawback is less obvious and more serious: cells—and any types that contain them—are not thread-safe. 

**补充：参考[`RefCell` 和内部可变性模式](https://rustwiki.org/zh-CN/book/ch15-05-interior-mutability.html#refcellt-和内部可变性模式)**。

- 因为 `RefCell<T>` 允许在运行时执行可变借用检查，所以我们可以在即便 `RefCell<T>` 自身是不可变的情况下修改其内部的值。

结构体是“与”的逻辑，而枚举是“或”的逻辑。

## Chapter 10. Enums and Patterns

### **Enums**
```rust
enum Ordering { 
  Less,
  Equal,
  Greater, 
}
```
这个是标准库中的，导入用use。也可以自己定义枚举，自己导入。
```rust
enum Pet { 
  Orca,
  Giraffe,
  ... 
}

use self::Pet::*;
```

In memory, values of C-style enums are stored as integers. Occasionally it’s useful to tell Rust
which integers to use:
```rust
enum HttpStatus { 
  Ok = 200,
  NotModified = 304,
  NotFound = 404,
  ...
}
```
Otherwise Rust will assign the numbers for you, starting at 0.

By default, Rust stores C-style enums using the smallest built-in integer type that can accommodate them. Most fit in a single byte.

`#[repr]`可以自己选择枚举的内存表示。

可以将C式枚举转换为整数，但是反过来不可以。

和结构体一样，可以给枚举加上属性，定义相关方法。

#### Enums with Data
一个枚举可以同时包含三种变体：
```rust
enum RelationshipStatus { 
  Single,
  InARelationship,
  ItsComplicated(Option<String>),
  ItsExtremelyComplicated {
    car: DifferentialEquation,
    cdr: EarlyModernistPoem,
  },
}
```
All constructors and fields of an enum share the same visibility as the enum itself.

#### Enums in Memory

在内存中，带数据的每个构造式都需要一个小整数标签。

`RoughTime`的每个构造式占用8个字节。

![image-20220715164735845](/img/2022-07-05-Programming-Rust/image-20220715164735845.png)

为了方便优化，Rust并未对枚举的内存布局方式做出任何承诺。For instance, some generic structs can be stored without a tag at all, as we’ll see later.

#### **Rich Data Structures Using Enums**
```rust
use std::collections::HashMap;

enum Json { 
  Null,
  Boolean(bool),
  Number(f64),
  String(String),
  Array(Vec<Json>), 
  Object(Box<HashMap<String, Json>>),
}
```

内存布局：

![image-20220716084624016](/img/2022-07-05-Programming-Rust/image-20220716084624016.png)

在内存中，Json类型的值占4个机器字。String和Vec值占3个机器字，Rust还会加一个标签字节。Null和Boolean值用不了那么多内存空间，但所有Json值的大小必须相同。

`Box<HashMap>`只占一个机器字，因为它只是一个指向分配到堆内存数据的指针。

#### **Generic Enums**
```rust
enum Option<T> { 
  None,
  Some(T), 
}

enum Result<T, E> { 
  Ok(T),
  Err(E), 
} 
```

**如果类型`T`是引用或`Box`或其他智能指针类型，Rust就会省掉`Option<T>`的标签字段。**因此`Option<Box<i32>>`在内存中只用1个机器字节存储。0表示`None`，非零表示`Some`封装的值。

```rust
// An ordered collection of `T`s.
enum BinaryTree<T> { 
  Empty,
  NonEmpty(Box<TreeNode<T>>),
}

// A part of a BinaryTree.
struct TreeNode<T> { 
  element: T,
  left: BinaryTree<T>,
  right: BinaryTree<T>, 
}
```

![image-20220716091503724](/img/2022-07-05-Programming-Rust/image-20220716091503724.png)

访问枚举数据唯一的方式是一种安全的方式：使用模式。

### **Patterns**

在枚举中不能直接通过`.`操作符访问枚举的字段。可以用match表达式来进行模式匹配。

表达式产生值，模式消费值。

模式匹配会从左到右对比模式的每个组件，依次检查当前值是否与之匹配。如果不匹配，就前进到下一个模式。

| 模式类型                | 示例               | 说明                                                  |
| ----------------------- | ------------------ | ----------------------------------------------------- |
| Literal                 |                    |                                                       |
| Range                   |                    |                                                       |
| Wildcard Variable       |                    |                                                       |
| ref variable            |                    |                                                       |
| Binding with subpattern |                    |                                                       |
| Enum pattern            |                    |                                                       |
| Tuple pattern           |                    |                                                       |
| Array pattern           |                    |                                                       |
| Slice pattern           |                    |                                                       |
| Struct pattern          |                    |                                                       |
| Reference               |                    |                                                       |
| Multiple patterns       | `'a' | 'A'`        | In refutable patterns only (match, if let, while let) |
| Guard expression        | `x if x * x <= r2` | In match only (not valid in let, etc.)                |

#### **Literals, Variables, and Wildcards in Patterns**
```rust
let calendar = match settings.get_string("calendar") { 
  "gregorian" => Calendar::Gregorian,
  "chinese" => Calendar::Chinese,
  "ethiopian" => Calendar::Ethiopian,
  other => return parse_error("calendar", other), 
};
```
可以使用通配符`_`。
```rust
let caption = match photo.tagged_pet() 
{
  Pet::Tyrannosaur => "RRRAAAAAHHHHHH",
  Pet::Samoyed => "*dog thoughts*",
  _ => "I'm cute, love me", // generic caption, works for any pet
};
```
即便你非常确定其他情况不会发生，也必须至少加上一个后备的panic分支：
```rust
// There are many Shapes, but we only support "selecting" 
// either some text, or everything in a rectangular area. 
// You can't select an ellipse or trapezoid.
match document.selection() {
  Shape::TextSpan(start, end) => paint_text_selection(start, end),
  Shape::Rectangle(rect) => paint_rect_selection(rect),
  _ => panic!("unexpected selection type"),
}
```

#### Tuple and Struct Patterns
元组模式匹配元组：
```rust
fn describe_point(x: i32, y: i32) -> &'static str { 
  use std::cmp::Ordering::*;
  match (x.cmp(&0), y.cmp(&0)) {
    (Equal, Equal) => "at the origin",
    (_, Equal) => "on the x axis",
    (Equal, _) => "on the y axis",
    (Greater, Greater) => "in the first quadrant",
    (Less, Greater) => "in the second quadrant",
    _ => "somewhere else",
  } 
}
```

结构体模式使用花括号：
```rust
match balloon.location {
  Point { x: 0, y: height } =>
    println!("straight up {} meters", height), 
  Point { x: x, y: y } =>
    println!("at ({}m, {}m)", x, y),
}
```

在结构体模式匹配中，可以使用`...`来告诉Rust你并不关心其他字段：
```rust
Some(Account { name, language, .. }) =>
  language.show_custom_greeting(name),
```

#### Array and Slice Patterns
数组匹配：
```rust
fn hsl_to_rgb(hsl: [u8; 3]) -> [u8; 3] { 
  match hsl {
    [_, _, 0] => [0, 0, 0],
    [_, _, 255] => [255, 255, 255],
    ...
  } 
}
```

切片匹配：
```rust
fn greet_people(names: &[&str]) { 
  match names {
    [] => { println!("Hello, nobody.") },
    [a] => { println!("Hello, {}.", a) },
    [a, b] => { println!("Hello, {} and {}.", a, b) },
    [a, .., b] => { println!("Hello, everyone from {} to {}.", a, b) }
  } 
}
```

#### Reference Patterns
对于引用，Rust支持两种模式：`ref`模式和`&`模式。**前者借用匹配值的元素，后者匹配引用**。

需要一种模式来借用而不`move`匹配的值。用关键字`ref`。
```rust
match account {
  Account { ref name, ref language, .. } => {
    ui.greet(name, language);
    ui.show_settings(&account); // ok 
  }
}
```

**使用`ref mut`借用`mut`引用**：

```rust
match line_result {
  Err(ref err) => log_error(err),   // `err` is &Error (shared ref) 
  Ok(ref mut line) => {             // `line` is &mut String (mut ref)
    trim_comments(line);            // modify the String in place
    handle(line);
  }
}
```

以`&`开头的模式匹配引用。
```rust
match sphere.center() { 
  &Point3d { x, y, z } => ... // 这里 x, y, z 是值，实现了 Copy trait
}
```
表达式和模式天生是相反的。表达式`(x, y)`用两个值创建一个新元组，模式`(x, y)`则相反：它匹配元组并破坏后取出两个值。对`&`而言也一样：表达式`&`创建引用，模式中的`&`匹配引用。匹配引用中，生命周期是必要条件。不能对共享引用采取`mut`操作。不能从引用（包括`mut`引用）中move出值。

```rust
match friend.borrow_car() {
  Some(&Car { engine, .. }) => // error: can't move out of borrow 这里 engine 没有实现 Copy trait
    ... 
  None => {}
}
```

修改：
```rust
Some(&Car { ref engine, .. }) => // ok, engine is a reference
```

可以使用`&`模式取得引用所指向的字符：
```rust
match chars.peek() { // chars.peek() 返回 Option<&char>
  Some(&c) => println!("coming up: {:?}", c), 
  None => println!("end of chars"),
}
```

#### Match Guards
```rust
match point_to_hex(click) {
  None => Err("That's not a game space."), 
  Some(hex) => {
    if hex == current_hex {
      Err("You are already there! You must click somewhere else")
    } else { 
      Ok(hex)
    } 
  }
}
```
But Rust also provides match guards, extra conditions that must be true in order for a match arm to apply, written as `if CONDITION`, between the pattern and the arm’s `=>` token:

```rust
match point_to_hex(click) {
  None => Err("That's not a game space."), 
  Some(hex) if hex == current_hex =>
    Err("You are already there! You must click somewhere else"),
  Some(hex) => Ok(hex)
}
```
If the pattern matches, but the condition is false, matching continues with the next arm.

#### Matching Multiple Possibilities
```rust
let at_end = match chars.peek() { 
  Some(&'\r') | Some(&'\n') | None => true, 
  _ => false,
};
```
或者
```rust
match next_char {
  '0'..='9' => self.read_number(),
  'a'..='z' | 'A'..='Z' => self.read_word(),
  ' ' | '\t' | '\n' => self.skip_whitespace(), 
  _ => self.handle_punctuation(),
}
```

#### Binding with @ Patterns
**`x @ pattern`匹配给定的 pattern，但成功之后，不是基于匹配值的元素来创建变量，而是把匹配值整个移动或复制到一个变量 `x` 中**。

```rust
rect @ Shape::Rect(..) => {
  optimized_paint(&rect)
}
```
`@ ` patterns are also useful with ranges:

```rust
match chars.next() {
  Some(digit @ '0'..='9') => read_number(digit, chars), 
  ...
},
```

#### Where Patterns Are Allowed
The meaning is always the same: instead of just storing a value in a single variable, Rust uses pattern matching to take the value apart.

```rust
// ...unpack a struct into three new local variables
let Track { album, track_number, title, .. } = song; 

// ...unpack a function argument that's a tuple
fn distance_to((x, y): (f64, f64)) -> f64 { ... }

// ...iterate over keys and values of a HashMap
for (id, document) in &cache_map {
  println!("Document #{}: {}", id, document.title);
}

// ...automatically dereference an argument to a closure
// (handy because sometimes other code passes you a reference 
// when you'd rather have a copy)
let sum = numbers.fold(0, |a, &num| a + num);
```
上面的都是 *irrefutable patterns*，下面的是*refutable pattern*。
```rust
 // ...handle just one enum variant specially
if let RoughTime::InTheFuture(_, _) = user.date_of_birth() { 
  user.set_time_traveler(true);
}

// ...run some code only if a table lookup succeeds
if let Some(document) = cache_map.get(&id) { 
  return send_cached_response(document);
}

// ...repeatedly try something until it succeeds
while let Err(err) = present_cheesy_anti_robot_task() { 
  log_robot_attempt(err);
  // let the user try again (it might still be a human)
}

// ...manually loop over an iterator
while let Some(_) = lines.peek() { 
  read_paragraph(&mut lines);
}
```

#### Populating a Binary Tree
```rust
enum BinaryTree<T> {
    Empty,
    NonEmpty(Box<TreeNode<T>>),
}

struct TreeNode<T> {
    element: T,
    left: BinaryTree<T>,
    right: BinaryTree<T>,
}

impl<T: Ord>  BinaryTree<T> {
    fn add(&mut self, value: T) {
        match *self {
            BinaryTree::Empty =>
                *self = BinaryTree::NonEmpty(Box::new(TreeNode{
                    element: value,
                    left: BinaryTree::Empty,
                    right: BinaryTree::Empty,
                })),
            BinaryTree::NonEmpty(ref mut node) =>
                if value <= node.element {
                    node.left.add(value);
                } else {
                    node.right.add(value);
                }
        }
    }
}

fn main() {
    let mut tree = BinaryTree::Empty;
    tree.add("Mercury");
    tree.add("Venus");
}
```
## **Chapter 11. Traits and Generics**

Rust supports polymorphism with two related features: **traits** and **generics**.

Generics and traits are closely related: generic functions use traits in bounds to spell out what types of arguments they can be applied to. So we’ll also talk about **how `&mut dyn Write` and `<T: Write>` are similar, how they’re different**, and how to choose between these two ways of using traits.

### **Using Traits**

* A value that implements `std::io::Write` can write out bytes.
* A value that implements `std::iter::Iterator` can produce a sequence of values.
* A value that implements `std::clone::Clone` can make clones of itself in memory.
* A value that implements `std::fmt::Debug` can be printed using println!() with the {:?} format specifier.

trait本身必须在作用域中。否则，trait的所有方法都是隐藏的。

But since Rust makes you import the traits you plan to use, crates are free to take advantage of this superpower. To get a conflict, you’d have to import two traits that add a method with the same name to the same type. This is rare in practice. (If you do run into a conflict, you can spell out what you want using fully qualified method syntax, covered later in the chapter.)

The reason **Clone and Iterator** methods work without any special imports is that they’re always in scope by default: they’re part of **the standard prelude**, names that Rust automatically imports into every module.

#### **Trait Objects**

Rust doesn’t permit variables of type dyn Write:

```rust
use std::io::Write;

let mut buf: Vec<u8> = vec![];
let writer: dyn Write = buf; // error: `Write` does not have a constant size
```

A variable’s size has to be known at compile time, and types that implement Write can be any size.

```rust
let mut buf: Vec<u8> = vec![];
let writer: &mut dyn Write = &mut buf; // ok
```

A reference to a trait type, like writer, is called a ***trait object***. Like any other reference, a trait object points to some value, it has a lifetime, and it can be either mut or shared.

#### **Trait object layout**

In memory, **a trait object is a fat pointer** consisting of a pointer to the value, plus a pointer to a table representing that value’s type. **Each trait object therefore takes up two machine words.**

![image-20220716142028472](/img/2022-07-05-Programming-Rust/image-20220716142028472.png)

 In Rust, as in C++, **the vtable is generated once, at compile time**, and shared by all objects of the same type. Everything shown in the darker shade in Figure 11-1, **including the vtable, is a private implementation** detail of Rust. Again, these aren’t fields and data structures that you can access directly. Instead, the language automatically uses the vtable when you call a method of a trait object, to determine which implementation to call.

Rust automatically converts ordinary references into trait objects when needed. 

```rust
let mut local_file = File::create("hello.txt")?; 
say_hello(&mut local_file)?;
```

Likewise, Rust will happily convert a `Box<File>` to a `Box<dyn Write>`, a value that owns a writer in the heap:

```rust
let w: Box<dyn Write> = Box::new(local_file);
```

This kind of conversion is the only way to create a trait object. What the compiler is actually doing here is very simple. At the point where the conversion happens, Rust knows the referent’s true type (in this case, File), so it just adds the address of the appropriate vtable, turning the regular pointer into a fat pointer.

#### **Generic Functions and Type Parameters**
```rust
fn say_hello(out: &mut dyn Write)   // plain function
fn say_hello<W: Write>(out: &mut W) // generic function
```
```rust
fn say_hello<W: Write>(out: &mut W) // generic function

say_hello(&mut local_file)?; // calls say_hello::<File>
say_hello(&mut bytes)?; // calls say_hello::<Vec<u8>>
```

This process is known as ***monomorphization***, and the compiler handles it all automatically.

```rust
// calling a generic method collect<C>() that takes no arguments 
let v1 = (0 .. 1000).collect();             // error: can't infer type
let v2 = (0 .. 1000).collect::<Vec<i32>>(); // ok
```

```rust
fn run_query<M, R>(data: &DataSet, map: M, reduce: R) -> Results 
  where M: Mapper + Serialize,
        R: Reducer + Serialize
{ ... }
```
生命周期参数：
```rust
/// Return a reference to the point in `candidates` that's
/// closest to the `target` point.
fn nearest<'t, 'c, P>(target: &'t P, candidates: &'c [P]) -> &'c P
  where P: MeasureDistance 
{
  ... 
}
```
生命周期不会对机器码有任何影响。

All the features introduced in this section—bounds, where clauses, lifetime parameters, and so forth—**can be used on all generic items**, not just functions.

#### Which to Use
```rust
trait Vegetable { 
  ...
}

struct Salad<V: Vegetable> { 
  veggies: Vec<V>
}
```
沙拉结构体用泛型函数实现，但是只能放一种蔬菜。

考虑到蔬菜值的大小可能差别比较大，不能直接用 Vec。
```rust
struct Salad {
  veggies: Vec<dyn Vegetable> // error: `dyn Vegetable` does
                              // not have a constant size
}
```
可以这样实现：

```rust
struct Salad {
  veggies: Vec<Box<dyn Vegetable>>
}
```

Another possible reason to use trait objects is to reduce the total amount of compiled code. 

那么在不考虑沙拉或者low-resource environments这两种情况下，选择泛型函数有三个优点：
* The first advantage is **speed**. 会通过静态分发的方式，消除动态查找的时间。
* The second advantage of generics is that **not every trait can support trait objects**.
* The third advantage of generics is that it’s easy to **bound a generic type parameter with several traits at once**, as our top_ten function did when it required its T parameter to implement Debug + Hash + Eq. Trait objects can’t do this: types like &mut (dyn Debug + Hash + Eq) aren’t supported in Rust. (You can work around this with subtraits, defined later in this chapter, but it’s a bit involved.)

### Defining and Implementing Traits
定义trait：
```rust
/// A trait for characters, items, and scenery -
/// anything in the game world that's visible on screen. 
trait Visible {
  /// Render this object on the given canvas.
  fn draw(&self, canvas: &mut Canvas);
  /// Return true if clicking at (x, y) should /// select this object.
  fn hit_test(&self, x: i32, y: i32) -> bool;
}
```
实现trait，用`impl TraitName for Type`。

Everything defined in a trait `impl` must actually be a feature of the trait.

不在trait里的辅助方法要单独放在`impl Type`里面实现。而且实现trait时也可以调用这个辅助方法。

#### Default Methods

trait的默认实现：

```rust
trait Write {
  fn write(&mut self, buf: &[u8]) -> Result<usize>; 
  fn flush(&mut self) -> Result<()>;

  fn write_all(&mut self, buf: &[u8]) -> Result<()> { 
    let mut bytes_written = 0;
    while bytes_written < buf.len() {
      bytes_written += self.write(&buf[bytes_written..])?;
    }
    Ok(()) 
  }
  ... 
}
```

#### **Traits and Other People’s Types**

Rust lets you implement any trait on any type, as long as **either the trait or the type is introduced in the current crate**.

The sole purpose of this particular trait is to add a method to an existing type, char. This is called an ***extension trait***.

```rust
trait IsEmoji {
  fn is_emoji(&self) -> bool;
}

/// Implement IsEmoji for the built-in character type.
impl IsEmoji for char {
  fn is_emoji(&self) -> bool {
    ... 
  }
}
assert_eq!('$'.is_emoji(), false);
```

You can even use a generic impl block to add an extension trait to a whole family of types at once.
```rust
/// You can write HTML to any std::io writer.
impl<W: Write> WriteHtml for W {
  fn write_html(&mut self, html: &HtmlDocument) -> io::Result<()> {
    ... 
  }
}
```
We said earlier that when you implement a trait, either the trait or the type must be new in the current crate. This is called the **orphan rule**. It helps Rust ensure that trait implementations are unique. 

#### Self in Traits
```rust
pub trait Clone {
  fn clone(&self) -> Self;
  ...
}
```
Self作为返回类型意味着`x.clone()`的类型就是`x`的类型，不管具体什么类型。

**使用Self类型的trait与trait objects不能共存。**

```rust
pub trait Spliceable {
  fn splice(&self, other: &Self) -> Self;
}

// error: the trait `Spliceable` cannot be made into an object
fn splice_anything(left: &dyn Spliceable, right: &dyn Spliceable) { 
  let combo = left.splice(right);
  // ...
}
```
Rust rejects this code because it has no way to type-check the call `left.splice(right)`. The whole point of trait objects is that the type isn’t known until run time. Rust has no way to know at compile time if left and right will be the same type, as required.

```rust
pub trait MegaSpliceable {
  fn splice(&self, other: &dyn MegaSpliceable) -> Box<dyn MegaSpliceable>;
}
```
There’s no problem type-checking calls to this `.splice()` method because the type of the argument other is not required to match the type of self, as long as both types are MegaSpliceable.

#### Subtraits
```rust
/// Someone in the game world, either the player or some other 
/// pixie, gargoyle, squirrel, ogre, etc.
trait Creature: Visible {
  fn position(&self) -> (i32, i32); 
  fn facing(&self) -> Direction; 
  ...
}
```
Here, we say that **Creature is a subtrait of Visible**, and that Creature is Visible’s supertrait.

But in Rust, **a subtrait does not inherit the associated items of its supertrait**; each trait still needs to be in scope if you want to call its methods.

In fact, **Rust’s subtraits are really just a shorthand for a bound on Self**. A definition of Creature like this is exactly equivalent to the one shown earlier:
```rust
trait Creature where Self: Visible { 
  ...
}
```

#### Type-Associated Functions
Traits can include type-associated functions, Rust’s analog to static methods:
```rust
trait StringSet {
  /// Return a new empty set. 
  fn new() -> Self; // 没有 self 参数

  /// Return a set that contains all the strings in `strings`.
  fn from_slice(strings: &[&str]) -> Self; // 没有 self 参数

  /// Find out if this set contains a particular `value`.
  fn contains(&self, string: &str) -> bool;

  /// Add a string to this set.
  fn add(&mut self, string: &str); 
}
```
If you want to use `&dyn StringSet` trait objects, you must change the trait, adding the bound where `Self: Sized` to each associated function that doesn’t take a self argument by reference:
```rust
trait StringSet { 
  fn new() -> Self
    where Self: Sized;
  
  fn from_slice(strings: &[&str]) -> Self
    where Self: Sized;
  
  fn contains(&self, string: &str) -> bool;
  
  fn add(&mut self, string: &str); 
}
```
**With these additions, StringSet trait objects are allowed; they still don’t support `new` or `from_slice`, but you can create them and use them to call `.contains()` and `.add()`.**

### Fully Qualified Method Calls
```rust
"hello".to_string()

str::to_string("hello")

ToString::to_string("hello")

<str as ToString>::to_string("hello") // a fully qualified method call
```
With fully qualified calls, you can say exactly which method you mean, and that can help in a few odd cases:
* When two methods have the same name.
  ```rust
  outlaw.draw(); // error: draw on screen or draw pistol? 
  
  Visible::draw(&outlaw);   // ok: draw on screen
  HasPistol::draw(&outlaw); // ok: corral
  ```
* When the type of the self argument can’t be inferred:
  ```rust
  let zero = 0; // type unspecified; could be `i8`, `u8`, ...
  
  zero.abs();     // error: can't call method `abs` 
                  // on ambiguous numeric type
  
  i64::abs(zero); // ok
  ```
* When using the function itself as a function value:
  ```rust
  let words: Vec<String> =
    line.split_whitespace()     // iterator produces &str values
      .map(ToString::to_string) // ok 
      .collect();
  ```
* When calling trait methods in macros.

### Traits That Define Relationships Between Types

Traits can also be used in situations where there are multiple types that have to work together. They can describe relationships between types.

#### Associated Types (or How Iterators Work)

```rust
pub trait Iterator {
	type Item;
	fn next(&mut self) -> Option<Self::Item>;
	... 
}
```
The first feature of this trait, `type Item;`, is an associated type.

```rust
use std::fmt::Debug;
fn dump<I>(iter: I)
  where I: Iterator, I::Item: Debug
{
  ...
}
```

trait object:
```rust
fn dump(iter: &mut dyn Iterator<Item=String>) { 
  for (index, s) in iter.enumerate() {
  	println!("{}: {:?}", index, s);
  }
}
```

#### Generic Traits (or How Operator Overloading Works)
```rust
/// std::ops::Mul, the trait for types that support `*`.
pub trait Mul<RHS> {
  /// The resulting type after applying the `*` operator 
  type Output;
  
  /// The method for the `*` operator
  fn mul(self, rhs: RHS) -> Self::Output; 
}
```

The type parameter here means the same thing that it means on a struct or function: **Mul is a generic trait, and its instances `Mul<f64>`, `Mul<String>`, `Mul<Size>`, etc., are all different traits**.

Generic traits get a special dispensation when it comes to the orphan rule: **you can implement a foreign trait for a foreign type**, so long as one of the trait’s type parameters is a type defined in the current crate. 

```rust
pub trait Mul<RHS=Self> { 
  ...
}
```
The syntax `RHS=Self` means that RHS defaults to Self. If I write `impl Mul for Complex`, without specifying Mul’s type parameter, it means impl `Mul<Complex> for Complex`. In a bound, if I write `where T: Mul`, it means where `T: Mul<T>`.

#### impl Trait
```rust
use std::iter;
use std::vec::IntoIter;
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) ->
  iter::Cycle<iter::Chain<IntoIter<u8>, IntoIter<u8>>> { 			 					             
  v.into_iter().chain(u.into_iter()).cycle()
}
```
这种写法太繁琐，可以这样写：
```rust
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> Box<dyn Iterator<Item=u8>> { 
    Box::new(v.into_iter().chain(u.into_iter()).cycle())
}
```
但是这是动态分发，影响效率。可以使用 `impl Trait`。
```rust
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> impl Iterator<Item=u8> { 
  v.into_iter().chain(u.into_iter()).cycle()
}
```

```rust
fn make_shape(shape: &str) -> impl Shape { 
  match shape {
    "circle" => Circle::new(),
    "triangle" => Triangle::new(), // error: incompatible types 
    "shape" => Rectangle::new(),
  } 
}
```
It’s important to note that **Rust doesn’t allow trait methods to use `impl Trait` return values**.

```rust
fn print<T: Display>(val: T) { 
  println!("{}", val);
}

fn print(val: impl Display) { 
  println!("{}", val);
}
```
There is one important exception. Using generics allows callers of the function to specify the type of the generic arguments, like `print::<i32>(42)`, while using impl Trait does not.

#### Associated Consts
```RUST
trait Greet {
  const GREETING: &'static str = "Hello"; 
  fn greet(&self) -> String;
}
```

```rust
fn fib<T: Float + Add<Output=T>>(n: usize) -> T { 
  match n {
    0 => T::ZERO,
    1 => T::ONE,
    n => fib::<T>(n - 1) + fib::<T>(n - 2)
  } 
}
```

### Reverse-Engineering Bounds
和编译器做斗争，加上 Bounds。
```rust
use std::ops::{Add, Mul};

fn dot<N>(v1: &[N], v2: &[N]) -> N
where N: Add<Output=N> + Mul<Output=N> + Default + Copy 
{
  let mut total = N::default(); 
  for i in 0 .. v1.len() {
    total = total + v1[i] * v2[i];
  }
  total
}

#[test]
fn test_dot() {
  assert_eq!(dot(&[1, 2, 3, 4], &[1, 1, 1, 1]), 10); 
  assert_eq!(dot(&[53.0, 7.0], &[1.0, 5.0]), 88.0);
}
```

或者用第三方库`num`。
```rust
use num::Num;

fn dot<N: Num + Copy>(v1: &[N], v2: &[N]) -> N { 
  let mut total = N::zero();
  for i in 0 .. v1.len() {
    total = total + v1[i] * v2[i];
  }
  total
}
```

Rust为什么要这样设计呢？为什么不模仿“鸭子类型”？这样设计的优点是：
* 可以让泛型代码具有向前兼容的能力。
* 你能通过编译器保存就知道要解决的麻烦在哪里。
* 或许，明确写出绑定最重要的好处是它们在代码和文档里都存在。

### Traits as a Foundation
Traits are one of the main organizing features in Rust, and with good reason. There’s nothing better to design a program or library around than a good interface.

## **Chapter 12. Operator Overloading**

The traits for operator overloading fall into a few categories depending on what part of the language they support.

| 类别                 | Trait                  | 操作符                               |
| -------------------- | ---------------------- | ------------------------------------ |
| Unary operators      | std::ops::Neg          | `-x`                                 |
|                      | std::ops::Not          | `!x`                                 |
| Arithmetic operators | std::ops::Add          | `x + y`                              |
|                      | std::ops::Sub          | `x - y`                              |
|                      | std::ops::Mul          | `x * y`                              |
|                      | std::ops::Div          | `x / y`                              |
|                      | std::ops::Rem          | `x % y`                              |
| Bitwise operators    | std::ops::BitAnd       | `x & y`                              |
|                      | std::ops::BitOr        | `x | y`                              |
|                      | std::ops::BitXor       | `x ^ y`                              |
|                      | std::ops::Shl          | `x << y`                             |
|                      | std::ops::Shr          | `x >> y`                             |
| Compound assignment  | std::ops::AddAssign    | `x += y`                             |
| arithmetic operators | std::ops::SubAssign    | `x -= y`                             |
|                      | std::ops::MulAssign    | `x *= y`                             |
|                      | std::ops::DivAssign    | `x /= y`                             |
|                      | std::ops::RemAssign    | `x %= y`                             |
| Compound assignment  | std::ops::BitAddAssign | `x &= y`                             |
| bitwise operators    | std::ops::BitOrAssign  | `x |= y`                             |
|                      | std::ops::BitXorAssign | `x ^= y`                             |
|                      | std::ops::ShlAssign    | `x <<= y`                            |
|                      | std::ops::ShrAssign    | `x >>= y`                            |
| Comparison           | std::cmp::PartialEq    | `x == y`, `x != y`                   |
|                      | std::cmp::PartialOrd   | `x < y`, `x <= y`, `x > y`, `x >= y` |
| Indexing             | std::ops::Index        | `x[y]`、`&x[y]`                      |
|                      | std::ops::IndexMut     | `x[y] = z`、`&mut x[y]`              |

### **Arithmetic and Bitwise Operators**

`std::ops::Add`定义：

```rust
trait Add<Rhs = Self> { 
  type Output;
  fn add(self, rhs: Rhs) -> Self::Output; 
}
```

实现:

```rust
use std::ops::Add;

impl<T> Add for Complex<T> 
  where T: Add<Output = T>, 
{
  type Output = Self;
  fn add(self, rhs: Self) -> Self {
    Complex {
      re: self.re + rhs.re, 
      im: self.im + rhs.im,
    } 
  }
}
```

混合实现，加法的左边和右边不要求是同一类型：

```rust
use std::ops::Add;

impl<L, R> Add<Complex<R>> for Complex<L> 
  where L: Add<R>, {
    type Output = Complex<L::Output>;
    fn add(self, rhs: Complex<R>) -> Self::Output {
      Complex {
        re: self.re + rhs.re, 
        im: self.im + rhs.im,
      } 
    }
}
```

#### **Unary Operators**

Note that ! complements bool values and performs a bitwise complement (that is, flips the bits) when applied to integers; it plays the role of both the ! and ~ operators from C and C++.

```rust
trait Neg { 
  type Output;
  fn neg(self) -> Self::Output; 
}

trait Not { 
  type Output;
  fn not(self) -> Self::Output; 
}
```

复数的实现：

```rust
use std::ops::Neg;
impl<T> Neg for Complex<T> 
  where T: Neg<Output = T>, 
{
  type Output = Complex<T>; 
  fn neg(self) -> Complex<T> {
    Complex {
      re: -self.re,
      im: -self.im,
    }
  } 
}
```

#### **Binary Operators**

All of Rust’s numeric types implement the arithmetic operators. Rust’s integer types and bool implement the bitwise operators. There are also implementations that accept references to those types as either or both operands.

You can use the + operator to concatenate a String with a &str slice or another String. However, Rust does not permit the left operand of + to be a &str, to discourage building up long strings by repeatedly concatenating small pieces on the left. (This performs poorly, requiring time quadratic in the final length of the string.) 

#### **Compound Assignment Operators**

A compound assignment expression is one like `x += y` or `x &= y`: it takes two operands, performs some operation on them like addition or a bitwise AND, and stores the result back in the left operand. In Rust, t**he value of a compound assignment expression is always `()`, never the value stored**.

Many languages have operators like these and usually define them as shorthand for expressions like `x = x + y` or `x = x & y`. However, Rust doesn’t take that approach. Instead, **`x += y` is shorthand for the method call `x.add_assign(y)`, where `add_assign` is the sole method of the `std::ops::AddAssign` trait**:

```rust
trait AddAssign<Rhs = Self> {
  fn add_assign(&mut self, rhs: Rhs);
}
```

The built-in trait for a compound assignment operator is **completely independent** of the built-in trait for the corresponding binary operator. 

### **Equivalence Comparisons**

```rust
pub trait PartialEq<Rhs: ?Sized = Self> {
    /// This method tests for `self` and `other` values to be equal, and is used
    /// by `==`.
    #[must_use]
    #[stable(feature = "rust1", since = "1.0.0")]
    fn eq(&self, other: &Rhs) -> bool;

    /// This method tests for `!=`.
    #[inline]
    #[must_use]
    #[stable(feature = "rust1", since = "1.0.0")]
    #[default_method_body_is_const]
    fn ne(&self, other: &Rhs) -> bool {
        !self.eq(other)
    }
}
```

用属性快速实现：

```rust
#[derive(Clone, Copy, Debug, PartialEq)]
struct Complex<T> { 
  ...
}
```

Rust’s automatically generated implementation is essentially identical to our hand-written code, comparing each field or element of the type in turn.

This means that comparing non-Copy values like Strings, Vecs, or HashMaps doesn’t cause them to be moved, which would be troublesome:

```rust
let s = "d\x6fv\x65t\x61i\x6c".to_string();
let t = "\x64o\x76e\x74a\x69l".to_string(); 
assert!(s == t); // s and t are only borrowed...

// ... so they still have their values here.
assert_eq!(format!("{} {}", s, t), "dovetail dovetail");
```

Why is this trait called PartialEq? The traditional mathematical definition of an *equivalence relation*, of which equality is one instance, imposes three requirements. For any values x and y:

* 对称性：If x == y is true, then y == x must be true as well. In other words, swapping the two sides of an equality comparison doesn’t affect the result. 

* 传递性：If x == y and y == z, then it must be the case thatx == z. Given any chain of values, each equal to the next, each value in the chain is directly equal to every other. Equality is contagious.

* 自反性：It must always be true that x == x.

**`PartialEq`只实现了前两条，自反性不保证**，比如`f32`和`f64`就不满足。`0.0/0.0`会产生非数字的值`NaN`。

```rust
assert!(f64::is_nan(0.0 / 0.0)); 
assert_eq!(0.0 / 0.0 == 0.0 / 0.0, false); 
assert_eq!(0.0 / 0.0 != 0.0 / 0.0, true);

assert_eq!(0.0 / 0.0 < 0.0 / 0.0, false); 
assert_eq!(0.0 / 0.0 > 0.0 / 0.0, false); 
assert_eq!(0.0 / 0.0 <= 0.0 / 0.0, false); 
assert_eq!(0.0 / 0.0 >= 0.0 / 0.0, false);
```

`Eq` Trait：

```rust
trait Eq: PartialEq<Self> {}
```

也可以用属性来快速实现：

```rust
#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct Complex<T> { 
  ...
}
```

### **Ordered Comparisons**

`std::cmp::PartialOrd`：

```rust
trait PartialOrd<Rhs = Self>: PartialEq<Rhs> 
where
      Rhs: ?Sized,
{
  fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;
  
  fn lt(&self, other: &Rhs) -> bool { ... } 
  fn le(&self, other: &Rhs) -> bool { ... } 
  fn gt(&self, other: &Rhs) -> bool { ... } 
  fn ge(&self, other: &Rhs) -> bool { ... }
}
```

```rust
pub enum Ordering {
    /// An ordering where a compared value is less than another.
    #[stable(feature = "rust1", since = "1.0.0")]
    Less = -1,
    /// An ordering where a compared value is equal to another.
    #[stable(feature = "rust1", since = "1.0.0")]
    Equal = 0,
    /// An ordering where a compared value is greater than another.
    #[stable(feature = "rust1", since = "1.0.0")]
    Greater = 1,
}
```

But if partial_cmp returns None, that means self and other are unordered with respect to each other: neither is greater than the other, nor are they equal.  Among all of Rust’s primitive types, only comparisons between floating-point values ever return None: specifically, **comparing a NaN (not-a-number) value with anything else returns None**.

If you know that values of two types **are always ordered with respect to each other**, then you can implement the stricter `std::cmp::Ord` trait:

```rust
pub trait Ord: Eq + PartialOrd<Self> {
  /// This method returns an [`Ordering`] between `self` and `other`.
  ///
  /// By convention, `self.cmp(&other)` returns the ordering matching the expression
  /// `self <operator> other` if true.
  ///
  /// # Examples
  ///
  /// ```
  /// use std::cmp::Ordering;
  ///
  /// assert_eq!(5.cmp(&10), Ordering::Less);
  /// assert_eq!(10.cmp(&5), Ordering::Greater);
  /// assert_eq!(5.cmp(&5), Ordering::Equal);
  /// ```
  #[must_use]
  #[stable(feature = "rust1", since = "1.0.0")]
  fn cmp(&self, other: &Self) -> Ordering;
  // ...
}
```

Almost all types that implement PartialOrd should also implement Ord. In the standard library, **f32 and f64 are the only exceptions to this rule**.

**排序应该同时实现 `Partial Ord` 和 `Ord`** 。You might want to sort by upper bound, for instance, and it’s easy to do that with `sort_by_key`:

```rust
intervals.sort_by_key(|i| i.upper);
```

**The `Reverse` wrapper type** takes advantage of this by implementing `Ord` with a method that simply inverts any ordering. 

```rust
use std::cmp::Reverse; 
intervals.sort_by_key(|i| Reverse(i.lower));
```

### **Index and IndexMut**

On any other type, the expression `a[i]` is normally shorthand for `*a.index(i)`, where index is a method of the `std::ops::Index` trait. However, if the expression is being assigned to or borrowed mutably, it’s instead shorthand for `*a.index_mut(i)`, a call to the method of the `std::ops::IndexMut` trait.

```rust
trait Index<Idx> {
  type Output: ?Sized;
  fn index(&self, index: Idx) -> &Self::Output;
}

trait IndexMut<Idx>: Index<Idx> {
  fn index_mut(&mut self, index: Idx) -> &mut Self::Output;
}
```

You can refer to a subslice with an expression like `a[i..j]` because they also implement `Index<Range<usize>>`. That expression is shorthand for:

```rust
*a.index(std::ops::Range { start: i, end: j })
```

```rust
use std::collections::HashMap; 

let mut m = HashMap::new(); 
// HashMap<&str, i32>, 实现了 Index<&str> trait
m.insert("十", 10); 
m.insert("百", 100); 
m.insert("千", 1000); 
m.insert("万", 1_0000); 
m.insert("億", 1_0000_0000);
assert_eq!(m["十"], 10); 
assert_eq!(m["千"], 1000);

// 等价于
use std::ops::Index; 
assert_eq!(*m.index("十"), 10); 
assert_eq!(*m.index("千"), 1000);
```

The Index trait’s associated type `Output` specifies what type an indexing expression produces: for our HashMap, the Index implementation’s Output type is `i32`.

Rust automatically selects `index_mut` when the indexing expression occurs in a context where it’s necessary. 

```rust
let mut desserts =
  vec!["Howalon".to_string(), "Soan papdi".to_string()];
desserts[0].push_str(" (fictional)");
desserts[1].push_str(" (real)");
```

后两行和下面的代码一样：

```rust
use std::ops::IndexMut; 
(*desserts.index_mut(0)).push_str(" (fictional)"); (*desserts.index_mut(1)).push_str(" (real)");
```

One limitation of `IndexMut` is that, by design, it **must return a mutable reference to some value**.

The most common use of indexing is **for collections**. 

This is how `Index` and `IndexMut` implementations are supposed to behave: **out-of-bounds access is detected and causes a panic**, the same as when you index an array, slice, or vector out of bounds.

### **Other Operators**

**Not all operators can be overloaded in Rust**. As of Rust 1.50, **the error-checking `?` operator** works only with Result and Option values, though work is in progress to expand this to user-defined types as well. Similarly, **the logical operators `&&` and `||`** are limited to Boolean values only. **The `..` and `..=` operators** always create a struct representing the range’s bounds, **the `&` operator** always borrows references, and **the `=` operator** always moves or copies values. None of them can be overloaded.

Rust **does not support overloading the function call operator, `f(x)`**. Instead, when you need a callable value, you’ll typically just write a closure. 

## **Chapter 13. Utility Traits**

*Language extension traits*

* Drop, Deref and DerefMut
* From and Into

*Marker traits*

* Sized
* Copy

*Public vocabulary traits*

* Default
* AsRef, AsMut, Borrow and BorrowMut
* TryFrom and TryInto
* ToOwned

| Trait                                     | Description                                                  |
| ----------------------------------------- | ------------------------------------------------------------ |
| [Drop](#drop)                                  | Destructors. Cleanup code that Rust runs automatically whenever a value is dropped. |
| [Sized](#sized)                                 | Marker trait for types with a fixed size known at compile time, as opposed to types (such as slices) that are dynamically sized. |
| [Clone](#clone)                                 | Types that support cloning values.                           |
| [Copy](#copy)                                  | Marker trait for types that can be cloned simply by making a byte-for-byte copy of the memory containing the value. |
| [Deref and DerefMut](#deref-and-derefmut) | Traits for smart pointer types.                              |
| [Default](#default)                               | Types that have a sensible “default value.”                  |
| [AsRef and AsMut](#asref-and-asmut)                       | Conversion traits for borrowing one type of reference from another. |
| [Borrow and BorrowMut](#borrow-and-borrowmut)                  | Conversion traits, like AsRef/AsMut, but additionally guaranteeing consistent hashing, ordering, and equality. |
| [From and Into](#from-and-into)                         | Conversion traits for transforming one type of value into another. |
| [TryFrom and TryInto](#tryfrom-and-tryinto)                   | Conversion traits for transforming one type of value into another, for transformations that might fail. |
| [ToOwned](#toowned)                               | Conversion trait for converting a reference to an owned value. |

### Drop

`std::ops::Drop` trait

```rust
trait Drop {
  fn drop(&mut self);
}
```

**This implicit invocation of drop is the only way to call that method**; if you try to invoke it explicitly yourself, Rust flags that as an error.

If a variable’s value gets moved elsewhere, so that the variable is uninitialized when it goes out of scope, then Rust will not try to drop that variable: there is no value in it to drop. Although a value may be moved from place to place, **Rust drops it only once**.

You usually won’t need to implement `std::ops::Drop` unless you’re defining **a type that owns resources Rust doesn’t already know about**.

**If a type implements `Drop`, it cannot implement the `Copy` trait**. If a type is Copy, that means that simple byte-for-byte duplication is sufficient to produce an independent copy of the value. But it is typically a mistake to call the same drop method more than once on the same data.

The standard prelude includes a function to drop a value, drop, but its definition is anything but magical:

```rust
fn drop<T>(_x: T) { }
```

从调用者那里获得所有权，然后什么也不做。Rust会在超出作用域时清楚`_x`的值，跟清除其他变量的值一样。

### **Sized**

A *sized type* is one whose values all have the same size in memory. 

All sized types implement the `std::marker::Sized` trait, which has no methods or associated types. Rust implements it automatically for all types to which it applies; **you can’t implement it yourself**. The only use for `Sized` is as a bound for type variables: a bound like **`T: Sized`** requires T to be a type whose size is known at compile time. Traits of this sort are called ***marker traits***, because the Rust language itself uses them to mark certain types as having characteristics of interest.

**Rust can’t store unsized values in variables or pass them as arguments**. You can only deal with them through pointers like `&str` or `Box<dyn Write>`, which themselves are sized.

In fact, this is necessary so often that it is the implicit default in Rust: if you write struct `S<T> { ... }`, Rust understands you to mean struct `S<T: Sized> { ... }`. If you do not want to constrain T this way, you must explicitly opt out, writing `struct S<T: ?Sized> { ... }`. The `?Sized` syntax is specific to this case and means “**not necessarily Sized**.” For example, if you write struct `S<T: ?Sized> { b: Box<T> }`, then Rust will allow you to write `S<str>` and ` S<dyn Write>`, where the box becomes a fat pointer, as well as `S<i32>` and `S<String>`, where the box is an ordinary pointer.

Aside from slices and trait objects, there is one more kind of unsized type. **A struct type’s last field (but only its last) may be unsized, and such a struct is itself unsized**. 

```rust
#[cfg_attr(not(test), rustc_diagnostic_item = "Rc")]
#[stable(feature = "rust1", since = "1.0.0")]
#[rustc_insignificant_dtor]
pub struct Rc<T: ?Sized> {
    ptr: NonNull<RcBox<T>>,
    phantom: PhantomData<RcBox<T>>,
}

// This is repr(C) to future-proof against possible field-reordering, which
// would interfere with otherwise safe [into|from]_raw() of transmutable
// inner types.
#[repr(C)]
struct RcBox<T: ?Sized> {
    strong: Cell<usize>,
    weak: Cell<usize>,
    value: T,
}
```

### **Clone**

`std::clone::Clone`

```rust
pub trait Clone: Sized {
    /// Returns a copy of the value.
    ///
    /// # Examples
    ///
    /// ```
    /// # #![allow(noop_method_call)]
    /// let hello = "Hello"; // &str implements Clone
    ///
    /// assert_eq!("Hello", hello.clone());
    /// ```
    #[stable(feature = "rust1", since = "1.0.0")]
    #[must_use = "cloning is often expensive and is not expected to have side effects"]
    fn clone(&self) -> Self;

    /// Performs copy-assignment from `source`.
    ///
    /// `a.clone_from(&b)` is equivalent to `a = b.clone()` in functionality,
    /// but can be overridden to reuse the resources of `a` to avoid unnecessary
    /// allocations.
    #[inline]
    #[stable(feature = "rust1", since = "1.0.0")]
    #[default_method_body_is_const]
    #[cfg_attr(not(bootstrap), allow(drop_bounds))] // FIXME remove `~const Drop` and this attr when bumping
    fn clone_from(&mut self, source: &Self)
    where
        Self: ~const Drop + ~const Destruct,
    {
        *self = source.clone()
    }
}
```

`clone_from`方法将`self`修改为`source`的一个副本。这个方法的默认定义知识简单地克隆了`source`，然后将副本转移到`*self`中。语句`s = t.clone();`必须先克隆`t`，清除`s`原来的值，然后再将克隆的值转移到`s`中。这涉及一次堆分配和一次堆释放。在泛型代码中，应该尽可能使用`clone_from`，从而在可能的情况下应用这种优化。

加上`#[derive(Clone)]`实现。

没有实现`Clone`：`std::sync::Mutex`、`stf::file::File`（但是提供了一个`try_clone()`方法）。

### **Copy**

A type is `Copy` if it implements the `std::marker::Copy` marker trait, which is defined as follows:

```rust
pub trait Copy: Clone {
    // Empty.
}
```

But because `Copy` is a marker trait with special meaning to the language, Rust permits a type to implement `Copy` only if **a shallow byte-for-byte copy** is all it needs. Types that own any other resources, like heap buffers or operating system handles, cannot implement `Copy`. **Any type that implements the `Drop` trait cannot be `Copy`**. Rust presumes that if a type needs special cleanup code, it must also require special copying code and thus can’t be `Copy`.

As with `Clone`, you can ask Rust to derive Copy for you, using `#[derive(Copy)]`. You will often see both derived at once, with `#[derive(Copy, Clone)]`.

### **Deref and DerefMut**

`std::ops::Deref`

```rust
#[lang = "deref"]
#[doc(alias = "*")]
#[doc(alias = "&*")]
#[stable(feature = "rust1", since = "1.0.0")]
#[rustc_diagnostic_item = "Deref"]
pub trait Deref {
    /// The resulting type after dereferencing.
    #[stable(feature = "rust1", since = "1.0.0")]
    #[rustc_diagnostic_item = "deref_target"]
    #[lang = "deref_target"]
    type Target: ?Sized;

    /// Dereferences the value.
    #[must_use]
    #[stable(feature = "rust1", since = "1.0.0")]
    #[rustc_diagnostic_item = "deref_method"]
    fn deref(&self) -> &Self::Target;
}
```

`std::ops::DerefMut`

```rust
#[lang = "deref_mut"]
#[doc(alias = "*")]
#[stable(feature = "rust1", since = "1.0.0")]
pub trait DerefMut: Deref {
    /// Mutably dereferences the value.
    #[stable(feature = "rust1", since = "1.0.0")]
    fn deref_mut(&mut self) -> &mut Self::Target;
}
```

Since the methods return a reference with the same lifetime as `&self`, **`self` remains borrowed for as long as the returned reference lives**.

由于`deref`接收`&self`引用并返回`&Self::Target`引用，因此Rust会利用这一点自动将前一种类型的引用转换为后一种类型的引用。换一句话说，**如果插入一次`deref`调用可以防止类型错配，那Rust会为你插入一次**。实现`DerefMut`可以实现对可修改引用的类型转换。这种类型转换称为解引用强制转型(deref coercion)，即一种类型被“强制”表现出另一种类型的行为。

使用解引用强制类型转换很方便：

* `Rc<T>`实现了`Deref<Target=T>`
* `String`实现了`Deref<Target=str>`
* `Vec<T>`实现了`Deref<Target=[T]>`

必要情况下，**Rust会连续多次应用解引用强制转换**。

`Deref`和`DerefMut` trait的设计初衷是为了实现**智能指针类型**（如`Box`、`Rc`和`Arc`），以及某些**频繁通过引用来使用的类型的所有者版本**（如`Vec<T>`和`String`就是`[T]`和`str`的所有者版本）。

Rust通过应用解引用强制转换来解决类型冲突，但**不会应用它来满足类型变量的绑定**。

```rust
use std::fmt::Display;
fn show_it_generic<T: Display>(thing: T) { println!("{}", thing); } show_it_generic(&s); // error
```

Rust不会在满足类型变量的绑定时应用解引用强制类型转换，所以检查失败。

```rust
// 解决这个问题
show_it_generic(&s as &str);
// Or
show_it_generic(&*s);
```

### **Default**

`std::default::Default`

```rust
#[cfg_attr(not(test), rustc_diagnostic_item = "Default")]
#[stable(feature = "rust1", since = "1.0.0")]
pub trait Default: Sized {
    /// Returns the "default value" for a type.
    ///
    /// Default values are often some kind of initial value, identity value, or anything else that
    /// may make sense as a default.
    ///
    /// # Examples
    ///
    /// Using built-in default values:
    ///
    /// ```
    /// let i: i8 = Default::default();
    /// let (x, y): (Option<String>, f64) = Default::default();
    /// let (a, b, (c, d)): (i32, u32, (bool, bool)) = Default::default();
    /// ```
    ///
    /// Making your own:
    ///
    /// ```
    /// # #[allow(dead_code)]
    /// enum Kind {
    ///     A,
    ///     B,
    ///     C,
    /// }
    ///
    /// impl Default for Kind {
    ///     fn default() -> Self { Kind::A }
    /// }
    /// ```
    #[stable(feature = "rust1", since = "1.0.0")]
    fn default() -> Self;
}
```

```rust
use std::collections::HashSet;
let squares = [4, 9, 16, 25, 36, 49, 64];
let (powers_of_two, impure): (HashSet<i32>, HashSet<i32>)
      = squares.iter().partition(|&n| n & (n-1) == 0);

assert_eq!(powers_of_two.len(), 3);
assert_eq!(impure.len(), 4);
```

But of course, partition isn’t specific to `HashSets`; you can use it to produce any sort of collection you like, as long as the collection type implements `Default`, to produce an empty collection to start with, and `Extend<T>`, to add a T to the collection. `String` implements `Default` and `Extend<char>`, so you can write:

```rust
let (upper, lower): (String, String)
  = "Great Teacher Onizuka".chars().partition(|&c| c.is_uppercase());
assert_eq!(upper, "GTO");
assert_eq!(lower, "reat eacher nizuka");
```

Another common use of `Default` is to produce default values for structs that represent a large collection of parameters, most of which you won’t usually need to change. 

```rust
let params = glium::DrawParameters { 
  line_width: Some(0.02), 
  point_size: Some(0.02),
	.. Default::default()
};

target.draw(..., &params).unwrap();
```

If a type `T` implements Default, then the standard library implements Default automatically for `Rc<T>`, `Arc<T>`, `Box<T>`, `Cell<T>`, `RefCell<T>`, `Cow<T>`, `Mutex<T>`, and `RwLock<T>`. The default value for the type `Rc<T>`, for example, is an Rc pointing to the default value for type T.

如果元组类型的所有类型都实现了Default，且该该元组类型也实现了Default，那么这个元组默认会持有每个元素的默认值。

Rust does not implicitly implement Default for **struct** types, but if all of a struct’s fields implement Default, you can implement Default for the struct automatically using `# [derive(Default)]`.

### **AsRef and AsMut**

`std::convert::AsRef`

```rust
#[stable(feature = "rust1", since = "1.0.0")]
#[cfg_attr(not(test), rustc_diagnostic_item = "AsRef")]
pub trait AsRef<T: ?Sized> {
    /// Converts this type into a shared reference of the (usually inferred) input type.
    #[stable(feature = "rust1", since = "1.0.0")]
    fn as_ref(&self) -> &T;
}
```

`std::convert::RefMut`

```rust
#[stable(feature = "rust1", since = "1.0.0")]
#[cfg_attr(not(test), rustc_diagnostic_item = "AsMut")]
pub trait AsMut<T: ?Sized> {
    /// Converts this type into a mutable reference of the (usually inferred) input type.
    #[stable(feature = "rust1", since = "1.0.0")]
    fn as_mut(&mut self) -> &mut T;
}
```

So, for example, `Vec<T>` implements `AsRef<[T]>`, and String implements `AsRef<str>`. You can also borrow a String’s contents as an array of bytes, so String implements AsRef<[u8]> as well.

But this can’t be the whole story. A string literal is a &str, but the type that implements `AsRef<Path>` is str, without an &. And as we explained in “Deref and DerefMut”, Rust doesn’t try deref coercions to satisfy type variable bounds, so they won’t help here either.

Fortunately, the standard library includes the blanket implementation:

```rust
impl<'a, T, U> AsRef<U> for &'a T 
	where T: AsRef<U>,
				T: ?Sized, U: ?Sized
{
	fn as_ref(&self) -> &U {
    (*self).as_ref()
  }
}
```

In other words, for any types T and U, if `T: AsRef<U>`, then `&T: AsRef<U>` as well: simply follow the reference and proceed as before. 

You might assume that if a type implements AsRef<T>, it should also implement AsMut<T>. However, there are cases where this isn’t appropriate.

### **Borrow and BorrowMut**
The std::borrow::Borrow trait is similar to AsRef: if a type implements `Borrow<T>`, then its borrow method efficiently borrows a &T from it. But Borrow imposes more restrictions: a type should implement Borrow<T> only when a &T hashes and compares the same way as the value it’s borrowed from. (Rust doesn’t enforce this; it’s just the documented intent of the trait.) This makes Borrow valuable in dealing with keys in hash tables and trees or when dealing with values that will be hashed or compared for some other reason.

```rust
#[stable(feature = "rust1", since = "1.0.0")]
#[rustc_diagnostic_item = "Borrow"]
pub trait Borrow<Borrowed: ?Sized> {
    #[stable(feature = "rust1", since = "1.0.0")]
    fn borrow(&self) -> &Borrowed;
}
```

Borrow is designed to address a specific situation with generic hash tables and other associative collection types.

```rust
impl<K, V> HashMap<K, V> where K: Eq + Hash 
{
	fn get<Q: ?Sized>(&self, key: &Q) -> Option<&V> 
  	where K: Borrow<Q>,
					Q: Eq + Hash
	{ ... } 
}
```

Vec<T> and [T: N] implement Borrow<[T]>. Every string-like type allows borrowing its corresponding slice type: String implements Borrow<str>, PathBuf implements Borrow<Path>, and so on. And all the standard library’s associative collection types use Borrow to decide which types can be passed to their lookup functions.

The standard library includes a blanket implementation so that every type T can be borrowed from itself: T: Borrow<T>. This ensures that &K is always an acceptable type for looking up entries in a HashMap<K, V>.

`std::borrow::BorrowMut`

```rust
trait BorrowMut<Borrowed: ?Sized>: Borrow<Borrowed> { 
  fn borrow_mut(&mut self) -> &mut Borrowed;
}
```

### **From and Into**

The `std::convert::From` and `std::convert::Into` traits represent conversions that consume a value of one type and return a value of another. 

```rust
trait Into<T>: Sized { 
  fn into(self) -> T;
}
trait From<T>: Sized {
	fn from(other: T) -> Self;
}
```

The standard library automatically implements the trivial conversion from each type to itself: every type T implements `From<T>` and `Into<T>`.

You generally use Into to make your functions more flexible in the arguments they accept.

The from method serves as a generic constructor for producing an instance of a type from some other single value.

Given an appropriate From implementation, the standard library automatically implements the corresponding Into trait.

However, cheap conversions are not part of Into and From’s contract. Whereas AsRef and AsMut conversions are expected to be cheap, From and Into conversions may allocate, copy, or otherwise process the value’s contents.

The `?` operator uses From and Into to help clean up code in functions that could fail in multiple ways by automatically converting from specific error types to general ones when needed.

```rust
type GenericError = Box<dyn std::error::Error + Send + Sync + 'static>; 
type GenericResult<T> = Result<T, GenericError>;

fn parse_i32_bytes(b: &[u8]) -> GenericResult<i32>
  Ok(std::str::from_utf8(b)?.parse::<i32>()?)
}	
```

```rust
impl<'a, E: Error + Send + Sync + 'a> From<E> 
  for Box<dyn Error + Send + Sync + 'a> {
    fn from(err: E) -> Box<dyn Error + Send + Sync + 'a> { 
      Box::new(err)
    } 
}
```

From and Into are infallible traits—their API requires that conversions will not fail.

### **TryFrom and TryInto**

Instead, i32 implements TryFrom<i64>. TryFrom and TryInto are the fallible cousins of From and Into and are similarly reciprocal; implementing TryFrom means that TryInto is implemented as well.

```rust
pub trait TryFrom<T>: Sized { 
  type Error;
	fn try_from(value: T) -> Result<Self, Self::Error>; 
}

pub trait TryInto<T>: Sized { 
  type Error;
	fn try_into(self) -> Result<T, Self::Error>; 
}
```

Where From and Into relate types with simple conversions, TryFrom and TryInto extend the simplicity of From and Into conversions with the expressive error handling afforded by Result. These four traits can be used together to relate many types in a single crate.

### **ToOwned**

The `std::borrow::ToOwned` trait provides a slightly looser way to convert a reference to an owned value:

```rust
trait ToOwned {
	type Owned: Borrow<Self>;
	fn to_owned(&self) -> Self::Owned;
}
```

### **Borrow and ToOwned at Work: The Humble Cow**

But in some cases you cannot decide whether to borrow or own until the program is running ; the std::borrow::Cow type (for “clone on write”) provides one way to do this.

```rust
enum Cow<'a, B: ?Sized> 
	where B: ToOwned
{
  Borrowed(&'a B),
  Owned(<B as ToOwned>::Owned),
}
```

A `Cow<B>` either borrows a shared reference to a B or owns a value from which we could borrow such a reference. Since Cow implements Deref, you can call methods on it as if it were a shared reference to a B: if it’s Owned, it borrows a shared reference to the owned value; and if it’s Borrowed, it just hands out the reference it’s holding.

You can also get a mutable reference to a Cow’s value by calling its to_mut method, which returns a &mut B. If the Cow happens to be Cow::Borrowed, to_mut simply calls the reference’s to_owned method to get its own copy of the referent, changes the Cow into a Cow::Owned, and borrows a mutable reference to the newly owned value. This is the “clone on write” behavior the type’s name refers to.

One common use for Cow is to return either a statically allocated string constant or a computed string.

```rust

use std::path::PathBuf;
use std::borrow::Cow;
fn describe(error: &Error) -> Cow<'static, str> {
	match *error {
		Error::OutOfMemory => "out of memory".into(), 
    Error::StackOverflow => "stack overflow".into(), 
    Error::MachineOnFire => "machine on fire".into(), 
    Error::Unfathomable => "machine bewildered".into(),
    Error::FileNotFound(ref path) => {
    	format!("file not found: {}", path.display()).into()
    }
	} 
}
```

## **Chapter 14. Closures**

It’s more concise to write the helper function as a *closure*, an anonymous function expression:

```rust
fn sort_cities(cities: &mut Vec<City>) { 
  cities.sort_by_key(|city| - city.population);
}
```

### **Capturing Variables**

A closure can use data that belongs to an enclosing function.

```rust
fn sort_cities(cities: &mut Vec<City>) { 
  cities.sort_by_key(|city| - city.population);
}
```

#### Closures That Borrow

In this case, when Rust creates the closure, it automatically borrows a reference to stat. It stands to reason: the closure refers to stat, so it must have a reference to it.

In short, Rust ensures safety by using lifetimes instead of garbage collection. Rust’s way is faster: even a fast GC allocation will be slower than storing stat on the stack, as Rust does in this case.

#### **Closures That Steal**

```rust
fn start_sorting_thread(mut cities: Vec<City>, stat: Statistic) 
	-> thread::JoinHandle<Vec<City>>
{
	let key_fn = move |city: &City| -> i64 { -city.get_statistic(stat) };
  
	thread::spawn(move || { 
    cities.sort_by_key(key_fn); 
    cities
  })
}
```

The only thing we’ve changed is to add the move keyword before each of the two closures. The move keyword tells Rust that a closure doesn’t borrow the variables it uses: it steals them.

Rust thus offers two ways for closures to get data from enclosing scopes: moves and borrowing.

We get something important by accepting Rust’s strict rules: thread safety. It is precisely because the vector is moved, rather than being shared across threads, that we know the old thread won’t free the vector while the new thread is modifying it.

### **Function and Closure Types**

```rust
/// Given a list of cities and a test function, 
/// return how many cities pass the test.
fn count_selected_cities(cities: &Vec<City>,
                          test_fn: fn(&City) -> bool) -> usize
{
  let mut count = 0; for city in cities {
    if test_fn(city) { 
      count += 1;
     } 
  }
  count
}

/// An example of a test function. Note that the type of 
/// this function is `fn(&City) -> bool`, the same as 
/// the `test_fn` argument to `count_selected_cities`. 
fn has_monster_attacks(city: &City) -> bool {
      city.monster_attack_risk > 0.0
}

// How many cities are at risk for monster attack?
let n = count_selected_cities(&my_cities, has_monster_attacks);
```

```rust
fn count_selected_cities<F>(cities: &Vec<City>, test_fn: F) -> usize 
	where F: Fn(&City) -> bool
{
  let mut count = 0; for city in cities {
    if test_fn(city) { 
      count += 1;
    } 
  }
  count
}
```

```rust
fn(&City) -> bool // fn type (functions only)
Fn(&City) -> bool // Fn trait (both functions and closures)
```

In fact, every closure you write has its own type, because a closure may contain data: values either borrowed or stolen from enclosing scopes. This could be any number of variables, in any combination of types. So every closure has an ad hoc type created by the compiler, large enough to hold that data. No two closures have exactly the same type. But every closure implements an Fn trait; the closure in our example implements `Fn(&City) -> i64`.

### **Closure Performance**

![image-20220719092655451](/img/2022-07-05-Programming-Rust/image-20220719092655451.png)

As the figure shows, these closures don’t take up much space. But even those few bytes are not always needed in practice. Often, the compiler can inline all calls to a closure, and then even the small structs shown in this figure are optimized away.

### **Closures and Safety**

#### **Closures That Kill**

```rust
let my_str = "hello".to_string(); 
let f = || drop(my_str);

f(); // ok
f(); // error: use of moved value
```

A closure that can be called only once may seem like a rather extraordinary thing, but we’ve been talking throughout this book about ownership and lifetimes. The idea of values being used up (that is, moved) is one of the core concepts in Rust. It works the same with closures as with everything else.

#### **FnOnce**

The first time you call a FnOnce closure, *the closure itself is used up.* It’s as though the two traits, Fn and FnOnce, were defined like this:

```rust
// Pseudocode for `Fn` and `FnOnce` traits with no arguments. 
trait Fn() -> R {
	fn call(&self) -> R; 
}

trait FnOnce() -> R {
	fn call_once(self) -> R;
}
```

Just as an arithmetic expression like a + b is shorthand for a method call, Add::add(a, b), Rust treats closure() as shorthand for one of the two trait methods shown in the preceding example. For an Fn closure, closure() expands to closure.call(). This method takes self by reference, so the closure is not moved. But if the closure is only safe to call once, then closure() expands to closure.call_once(). That method takes self by value, so the closure is used up.

#### FnMut

Therefore, Rust has one more category of closure, FnMut, the category of closures that write. FnMut closures are called by mut reference, as if they were defined like this:

```rust
// Pseudocode for `Fn`, `FnMut`, and `FnOnce` traits.
trait Fn() -> R {
  fn call(&self) -> R;
}

trait FnMut() -> R {
	fn call_mut(&mut self) -> R;
}

trait FnOnce() -> R {
  fn call_once(self) -> R;
}
```

Any closure that requires mut access to a value, but doesn’t drop any values, is an FnMut closure.

* Fn is the family of closures and functions that you can call multiple times without restriction. This highest category also includes all fn functions.
* FnMut is the family of closures that can be called multiple times if the closure itself is declared mut.
* FnOnce is the family of closures that can be called once, if the caller owns the closure.

![image-20220719095623728](/img/2022-07-05-Programming-Rust/image-20220719095623728.png)

#### **Copy and Clone for Closures**

Just as Rust automatically figures out which closures can be called only once, it can figure out which closures can implement Copy and Clone, and which cannot.

A non-move closure that doesn’t mutate variables holds only shared references, which are both Clone and Copy, so that closure is both Clone and Copy as well.

On the other hand, a non-move closure that *does* mutate values has mutable references within its internal representation. Mutable references are neither Clone nor Copy, so neither is a closure that uses them.

For a move closure, the rules are even simpler. If everything a move closure captures is Copy, it’s Copy. If everything it captures is Clone, it’s Clone. 

### **Callbacks**

Closures have unique types because each one captures different variables, so among other things, they’re each a different size. If they don’t capture anything, though, there’s nothing to store. By using fn pointers in functions that take callbacks, you can restrict a caller to use only these noncapturing closures, gaining some perfomance and flexibility within the code using callbacks at the cost of flexibility for the users of your API.

### **Using Closures Effectively**

MVC

![image-20220719103619045](/img/2022-07-05-Programming-Rust/image-20220719103619045.png)

替代方案：

![image-20220719104801927](/img/2022-07-05-Programming-Rust/image-20220719104801927.png)

## **Chapter 15. Iterators**

```rust
fn triangle(n: i32) -> i32 {
  let mut sum = 0;
  for i in 1..=n {
    sum += i;
  }
  sum
}
```

这里`1..=n`是一个`RangeInclusive<i32>`的值。A `RangeInclusive<i32>` is an iterator that produces the integers from its start value to its end value (both inclusive), so you can use it as the operand of the for loop to sum the values from 1 to n.

也可以用闭包写，更简洁和优雅：

```rust
fn triangle(n: i32) -> i32 {
  (1..=n).fold(0, |sum, item| sum + item)
}
```

### **The Iterator and IntoIterator Traits**

`std::iter::Iterator`

```rust
trait Iterator { 
  type Item;
	fn next(&mut self) -> Option<Self::Item>;
	... // many default methods 
}
```

`std::iter::IntoIterator`

```rust
trait IntoIterator where Self::IntoIter: Iterator<Item=Self::Item> { 
  type Item;
	type IntoIter: Iterator;
	fn into_iter(self) -> Self::IntoIter; 
}
```

这个表示可迭代的，可以调用 `into_iter()` 方法产生一个迭代器。

```rust
println!("There's:");
let v = vec!["antimony", "arsenic", "aluminum", "selenium"];
for element in &v { 
  println!("{}", element);
}
```

等价于

```rust
let mut iterator = (&v).into_iter();
while let Some(element) = iterator.next() {
  println!("{}", element);
}
```

Although a for loop always calls into_iter on its operand, you can also pass iterators to for loops directly; this occurs when you loop over a Range, for example. All iterators automatically implement IntoIterator, with an into_iter method that simply returns the iterator.

### **Creating Iterators**

#### **iter and iter_mut Methods**

Most collection types provide iter and iter_mut methods that return the natural iterators over the type, producing a shared or mutable reference to each item.

```rust
let v = vec![4, 20, 12, 8, 6];
let mut iterator = v.iter(); 
assert_eq!(iterator.next(), Some(&4)); 
assert_eq!(iterator.next(), Some(&20)); 
assert_eq!(iterator.next(), Some(&12)); 
assert_eq!(iterator.next(), Some(&8)); 
assert_eq!(iterator.next(), Some(&6)); 
assert_eq!(iterator.next(), None);
```

This iterator’s item type is &i32: each call to next produces a reference to the next element, until we reach the end of the vector.

For example, there is no iter method on the &str string slice type. Instead, if s is a &str, then s.bytes() returns an iterator that produces each byte of s, whereas s.chars() interprets the contents as UTF-8 and produces each Unicode character.

#### **IntoIterator Implementations**

Most collections actually provide several implementations of IntoIterator, for shared references (&T), mutable references (&mut T), and moves (T):

* Given a *shared reference* to the collection, into_iter returns an iterator that produces shared references to its items. For example, in the preceding code, (&favorites).into_iter() would return an iterator whose Item type is &String.
* Given a *mutable reference* to the collection, into_iter returns an iterator that produces mutable references to the items. For example, if vector is some Vec<String>, the call (&mut vector).into_iter() returns an iterator whose Item type is &mut String.
* When passed the collection *by value*, into_iter returns an iterator that takes ownership of the collection and returns items by value; the items’ ownership moves from the collection to the consumer, and the original collection is consumed in the process. For example, the call favorites.into_iter() in the preceding code returns an iterator that produces each string by value; the consumer receives ownership of each string. When the iterator is dropped, any elements remaining in the BTreeSet are dropped too, and the set’s now-empty husk is disposed of.

The general principle is that iteration should be efficient and predictable, so rather than providing implementations that are expensive or could exhibit surprising behavior (for example, rehashing modified HashSet entries and potentially encountering them again later in the iteration), Rust omits them entirely.

用在泛型上：

```rust
use std::fmt::Debug;

fn dump<T, U>(t: T)
	where T: IntoIterator<Item=U>,
				U: Debug
{
  for u in t {
    println!("{:?}", u);
  }
}
```

#### **from_fn and successors**

`std::iter::from_fn`

```rust
pub fn from_fn<T, F>(f: F) -> FromFn<F>
where
    F: FnMut() -> Option<T>, 

impl<T, F> Iterator for FromFn<F> 
where
    F: FnMut() -> Option<T>, 
    type Item = T;
```

```rust
use rand::random; // In Cargo.toml dependencies: rand = "0.7" 
use std::iter::from_fn;

// Generate the lengths of 1000 random line segments whose endpoints 
// are uniformly distributed across the interval [0, 1]. (This isn't a 
// distribution you're going to find in the `rand_distr` crate, but 
// it's easy to make yourself.)
let lengths: Vec<f64> =
	from_fn(|| Some((random::<f64>() - random::<f64>()).abs())) 
	.take(1000)
	.collect();
```

If each item depends on the one before, `std::iter::successors`.

```rust
pub fn successors<T, F>(first: Option<T>, succ: F) -> Successors<T, F>ⓘ 
where
    F: FnMut(&T) -> Option<T>, 

impl<T, F> Iterator for Successors<T, F> 
where
    F: FnMut(&T) -> Option<T>, 
    type Item = T;
```

```rust
use num::Complex;
use std::iter::successors;

fn escape_time(c: Complex<f64>, limit: usize) -> Option<usize> { 
  let zero = Complex { re: 0.0, im: 0.0 }; 
  successors(Some(zero), |&z| { Some(z * z + c) })
    .take(limit)
    .enumerate()
    .find(|(_i, z)| z.norm_sqr() > 4.0)
    .map(|(i, _z)| i)
}
```

斐波那契数列

```rust
fn fibonacci() -> impl Iterator<Item=usize> { 
  let mut state = (0, 1); 
  std::iter::from_fn(move || {
    state = (state.1, state.0 + state.1);
    Some(state.0)
  })
}

assert_eq!(fibonacci().take(8).collect::<Vec<_>>(),
  					vec![1, 1, 2, 3, 5, 8, 13, 21]);
```

#### **drain Methods**

Many collection types provide a drain method that takes a mutable reference to the collection and returns an iterator that passes ownership of each element to the consumer. However, unlike the into_iter() method, which takes the collection by value and consumes it, drain merely borrows a mutable reference to the collection, and when the iterator is dropped, it removes any remaining elements from the collection and leaves it empty.

```rust
use std::iter::FromIterator;
let mut outer = "Earth".to_string();
let inner = String::from_iter(outer.drain(1..4)); assert_eq!(outer, "Eh");
assert_eq!(inner, "art");
```

#### **Other Iterator Sources**

省略

### **Iterator Adapters**

#### **map and filter**

The Iterator trait’s map adapter lets you transform an iterator by applying a closure to its items. The filter adapter lets you filter out items from an iterator, using a closure to decide which to keep and which to drop.

```rust
let text = " ponies \n giraffes\niguanas \nsquid".to_string(); 
let v: Vec<&str> = text.lines()
	.map(str::trim)
	.collect();
assert_eq!(v, ["ponies", "giraffes", "iguanas", "squid"]);
```

```rust
let text = " ponies \n giraffes\niguanas \nsquid".to_string(); 
let v: Vec<&str> = text.lines()
	.map(str::trim)
	.filter(|s| *s != "iguanas") .collect();
assert_eq!(v, ["ponies", "giraffes", "squid"]);
```

```rust
// 标准库中返回的是 std::iter::Map
fn map<B, F>(self, f: F) -> impl Iterator<Item=B> 
	where Self: Sized, F: FnMut(Self::Item) -> B;

// 标准库中返回的是 std::iter::Filter
fn filter<P>(self, predicate: P) -> impl Iterator<Item=Self::Item> 
	where Self: Sized, P: FnMut(&Self::Item) -> bool;
```

First, simply calling an adapter on an iterator doesn’t consume any items; it just returns a new iterator, ready to produce its own items by drawing from the first iterator as needed. In a chain of adapters, the only way to make any work actually get done is to call next on the final iterator.

**The term “lazy”** in the error message is not a disparaging term; it’s just jargon for any mechanism that puts off a computation until its value is needed. It is Rust’s convention that iterators should do the minimum work necessary to satisfy each call to next; in the example, there are no such calls at all, so no work takes place.

The second important point is that iterator adapters are a **zero-overhead abstraction**. 

#### **filter_map and flat_map**

```rust
fn filter_map<B, F>(self, f: F) -> impl Iterator<Item=B> 
	where Self: Sized, F: FnMut(Self::Item) -> Option<B>;
```

```rust
use std::str::FromStr;
let text = "1\nfrond .25 289\n3.1415 estuary\n"; 
for number in text
	.split_whitespace()
	.filter_map(|w| f64::from_str(w).ok()) {
      println!("{:4.2}", number.sqrt());
  }
```

```rust
fn flat_map<U, F>(self, f: F) -> impl Iterator<Item=U::Item>
	where F: FnMut(Self::Item) -> U, U: IntoIterator;
```

```rust
use std::collections::HashMap;
let mut major_cities = HashMap::new();
major_cities.insert("Japan", vec!["Tokyo", "Kyoto"]); major_cities.insert("The United States", vec!["Portland", "Nashville"]); major_cities.insert("Brazil", vec!["São Paulo", "Brasília"]); major_cities.insert("Kenya", vec!["Nairobi", "Mombasa"]); major_cities.insert("The Netherlands", vec!["Amsterdam", "Utrecht"]);

let countries = ["Japan", "Brazil", "Kenya"];

for &city in countries.iter().flat_map(|country| &major_cities[country]) { 		println!("{}", city);
}
```

运行结果：

```
Tokyo
Kyoto
São Paulo
Brasília
Nairobi
Mombasa
```

#### **flatten**

```rust
use std::collections::BTreeMap;

// A table mapping cities to their parks: each value is a vector.
let mut parks = BTreeMap::new();
parks.insert("Portland", vec!["Mt. Tabor Park", "Forest Park"]); parks.insert("Kyoto", vec!["Tadasu-no-Mori Forest", "Maruyama Koen"]); parks.insert("Nashville", vec!["Percy Warner Park", "Dragon Park"]);

// Build a vector of all parks. `values` gives us an iterator producing 
// vectors, and then `flatten` produces each vector's elements in turn. 
let all_parks: Vec<_> = parks.values().flatten().cloned().collect();

assert_eq!(all_parks,
             vec!["Tadasu-no-Mori Forest", "Maruyama Koen", "Percy Warner Park", 
               "Dragon Park", "Mt. Tabor Park", "Forest Park"]);
```

```rust
fn flatten(self) -> impl Iterator<Item=Self::Item::Item>
	where Self::Item: IntoIterator;
```

#### **take and take_while**

The Iterator trait’s take and take_while adapters let you end an iteration after a certain number of items or when a closure decides to cut things off.

```rust
fn take(self, n: usize) -> impl Iterator<Item=Self::Item> 
	where Self: Sized;
fn take_while<P>(self, predicate: P) -> impl Iterator<Item=Self::Item> 
	where Self: Sized, P: FnMut(&Self::Item) -> bool;
```

```rust
let message = "To: jimb\r\n\
							 From: superego <editor@oreilly.com>\r\n\
							 \r\n\
							 Did you get any writing done today?\r\n\
							 When will you stop wasting time plotting fractals?\r\n";
for header in message.lines().take_while(|l| !l.is_empty()) { 
  println!("{}" , header);
}
```

结果：

```rust
To: jimb
From: superego <editor@oreilly.com>
```

#### **skip and skip_while**

The Iterator trait’s skip and skip_while methods are the complement of take and take_while: they drop a certain number of items from the beginning of an iteration, or drop items until a closure finds one acceptable, and then pass the remaining items through unchanged.

```rust
fn skip(self, n: usize) -> impl Iterator<Item=Self::Item>
	where Self: Sized;
fn skip_while<P>(self, predicate: P) -> impl Iterator<Item=Self::Item> 
	where Self: Sized, P: FnMut(&Self::Item) -> bool;
```

```rust
for body in message.lines() 
	.skip_while(|l| !l.is_empty()) 
	.skip(1) {
	println!("{}" , body);
}
```

输出结果为：

```
Did you get any writing done today?
When will you stop wasting time plotting fractals?
```

#### **peekable**

A peekable iterator lets you peek at the next item that will be produced without actually consuming it. You can turn any iterator into a peekable iterator by calling the Iterator trait’s peekable method:

```rust
fn peekable(self) -> std::iter::Peekable<Self> 
	where Self: Sized;
```

Here, `Peekable<Self>` is a struct that implements `Iterator<Item=Self::Item>`, and Self is the type of the underlying iterator.

#### **fuse**

`fuse` 适配器可以将任何适配器转换为第一次返回`None`之后始终继续返回`None`的迭代器。

```rust
struct Flaky(bool);
impl Iterator for Flaky {
	type Item = &'static str;
	fn next(&mut self) -> Option<Self::Item> {
    if self.0 {
      self.0 = false;
      Some("totally the last item") 
    } else {
      self.0 = true; // D'oh!
      None
    } 
  }
}

let mut flaky = Flaky(true);
assert_eq!(flaky.next(), Some("totally the last item")); 
assert_eq!(flaky.next(), None);
assert_eq!(flaky.next(), Some("totally the last item"));

let mut not_flaky = Flaky(true).fuse(); 
assert_eq!(not_flaky.next(), Some("totally the last item")); 
assert_eq!(not_flaky.next(), None); 
assert_eq!(not_flaky.next(), None);
```

#### **Reversible Iterators and rev**

从序列两端取得项。实现`std::iter::DoubleEndedIterator` trait。

```rust
trait DoubleEndedIterator: Iterator {
	fn next_back(&mut self) -> Option<Self::Item>;
}
```

```rust
let bee_parts = ["head", "thorax", "abdomen"];

let mut iter = bee_parts.iter(); 
assert_eq!(iter.next(), Some(&"head")); 
assert_eq!(iter.next_back(), Some(&"abdomen")); 
assert_eq!(iter.next(), Some(&"thorax"));

assert_eq!(iter.next_back(), None);
assert_eq!(iter.next(),      None);
```

If an iterator is double-ended, you can reverse it with the rev adapter:

```rust
fn rev(self) -> impl Iterator<Item=Self>
	where Self: Sized + DoubleEndedIterator;
```

The returned iterator is also double-ended: its next and next_back methods are simply exchanged:

```rust
let meals = ["breakfast", "lunch", "dinner"];

let mut iter = meals.iter().rev(); 
assert_eq!(iter.next(), Some(&"dinner")); 
assert_eq!(iter.next(), Some(&"lunch")); 
assert_eq!(iter.next(), Some(&"breakfast")); 
assert_eq!(iter.next(), None);
```

#### **inspect**

```rust
let upper_case: String = "große".chars() 
	.inspect(|c| println!("before: {:?}", c)) 
	.flat_map(|c| c.to_uppercase())
	.inspect(|c| println!(" after: {:?}", c)) 
	.collect();
assert_eq!(upper_case, "GROSSE");
```

#### **chain**

将一个迭代器加到另一个迭代器后面。

```rust
fn chain<U>(self, other: U) -> impl Iterator<Item=Self::Item>
	where Self: Sized, U: IntoIterator<Item=Self::Item>;
```

In other words, you can chain an iterator together with any iterable that produces the same item type.

```rust
let v: Vec<i32> = (1..4).chain(vec![20, 30, 40]).collect(); 
assert_eq!(v, [1, 2, 3, 20, 30, 40]);
```

#### **enumerate**

```rust
for (i, band) in bands.into_iter().enumerate() {
  let top = band_rows * i;
  // start a thread to render rows `top..top + band_rows` ...
}
```

You can think of the (index, item) pairs that enumerate produces as analogous to the (key, value) pairs that you get when iterating over a HashMap or other associative collection. If you’re iterating over a slice or vector, the index is the “key” under which the item appears.

#### **zip**

The zip adapter combines two iterators into a single iterator that produces pairs holding one value from each iterator, like a zipper joining its two sides into a single seam. The zipped iterator ends when either of the two underlying iterators ends.

```rust
let v: Vec<_> = (0..).zip("ABCD".chars()).collect(); 
assert_eq!(v, vec![(0, 'A'), (1, 'B'), (2, 'C'), (3, 'D')]);
```

The argument to zip doesn’t need to be an iterator itself; it can be any iterable:

```rust
use std::iter::repeat;

let endings = vec!["once", "twice", "chicken soup with rice"]; 
let rhyme: Vec<_> = repeat("going")
		.zip(endings)
    .collect();
assert_eq!(rhyme, vec![("going", "once"),
                       ("going", "twice"),
                       ("going", "chicken soup with rice")]);
```

#### **by_ref**

An iterator’s by_ref method borrows a mutable reference to the iterator so that you can apply adapters to the reference. When you’re done consuming items from these adapters, you drop them, the borrow ends, and you regain access to your original iterator.

```rust
let message = "To: jimb\r\n\ 
							From: id\r\n\
							\r\n\
							Oooooh, donuts!!\r\n";

let mut lines = message.lines();

println!("Headers:");
for header in lines.by_ref().take_while(|l| !l.is_empty()) {
  println!("{}" , header);
}

println!("\nBody:"); 
for body in lines {
  println!("{}" , body);
}
```

#### cloned, copied

The cloned adapter takes an iterator that produces references and returns an iterator that produces values cloned from those references, much like iter.map(|item| item.clone()). Naturally, the referent type must implement Clone. 

```rust
let a = ['1', '2', '3', '∞'];

assert_eq!(a.iter().next(),          Some(&'1'));
assert_eq!(a.iter().cloned().next(), Some('1'));
```

The copied adapter is the same idea, but more restrictive: the referent type must implement Copy. A call like iter.copied() is roughly the same as iter.map(|r| *r).

#### **cycle**

The cycle adapter returns an iterator that endlessly repeats the sequence produced by the underlying iterator. The underlying iterator must implement std::clone::Clone so that cycle can save its initial state and reuse it each time the cycle starts again.

```rust
let dirs = ["North", "East", "South", "West"]; 
let mut spin = dirs.iter().cycle(); 
assert_eq!(spin.next(), Some(&"North")); 
assert_eq!(spin.next(), Some(&"East")); 
assert_eq!(spin.next(), Some(&"South")); 
assert_eq!(spin.next(), Some(&"West")); 
assert_eq!(spin.next(), Some(&"North")); 
assert_eq!(spin.next(), Some(&"East"));
```

### **Consuming Iterators**

#### **Simple Accumulation: count, sum, product**

```rust
fn count(self) -> usize
  where
      Self: Sized,
  {
      self.fold(
          0,
          #[rustc_inherit_overflow_checks]
          |count, _| count + 1,
      )
  }
```

The count method draws items from an iterator until it returns None and tells you how many it got. 

The sum and product methods compute the sum or product of the iterator’s items, which must be integers or floating-point numbers.

#### **max, min**

`std::iter::Iterator`

```rust
fn max(self) -> Option<Self::Item>
where
    Self: Sized,
    Self::Item: Ord,
{
    self.max_by(Ord::cmp)
}
```

#### **max_by, min_by**

```rust
fn max_by<F>(self, compare: F) -> Option<Self::Item>
where
		Self: Sized,
    F: FnMut(&Self::Item, &Self::Item) -> Ordering, 
```

```rust
use std::cmp::Ordering;

// Compare two f64 values. Panic if given a NaN.
fn cmp(lhs: &f64, rhs: &f64) -> Ordering { 
  lhs.partial_cmp(rhs).unwrap()
}

let numbers = [1.0, 4.0, 2.0]; 
assert_eq!(numbers.iter().copied().max_by(cmp), Some(4.0)); 
assert_eq!(numbers.iter().copied().min_by(cmp), Some(1.0));

let numbers = [1.0, 4.0, std::f64::NAN, 2.0]; 
assert_eq!(numbers.iter().copied().max_by(cmp), Some(4.0)); // panics
```

#### **max_by_key, min_by_key**

```rust
fn max_by_key<B: Ord, F>(self, f: F) -> Option<Self::Item>
where
    Self: Sized,
    F: FnMut(&Self::Item) -> B,

fn min_by_key<B: Ord, F>(self, f: F) -> Option<Self::Item>
where
    Self: Sized,
    F: FnMut(&Self::Item) -> B,
```

```rust
use std::collections::HashMap;

let mut populations = HashMap::new(); 
populations.insert("Portland", 583_776);
populations.insert("Fossil",  449);
populations.insert("Greenhorn", 2);
populations.insert("Boring", 7_762);
populations.insert("The Dalles", 15_340);

assert_eq!(populations.iter().max_by_key(|&(_name, pop)| pop),
  					Some((&"Portland", &583_776)));
assert_eq!(populations.iter().min_by_key(|&(_name, pop)| pop),
  					Some((&"Greenhorn", &2)));
```

#### **Comparing Item Sequences**

Iterators provide the eq and ne methods for equality comparisons, and lt, le, gt, and ge methods for ordered comparisons. The cmp and partial_cmp methods behave like the corresponding methods of the Ord and PartialOrd traits.

#### **any and all**

```rust
fn any<F>(&mut self, f: F) -> bool
where
		Self: Sized,
    F: FnMut(Self::Item) -> bool, 

fn all<F>(&mut self, f: F) -> bool
where
		Self: Sized,
    F: FnMut(Self::Item) -> bool, 
```

#### **position, rposition, and ExactSizeIterator**

The position method applies a closure to each item from the iterator and returns the index of the first item for which the closure returns true.

The rposition method is the same, except that it searches from the right.

```rust
fn rposition<P>(&mut self, predicate: P) -> Option<usize>
where
    P: FnMut(Self::Item) -> bool,
    Self: ExactSizeIterator + DoubleEndedIterator, 
```

```rust
trait ExactSizeIterator: Iterator { 
  fn len(&self) -> usize { ... } 			// 返回剩余项数
	fn is_empty(&self) -> bool { ... }	// 在迭代完成时返回 true
}
```

#### **fold and rfold**

```rust
fn fold<B, F>(self, init: B, f: F) -> B
where
		Self: Sized,
    F: FnMut(B, Self::Item) -> B, 
```

fold方法是一个通用工具，可以对迭代器产生项的整个序列执行某些累计操作。这个方法接收一个名为累加器的初始值和一个闭包，然后对当前累加器和迭代器的下一项重复应用闭包。每次闭包的返回值都会成为累加器的新值，然后再和迭代器的下一项一块传给闭包。累加器的最终值也是fold方法返回的值。如果序列是空的，则fold返回累加器的初始值。

```rust
let a = [5, 6, 7, 8, 9, 10];
assert_eq!(a.iter().fold(0, |n, _| n+1), 6); 			// count
assert_eq!(a.iter().fold(0, |n, i| n+i), 45); 		// sum
assert_eq!(a.iter().fold(1, |n, i| n*i), 151200);	// product
assert_eq!(a.iter().cloned().fold(i32::min_value(), std::cmp::max), 10); // max
```

累加器的值会转移到闭包中再转移出来，因此可以对非Copy类型的累加器使用fold：

```rust
let a = ["Pack", "my", "box", "with",
           "five", "dozen", "liquor", "jugs"];

// See also: the `join` method on slices, which won't 
// give you that extra space at the end.
let pangram = a.iter()
      .fold(String::new(), |s, w| s + w + " ");
assert_eq!(pangram, "Pack my box with five dozen liquor jugs ");
```

The rfold method is the same as fold, except that it requires a double-ended iterator, and processes its items from last to first.

#### **try_fold and try_rfold**

The try_fold method is the same as fold, except that the process of iteration can exit early, without consuming all the values from the iterator. The closure you pass to try_fold must return a Result: if it returns Err(e), try_fold returns immediately with Err(e) as its value. Otherwise, it continues folding with the success value. The closure can also return an Option: returning None exits early, and the result is an Option of the folded value.

```rust
fn try_fold<B, F, R>(&mut self, init: B, f: F) -> R
where
		Self: Sized,
    F: FnMut(B, Self::Item) -> R,
    R: Try<Output = B>, 
```

```rust
use std::error::Error; 
use std::io::prelude::*; 
use std::str::FromStr;

fn main() -> Result<(), Box<dyn Error>> { 
  let stdin = std::io::stdin();
  let sum = stdin.lock()
      .lines()
      .try_fold(0, |sum, line| -> Result<u64, Box<dyn Error>> {
        Ok(sum + u64::from_str(&line?.trim())?) 
      })?;
  println!("{}", sum);
  Ok(()) 
}

```

```rust
fn all<P>(&mut self, mut predicate: P) -> bool 
	where P: FnMut(Self::Item) -> bool,
		Self: Sized
{
  self.try_fold((), |_, item| {
		if predicate(item) { Some(()) } else { None } 
  }).is_some()
}
```

#### **nth, nth_back**

```rust
fn nth(&mut self, n: usize) -> Option<Self::Item>
```

#### **last**

```rust
fn last(self) -> Option<Self::Item>
```

This consumes all the iterator’s items starting from the front, even if the iterator is reversible. If you have a reversible iterator and don’t need to consume all its items, you should instead just write iter.next_back().

#### **find, rfind, and find_map**

```rust
fn find<P>(&mut self, predicate: P) -> Option<Self::Item>
	where
    Self: Sized,
    P: FnMut(&Self::Item) -> bool,
```

```rust
assert_eq!(populations.iter().find(|&(_name, &pop)| pop > 1_000_000),
           None);
assert_eq!(populations.iter().find(|&(_name, &pop)| pop > 500_000),
  			   Some((&"Portland", &583_776)));
```

```rust
fn find_map<B, F>(&mut self, f: F) -> Option<B>
	where
    Self: Sized,
    F: FnMut(Self::Item) -> Option<B>,
```

#### **Building Collections: collect and FromIterator**

```rust
fn collect<B: FromIterator<Self::Item>>(self) -> B
where
	Self: Sized,
{
  FromIterator::from_iter(self)
}
```

```rust
trait FromIterator<A>: Sized {
	fn from_iter<T: IntoIterator<Item=A>>(iter: T) -> Self;
}
```

分配时提高效率：

```rust
trait Iterator { 
  ...
	fn size_hint(&self) -> (usize, Option<usize>) { 
    (0, None)
	} 
}
```

This method returns a lower bound and optional upper bound on the number of items the iterator will produce. The default definition returns zero as the lower bound and declines to name an upper bound, saying, in effect, “I have no idea,” but many iterators can do better than this. An iterator over a Range, for example, knows exactly how many items it will produce, as does an iterator over a Vec or HashMap. Such iterators provide their own specialized definitions for size_hint.

#### **The Extend Trait**

If a type implements the std::iter::Extend trait, then its extend method adds an iterable’s items to the collection.

```rust
trait Extend<A> {
	fn extend<T>(&mut self, iter: T)
		where T: IntoIterator<Item=A>;
}
```

#### **partition**

The partition method divides an iterator’s items among two collections, using a closure to decide where each item belongs.

```rust
fn partition<B, F>(self, f: F) -> (B, B)
where
    Self: Sized,
    B: Default + Extend<Self::Item>,
    F: FnMut(&Self::Item) -> bool,
{
    #[inline]
    fn extend<'a, T, B: Extend<T>>(
        mut f: impl FnMut(&T) -> bool + 'a,
        left: &'a mut B,
        right: &'a mut B,
    ) -> impl FnMut((), T) + 'a {
        move |(), x| {
            if f(&x) {
                left.extend_one(x);
            } else {
                right.extend_one(x);
            }
        }
    }

    let mut left: B = Default::default();
    let mut right: B = Default::default();

    self.fold((), extend(f, &mut left, &mut right));

    (left, right)
}
```

Other languages offer partition operations that just split the iterator into two iterators, instead of building two collections. But this isn’t a good fit for Rust: items drawn from the underlying iterator but not yet drawn from the appropriate partitioned iterator would need to be buffered somewhere; you would end up building a collection of some sort internally, anyway.

#### **for_each and try_for_each**
The for_each method simply applies a closure to each item.
```rust
fn for_each<F>(self, f: F)
where
    Self: Sized,
    F: FnMut(Self::Item),
{
    #[inline]
    fn call<T>(mut f: impl FnMut(T)) -> impl FnMut((), T) {
        move |(), item| f(item)
    }

    self.fold((), call(f));
}
```
```rust
["doves", "hens", "birds"].iter()
      .zip(["turtle", "french", "calling"].iter())
      .zip(2..5)
      .rev()
      .map(|((item, kind), quantity)| {
          format!("{} {} {}", quantity, kind, item)
      })
      .for_each(|gift| {
          println!("You have received: {}", gift);
});
```
打印输出：
```bash
You have received: 4 calling birds
You have received: 3 french hens
You have received: 2 turtle doves
```

If your closure needs to be fallible or exit early, you can use try_for_each.

```rust
...
    .try_for_each(|gift| {
      writeln!(&mut output_file, "You have received: {}", gift) 
    })?;
```

### Implementing Your Own Iterators
```rust
use std::iter::Iterator;

struct I32Range {
    start: i32, // 当前的值
    end: i32    // 迭代结束的值
}

impl Iterator for I32Range {
    type Item = i32;
    fn next(&mut self) -> Option<i32> {
        if self.start >= self.end {
            return None;
        }
        let result = Some(self.start);
        self.start += 1;
        result
    }
}


fn main() {
    let mut pi = 0.0;
    let mut numerator = 1.0;

    for k in (I32Range { start: 0, end: 14}) {
        pi += numerator / (2*k + 1) as f64;
        numerator /= -3.0;
    }
    pi *= f64::sqrt(12.0);

    // IEEE 754 specifies this result exactly
    assert_eq!(pi as f32, std::f32::consts::PI);
}
```

```rust
use crate::BinaryTree::NonEmpty;

enum BinaryTree<T> {
    Empty,
    NonEmpty(Box<TreeNode<T>>)
}

struct TreeNode<T> {
    element: T,
    left: BinaryTree<T>,
    right: BinaryTree<T>
}

impl<T: Ord>  BinaryTree<T> {
    fn add(&mut self, value: T) {
        match *self {
            BinaryTree::Empty =>
                *self = BinaryTree::NonEmpty(Box::new(TreeNode{
                    element: value,
                    left: BinaryTree::Empty,
                    right: BinaryTree::Empty,
                })),
            BinaryTree::NonEmpty(ref mut node) =>
                if value <= node.element {
                    node.left.add(value);
                } else {
                    node.right.add(value);
                }
        }
    }
}

struct TreeIter<'a, T> {
    // A stack of references to tree nodes. Since we use `Vec`'s
    // `push` and `pop` methods, the top of the stack in the end of the
    // vector.
    //
    // The node the iterator will visit next is at the top of the stack,
    // with those ancestors still unvisited below it. If the stack is empty,
    // the iteration is over.
    unvisited: Vec<&'a TreeNode<T>>
}

impl<'a, T: 'a> TreeIter<'a, T>  {
    fn push_left_edge(&mut self, mut tree: &'a BinaryTree<T>) {
        while let NonEmpty(ref node) = *tree {
            self.unvisited.push(node);
            tree = &node.left;
        }
    }
}

impl<T> BinaryTree<T> {
    fn iter(&self) -> TreeIter<T> {
        let mut iter = TreeIter{ unvisited: Vec::new() };
        iter.push_left_edge(self);
        iter
    }
}

impl<'a, T: 'a> IntoIterator for &'a BinaryTree<T> {
    type Item = &'a T;
    type IntoIter = TreeIter<'a, T>;

    fn into_iter(self) -> Self::IntoIter {
        self.iter()
    }
}

impl<'a, T> Iterator for TreeIter<'a, T> {
    type Item = &'a T;

    fn next(&mut self) -> Option<&'a T> {
        // Find the node this iteration must produce,
        // or finish the iteration. (Use the `?` operator
        // to return immediately if it's `None`.)
        let node = self.unvisited.pop()?;

        // After `node`, the next thing we produce must be the leftmost
        // child in `node`'s right subtree, so push the path from here
        // down. Our helper method turns out to be just what we need.
        self.push_left_edge(&node.right);

        // Produce a reference to this node's value.
        Some(&node.element)
    }
}
fn main() {
    // Build a small tree.
    let mut tree = BinaryTree::Empty;
    tree.add("jaeger");
    tree.add("robot");
    tree.add("droid");
    tree.add("mecha");

    // Iterate over it.
    let mut v = Vec::new();
    for kind in &tree {
        v.push(*kind);
    }
    assert_eq!(v, ["droid", "jaeger", "mecha", "robot"]);

    assert_eq!(tree.iter()
                .map(|name| format!("mega-{}", name))
                .collect::<Vec<_>>(),
                vec!["mega-droid", "mega-jaeger",
                     "mega-mecha", "mega-robot"]);
}
```
![image-20220721130532202](/img/2022-07-05-Programming-Rust/image-20220721130532202.png)

## Chapter 19. Concurrency

惯用的多线程代码写法：

* 一个**后台线程**只负责一件事，而且周期性“醒来”去做这件事。
* 通用**线程池**通过**任务队列**与客户端通信。
* **管道**将数据从一个线程导入到另一个线程，每个线程只做一小部分工作。
* **数据并行**假设（不管正确与否）整个计算机主要用于一项大型计算，这个大型计算进而又拆分成n个小任务，在n个线程上执行，希望所有n个机器的核心同时工作。
* **同步对象海**中多个线程拥有同一个数据权限，使用基于互斥量等低级原语的临时锁方案避免争用。
* **原子整数操作**允许多核心通过一个机器字大小的字段传递信息而实现通信。（除非要交换的数据就是整数值，否则这种方法比其他手段更难以保证正确。这通常意味着传递指针。）

### Fork-Join Parallelism

![image-20220806202715909](/img/2022-07-05-Programming-Rust/image-20220806202715909.png)

Fork-join parallelism 有如下优点：

* 非常简单。
* 避免瓶颈。
* 性能计算直观。
* 容易推断程序是否正确。

Fork-join 的主要缺点是要求工作单元隔离。

#### spawn and join

`std::thread::spawn` 创建一个新的线程：

```rust
use std::thread;

thread::spawn(|| {
  println!("hello from a child thread");
});
```

它接收一个参数，一个 FnOnce 闭包或者函数。

用 `spawn` 来实现前面的 `process_file` 函数的并行版：

```rust
use std::{thread, io};

fn process_file_in_parallel(filename: Vec<String>) -> io::Result<()> {
  // Divide the work into several chunks.
  const NTHREADS: usize = 8;
  let worklists = split_vec_into_chunks(filename, NTHREADS);
  
  // Fork: Spawn a thread to handle each chunck.
  let mut thread_handles = vec![];
  for worklist in worklists {
    thread_handles.push(
      // worklist move 进闭包中。
      // spawn move 闭包到子线程中，当然也包括 worklist vec。
      thread::spawn(move || process_files(worklist))
    );
  }
  
  // Join: Wait for all threads to finish.
  // 每一个 handle 是一个 JoinHandle。
  for handle in thread_handles {
    handle.join().unwrap()?;
  }
  
  // 循环结束，8个子线程都已经成功完成。
  Ok(())
}
```

`spawn` 函数定义：

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T> where
    F: FnOnce() -> T,
    F: Send + 'static,
    T: Send + 'static, 
```

因此 `thread::spawn(move || process_files(worklist))` 会返回 `JoinHandle` ，并将它们放进一个 vector 中。

#### Error Handling Across Threads

```rust
handle.join().unwrap()?;
```

这个`.join()`方法做了两件事情。

首先，`handle.join()`返回一个 `std::thread::Result`。

其次，`handle.join()` 把子线程返回的值传给了父线程。`handle.join()` 返回的完整类型是`std::thread::Result<std::io::Result<()>>`。其中`thread::Result` 是 `spawn/join` API的一部分，而 `io::Result` 是我们应用的相关类型。

在Rust中，错误是一种 Result 值（数据）而不是异常（控制流）。可以像任何值一样跨线程传送它们。

#### Sharing Immutable Data Across Threads

```rust
// before
fn process_files(filename: Vec<String>)

// after
fn process_files(filename: Vec<String>, glossary: &GigabyteMap)
```

如果像之前那样传进线程会报错。

```rust
fn process_file_in_parallel(filename: Vec<String>, glossary: &GigabyteMap) -> io::Result<()> {
  ...
  for worklist in worklists {
    thread_handles.push(
      thread::spawn(move || process_files(worklist, glossary)) // error
    );
  }
  ...
}
```

会抱怨 `spawn` 中闭包的生命周期不是 `'static`。`spawn` 会启动一个独立的线程。Rust无法知道一个子线程会允许多长时间，因此它假设一种最坏的情况，即子线程可能会在父线程已经完成且父线程中所有的值都消失之后继续运行。

可以使用 `Arc` 来解决这个问题：

```rust
use std::sync::Arc;

fn process_file_in_parallel(filename: Vec<String>, glossary: Arc<GigabyteMap>) -> io::Result<()> {
  ...
  for worklist in worklists {
    // This call to .clone() only clones the Arc and bumps the
    // reference count. It does not clone the GigabyteMap.
    let glossary_for_child = glossary.clone();
    thread_handles.push(
      thread::spawn(move || process_files(worklist, &glossary_for_child))
    );
  }
  ...
}
```

调用`glossary.clone()`后，会创建`Arc`智能指针而不是整个 `GigabyteMap` 的一个副本。这相当于增加一次引用计数。

这样修改之后，程序就可以编译通过并运行了。因为它不再依赖引用的生命周期。只要有任何线程拥有 `Arc<GigabyteMap>` ，映射就不会释放，即使父线程早就退出了。因为 `Arc` 中的数据是不可修改的，所以也不会出现任何数据争用。

#### Rayon

Rayon 库：

```rust
use rayon::prelude::*;

// "do 2 things in parallel"
let (v1, v2) = rayon::join(fn1, fn2);

// "do N things in parallel"
giant_vector.par_iter().for_each(|value| {
  do_thing_with_value(value);
});
```

`rayon::join(fn1, fn2)` 就是调用两个函数并返回两个结果。而`.par_iter()` 方法会创建一个 `ParallelIterator` ，这个值有 map、filter和其他方法，非常类似于Rust的Iterator。

下图展示了两种理解 `giant_vector.par_iter().for_each(...)` 调用的方式。(a) 表面上看，Rayon会为向量中的每个元素都启动一个线程。(b)在后台，Rayon会让每个工作线程对应一个CPU核心，这样效率更高。这个工作线程池由程序的所有线程共享。在同时数千个任务时，Rayon会自动拆分工作。

![image-20220807082831278](/img/2022-07-05-Programming-Rust/image-20220807082831278.png)

用Rayon写`process_files_in_parallel`。

```rust
use rayon::prelude::*;

fn process_files_in_parallel(filenames: Vec<String>, glossary: &GiabyteMap) -> io::Result<()>
{
  filenames.par_iter()
  	.map(|filename| process_file(filename, glossary))
  	.reduce_with(|r1, r2| {
      if r1.is_err() { r1 } else { r2 } 
  	})
  	.unwrap_or(Ok(()))
}
```

* First, we use filenames.par_iter() to create a parallel iterator.

* `.map()` 对每个文件名调用 `process_file`。这样会得到 `io::Result<()>`值的一个 ParallelIterator。

* 然后用 .reduce_with() 组合结果。在这里，我们保留第一个错误（如果有的话），然后丢弃其他错误。

  The .reduce_with() method is also handy when you pass a .map() closure that returns a useful value on success. Then you can pass .reduce_with() a closure that knows how to combine two success results. 

* reduce_with returns an Option that is None only if filenames was empty. We use the Option’s .unwrap_or() method to make the result Ok(()) in that case.

### Channels

A *channel* is a one-way conduit for sending values from one thread to another. In other words, it’s a thread-safe queue.

![image-20220807090828202](/img/2022-07-05-Programming-Rust/image-20220807090828202.png)

图片中应该有误，下面一个线程应该是 thread 2。

Rust channels are faster than Unix pipes. Sending a value moves it rather than copying it, and moves are fast even when you’re moving data structures that contain many megabytes of data.

#### Sending Values

![image-20220807092647679](/img/2022-07-05-Programming-Rust/image-20220807092647679.png)

The code to start our file-reading thread looks like this:

```rust
use std::{fs, thread};
use std::sync::mpsc;

let (sender, receiver) = mpsc::channel();

let handle = thread::spawn(move || {
  for filename in documents {
    let text = fs::read_to_string(filename)?;
    
    // 将 text 值 move 进 channel，
    // 最终，它会再次 move 到接收到这个值的地方。
    if sender.send(text).is_err() {
      break;
    }
  }
  Ok(())
});
```

Whether text contains 10 lines of text or 10 megabytes, this operation copies three machine words (the size of a String struct), and t**he corresponding receiver.recv() call will also copy three machine words**.

**The send and recv methods both return Results, but these methods fail only if the other end of the channel has been dropped.** 

In our code, sender.send(text) will fail only if the receiver’s thread has exited early. This is typical for code that uses channels. Whether that happened deliberately or due to an error, it’s OK for our reader thread to quietly shut itself down.

```rust
fn start_file_reader_thread(documents: Vec<PathBuf>)
	-> (mpsc::Receiver<String>, thread::JoinHandle<io::Result>)
{
  let (sender, receiver) = mpsc::channel();
  
  let handle = thread::spawn(move || {
    ...
  });
  
  (receiver, handle)
}
```

#### Receiving Values

```rust
while let Ok(text) = receiver.recv() {
  do_something_with(text);
}
```

等价于

```rust
for text in receiver {
  do_something_with(text);
}
```

The loop will exit normally when the channel is empty and the Sender has been dropped. 

```rust
fn start_file_indexing_thread(texts: mpsc::Receiver<String>)
	-> (mpsc::Receiver<InMemoryIndex>, thread::JoinHandle<()>)
{
  let (sender, receiver) = mpsc::channel();
  
  let handle = thread::spawn(move || {
    for (doc_id, text) in texts.into_iter().enumerate() {
      let index = InMemoryIndex::from_single_document(doc_id, text);
      if sender.send(index).is_err() {
        break;
      }
    }
  });
  
  (receiver, handle)
}
```

#### Running the Pipeline

Stage3:

```rust
fn start_in_memory_merge_thread(file_indexs: mpsc::Receiver<InMemoryIndex>)
	-> (mpsc::Receiver<InMemoryIndex>, thread::JoinHandle<()>)
```

Stage 4:

```rust
fn start_index_writer_thread(big_indexs: mpsc::Receiver<InMemoryIndex>, output_dir: &Path)
	-> (mpsc::Receiver<PathBuf>, thread::JoinHandle<io::Result<()>>)
```

Stage 5:

```rust
fn merge_index_files(files: mpsc::Receiver<PathBuf>, output_dir: &Path)
	-> io::Result<()>
```

最终运行代码：

```rust 
fn run_pipeline(documents: Vec<PathBuf>, output_dir: PathBuf)
	-> io::Result<()>
{
  // Launch all five stages of the pipeline.
  let (texts, h1) = start_file_reader_thread(documents);
  let (pints, h2) = start_file_indexing_thread(texts);
  let (gallons, h3) = start_in_memory_merge_thread(pints);
  let (files, h4) = start_index_writer_thread(gallons, &output_dir); 
  let result = merge_index_files(files, &output_dir);
 
  // Wait for threads to finish, holding on to any errors that they encounter.
  let r1 = h1.join().unwrap(); 
  h2.join().unwrap(); 
  h3.join().unwrap();
  let r4 = h4.join().unwrap();
  
  // Return the first error encountered, if any.
  // (As it happens, h2 and h3 can't fail: those threads 
  // are pure in-memory data processing.)
  r1?;
  r4?;
  result
}
```

#### Channel Features and Performance

`std::sync::mpsc` 就是 "multiproducer, single-consumer"。

![image-20220807101845246](/img/2022-07-05-Programming-Rust/image-20220807101845246.png)

`Sender<T>` implements the Clone trait. To get a channel with multiple senders, simply create a regular channel and clone the sender as many times as you like. You can move each Sender value to a different thread.

A `Receiver<T>` can’t be cloned, so if you need to have multiple threads receiving values from the same channel, you need a `Mutex`. 

Rust通道是经过认真优化的。在刚创建通道时，Rust使用“一次性”队列实现。如果只是用这个通道发送一个对象，那可以保证开销最小。如果再发送第二个值，Rust则会切换到一个不同的队列实现。这个实现会从长远考虑，准备让通道传输很多值，同时又保持分配开销最小化。如果你选择克隆 Sender ，Rust则必须回退到另外一个实现，该实现可以保证多个线程同时发送值时的安全。不过即使是这3个实现中最慢的实现也是没有锁的队列，因此发送和接收值最多只是几个原子操作，涉及一次堆内存分配，外加转移自身。只有在队列为空且接收线程需要休眠时才需要系统调用。当然，此时经过通道的流量无论如何也不是最大的。

发送至的速度超过接收和处理值的速度。这会导致通道内部的值越积越多。Rust借用Unix管道。Unix使用 *backpressure* ，从而强迫快速发送端放慢速度。Unix系统的每个管道都有固定大小，如果一个进程尝试向随时可能满的管道写入数据，系统就会直接阻塞该进程，直至管道中有了空间。Rust中的等价机制加 *synchronous channel*:

```rust
use std::sync::mpsc;

let (sender, receiver) = mpsc::sync_channel(1000);
```

同步通道就像常规通道一样，只是在创建时需要指定它可以保存多少值。对于同步通道而言，`sender.send(value)` 是一个潜在的阻塞操作。

#### Thread Safety: Send and Sync

This is mostly true, but Rust’s full thread safety story hinges on two built-in traits, `std::marker::Send` and `std::marker::Sync`.

* Types that implement `Send` are safe to pass by value to another thread. **They can be moved across threads**.
* Types that implement `Sync` are safe to pass by **non-mut reference to another thread**. They can be shared across threads.

By *safe* here, we mean the same thing we always mean: free from data races and other undefined behavior.

![image-20220807104741772](/img/2022-07-05-Programming-Rust/image-20220807104741772.png)

The few types that are neither Send nor Sync are mostly those that use mutability in a way that isn’t thread-safe. For example, consider `std::rc::Rc<T>`, the type of reference-counting smart pointers.

#### Piping Almost Any Iterator to a Channel

统一迭代器管道和线程管道。

```rust
documents.into_iter()
  .map(read_whole_file)
  .errors_to(error_sender) // filter out error results
  .off_thread()            // spawn a thread for the above work
  .map(make_single_file_index)
  .off_thread()             // spawn another thread for stage 2 
  ...
```

可以定义一个 trait。

```rust
use std::sync::mpsc;

pub trait OffThreadExt: Iterator {
  /// Transform this iterator into an off-thread iterator: the
  /// `next()` calls happen to a separate worker thread, so the
  /// iterator and the body of your loop run concurrently.
  fn off_thread(self) -> mpsc::IntoIter<Self::Item>;
}
```

然后为这个迭代器实现这个 trait。

```rust
use std::thread;

impl<T> OffThreadExt for T
where T: Iterator + Send + 'static,
      T::Item: Send + 'static
{
  fn off_thread(self) -> mpsc::IntoIter<Self::Item> {
    // Create a channel to transfer items from the worker thread.
    let (sender, receiver) = mpsc::sync_channel(1024);

    // Move this iterator to a new worker thread and run it there.
    thread::spawn(move || { 
      for item in self {
        if sender.send(item).is_err() { 
          break;
        } 
      }
    });
    // Return an iterator that pulls values from the channel.
    receiver.into_iter()
  }
}
```

#### Beyond Pipelines
通道不仅在管道中有用，它们也是在相同进程中为其他线程提供异步服务的快速而简单的方式。

通道也适用于一个线程向另一个线程发送请求并期待得到某种响应的情形。

### Shared Mutable State
#### What Is a Mutex?
**互斥量**（或者叫锁）用于强制多线程依次访问特定的数据。

互斥量保护数据。互斥量的作用体现在以下几方面。

* 防止 data races，即避免多个线程并发读写同一块内存。
* 即使没有数据争用，即使所有读写在程序中都是顺序执行，如果没有互斥量，不同线程的操作也可能以任意方式相互交错。
* 互斥量支持通过 invariant 编程，即受保护数据由你负责初始化但每个临界区来维护的规则。

#### `Mutex<T>`

因为等待列表既是共享的也是可修改的，所以必须由一个 `Mutex` 来提供保护：

```rust
use std::sync::Mutex;

/// All threads have shared access to this big contex struct.
struct FernEmpireApp {
  ...
  waiting_list: Mutex<WaitingList>,
  ...
}
```

创建 `Mutex` 代码：

```rust
use std::sync::Arc;

let app = Arc::new(FernEmpireApp {
  ...
  waiting_list: Mutex::new(vec![]),
  ...
});
```

创建一个新的 `Mutex` 就像创建一个新 `Box` 或 `Arc`，但 `Box` 和 `Arc` 都意味着堆分配，而 `Mutex` 就是单纯的一种锁。如果想把 `Mutex` 分配在堆上，则必须明确地表示出来，就像这里使用 `Arc::new` 创建整个应用，而使用 `Mutex::new` 只是为了保护数据一样。这两个类型经常一块使用， `Arc` 方便跨线程共享数据，而 `Mutex` 方便跨线程共享可修改数据。

使用 `Mutex` ：

```rust
impl FernEmpireApp {
  /// Add a player to the waiting list for the next game.
  /// Start a new game immediately if enough players are waiting.
  fn join_waiting_list(&self, player: PlayerId) {
    // Lock the mutex and gain access to the data inside.
    // The scope of `guard` is a critical section.
    let mut guard = self.waiting_list.lock().unwrap();
    
    // Now do the game logic.
    guard.push(player);
    if guard.len() == GAME_SIZE {
      let players = guard.split_off(0);
      self.start_game(players);
    }
  }
}
```

取得数据的唯一方法是调用 `.lock()` 方法：

```rust
let mut guard = self.waiting_list.lock().unwrap();
```

`self.waiting_list.lock()` 会一直阻塞到可以再次获得互斥量。这个方法调用返回的 `MutexGuard<WaitingList>` 值是对 `&mut WaitingList` 的一个简单封装。借助 Deref 类型转换，可以直接在这个 guard 上调用 WaitingList 方法：

```rust
guard.push(player);
```

这个 guard 甚至还允许我们直接引用底层数据。Rust的生命周期系统保证这些引用的寿命比不会超出 guard 自身。如果没有拿到锁，则不可能在 Mutex 中访问数据。

在 guard 被清除后，锁也会被释放。通常这会在阻塞结束时发生，但也可以手工清除：

```rust
if guard.len() == GAME_SIZE {
  let players = guard.split_off(0);
  drop(guard); // don't keep the list locked while starting a game
  self.start_game(players);
}
```

#### mut and Mutex

在Rust中，`&mut` 意味着 *exclusive access*。Plain & means *shared access*.

But Mutex does have a way: the lock. In fact, a mutex is little more than a way to do exactly this, to provide *exclusive* (mut) access to the data inside, even though many threads may have *shared* (non-mut) access to the Mutex itself.

Rust’s type system is telling us what Mutex does. It dynamically enforces exclusive access, something that’s usually done statically, at compile time, by the Rust compiler.

(You may recall that `std::cell::RefCell` does the same, except without trying to support multiple threads. `Mutex` and `RefCell` are both flavors of **interior mutability**, which we covered .)

#### Why Mutexes Are Not Always a Good Idea

However, threads that use mutexes are subject to some other problems that Rust doesn’t fix for you:

* Valid Rust programs can’t have data races, but they can still have other *race conditions*— situations where a program’s behavior depends on timing among threads and may therefore vary from run to run. Some race conditions are benign. Some manifest as general flakiness and incredibly hard-to-fix bugs. Using mutexes in an unstructured way invites race conditions. It’s up to you to make sure they’re benign.
* Shared mutable state also affects program design. Where channels serve as an abstraction boundary in your code, making it easy to separate isolated components for testing, mutexes encourage a “just-add-a-method” way of working that can lead to a monolithic blob of interrelated code.
* Lastly, mutexes are just not as simple as they seem at first, as the next two sections will show.

All of these problems are inherent in the tools. Use a more structured approach when you can; use a Mutex when you must.

#### Deadlock

```rust
let mut guard1 = self.waiting_list.lock().unwrap();
let mut guard2 = self.waiting_list.lock().unwrap(); // deadlock
```

Suppose the first call to self.waiting_list.lock() succeeds, taking the lock. The second call sees that the lock is held, so it blocks, waiting for it to be released. It will be waiting forever. The waiting thread is the one that’s holding the lock.

To put it another way, the lock in a Mutex is not a recursive lock.

Rust’s borrow system can’t protect you from deadlock. The best protection is to keep critical sections small: get in, do your work, and get out.

It’s also possible to get deadlock with channels. 

#### Poisoned Mutexes

If a thread panics while holding a Mutex, Rust marks the Mutex as *poisoned.* Any subsequent attempt to lock the poisoned Mutex will get an error result. Our `.unwrap()` call tells Rust to panic if that happens, propagating panic from the other thread to this one.

#### **Multiconsumer Channels Using Mutexes**
```rust
pub mod shared_channel {
  use std::sync::{Arc, Mutex};
  use std::sync::mpsc::{channel, Sender, Receiver};
  
  /// A thread-safe wrapper around a `Receiver`.
  #[derive(Clone)]
  pub struct SharedReceiver<T>(Arc<Mutex<Receiver<T>>>); 
  
  impl<T> Iterator for SharedReceiver<T> {
    type Item = T;
    
    /// Get the next item from the wrapped receiver.
    fn next(&mut self) -> Option<T> {
      let guard = self.0.lock().unwrap(); 
      guard.recv().ok()
    } 
  }

  /// Create a new channel whose receiver can be shared across threads. 
  /// This returns a sender and a receiver, just like the stdlib's
  /// `channel()`, and sometimes works as a drop-in replacement.
  pub fn shared_channel<T>() -> (Sender<T>, SharedReceiver<T>) {
    let (sender, receiver) = channel();
    (sender, SharedReceiver(Arc::new(Mutex::new(receiver))))
  }
}
```

![image-20220807141745840](/img/2022-07-05-Programming-Rust/image-20220807141745840.png)

#### Read/Write Locks(`RwLock<T>`)

Whereas a mutex has a single lock method, a read/write lock has two locking methods, read and write. The RwLock::write method is like Mutex::lock. It waits for exclusive, mut access to the protected data. The RwLock::read method provides non-mut access, with the advantage that it is less likely to have to wait, because many threads can safely read at once. With a mutex, at any given moment, the protected data has only one reader or writer (or none). With a read/write lock, it can have either one writer or many readers, much like Rust references generally.

#### Condition Variables (Condvar)

In Rust, the std::sync::Condvar type implements condition variables. A Condvar has methods .wait() and .notify_all(); .wait() blocks until some other thread calls .notify_all().

#### **Atomics**

The std::sync::atomic module contains atomic types for lock-free concurrent programming. These types are basically the same as Standard C++ atomics, with some extras:

AtomicIsize and AtomicUsize are shared integer types corresponding to the single- threaded isize and usize types.

AtomicI8, AtomicI16, AtomicI32, AtomicI64, and their unsigned variants like AtomicU8 are shared integer types that correspond to the single-threaded types i8, i16, etc.

An AtomicBool is a shared bool value.

An `AtomicPtr<T>` is a shared value of the unsafe pointer type *mut T.

#### **Global Variables**

The simplest way to support incrementing PACKETS_SERVED, while keeping it thread-safe, is to make it an atomic integer:

```rust
use std::sync::atomic::AtomicUsize;

static PACKETS_SERVED: AtomicUsize = AtomicUsize::new(0);
```

Once this static is declared, incrementing the packet count is straightforward:

```rust
use std::sync::atomic::Ordering; 

PACKETS_SERVED.fetch_add(1, Ordering::SeqCst);
```

Atomic globals are limited to simple integers and Booleans. Still, creating a global variable of any other type amounts to solving two problems.

First, the variable must be made thread-safe somehow, because otherwise it can’t be global: for safety, static variables must be both Sync and non-mut. Fortunately, we’ve already seen the solution for this problem. Rust has types for safely sharing values that change: Mutex, RwLock, and the atomic types. These types can be modified even when declared as non-mut. It’s what they do. (See “mut and Mutex”.)

Second, static initializers can only call functions specifically marked as const, which the compiler can evaluate during compile time. Put another way, their output is deterministic; it depends only on their arguments, not any other state or I/O. That way, the compiler can embed the results of that computation as a compile-time constant. This is similar to C++ constexpr.

 Rust limits what const functions can do to a small set of operations, which are enough to be useful while still not allowing any nondeterministic results. const functions can’t take types as generic arguments, only lifetimes, and it’s not possible to allocate memory or operate on raw pointers, even in unsafe blocks. We can, however, use arithmetic operations (including wrapping and saturating arithmetic), logical operations that don’t short-circuit, and other const functions. For example, we can create convenience functions to make defining statics and consts easier and reduce code duplication:

```rust
const fn mono_to_rgba(level: u8) -> Color { 
  Color {
    red: level, 
    green: level, 
    blue: level, 
    alpha: 0xFF
  } 
}
const WHITE: Color = mono_to_rgba(255); 
const BLACK: Color = mono_to_rgba(000);
```

Combining these techniques, we might be tempted to write:

```rust
static HOSTNAME: Mutex<String> =
  Mutex::new(String::new());  // error: calls in statics are limited to
                              // constant functions, tuple structs, and
                              // tuple variants
```

Unfortunately, while AtomicUsize::new() and String::new() are const fn, Mutex::new() is not. In order to get around these limitations, we need to use the lazy_static crate.

We can declare a global Mutex-controlled HashMap with lazy_static like this:

```rust
use lazy_static::lazy_static;
use std::sync::Mutex;

lazy_static! {
	static ref HOSTNAME: Mutex<String> = Mutex::new(String::new());
}
```

Using lazy_static! imposes a tiny performance cost on each access to the static data. The implementation uses std::sync::Once, a low-level synchronization primitive designed for one-time initialization. Behind the scenes, each time a lazy static is accessed, the program executes an atomic load instruction to check that initialization has already occurred. (Once is rather special purpose, so we will not cover it in detail here. It is usually more convenient to use lazy_static! instead. However, it is handy for initializing non-Rust libraries; for an example, see “A Safe Interface to libgit2”.)

### **What Hacking Concurrent Code in Rust Is Like**

Rust insists on safety, so from the moment you decide to write a multithreaded program, the focus is on building safe, structured communication. Keeping threads mostly isolated is a good way to convince Rust that what you’re doing is safe. It happens that isolation is also a good way to make sure what you’re doing is correct and maintainable. Again, Rust guides you toward good programs.

## **Chapter 20. Asynchronous Programming**

Asynchronous tasks are similar to threads, but are much quicker to create, pass control amongst themselves more efficiently, and have memory overhead an order of magnitude less than that of a thread. 

### **From Synchronous to Asynchronous**

![image-20220817164154643](/img/2022-07-05-Programming-Rust/image-20220817164154643.png)

文中举了同步的获得HTTP请求再返回的例子，可以看到大部分时间都花在等待上了。

#### **Futures**

`std::future::Future`

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

```rust
pub enum Poll<T> {
    /// Represents that a value is immediately ready.
    #[lang = "Ready"]
    #[stable(feature = "futures_api", since = "1.36.0")]
    Ready(#[stable(feature = "futures_api", since = "1.36.0")] T),

    /// Represents that a value is not ready yet.
    ///
    /// When a function returns `Pending`, the function *must* also
    /// ensure that the current task is scheduled to be awoken when
    /// progress can be made.
    #[lang = "Pending"]
    #[stable(feature = "futures_api", since = "1.36.0")]
    Pending,
}
```

A Future represents an operation that you can test for completion. A future’s poll method never waits for the operation to finish: it always returns immediately. If the operation is complete, poll returns Poll::Ready(output), where output is its final result. Otherwise, it returns Pending. If and when the future is worth polling again, it promises to let us know by invoking a *waker*, a callback function supplied in the Context. We call this the “piñata model” of asynchronous programming: the only thing you can do with a future is whack it with a poll until a value falls out.



## Chapter 21. Macros

在编译期间，在类型检查和机器码生成之前，宏会进行展开。联系rust编译过程图理解。

### Macro Basics

* 用 `macro_rules!` 定义的声明宏，通过模式匹配来工作。
* 在 pattern 或者 template 中，可以用方括号或者花括号代替圆括号。它们没有什么区别。

```rust
assert_eq!(gcd(6, 10), 2);
assert_eq![gcd(6, 10), 2)];
assert_eq!{gcd(6, 10), 2}			// 注意结尾这里分号是可选的，也可以加上
```

* 对于宏的调用也是一样的。按照惯例，调用 `assert_eq!` 用圆括号，调用 `vec!` 用方括号，对于 `macro_rules!` 用花括号。

#### Basics of Macro Expansion

* 在编译期间，Rust展开宏非常早。在宏定义之前是不能调用它的，因为Rust会展开每一个宏调用，这发生在查看程序剩余代码之前。
* 正则表达式是在字符集上操作，而 pattern 是在 token 上操作。正则表达式和宏模式另一个重要的区别是在Rust中圆括号、块总是成对出现的。在宏展开之前就会进行检查。
* 注解和空白不是 token ，所以不会影响模式匹配。
* 一般可能会有一些错误，在参数部分用 `$left:expr` 而不是 `$left`。Rust不会立刻发现这个错误，它会将 `$left` 当作一个替换，直到调用这个宏的时候会发生错误。

#### Unintended Consenquences

正确的 `assert_eq!` 宏的定义是这样的：

```rust
// 标准库中的定义
macro_rules! assert_eq {
    ($left:expr, $right:expr $(,)?) => {
        match (&$left, &$right) {				// 这里用的是引用
            (left_val, right_val) => {
                if !(*left_val == *right_val) {
                    let kind = $crate::panicking::AssertKind::Eq;
                    // The reborrows below are intentional. Without them, the stack slot for the
                    // borrow is initialized even before the values are compared, leading to a
                    // noticeable slow down.
                    $crate::panicking::assert_failed(kind, &*left_val, &*right_val, $crate::option::Option::None);
                }
            }
        }
    };
}
```

为什么不这样写呢？

```rust
if !($left == $right) {
  panic!("assertion failed: `(left == right)` \
    				(left: `{:?}`, right: `{:?}`)", $left, $right)
}
```

如果 `assert_eq!(letters.pop(), Some('z'))` 这样调用，由于 `letters.pop()` 会从一个 vector 中移除一个值，那么当第二次调用的时候就会产生一个不同的值，这也就是为什么实际宏当中 `$left` 和 `$right` 只会保存一次它们的值。



那为什么宏里面要用引用，不能这样写吗？

```rust
macro_rules! bad_assert_eq {
    ($left:expr, $right:expr) => {
        match ($left, $right) {		// 这里不用引用
            (left_val, right_val) => {
                if !(left_val == right_val) {
                    panic!("assertion failed" /* ... */);
                }
            }
        }
    };
}
```

如果传入的参数是 `String` ，那么就会移动所有权到变量里面。因此这里要使用引用。

> In short, macros can do surprising things.

#### Repetition

标准库中 `vec!` 宏

```rust
macro_rules! vec {
    () => (
        $crate::__rust_force_expr!($crate::vec::Vec::new())
    );
    ($elem:expr; $n:expr) => (
        $crate::__rust_force_expr!($crate::vec::from_elem($elem, $n))
    );
    ($($x:expr),+ $(,)?) => (
        $crate::__rust_force_expr!(<[_]>::into_vec(box [$($x),+]))
    );
}
```

* `$( PATTERN ), *` 用来匹配任何用 `,` 分隔的立标，在列表中每个匹配一个 `PATTERN`。

| Pattern    | Meaning                                        |
| ---------- | ---------------------------------------------- |
| $( ... )*  | Match 0 or more times with no separator        |
| $( ... ),* | Match 0 or more times, separated by commas     |
| $( ... );* | Match 0 or more times, separated by semicolons |
| $( ... )+  | Match 1 or more times with no separator        |
| $( ... ),+ | Match 1 or more times, separated by commas     |
| $( ... );+ | Match 1 or more times, separated by semicolons |
| $( ... )?  | Match 0 or 1 times with no separator           |
| $( ... ),? | Match 0 or 1 times, separated by commas        |
| $( ... );? | Match 0 or 1 times, separated by semicolons    |

总结一下啦：

* `*` 匹配0次或多次
* `+` 至少匹配一次
* `?` 最多匹配一次

理解理解这个复杂的鬼东西

```rust
<[_]>::into_vec(box [$($x),+])
```

* 创建一个 boxed 数组然后用 `[T]::into_vec` 方法将 boxed 数组转换成一个 vector 。
* `<[_]>` 是一个非常规的写法，用来表示一些类型的切片（"slice of something"）。像 `fn()`,`&str`,或者`[_]`，没有明确的类型，就必须包在尖括号`<>`里面。

### Built-In Macros

下面的这些宏是内置的，硬编码在 `rustc` 中。

* `file!()`, `line!()`, `column!()`
* `stringingfy!(...tokens...)`
* `concat!(str0, str1, ...)`

## Chapter 22. Unsafe Code



