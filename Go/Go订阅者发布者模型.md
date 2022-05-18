## Pub/Sub

发布者订阅者通常由`Publisher`和`Subscriber`组成



## Publish

```go
package main

import (
	"context"
	"sync"
)

// publisher
type hub struct {
	sync.Mutex
	subs map[*subscriber]struct{}
}

// 创建publisher
func newHub() *hub {
	return &hub{
		subs: map[*subscriber]struct{}{},
	}
}

// 发布信息
func (h *hub) publish(ctx context.Context, msg *message) {
	h.Lock()
	for s := range h.subs {
		select {
		case <-ctx.Done():
			return
		case s.handler <- msg:
		}
	}
	h.Unlock()
}

```



## Subscribe

```go
package main

import (
	"context"
	"log"
	"sync"
)

type message struct {
	data []byte
}

type subscriber struct {
	sync.Mutex
	name    string
	handler chan *message
	quit    chan struct{}
}

// 创建subscriber
func newSubscriber(name string) *subscriber {
	return &subscriber{
		name:    name,
		handler: make(chan *message, 100),
		quit:    make(chan struct{}),
	}
}

// 订阅
func (s *subscriber) subscribe(ctx context.Context, h *hub) error {
	h.Lock()
	h.subs[s] = struct{}{}
	h.Unlock()

	// 可以通过cancel()来取消订阅
	go func() {
		select {
		case <-s.quit:
		case <-ctx.Done():
			h.Lock()
			delete(h.subs, s)
			h.Unlock()
		}
	}()

	go s.receive(ctx)
	return nil
}

// 启动subscriber,使其可以接受信息
func (s *subscriber) receive(ctx context.Context) {
	for {
		select {
		case msg := <-s.handler:
			log.Println(s.name, string(msg.data))
		case <-s.quit:
			return
		case <-ctx.Done():
			return
		}
	}
}

// 手动取消订阅
func (s *subscriber) unsubscribe(ctx context.Context, h *hub) error {
	h.Lock()
	delete(h.subs, s)
	h.Unlock()
	close(s.quit)
	return nil
}

```



## Test

通过unsubscribe取消订阅

```go
package main

import (
	"context"
	"time"
)

func main() {
	ctx := context.Background()
	//创建publisher
	hub := newHub()

	//创建subscriber
	sub1 := newSubscriber("sub1")
	sub2 := newSubscriber("sub2")
	sub3 := newSubscriber("sub3")

	//订阅
	sub1.subscribe(ctx, hub)
	sub2.subscribe(ctx, hub)
	sub3.subscribe(ctx, hub)

	//发布
	hub.publish(ctx, &message{data: []byte("test")})
	time.Sleep(1 * time.Second)

	sub3.unsubscribe(ctx, hub)
	hub.publish(ctx, &message{data: []byte("test000")})
	time.Sleep(1 * time.Second)

}

```

通过cancel取消订阅

```go
package main

import (
	"context"
	"time"
)

func main() {
	ctx := context.Background()
	//创建publisher
	hub := newHub()

	//创建subscriber
	sub1 := newSubscriber("sub1")
	sub2 := newSubscriber("sub2")
	sub3 := newSubscriber("sub3")

	//订阅
	sub1.subscribe(ctx, hub)
	sub2.subscribe(ctx, hub)

	cctx, cancel := context.WithCancel(ctx)
	sub3.subscribe(cctx, hub)

	//发布
	hub.publish(ctx, &message{data: []byte("test")})
	time.Sleep(1 * time.Second)

	//通过cancel取消订阅
	cancel()
	hub.publish(ctx, &message{data: []byte("test000")})
	time.Sleep(1 * time.Second)
}

```

