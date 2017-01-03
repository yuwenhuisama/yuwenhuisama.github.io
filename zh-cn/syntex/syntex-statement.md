# Iris语言的语句（Statement）
>Iris的语句是对表达式的组合，从而形成具有具体含义的语句。目前*Iris*拥有18种语句，下面将进行介绍。

<span id="s1"/>
## 1. 常语句（Normal Statement）
常语句就是对一个表达式的执行，它的格式为：
>;表达式。

请注意，*Iris*一个相当大的特点就是每一条常语句都是分号在前。
例：

```
;a = 10
;b = a + 20
;print(a, ",",b)
```

<span id="s2"/>
## 2. if语句（*if Statement*）
*Iris*有**两种if语句**，一种是__用于控制条件执行的，另一种适用于循环的__，分别称为**条件if**和**循环if**。下面分别进行介绍。

<span id="s2-1"/>
### 2.1. 条件if
条件if有四种表达形式：
>1. if(条件) { … }
>2. if(条件) { … } else {…}
>3. if(条件) { … } elseif(条件1) {…} elseif(条件2) {…} … elseif(条件n)
>4. if(条件) { … } elseif(条件1) {…} elseif(条件2) {…} … elseif(条件n) { … } else { … }

条件判断由上至下，当满足其中一个条件的时候（if/elseif），就会执行其后紧随的大括号内的一系列语句——两个大括号及其内部的语句被称为块（*Block*）。而如果条件都没有满足，同时存在else语句的话，那么将会执行else后的块。

请注意，Iris中除了false和nil以外都被视作真。

例：

```
;a = 1
if(a == 0) {
    ;print("a = 0")
}
elseif(a == 1) {
    ;print("a = 1")
}
else {
    ;print("a = other")
}
```

<span id="s2-2"/>
### 2.2 循环if
循环if是*Iris*特有的循环语法。您可能会发现，*Iris*没有while语句，是的，*Iris*移除了while语句，同时加入了可能更加方便使用的便的**循环if**。

Iris的if循环的基本语法如下：

>if(循环基础条件, 循环次数 [, 计次变量]){  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 循环体  
}

首先，循环基础条件是循环得以进行的基础，*Iris*第一次遇到到循环体的时候，将会检测循环基础条件，如果条件为真，那么就开始执行循环体。

而循环次数可以指定一个具体的数字来控制这个循环的执行次数，但是，请注意，__如果循环次数没有达到但是循环基础条件已经为假了的话，那么并不会继续执行循环__。

计次变量是一个可选的参数，它可以方便地记录下当前循环进行的次数，需要注意的是，计次变量必须是一个局部变量（不能为类变量、实例变量、全局变量等）。

循环的停止条件有如下几个：
>1. 遇到return语句；
>2. 遇到break语句；
>3. 基础循环条件为假；
>4. 循环次数达到指定值；

以上四点任何一点达到，循环都会停止。

例：

```
if(i <= 3, 4, i) {
    ;print(i)
}
```

上面这个例子将会顺序打印1、2、3，但并不会打印4，因为当i=4的时候，循环基础条件为假，结束循环。

另外，如果您设定的循环次数为小于等于零的数的话，那么if循环将会完全视基础循环条件是否成立而进行，因此您可以这样子实现一个无限循环：

```
if(true, 0) {
    // 循环体
}
```

而以下表达和直接使用if语句效果相同：

```
if(true, 1) {
    // 条件体
}
```

<span id="s3"/>
## 3. switch语句（*switch Statement*）
*Iris*的switch语句和C\+\+的switch语句非常类似，但是更加易用，并且在特性上也有所不同。

switch语句是多路条件选择语句，它可以和条件if语句等价互换，但如果选择项较多的话switch语句还是比较方便的。

switch语句的基本语法如下：

>switch(待比较的表达式) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;when(表达式1, 表达式2,…, &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表达式n) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 语句  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;when(表达式n+1, 表达式n+2,…, 表达式m) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 语句  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//…  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[else {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 语句  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}]  
}  

switch语句的执行首先计算待比较的表达式，保存其值，然后对每一个when子语句的表达式列表中对每一个表达式进行计算，如果遇到有一个when子语句的表达式列表中的某个表达式计算结果和待比较的表达式的值相同的话，那么就执行这条when子语句的块。如果全部不满足，且存在else子语句的话，那么将执行else语句中的内容。

