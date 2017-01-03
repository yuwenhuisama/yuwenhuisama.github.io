# Iris对象模型介绍
#### 导语
> 本文档用于介绍Iris语言面向对象模型的具体概念，以及在具体实现上的建议实现方法。

<span id="s1"></span>
## 1、术语定义
- **Host Side**：特指用于实现Iris语言的解释器/虚拟机的语言方（如C++/Java/Javascript等）；
- **Appear Side**：特指Iris脚本语言的文本、所要执行的内容等；
- **Iris Core**：特指用于执行Iris脚本的核心构件，在用面向对象的描述方式下，包括且不仅包括（依据具体实现而定）*IrisObject、IrisClass、IrisModule、IrisInterface、IrisMethod、IrisClosureBlock*这六个类；
- **Extend Class/Module**：特指在*Host Side*编写的、可以在Iris中使用的类/模块；
- **User Class/Module**：是Extend Class的一类，指的是第三方用于在Host Side编写的、用于扩展Iris的类/模块；
- **Iris Bytecode**：指为Iris语言设计的虚拟机字节码
- **Interpreter**：指用于解释并执行Iris Bytecode的解释器以及相关的构件比如*GC模块*等（视具体实现而定），有时*Interpreter*也用*Virtual Machine*代称；

>注1：在本文档对于Iris对象模型的实现的描述上，约定均以面向对象的观点来进行描述；  
>注2：为避免混淆，下文中凡是以*Iris*作为前缀的类名都是指在*Host Side*实现的类，其它的均为在*Appear Side*实现的类。

<span id="s2"></span>
## 2、Everything is Object
在Iris中，一切语言组件都被视为对象。Iris的字面语言组件有以下几类：

>1. 对象（Object）
>2. 类（Class）
>3. 模块（Module）
>4. 接口（Interface）
>5. 方法（Method）
>6. 块（Block）

其中*对象*即为字面意义上的对象，它在*Host Side*均以*IrisObject*的形式表示，而剩下的*类、模块、接口、方法、块*在*Host Side*分别以*IrisClass、IrisModule、IrisInterface、IrisMethod、IrisClosureBlock*来表示。然而在*Appear Side*，所有的这些组件均视为某各类的对象——具体而言，也就是说__任何一个Class、Module、Interface、Method、Block都分别是Iris语言中*Class、Module、Interface、Method、Block*类的一个对象。__

以Class类为例，考察如下Iris代码：

```
class MyClass {
    fun foo(bar) {
        // do something
    }
}
;obj = MyClass.new()
;obj.foo(nil)
```

在上面的代码中我们定义了一个类名字叫MyClass、它包含一个实例方法foo，然后生成了一个*MyClass*类的对象，并调用实例方法*foo*。这是一个比较浅显的观点，而具体而微地站在Iris面向对象模型上来看，实际上这段代码做了这样一些事情：

>1. 生成一个*Class类*的对象，__ 并将这个对象赋值给常量*MyClass*__；
>2. 生成一个*Method类*的对象，并将这个对象注册给*MyClass*常量所保存的那个*Class*对象以作为一个**实例方法**，并且*MyClass*中的那个类对象的实例能够通过*foo*来访问到这个方法；
>3. __*MyClass*中的那个类对象调用了它自己的实例方法*new*__（**这个实例方法定义在 *Class*类中**），生成了一个新的对象，并保存在变量*obj*中；
>4. *obj*对象通过*foo*查找到自己的实例方法*foo*；
>5. *foo*方法调用自己的实例方法*call*（**定义在 *Method*类中**）；

也就是说，去掉语法糖衣，真实的代码类似于下面这样：

```
;MyClass = Class.new()
;foo = Method.new() {
    iterator => [bar] :
    // do something
 } // 这里用到了Iris元编程的内容，看不懂可以不管
;MyClass.add_instance_method(foo)
;obj = MyClass.new()
;foo_mtd = obj.get_instance_method("foo")
;foo_mtd.call(obj, nil)
```

>注1: 顺便一提，从这里可以看出对于Iris而言，*class语句、fun语句*都算是语法糖衣，我们完全可以通过**元编程**的方法完成一切类、方法的定义等操作。  
>
>注2：对于`foo_mthd.call(obj, nil)`这一段代码，显然存在一个疑问，按照这个推理下去，应该是*foo_mthd*继续去查找*call*方法，然后找到*call*方法对应的那个*Method*对象，然后这个对象再去调用*call*……这样下去就出现了一个*反复查找-调用*的死循环。在逻辑上这个问题确实是讲得通的，因此在Iris的脚本书写中，调用方法直接使用 *method_name()* 的形式，其实是用这个小trick屏蔽了这样一个无限循环的怪圈，至于实际实现上其实并不存在这样一个问题，具体的原因在于任何*Appear Side*的方法调用最终都会转换到*Host Side*的方法调用（比如用C\+\+实现那么最终调用的是一个C\+\+函数，用Java实现最终调用的是一个MethodHandle），具体见后文。

