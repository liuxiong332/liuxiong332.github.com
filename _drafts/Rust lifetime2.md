Rust的生命周期是Rust中一个非常难以理解的概念，也是Rust独有的概念。这篇文章将会继续上一篇文章继续讲述Rust中的生命周期的注意事项。如果还没有阅读上一章关于[Rust生命周期基础部分](https://zhuanlan.zhihu.com/p/93193353)的请移步。

# Rust的生命周期是动态推断的

不同于Rust中的泛型参数，程序员是可以手动指定的。Rust的生命周期是不能手动指定的，需要**编译器根据传入的参数进行推断**。当编译器在某条语句上不能根据参数进行推断时，他会继续往下执行并推断生命周期参数。**编译器会持续根据语句上下文推断出生命周期参数，并选择最小的那个**。

```rust
struct Context<'a> {
    vars: Vec<&'a str>,
}

fn main() {
    let mut v = Context { vars: vec![] };
    v.vars.push("hello");
    
    println!("{:?}", v.vars);
}
```

对于上述程序片段来说，当运行`let mut v = Context { vars: vec![] }`的时候，此时`'a`并不能推断出来，于是编译器继续查看下一条语句，当看到`v.vars.push("hello")`时，编译器推断出`'a`为`'static`。

同理，当运行下列片段时：

```rust
struct Context<'a> {
    vars: Vec<&'a str>,
}

fn main() {
    let mut v = Context { vars: vec![] };
    v.vars.push("hello");               // 'a
    
    {
        let s = String::from("dd");     // 'b
        v.vars.push(&s);
    }
    println!("{:?}", v.vars);
}
```

编译器首先会在`v.vars.push("hello")`推断出`'a`为`'static`，然后当运行到里面的大括号的时候，发现`v.vars.push(&s)`，而s的生命周期为`'b`，此时编译器发现两次推断不一样，于是他会选择生命周期较小的那个`'b`。而此时v的生命周期大于`'b`，编译器会报错，提示*borrowed value does not live long enough*。

对于大型程序来说，生命周期推断往往比较复杂。当编译器报错的时候，我们要扮演一次编译器的角色来弄清楚错误究竟是什么，此时这条原则就很有用。

# 生命周期声明是入参和返回值或者结构体成员之间的一种生命周期约定和限制

这一条其实之前谈过了，这里拿出来是用来强调这一点。在我看来，很多生命周期错误都是没有好好理解生命周期表现的意义而出现的。

```rust
struct Context<'a> {
    name: &'a str,
    vars: Vec<&'a str>,
}
```

例如对于上面的结构体，我们可以看到，编程者的意思是希望`name`和`vars`是通过同一生命周期作用域引进来的。但是这确定是你想要的吗？

假设我们现在在编写一个自定义语言运行时，Context是我们的运行上下文。在不同的Context中，vars是上下文的变量名字，而name是我们为不同上下文命名的名字。当我们创建一个新的上下文时，我们会给每个Context创建一个新的临时名字。很明显，vars应该和自定义语言字符串拥有同一生命周期，因为vars应该引用那些语言字符串，而name则是我们人为加上的自定义字符串。如果name是一个临时的名字，则我们就会把vars标记为和name一样的临时生命周期。此时我们的Context将会毫无用处！我们的Context将只能在这个短暂的临时生命周期中运行！

在这种情况下，我们需要把上面的写成如下所示：

```rust
struct Context<'a, 'b> {
    name: &'a str,
    vars: Vec<&'b str>,
}
```

此时，name和vars将有不同的生命周期参数，编译器会分别推断出他们的生命周期。于是问题就解决了。

# 警惕Self的隐含生命周期参数

对于impl中的方法来说，self有一个隐含的生命周期参数。

```rust
struct A<'a> {
    name: &'a str,
}

impl<'a> A<'a> {
    fn get(&self) -> &str {
        self.name
    }
}

fn main() {
    let s = String::from("hello");
    let s_ref;
    
    {
        let a = A { name: &s };
        s_ref = a.get();
    }
    println!("{:?}", s_ref);
}
```

对于A的get方法来说，**&self有一个隐含的生命周期参数，这个生命周期就是实例化A所在的区域**。如果返回的`&str`不写生命周期参数，根据生命周期省略原则，**返回的参数将会和&self一样的生命周期**。

在上述示例中，返回的&str的生命周期明显大于self的生命周期。但是在这里返回的str将会限制在A的实例所在的生命周期内。当A的实例`a`脱离内部作用区域时，`s_ref`生命周期就结束了，也不能被引用了。

```rust
struct A<'a> {
    name: &'a str,
}

impl<'a> A<'a> {
    fn get(&self) -> &'a str {
        self.name
    }
}

fn main() {
    let s = String::from("hello");
    let s_ref;
    
    {
        let a = A { name: &s };
        s_ref = a.get();
    }
    println!("{:?}", s_ref);
}
```

正确的是我们显式声明我们的返回值的生命周期为`'a`，于是一切都正常了，此时`s_ref`的生命周期就扩充到了外部作用域，println就可以正常打印了。

有的时候，我们希望内部的引用变量和self具有相同的生命周期。此时我们需要显示声明self的生命周期参数。就像下面一样。

```rust
struct A<'a> {
    name: &'a str,
}

impl<'a> A<'a> {
    fn set(&'a self, name: &'a str) -> A<'a> {
        A { name }
    }
}
```

在&self中间加上`'a`生命周期参数，能够显示声明self和其他参数的生命周期的关系。

以上就是关于生命周期说明的全部内容，也是学习和练习的提炼和总结。