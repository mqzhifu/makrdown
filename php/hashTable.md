# hashTable

```
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

```
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

%0A%0A%60%60%60%0Atypedef%20struct%20\_hashtable%20%7B%20%0A%20%20%20%20uint%20nTableSize%3B%20%20%20%20%20%20%20%20%2F%2F%20hash%20Bucket%E7%9A%84%E5%A4%A7%E5%B0%8F%EF%BC%8C%E6%9C%80%E5%B0%8F%E4%B8%BA8%EF%BC%8C%E4%BB%A52x%E5%A2%9E%E9%95%BF%E3%80%82%0A%20%20%20%20uint%20nTableMask%3B%20%20%20%20%20%20%20%20%2F%2F%20nTableSize\-1%EF%BC%8C%20%E6%8E%A9%E7%A0%81%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%A0%B9%E6%8D%AEhash%E5%80%BC%E8%AE%A1%E7%AE%97%E5%AD%98%E5%82%A8%E4%BD%8D%E7%BD%AE%E3%80%82%0A%20%20%20%20uint%20nNumOfElements%3B%20%20%20%20%2F%2F%20hash%20Bucket%E4%B8%AD%E5%BD%93%E5%89%8D%E5%AD%98%E5%9C%A8%E7%9A%84%E5%85%83%E7%B4%A0%E4%B8%AA%E6%95%B0%EF%BC%8Ccount\(\)%E5%87%BD%E6%95%B0%E4%BC%9A%E7%9B%B4%E6%8E%A5%E8%BF%94%E5%9B%9E%E6%AD%A4%E5%80%BC%20%0A%20%20%20%20ulong%20nNextFreeElement%3B%20%2F%2F%20%E4%B8%8B%E4%B8%80%E4%B8%AA%E6%95%B0%E5%AD%97%E7%B4%A2%E5%BC%95%E7%9A%84%E4%BD%8D%E7%BD%AE%EF%BC%8C%24arr%5B%5D%20%3D%20%22hello%22%E6%97%B6%E4%BC%9A%E7%94%A8%E5%88%B0%0A%20%20%20%20Bucket%20\*pInternalPointer%3B%20%20%20%2F%2F%20%E5%BD%93%E5%89%8D%E9%81%8D%E5%8E%86%E7%9A%84%E6%8C%87%E9%92%88%EF%BC%88foreach%E6%AF%94for%E5%BF%AB%E7%9A%84%E5%8E%9F%E5%9B%A0%E4%B9%8B%E4%B8%80%EF%BC%89%0A%20%20%20%20Bucket%20\*pListHead%3B%20%20%20%20%20%20%20%20%20%20%2F%2F%20%E5%AD%98%E5%82%A8%E6%95%B0%E7%BB%84%E5%A4%B4%E5%85%83%E7%B4%A0%E6%8C%87%E9%92%88%0A%20%20%20%20Bucket%20\*pListTail%3B%20%20%20%20%20%20%20%20%20%20%2F%2F%20%E5%AD%98%E5%82%A8%E6%95%B0%E7%BB%84%E5%B0%BE%E5%85%83%E7%B4%A0%E6%8C%87%E9%92%88%0A%20%20%20%20Bucket%20\*\*arBuckets%3B%20%20%20%20%20%20%20%20%20%2F%2F%20%E5%AD%98%E5%82%A8hash%E6%95%B0%E7%BB%84%0A%20%20%20%20dtor\_func\_t%20pDestructor%3B%0A%20%20%20%20zend\_bool%20persistent%3B%0A%20%20%20%20unsigned%20char%20nApplyCount%3B%20%2F%2F%20%E6%A0%87%E8%AE%B0%E5%BD%93%E5%89%8Dhash%20Bucket%E8%A2%AB%E9%80%92%E5%BD%92%E8%AE%BF%E9%97%AE%E7%9A%84%E6%AC%A1%E6%95%B0%EF%BC%88%E9%98%B2%E6%AD%A2%E5%A4%9A%E6%AC%A1%E9%80%92%E5%BD%92%EF%BC%89%0A%20%20%20%20zend\_bool%20bApplyProtection%3B%2F%2F%20%E6%A0%87%E8%AE%B0%E5%BD%93%E5%89%8Dhash%E6%A1%B6%E5%85%81%E8%AE%B8%E4%B8%8D%E5%85%81%E8%AE%B8%E5%A4%9A%E6%AC%A1%E8%AE%BF%E9%97%AE%EF%BC%8C%E4%B8%8D%E5%85%81%E8%AE%B8%E6%97%B6%EF%BC%8C%E6%9C%80%E5%A4%9A%E5%8F%AA%E8%83%BD%E9%80%92%E5%BD%923%E6%AC%A1%0A%23if%20ZEND\_DEBUG%0A%20%20%20%20int%20inconsistent%3B%0A%23endif%0A%7D%20HashTable%3B%0A%60%60%60%0A%0A%60%60%60%0Atypedef%20struct%20bucket%20%7B%0A%20%20%20%20%2F\*%20Used%20for%20numeric%20indexing%20\*%2F%0A%20%20%20%20ulong%20h%3B%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%20%E5%AF%B9char%20\*key%E8%BF%9B%E8%A1%8Chash%E5%90%8E%E7%9A%84%E5%80%BC%EF%BC%8C%E6%95%B0%E5%AD%97%E7%B4%A2%E5%BC%95%E7%9A%84%E8%AF%9D%E5%B0%B1%E6%98%AF%E7%B4%A2%E5%BC%95%E5%80%BC%0A%20%20%20%20uint%20nKeyLength%3B%20%20%20%20%2F%2F%20hash%E5%85%B3%E9%94%AE%E5%AD%97%E7%9A%84%E9%95%BF%E5%BA%A6%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%95%B0%E7%BB%84%E7%B4%A2%E5%BC%95%E4%B8%BA%E6%95%B0%E5%AD%97%EF%BC%8C%E6%AD%A4%E5%80%BC%E4%B8%BA0%0A%20%20%20%20void%20\*pData%3B%20%20%20%20%20%20%20%20%2F%2F%20%E6%8C%87%E5%90%91value%EF%BC%8C%E4%B8%80%E8%88%AC%E6%98%AF%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E7%9A%84%E5%89%AF%E6%9C%AC%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%98%AF%E6%8C%87%E9%92%88%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%88%99%E6%8C%87%E5%90%91pDataPtr%0A%20%20%20%20void%20\*pDataPtr%3B%20%20%20%20%20%2F%2F%E5%A6%82%E6%9E%9C%E6%98%AF%E6%8C%87%E9%92%88%E6%95%B0%E6%8D%AE%EF%BC%8C%E6%AD%A4%E5%80%BC%E4%BC%9A%E6%8C%87%E5%90%91%E7%9C%9F%E6%AD%A3%E7%9A%84value%EF%BC%8C%E5%90%8C%E6%97%B6%E4%B8%8A%E9%9D%A2pData%E4%BC%9A%E6%8C%87%E5%90%91%E6%AD%A4%E5%80%BC%0A%20%20%20%20struct%20bucket%20\*pListNext%3B%20%20%20%2F%2F%20%E6%95%B4%E4%B8%AAhash%E8%A1%A8%E7%9A%84%E4%B8%8B%E4%B8%80%E5%85%83%E7%B4%A0%0A%20%20%20%20struct%20bucket%20\*pListLast%3B%20%20%20%2F%2F%20%E6%95%B4%E4%B8%AA%E5%93%88%E5%B8%8C%E8%A1%A8%E8%AF%A5%E5%85%83%E7%B4%A0%E7%9A%84%E4%B8%8A%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0%0A%20%20%20%20struct%20bucket%20\*pNext%3B%20%20%20%20%20%20%20%2F%2F%20%E5%AD%98%E6%94%BE%E5%9C%A8%E5%90%8C%E4%B8%80%E4%B8%AAhash%20Bucket%E5%86%85%E7%9A%84%E4%B8%8B%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0%0A%20%20%20%20struct%20bucket%20\*pLast%3B%20%20%20%20%20%20%20%2F%2F%20%E5%90%8C%E4%B8%80%E4%B8%AA%E5%93%88%E5%B8%8Cbucket%E7%9A%84%E4%B8%8A%E4%B8%80%E4%B8%AA%E5%85%83%E7%B4%A0%0A%20%20%20%20char%20arKey%5B1%5D%3B%20%20%0A%20%20%20%20%2F\*%E5%AD%98%E5%82%A8%E5%AD%97%E7%AC%A6%E7%B4%A2%E5%BC%95%EF%BC%8C%E6%AD%A4%E9%A1%B9%E5%BF%85%E9%A1%BB%E6%94%BE%E5%9C%A8%E6%9C%80%E6%9C%AA%E5%B0%BE%EF%BC%8C%E5%9B%A0%E4%B8%BA%E6%AD%A4%E5%A4%84%E5%8F%AA%E5%AD%97%E4%B9%89%E4%BA%861%E4%B8%AA%E5%AD%97%E8%8A%82%EF%BC%8C%E5%AD%98%E5%82%A8%E7%9A%84%E5%AE%9E%E9%99%85%E4%B8%8A%E6%98%AF%E6%8C%87%E5%90%91char%20\*key%E7%9A%84%E5%80%BC%EF%BC%8C%0A%20%20%20%20%E8%BF%99%E5%B0%B1%E6%84%8F%E5%91%B3%E7%9D%80%E5%8F%AF%E4%BB%A5%E7%9C%81%E5%8E%BB%E5%86%8D%E8%B5%8B%E5%80%BC%E4%B8%80%E6%AC%A1%E7%9A%84%E6%B6%88%E8%80%97%EF%BC%8C%E8%80%8C%E4%B8%94%EF%BC%8C%E6%9C%89%E6%97%B6%E6%AD%A4%E5%80%BC%E5%B9%B6%E4%B8%8D%E9%9C%80%E8%A6%81%EF%BC%8C%E6%89%80%E4%BB%A5%E5%90%8C%E6%97%B6%E8%BF%98%E8%8A%82%E7%9C%81%E4%BA%86%E7%A9%BA%E9%97%B4%E3%80%82%0A%20%20%20%20\*%2F%0A%7D%20Bucket%3B%0A%60%60%60%0A%0A%E6%A0%B8%E5%BF%83%E5%87%BD%E6%95%B0%0A%60%60%60%0A\_zend\_hash\_add\_or\_update%20%20%0A\(HashTable%20\*ht%2C%20const%20char%20\*arKey%2C%20uint%20nKeyLength%2C%20void%20\*pData%2C%20uint%20nDataSize%2C%20void%20\*\*pDest%2C%20int%20flag%20ZEND\_FILE\_LINE\_DC\)%0A%0Azend\_hash\_next\_index\_insert%20%20%0Azend\_hash\_find%20%20%0Azend\_hash\_index\_%20find%0A%0A%0A%60%60%60%0A%0A%0A%E5%8C%85%E5%90%AB%E5%85%B3%E7%B3%BB%EF%BC%9A%20%20%0A%3EhashTable\-%3Ebuckers%20list\-%3Ebucket%0A%0A%E6%A6%82%E6%8B%AC%0A%3E%E4%B8%80%E4%B8%AAC%E6%95%B0%E7%BB%84%EF%BC%88hashTable%E4%B8%ADarBuckets%EF%BC%89%E5%AE%B9%E5%99%A8%EF%BC%8C%E5%8C%85%E5%90%AB%E6%89%80%E6%9C%89%E7%9A%84bucket%EF%BC%8C%E6%95%B0%E7%BB%84%E7%9A%84%E4%B8%8B%E6%A0%87%EF%BC%9A%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AAhash%E5%90%8E%E7%9A%84h%E5%80%BC%E3%80%82bucket%E6%98%AF%E4%B8%80%E4%B8%AA%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E5%8C%85%E5%90%AB%E5%90%91%E4%B8%8A%E6%8C%87%E9%92%88%E5%92%8C%E5%90%91%E4%B8%8B%E6%8C%87%E9%92%88%E3%80%82%E5%BD%93%E6%9C%89HASH%E5%86%B2%E7%AA%81%E6%97%B6%EF%BC%8Cbucket%E5%8F%88%E5%8F%AF%E4%BB%A5%E6%8C%87%E5%90%91%E7%9B%B8%E5%90%8CKEY%E7%9A%84bucket.%0A%0A%E6%80%BB%E7%BB%93%0A%3E%E7%94%A8%E4%B8%80%E4%B8%AA%E6%95%B0%E7%BB%84%E5%AD%98%E5%82%A8%E6%89%80%E6%9C%89bucket%EF%BC%8Cbucket%E5%9C%A8%E6%95%B0%E7%BB%84%E9%87%8C%E7%BB%84%E6%88%90%E4%B8%80%E4%B8%AA%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8%EF%BC%8Cbucket%E8%87%AA%E5%B7%B1%E5%8F%88%E6%98%AF%E4%B8%80%E4%B8%AA%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8%20%20%0A%0A%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E6%95%B0%E5%AD%97%E4%B8%8B%E6%A0%87%E4%B8%8E%E5%AD%97%E7%AC%A6%E4%B8%B2KEY%EF%BC%8C%E5%B9%B6%E8%A1%8C%EF%BC%9F%20%20%0A%3Ehash%20func\(number%7Cstring\)%20%20%3D%20%E4%B8%80%E4%B8%AAuint%20%E6%95%B4%E6%95%B0%EF%BC%8C%E4%B8%8D%E7%AE%A1%E6%98%AFint%20%E8%BF%98%E6%98%AFstring%E5%81%9A%E4%B8%BAkey%20%EF%BC%8C%E6%9C%80%E7%BB%88%E9%83%BD%E8%83%BD%E8%BD%AC%E6%8D%A2%E6%88%90%E4%B8%80%E4%B8%AA%E6%95%B4%E6%95%B0%E3%80%82%E4%BD%86%E6%95%B4%E6%95%B0%E8%8C%83%E5%9B%B4%E5%BE%88%E5%A4%A7%EF%BC%8C%E6%95%B0%E7%BB%84%E4%B8%8B%E6%A0%87%E5%8D%B4%E6%98%AF%E6%9C%89%E4%B8%8A%E9%99%90%E7%9A%84%EF%BC%8Cint%E5%A4%AA%E5%A4%A7%EF%BC%8C%E6%89%80%E4%BB%A5%E5%86%8D%E7%BB%8F%E8%BF%87%E4%B8%80%E4%B8%AA%E6%8E%A9%E7%BD%91%E8%BF%90%E7%AE%97%EF%BC%8C%E5%B0%B1%E5%8F%AF%E4%BB%A5%20%E5%BE%97%E5%88%B0%20%E6%95%B0%E7%BB%84%E7%9A%84%E4%B8%8B%E6%A0%87%EF%BC%8C%E5%81%9A%E4%B8%BAKEY%E4%BA%86%0A%0A%3E%E5%85%B6%E4%B8%AD%EF%BC%9A%20hash%20func%20%3D%20time33%E7%AE%97%E6%B3%95\(%20hash%EF%BC%88i%EF%BC%89%20%3D%20hash%EF%BC%88i\-1%EF%BC%89\*33%20%2B%20str%5Bi%5D\)%20%0A%0Abucket%E4%B8%AD%EF%BC%8CnKeyLength%20%3E%200%20%EF%BC%8C%E8%AF%81%E6%98%8E%E6%98%AF%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%9A%84KEY%EF%BC%8C%E5%A6%82%E6%9E%9CnKeyLength%20%3D%200%20%EF%BC%8C%E5%8D%B3%E6%98%AF%E6%95%B0%E5%AD%97%E7%B4%A2%E5%BC%95%E3%80%82nKeyLength%20%3D%200%20%EF%BC%8C%E5%8D%B3h%E4%B8%BA%E9%94%AE%E5%80%BC%EF%BC%8CnKeyLength%20%3E%200%20%EF%BC%8C%E5%8D%B3arKey%E4%B8%BA%E9%94%AE%E5%80%BC%0A%0A%E6%97%A5%E5%B8%B8%E5%AF%B9%E6%95%B0%E7%BB%84%E7%9A%84%E6%93%8D%E4%BD%9C%EF%BC%8C%E6%9C%80%E7%BB%88%E9%83%BD%E4%BC%9A%E5%B8%A6%E4%B8%8Astruct%20\_hashtable%20%20%0A%3E%E6%AF%94%E5%A6%82%EF%BC%9Acount\(arr\)%20%20%20%20\-%3E%20nNumOfElements%20%20%0Aforeach%E9%81%8D%E5%8E%86%20\-%3E%20pInternalPointer%20pListHead%20%20pListTail%20%20%0Aa%5B%5D%20%3D%201%20\-%3E%20nNextFreeElement%20%20%0A%0A%E8%80%8C%E5%85%B7%E4%BD%93%E5%88%B0%E5%85%83%E7%B4%A0%E6%97%B6%EF%BC%8C%E6%9C%80%E7%BB%88%E9%83%BD%E4%BC%9A%E4%BD%BF%E7%94%A8%20%20struct%20bucket%20%20%0A%3E%E5%A6%82%EF%BC%8C%E6%9F%A5%E6%89%BEKEY%20\-%3E%20%20%20nKeyLength%20%20arKey%20%20%0AKEY%E5%AF%B9%E5%BA%94%E7%9A%84%E5%80%BC%20\-%3E%20%20%20pData%20pDataPtr%20%20%0A%E5%BD%93hash%E5%86%B2%E7%AA%81%E6%97%B6%20\-%3E%20%20h%20pNext%20pLast%0A%0A%E5%A6%82%E6%9E%9CarBuckets%20%E6%98%AF%E4%B8%AA%E6%95%B0%E7%BB%84%EF%BC%8C%E9%82%A3%E6%9C%89%E6%AF%9B%E7%94%A8%EF%BC%9F%20%20%0A%E6%98%AF%E4%BF%9D%E8%AF%81%E7%BA%BF%E6%80%A7%E9%A1%BA%E5%BA%8F%EF%BC%81%0A%0A%E6%AF%94%E5%A6%82%EF%BC%9A%E6%AD%A3%E5%B8%B8%E6%8F%92%E5%85%A52%E4%B8%AA%E5%80%BC~%E4%BD%86%E6%98%AF%E8%BF%99%E4%B8%A4%E4%B8%AA%E5%80%BC%E5%93%88%E5%B8%8C%E5%80%BC%E5%86%B2%E7%AA%81%EF%BC%8C%E6%8C%89%E7%85%A7%E6%8B%89%E9%93%BE%E6%B3%95%EF%BC%8C%E8%BF%99%E4%B8%A4%E4%B8%AA%E5%80%BC%E5%BA%94%E8%AF%A5%E6%94%BE%E5%9C%A8%E7%9B%B8%E8%BF%9E%E7%9A%84%E4%BD%8D%E7%BD%AE%EF%BC%8C%E9%82%A3%E5%B0%B1%E6%97%A0%E6%B3%95%E4%BF%9D%E8%AF%81%E9%A1%BA%E5%BA%8F%E4%BA%86~%0A%0APHP%E9%87%8C%E7%9A%84%E5%81%9A%E6%B3%95%E6%98%AF%EF%BC%8CarBuckets%E6%95%B0%E7%BB%84%E5%B0%B1%E6%98%AF%E6%8C%89%E7%85%A7%E6%8F%92%E5%85%A5%E6%97%B6%E9%97%B4%E6%9D%A5%E5%AD%98%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%86%B2%E7%AA%81%EF%BC%8C%E9%80%9A%E8%BF%87pNext%E7%9A%84%E5%9C%B0%E5%9D%80%E6%9D%A5%E8%AE%BF%E9%97%AE%0A%0A%E4%BB%A5%E4%B8%8A%E4%B9%9F%E8%A7%A3%E9%87%8A%E4%BA%86php%20foreach%20%E9%81%8D%E5%8E%86%E6%97%B6%EF%BC%8C%E5%B9%B6%E4%B8%8D%E4%BC%9A%E7%BB%99KEY%E8%BF%9B%E8%A1%8C%E6%8E%92%E5%BA%8F%EF%BC%8C%E8%80%8C%E6%8C%89%E7%85%A7%E6%8F%92%E5%85%A5%E9%A1%BA%E5%BA%8F%E8%BF%9B%E8%A1%8C%E9%81%8D%E5%8E%86%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9A%0Aarray\(2%3D%3E%22a%22%2C1%3D%3E%22b%22\)%0A%0A%E6%8C%89%E7%85%A7%E5%B8%B8%E8%A7%84%E7%90%86%E6%83%B3%E8%BE%93%E5%87%BA%E5%BA%94%E8%AF%A5%E6%98%AF%20%20%20b%20a%20%20%EF%BC%8C%E4%BD%86%E5%AE%9E%E9%99%85%E4%B8%8A%E6%98%AF%20%20a%20%20b%0A%0A%E5%BC%95%E7%94%B3%E5%87%BA%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E6%AF%8F%E6%AC%A1foreach%E9%83%BD%E6%98%AF%E4%BB%8E%E5%A4%B4%E5%BC%80%E5%A7%8B%EF%BC%9F%20%20%0A%E5%9B%A0%E4%B8%BAforeach%20%E8%87%AA%E5%8A%A8%E6%89%A7%E8%A1%8C%E4%BA%86reset\(\)%EF%BC%8C%E5%B0%86hashTable%E5%86%85%E9%83%A8%E6%8C%87%E9%92%88%EF%BC%8C%E8%87%AA%E5%8A%A8%E6%8C%87%E5%90%91%E6%95%B0%E7%BB%84%E5%A4%B4%E5%85%83%E7%B4%A0%EF%BC%8C%E8%80%8Cwhile%20%E5%B0%B1%E4%B8%8D%E4%BC%9A%E8%BF%99%E4%B9%88%E5%81%9A%0A%0A%0A