因此，要实现Iris的面向对象模型，首先要明白在Iris中**对象是一等公民**，一切的实现都应该围绕着对象实现。因此在*Host Side*，建议进行这样子的实现：

>1. 实现名为*IrisObject*的类，它代表了Iris中所有的对象在*Host Side*的表现形式：即__任何一个Iris对象都对应一个*IrisObject*对象__；
>2. 分别为其它字面对象实现*IrisClass、IrisModule、IrisInterface、IrisMethod、IrisClosureBlock*类，__对于类、模块、接口、方法以及块的Iris对象，除了对应IrisObject类以外，还应该分别是上述五个类的一个对象__；
>3. 在2的基础上，每个*IrisClass、IrisModule、IrisInterface、IrisMethod、IrisClosureBlock*类的对象应该有一个字段名为 *(class/module/interface/method/closureBlock)Object*，用于保存该对象实际对应的*IrisObject*对象。

<span id="s3"></span>
## 3、对象之间的关系
由上可以看出，Iris语言是由对象构成的一个庞大体系，因此对象和对象之间自然存在各种各样的关系，总结出来有如下5种：
>1. is-a/instance-of关系
>2. 组合关系
>3. 继承关系
>4. mix-in关系
>5. joint关系

<span id="s3-1"></span>
### 3.1、is-a/instance-of关系
is-a或者说instance-of是普通对象与类对象的关系，对于表达式

> a is-a B

或者

> a instance_of B

而言，表示的是对象a是否为类B的一个对象，如果是，返回*true*，否则返回*false*。其次，这里的判断进一步包含了这样一层意思：

>若类C为类B的父类，那么表达式
>
>> a is-a B
>
>或者
>
>> a instance-of B
>
>的结果，与表达式
>> a is-a C
>
>或者
>
>> a instance-of C
>
>的结果相同，但反之则不然。

<span id="s3-1-1"></span>
#### 注：Iris中的特殊is-a/instance-of关系
<span id="s3-1-1-1"></span>
##### 1. *nil/false/true*对象

在Iris中*nil/false/true*关键字分别表示**空/假/真**值，它们分别是*NilClass/FalseClass/TrueClass*类的对象且全局仅允许存在这么一个*NilClass/FlaseClass/TrueClass*对象，__也即是说*NilClass/FlaseClass/TrueClass*禁止再生成新的对象。__
<span id="s3-1-1-2"></span>
##### 2. 类对象
Iris中任何一个类都是*Class*类的对象，这一点在上面已经讨论过了，那么*Class*类这个对象是哪个类的对象呢？

答案是*Class*类本身就是*Class*类的对象。听起来似乎有些天方夜谭，但按照Iris的面向对象模型而言，确实如此。事实上*Class*类虽然是一个类，但是由于在Iris中对象是一等公民，所以*Class*类同时也就是一个对象，而它自己（站在对象的角度上来看）确实就是自己的对象（站在类的角度上来看）了。

也即是说以下表达式成立：

> Class is-a Class 或者 Class instance-of Class
<span id="s3-1-1-3"></span>
##### 3. *Object*对象
Iris中*Object*类是所有类的父类，Iris中的任何一个对象都是*Object*类的对象也即是说**对于任何对象obj，以下表达式成立**

> obj is-a *Object* 或者 obj instance-of *Object*

同时由于*Object*是一个类，因此以下表达式也成立：

> *Object* is-a *Class* 或者 *Object* instance-of *Class*

当然，更进一步的，还有下面的表达式成立：

> *Class* is-a *Object* 或者 *Class* instance-of *Object*  
> *Object* is-a *Object* 或者 *Object* instance-of *Object*

<span id="s3-2"></span>
### 2、组合关系
组合关系即对象a为对象b的一个成员，在Iris中以实例变量的形式体现。

<span id="s3-3"></span>
### 3、继承关系
继承关系是类对象与类对象之间的关系，在Iris中若类A继承类B或为B的间接子类，则有以下特征：

>1. 在类B中定义的实例方法m，类A的实例obj按以下规则访问：
>    1. 若m的访问权限为everyone，那么obj可在任何情况下访问调用m；
>    2. 若m的访问权限为relative，那么obj仅能在其可访问的方法内部访问调用m；
>    3. 若m的访问权限为personal，那么obj无法访问调用m；
>2. 任何一个类A的对象都是类B的对象，反之则不然；
>3. 在类A中删除任何一个类B中继承过来的实例方法，对于类B没有任何影响；
>4. 在类A中改变任何一个类B中继承过来的实例方法的访问权限，对于类B没有任何影响；
>5. 在类B中没有任何方法可以隐式访问类A的类方法，只能显式访问。

