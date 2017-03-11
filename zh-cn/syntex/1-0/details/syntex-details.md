# 关于Iris的细节说明
<span id="s1"/>
## 1. 常量
_Iris_中的常量约定以大写字母开头进行表示，亦即：**凡是以大写英文字母开头的标识符都代表一个常量**。此外，常量一旦定义则不能被改变，**重新为已经定义的常量赋值会导致Iris解释器抛出运行时异常**。

只能在三种情况下定义常量：
>1. 全局环境下；
>2. 类体中；
>3. 模块体中。

除此之外在任何地方定义常量都会被视为非法行为，**Iris解释器将会抛出运行时异常**。

定义常量的位置决定了常量的所属，以下是一个例子：

```
;TOP_CONST = "Top Const"
module TestModule {
    ;MODULE_CONST = "Module Const"
    class TestClass {
        ;CLASS_CONST = "Class Const"
        
        fun self.print_const() {
            ;print(CLASS_CONST, "\n")
            ;print(MODULE_CONST, "\n")
            ;print(TOP_CONST, "\n")
        }
    }
    
    fun self.print_const() {
        ;print(MODULE_CONST, "\n")
        ;print(TOP_CONST, "\n")
    }
}

fun main_print() {
    ;print(TOP_CONST, "\n")
}

;print(TOP_CONST, "\n")
;print("\n")
;main_print()
;print("\n")
;TestModule.print_const()
;print("\n")
;TestModule::TestClass.print_const()
```

以上代码将会打印出：

> Top Const
>
> Top Const
> Module Const
>
> Top Const
> Module Const
> Class Const

### 补充1：Iris中常量的查找顺序
*Iris*中的常量按照如下顺序查找：

>1. 如果当前上下文为**运行时上下文**，那么查找顺序为当前对象的类、当前对象的类所包含的模块、当前类的父类、当前类的父类所包含的模块……一直上升到全局环境下查找，如果中途查找到，则返回之，否则报错；（如果当前环境已经是全局环境，那么该常量将被添加到全局环境中）
>2. 如果当前上下文为**模块定义上下文**，那么查找顺序为当前模块、当前模块的上层模块……一直上升到全局环境下查找，如果中途查找到，则返回之，否则往该模块添加此常量，常量值为*nil*；
>3. 如果当前上下文为**类定义上下文**，那么查找顺序为当前类、当前对象的类所包含的模块、当前类的父类、当前类的父类所包含的模块……一直上升到全局环境下查找，如果中途查找到，则返回之，否则往该类添加此常量，常量值为*nil*；

### 补充2：Iris中类、模块以及接口的定义与常量的关系
*Iris*中规定类、模块以及接口名必须以大写字母开头，类似以下的定义将会导致*Iris*解释器抛出运行时错误：

```
class test_class {}
module $test_module {}
interface @interface {}
```

之所以这样子规定，是因为***Iris*中所有的类、模块以及接口都是以常量的形式存在于各自的环境中**，换言之，这些类、模块以及接口对象都能够按照常量的方式来引用。

>注：*Iris*秉承“Everything is Object”的理念实现了面向对象模型，这也直接导致了*Iris*中的常量有这样子的特性，具体请移步至《Iris面向对象模型介绍》进行深入了解。

考虑如下代码：

```
module A {
    module B {
        class C {
        }
        
        interface D {
        }
    }
    
    class C {
    }
}

;print(A.__constance.keys.include?("B"), "\n")
;print(A.__constance.keys.include?("C"), "\n")
;print(A::B.__constance.keys.include?("C"), "\n")
;print(A::B.__constance.keys.include?("D"), "\n")
```

以上代码会打印出：
> true
> true
> true
> true

可见，定义模块、类以及接口事实上就是在往相应的环境里添加常量对象（__Class类对象、Module类对象、Interface类对象__）的过程，以上代码完全可以用以下元编程的方法来实现：

```
;A = Module.new() // 全局环境中
;A.add_constance("B", Module.new()).add_constance("C", Class.new())
;A.constance("B").add_constance("C", Class.new()).add_constance("D", Interface.new())

;print(A.__constance.keys.include?("B"), "\n")
;print(A.__constance.keys.include?("C"), "\n")
;print(A::B.__constance.keys.include?("C"), "\n")
;print(A::B.__constance.keys.include?("D"), "\n")
```

以上代码会打印出：
> true
> true
> true
> true

<span id="s2"/>
## 2. 变量

*Iris*有以下几种变量：
>1. 局部变量；
>2. 实例变量；
>3. 类变量；
>4. 全局变量。