需要注意的是Iris的switch语句执行完一个when或者else子语句之后，将会直接结束，而__不会像C\+\+的switch语句一样顺序往下继续执行__。

例：
```
;c = "Hello"
switch (c) {
    when(1, 2) {
        ;print("Number")
    }
    when("Hello") {
        ;print("String")
    }
    else {
        ;print("Other")
    }
}
```
<span id="s4"/>
## 4. for语句（*for Statement*）
for语句是对*Iris*中有范围概念的对象的遍历的一种糖衣循环，它可以用if循环替代。
基本语法为：

1、对数组：
>for(迭代变量in 数组) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 语句  
}

例：
```
;array = [1, 2, "array"]
for(I in array){
    ;print(i, "\n")
}
```
上面这个例子将会顺序打印出数组中的所有元素。

2、对*Hash*：
>for((键,值) in Hash) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 语句  
}  

例：
```
;hash = {1 => "Kuroneko", 2=> "Kurumi", 3=>10}
for((key, value) in hash) {
    ;print(key, "=>", value)
}
```
上面这个例子将会打印出该Hash的所有键值对。

3、对*Range*对象
>for(值 in Range对象) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 语句  
}

例：

```
for(i in (0 -> 10]) {
    ;print(I, "\n")
}
```

上面这个例子将会顺序打印出1-10的所有整数。

这里提一下*Range*对象，Range对象是Iris中对范围的抽象，比如"a"-"z",0-9等都可以被称为范围。Iris内置了一些范围，当然，您也可以继承*Range*类自定义一些自己的*Range*。

Iris的*Range*是基于数学的区间概念（离散的区间），比如您可以方便地字面指定一个拥有开闭的区间：

>**(0 -> 100)**：1-99的整数区间；**（左开右开区间）**  
**("a" -> "z"]**："b"-"z"的字符串区间；**（左开右闭区间）**  
**["Z" -> "A")**："Z"-"B"的字符串区间；**（左闭右开区间）**  
**[-100 -> 20]**：-100-20的整数区间；**（左闭右闭区间）**。  

<span id="s5"/>
## 5. break语句（*break Statement*）
break语句用于从循环中跳出。

注意，break语句仅能跳出包含该语句的所有循环的最内层的循环。而如果break语句出现在非循环语句的块中的话，Iris将会**报错**。

例：
```
for(i in [0 -> 10]) {
    if(i == 8) {
        ;break
    } else {
        ;print(i, "\n")
    }
}
```
上面这个例子将会顺序打印0-7的所有整数。

<span id="s6"/>
## 6. continue语句（*continue Statement*）
continue语句用于立即结束本次循环进入下一次循环。

同样的，continue语句仅能结束包含该语句的所有循环的最内层的循环。而如果continue语句出现在非循环语句的块中的话，Iris将会**报错**。

例：
```
for(i in [0->10]) {
    if(i % 2 == 0) {
        ;continue
    }
    ;print(i)
}
```
上面这个例子将会顺序打印0-10中所有的奇数。

<span id="s7"/>
## 7. 方法定义语句（*Method-Define Statement*）
方法定义语句用于在*Iris*中定义方法，而定义的同时将会产生相应的方法（*Method*）对象。至于方法定义在何处需要看定义该方法处于的上下文，在*Iris*中定义方法有三种上下文：**全局上下文、类上下文、模块上下文**。

__全局上下文表示该方法定义在全局，可以直接全局调用__。全局定义的方法无论是实例方法还是类方法，如果名称相同那么会视为同一方法，后定义的将会覆盖前定义的。
类上下文表示该方法定义在某个类中，此时实例方法和类方法将会区分对待。

__模块上下文和类上下文类似，表示该方法定义在某个模块中，此时实例方法和类方法也会区分对待__。

定义实例方法的语句为：

>fun 方法名([参数1, 参数2, … 参数n] [, *可变参数]) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 方法体  
}

定义类方法的语句为：

>fun self.方法名([参数1, 参数2, … 参数n] [, *可变参数]) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 方法体  
}