<span id="s3-3-1"></span>
#### 注：Iris中的特殊继承关系
>1. Class类的父类是Object类，而Object类对象是Class类的一个对象；
>2. Object类的父类是Object类，而Object类对象是一个Class类实例；
>3. 任何一个在定义时没有显示指明父类的类的父类都是Object类。

<span id="s3-4"></span>
### 4、mix-in关系
mix-in关系是类对象与模块对象之间的关系。在Iris中模块被看做是方法的集合，用于弥补Iris单继承存在的某些问题。

若类A与模块B有mix-in关系，那么存在以下特征：
>1. 在模块B中定义的所有实例方法都作为类A中定义的实例方法看待，包括其访问权限；
>2. 在类A中删除模块B中存在的实例方法对模块B没有任何影响；
>3. 在类A中修改模块B中存在的实例方法的访问权限对模块B没有任何影响；
>4. 在类A中没有任何方法可以隐式访问模块B的类方法，只能显示访问。

<span id="s3-5"></span>
### 5、joint关系
joint关系是类对象与接口对象之间的关系，joint关系主要用于实现一些约定，即功能提供方对功能使用方的一些约束。

若类A与接口B有joint关系，那么存在以下特征：
>1. 类A必须实现接口B中定义要实现的所有实例方法，并且需要保证方法名、方法参数个数以及是否有可变参数一一对应；
>2. 如果存在未对应的情况，那么Iris主动抛出异常。

<span id="s4"></span>
## 4、对象和方法的关系
<span id="s4-1"></span>
### 1、实例方法和类方法
在Iris语法上来看存在两类方法：实例方法和类方法。其中实例方法是一个类的对象可以调用的方法，而类方法则是类本身可以调用的方法。然而在这里所谓的实例方法和类方法仅仅是一个语法糖衣，对于Iris对象模型而言，只存在实例方法而没有类方法。

那么类方法到哪儿去了呢？类方法事实上就是类对象的实例方法。

考察以下代码：

```
class MyClass {
    fun instance_method() {
        // do something
    }
    
    fun self.class_method() {
        // do something
    }
}

;obj = MyClass.new()
;obj.instance_method()
;MyClass.class_method()

```

这里为*MyClass*定义了实例方法*instance_method*和类方法*class_method*，*MyClass*调用了自己的类方法*new*和*class_method*，*MyClass*的实例调用了它的实例方法*instance_method*。而事实上定义*class_method*的时候是把*class_method*注册成了*MyClass*常量所保存的那个类对象的实例方法，因此事实上`MyClass.class_method()`这段代码__仍旧是一个对象调用实例方法的过程。__

同理，对于模块而言，模块也可以调用它定义的类方法，但事实上用模块调用类方法的过程依旧如上。

因此，在实现上，建议*IrisClass/IrisModule*的*RegistClassMethod(method)* 方法都转为将*method*注册到*IrisClass/IrisModule*的 *(class/module)Object*的实例方法上。

而在调用方法的时候，就统一用对象去调用实例方法就行了，免去了区分类方法和实例方法的麻烦。

<span id="s4-2"></span>
### 2、 实例方法的存储与查找
在Iris中，任何方法都是*Method*类的一个对象，按照前面的建议，这个对象在*Host Side*保存在*methodObject*中，而真正和*Host Side*挂上关系的其实是*IrisMethod*这个类，保存到*IrisClass、IrisModule*中的，也是*IrisMethod*对象。

对于存储方式，建议使用*key-value*的方式，在*IrisObject*中使用一个类似*HashMap*的结构进行存储，比如在类中定义了一个名为*foo*的实例方法，那么就在这个*HashMap*中按照 *"foo"=> object*的形式进行保存，查找的时候直接使用 *"foo"*查找即可。

>注：Iris每一个对象都可以拥有自己的实例方法，考虑以下代码：
>```
>class MyClass {
>     fun foo() {
>         // do something
>     }
>}
>;a = MyClass.new()
>;b = MyClass.new()
>;class >> b {
>     fun foo2() {
>         // do something
>     }
>  }
>;a.foo()
>;b.foo()
>;b.foo2()
>;a.foo2() // Error
>```
>
>这里单独为实例b定义了*foo2*方法，而虽然a、b同属于一个类*MyClass*，但是a则没有*foo2*方法可以调用。


对于一个实例方法的调用：

>`;obj.foo()`

Iris将会按照以下顺序进行查找：

>1. 查找obj自己的实例方法，如果有调用之，否则进入2；
>2. 在obj的类中查找实例方法，如果有调用之，否则进入3；
>3. 在和obj有mix-in关系的模块中查找实例方法，如果有调用之，否则进入4；
>4. 在obj的父类、间接父类中按照2和3的顺序进行查找，如果有调用之，否则进入5；
>5. 查找到Object类如果还是没有查找到，那么Iris主动调用obj.missing_method()方法。

