---
title: 函数式编程与闭包
date: 2023-04-01 20:43:26
categories: 
- Python
tags:
- 函数式编程
- 闭包
---

## 函数式编程

**函数式编程**这个概念我们可能或多或少都听说过，刚听说的时候不明觉厉，觉得这是一个非常黑科技的概念。但是实际上它的含义很朴实，但是延伸出来许多丰富的用法。

在早期编程语言还不是很多的时候，我们会将语言分成**高级语言与低级语言**。比如汇编语言，就是低级语言，几乎什么封装也没有，做一个赋值运算还需要我们手动调用寄存器。而高级语言则从这些面向机器的指令当中抽身出来，转而面向过程或者是对象。也就是说我们写代码面向的是一段计算过程或者是一个计算机当中抽象出来的对象。如果你学过面向对象，你会发现和面向过程相比，面向对象的抽象程度更高了一些，做了更加完善的封装。

在面向对象之后呢，我们还可以做什么封装和抽象呢？这就轮到了函数式编程。

函数我们都了解，就是我们定义的一段程序，它的输入和输出都是确定的。我们把一段函数写好，它可以在任何地方进行调用。既然函数这么好用，那么能不能**把函数也看成是一个变量进行返回和传参**呢？

OK，这个就是函数式编程最直观的特点。也就是说我们写的一段函数也可以作为变量，既可以用来赋值，还可以用来传递，并且还能进行返回。这样一来，大大方便了我们的编码，但是这并不是有利无害的，相反它带来许多问题，最直观的问题就是由于函数传入的参数还可以是另一个函数，这会**导致函数的计算过程变得不可确定**，许多超出我们预期的事情都有可能发生。

所以函数式编程是有利有弊的，它的确简化了许多问题，但也产生了许多新的问题，我们在使用的过程当中需要谨慎。

## 传入、返回函数

在我们之前介绍`filter`、`map`、`reduce`以及自定义排序的时候，其实我们已经用到了函数式编程的概念了。

比如在我们调用sorted进行排序的时候，如果我们传入的是一个对象数组，我们希望根据我们制定的字段排序，这个时候我们往往需要传入一个**匿名函数**，用来制定排序的字段。其实传入的匿名函数，其实就是函数式编程最直观的体现了：

```python
sorted(kids, key=lambda x: x['score'])
```

除此之外，我们还可以返回一个函数，比如我们来看一个例子：

```python
def delay_sum(nums):
    def sum():
        s = 0
        for i in nums:
            s += i
        return s
    return sum
```

如果这个时候我们调用`delay_sum`传入一串数字，我们会得到什么？

答案是一个函数，我们可以直接输出，从打印信息里看出这一点：

```python
>>> delay_sum([1, 3, 4, 2])
<function delay_sum.<locals>.sum at 0x1018659e0>
```

我们想获得这个运算结果应该怎么办呢？也很简单，我们用一个变量去接收它，然后执行这个新的变量即可：

```python
>>> f = delay_sum([1, 3, 4, 2])
>>> f()
10
```

这样做有一个好处是我们可以**延迟计算**，如果不使用函数式编程，那么我们需要在调用`delay_sum`这个函数的时候就计算出结果。如果这个运算量很小还好，如果这个运算量很大，就会造成开销。并且当我们计算出结果来之后，这个结果也许不是立即使用的，可能到很晚才会用到。既然如此，我们返回一个函数代替了运算，当后面真正需要用到的时候再执行结果，从而延迟了运算。这也是很多计算框架的常用思路，比如**spark**。

## 闭包

我们再来回顾一下我们刚才举的例子，在刚才的`delay_sum`函数当中，我们内部实现了一个`sum`函数，我们在这个函数当中调用了`delay_sum`函数传入的参数。这种对**外部作用域**的变量进行引用的内部函数就称为**闭包**。

其实这个概念很形象，因为这个函数内部调用的数据对于调用方来说是封闭的，完全是一个黑盒，除非我们查看源码，否则我们是不知道它当中数据的来源的。除了不知道来源之外，更重要的是它引用的是外部函数的变量，既然是变量就说明是动态的。也就是说我们**可以通过改变某些外部变量的值来改变闭包的运行效果**。

这么说有点拗口，我们来看一个简单的例子。在Python当中有一个函数叫做`math.pow`其实就是计算次方的。比如我们要计算`x`的平方，那么我们应该这样写：

```python
math.pow(x, 2)
```

但是如果我们当前场景下只需要计算平方，我们每次都要传入额外再传入一个2会显得非常麻烦，这个时候我们使用闭包，可以简化操作：

```python
def mypow(num):
    def pw(x):
        return math.pow(x, num)
    return pw
    
pow2 = mypow(2)
print(pow2(10))
```

通过闭包，我们**把第二个变量给固定了**，这样我们只需要使用`pow2`就可以实现原来`math.pow(x, 2)`的功能了。如果我们突然需求变更需要计算3次方或者是4次方，我们只需要修改`mypow`的传入参数即可，完全不需要修改代码。

实际上这也是闭包最大的使用场景，我们可以**通过闭包实现一些非常灵活的功能**，以及通过配置修改一些功能等操作，而不再需要通过代码写死。要知道对于工业领域来说，线上的代码是不能随便变更的，尤其是客户端，比如apple store或者是安卓商店当中的软件包，只有用户手动更新才会拉取。如果出现问题了，几乎没有办法修改，只能等用户手动更新。所以常规操作就是使用一些类似闭包的灵活功能，**通过修改配置的方式改变代码的逻辑**。

