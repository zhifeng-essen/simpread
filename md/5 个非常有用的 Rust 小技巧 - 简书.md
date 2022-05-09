> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/a76730f8218e)

自从做了 [https://github.com/chainx-org/ChainX](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fchainx-org%2FChainX)  
项目以后，主力语言就转到了 Rust，今天刚好这个文章，比较剪短，跟大家分享一下。

在开始之前，跟大家简单介绍 ChainX 项目。ChainX 是一个基于 [substrate](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fparitytech%2Fsubstrate) 专注于区块链资产跨链项目，目前已经实现了 BTC 跨链, 可以在我们的测试网进行充值体验，如何参与测试网请[点击这里](https://links.jianshu.com/go?to=%255Bhttps%3A%2F%2Fgithub.com%2Fchainx-org%2FChainX%2Fwiki%2FTestnet%255D%28https%3A%2F%2Fgithub.com%2Fchainx-org%2FChainX%2Fwiki%2FTestnet%29)。

我们将会在最近上线主网，并进行开源，欢迎有识之士进行关注，项目地址是： [](https://links.jianshu.com/go?to=%255Bhttps%3A%2F%2Fgithub.com%2Fchainx-org%2FChainX%255D%28https%3A%2F%2Fgithub.com%2Fchainx-org%2FChainX%29)[https://github.com/chainx-org/ChainX](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fchainx-org%2FChainX)  
。此外，也欢迎开发者加入我们的开发群，有兴趣的可以私信我加群。

ChainX 也在不断招人，运营，技术都招，如果有任何想法，欢迎与我联系，期待大牛加入！

好了，开始今天的 “正题”:

更 “护眼” 的 print 调试
-----------------

当我们用 print 大法进行调试的时候，经常会用到 `:?` 格式化操作符。但是除此以外，还有另外一些非常好用的操作符！另一个非常有用的就是 `:#?`，它会自动加入换行和缩进来增强输出的可读性。

```
#[derive(Debug)]
struct Foo {
    x: i32,
    y: i32,
}

let foo = Foo { x: 1, y: 2 };

println!("Simple debug:\n{:?}", foo);
println!("Pretty debug:\n{:#?}", foo);
```

```
Simple debug: 
Foo { x: 1, y: 2 }

Pretty debug: 
Foo {
  x: 1,
  y: 2,
}
```

关于调试，还可以了解一下最近新加的 `dbg!` 宏 [https://doc.rust-lang.org/std/macro.dbg.html](https://links.jianshu.com/go?to=https%3A%2F%2Fdoc.rust-lang.org%2Fstd%2Fmacro.dbg.html) 。

`unimplemented!`
----------------

有时候，你可能会想要一个不用进行完整实现的函数。比如，你可能想要一些方法的测试，又或者你想要为以后的开发保留某个 feature，这时 `unimplemented!` 就会派上用场。如果想要的类型是什么，`unimplemented!` 都会被展开为能够编译的表达式。

```
enum VerySimpleList<T> {
    Empty,
    Elem(T, Box<VerySimpleList>),
}

impl<T> VerySimpleList<T> {
    fn len(&self) -> usize {
        match self {
            VerySimpleList::Empty => 0,
            VerySimpleList::Elem(..) => unimplemented!(),
        }
    }
}
```

`..` 结构体字面操作符
-------------

有时候，你想要部分地复制一个结构体，也就是里面有部分字段不一样，但是其他字段保留复制结构体里面的内容。尽管你可以通过手动 clone 然后进行修改，但是还有更简单的方式！通过 `..` 操作符后面跟着这个结构体的另一个实例，剩下的字段就会用后面这个实例的字段填充。此外，它并不要求结构体实现 `Clone` 约束。

```
#[derive(Debug, Default)]
struct Foo {
    x: i32,
    y: i32,
}

let a = Foo { x: 1, y: 2 };
let b = Foo { x: 2, ..a };
let c = Foo { x: 2, ..Default::default() };
```

模式匹配 guard
----------

有时当使用模式匹配时，你所匹配的模式并没有跟你想要处理的格式完美匹配。比如，你可能会写这样的代码：

```
fn divide_opt(x: Option<i32>, y: Option<i32>) -> Option<i32> {
    match (x, y) {
        (Some(i), Some(0)) => None
        (Some(i), Some(j)) => Some(i / j)
        _ => None,
    }
}
```

你大可以把这两种情况组合起来：

```
fn divide_opt(x: Option<i32>, y: Option<i32>) -> Option<i32> {
    match (x, y) {
        (Some(i), Some(j)) => {
            if j == 0 {
                None
            } else {
                Some(i / j)
            }
        }
        _ => None,
    }
}
```

但是，还有更好的方式！模式后面可以跟着一个条件表达式，叫做 “guard":

```
fn divide_opt(x: Option<i32>, y: Option<i32>) -> Option<i32> {
    match (x, y) {
        (Some(i), Some(j)) if j != 0 => Some(i / j),
        _ => None,
    }
}
```

填充格式操作符
-------

想要在 Rust 里面进行左侧填充字符？不需要任何额外的包就可以实现！只要用 `:>` 操作符后面跟上一个长度，然后就好了！

```
let score1 = 100;
let score2 = 1000;
let score3 = 10000;

println!("{:>5}", score1);
println!("{:>5}", score2);
println!("{:>5}", score3);
```

这会打印出：

```
first player:  padded!
second player: padded!
third player:  padded!
```

等一下，还没完！如果你想要两边填充，也有办法, `:^` 后面跟上填充后的字符串宽度就行了：

```
let padded = "padded";
println!("[{:^10}]", padded)
```

这会打印出：

```
[  padded  ]
```

你知道还可以用不同的字符进行填充吗？只要在箭头前面指定填充字符就行了！

```
let title = "SCORES";

let player1 = "first player:";
let player2 = "second player:";
let player3 = "third player:";

let score1 = 100;
let score2 = 1000;
let score3 = 10000;

println!("{:_^20}", title);
println!("{:<14} {:>5}", player1, score1);
println!("{:<14} {:>5}", player2, score2);
println!("{:<14} {:>5}", player3, score3);
```

这会打印出：

```
_______SCORES_______
first player:    100
second player:  1000
third player:  10000
```

原文：[https://saghm.github.io/five-rust-things/](https://links.jianshu.com/go?to=https%3A%2F%2Fsaghm.github.io%2Ffive-rust-things%2F)