<span id="s4-3"></span>
### 补充：类方法的查找顺序
Iris中任何类都是*Class*类的对象，这里以*new*方法为例进行一个介绍：

考虑代码：

```
class MyClass {}
;obj = MyClass.new()
```

当要调用*new*方法的时候，Iris做以下工作：

>1. 在*MyClass*对应的*IrisObject*中查找是否有名为*new*的实例方法，如果有，调用之，否则进入2；
>2. *MyClass*是*Class*类的一个实例，因此Iris进入*Class*类定义的实例方法中进行查找，此时查找到有该方法存在，调用之。

此时把*MyClass*换成*Class*，那么对于`;Class.new()`的调用过程和上述完全一致。

进一步的，由于Iris中任何类都是*Object*类的子类，因此如果*MyClass*想要调用*Object*类的方法比如*to_string*，那么*MyClass*不会在*Class*类中查找到该方法，进而转到*Class*类的父类*Object*中查找到该方法然后调用之。

我们总结一下，关于类对象可以调用的方法如下：

>1. 类对象自己的实例方法（也即类中定义的类方法）；
>2. *Class*类的中定义的实例方法；
>3. *Object*类中定义的实例方法。

<span id="s5"></span>
## 5、实例变量和类变量
在Iris中以@开头的变量为实例变量，以@@开头的变量为类变量。

实例变量，顾名思义，即是一个实例单独享有的变量，不和本类的其他实例共享；而类变量则是类（或者模块）拥有的变量，所有的类的实例都共享同一个类变量。此外，所有子类以及间接子类中都享有父类（或者父类的模块）中出现的类变量。

类变量是一种特殊的变量，本质上来讲它是一个语法糖衣，严格来讲并不属于Iris对象模型的任何一个方面，Iris提供类变量仅仅是出于方便的考虑，事实上类变量与Iris对象模型是格格不入的。

在实现上建议每个*IrisObject*对象都拥有一个类似于*HashMap*的结构用于能够以*key-value*的形式保存实例变量的名字和相应的值。

而类变量仅能够存在于*IrisClass和IrisModule*中，也建议以*key-value*的形式保存。

<span id="s6"></span>
## 6、语言扩展
Iris所拥有的功能在于附加于其上的扩展，一般在*Host Side*进行扩展，而对于Iris而言，可以扩展的有*类、模块以及接口*。

扩展的编写是将*Host Side*连接到*Appear Side*的过程，比如如果要为Iris扩展一个类，那么就需要在*Host Side*先编写相应的类功能，然后再连接到*Appear Side*，使得*Appear Side*能够展现*Host Side*的功能。

一种可行的同时也是建议的实现方式是通过*Native Object*的形式进行绑定扩展。这里的*Native Object*指的是*Host Side*的*Object*。比如如果要在*Appear Side*生成一个*String*，那么相应的就需要在*Host Side*生成一个*Native String*（在C\+\+上可能以stl::string展示而在Java上可能以String展示），然后将*Host Side*的对象绑定到*Appear Side*上，更具体地来说，就是将*Host Side*的对象绑定到*IrisObject*上，为此需要在*IrisObject*中添加一个*nativeObject*字段，用于存储这个*Host Side*的Object。（在C\+\+上可能使用*void\**在Java上可能使用*Object*）。

更进一步的，完成了*Host Side*到*Appear Side*的对象绑定之后，还需要完成方法的注册（绑定）。至于方法的注册，一种可行且建议的方法是让*IrisMethod*去保存__在*Host Side*用以表示一个函数或者方法的数据结构__，比如在C\+\+中是函数指针而在Java中是MethodHandle，它拥有类似的签名：

    IrisValue MethodName(IrisValue self, List<IrisValue> parameters, List<IrisValue> variableParameters, ...)
    
其中*self*表示调用者，*parameters*表示传进来的固定参数，*variableParameters*表示传进来的可变参数。

>注：这里的*IrisValue*表示在*Host Side*用于保存*IrisObject*的数据结构，为了数据交换方便一般不直接使用*IrisObject*。

当完成了这样子的*类似函数或者方法的数据结构*，就可以将它保存到一个新的*IrisMethod*中，为此*IrisMethod*应该有一个字段用来保存这个数据结构。

>注：在Iris中应该存在两类方法，一类是按照上面的说明直接在*Host Side*定义出来的*Native Method*，而另外一类，则是在*Appear Side*即在脚本代码中定义的*User Method*（用*fun*语句）。因此调用的时候，应该区分进行。但是可以确定的是，一个方法的最终调用都会归为*Native Method*的调用。