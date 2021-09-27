---
title: ETCD 服务注册和发现
date: 2021-09-27 11:01:23
updated: 2021-09-27 11:01:23
tags:
	- 微服务
categories:
---


ETCD 服务发现

在微服务中各个服务都是无状态的，不会在配置中写死上下游的访问地址，所以需要有一个地方去维护各个节点的信息。

服务起来的时候会去注册中心拉取其他服务的节点信息，并且把自己的信息推送到注册中心。

当有运行的服务下线或者出现问题的时候，把自己从配置中心摘除，或者配置中心来检测服务状态（心跳、健康检查的协议）来摘除服务。

```golang
package main

import (
    "encoding/json"
    "fmt"
    "github.com/coreos/etcd/clientv3"
    "context"
    "log"
)

type Discovery struct {
    cli       *clientv3.Client
    info      *NodeInfo
    nodes     *NodesManager
}

func NewDiscovery(info *NodeInfo, conf clientv3.Config, mgr *NodesManager) (dis *Discovery, err error) {
    d := &Discovery{}
    d.info = info
    if mgr == nil {
        return nil, fmt.Errorf("[Discovery] mgr == nil")
    }
    d.nodes = mgr
    d.cli, err = clientv3.New(conf)
    return d, err
}

func (d *Discovery) pull() {
    kv := clientv3.NewKV(d.cli)
    resp, err := kv.Get(context.TODO(), "discovery/", clientv3.WithPrefix())
    if err != nil {
        log.Fatalf("[Discovery] kv.Get err:%+v", err)
        return
    }
    for _, v := range resp.Kvs{
        node := &NodeInfo{}
        err = json.Unmarshal(v.Value, node)
        if err != nil {
            log.Fatalf("[Discovery] json.Unmarshal err:%+v", err)
            continue
        }
        d.nodes.AddNode(node)
        log.Printf("[Discovery] pull node:%+v", node)
    }
}

func (d *Discovery) watch() {
    watcher := clientv3.NewWatcher(d.cli)
    watchChan := watcher.Watch(context.TODO(), "discovery", clientv3.WithPrefix())
    for {
        select {
        case resp := <-watchChan:
            d.watchEvent(resp.Events)
        }
    }
}

func (d *Discovery) watchEvent(evs []*clientv3.Event) {
    for _, ev := range evs{
        switch ev.Type{
        case clientv3.EventTypePut:
            node := &NodeInfo{}
            err := json.Unmarshal(ev.Kv.Value, node)
            if err != nil {
                log.Fatalf("[Discovery] json.Unmarshal err:%+v", err)
                continue
            }
            d.nodes.AddNode(node)
            log.Printf(fmt.Sprintf("[Discovery] new node:%s",string(ev.Kv.Value)))
        case clientv3.EventTypeDelete:
            d.nodes.DelNode(string(ev.Kv.Key))
            log.Printf(fmt.Sprintf("[Discovery] del node:%s data:%s",string(ev.Kv.Key),string(ev.Kv.Value)))
        }
    }
}

```




服务注册

在etcd中服务的注册使用租约（lease）来实现，设置租约的时候按需求设置租约的时间（ttl），类似redis中的EXPIRE key，再把服务自身的节点等信息写到对应的key中。然后定时去调用KeepAliveOnce来保持租约，如果在期间KeepAliveOnce的消息丢失或者延迟大于这个租约的ttl则etcd中将会把这个节点的信息删除，恢复正常时重新发起租约流程。

```golang


package main

import (
    "encoding/json"
    "fmt"
    "github.com/coreos/etcd/clientv3"
    "github.com/coreos/etcd/etcdserver/api/v3rpc/rpctypes"
    "github.com/pkg/errors"
    "log"
    "time"
    "context"
)

const (
    _ttl = 10
)


type Register struct {
    cli       *clientv3.Client
    leaseId   clientv3.LeaseID
    lease     clientv3.Lease
    info      *NodeInfo
    closeChan chan error
}

func NewRegister(info *NodeInfo, conf clientv3.Config) (reg *Register, err error) {
    r := &Register{}
    r.closeChan = make(chan error)
    r.info = info
    r.cli, err = clientv3.New(conf)
    return r, err
}


func (r *Register) Run() {
    dur := time.Duration(time.Second)
    timer := time.NewTicker(dur)
    r.register()
    for {
        select {
        case <-timer.C:
            r.keepAlive()
        case <-r.closeChan:
            goto EXIT
        }
    }
EXIT:
    log.Printf("[Register] Run exit...")
}

func (r *Register) Stop() {
    r.revoke()
    close(r.closeChan)
}

func (r *Register) register() (err error) {
    r.leaseId = 0
    kv := clientv3.NewKV(r.cli)
    r.lease = clientv3.NewLease(r.cli)
    leaseResp, err := r.lease.Grant(context.TODO(), _ttl)
    if err != nil {
        err = errors.Wrapf(err, "[Register] register Grant err")
        return
    }
    data, err := json.Marshal(r.info)
    _, err = kv.Put(context.TODO(), r.info.UniqueId, string(data), clientv3.WithLease(leaseResp.ID))
    if err != nil {
        err = errors.Wrapf(err, "[Register] register kv.Put err %s-%+v", r.info.Name, string(data))
        return
    }
    r.leaseId = leaseResp.ID
    return
}

func (r *Register) keepAlive() (err error) {
    _, err = r.lease.KeepAliveOnce(context.TODO(), r.leaseId)
    if err != nil {
        // 租约丢失，重新注册
        if err == rpctypes.ErrLeaseNotFound {
            r.register()
            err = nil
        }
        err = errors.Wrapf(err, "[Register] keepAlive err")
    }
    log.Printf(fmt.Sprintf("[Register] keepalive... leaseId:%+v", r.leaseId))
    return err
}

func(r *Register) revoke() (err error) {
    _, err = r.cli.Revoke(context.TODO(), r.leaseId)
    if err != nil {
        err = errors.Wrapf(err, "[Register] revoke err")
        return
    }
    log.Printf(fmt.Sprintf("[Register] revoke node:%+v", r.leaseId))
    return
}

```

