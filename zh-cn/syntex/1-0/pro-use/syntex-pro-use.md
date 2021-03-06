# Iris语言的高级用法
<span id="s1"/>
## 1. Iris的上下文环境
在进一步阐述*Iris*的闭包之前，首先需要介绍一下Iris的上下文环境的内容。

上下文环境（Context Environment）指的是任何一段*Iris*代码在执行时所处的环境，该环境包括了该环境下的局部变量、调用者等信息，它类似于*Python*中的栈帧（Frame）的概念。以下简称“上下文”。

在*Iris*中，上下文环境的存在保证了：

>1. 方法的调用
2. 类、模块、接口的定义
3. 迭代器块的传递
4. 闭包

当*Iris*初始化执行状态的时候，将自动生成一个上下文，此上下文环境称为*Main*上下文，*Main*上下文有时也被称为全局上下文，它代表了*Iris*执行时的最基本的上下文。

换言之，在*Iris*中，所有**不在方法体、类体、模块体、接口体以及闭包中执行的环境统称Main上下文**。

*Iris*的任何局部变量都会被注册到当前的上下文中**（如果在*Main*环境下，实例变量、类变量也会被注册到当前上下文中；而在类、模块的上下文中，实例变量会定义到当前上下文中，类变量会直接定义到类、模块中，而接口中禁止定义任何变量）**。除此之外，上下文还会携带一些其他的解释器执行过程中需要的信息。

*Iris*的上下文切换保证了方法调用与闭包等功能的实现，简单来讲，以下几种情形会存在上下文切换：

>1. 调用方法的时候；
2. 创建或修改类、模块以及接口的时候；
3. 创建闭包的时候。

分别以以上三种情况进行说明：

### 1. 调用方法的时候
当方法被调用的时候，**如果调用方处于*Main*环境下、且没有显式的调用者，那么*Iris*认为方法的调用者为一个特殊的*Main*对象**，该对象不对外公开（在实现上事实上仅仅是将调用方设置为空指针）。

考虑以下代码：

```
fun foo(param) {
    // do something
}

;foo(bar)
```

上述代码在*Main*环境注册了一个*foo*方法，之后同样在*Main*环境下调用了*foo*方法，此时调用方为*Main*对象。

在执行这个调用的时候，会从*Main*环境“进入”到*foo*方法内部，此时会出现上下文环境的切换，*Iris*解释器会做这样子的工作：

>1. **创建一个新的上下文环境，并将该上下文环境的上级环境指定为*Main*环境，当前上下文环境压栈**；
2. 将*bar*作为参数进行压栈；
3. 将新建的上下文环境作为当前的上下文环境，代码执行流转到*foo*方法内部；
4. 在当前上下文环境中注册局部变量*param*，同时将栈顶的*bar*赋值给*param*（如果有多个参数，那么在调用方法时最后压栈的值对应最后一个参数）；
5. 执行*foo*中的代码； 
6. 执行完毕解释器携带最后执行语句所返回的值退出方法，执行流重新转回之前调用方法的地方；
7. 从上下文栈中弹出顶部的上下文环境作为当前的上下文环境。

再考虑以下代码：

```
class A {
    fun foo() {
        // do something
    }
    
    fun foo2() {
        ;foo()  // 2
    }
}

class B {
    fun __format(obj) {
        ;@obj = obj;
    }
    
    fun bar() {
        ;obj.foo2()
        // do something
    }
}

;obj = B.new(A.new())
;obj.bar() // 1
```

此处执行调用的流程和前述代码大同小异，需要注意的是代码中标记的的两个地方：

>1: 此处存在显式的调用方*obj*，因此切换上下文环境的时候，新的上下文环境将会指定*obj*作为调用方；
>2: 此处虽然没有显式的调用方，但是由于是在*A*类对象的实例方法内调用另外一个实例方法，因此此处会默认将调用方*@obj*作为调用方。

### 2. 创建或修改类、模块以及接口的时候
*Iris*在创建类、模块或者接口的时候，也会发生上下文环境的切换，下面以一个*Class*的创建为例。

