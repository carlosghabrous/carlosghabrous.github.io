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
Clearly, the main function should just get the command from the command line, and let it do the job. I don't know yet how I am going to do it, but I would like it to look like this: 
```go
func main() {
	if len(os.Args) < minimumArgumentsNumber {
		fmt.Println("Missing command!")
		commandsRegistry["help"].run()
		os.Exit(1)
	}

	commandName := os.Args[1]
	command, ok := commandsRegistry[commandName]

	if !ok {
		fmt.Printf("Unknown command %v\n", commandName)
		commandsRegistry["help"].run()
		os.Exit(1)
	}

	command.run(os.Args[2:]...)
```

Here, _commandRegistry_ is some data structure where commands are, well, registered. For now, let it be a map that gets populated in the main's package _init_ function:
```go
type FirstStoneCommand struct {
	cmdName string
}

func (fsCmd FirstStoneCommand) run(cmdFlagsSubs ...string) {
	fmt.Printf("Running command %v\n", fsCmd.cmdName)
}

var commandsRegistry = make(map[string]FirstStoneCommand)

func init() {
	commandsRegistry["init"] = FirstStoneCommand{cmdName: "init"}
	commandsRegistry["help"] = FirstStoneCommand{cmdName: "help"}
}
```
In order to make the code compile, I have added defined the _FirstStoneCommand_ datatype. Variables of this type are able to invoke a method _run_, that executes the given command. The idea is to move this into a package of its own, and expand it to implement the desired functionality. 

But at least the main function has been simplified and cleaned up a great deal!
The result of running a known command is now: 
```
go run . init argument
Running command init with arguments [argument]
```

## Refactor 2: Move command-related stuff into its own package
So, I have created the package... wait for it... _commands_! That sits on its own directory:
--- add screenshot here
Package _commands_ implements the type I defined before plus two more that represent the commands I have so far:
```go
package commands

import "fmt"

type FirstStoneCommand struct {
	CmdName string
}

func (fsCmd FirstStoneCommand) Run(cmdFlagsSubs ...string) {
	fmt.Printf("Running command %v with arguments %v\n", fsCmd.CmdName, cmdFlagsSubs)
}

type InitCommand FirstStoneCommand

func (initCmd InitCommand) Run(cmdFlagsSubs ...string) {
	fmt.Printf("Running init command with args %v\n", cmdFlagsSubs)
}

type HelpCommand FirstStoneCommand

func (helpCmd HelpCommand) Run(cmdFlagsSubs ...string) {
	fmt.Printf("Running help command with args %v\n", cmdFlagsSubs)
}
```

The main package needs to be changed accordingly: 
```go

var commandsRegistry = make(map[string]commands.FirstStoneCommand)

func init() {
	commandsRegistry["init"] = commands.FirstStoneCommand(commands.InitCommand{CmdName: "init"})
	commandsRegistry["help"] = commands.FirstStoneCommand(c
    commands.HelpCommand{CmdName: "help"})
}

```
But notice a cast is needed to allow inserting commands in a map. If we are attempting some kind of polymorphism (I would like to use the commands all the same regardless of the specificity of each one), this smells bad already. 

# Refactor #3: Polymorphism
To solve this, I have refactor commands. _FirstStoneCommand_ becomes a struct _firstStoneCommand_ (private), and there is now an interface that exposes the type's behaviour, _FirstStoneCommand_. I have also created a new type per command and specialized their behaviours. 
The main package's init function looks like this now: 
```go
func init() {
	var c commands.FirstStoneCommand

	c = commands.NewInitCmd()
	commandsRegistry[c.CmdName()] = c

	c = commands.NewHelpCmd()
	commandsRegistry[c.CmdName()] = c
}
```



