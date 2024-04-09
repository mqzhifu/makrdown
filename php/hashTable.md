# hashTable

```c
typedef struct _hashtable { 
    uint nTableSize;        // hash Bucket的大小，最小为8，以2x增长。
    uint nTableMask;        // nTableSize-1， 掩码，用于根据hash值计算存储位置。
    uint nNumOfElements;    // hash Bucket中当前存在的元素个数，count()函数会直接返回此值 
    ulong nNextFreeElement; // 下一个数字索引的位置，$arr[] = "hello"时会用到
    Bucket *pInternalPointer;   // 当前遍历的指针（foreach比for快的原因之一）
    Bucket *pListHead;          // 存储数组头元素指针
    Bucket *pListTail;          // 存储数组尾元素指针
    Bucket **arBuckets;         // 存储hash数组
    dtor_func_t pDestructor;
    zend_bool persistent;
    unsigned char nApplyCount; // 标记当前hash Bucket被递归访问的次数（防止多次递归）
    zend_bool bApplyProtection;// 标记当前hash桶允许不允许多次访问，不允许时，最多只能递归3次
#if ZEND_DEBUG
    int inconsistent;
#endif
} HashTable;
```

```c
typedef struct bucket {
    /* Used for numeric indexing */
    ulong h;            // 对char *key进行hash后的值，数字索引的话就是索引值
    uint nKeyLength;    // hash关键字的长度，如果数组索引为数字，此值为0
    void *pData;        // 指向value，一般是用户数据的副本，如果是指针数据，则指向pDataPtr
    void *pDataPtr;     //如果是指针数据，此值会指向真正的value，同时上面pData会指向此值
    struct bucket *pListNext;   // 整个hash表的下一元素
    struct bucket *pListLast;   // 整个哈希表该元素的上一个元素
    struct bucket *pNext;       // 存放在同一个hash Bucket内的下一个元素
    struct bucket *pLast;       // 同一个哈希bucket的上一个元素
    char arKey[1];  
    /*存储字符索引，此项必须放在最未尾，因为此处只字义了1个字节，存储的实际上是指向char *key的值，
    这就意味着可以省去再赋值一次的消耗，而且，有时此值并不需要，所以同时还节省了空间。
    */
} Bucket;
```

核心函数

```
_zend_hash_add_or_update  
(HashTable *ht, const char *arKey, uint nKeyLength, void *pData, uint nDataSize, void **pDest, int flag ZEND_FILE_LINE_DC)

zend_hash_next_index_insert  
zend_hash_find  
zend_hash_index_ find

```

包含关系：

> hashTable\-\>buckers list\-\>bucket

概括

> 一个C数组（hashTable中arBuckets）容器，包含所有的bucket，数组的下标：就是一个hash后的h值。bucket是一个结构体，包含向上指针和向下指针。当有HASH冲突时，bucket又可以指向相同KEY的bucket.

总结

> 用一个数组存储所有bucket，bucket在数组里组成一个双向链表，bucket自己又是一个双向链表

如何实现数字下标与字符串KEY，并行？

> hash func\(number|string\) = 一个uint 整数，不管是int 还是string做为key ，最终都能转换成一个整数。但整数范围很大，数组下标却是有上限的，int太大，所以再经过一个掩网运算，就可以 得到 数组的下标，做为KEY了

> 其中： hash func = time33算法\( hash（i） = hash（i\-1）\*33 \+ str\[i\]\)

bucket中，nKeyLength \> 0 ，证明是字符串的KEY，如果nKeyLength = 0 ，即是数字索引。nKeyLength = 0 ，即h为键值，nKeyLength \> 0 ，即arKey为键值

日常对数组的操作，最终都会带上struct \_hashtable

> 比如：count\(arr\) \-\> nNumOfElements
> 
> 
> foreach遍历 \-\> pInternalPointer pListHead pListTail
> 
> 
> a\[\] = 1 \-\> nNextFreeElement

而具体到元素时，最终都会使用 struct bucket

> 如，查找KEY \-\> nKeyLength arKey
> 
> 
> KEY对应的值 \-\> pData pDataPtr
> 
> 
> 当hash冲突时 \-\> h pNext pLast

如果arBuckets 是个数组，那有毛用？

是保证线性顺序！

比如：正常插入2个值~但是这两个值哈希值冲突，按照拉链法，这两个值应该放在相连的位置，那就无法保证顺序了~

PHP里的做法是，arBuckets数组就是按照插入时间来存，如果冲突，通过pNext的地址来访问

以上也解释了php foreach 遍历时，并不会给KEY进行排序，而按照插入顺序进行遍历，比如：

array\(2=\>"a",1=\>"b"\)

按照常规理想输出应该是 b a ，但实际上是 a b

引申出另外一个问题：为什么每次foreach都是从头开始？

因为foreach 自动执行了reset\(\)，将hashTable内部指针，自动指向数组头元素，而while 就不会这么做
