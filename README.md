# Lmstfy client

## Usage

### Initialize

```
import github.com/bitleak/lmstfy/client

c := client.NewLmstfyClient(host, port, namespace, token)

c.ConfigRetry(3, 50) // optional, config the client to retry when some errors happened. retry 3 times with 50ms interval 
```

### Producer example

```
// Publish a job with ttl==forever, tries==3, delay==5s
jobID, err := c.Publish("q1", []byte("hello"), 0, 3, 5)
```

### Consumer example

```
// Consume a job from the q1, if there's not job availble, wait until 12s passed (polling).
// And if this consumer fail to ACK the job in 10s, the job can be retried by other consumers.
job, err := c.Consume("q1", 10, 12)
if err != nil {
    panic(err)
}

// Do something with the `job`
fmt.Println(string(job.Data))

err := c.Ack("q1", job.ID)
if err != nil {
    panic(err)
}
```

```$golang
// Consume 5 jobs from the q1, if there's not job availble, wait until 12s passed (polling).
// If there are some jobs but not enough 5, return jobs as much as possible.
// And if this consumer fail to ACK any job in 10s, the job can be retried by other consumers.
jobs, err := c.BatchConsume("q1", 5, 10, 12)
if err != nil {
    panic(err)
}

// Do something with the `job`
for _, job := range jobs {
    fmt.Println(string(job.Data))
    err := c.Ack("q1", job.ID)
    if err != nil {
        panic(err)
    }
}
```

```$golang
// High-level Consumer API.
// Make it easier to consume jobs continuously and don't need to the concern empty job.
consumer := NewConsumer(&ConsumerConfig{
    Host:          "localhost",
    Port:          "7777",
    Namespace:     "namespace",
    Token:         "token",
    Queues:         []string{"q1"},
    TTR:            30,
})
err := consumer.Receive(context.Background(), func(ctx context.Context, job *Job) {
    // TODO: process the job here
	if err := job.Ack(); err != nil {
    }
})

// Use `consumer.Close()` to stop the consumer 
```