除此之外闭包还有一个用处是可以**暂存变量或者是运行时的环境**。

举个例子，我们来看下面这段代码：

```python
def step(x=0):
    x += 5
    return x
```

这是没有使用闭包的函数，不管我们调用多少次，答案都是5，执行完`x+=5`之后的结果并不会被保存起来，当函数返回了，这个暂存的值也就被抛弃了。那如果我**希望每次调用都是依据上次调用的结果**，也就是说我们每次修改的操作都能保存起来，而不是丢弃呢？

这个时候就需要使用闭包了：

```python
def test(x=0):
    def step():
        nonlocal x
        x += 5
        return x
    return step
    
t = test()
t()
>>> 5
t()
>>> 10
```

也就是说我们的`x`的值被存储起来了，**每次修改都会累计**，而不是丢弃。这里需要注意一点，我们用到了一个新的关键字叫做**nonlocal**，这是Python3当中独有的关键字，用来申明当前的变量`x`不是局部变量，这样Python解释器就会去全局变量当中去寻找这个`x`，这样就能关联上`test`方法当中传入的参数x。Python2官方已经不更新了，不推荐使用。

由于在Python当中也是一切都是对象，如果我们**把闭包外层的函数看成是一个类**的话，其实闭包和类区别就不大了，我们甚至可以给闭包返回的函数关联函数，这样几乎就是一个对象了。来看一个例子：

```python
def student():
    name = 'xiaoming'
    
    def stu():
        return name
        
    def set_name(value):
        nonlocal name
        name = value
        
    stu.set_name = set_name
    return stu
    
stu = student()
stu.set_name('xiaohong')
print(stu())
```

最后运算的结果是`xiaohong`，因为我们调用`set_name`改变了闭包外部的值。这样当然是可以的，但是一般情况下我们并不会用到它。和写一个`class`相比，通过闭包的方法**运算速度会更快**。原因比较隐蔽，是因为闭包当中没有`self`指针，从而节省了大量的变量的访问和运算，所以计算的速度要快上一些。但是闭包搞出来的伪对象是**不能使用继承、派生**等方法的，而且和正常的用法格格不入，所以我们知道有这样的方法就可以了，现实中并不会用到。

## 闭包的坑

闭包虽然好用，但是不小心的话也是很容易踩坑的，下面介绍几个常见的坑点。

### 闭包不能直接访问外部变量

这一点我们刚才已经提到了，在闭包当中我们**不能直接访问外部的变量**的，必须要通过nonlocal关键字进行标注，否则的话是会报错的。

```python
def test():
    n = 0
    def t():
        n += 5
        return n
    return t
```

比如这样的话，就会报错：


![](https://mmbiz.qpic.cn/mmbiz_png/4lVbQH4ShicUH5Q5NIVsAdDjMiaw795L44EwzeJicNRQu45vvgjCjCPiaHHhuudDK38YciaFDqzLHT90KBBdTWMw9YA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


### 闭包当中不能使用循环变量

闭包有一个很大的问题就是**不能使用循环变量**，这个坑藏得很深，因为单纯从代码的逻辑上来看是发现不了的。也就是说逻辑上没问题的代码，运行的时候往往会出乎我们的意料，这需要我们对底层的原理有深刻地了解才能发现，比如我们来看一个例子：


```python
def test(x):
    fs = []
    for i in range(3):
        def f():
            return x + i
        fs.append(f)
    return fs


fs = test(3)
for f in fs:
    print(f())
```

在上面这个例子当中，我们使用了`for`循环来创建了3个闭包，我们使用`fs`存储这三个闭包并进行返回。然后我们通过调用`test`，来获得了这3个闭包，然后我们进行了调用。

这个逻辑看起来应该没有问题，按照道理，这3个闭包是通过for循环创建的，并且在闭包当中我们用到了循环变量`i`。那按照我们的想法，最终输出的结果应该是`[3, 4, 5]`，但是很遗憾，最后我们得到的**结果是[5, 5, 5]**。

看起来很奇怪吧，其实一点也不奇怪，因为**循环变量`i`并不是在创建闭包的时候就set好的**。而是当我们执行闭包的时候，我们再去寻找这个`i`对应的取值，显然当我们运行闭包的时候，循环已经执行完了，此时的`i`停在了2。所以这3个闭包的执行结果都是2+3也就是5。这个坑是由Python解释器当中对于闭包执行的逻辑导致的，我们编写的逻辑是对的，但是它并不按照我们的逻辑来，所以这一点要千万注意，如果忘记了，想要通过debug查找出来会很难。

## 总结

虽然从表面上闭包存在一些问题和坑点，但是它依然是我们**经常使用的Python高级特性**，并且它也是很多其他高级用法的基础。所以我们理解和学会闭包是非常有必要的，千万不能因噎废食。

其实并不只是闭包，很多高度抽象的特性都或多或少的有这样的问题。因为当我们进行抽象的时候，我们固然简化了代码，增加了灵活度，但与此同时我们也让**学习曲线变得陡峭**，带来了更多我们需要理解和记住的内容。本质上这也是一个trade-off，好用的特性需要付出代码，易学易用的往往意味着比较死板不够灵活。对于这个问题，我们需要保持心态，不过好在初看时也许有些难以理解，但总体来说闭包还是比较简单的，我相信对你们来说一定不成问题。