例：
```
fun fib(a, b, from, to) {
    ;print(b)
    if(from == to) {
        ;return
    }
    ;return fib(b, a + b, from + 1, to)
}
;print(1)
;fib(1, 1, 1, 10)
```
上面这个例子将会递归地顺序打印斐波那契数列的前10项。

类方法的定义将会在介绍类的时候说明。

注意如果定义了可变参数，那么调用方法的时候，属于可变参数的那些实参__将会被打包成为一个数组传给定义的那个可变参数形式参数变量__。

<span id="s8"/>
## 8. return语句（*return Statement*）
return语句用于从块中返回值。在*Iris*中return除了可以在方法中返回值以外，还可以从块中返回值，关于块是*Iris*中比较高级的概念，它与*Iris*的**闭包**息息相关，将会放到后面进行说明。

return将会立即结束当前方法的执行并返回值，如果return是在循环中，那么无论循环深度有多大，所有的循环都会被跳出然后返回值。

例：
```
;array = [0, 1, 2, 3, nil, 4, 5, 6]
fun elem_count_before_nil(array) {
    ;count = 0
    for(i in [0, array.size())) {
        if(array[i] != nil){
            ;count += 1
        }
        else {
            ;return count
        }
    }
}
;print(elem_count_before_nil(array))
```
上面这个例子将会计算数组中nil元素之前的所有元素的个数并打印出来。

请注意，return仅能出现在方法或者块中，如果在其他情况下使用return语句那么Iris将会报错。

<span id="s9"/>
## 9. 类定义语句（*Class-Define Statement*）
类是*Iris*这种面向对象语言中非常重要的组件，*Iris*的一切设计都是基于类的，因此可以说没有类，*Iris*不可能正常运作。

一个类就类似于一件产品的设计图纸，有了这张图纸就可以批量生产出无穷多个实实在在的产品，这一般被称为类的实例或者对象，在*Iris*中一般习惯称其为对象（*Object*）。

定义一个类非常简单：

>class 类名 [extends 父类]  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[involves 模块名1, 模块名2, …, 模块名n]  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[joints 接口名1, 接口名2, …, 接口名n]  
>{  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 类体  
}

*Iris*的继承是单继承，也就是说一个*Iris*类只允许有一个父类——*Iris*的设计屏蔽掉了多继承可能造成的继承混乱问题，如果您没有指定具体的父类，那么默认任何您创建的类的父类都是*Object*类，*Object*类是所有类的父类。

这里简单介绍一下*Iris*的继承体系，*Iris*底层的几个类分别为*Object、Class、Module、Method、Interface*类，它们的继承关系为*Object*类为其他类的父类，__而包括*Object*类在内的所有类的类对象本身又是*Class*类的对象__。——这里可能有些不好理解，简单说明一下。在*Iris*中，类本身也是一个对象，称为类类的对象（*Object of class Class*），下文如果没有做特殊说明，类对象，都是指某个类的类对象。而且类对象本身是可以被引用的，不过类对象永远被保存在常量中（*Iris*中要求类名必须大写字母开头，而以大写字母开头的标识符都是常量），因此保存类对象的容器是不可修改的。

类中允许定义实例方法、类方法、类变量，以及设定访问器等。但是请注意，*Iris*类不允许定义类中类，类中类这种类似命名空间的效用在*Iris*中可以用模块来代替。

类中定义的实例方法，可以为类的对象所调用；而类中定义的类方法，则仅能够由类本身调用。请注意，*Iris*在这里和C\+\+的特性是不一样的，在C\+\+中，类的对象可以调用类的静态成员函数，但是*Iris*中对象是不允许调用类的静态成员函数的。

