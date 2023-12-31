# context

## 概览

Context更像是一种设计模式，而不是具体的一个东西，其结构如下：

一个主接口，一个辅助接口，4个实现类，6个函数

> 网上也有管context叫做：家族

```mermaid

graph LR

Context接口-->emptyCtx实现类

Context接口-->cancelCtx实现类

Context接口-->timerCtx实现类

Context接口-->valueCtx实现类

  

canceler接口-->cancelCtx实现类

  

emptyCtx实现类-->background

emptyCtx实现类-->todo

cancelCtx实现类-->withCancel

cancelCtx实现类-->withDeadline

cancelCtx实现类-->withTimeout

valueCtx实现类-->withValue

```

  

  

```mermaid

graph TD

Context接口-->emptyCtx实现类

emptyCtx实现类-->background方法

emptyCtx实现类-->todo方法

background方法-->cancelCtx实现类

todo方法-->cancelCtx实现类

background方法-->valueCtx实现类

todo方法-->valueCtx实现类

valueCtx实现类-->withValue方法

cancelCtx实现类-->timerCtx实现类

cancelCtx实现类-->withCancel方法

cancelCtx实现类-->withTimeout方法

cancelCtx实现类-->withDeadline方法

  

  

style Context接口 fill:#f9f,stroke:#333,stroke-width:4px,fill-opacity:0.5

style withCancel方法 fill:#F00,stroke:#333,stroke-width:4px,fill-opacity:0.5

  

style withTimeout方法 fill:#F00,stroke:#333,stroke-width:4px,fill-opacity:0.5

  

style withDeadline方法 fill:#F00,stroke:#333,stroke-width:4px,fill-opacity:0.5

  

  

style background方法 fill:#F00,stroke:#333,stroke-width:4px,fill-opacity:0.5

  

style todo方法 fill:#F00,stroke:#333,stroke-width:4px,fill-opacity:0.5

  

```

## Context 接口

```
type Context interface {

    Deadline() (deadline time.Time, ok bool)

    Done() <-chan struct{}

    Err() error

    Value(key interface{}) interface{}
}

```

1. Deadline:返回绑定当前context的任务被取消的截止时间；如果没有设定期限，将返回ok == false。
2. Done :当绑定当前context的任务被取消时，将返回一个关闭的channel；如果当前context不会被取消，将返回nil。
3. Err :如果Done返回的channel没有关闭，将返回nil;如果Done返回的channel已经关闭，将返回非空的值表示任务结束的原因。如果是context被取消，Err将返回Canceled；如果是context超时，Err将返回DeadlineExceeded。
4. Value 返回context存储的键值对中当前key对应的值，如果没有对应的key,则返回nil。

## 4个实现了context的类

1. emptyCtx 空的Context，只实现了Context interface 4个方法
2. cancelCtx 继承自Context并实现了canceler interface
3. timerCtx 继承自cancelCtx，可以用来设置timeout
4. valueCtx 继承自Context，可以储存一对键值对

## emptyCtx

```
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil
}

func (*emptyCtx) Err() error {
    return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}

func (e *emptyCtx) String() string {
    switch e {
    case background:
        return "context.Background"
    case todo:
        return "context.TODO"
    }
    return "unknown empty Context"
}
```

好像也没什么，就是空实现了context 接口...

emptyCtx:没有超时时间，不能取消，也不能存储任何额外信息，所以emptyCtx用来作为context树的根节点。

```
var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

func Background() Context {
    return background
}

func TODO() Context {
    return todo
}
```

emptyCtx并不会直接使用，GO提供了两个方法：background todo来实例化.

1. Background：通常被用于主函数、初始化以及测试中，作为一个顶层的context，也就是说一般我们创建的context都是基于Background；
2. TODO：在不确定使用什么context的时候才会使用

> 我是没搞懂有啥本质区别，想起学英语的时候用todo something 这种东西做点位符....

接下来，看下context的3个具体实现类~

## valueCtx

```
type valueCtx struct {
    Context
    key, val interface{}
}

func (c *valueCtx) Value(key interface{}) interface{} {
    if c.key == key {
        return c.val
    }
    return c.Context.Value(key)
}
```

Context是继承，另外还包含了两个变量：key val

valueCtx\-value方法：覆写了父类的value方法

总结下：valueCtx继承父类context，同时可以有附带值（K/V），覆写了value方法，可用于路径判定。

## cancelCtx

```
type cancelCtx struct {
	Context

	mu       sync.Mutex            
	done     chan struct{}         
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
}

type canceler interface {
    cancel(removeFromParent bool, err error)
    Done() <-chan struct{}
}
```

Context是继承，但多了几个成员属性：

1. mu:悲观锁，防止协程安全
2. done:一个管道，用来传递关闭信号
3. children：用于存储当前节点下的所有子节点
4. err

