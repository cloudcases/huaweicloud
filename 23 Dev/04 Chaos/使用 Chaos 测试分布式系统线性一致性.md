# 使用 Chaos 测试分布式系统线性一致性

[![img](https://cdn2.jianshu.io/assets/default_avatar/4-3397163ecdb3855a0a4139c34a695885.jpg)](https://www.jianshu.com/u/1yJ3ge)

[siddontang](https://www.jianshu.com/u/1yJ3ge)[![  ](https://upload.jianshu.io/user_badge/11f8cfa8-ec9f-4f82-be92-d6a39f61b5c1)](http://www.jianshu.com/p/d1d89ed69098)关注

0.6192017.10.01 20:23:10字数 2,331阅读 5,918

## 背景

在之前的文章 [测试分布式系统的线性一致性](https://www.jianshu.com/p/bddfce1494d6) 以及 [使用 Porcupine 进行线性一致性测试](https://www.jianshu.com/p/9aedd234ef62) 中，我介绍了 Go 的线性一致性测试工具 [Porcupine](https://link.jianshu.com/?t=https://github.com/anishathalye/porcupine) 以及一些简单使用的例子，这里我将简单介绍一下基于 Porcupine 的一款简单的分布式线性一致性测试框架：[Chaos](https://link.jianshu.com/?t=https://github.com/siddontang/chaos)。

对于分布式系统的线性一致性测试，通常我们都会使用 [jepsen](https://link.jianshu.com/?t=https://github.com/jepsen-io/jepsen)，TiDB 当然也支持 jepsen，那么为啥还是费力的再去捣鼓一个线性一致性测试框架呢？我觉得主要有以下几点：

- Clojure：jepsen 使用的是 clojure，一门跑在 JVM 上面的函数式编程语言。虽然它很强大，但我并不精通。所以每次看 jepsen 的代码对我都是一种折磨，而且我们 team 里面大部分同学也完全不会。
- OOM：jepsen 的 linearizability check 只要稍微跑长一点时间，就非常容易 OOM，所以我们的测试 case 都不会跑特别久。

我其实一直有一个用 Go 写一个线性一致性测试框架的想法，但主要困难在于如何去 linearizability check，幸运的是我找到了 porcupine，自然整个工作就能开动了，于是先捣鼓了一个简单的 chaos，如果可行就继续完善。

## 架构

类似于 jepsen，chaos 也将 DB service 跑在五个 node 上面，node 的命名就是 n1 到 n5，我们也能够通过名字连接到对应的 node 上，譬如我们可以通过 `ssh n1` 就能直接登录到 node n1。

Chaos 也有一个 controller 节点，用来控制整个集群，包括初始化要测试的 DB，创建对应的 client 跑实际的 test，启动 nemesis 来干扰系统，最后验证 history 的 linearizability 等。架构图如下：

![img](https://upload-images.jianshu.io/upload_images/2224-84dbbd50c3bae708.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/720/format/webp)

不同于 jepsen 的地方在于，在 jepsen 里面，controller 是全部通过 ssh 发送命令到 node 节点去执行所有的操作，但 chaos 会在每一个 node 上面启动一个 agent，controller 通过 HTTP API 跟 agent 交互，来操作 node。之所以这么设计，主要就是想用 Go 直接写相关的 DB，nemesis 逻辑，而不是像 jepsen 那样每次用 Linux 的 command 来操作。

但是用 agent 唯一问题在于需要显式的在不同的 node 上面先启动 agent，使用上面比 jepsen 稍微麻烦一点，但也可以通过脚本来搞定。

因为 Go 是一门静态语言，所以如果我们需要在 chaos 里面验证自己 DB 的线性一致性，需要首先实现相关的 interface，然后注册给 chaos，这样 chaos 才能对其验证。这里，我们以 TiDB 为例来进行说明。

## DB

DB interface 对应的就是我们实际要进行测试的 DB，DB interface 定义如下：



```go
type DB interface {
    SetUp(ctx context.Context, nodes []string, node string) error
    TearDown(ctx context.Context, nodes []string, node string) error
    Start(ctx context.Context, node string) error
    Stop(ctx context.Context, node string) error
    Kill(ctx context.Context, node string) error
    IsRunning(ctx context.Context, node string) bool
    Name() string
}
```

我们在 SetUp 函数里面初始化整个 DB 集群，用 TearDown 来析构整个集群。Start，Stop 等函数的含义非常明了，这里不做说明。Name 就是 DB 名字，因为我们是要注册给 chaos 的，所以名字必须唯一，譬如我们的 TiDB 的 Name 就是
“tidb”。

参数 nodes 就是整个集群所在的 Node 信息，通常就是 n1 到 n5，node 就是当前 Node 节点的名字。

在 TiDB 中，我们在 SetUp 函数里面，下载 TiDB binary，解压放到固定位置，然后更新配置文件，然后启动整个集群。而 TearDown 则是直接发送 kill 命令干掉了整个集群。在 Start 函数里面，我们会在每个 Node 上面分别启动 pd-server，tikv-server 和 tidb-server。

当我们实现了 TiDB 的 DB 接口之后，我们就通过 `RegisterDB` 函数将 TiDB 注册到 chaos，这样我们就能在 agent 里面通过 DB name 找到 TiDB 并操作了。

## Client

Client 就是 controller 这边用来跟要测试的 DB 交互的组件。Client interface 定义如下：



```go
type Client interface {
    SetUp(ctx context.Context, nodes []string, node string) error
    TearDown(ctx context.Context, nodes []string, node string) error
    Invoke(ctx context.Context, node string, request interface{}) interface{}
    NextRequest() interface{}
}
```

Invoke 函数就是 Client 实际给 DB 发送命令的接口，因为我们并不知道不同 DB client 的命令参数，所以这里的 request 就是一个 interface。Invoke 执行完毕会返回一个 response，我们也不知道各个 client 实际的 response，也用 interface 来表示。

NextRequest 返回的是下一个可以被 Invoke 的 request，因为只有 client 自己知道如何去构造一个 request。

在 TiDB bank case 里面，我们会定义一个 bank client，每次 NextRequest 的时候会随机选择是查询所有账户的数据，还是选择两个账户进行转账。如果是 read，那么 response 就是查询的数据，如果是 transfer，那么 response 就是是否成功。这里需要注意，对于分布式系统来说，一个操作可能有三种结果，成功，失败和未知，所以我们在 response 这边也需要考虑处理 Unknown 的情况。具体可以参考 [issue](https://link.jianshu.com/?t=https://github.com/anishathalye/porcupine/issues/1#issuecomment-329215748) 上面的讨论。

因为我们有 5 个 Node，controller 这边每个 Node 会有一个 client 对应，所以实际我们也需要实现一个 client creator，用来生成多个 client。



```go
type ClientCreator interface {
    Create(node string) Client
}
```

## Linearizability check

上面我们说到了 client 的接口，我们会用 NextRequest 生成一个 request，然后去 invoke 这个 request，得到一个 response。Controller 这边会将 request 和 response 都记录到一个 history 文件里面。所以一次 operation，是有一个 request 和 一个 response 两个事件的。

为了简单，我们是将 request 和 response 直接用 JSON 编码写入到 history 里面的。当测试跑完之后，我们需要分析这个 history 文件是否是线性一致的。首先，我们就需要去解析这个 history，这里我们需要实现一个 record parser：



```go
type RecordParser interface {
    OnRequest(data json.RawMessage) (interface{}, error)
    OnResponse(data json.RawMessage) (interface{}, error)
    OnNoopResponse() interface{}
}
```

当 parser 读取一行 record 之后，我们会首先判断这行 record 是 request 还是 response，然后调用对应的 RecordParser 接口，再对数据进行解码成实际的类型。

这里需要注意 `OnNoopResposne` 接口，上面说过，所以 Unknown 的 response，我们在 `OnResponse` 这个函数需要返回 nil，让 chaos 先忽略这次事件，然后在最后调用 `OnNoopResposne` 得到一个 Response，补全之前的 operation。

要实现 linearizability check，我们还需要实现自己的 porcupine model，然后调用函数 `VerifyHistory(historyFile string, m porcupine.Model, p RecordParser)` 来对生成的 history 进行验证。

在 TiDB bank 的 porcupine model 关键 step 函数定义如下：



```go
Step: func(state interface{}, input interface{}, output interface{}) (bool, interface{}) {
    st := state.([]int64)
    inp := input.(bankRequest)
    out := output.(bankResponse)

    if inp.Op == 0 {
        // read
        ok := out.Unknown || reflect.DeepEqual(st, out.Balances)
        return ok, state
    }

    // for transfer
    if !out.Ok && !out.Unknown {
        return true, state
    }

    newSt := append([]int64{}, st...)
    newSt[inp.From] -= inp.Amount
    newSt[inp.To] += inp.Amount
    return out.Ok || out.Unknown, newSt
}
```

如果是 read 操作，那么就判断这次的结果跟上次状态的是否一致，或者是否是 Unknown，如果是 transfer，那么就在现有状态上面，执行一次转账操作，返回新的状态。

## Nemesis

在跑测试的时候，controller 也会定期的执行一些 nemesis 操作去干扰整个系统，譬如一下子 kill 所有的 DB，或者 drop 掉相关从一些 Node 上面发过来的网络包这些。Nemesis interface 定义如下：



```go
type Nemesis interface {
    Invoke(ctx context.Context, node string, args ...string) error
    Recover(ctx context.Context, node string, args ...string) error
    Name() string
}
```

因为 nemesis 也是要注册给 chaos 使用，所以 Name 必须唯一。我们使用 Invoke 对系统干扰，然后 Recover 恢复系统。当实现了自己的 nemesis 之后，也需要调用 `RegisterNemesis` 来进行注册，这样 agent 才能使用。

在 controller 这边，我们需要实现 NemesisGenerator：



```go
type NemesisGenerator interface {
    Generate(nodes []string) []*NemesisOperation
    Name() string
}
```

Generate 会对每个 Node 生成一个 NemesisOperation 操作，NemesisOperation 里面就定义了需要执行的 nemesis，以及相关的参数，和执行时间。Controller 会将 NemesisOperation 发送给 agent，让 agent 去执行对应的 nemesis。

## Agent and Controller

当我们定义好自己的 DB，Client，Nemesis 等之后，我们就需要将其整合到一起了。我们需要在 agent 里面首先注册自己的 DB 以及相关的 nemesis。在 `cmd/agent/main.go` 文件里面，TiDB 相关的注册代码如下：



```go
    // register nemesis
_ "github.com/siddontang/chaos/pkg/nemesis"

// register tidb
_ "github.com/siddontang/chaos/tidb"
```

然后启动 node， 之后我们通过



```go
NewController(cfg *Config, clientCreator core.ClientCreator, nemesisGenerators []core.NemesisGenerator) *Controller`
```

创建一个 controller，controller 需要接受一个 ClientCreator 以及一个 nemesis generator 的列表。Config 里面会指定这次测试每个 client 最多发送的 request 个数，以及整个测试执行的时间，以及要操作的 DB name 等。

启动 controller，执行测试，最后结束之后，会有一个 history 文件生成，我们就可以验证线性一致性了。

## 总结

Chaos 现阶段只是一个非常初级的版本，还有很多工作需要完善，譬如更好的 interface 定义，更易于使用这些。但现在至少是能 work 的，现在只有 TiDB 的转账测试，后面，我会给 TiDB 多加入几个线性一致性测试，如果大家感兴趣，也欢迎加入其他开源项目的线性一致性测试 case。

chaos: [https://github.com/siddontang/chaos](https://link.jianshu.com/?t=https://github.com/siddontang/chaos)



https://www.jianshu.com/p/2e65e6f37c76