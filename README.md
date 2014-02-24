
# sh
    import "github.com/natefinch/sh"

Package sh is intended to make working with shell commands more shell-like.
This package is basically just syntactic sugar wrapped around os/exec, but it
can make your code a lot easier to read when you have a simple shell-like
line rather than a huge mess of pipes and commands.


	fmt.Print(sh.Pipe(echo("Hi there!"), grep("Hi", "-o"), wc("-w")))
	// output:
	// 1






## func Cmd
``` go
func Cmd(name string, args ...string) func(args ...string) Command
```
Cmd returns a function that runs the given command with the given args.  The
args that are passed to Cmd cannot be overridden, but the function that is
returned can add more arguments when you call it.


## func Pipe
``` go
func Pipe(cmds ...Command) pipeResult
```
Pipe connects the output of one Command to the input of the next Command,
returning immediately if any of the commands produce an error.  The result is
a function that returns the output of the last function run, and any error it
might have had.  Alternatively, you can call String() on the result to get
just the output. You may also pass the result into a fmt.Print-style function
as if it were a string.



## type Command
``` go
type Command func(arg ...string) (string, error)
```
Command is the type that is returned from running a Cmd.  You can pass it
into a fmt.Print style function as if it were a string and it'll do the right
thing, or you can execute it to get the output and any errors, or call
String() to just get the output.











### func (Command) String
``` go
func (c Command) String() string
```
String implements Stringer.String









- - -
Generated by [godoc2md](http://godoc.org/github.com/davecheney/godoc2md)