canceler是接口,看一下，cancelCtx 如何实现这个接口

```
func (c *cancelCtx) Done() <-chan struct{} {
    c.mu.Lock()
    if c.done == nil {
        c.done = make(chan struct{})
    }
    d := c.done
    c.mu.Unlock()
    return d
}

func (c *cancelCtx) Err() error {
    c.mu.Lock()
    err := c.err
    c.mu.Unlock()
    return err
}

func (c *cancelCtx) cancel(removeFromParent bool, err error) {
    if err == nil {
        panic("context: internal error: missing cancel error")
    }
    c.mu.Lock()
    if c.err != nil {
        c.mu.Unlock()
        return // already canceled
    }
    // 设置取消原因
    c.err = err
    设置一个关闭的channel或者将done channel关闭，用以发送关闭信号
    if c.done == nil {
        c.done = closedchan
    } else {
        close(c.done)
    }
    // 将子节点context依次取消
    for child := range c.children {
        // NOTE: acquiring the child's lock while holding parent's lock.
        child.cancel(false, err)
    }
    c.children = nil
    c.mu.Unlock()

    if removeFromParent {
        // 将当前context节点从父节点上移除
        removeChild(c.Context, c)
    }
}
```

done\(\):就是获取一个管道（锁控制安全）

err:没啥说的

cancel:这才是核心方法，还是使用锁保证安全，发送关闭信号给done管道，接着取消该context下的子节点，最后将自己从父context节点中移除.

挺简单的，核心值是：children属性，通过这个实现了连带关闭

## WithCancel

上面的cancelCtx类的使用，得用WithCancel来创建

```
type CancelFunc func()

func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}

// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
    // 将parent作为父节点context生成一个新的子节点
    return cancelCtx{Context: parent}
}

func propagateCancel(parent Context, child canceler) {
    if parent.Done() == nil {
        // parent.Done()返回nil表明父节点以上的路径上没有可取消的context
        return // parent is never canceled
    }
    // 获取最近的类型为cancelCtx的祖先节点
    if p, ok := parentCancelCtx(parent); ok {
        p.mu.Lock()
        if p.err != nil {
            // parent has already been canceled
            child.cancel(false, p.err)
        } else {
            if p.children == nil {
                p.children = make(map[canceler]struct{})
            }
            // 将当前子节点加入最近cancelCtx祖先节点的children中
            p.children[child] = struct{}{}
        }
        p.mu.Unlock()
    } else {
        go func() {
            select {
            case <-parent.Done():
                child.cancel(false, parent.Err())
            case <-child.Done():
            }
        }()
    }
}

func parentCancelCtx(parent Context) (*cancelCtx, bool) {
    for {
        switch c := parent.(type) {
        case *cancelCtx:
            return c, true
        case *timerCtx:
            return &c.cancelCtx, true
        case *valueCtx:
            parent = c.Context
        default:
            return nil, false
        }
    }
}
```

1. 创建一个CancelCtx
2. propagateCancel：这个就是核心了，主要就是将新的节点，加到父节点的map中。

总结：实现了将某个节点加到父节点上，父节点一但cancel，那么所有节点均退出。

## timerCtx

```
type timerCtx struct {
    cancelCtx
    timer *time.Timer // Under cancelCtx.mu.

    deadline time.Time
}

func (c *timerCtx) Deadline() (deadline time.Time, ok bool) {
    return c.deadline, true
}

func (c *timerCtx) cancel(removeFromParent bool, err error) {
    将内部的cancelCtx取消
    c.cancelCtx.cancel(false, err)
    if removeFromParent {
        // Remove this timerCtx from its parent cancelCtx's children.
        removeChild(c.cancelCtx.Context, c)
    }
    c.mu.Lock()
    if c.timer != nil {
        取消计时器
        c.timer.Stop()
        c.timer = nil
    }
    c.mu.Unlock()
}
```

基于cancelCtx，多了一个timer ,赋值给deadline，没啥特殊的...

## WithDeadline

父协程可取消的context

## WithTimeout

与上面类似

总结一下吧

1. 有个context接口，该接口算是根接口吧。里面有4个方法，使用者可以都实现，也可以只实现部分（不想实现的直接空实现即可），取决于使用方。
2. 具体实现context 接口的类是emptyCtx，但是不给开发者直接使用
3. 使用background todo 两个方法来做实例化
4. 虽然实例化了，得到了emptyCtx，但是只是空实现
5. valueCtx \-》 WithValue
6. cancelCtx \-\> WithCancel timerCtx
7. WithDeadline WithTimeout
8. 通过golang自带的valueCtx cancelCtx 来最终爆给用户使用
9. 用户的使用过程中，可以根据需求，自行决定使用哪种类型的context
10. 最主要的是：所有的context最终被组织成一个链表

```
graph LR
emptyCtx-->background/todo
background/todo-->valueCtx
```
