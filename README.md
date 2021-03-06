# sh
    import "github.com/natefinch/sh"

Package sh is intended to make working with shell commands more shell-like.
This package is basically just syntactic sugar wrapped around
labix.org/v2/pipe, which in turn just wraps os/exec, but it can make your
code a lot easier to read when you have a simple shell-like line rather than
a huge mess of pipes and commands and conversions etc.

#####Example:
``` go
// create functions that runs the echo, grep, and wc shell commands
echo := sh.Cmd("echo")
grep := sh.Cmd("grep")
wc := sh.Cmd("wc")
	
// run echo, pipe the output through grep and then through wc
// effectively the same as
// $ echo Hi there! | grep -o Hi | wc -w
fmt.Print(sh.Pipe(echo("Hi there!"), grep("-o", "Hi"), wc("-w")))
```	
#####Output:
```
1
```


## func Cmd
``` go
func Cmd(name string, args0 ...string) func(args ...string) Executable
```
Cmd returns a function that will return an Executable for the given command
with the given args.  This is a convenience for defining functions for
commonly reused Executables, such as grep, ls, mkdir, etc.

The args that are passed to Cmd are passed to the Executable when the
returned function is run, allowing you to pre-set some common arguments.


##### Example:
``` go	
echo := sh.Cmd("echo")

fmt.Print(echo("Hi there!"))
```

##### Output:
```
Hi there!
```

## func Runner
``` go
func Runner(name string, args0 ...string) func(args ...string) error
```
Runner returns a function that will run the given shell command with
specified arguments. This is a convenience for creating one-off commands that
aren't going to be put into Pipe, but instead just run standalone.

The args that are passed to Runner are passed to the shell command when the
returned function is run, allowing you to pre-set some common arguments.


##### Example:
	
``` go
echo := sh.Runner("echo")
	
// functions created with runner call the underlying shell command
// immediately and return its standard output.
out, err := echo("Hi there!")
fmt.Println(out)
fmt.Println(err)
```
##### Output:

```
Hi there!
<nil>
```

## type Executable
``` go
type Executable struct {
    pipe.Pipe
}
```
Executable is a runnable construct.  You can run it by calling Run(), or by
calling String() (which is automatically done when passing it into a
fmt.Print style function).  It can be passed into Pipe to form a chain of
Executables that are executed in series.









### func Dump
``` go
func Dump(filename string) Executable
```
Dump returns an excutable that will read the given file and dump its contents
as the Executable's stdout.


##### Example:
``` go
grep := sh.Cmd("grep")

name := "ExampleDumpTest"
defer writeTempFile(name, SWCrawl)()

// Equivalent of shell command
// $ cat ExampleDumpTest | grep far
fmt.Print(sh.Pipe(sh.Dump(name), grep("far")))
```

##### Output:
```
A long time ago, in a galaxy far, far away....
```

### func Pipe
``` go
func Pipe(cmds ...Executable) Executable
```
Pipe connects the output of one Executable to the input of the next
Executable in the list.  The result is an Executable that, when run, returns
the output of the last Executable run, and any error it might have had.

If any of the Executables fails, no further Executables are run, and the
failing Executable's stderr and error are returned.


##### Example:

``` go
echo := sh.Cmd("echo")
	
// note, you can "bake in" arguments when you create these functions.
upper := sh.Cmd("tr", "[:lower:]", "[:upper:]")
grep := sh.Cmd("grep")

// Equivalent of shell command:
// $ echo Hi there! | grep -o Hi | wc -w
fmt.Print(sh.Pipe(echo(SWCrawl), grep("far"), upper()))
```

##### Output:

```
A LONG TIME AGO, IN A GALAXY FAR, FAR AWAY....
```

### func PipeWith
``` go
func PipeWith(stdin string, cmds ...Executable) Executable
```
PipeWith functions like Pipe, but runs the first command with stdin as the
input.


##### Example:
``` go
upper := sh.Cmd("tr", "[:lower:]", "[:upper:]")
grep := sh.Cmd("grep")
	
fmt.Print(sh.PipeWith(SWCrawl, grep("far"), upper()))
```	

##### Output:
```
A LONG TIME AGO, IN A GALAXY FAR, FAR AWAY....
```

### func Read
``` go
func Read(r io.Reader) Executable
```
Read returns an executable that will read from the given reader and use it as
the Executable's stdout.


##### Example:

``` go
grep := sh.Cmd("grep")
	
name := "ExampleReadTest"
f, cleanup := openTempFile(name, SWCrawl)
defer cleanup()

fmt.Print(sh.Pipe(sh.Read(f), grep("far")))
```

##### Output:
```
A long time ago, in a galaxy far, far away....
```



### func (Executable) Run
``` go
func (c Executable) Run() (string, error)
```
Run executes the command and returns stdout and a nil error on success, or
stderr and a non-nil error on failure.



### func (Executable) RunWith
``` go
func (c Executable) RunWith(stdin string) (string, error)
```
RunWith executes the command with the given string as standard input, and
returns stdout and a nil error on success, or stderr and a non-nil error on
failure.



### func (Executable) String
``` go
func (c Executable) String() string
```
String runs the Executable and returns the standard output if the command
succeeds, or stderr if the command fails.  If stderr is empty on failure, the
Error() value of the error is returned. This is most useful for passing an
executable into a fmt.Print style function.









- - -
Generated by [godoc2md](http://godoc.org/github.com/davecheney/godoc2md)
