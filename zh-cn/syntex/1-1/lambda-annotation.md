# Iris的Lambda表达式和注解

> 在*Iris* 1.1中主要为*Iris*语言增加了两项语言特性：
>
> 1. Lambda表达式：便于函数式风格的程序设计；
> 2. 注解：提供安全的动态类型检查以及可定制的语言扩展；

<span id='s1'/>

## 1、Lambda表达式

<span id='s1-1'/>

### 1.1 Lambda表达式的语法

*00Iris*的*Lambda*表达式其实是生成*Block*对象的语法糖，提供类似于函数语言中的*Lambda*表达式一样的语法来简化*Block*对象的生成。

*Iris*中*Lambda*表达式的语法如下：

> ([参数列表]) => {[*Lambda*表达式要执行的代码块]}

以下的代码示例都是合法的*Iris**Lambda*表达式：

```clike
() => { ;print("Hello, World") }
(param) => { ;print(param) }
(arg, *params) => { 
	;print(arg, '\n')
	;params.each() { iterator => [e] : ;print(e, '\n') }
}
```

*Lambda*表达式的参数列表和*Iris*中方法、*Block*对象的参数列表意义完全相同。

同时，由于***Lambda*表达式是对生成*Block*对象的简化**，因此上述示例代码事实上等同于下述代码：

```clike
Block.new() { ;print("Hello, World") }
Block.new() { iterator => [param] : ;print(param)}
Block.new() {
    iterator => [arg, *params] :
	;print(arg, '\n')
    ;params.each() {iterator => [e] : ;print(e, '\n') }
}
```

因此，当我们要调用*Lambda*表达式的时候，我们要按照*Block*对象的方式来进行调用，下面是一个完整的示例：

```clike
;max = (a, b) => { ;return a > b ? a : b }
;i = 100
;j = 200
;print(max.call(a, b))
```

上述代码执行后将会打印

> 200

<span id='s1-2'/>

### 1.2 Iris Kernel模块中的lambda方法

在*Iris* 1.1中，*Kernel*模块也提供了一个名字叫做*lambda*的方法用于创建*Block*对象，它的实现代码很简单：

```clike
fun lambda() {
    ;return block
}
with {
	;return cast
}
without {
	;groan("No block taken!")
}
```

使用的时候如下：

```
;max = lambda() { iterator => [a, b] : ;return a > b ? a : b }
```

*Iris*建议使用*Lambda*表达式来创建闭包对象。

<span id='s1-3'/>

### 1.3 Iris中的self关键字的行为

在*Iris* 1.1之前对于各个环境下*self*关键字的行为一直是未定义的，为了避免*Iris*中的*self*关键字出现*Javascript*中*this*关键字那样令人迷惑的效果，在这里对*self*关键字的行为作出唯一的规定：

> 1. *self*关键字**只允许出现在方法体或者*Block*中**；
> 2. 当*self*关键字出现在方法体中时，如果不存在调用该方法的对象，那么*self*的使用将会被视为非法调用，解释器将会抛出错误；如果存在调用该方法的对象，那么*self*就是这个对象；
> 3. 当*self*关键字出现在*Block*中时，*self*将会沿着闭包链一直往上查找，直至查到第一个**不是闭包环境的环境**，此时这个环境中如果存在调用对象，那么*self*便是这个环境中的调用对象，否则，*self*的使用将会被视为非法调用，解释器将会抛出错误。

针对以上几个情况给出几个例子：

```clike
;obj = self // 1

fun foo() {
  	;return self  // 2
}

;f = () => { ;print(self.name) }

class Test {
  	;gset [@name]
    
    fun __format(f) {
      	;@f = f
        ;@name = "Hello"
    }
  
  	fun call_block() {
      	;@f.call()
  	}
}

;obj = Test.new(f)
;obj.call_block()  // 3
```

在上述代码中：

> 1 处将会在编译期就报错，因为*self*必须存在于方法体或者*Block*中；  
>
> 2 处如果直接调用**foo()**的话，也会报错，因为没有具体的调用对象；  
>
> 3 处将的*obj*对象的实例变量*@f*是一个内部定义了*self*关键字的*Block*对象，当这里调用该*Block*对象的时候，*self*将会向上回溯查询，最终查询到第一个调用者*obj*，于是*self*指向*obj*，此处打印出**Hello**。

<span id='s2'/>

## 2、注解

<span id='s2-1'/>

### 2.1 注解的作用

在*Java*、*C#*这样子的语言中都存在**注解**这样子的元数据，它提供了代码级别的说明，能够对代码中的类、方法等元素进行说明、注释，提供辅助文档编写、代码分析以及提供更多编译信息的功能，并且这些注解都是可以由用户自定义的。

 在*Iris* 1.1中，也增加了名为**注解（*Annotation*）**的语法，*Iris*中的注解主要是用来为**任何一个*Iris*对象做额外的预先标注、处理工作，以在动态执行的过程中完成代码层面以外的额外工作**。

*Iris*中的注解的应用场景有且不限于以下几点：