节点管理

```golang
package main

import (
    "log"
    "math/rand"
    "strings"
    "sync"
)

type NodeInfo struct {
    Addr     string
    Name     string
    UniqueId string
}

type NodesManager struct {
    sync.RWMutex
    // <name,<id,node>>
    nodes map[string]map[string]*NodeInfo
}

func NewNodeManager() (m *NodesManager){
    return &NodesManager{
        nodes: map[string]map[string]*NodeInfo{},
    }
}

func (n *NodesManager) AddNode(node *NodeInfo) {
    if node == nil {
        return
    }
    n.Lock()
    defer n.Unlock()
    if _, exist := n.nodes[node.Name]; !exist {
        n.nodes[node.Name] = map[string]*NodeInfo{}
    }
    n.nodes[node.Name][node.UniqueId] = node
}

func (n *NodesManager) DelNode(id string) {
    sli := strings.Split(id,"/")
    name := sli[len(sli)-2]
    n.Lock()
    defer n.Unlock()
    if _, exist := n.nodes[name]; exist {
        delete(n.nodes[name], id)
    }
}

func (n *NodesManager) Pick(name string) *NodeInfo {
    n.RLock()
    defer n.RUnlock()
    if nodes, exist := n.nodes[name]; !exist {
        return nil
    } else {
        // 纯随机取节点
        idx := rand.Intn(len(nodes))
        for _, v := range nodes {
            if idx == 0 {
                return v
            }
            idx--
        }
    }
    return nil
}

func (n *NodesManager) Dump () {
    for k, v := range n.nodes {
        for kk, vv := range v {
            log.Printf("[NodesManager] Name:%s Id:%s Node:%+v", k, kk, vv)
        }
    }

}

```

测试效果

```golang
package main

import (
    "github.com/coreos/etcd/clientv3"
    "log"
    "time"
)

func main() {

    nodes := NewNodeManager()
    dis, _ := NewDiscovery(&NodeInfo{
        Name: "server name/aaaa",
        Addr: "127.0.0.1:8888",
    }, clientv3.Config{
        Endpoints:   []string{"tx.cxc233.com:8888"},
        DialTimeout: 5 * time.Second,
    }, nodes)

    reg, _ := NewRegister(&NodeInfo{
        Name: "testsvr",
        Addr: "127.0.0.1:8888",
        UniqueId: "discovery/testsvr/instance_id/aaabbbccc",
    }, clientv3.Config{
        Endpoints:   []string{"tx.cxc233.com:8888"},
        DialTimeout: 5 * time.Second,
    })

    reg2, _ := NewRegister(&NodeInfo{
        Name: "testsvr",
        Addr: "127.0.0.1:8888",
        UniqueId: "discovery/testsvr/instance_id/testqqqqq",
    }, clientv3.Config{
        Endpoints:   []string{"tx.cxc233.com:8888"},
        DialTimeout: 5 * time.Second,
    })

    go reg.Run()
    time.Sleep(time.Second * 2)
    dis.pull()
    go dis.watch()
    time.Sleep(time.Second * 1)
    go reg2.Run()
    time.Sleep(time.Second * 1)
    nodes.Dump()
    log.Printf("[Main] nodes pick:%+v",nodes.Pick("testsvr"))
    time.Sleep(time.Second * 5)
}
```

运行结果

```bash
2020-10-18 22:36:39.102233 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:40.100365 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:40.100365 I | [Discovery] pull node:&{Addr:127.0.0.1:8888 Name:testsvr UniqueId:discovery/testsvr/instance_id/aaabbbccc}
2020-10-18 22:36:41.100389 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:41.143304 I | [Discovery] new node:{"Addr":"127.0.0.1:8888","Name":"testsvr","UniqueId":"discovery/testsvr/instance_id/testqqqqq"}
2020-10-18 22:36:42.097662 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:42.100819 I | [NodesManager] Name:testsvr Id:discovery/testsvr/instance_id/aaabbbccc Node:&{Addr:127.0.0.1:8888 Name:testsvr UniqueId:discovery/testsvr/instance_id/aaabbbccc}
2020-10-18 22:36:42.100819 I | [NodesManager] Name:testsvr Id:discovery/testsvr/instance_id/testqqqqq Node:&{Addr:127.0.0.1:8888 Name:testsvr UniqueId:discovery/testsvr/instance_id/testqqqqq}
2020-10-18 22:36:42.100819 I | [Main] nodes pick:&{Addr:127.0.0.1:8888 Name:testsvr UniqueId:discovery/testsvr/instance_id/testqqqqq}
2020-10-18 22:36:42.107588 I | [Register] keepalive... leaseId:7587849142468224216
2020-10-18 22:36:43.091650 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:43.109634 I | [Register] keepalive... leaseId:7587849142468224216
2020-10-18 22:36:44.093966 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:44.108130 I | [Register] keepalive... leaseId:7587849142468224216
2020-10-18 22:36:45.093075 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:45.114071 I | [Register] keepalive... leaseId:7587849142468224216
2020-10-18 22:36:46.107634 I | [Register] keepalive... leaseId:7587849142468224212
2020-10-18 22:36:46.114016 I | [Register] keepalive... leaseId:7587849142468224216
2020-10-18 22:36:47.091863 I | [Register] keepalive... leaseId:7587849142468224212
```

