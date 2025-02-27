# ZKTECO

Get inspired from [this project](https://github.com/canhlinh/zktecoGo)

ZKTECO is a library to help gopher interact with ZK fingerprint machine

## Features:

- Get all check-in list in a ZK fingerprint machine
- Get all users
- Reatime capturing events

## Getting started

Get source: `go get github.com/sumonskys/zktecoGo`

Sample:

```
package main

import (
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/sumonskys/zktecoGo"
)

func main() {
	zkSocket := zktecoGo.NewZK("192.168.0.201", 4370, 0, zktecoGo.DefaultTimezone)
	if err := zkSocket.Connect(); err != nil {
		panic(err)
	}

	c, err := zkSocket.LiveCapture()
	if err != nil {
		panic(err)
	}

	go func() {
		for event := range c {
			log.Println(event)
		}
	}()

	gracefulQuit(zkSocket.StopCapture)
}

func gracefulQuit(f func()) {
	sigChan := make(chan os.Signal)
	signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)
	go func() {
		<-sigChan

		log.Println("Stopping...")
		f()

		time.Sleep(time.Second * 1)
		os.Exit(1)
	}()

	for {
		time.Sleep(10 * time.Second) // or runtime.Gosched() or similar per @misterbee
	}
}
```