```
;$flag = false

// 1
class A {
    // 2
    fun a() {
        // do something
    }
    
    // 3
    fun self.b() {
        // do something
    }
    
    if($flag) {
        fun c() {
        }
    }
}
// 4
```

在上面的代码中，一共标注了4个关键地方，下面对这4个地方进行简单说明。

>1: 在这里*Iris*解释器将会发生上下文切换，此处将会新建一个上下文环境，整个过程和方法调用相同，不同的是这里的上下文环境会被标识为特殊的上下文环境，并且它保存了一个调用方，调用方是*A*这个类对象；
2: 在2这个地方，*Iris*解释器通过当前上下文获取到了调用方（*A*类对象），然后向其注册了*a()*这个**实例方法**；
3: 在3这个地方，*Iris*解释器通过当前上下文获取到了调用方（*A*类对象），然后向其注册了*b()*这个**类方法**；
4: 在4这个地方，类定义结束，上下文环境切换回去。

模块、接口定义时的上下文切换和类定义时的上下文切换类似，此处不再赘述。

### 3. 创建闭包的时候
创建闭包时的上下文切换的内容放在下一节**Block闭包**中说明，此处从略。

<span id="s2"/>
## 2. Block闭包
闭包是携带变量穿越作用域或者延迟执行一段代码的一种实现手段。在*Iris*中，闭包通过*Block*对象来实现，***Iris*中的任何闭包都是*Block*类的一个对象**。

在*Iris*中一个*Block*对象代表了一组代码，它从表面上看起来和*Method*对象并没有什么区别，*Block*对象比起*Method*对象有一个特殊的功能，那就是**它能够记录创建它时候的上下文环境**，除此之外，*Block*对象还有一个自己的上下文环境。

考虑以下代码：

```
fun get_block() {
    return Block.new() {
        iterator => [msg] :
        ;print(msg)
    }
}

;blk = get_block()
;blk.call("Hello, World!")
```
上面的代码会打印

> Hello, World!

在了解*Block*的时候，需要先了解*Iris*中闭包的写法。

当没有参数要传递给闭包的时候，这个闭包写成：

```
;Block.new() {
    // do something
 }
```

当有参数要传递给闭包的时候，这个闭包写成：

```
;Block.new() {
    iterator => [param1, [param2, ..., *variableParam]]:
    // do something
 }
```
其中*variableParam*表示可变参数，意义与方法中的可变参数相同。

在示例代码中，当我们调用*get_block*方法的时候，将会返回一个*Block*对象，然后我们对这个对象调用*call*方法并传递了一个参数，此时就完成了这个*Block*对象的调用。

在创建*Block*对象（调用```Block.new()```）的时候，会完成这样一些工作：

>1. 和调用普通方法一样，创建一个新的上下文环境并进行其他相同的操作，然后根据后面的*Block*块创建一个临时的*InnerBlock*（不对外公开）对象，这个对象保存了新的上下文环境的上级上下文环境的引用（即创建*Block*对象时的上下文环境），并将其保存在新的上下文中，然后调用```Block.new()```方法；
2. ```Block.new()```方法中通过当前上下文环境获取到*InnerBlock*对象，并通过这个*InnerBlock*对象获取到创建当前*Block*对象时的上下文环境以及块中的代码；
3. **当前的*Block*对象新建一个自己的上下文环境，并将创建当前*Block*对象时上下文环境作为自己的上下文环境的上级上下文环境**。
4. 返回对象。

当*Block*对象调用自己的*call*实例方法的时候，将会把传递给*call*的参数原封不动地传给定义块的时候定义的参数，这个过程和调用方法并没有什么区别。

下面来解释*Block*实现闭包的原理。

首先，需要注意到上面关于创建*Block*对象时提到的**第3点**，一个*Block*对象内部事实上是保存了这样一个**上下文引用链**：

> *Block*对象自己的上下文环境  
   <- *Block*对象定义时的上下文环境
   <- *Block*对象定义时的上下文环境的上级上下文环境
   <- ...
   <- *Main*环境
   
