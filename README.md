# Machine [![GoDoc](https://godoc.org/github.com/autom8ter/machine/v2?status.svg)](https://godoc.org/github.com/autom8ter/machine/v2)

![concurrency](images/concurrency.jpg)


`import "github.com/autom8ter/machine/v2"`

Machine is an interface for highly asynchronous Go applications

```go
type Machine interface {
	// Publish synchronously publishes the Message to all subscribers of the msg.channel
	Publish(ctx context.Context, msg Message)
	// Subscribe synchronously subscribes to messages published on a given channel, executing the given Handler UNTIL the context cancels OR false is returned by the Handler function.
	// Glob matching IS supported for subscribing to multiple channels at once.
	Subscribe(ctx context.Context, channel string, handler MessageHandlerFunc, opts ...SubscriptionOpt)
	// Go asynchronously executes the given Func
	Go(ctx context.Context, fn Func)
	// Cron asynchronously executes the given function on a timed interval UNTIL the context cancels OR false is returned by the Cron function
	Cron(ctx context.Context, interval time.Duration, fn CronFunc)
	// Wait blocks until all active routine's exit
	Wait()
	// Close blocks until all active routine's exit then closes all subscriptions
	Close()
}
```

Example:

```go
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
  	defer cancel()
  	var (
  		m       = machine.New()
  		results []string
  		mu      sync.RWMutex
  	)
  	defer m.Close()
  
  	m.Go(ctx, func(ctx context.Context) error {
  		m.Subscribe(ctx, "accounting.*", func(ctx context.Context, msg machine.Message) (bool, error) {
  			mu.Lock()
  			results = append(results, fmt.Sprintf("(%s) received msg: %v\n", msg.GetChannel(), msg.GetBody()))
  			mu.Unlock()
  			return true, nil
  		})
  		return nil
  	})
  	m.Go(ctx, func(ctx context.Context) error {
  		m.Subscribe(ctx, "engineering.*", func(ctx context.Context, msg machine.Message) (bool, error) {
  			mu.Lock()
  			results = append(results, fmt.Sprintf("(%s) received msg: %v\n", msg.GetChannel(), msg.GetBody()))
  			mu.Unlock()
  			return true, nil
  		})
  		return nil
  	})
  	m.Go(ctx, func(ctx context.Context) error {
  		m.Subscribe(ctx, "human_resources.*", func(ctx context.Context, msg machine.Message) (bool, error) {
  			mu.Lock()
  			results = append(results, fmt.Sprintf("(%s) received msg: %v\n", msg.GetChannel(), msg.GetBody()))
  			mu.Unlock()
  			return true, nil
  		})
  		return nil
  	})
  	m.Go(ctx, func(ctx context.Context) error {
  		m.Subscribe(ctx, "*", func(ctx context.Context, msg machine.Message) (bool, error) {
  			mu.Lock()
  			results = append(results, fmt.Sprintf("(%s) received msg: %v\n", msg.GetChannel(), msg.GetBody()))
  			mu.Unlock()
  			return true, nil
  		})
  		return nil
  	})
  	<-time.After(1 * time.Second)
  	m.Publish(ctx, machine.Msg{
  		Channel: "human_resources.chat_room6",
  		Body:    "hello world human resources",
  	})
  	m.Publish(ctx, machine.Msg{
  		Channel: "accounting.chat_room2",
  		Body:    "hello world accounting",
  	})
  	m.Publish(ctx, machine.Msg{
  		Channel: "engineering.chat_room1",
  		Body:    "hello world engineering",
  	})
  	m.Wait()
  	sort.Strings(results)
  	for _, res := range results {
  		fmt.Print(res)
  	}
  	// Output:
  	//(accounting.chat_room2) received msg: hello world accounting
  	//(accounting.chat_room2) received msg: hello world accounting
  	//(engineering.chat_room1) received msg: hello world engineering
  	//(engineering.chat_room1) received msg: hello world engineering
  	//(human_resources.chat_room6) received msg: hello world human resources
  	//(human_resources.chat_room6) received msg: hello world human resources
```

[Machine](https://pkg.go.dev/github.com/autom8ter/machine/v2#Machine) is a zero dependency library for highly concurrent Go applications. 
It is inspired by [`errgroup`](https://pkg.go.dev/golang.org/x/sync/errgroup)`.`[`Group`](https://pkg.go.dev/golang.org/x/sync/errgroup#Group) with extra bells & whistles:
- In memory Publish Subscribe for asynchronously broadcasting & consuming messages in memory
- Asynchronous worker groups similar to errgroup.Group
- Throttled max active goroutine count
- Asynchronous error handling(see `WithErrorHandler` to override default error handler)
- Asynchronous cron jobs- `Cron()`

## Use Cases

[Machine](https://pkg.go.dev/github.com/autom8ter/machine#Machine) is meant to be completely agnostic and dependency free- its use cases are expected to be emergent.
Really, it can be used **anywhere** goroutines are used. 

Highly concurrent and/or asynchronous applications include:

- gRPC streaming servers

- websocket servers

- pubsub servers

- reverse proxies

- cron jobs

- custom database/cache

- ETL pipelines

- log sink

- filesystem walker

- code generation

## Examples

All examples are < 500 lines of code(excluding code generation)

- [gRPC Bidirectional Chat Stream Server](examples/README.md#grpc-bidirectional-chat-server)
- [TCP Reverse Proxy](examples/README.md#tcp-reverse-proxy)
- [Concurrent Cron Job Server](examples/README.md#concurrent-cron-server)


## Examples (v2)

All examples are < 500 lines of code(excluding code generation)

- [gRPC Bidirectional Chat Stream Server](v2/examples/README.md#grpc-bidirectional-chat-server)

