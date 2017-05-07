## Rabbus

* A tiny wrapper around [amqp](github.com/streadway/amqp) exchanges and queues.
* Auto reconnect to RabbitMQ broker.
* Relies on circuit breaker pattern for sending messages.
* Golang channel API.

## Installation
```bash
go get -u github.com/rafaeljesus/rabbus
```

## Usage
The rabbus package exposes a interface for emitting and listening RabbitMQ messages.

### Emit
```go
import "github.com/rafaeljesus/rabbus"

r, err := rabbus.NewRabbus(rabbus.Config{
  Dsn:      "amqp://guest:guest@localhost:5672",
  Attempts: 5,
  Timeout:  time.Second * 2,
})

for {
  select {
    case r.Emit() <- &Message{
      ExchangeName: "test_ex",
      ExchangeType: "topic",
      RoutingKey:   "test_key",
      Payload:      []byte(`foo`),
      DeliveryMode: Transient,
    }
    case r.EmitOk():
     // message was sent
    case r.EmitErr():
     // failed to send message
  }
}
```

### Listen
```go
import (
  "encoding/json"

  "github.com/rafaeljesus/rabbus"
)

r, err := rabbus.NewRabbus(rabbus.Config{
  Dsn:      "amqp://guest:guest@localhost:5672",
  Attempts: 5,
  Timeout:  time.Second * 2,
})

if err := r.Listen(ListenConfig{
  ExchangeName: "events_ex",
  ExchangeType: "topic",
  RoutingKey:   "events_key",
  QueueName:    "events_q",
  HandlerFunc:  handler,
}); err != nil {
  // handle errors during adding listener
}

func handler(d *Delivery) {
  e := &event{}
  if err := json.NewDecoder(d.Body).Decode(e); err != nil {
    d.Ack(false)
    return
  }

  _, err = doWork(e)
  if err != nil {
    d.Ack(false)
    return
  }

  d.Ack(true)
  return
}
```

## Contributing
- Fork it
- Create your feature branch (`git checkout -b my-new-feature`)
- Commit your changes (`git commit -am 'Add some feature'`)
- Push to the branch (`git push origin my-new-feature`)
- Create new Pull Request

## Badges

[![Build Status](https://circleci.com/gh/rafaeljesus/rabbus.svg?style=svg)](https://circleci.com/gh/rafaeljesus/rabbus)
[![Go Report Card](https://goreportcard.com/badge/github.com/rafaeljesus/rabbus)](https://goreportcard.com/report/github.com/rafaeljesus/rabbus)
[![Go Doc](https://godoc.org/github.com/rafaeljesus/rabbus?status.svg)](https://godoc.org/github.com/rafaeljesus/rabbus)

---

> GitHub [@rafaeljesus](https://github.com/rafaeljesus) &nbsp;&middot;&nbsp;
> Medium [@_jesus_rafael](https://medium.com/@_jesus_rafael) &nbsp;&middot;&nbsp;
> Twitter [@_jesus_rafael](https://twitter.com/_jesus_rafael)
