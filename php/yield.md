function readTxt(){

$handle = fopen("./test.txt", 'rb');

while (feof($handle)===false) {

yield fgets($handle);

}

fclose($handle);

}

foreach (readTxt() as $key => $value) {

//doing some thing

echo $value.'<br />';

}

上面这个例子，是打开个文件FD，然后逐行把内容读出来。传统的方式，可能是，一次把整个文件内容都读到内存中，然后再做操作。

用上yield之后，实际上，当执行到yield之后 ：程序停止，你得再调用一次，他才继续读取。这样做的好处是，用多少读多少，可能读到一个点就停止读了，减少内存浪费。

yield XXX

当一个函数中有yield出现时，调用后，即会返回一个迭代器，而不正常执行一个函数。

$gen = readTxt();

如何执行这个函数呢？foreach，因为返回是个迭代器，你循环就行了。

也可以单行控制

执行一次

$gen->current();

判断是否已执行到最后一个元素了

$gen->valid();

执行下一个元素

$rs = $gen->next();

实际上就是函数中，出现yield就停住，如果外层是个死循环，你每次调 都会执行一次这个函数。如果不是死循环，那迭代器就能执行一次。如果是N个yield 且 外层没有死循环，那迭代器就执行N次。

yield 1;yield 2;

类似这种，就是返回 1 2 ，有点类似return，如果函数内即有yield 和return ，return 被忽略

echo (yield 1)."I\n";

echo (yield 2)."I\n";

加上括号了，那这条语句第一次执行，返回的是1，但是你可以通过 send 传值进去，第二次执行的结果就是你传的值加 "I\n";

这也是，yield 神奇的地方，也是可以实现协程的基础。

function aaa(){

yield init()

yield process()

yield end()

}

function bbb(){

yield init()

yield process()

yield end()

}

通过yield 就实现了，可以先执行aaa中的init 方法，接着执行bbb 中的 init()方法