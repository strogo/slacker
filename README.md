# slacker [![Go Report Card](https://goreportcard.com/badge/github.com/shomali11/slacker)](https://goreportcard.com/report/github.com/shomali11/slacker) [![GoDoc](https://godoc.org/github.com/shomali11/slacker?status.svg)](https://godoc.org/github.com/shomali11/slacker) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Built on top of the Slack API [github.com/nlopes/slack](https://github.com/nlopes/slack) with the idea to simplify the Real-Time Messaging feature to easily create Slack Bots, assign commands to them and extract parameters.

## Features

* Easy definitions of commands and their input
* Simple parsing of String, Integer, Float and Boolean parameters
* Built-in `help` command
* Bot responds to mentions and direct messages
* Handlers run concurrently via goroutines
* Full access to the Slack API [github.com/nlopes/slack](https://github.com/nlopes/slack)

# Usage

Using `govendor` [github.com/kardianos/govendor](https://github.com/kardianos/govendor):

```
govendor fetch github.com/shomali11/slacker
```

Using `go get`:

```
go get github.com/shomali11/slacker
go get github.com/nlopes/slack
```

# Examples

## Example 1

Defining a command using slacker

```go
package main

import (
	"github.com/shomali11/slacker"
	"log"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Command("ping", "Ping!", func(request *slacker.Request, response *slacker.Response) {
		response.Reply("pong")
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Example 2

Adding handlers to when the bot is connected and encounters an error

```go
package main

import (
	"github.com/shomali11/slacker"
	"log"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Init(func() {
		log.Println("Connected!")
	})

	bot.Err(func(err string) {
		log.Println(err)
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Example 3

Defining a command with a parameter

```go
package main

import (
	"github.com/shomali11/slacker"
	"log"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Command("echo <word>", "Echo a word!", func(request *slacker.Request, response *slacker.Response) {
		word := request.Param("word")
		response.Reply(word)
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Example 4

Defining a command with two parameters. Parsing one as a string and the other as an integer. 
_(The second parameter is the default value in case no parameter was passed or could not parse the value)_

```go
package main

import (
	"github.com/shomali11/slacker"
	"log"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Command("repeat <word> <number>", "Repeat a word a number of times!", func(request *slacker.Request, response *slacker.Response) {
		word := request.StringParam("word", "Hello!")
		number := request.IntegerParam("number", 1)
		for i := 0; i < number; i++ {
			response.Reply(word)
		}
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Example 5

Send an error message to the Slack channel

```go
package main

import (
	"errors"
	"github.com/shomali11/slacker"
	"log"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Command("test", "Tests something", func(request *slacker.Request, response *slacker.Response) {
		response.ReportError(errors.New("Oops!"))
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Example 6

Send a "Typing" indicator

```go
package main

import (
	"github.com/shomali11/slacker"
	"log"
	"time"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Command("time", "Server time!", func(request *slacker.Request, response *slacker.Response) {
		response.Typing()

		time.Sleep(time.Second)
		
		response.Reply(time.Now().Format(time.RFC1123))
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```

## Example 7

Showcasing the ability to access the [github.com/nlopes/slack](https://github.com/nlopes/slack) API. 
_In this example, we upload a file using the Slack API._

```go
package main

import (
	"github.com/nlopes/slack"
	"github.com/shomali11/slacker"
	"log"
)

func main() {
	bot := slacker.NewClient("<YOUR SLACK BOT TOKEN>")

	bot.Command("upload <word>", "Upload a word!", func(request *slacker.Request, response *slacker.Response) {
		word := request.Param("word")
		channel := request.Event.Channel
		bot.Client.UploadFile(slack.FileUploadParameters{Content: word, Channels: []string{channel}})
	})

	err := bot.Listen()
	if err != nil {
		log.Fatal(err)
	}
}
```
