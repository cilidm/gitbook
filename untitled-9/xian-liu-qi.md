# 限流器

## 1. 限流器 <a id="&#x9650;&#x6D41;&#x5668;"></a>

限流器是后台服务中的非常重要的组件，可以用来限制请求速率，保护服务，以免服务过载。 限流器的实现方法有很多种，例如滑动窗口法、Token Bucket、Leaky Bucket等。

其实golang标准库中就自带了限流算法的实现，即golang.org/x/time/rate。 该限流器是基于Token Bucket\(令牌桶\)实现的。

简单来说，令牌桶就是想象有一个固定大小的桶，系统会以恒定速率向桶中放Token，桶满则暂时不放。 而用户则从桶中取Token，如果有剩余Token就可以一直取。如果没有剩余Token，则需要等到系统中被放置了Token才行。

本文则主要集中介绍下该组件的具体使用方法：

我们可以使用以下方法构造一个限流器对象：

```text
limiter := NewLimiter(10, 1);
```

这里有两个参数：

* 第一个参数是r Limit。代表每秒可以向Token桶中产生多少token。Limit实际上是float64的别名。
* 第二个参数是b int。b代表Token桶的容量大小。 那么，对于以上例子来说，其构造出的限流器含义为，其令牌桶大小为1, 以每秒10个Token的速率向桶中放置Token。

除了直接指定每秒产生的Token个数外，还可以用Every方法来指定向Token桶中放置Token的间隔，例如：

```text
limit := Every(100 * time.Millisecond);
limiter := NewLimiter(limit, 1);
```

以上就表示每100ms往桶中放一个Token。本质上也就是一秒钟产生10个。

Limiter提供了三类方法供用户消费Token，用户可以每次消费一个Token，也可以一次性消费多个Token。 而每种方法代表了当Token不足时，各自不同的对应手段。

### 1.1.1. Wait/WaitN <a id="waitwaitn"></a>

```text
func (lim *Limiter) Wait(ctx context.Context) (err error)
func (lim *Limiter) WaitN(ctx context.Context, n int) (err error)
```

Wait实际上就是WaitN\(ctx,1\)。

当使用Wait方法消费Token时，如果此时桶内Token数组不足\(小于N\)，那么Wait方法将会阻塞一段时间，直至Token满足条件。如果充足则直接返回。

这里可以看到，Wait方法有一个context参数。 我们可以设置context的Deadline或者Timeout，来决定此次Wait的最长时间。

### 1.1.2. Allow/AllowN <a id="allowallown"></a>

```text
func (lim *Limiter) Allow() bool
func (lim *Limiter) AllowN(now time.Time, n int) bool
```

Allow实际上就是AllowN\(time.Now\(\),1\)。

AllowN方法表示，截止到某一时刻，目前桶中数目是否至少为n个，满足则返回true，同时从桶中消费n个token。 反之返回不消费Token，false。

通常对应这样的线上场景，如果请求速率过快，就直接丢到某些请求。

### 1.1.3. Reserve/ReserveN <a id="reservereserven"></a>

```text
func (lim *Limiter) Reserve() *Reservation
func (lim *Limiter) ReserveN(now time.Time, n int) *Reservation
```

Reserve相当于ReserveN\(time.Now\(\), 1\)。

ReserveN的用法就相对来说复杂一些，当调用完成后，无论Token是否充足，都会返回一个Reservation\*对象。

你可以调用该对象的Delay\(\)方法，该方法返回了需要等待的时间。如果等待时间为0，则说明不用等待。 必须等到等待时间之后，才能进行接下来的工作。

或者，如果不想等待，可以调用Cancel\(\)方法，该方法会将Token归还。

举一个简单的例子，我们可以这么使用Reserve方法。

```text
r := lim.Reserve()
f !r.OK() {
    
    return
}
time.Sleep(r.Delay())
Act() 
```

### 1.1.4. 动态调整速率 <a id="&#x52A8;&#x6001;&#x8C03;&#x6574;&#x901F;&#x7387;"></a>

Limiter支持可以调整速率和桶大小：

```text
SetLimit(Limit) 改变放入Token的速率
SetBurst(int) 改变Token桶大小
```

有了这两个方法，可以根据现有环境和条件，根据我们的需求，动态的改变Token桶大小和速率

实例代码

```text
package main

import (
    "context"
    "log"
    "time"

    "golang.org/x/time/rate"
)






func main() {
    l := rate.NewLimiter(1, 5)
    log.Println(l.Limit(), l.Burst())
    for i := 0; i < 100; i++ {
        
        log.Println("before Wait")
        c, _ := context.WithTimeout(context.Background(), time.Second*2)
        if err := l.Wait(c); err != nil {
            log.Println("limiter wait err:" + err.Error())
        }
        log.Println("after Wait")

        
        r := l.Reserve()
        log.Println("reserve Delay:", r.Delay())

        
        a := l.Allow()
        log.Println("Allow:", a)
    }
}
```

