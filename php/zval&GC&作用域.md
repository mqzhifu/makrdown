
# zval / 变量

PHP中变量类型：

- 分类：标量类型、复杂类型、特殊类型。
- 标量：int string bool float
- 复杂：array object
- 特殊：null 资源

所有的变量，包括：简单类型、复杂类型，到最后的 ZEND 执行的时候，都是申请一个 zval 结构体保存。

```C
  typedef struct _zval_struct {
    zvalue_value value;
    zend_uint refcount;
    zend_uchar type;
    zend_uchar is_ref;
  } zval;

```

```c
typedef union _zvalue_value {
    long lval;
    double dval;
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;
    zend_object_value obj;
} zvalue_value;

```

zval 中的 type 字段，标识该变量的类型，如：

- type = IS_LONG;//整形 -> lval
- type = IS_BOOL;//布尔值 -> lval
- type = IS_STRING //字符串 -> str
- type = IS_ARRAY //数组 -> \*ht
- type = IS_RESOURCE //文件资源|obj  -> obj

这里又包含了一个 union 结构体，实际上就是通过这个结构体，实现了弱类型。

#### DEMO：

$b= 3;

1、申请一个zval结构体
2、3是个整形，于是，zval.type = long
3、union结构体最后给了value变量，给变量赋值value.long = 3

字符串类型的会用 union 中的str保存

bool、资源、int 通用： lval存

数组:HashTable \*ht
对象：zend_object_value

神奇的\<弱类型\>就这样被完整的实现了。

# GC

回收主要是内存，内存即变量，其中：复合型变量，如：数组、对象等，占资源大，非常需要回收。

zval 结构体中，还有refcount\_\_gc is\_ref\_\_gc 两个成员属性没有介绍

refcount\_\_gc ：有多少个变量在引用，如:

$a=1;//refcount\_\_gc = 1,is\_ref\_\_gc = false

$b=$a;//refcount\_\_gc = 2,is\_ref\_\_gc = false

实际上引擎是不会开辟两个内存空间来存的，除非其中一个变量值发生了变化，也就是写时复制，copy on wtrit.

is\_ref\_\_gc ：是否被引用，简单说，是否有使用&，如：

$a=1;//refcount\_\_gc = 1,is\_ref\_\_gc = false

$b= &$a;//refcount\_\_gc = 2,is\_ref\_\_gc = true

当refcount\_\_gc =0 时就会被删除，unset 就是累减refcount\_\_gc

$a = array\( 'one' \);

$a\[\] =& $a;

数组$a 的第2个元素是自身的引用，成了 循环递归。无法被GC回收，内存泄漏。

5.3以前，就是这样，之后的版本做了修改，有如下3条：

1    如果一个zval的refcount增加，那么此zval还在使用，不属于垃圾

2    如果一个zval的refcount减少到0， 那么zval可以被释放掉，不属于垃圾

3    如果一个zval的refcount减少之后大于0，那么此zval还不能被释放，此zval可能成为一个垃圾。

GC把这个变量移入到GC buffer 中，等待buffer满了，再统一做清理。

总结以上：unset 的时候，会触发GC

只有在准则3下，GC才会把zval收集起来。

A 为了避免每次变量的refcount减少的时候都调用GC的算法进行垃圾判断，此算法会先把所有前面准则3情况下的zval节点放入一个节点\(root\)缓 冲区\(root buffer\)，

并且将这些zval节点标记成紫色，同时算法必须确保每一个zval节点在缓冲区中之出现一次。当缓冲区被节点塞满的时候，GC才开始开 始对缓冲区中的zval节点进行垃圾判断。

B：当缓冲区满了之后，算法以深度优先对每一个节点所包含的zval进行减1操作，为了确保不会对同一个zval的refcount重复执行减1操作，一 旦zval的refcount减1之后会将zval标记成灰色。

需要强调的是，这个步骤中，起初节点zval本身不做减1操作，但是如果节点zval中包 含的zval又指向了节点zval（环形引用），那么这个时候需要对节点zval进行减1操作。

C：算法再次以深度优先判断每一个节点包含的zval的值，如果zval的refcount等于0，那么将其标记成白色\(代表垃圾\)，如果zval的 refcount大于0，那么将对此zval以及其包含的zval进行refcount加1操作，这个是对非垃圾的还原操作，同时将这些zval的颜色变 成黑色（zval的默认颜色属性）。

D：遍历zval节点，将C中标记成白色的节点zval释放掉。

# 作用域

symble\_table：保存全局变量，具体是，全局变量名和指向该变量名值的指针\(指向zval\)。

该结构体内还包含一个：active\_symble\_table，创建局部变量的时候，会往这个表里添加。

两个结构体，都是用hashTable结构存储。

实际上，我们每创建一个变量，最后都要加到symble\_table或active\_symble\_table。也就是说：一个函数内的所有变量是可以遍历的。

以上两个结构体保存于executor\_globals结构体中。

# 内存管理

主要工作就是维护三个列表：小块内存列表（free\_buckets）、大块内存列表（large\_free\_buckets）和剩余内存列表（rest\_buckets）。

ZEND提供了几个内存申请释放的接口，应用层在申请内存，如：定义一个变量，是通过这些接口向OS申请内存，但不是用多少申请多少，而是一次申请略大一点的内存，释放的时候，也不是直接向OS释放，而是把这部分内存放到剩余内存列表中去。

申请内存，逻辑如下：

![ae016389f34b7982f9004a7de0c44c55.jpg](image/ae016389f34b7982f9004a7de0c44c55.jpg)

![c6b456795c8a1a2c710feaf778efb5b9.jpg](image/c6b456795c8a1a2c710feaf778efb5b9.jpg)

先确定是大/小，如果是小，在小内存中，找不到，就去大中找，如果大找不到，就去空闲中找，如果还没有，就找OS要。

销毁变量

当执行unset 发现refcount=0，要归还内存的时候，就是把这段内存地址，放到rest\_buckets

实际发现：PHP 会先到OS，申请在大块内存，然后统计管理这块内存，这期间如果再有内存申请，直接从已申请的内存中操作，提升执行速度。缺点：一但某个请求执行时间过长，或者长驻内存，会导致大块内存长时间占用。但WEB开发就是一次性请求，然后全释放。