> 1. 参数类型检查
> 2. 入口标注
> 3. 预处理

此外，*Iris*的注解语法可能会在未来的版本中发生新增、变化，且尽量保证向下兼容。

<span id='s2-2'/>

### 2.2 注解的语法

*Iris*中的注解需要放在一个**表达式、类、模块、接口、方法、块**的前方，且中间不能有别的其他元素。

*Iris*中一个合法的注解形式为：

```
#注解名([参数列表])
[表达式|类定义语句|模块定义语句|接口定义语句|方法定义语句|块语句]
```

***Iris*中任何一个注解都是*Annotation*类及其子类的一个对象**，它将会作用于被它所注解的那个对象身上，作用时机为**被注解的那个表达式、语句执行完毕**的时候。

下面以*Iris*内置的名为***TypeCheck***的注解为例，来直观地说明一下*Iris*注解的作用。

```clike
fun print_if_string(obj) {
	;print(
		#TypeCheck(String)
		obj
	 )
}

;print_if_string("Hello, World!") // Correct
;print_if_string(1) // Wrong type!
```

上面的例子等价于：

```clike
fun check_type(obj, tar_class) {
	if(obj.__class == tar_class) {
		;return obj
	}
	;groan("Wrong type!")
}

fun print_if_string(obj)}
	;print(check_type(obj, String))
}
```

可以看到，虽然可以定义方法来进行相同的类型检查工作，但是**注解使得这一工作更加一目了然**。

需要注意的是，在上述例子中*TypeCheck*看起来似乎是作用到了*obj*这个对象身上了，但实际并不然，这其中有这样一个过程：

> 1. ```obj```是一个表达式，它**返回*obj*这个实例变量中的对象**；
> 2. 表达式执行完毕，注解*TypeCheck*生成一个实例并作用在这个返回的对象身上。

<span id='s2-3'/>

### 2.3 注解的实质

注解的实质是*Iris*编译器在进行编译的过程中，编译器扫描到注解的时候，会在当前的**表达式、类、模块、接口、方法、块**后方自动插入一些语句，以上面的**TypeCheck**的例子来说，编译器事实上是插入了这样一段代码：

```clike
fun print_if_string(obj) {
	;print(
		;TypeCheck.new().apply(obj, String)
	 )
}
```

根据注解的对象的不同，注解的作用时间也不一样：

> 1. 如果是表达式，那么会在表达式返回值的时候作用；
> 2. 如果是类、模块、接口，那么会在该类、模块、接口定义完毕的时候作用；
> 3. 如果是方法，那么会在方法被定义后、添加到其对应的环境中之前作用；
> 4. 如果是块，那么会在块被定义之后作用；

其次，每一个注解事实上都是***Annotation*类及其子类的一个对象**，上述**TypeCheck**这个注解事实上也是如此，它的定义类似下面这样：

```clike
class TypeCheck extends Annotation {
	fun __format() {
      	;super()
	}
  	
  	fun parameter_define() {
      	;return [Class]
  	}
  
  	fun apply(obj, *params) {
      	if(obj.__class != params[0]) {
          	;groan("Error Type!")
      	}
      	;return obj
  	}
}
```

一般来说，一个可用的注解类应该重写*parameter_define*以及*apply*方法，这两个方法的签名如下：

``` clike
fun parameter_define();
fun apply(obj, *params);
```

其中*parameter_define*用于定义*apply*方法*params*这个可变参数所能接受的参数类型，这个方法需要返回一个数组，其中按顺序存放要传入注解*params*中的各个参数的类型（比如上面```#TypeCheck(String)```这行注解传入到*apply*的*params*中的就是*[String]*）；如果参数可变或者需要用户自行控制，那么直接返回*nil*即可。

而*apply*是**将注解应用到目标对象身上的关键方法**，它的第一个参数就是被注释的对象，第二个参数就是在注解中传入的参数。

<span id='s2-4'/>

### 2.4 注解作用示例

> 下面以不同对象为例，说明注解是如何作用在不同对象身上的

<span id='s2-4-1'/>

#### 2.4.1 注解作用在表达式上

当注解用于表达式上的时候，它只会**对最接近它的那个表达式的返回值作用**。

考虑以下代码：

```clike
;#TypeCheck(<ClassName>)
 xxx().yyy().zzz()

;(#TypeCheck(<ClassName>)
  xxx()).yyy().zzz()
/*
也可以写成
;(#TypeCheck(<ClassName> xxx()).yyy().zzz()
*/

;(#TypeCheck(<ClassName>)
  xxx().yyy()).zzz()
/*
也可以写成
;(#TypeCheck(<ClassName>) xxx().yyy()).zzz()
*/
```

上面三个注解的作用对象并不相同：

1. 第一个注解的作用对象是表达式```xxx().yyy().zzz()```的整体返回值；
2. 第二个注解的作用对象是表达式```xxx()```的返回值；
3. 第三个注解的作用对象是表达式```xxx().yyy()```的整体返回值；

再次强调，**注解只能作用于表达式（Expression）上，而不能作用于语句（Statement）**，所以如果上述代码写成：