关于**实例变量**以及**类变量**的说明在《Iris面向对象模型》第5节中有所提及，这里结合局部变量进行更进一步的说明。

<span id="s2-1"/>
### 2.1. 局部变量
*Iris*中的局部变量是指**以小写字母开头的合法标识符命名的变量**。局部变量的作用域为：**全局环境、方法内部、闭包**。

> 注意：*Iris*禁止在类体、模块体以及接口体内定义局部变量。

在全局环境下定义的局部变量将会被注册到全局环境中，在方法内部定义的局部变量将只会在方法执行的过程中被定义到当前方法的执行上下文中使用，而闭包下的局部变量的行为就比较特殊，这里放到4.3节来进行深入介绍。

**除闭包情况以外，在任何环境下定义的局部变量能且仅能在同一环境下被访问到**，考虑如下代码：

```
;a = 10
fun my_print() {
    ;print(a)
}

;print(a, "\n")
;my_print()
```

以上代码将会打印出：

> 10
> nil

注意到*my_print*()方法内部是无法访问到定义在全局环境中的局部变量*a*的，当*my_print*()方法调用的时候，它的执行上下文被切换到了一个新的上下文中，而在这个新的上下文中并没有*a*存在，因此*Iris*自动定义*a*为*nil*并注册到该环境中。

<span id="s2-2"/>
### 2.2. 实例变量
*Iris*中的实例变量是指**以@开头的合法标识符命名的变量**。实例变量的作用域为：**全局环境、对象内部、闭包**。

实例变量在全局环境中的行为与局部变量一致，而关于闭包中的实例变量同样放到4.3节中来进行详细阐述。

实例变量是**一个实例专有的变量**，同一个类的不同的实例即便变量名同名，值也可能是不同的。在该实例的所有实例方法中，都可以引用到该对象的实例变量。

下面是一个例子：

```
class A {
    fun __format(param) {
        ;@param = param
    }
    
    fun my_print() {
        ;print("@a", "\n")
    }
}

;obj1 = A.new("Hahaha")
;obj2 = A.new("Nonono")
;obj1.my_print()
;obj2.my_print()
```

以上代码将会打印：

> Hahaha
> Nonono

实例变量的定义发生在**在对象第一次引用它的时候**。

那么，如果在类方法中定义实例变量是怎么样的行为呢？由于*Iris*规定**在类或者模块中定义的类方法事实上就是该类对象或者该模块对象的实例方法**，因此，类方法中定义的实例变量将会成为**该类对象或者该模块对象的实例变量**。

以下是一个例子，由于*getter/setter*语句仅用于类对象的实例变量，因此这里采用元编程的方式取出模块对象的实例变量。

```
module TestModule {
    fun self.my_define_instance_variable() {
        ;@value = "Hello"
    }
}

;print(TestModule.__instance_variables.include?("@value"), "\n")
;TestModule.my_define_instance_variable()
;print(TestModule.instance_variable("@value"))
```

以上代码将会打印：
> false
> Hello

<span id="s2-3"/>
### 2.3. 类变量

*Iris*中的类变量是指**以@@开头的合法标识符命名的变量**。实例变量的作用域为：**全局环境、对象内部、闭包**。

同样，全局环境下类变量的行为与局部变量相同，而关于闭包中的类变量同样放到4.3节中来进行详细阐述。

类变量是一个类的实例共享的变量，严格来说类变量不符合*Iris*面向对象模型的要求，类变量完全可以用其他方式模拟出来，类变量仅仅是语法糖一般的存在。

以一段代码来解释类变量的行为：

```
class TestClass {
    ;@@value = 0
    
    fun increase() {
        ;@@value += 1
    }
    
    fun print_class_variable() {
        ;print(@@value, "\n")
    }
}

;obj1 = TestClass.new()
;obj2 = TestClass.new()
;obj1.print_class_variable()
;obj2.increase()
;obj2.print_class_variable()
```

以上代码会打印出：

> 0
> 1

类变量的查找顺序类似常量，在此不再赘述。

<span id="s2-4"/>
### 2.4. 全局变量

*Iris*中的类变量是指**以$开头的合法标识符命名的变量**。实例变量的作用域为：**任何作用域**。

*Iris*的全局变量无论在何处被定义，都会自动注册到全局环境中，而在任何地方引用，也都会在全局环境中查找。

<span id="s3"/>
## 3. 类、模块与接口
这部分内容请参考《Iris面向对象模型》。