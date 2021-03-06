[![GoDoc](https://godoc.org/github.com/vinbyte/logpaniccollector?status.svg)](https://godoc.org/github.com/vinbyte/logpaniccollector)

# What is it ?

The [Golang](https://golang.org) package for writing log or http panic into file. You can combine this package with [Filebeat](https://www.elastic.co/products/beats/filebeat) or [Promtail](https://github.com/grafana/loki/tree/master/docs/clients/promtail).

## Install

`go get -u github.com/vinbyte/logpaniccollector`

You can also use vendoring tools like [Govendor](https://github.com/kardianos/govendor), [Dep](https://github.com/golang/dep), or something else.

## Docs

<https://godoc.org/github.com/vinbyte/logpaniccollector>

## Usage

Please make your middleware to use the [WritePanic](https://godoc.org/github.com/vinbyte/logpaniccollector#WritePanic) function. Below is the example of Gin middleware

middleware/middleware.go :
```
package middleware

import (
	"fmt"
	"runtime/debug"

	"github.com/gin-gonic/gin"
	"github.com/vinbyte/logpaniccollector"
)

//Middleware ...
type Middleware struct{}

// PanicCatcher is use for collecting panic that happened in endpoint
func (w *Middleware) PanicCatcher(lpc *logpaniccollector.LogPanic) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			isPanic := lpc.RecoverPanic(c.Request.RequestURI, recover())
			if isPanic {
				c.JSON(500, gin.H{
					"status":  500,
					"message": "Internal server error",
				})
				c.Abort()
			}
		}()
		c.Next()
	}
}
```

main.go
```
package main

import (
	"[YOUR_MODULE_NAME]/middleware"
	"github.com/vinbyte/logpaniccollector"
	"github.com/gin-gonic/gin"
)

func main() {
    //...
	r := gin.Default()
	lpc := logpaniccollector.New()
	lpc.LogFile = "log.log"
	lpc.AutoRemoveLog("* * * * *")
	var midd = new(middleware.Middleware)
	r.Use(midd.PanicCatcher(lpc))
    //...
}
```

# To Do

- [x] simplify middleware
- [x] feature cron for auto clean the file
- [ ] feature enable/disable [the Filebeat multiline support](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html) (currently is enabled)
- [ ] feature enable/disable panic log 1 line mode as a follow up from [Promtail issue](https://github.com/grafana/loki/issues/74)