```clike
#TypeCheck(<ClassName>)
;xxx().yyy().zzz()
```

那么编译器将会直接报错，因为```;xxx().yyy().zzz()```是一个常语句。

<span id='s2-4-2'/>

#### 2.4.2 注解作用在类、模块、接口上

由于注解作用在类、模块、接口上的逻辑是一致的，所以这里放在一起说明。

作用在类、模块、接口上的注解将会在类、模块、接口定义完毕之后作用。下面我们自定义一个注解，用来检查该类的所有实例方法的名字是否都以小写字母开头。

```
class CheckInstanceMethodName extends Annotation {
	fun __format() {
      	;super()
	}
  	
  	fun parameter_define() {
      	;return nil
  	}
  
  	fun apply(obj, *params) {
      	;regex = Regex.new(@"^[a-z].*")
      	;(#TypeCheck(Class) obj).__own_instance_methods.each() {
			iterator => [m] :
			if(!regex.match(m.name)) {
				;print('Wrong method name : ', m.name, '\n')
			}
      	}
      	
      	return obj
  	}
}
```

然后我们定义类：

```
#CheckInstanceMethodName()
class TestClass {
	fun Test() {}
	fun test() {}
}
```

这个示例将会打印：

> Wrong method name : Test

<span id='s2-4-3'/>

#### 2.4.3 注解作用在方法、块上

当注解作用在方法或块上时，可以控制方法或块的作用时机。

> 1. 调用方法或块前
> 2. 调用方法或块后
> 3. 定义方法或块时

除3以外，作用时机的控制的关键在于*Method*（*Block*）对象的两个属性：\_\_pre_call/\_\_post_call。

其中：

> *__pre_call*接受签名为 (obj, \*params) => {} 的*Block*对象，其中*obj*表示调用该方法的对象，*params*表示调用该方法时传入的参数；
>
> *__post_call*接受签名为(result) => {} 的*Block*对象，其中*result*表示该方法执行完毕的返回值。

下面以定义方法的注解为例，举一个具体的例子来说明这三种作用时机。在下面的例子中，我们定义一个*StartWithLowerCase*注解来检查被注解的方法是否是以小写字母开头，如果是，则打印出该方法的名字；然后我们再定义一个*ParameterCheck*注解来在方法被调用前检查传入的参数是否为指定类型（实现类似静态语言类型检查的功能）；最后，我们定义一个*ConvertResultToString*注解来把被注解的方法的返回值都转换成*String*。

```clike
class StartWithLowerCase extends Annotation {
	fun __format() {
      	;super()
	}
  
    fun parameter_define() {
      	;return nil
  	}
  
  	fun apply(obj, *params) {
      	if(['a'=>'z'].include(obj.name[0])) {
          	;print(obj.name, "\n")
      	}
      	;return obj
  	}
}

class ParameterCheck extends Annotation {
  	fun __format() {
      	;super()
  	}
  
    fun parameter_define() {
      	;return nil
  	}
  	
  	fun apply(obj, *params) {
		;obj.__pre_call = (obj, *rparams) => {
          	if(!all_extends(rparams, params)) {
              	;groan("Invaild parameter list.\n")
          	}
		 }
      	;return obj
    }
  
  	fun all_extends(objs, classes) {
      	if(objs.size != classes.size) {
          	;return false
      	}
      	;i = 0
        if(true, i < objs.size) {
          	if(!objs[i].__instance_of(classes[i])) {
              	;return false
          	}
        }
      	;return true
  	}
}

class ConvertResultToString extends Annotation {
  	fun __format() {
      	;super()
  	}
  
    fun parameter_define() {
      	;return []
  	}
  
  	fun apply(obj, *params) {
		;obj.__post_call = (result) => {
          	;return result.to_string()
		 }
    }
}
```

然后编写我们的测试类：

```clike
class TestClass {
    #StartWithLowerCase()
  	fun foo1() {}
  	
  	#StartWithLowerCase()
  	fun Foo2() {}
  
  	#ParameterCheck(String, String)
  	fun foo3(p1, p2) {
      	;print("Hello, ", p1, " ", p2, '\n')
  	}
  
  	#ConvertResultToString()
	fun foo4() {
      	;return true
	}
}

;obj = TestClass.new()
;obj.foo3("One", "Two")
;print(obj.foo4())
;obj.foo3(1, 2)
```

执行代码后，将会打印：

> foo1
>
> Hello, One Two
>
> true
>
> Invaild parameter list

<span id='s2-5'/>

### 2.5 注解作用的顺序

同一个对象可以作用多个注解，注解的作用顺序为：**离作用对象最近的那个注解先作用，然后再以此往前作用**。

比如，如果有3个注解*A、B、C*，作用在一个表达式上：

```clike
#C()
#B()
#A()
expression
```

那么执行顺序将会是：

```clike
C.new().apply(B.new().apply(A.new().apply(expression)))
```

事实上上述代码与下面的代码等价：

```clike
#C() (#B() (#A() expression))
```