下面的例子定义了一个分数类用于展示*Iris*的类定义：
```
class Fraction {
    ;gset [@numerator]
    ;gset [@denominator]

    // 构造方法
    fun __format(numerator, denominator) {
        // 约分处理
        ;gcd = Fraction.gcd(numerator, denominator)
        ;@numerator = numerator / gcd
        ;@denominator = denominator / gcd
    }
    
    // 显示
    fun display() {
        ;print(numerator, "/", denominator)
    }
    
    // 重写+方法
    fun +(obj) {
        if(obj.instance_of(Fraction) {
            ;gcd = Fraction.gcd(@denominator, obj.denominator)
            ;lcm = Fraction.lcm(@denominator, obj.denominator)
            ;return Fraction.new(
                @numerator * lcm / @denominator
                + obj.numerator * lcm / @denominator, lcm)
        }
    }
    
    // 最大公约数
    fun self.gcd(a, b) {
        if(a < b) {
            ;t = a
            ;a = b
            ;b = t
        }
        if(t = a % b != 0, 0) {
            ;a = b
            ;b = t
        }
        ;return b
    }
    
    // 最小公倍数
    fun self.lcm(a, b) {
        ;return a * b / gcd(a, b)
    }
}
;fra1 = Fraction.new(1, 2)
;fra2 = Fraction.new(6, 8)
;(fra1 + fra2).show()
```

这个例子将会输出5/4。

*Iris*一个类的对象通过调用类方法*new*来通过类对象生成。

上面的例子中涉及到了访问器，关于访问器将会在后面介绍。另外，在上面的例子中，还出现了构造方法*__format*，构造方法是在生成对象之后对象首先会隐式调用的方法，它是*Iris*完成的调用。这里需要说明的是，*Iris*只有构造方法而没有析构方法，因为*Iris*拥有垃圾回收机制，因此会在恰当的时刻回收所有托管到*Iris*上的垃圾内存，因此不需要析构方法。

<span id="s10"/>
## 10. 模块定义语句（*Module-Define Statement*）
*Iris*的模块在*Iris*中起到了命名空间的作用，一个模块能够定义模块中的模块，以及模块中的类，但模块的功能不仅仅是如此，*Iris*学习了*Ruby*的特征，模块实现了*mix-in*，弥补了因为*Iris*没有采用多继承而造成的一些缺点。

模块中可以定义实例方法、类方法和类变量、常量。一旦一个类包括了一个模块，那么当一个类的对象调用实例方法的时候，那么*Iris*还会在这个模块中搜索是否有这个方法——类变量和类方法同理。

模块需要被包括才能生效，而类和模块都能包括模块，包括的关键字是involve。定义一个模块非常简单：

>module 模块名 [involves 模块名1, 模块名2, …, 模块名n] {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 模块体  
}

下面定义了一个简单的Math模块。
```
module Math {
    fun self.sqrt(n) {
        // 平方根倒数速算法
        ;threehalfs = 1.5
        ;x2 = n * 0.5
        ;y = n
        ;i = y
        ;i = 0x5f3759df – (i >> 1);
        ;y = i.to_float()
        ;y = y * (threehalfs – x2 * y * y)
        ;return 1.0 / y
    }
}

;print(Math.sqrt(2))
```

这个例子将会返回2的平方根并打印。

<span id="s11"/>
## 11. 接口定义语句（*Interface-Define Statement*）

*Iris*的接口和静态预言的接口不同，*Iris*的接口仅仅作为一种规范的工具存在的。事实上您完全可以不使用接口，但如果您需要一种类扩展时候的规范的话，那么接口也是一种比较好的选择。

*Iris*的接口的功能非常单纯：一个类如果接入了一个接口的话，那么它必须实现接口中声明了的实例方法——并且实现的实例方法不仅要同名，参数个数和类型也要一样（这里的类型一样指的是如果有可变参数那么在实现里面也必须要有可变参数）。

接口中不定义具体方法，它只是一种对类定义的约束手段，以满足"__某一大类必须要实现某某方法以保证统一调用__"这一要求。另外接口是可以继承的，而且接口是多继承。

定义一个接口：
>interface 接口名 [joints 接口名1, 接口名2, …, 接口名n] {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 接口体  
}

关于接口中的方法的声明在下一节中介绍。

<span id="s12"/>
## 12. 接口方法声明语句（*Interface Function Declare Statement*）
接口方法声明语句用于在接口中声明形式上的方法，类似于C语言中的函数原型。但接口中是不允许定义具体的方法的，前面已经说过了，接口就是一种约束规范。

接口方法声明语句的格式为：

>;fun 方法名([参数1, 参数2, …, 参数n] [, *可变参数])

