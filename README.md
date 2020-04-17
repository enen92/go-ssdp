# SSDP library

[![GoDoc](https://godoc.org/github.com/koron/go-ssdp?status.svg)](https://godoc.org/github.com/koron/go-ssdp)
[![CircleCI](https://img.shields.io/circleci/project/github/koron/go-ssdp/master.svg)](https://circleci.com/gh/koron/go-ssdp/tree/master)
[![Go Report Card](https://goreportcard.com/badge/github.com/koron/go-ssdp)](https://goreportcard.com/report/github.com/koron/go-ssdp)

Based on <https://tools.ietf.org/html/draft-cai-ssdp-v1-03>.

## Examples

There are tiny snippets for example.  See also examples/ directory for working
examples.

### Respond to search

```go
import "github.com/koron/go-ssdp"

locationHandler := func(originator net.Addr) string {
	if originator != nil {
		ssdp.Logger.Printf("Location header handler called in the context of an M-SEARCH from %s", originator.String())
	}
	return "http://192.168.0.1:57086/foo.xml"
}

ad, err := ssdp.Advertise(
    "my:device",                        // send as "ST"
    "unique:id",                        // send as "USN"
    locationHandler, 					// callback to get the "LOCATION" header
    "go-ssdp sample",                   // send as "SERVER"
    1800)                               // send as "maxAge" in "CACHE-CONTROL"
if err != nil {
    panic(err)
}

// run Advertiser infinitely.
quit := make(chan bool)
<-quit
```

### Send alive periodically

```go
import "time"

aliveTick := time.Tick(300 * time.Second)

for {
    select {
    case <-aliveTick:
        ad.Alive()
    }
}
```

### Send bye when quiting

```go
import (
    "os"
    "os/signal"
)

// to detect CTRL-C is pressed.
quit := make(chan os.Signal, 1)
signal.Notify(quit, os.Interrupt)

loop:
for {
    select {
    case <-aliveTick:
        ad.Alive()
    case <-quit:
        break loop
    }
}

// send/multicast "byebye" message.
ad.Bye()
// teminate Advertiser.
ad.Close()
```

### Limitate interfaces to multicast

go-ssdp will send multicast messages to all IPv4 interfaces as default.
When you want to limitate interfaces, see below snippet.

```go
import (
    "github.com/koron/go-ssdp"
    "net"
)

en0, err := net.InterfaceByName("en0")
if err != nil {
    panic(err)
}
ssdp.Interfaces = []net.Interface{*en0}
```

go-ssdp will send multicast message only "en0" after this.
