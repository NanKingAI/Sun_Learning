[toc]

## 函数的参数

`*args`是可变参数，args接收的是一个tuple；

`**kw`是关键字参数，kw接收的是一个dict。

以及调用函数时如何传入可变参数和关键字参数的语法：

可变参数既可以直接传入：`func(1, 2, 3)`，又可以先组装list或tuple，再通过`*args`传入：`func(*(1, 2, 3))`；

关键字参数既可以直接传入：`func(a=1, b=2)`，又可以先组装dict，再通过`**kw`传入：`func(**{'a': 1, 'b': 2})`。

命名的关键字参数是为了限制调用者可以传入的参数名，同时可以提供默认值。

定义命名的关键字参数在没有可变参数的情况下不要忘了写分隔符`*`，否则定义的将是位置参数。

## 高级特性

### 列表生成式

在一个列表生成式中，for前面的if ... else是表达式，而for后面的if是过滤条件，不能带else

```python
[x for x in range(1, 11) if x % 2 == 0]

[x if x % 2 == 0 else -x for x in range(1, 11)]

```

```python
L1 = ['Hello', 'World', 18, 'Apple', None]
L2 = [s.lower() for s in L1 if isinstance(s, str)]
```

全排列

```py
[m + n for m in 'ABC' for n in 'XYZ']
```

对字典，提取key value

```python
d = {'x': 'A', 'y': 'B', 'z': 'C' }
[k+'='+v for k,v in d.items()]
```

### 生成器

斐波那契：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

杨辉三角：

```py
def triangles():
    a=[1]
    yield a;
    triangle=[1,1]
    while True:
        yield triangle
        a=triangle
        triangle = [1, 1]
        for i in list(range(1,len(a))):
            triangle.insert(i,a[i]+a[i-1])
```



## 高阶函数

### mapReduce

```python
# str 转int
DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def char2num(s):
    return DIGITS[s]

def str2int(s):
    return reduce(lambda x, y: x * 10 + y, map(char2num, s))

```



### filter

通过过滤器返回素数

```python
def _odd_iter():
    n = 1
    while True:
        n = n + 2
        yield n

def _not_divisible(n):
    #一种遍历方式
    return lambda x: x % n > 0



def primes():
    yield 2
    it=_odd_iter()
    while True:
        n=next(it)
        yield n
        it=filter(_not_divisible(n),it)

#打印素数
for num in primes():
    if(num<1000):
        print(num)
    else:
        break;
```



## 返回函数

### 闭包

```python
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()
>>> f1()
9
>>> f2()
9
>>> f3()
9
```

返回的函数引用了变量`i`，但它并非立刻执行。等到3个函数都返回时，它们所引用的变量`i`已经变成了`3`，因此最终结果为`9`

### 闭包实现计数器

```python
def createCounter():
    i=0
    def counter():
        nonlocal i
        i=i+1
        return i
    return counter

```
## 匿名函数

关键字`lambda`表示匿名函数，冒号前面的`x`表示函数参数。

```
L = list(filter(lambda x: x%2>0, range(1, 20)))
```


## 协程

### 无锁 生产者-消费者

注意到consumer函数是一个generator，把一个consumer传入produce后：

首先调用c.send(None)启动生成器；
 然后，一旦生产了东西，通过c.send(n)切换到consumer执行；
consumer通过yield拿到消息，处理，又通过yield把结果传回；
produce拿到consumer处理的结果，继续生产下一条消息；
produce决定不生产了，通过c.close()关闭consumer，整个过程结束。

```python

def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)

```



## 正则表达式

- `[0-9a-zA-Z\_]`可以匹配一个数字、字母或者下划线；
- `[0-9a-zA-Z\_]+`可以匹配至少由一个数字、字母或者下划线组成的字符串，比如`'a100'`，`'0_Z'`，`'Py3000'`等等；
- `[a-zA-Z\_][0-9a-zA-Z\_]*`可以匹配由字母或下划线开头，后接任意个由一个数字、字母或者下划线组成的字符串，也就是Python合法的变量；
- `[a-zA-Z\_][0-9a-zA-Z\_]{0, 19}`更精确地限制了变量的长度是1-20个字符（前面1个字符+后面最多19个字符）。

`A|B`可以匹配A或B，所以`(P|p)ython`可以匹配`'Python'`或者`'python'`。

`^`表示行的开头，`^\d`表示必须以数字开头。

`$`表示行的结束，`\d$`表示必须以数字结束。

```python
## 邮箱验证
def is_valid_email(addr):

    pattern=re.compile(r'^(?!.*[._%]{2})[a-zA-Z0-9]+(?:[._%][a-zA-Z0-9]+)*@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*\.[a-zA-Z]{2,}$')

    return re.match(pattern,addr)
```