而由于*Block*对象有了这么一条**上下文引用链**，因此它能够**逐层地去搜索变量**，从而达到携带变量穿透定义域的功能。

考虑以下代码：

```
;a = 1
;blk = Block.new() {
    ;b = 2
    ;c = 3
    ;return Block.new() {
        ;c = 4
        ;print(String.format("a={0},b={1},c={2}", a, b, c))
    }
 }
;blk2 = blk.call()
;blk2.call()
```
以上代码将会打印：

>a=1,b=2,c=4

可见，最里层的*Block*（即*blk2*）通过**上下文引用链**查找到了整条链上的所有变量，并且注意到由于在*blk*自身的块中定义了变量```;c=4```，因此外层的```;c=3```并没有被引用到。

进一步地，这里说明一下如果在*Block*中引用了一个变量，那么这个变量会视情况进行定义：

> 1. 变量为局部变量的情形
*Iris*解释器将会顺着*上下文引用链进行查找*，如果找到该局部变量，那么直接引用之；**否则在*Block*对象自己的上下文环境中定义之**；
2. 变量为实例变量的情形
同样的，*Iris*解释器将会顺着*上下文引用链进行查找*，此时查找的方法是如果某个上下文环境中拥有调用者（该情况的出现一般是由于该*Block*是由某个对象的实例方法生成并返回的），那么将会在这个调用者对象中去查找实例变量，而如果该上下文环境中没有调用者那么按照一般情况查找，如果查找到，则引用之；**否则在*Block*对象自己的上下文环境中定义之**；
3. 变量为类变量的情形
同样的，*Iris*解释器将会顺着*上下文引用链进行查找*，此时查找的方法是如果某个上下文环境中拥有调用者（该情况的出现一般是由于该*Block*是由某个对象的实例方法生成并返回的），那么将会在这个调用者对象所属的类/模块中去查找类变量，而如果该上下文环境中没有调用者那么按照一般情况查找，如果查找到，则引用之；**否则在*Block*对象自己的上下文环境中定义之**；
4. 变量为全局变量的情形
按照一般情况处理。

此外还有**常量**的查询，常量的查询和**类变量的查询基本一致，唯一的区别在于如果查找不到常量则*Iris*解释器直接报错，*Iris*禁止在闭包中定义常量**。

了解完*Block*对象，现在来看看*Iris*的**迭代器块**的写法。

*Iris*的迭代器块指的是在调用某个方法的时候，在圆括号后再跟一个块，里面编写代码；这个块将会作为一个*Block*对象传进这个方法中，而在方法中可以通过*cast*关键字引用到该*Block*对象。

考虑以下代码：

```
fun call_fun() {
    ;print("Before", "\n")
    if(cast) {
        ;cast.call("With Block")
    } else {
        ;print("Without Block", "\n")
    }
    ;print("After", "\n")
}

;call_fun()
;call_fun() {
    ;print(msg, "\n"}
 }
```
以上代码将会打印：

>Before
>Without Block
>After
>Before
>With Block
>After

可见，如果不给该方法编写块的话，那么*cast*将为*nil*。同时，如果需要根据有没有传入块分别进行处理，那么可以使用*with/without*语法：

```
fun call_fun() {
    ;print("Before", "\n")
    ;block
    ;print("After", "\n")
}
with {
    ;cast.call("With Block")
}
without {
    ;print(msg, "\n"}
}
```
其中*block*是一个占位符，调用时根据是否传入*Block*而分别执行*with*和*without*块中的代码。

由于*Iris*的*Block*对象类似函数式语言中的*lamba*表达式，因此*Iris*中提供了一个*lambda*方法，如果用*Iris*语言实现的话大概是下面这样（事实上是在*Kernel*中实现的内置方法）：

```
fun lambda() {
   if(cast) {
        ;return cast
   }
   ;groan(Error.new("No block taken"))
}
```

使用的方法如下：

```
;printer = lambda() { [msg]: ;print(msg) }
;adder = lambda() { [a, b]: ;return a + b }

;printer.call(adder.call(1, 2))
```

以上代码将会打印：
> 3