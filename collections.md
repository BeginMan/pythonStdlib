
Python在一些内置的数据类型，比如str, int, list, tuple, dict等，之后又提供了比较高级的额外的**数据类型**， 共有以下几种：`Counter`，`deque`，`defaultdict`，`namedtuple`，`OrderedDict`

那么接下来一个个的攻克它们。

# 一.namedtuple
namedtuple的函数原型如下：

    def namedtuple(typename, field_names, verbose=False, rename=False):
        """Returns a new subclass of tuple with named fields.

作用就是通过将可迭代对象设置字段名，可使用名称来访问元素的数据对象。

比较重要参数释义：

- typename: 自定义名，字符串类型
- field_names： 字段名，list类型


如下例子：

    >>> Point = namedtuple('Point', ['x', 'y'])
    >>> Point.__doc__                   # 新类的文档字符串
    'Point(x, y)'

    >>> p = Point(11, y=22)             # 通过位置参数或关键字参数实例化
    >>> p[0] + p[1]                     # 像普通元组一样使用索引
    33

    >>> x, y = p                        # 像普通元组一样解包
    (11, 22)

    >>> p.x + p.y                       # 通过字段名访问
    33

    >>> d = p._asdict()                 # 转换为字典
    >>> d['x']
    11

    >>> Point(**d)                      # 从字典转换过来
    Point(x=11, y=22)

    >>> p._replace(x=100)               # _replace() is like str.replace() but targets named fields
    Point(x=100, y=22)

再举一个🌰：

    from collections import namedtuple

    tuples = [
        ('Bman', 22, 'Python'),
        ('Jack', 24, 'C'),
    ]

    p = namedtuple("Code", ['name', 'age', 'language'])
    print p             # <class '__main__.Code'>
    print p._fields     # ('name', 'age', 'language')
    print p.__doc__     # Code(name, age, language)

    for i in tuples:
        _i = p._make(i)
        print i, _i, _i.name, _i.age, _i.language

    # ('Bman', 22, 'Python') Code(name='Bman', age=22, language='Python') Bman 22 Python
    # ('Jack', 24, 'C') Code(name='Jack', age=24, language='C') Jack 24 C

# 二.deque
deque其实是 double-ended queue 的缩写，翻译过来就是双端队列，它最大的好处就是实现了从队列 头部快速增加和取出对象: `.popleft()`, `.appendleft()` 。该类的原型如下：

    deque([iterable[, maxlen]]) --> deque object

该类有以下方法：

- `append`: 添加元素到右侧的双端队列
- `appendleft`: 添加元素到左侧的双端队列
- `clear`: 移除所有元素
- `count`: D.count(value) -> integer, 返回出现的值数
- `extend`: 通过可迭代元素扩展右侧的双端队列
- `extendleft`: 与上相反
- `pop`:删除并返回最右边的元素
- `popleft`:与上相反
- `remove`:D.remove(value). 移除第一次出现的值
- `reverse`:取反
- `rotate`: rotate是回转的意思，旋转双端队列n步向右（默认值n=1）。如果n是负的，向左旋转时

虽然原生的list也可以从头部添加和取出对象等方法：
   
    lis.insert(0, v)
    lis.pop(0)

但是与list不同的是**list对象的这两种用法的时间复杂度是 O(n) ，也就是说随着元素数量的增加耗时呈线性上升。而使用deque对象则是 O(1) 的复杂度，所以当你的代码有这样的需求的时候， 一定要记得使用deque。**

我们可以创建一个空的deque对象：

    d = deque()
    # deque([])

然后进行操作：

    d.append(1)
    d.append('2')
    print d, len(d)     # deque([1, '2']) 2
    print d[0]          # 1

    d.appendleft('a')
    print d             # deque(['a', 1, '2'])

    print d.count('a')  # 1

    d.extend(['a', 'b', 'c'])
    print d             # deque(['a', 1, '2', 'a', 'b', 'c'])

    d.extendleft(['m', 'n'])
    print d             # deque(['n', 'm', 'a', 1, '2', 'a', 'b', 'c'])

    print d.pop()       # c
    print d.popleft()   # n

    print d             # deque(['m', 'a', 1, '2', 'a', 'b'])
    d.remove('a')
    print d             # deque(['m', 1, '2', 'a', 'b'])

    d = deque('abcde')
    print d
    for i in range(len(d)):
        d.rotate()
        print i, d

    # 输出：

    deque(['a', 'b', 'c', 'd', 'e'])
    0 deque(['e', 'a', 'b', 'c', 'd'])
    1 deque(['d', 'e', 'a', 'b', 'c'])
    2 deque(['c', 'd', 'e', 'a', 'b'])
    3 deque(['b', 'c', 'd', 'e', 'a'])
    4 deque(['a', 'b', 'c', 'd', 'e'])

下面例子实现一个跑马灯效果：

    import sys
    import time
    from collections import deque

    loading = deque('>--------------------')

    while True:
        print '\r%s' % ''.join(loading),        # 注意："\r"表示回车（将光标移至本行开头）这里不能省略后面的逗号
        loading.rotate()        # 默认1
        sys.stdout.flush()     
        time.sleep(0.08)

注意：

- CR+LF (`\r\n`);
- LF (`\n`);
- CR (`\r`).

# 三.Counter
实现计数功能

    >>> c = Counter('abcdeabcdabcaba')  # count elements from a string

    >>> c.most_common(3)                # 出现最多的三个元素
    [('a', 5), ('b', 4), ('c', 3)]
    
    >>> sorted(c)                       # 列出所有唯一元素
    ['a', 'b', 'c', 'd', 'e']
    
    >>> c.elements()                    # 迭代器
    <itertools.chain at 0x1094c0e50>
    >>> sorted(c.elements())            # 列出所有元素
    ['a', 'a', 'a', 'a', 'a', 'b', 'b', 'b', 'b', 'c', 'c', 'c', 'd', 'd', 'e']

    >>> ''.join(sorted(c.elements()))   
    'aaaaabbbbcccdde'
    
    >>> sum(c.values())                 # total of all counts
    15

    >>> c['a']                          # 元素a出现次数
    5
    
    >>> for elem in 'shazam':           # 通过可迭代对象更新counts
    ...     c[elem] += 1                # by adding 1 to each element's count
    >>> c['a']                          # now there are seven 'a'
    7
    
    >>> del c['b']                      # remove all 'b'
    >>> c['b']                          # now there are zero 'b'
    0

    >>> d = Counter('simsalabim')       # make another counter
    >>> c.update(d)                     # add in the second counter
    >>> c['a']                          # now there are nine 'a'
    9

    >>> c.clear()                       # empty the counter
    >>> c
    Counter()

    Note:  If a count is set to zero or reduced to zero, it will remain
    in the counter until the entry is deleted or the counter is cleared:

    >>> c = Counter('aaabbc')
    >>> c['b'] -= 2                     # reduce the count of 'b' by two
    >>> c.most_common()                 # 'b' is still in, but its count is zero
    [('a', 3), ('c', 1), ('b', 0)]


# 四.OrderedDict
OrderedDict相对于dict来说也就是有序字典。用法与dict类似，它有如下常用方法：

- `clear`:od.clear() -> None.  Remove all items from od
- `keys`: 同dict
- `values`: 同dict
- `items`: 同dict
- `iterkeys`: 返回一个keys迭代器
- `itervalues`:返回一个values迭代器
- `iteritems`: 返回一个(key, value)键值对迭代器
- `pop`:od.pop(k[,d]) -> v, 删除指定键和返回对应的值。如果无则触发KeyError异常
- `setdefault`:od.setdefault(k[,d]) -> od.get(k,d), also set od[k]=d if k not in od

如下例子：

    from collections import OrderedDict

    items = (
        ('A', 1),
        ('B', 2),
        ('C', 3)
    )

    regular_dict = dict(items)
    ordered_dict = OrderedDict(items)

    # 无序
    for k, v in regular_dict.items():
        print k, v

    # 有序
    for k, v in ordered_dict.items():
        print k, v


    # Result:
    Regular Dict:
    A 1
    C 3
    B 2
    Ordered Dict:
    A 1
    B 2
    C 3

# 五.defaultdict
Python原生的数据结构dict的时候，如果用 `d[key]` 这样的方式访问， 当指定的key不存在时，是会抛出KeyError异常的。如果使用defaultdict，只要你传入一个默认的工厂方法，那么请求一个不存在的key时， 便会调用这个工厂方法使用其结果来作为这个key的默认值。

    members = [
        # Age, name
        ['male', 'John'],
        ['male', 'Jack'],
        ['female', 'Lily'],
        ['male', 'Pony'],
        ['female', 'Lucy'],
    ]

    result = defaultdict(list)
    print result                # defaultdict(<type 'list'>, {})

    for sex, name in members:
        result[sex].append(name)        # 将sex做key，value为list类型，append(name)

    print result                        # defaultdict(<type 'list'>, {'male': ['John', 'Jack', 'Pony'], 'female': ['Lily', 'Lucy']})

**defaultdict(list)的用法和dict.setdefault(key, [])比较类似**，上述代码使用setdefault实现如下：

    result = {}
    for sex, name in members:
        result.setdefault(sex, []).append(name)

    print result

大部分内容看源码就弄明白了，同时也参考了[不可不知的Python模块: collections](http://www.zlovezl.cn/articles/collections-in-python/)