下面定义一个简单的接口来展示简单的插件系统的一个模拟：
```
interface Plugin {
    ;fun install(env)
    ;fun run(env)
    ;fun uninstall(env)
}

class MyPlugin joints Plugin {
    fun install(env) {
        ;print(env, ", ", "installed!")
    }

    fun run(env) {
        ;print(env, ", ", "run!")
    }

    fun uninstall(env) {
        ;print(env, ", ", "uninstalled!")
    }
}
```

<span id="s13"/>
## 13. Groan语句（*Groan Statement*）
*Iris*具有异常处理机制，异常处理是替代传统错误处理的比较先进的方式。*Iris*的异常处理机制是这样的：
>*Iris*在执行的时候抛出一个异常，那么解释器将会带着这个异常往上回溯，一直到顶层为止，如果遇到异常处理语句，那么*Iris*将这个异常交付给异常处理语句进行相应的处理，否则如果到顶层环境*Iris*将会报错。

Groan语句用于抛出一个异常对象，这个异常对象可以是任何*Iris*的对象。调用方法如下：

>;groan(表达式)

表达式产生一个对象，Iris解释器将会停止接下来的语句的运行，而带着这个对象沿着调用链往上走，直至遇到一个异常处理语句为止，并将这个对象交给异常处理语句的serve子语句；如果一直到调用链顶层（即*Main*环境）都没有遇到异常处理语句，那么*Iris*将会报错。

<span id="s14"/>
## 14. 异常处理语句（*Exception Handling Statement*）
*Iris*的异常处理语句用于处理由*Groan*语句产生的异常对象，并截获这个对象进行指定的处理。异常处理语句的结构如下：

>order {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 可能出现异常的语句  
}  
serve(接受异常对象的变量) {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 异常处理  
}  
[ignore {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// 无论是否有异常都必须做的处理  
}]  

下面结合Groan语句和异常处理语句处理一个只能够接受*Ingeter*参数的方法。

```
fun print_integer(i) {
    if(!i.instance_of(Integer)) {
        ;groan("Error parameter : it must be an integer !\n")
    }
    ;print("Integer is ", I, "\n")
}

order {
    ;print_integer("Hello, Iris!")
}
serve(e) {
    ;print(e)
}
ignore {
    ;print("must do.")
}
```

上述语句将会打印出：
>Error parameter : it must be an integer !  
must do.

<span id="s15"/>
## 15. 访问器定义语句（*Accessor-Define Statement*）
访问器定义语句用于公开化*Iris*类的实例变量，使其能够通过对象在外部被访问、设置，因为*Iris*的实例变量是强制私有的。这是类似C#的get/set语句的语法。

*Iris*的访问器定义语句有三种：get、set、gset，其中get/set语句又有默认和自定义两种。默认的访问器语句，是*Iris*将会自动为您定义一般行为的访问器，而自定义的访问器将会运行您所自定义的访问器逻辑。gset语句将会为您设定默认的get和set访问器。

