---
---
Have you ever listen to jazz music live? Doesn't have to be jazz, but it happens many times that musicians start improvising, and when 
they are good, it can be beautiful to listen. Same with MCs when they start improvising with lyrics and rythm, they are writing poems live! 

A couple of days ago I decided to do kind-of-the-same thing while starting a project using the Go language. I guess you could call it freestyle coding. It might be a thing, I don't know. I was just curious of what would happen if I tried, given a vague idea of what I wanted to achieve, to write a program without thinking and as fast as I could. Would it also be magic? Would it also led to beautiful code? 

As I expected, the answer is a big NO. I wrote some ~~f#%(@√g piece of shit~~ code that could be improved substantially. After around 15 minutes of writing (I am new to Go, so I don't know the libraries' APIs etc and takes some time to find how to do things), I came up with what is likely the worst piece of software I have written, and it is full of coding horrors. So, I thought it would be nice to post it here and write about how I would improve its design. Of course my solution may not be perfect, the best or even good to someone's eyes, but this is my party and I do as I want to. 

# Original idea 
Talk about original idea
# Original code and structure
Mention I don't know what the languages package contain yet

Refactoring, top-down approach
- register commands in dictionary. Get command and get runner from the dictionary. The runner executes all that command needs to do, including parsing arguments/flags and extra stuff. This should leave the main function pretty clean

- how to register commands? 
  -- via plug-in package
  -- can init functions do it? For example, by using a main package variable (map)
# Refactor 1: Simplify main function
Clearly, it is not a good idea to put everything in the main function/package. Ideally, commands should be registered before running the main function, which I would like to do something like this:
```go
package main

import (
	"github.com/carlosghabrous/firststone/commands"
)

func main() {
	commands.Run()
}
```

I have no idea how I am going to achieve this at this point, but let's continue and see what happens. 

# Refactor 2: Commands package
The main package now imports the commands package, that should have a function to run some kind of "main" command.
```go
package commands

import (
	"fmt"
	"os"
)

var mainCommand = commands.Command()

func Run() {
	err := mainCommand.Run()
	if err != nil {
		fmt.Printf("Error while running the command: %v\n", err)
		os.Exit(1)
	}
}
```
_mainCommand_ is a variable of type Command, which I have not defined yet, so I guess I should initialize it with some values, like the commands' name, help message etc. Upon reading about the 'if' statements in go, I realized I can re-write the previous one in a more concised manner: 
```go
if err := mainCommand.Run(); err != nil {
		fmt.Printf("Error while running the command: %v\n", err)
		os.Exit(1)
	}
```

Now, let's define the command type. I guess this could be as complex as anybody would want it to be, but I will add the bare minimum in a new file _abstractCommand_, where I will define the types and interfaces.
_abstractCommand_ looks like this now:

```go
package commands

type Command struct {
	Name string
}

type Commander interface {
	Execute()
	AddCmd(cmd Command)
}

func (cmd *Command) Execute() error {
	cmd.executeCommand()
	return nil
}
```
I know, it does nothing at all, but I just want to define types and interfaces so that the code can be compiled. 