>get访问器：  
默认：;get \[实例变量\] ()  
自定义：get \[实例变量\] () { // 您自己的逻辑 }

>set访问器：  
默认：;set \[实例变量\](临时变量)  
自定义：set \[实例变量\](临时变量) { // 您自己的逻辑 } 

>gset访问器只有默认形式：  
;gset \[实例变量\](临时变量)

访问器定义语句的实质，__是*Iris*自动地在类内部定义"\_\_get_+实例变量名"和"\_\_set_+实例变量名"的方法__，而用户在调用成员访问表达式的时候，*Iris*将会调用这两个方法。

下面是一个例子，类A的对象可以通过成员访问表达式来访问到它的实例变量 *@interger*，并且只能够为 *@interger*赋 *Interger*类型的值，否则会抛出异常。

```
class A {
    ;get [@integer] ()
    set [@interger] (value) {
        if(!value.instance_of(Integer) {
            ;groan("Error parameter : it must be an Integer !")
        }
        ;@integer = value
    }
    
    fun __format() {
        ;@interger = 0
    }
}

;obj = A.new()
;print(obj.integer, "\n")
;obj.interger = 10
;print(obj.interger, "\n")
order {
    ;obj.integer = "Hello, Iris!"
}
serve(e) {
    ;print(e)
}
```
上面的例子会打印
>0  
10  
Error parameter : it must be an Integer !

<span id="s16"/>
## 16. 方法权限定义语句（*Method Authority Statement*）
方法权限定义语句用于定义类方法、实例方法的访问权限，请注意，在*Iris*中权限定义仅仅是对于方法而言的，因为*Iris*不同于C\+\+没有成员变量的说法。

方法权限定义的关键字有三个：everyone/native/personal，分别对应公开/继承/私有方法，与C\+\+中的public/protected/private类似。

定义权限的语句格式为：
>;everyone/native/personal \[方法名\]

其中everyone为方法定义了这样的访问行为：
>对于实例方法，类内部实例方法、对象、子类实例方法内部都能够调用该方法；对于类方法，类内部类方法、类对象都能进行调用。

native为方法定义了这样的访问行为：
>对于实例方法，类内部实例方法、子类实例方法内部都能够调用该方法；对于类方法，仅类内部类方法能进行调用。

personal为方法定义了这样的访问行为：
>对于实例方法，仅类内部实例方法可以调用；对于类方法，仅类内部类方法可以调用。

由上面可以看到，native和personal对于类方法而言作用一致。

请注意，必须在定义了方法之后才能定义其权限，否则Iris将因为找不到指定名称的方法而报错。

下面是一个例子，展示了方法的权限定义：

```
class A {
    fun __personal_method() {
        ;print("Personal Method!", "\n")
    }

    fun _native_method() {
        ;print("Native Method!" , "\n")
    }

    fun everyone_method() {
	    ;print("Everyone Method!" , "\n")
    }

    ;personal [__personal_method]
    ;native [_native_method]
    ;everyone [everyone_method]
}

class B extends A {
    fun call_method() {
        ;_native_method()
        ;everyone_method()
    }
}

;obj= B.new()
;obj.call_method()
```

这个例子的结果将会打印出
>Native Method!  
Everyone Method!

而如果您使用obj对象调用 *\_\_personal_method*()或者 *_native_method*()的时候，*Iris*将会报错。

<span id="s17"/>
## 17. block语句和cast语句（*block Statement and cast Statement*）
block语句和cast语句是配合使用的语句，其中block用于判断调用方法的时候是否存在隐式传入的*Block*，如果有进入with语句块否则进入without语句块；而cast则代表了那个传进来的*Block对象*。

以下例子根据是否有*Block*产生两种行为：

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
    ;print("Without Block", "\n")
}

;call_fun() {iterator => [msg] : ;print(msg, "\n")}
;call_fun()
```

上面的例子将会分别打印出：

>Before  
With Block  
After  
Before  
Without Block  
After  

block/cast语句事实上是一个语法糖，上述代码可以按照下面这样子实现：

```
fun call_fun(blk) {
    ;print("Before", "\n")
    if(blk) {
        ;blk.call("With Block")
    } else {
        ;print("Without Block", "\n")
    }
    ;print("After", "\n")
}

;call_fun(nil)
;call_fun(Block.new() {iterator => [msg] : ;print(msg, "\n"})
// 或者 ;call_fun(lambda() { iterator => [msg] : ;print(msg, "\n"})
```

顺便一提，上面提到的*lambda*()函数的实现非常简单：

```
module Kernel {
    fun self.lambda() {
        ;return block
    }
    with {
        ;return cast
    }
    without {
        ;groan("No block taken!")
    }
}
```

关于*Block*的用法将会在后面进行介绍。

<span id="s18"/>
## 18. super语句（*super Statement*）
super语句用于子类的实例方法调用同名的父类实例方法，如果父类不存在这个方法那么将会报错。值得一提的是，由于*Iris*的语言性质，所以子类在覆盖父类方法的时候，只需要子类方法的名字和父类方法相同即可，并不要求参数个数要相同。

super语句的格式为：
	
>;super(参数列表)
	
注意，这里的参数是传给父类的同名方法的。同时*Iris*中super关键字只能用于实例方法的调用中。
以下例子展示了super语句的用法。

```	
class A {
    fun hello(msg) {
        ;print(msg, "from class A!", "\n")
    }
}

class B {
    fun hello(msg) {
        ;super(msg)
        ;print(msg, "from class B!" , "\n")
    }
}

;obj = B.new()
;obj.hello("Hello")
```

以上例子将会打印如下信息：
>Hello from class A!  
Hello from class B!