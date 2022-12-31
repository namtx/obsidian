
---
title: "Working with Processes"
---

### Working with Processes

#### Primer

##### System calls

The kernel of your Unix system sits atop the hardware of your computer.

It’s the middle man for any interactions that need to happen with the hardware.

This include things like writing/reading from the filesystem, sending data over the network, allocating memory, or playing audio over the speakers.

Given its power, programs are not allowed direct access to the kernel. Any communication is done via system calls.

![[user application.png]]

System calls are at the heart of C programming.

System calls allow you user-space programs to interact indirectly with the hardware of your computer, via the kernel.


##### Processes: The atoms of Unix

Processes are the building block of a Unix system. Why? Because any **code that executed happens inside a process.**

One process can spawn and manage many others.

#### Processes have IDs

Every process running on your system has a unique process identifier. Hereby referred to as `pid`

```ruby
puts Process.pid
```

```shell
$ irb
2.7.2 :001 > Process.pid
 => 69625
2.7.2 :002 >
```

```shell
$ ps aux | grep irb
namtx            69633   0.0  0.0 34122844    864 s005  S+   12:55PM   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox irb
namtx            69625   0.0  0.1 34249756  22564 s002  S+   12:54PM   0:00.79 irb

```

##### Sytem calls
Ruby's `Process.pid` maps to `getpid(2)`
There is alos a global variable that holds the value of the current `pid`. You can access it with `$$`.
```shell
$ irb
2.7.2 :001 > $$
 => 69929
2.7.2 :002 >
```

#### Processes have parents
every process running on your system has a parent process. Each process knows its parent process identifier `ppid`.
In the majority of cases the parent process for a given process is the process that invoked it.

```shell
$ irb
2.7.2 :001 >
2.7.2 :001 > Process.ppid
 => 69398
2.7.2 :002 >
```

```shell
$ ps aux | grep 69398
namtx            70430   0.0  0.0 34122844    864 s003  S+    1:01PM   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox 69398
namtx            69398   0.0  0.1 34191384   8408 s005  Ss   12:54PM   0:02.76 -zsh
```

##### In the real world
There aren't a ton of uses for the `ppid` in the real world. It can be important when detecting deamon processes.

##### System calls
Ruby's `Process.ppid` maps to `getppid(2)`.

#### Processes Have File Descriptors
##### Everything is a File
A part of the Unix philosophy: in the land of Unix `everything is a file`.
This means that devices a treated as files, sockets and pipes are treated as files. and files are treated as files.

##### Descriptors represent resources
Anytime that you open a resource in a running process it is assigned a file descriptor number. File descriptors are not shared between unrelated processes, they live and die with the process they are bound to, just as any open resources for a process are close when it exits.

In Ruby, open resources are presented by the `IO` class. Any `IO` object can have an associated file descriptor number. Use `IO#fileno` to get access to it.

```ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno
3
```

- File descriptor numbers are assigned the lowest unused value.
- Once a resource is closed its file descriptor number becomes available again.

File descriptors keep track of open resources only. Closed resources are not given a file descriptor number.

Trying to read file descriptor number from closed resource will raise ans exception:

```ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno 
3
passwd.close 
puts passwd.fileno
-e:4:in `fileno': closed stream (IOError)
```

You may have notices that when we open a file and ask for its file descriptor number, the lowest value we get is 3. What happend to 0, 1 and 2?

##### Standard Streams
Every Unix process comes with three open resources. These are your standard input (`STDIN`), standard output (`STDOUT`) and standard error (`STDERR`) resources.

These standard resources exits for a very important reason that we take for granted today. `STDIN` provides generic way to read input from keyboard devices or pipes, `STDOUT` and `STDERR` provide generic way to write output to monitors, files, printers, etc. This was one of innovations of Unix.

```ruby
puts STDIN.fileno
0
puts STDOUT.fileno
1
puts STDERR.fileno
2
```

##### In the real world
File descriptors are at the core of network programming using sockets, pipes, etc. and are also at the core of any file system descriptors.

#### Processes Have Resource Limit
##### finding the limit
```ruby
p Process.getrlimit(:NOFILE)
# [256, 9223372036854775807] // [soft_limit, hard_limit]
```

The soft limit isn't really a limit. Meaning that if you exceed the soft limit (in this case by opening more than 256 resouces at once) an exception will be raised, but you can always change that limit if you want.

> hard_limit == Process::RLIM_INFINITY

So any process is able to change its own soft limit, but only superuser can change the hard limit.

However, you process is also able to bump the hard limit assuming it has the required permissions.

If you interested in changing the limit at a system-wide level then start by having a look at `sysctl(8)`.

##### Bump the Soft limit

```ruby
Process.setrlimit(:NOFILE, 4096)
p Process.getrlimit(:NOFILE)
# [4096, 4096]
```

You can see that we set the new limit for the number of open files, and upon asking for that limit again both the hard limit and the soft limit were set to the new value `4-96`.

##### Exceeding the limit

```ruby
Process.setrlimit(:NOFILE, 3)
File.open('/dev/null')
# Errno::EMFILE: Too many open files - /dev/null
```

##### Other resources

```ruby
# The maximum number of simultaneous processes 
# allowed for the current user. 
Process.getrlimit(:NPROC) 
# The largest size file that may be created. 
Process.getrlimit(:FSIZE) 
# The maximum size of the stack segment of the 
# process. 
Process.getrlimit(:STACK)
```

##### In the real world

Needing to modify limits for system resources isn't a common need for most programs.

`httperf(1)` is the http performance tool, and it has to change the soft limit when does something like:
`httperf --hog --server www --num-conn 5000`

#### Processes Have an Environment
- environment variables

- Every process inherits ENVs from its parent. They are set by parent process and inherited by its children processes. Environment variables are per-process and are global to each process.

##### It's a hash, right?
Although `ENV` uses the hash-style accessor API, it's not actually a `Hash`. For instance, it implements `Enumerable` and some of `Hash` API, but not all of it.

##### System calls
- `setenv(3)`

- `getenv(3)`

- `environ(7)`

#### Processes Have Arguments

Every process has access to a special array called `ARGV`. Other programming languages may implement it slightly differently, but every one has something called `argv`.
`argv` is a short form for `argument vector`. In other words: a vector, or array, of arguments.

##### It's an Array!

`ARGV` is simply an `Array`. You can add elements to it, remove elements from it, change the element it contains, whatever you like.

Some libraries will read from `ARGV` to parse command line optinos, for example. You can programmatically change `ARGV` before they have a chance to see it in order to modify the options at runtime.

##### In the real world
- file names
- parsing command line input. There are many Ruby libraries for dealing with command line input. One called `optparse` is available as part of the standard library.

#### Processes Have Names

Unix processes have very few inherent ways of communicating about their state.
Programmers have worked around this and invented things like `logfiles`.
`Logfiles` allow processes to communicate anything they want about their state by writing to the filesystem, but this operates at the level of the filesystem rather than being inherent to the process itself.

Similarly, processes can use the network to open sockets and communicate with other processes. But again, that operates at a different level than the process itself, since it relies on the network.

There are two mechanisms that operate at the level of process itself that can be used to communicate information. One is the process name, the other is exit codes.

##### Naming Processes

Every process on the system has a name.

- `$PROGRAM_NAME` variable in Ruby
You can assign a value to ther global variable to change the name of current process.


#### Processes Have Exit Codes

When a process comes to the end of it has one last chance to make its mark on the world: its `exit code`.
Every process that exits does so with a numeric exit code (0-255) denoting whether it exited successfully or with an error.

Traditionally, a process that exits with an `exit code 0 is said to be succesful`.
Any other exit code denotes an error, with different codes poiting to different errors.

It's usually a good idea to stick with the `0 as success` exit code tradition so that your program will play nicely with other Unix tools.

##### How to exit a Process

- `exit`, `Kernel#exit`

```ruby
exit
```

You can pass a custom exit code to this method

```ruby
exit 22
```

```ruby
# When Kernel#exit is invoked, before exiting Ruby invokes any blocks 
# defined by Kernel#at_exit. 
at_exit { puts 'Last!' }
exit
```

will output:

```console
Last!
```

- `exit!`

is almost exactly the same as `Kernel#exit`, but with two key differences. The first is that it sets unsuccessful status code by default (1), and the second is that it will not invoke any blocks defined using `Kernel#at_exit`

- `abort`

`Kernel#abort` provides a generic way to exit a process unsuccessful. `Kernel#abort` will set the exit code to `1` for the current process.

```ruby
# Will exit with exit code 1. 
abort

# You can pass a message to Kernel#abort. This message will be printed 
# to STDERR before the process exits. 
abort "Something went horribly wrong."

# Kernel#at_exit blocks are invoked when using Kernel#abort. 
at_exit { puts 'Last!' } 
abort "Something went horribly wrong."

Something went horribly wrong. 
Last!
```

- `raise`

A different way to end a process is with an unhandled exception.

Ending process this way still invoke any `at_exit` handlers and will print the exception message and backtrace to `STDERR`

```ruby
# Similar to abort, an unhandled exception will set the exit code to 1. 
raise 'hell'
```

#### Processes Can Fork

Forking is one of the most powerful concepts in Unix programming.
The `fork(2)` system call allows a running process to create new process programmatically. This new process is en exact copy of the original process.
When forking, the process that initiates the `fork(2)` is called the `parent`, and the newly created process is called the `child`.

> The child process inherits a copy of all of the memory in use by the parent proces, as well as open file descriptors belongings to the parent process.

Since the child process is an entirely new process, it gets its own unique `pid`

The parent of the child process is, obviously, its parent process. So its `pid` is set to the pid of the processs that initiated the `fork(2)`.

The child proces inherits any open file descriptors from the parent process at the time of `fork(2)`. It's given the same map of the file descriptor numbers that the parent process has. In this way, the two processes can share open files, sockets, etc.

The child process inherits copy of everything that the parent process has in the main memory. In this way a process could loaded up a large code base, say a Rails app, that occupies 500MB of main mermory. Then this process can fork 2 new child processes. Each of these child processes effectivly have their own copy of that code base loaded in memory.

The call to `fork` returns nearly-instantly so we have 3 processes with each using 500MB of memory. Perfect for when you want to have multiple instances of your application loaded in memory at the same time. Because, only once process needs to load the appl and forking is fast, this method is faster than loading the app 3 times in separate instances.

The child processes would be free to modify their copy of the memory without effecting what the parent process has in memory.

```ruby
if fork
	puts "entered the if block"
else
	puts "entered the else block"
end

# entered the if block 
# entered the else block
```

One call to ther `fork` method actually returns twice. Remember that `fork` create a new process. So it returns once in the calling process parent and once in the newly created process child.

```ruby
puts "parent process pid is #{Process.pid}"

if fork 
	puts "entered the if block from #{Process.pid}"
else
	puts "entered the else block from #{Process.pid}"
end
```

output:
```console
parent process is 21268
entered the if block from 21268
entered the else block from 21282
```

Now it becomes clear that the code in the if block is being excuted by the parent process, while the code in the else block is being excuted by the child process. Both the child process and the parent process will carry on excuting the code after the if construct.

**In the child process `fork` return `nil`.  In the parent process `fork` returns the pid if the newly created child process.**

```ruby
puts fork
21423
nil
```


##### Multicore programming?
By making new process it means that your code is able, but not guaranteed, to be distributed accross multiple CPU cores.

However, there's no guarantee that stuff will be happening in parallel. On a busy system it's possible that all 4 of your processes are handled by the same CPU.

> fork(2) creates a new process that's a copy of the old process. So if a process is using 500MB of main memory, then it forks, now you have 1GB in main memory.
> 
> Do this another ten times and you can quickly exhaust main memory. This is often called a _fork bomb_. Before you turn up the concurrency make sure that you know the consequences.

##### Using a Block

In the example above we've demonstrated `fork` with an if/else construct. It's also possible, and more common in Ruby code, to use `fork` with a block.

When you pass a block to the `fork` method that block will be execute in the new child process, while the parent process simply skips over it. The child process exits when it's done excuting the block. It doesn not continue along the same code path as the parent.

```ruby
for do
  # Code here is only executed in the child process
end

# Code here is only executed in the parent process
```

#### Orphaned Processes
```ruby
fork do
	5.times do
		sleep 1
		puts "I'm an orphan"
	end
end

abort "Parent process die..."
```


##### Abandoned Children

What happens to a child process when its parent dies?

_Nothing_, that is to say, the OS doesn't treat child processes any differently than any other processes. So, when the parent process dies the child process continues on; the parent process does not take the child down with it.

##### Managing Orphans

- `Daemon processes` are long running processes that are intentionally orphaned and meant to stay running forever.
- Communicationg with processes that are not attached to a terminal session. You can do this using something 

#### CoW
As metioned, `fork(2)` creates a new child process that's an exact copy of the parent process. This includes a copy of everything the parent process has in memory.

Physically copying all of data can be considerable overhead, so modern Unix systems employ something called `copy-on-write` sematics (CoW) to combat this.

CoW delays the actual copying on memory until it needs to be written.

So the parent process and child process will actually share the same physical data in memory until one of them need to modify it, at which point the memory will be copied so the proper separation between the two processes can be preserved.

```ruby
arr = [1,2,3] fork do
# At this point the child process has been initialized. 
# Using CoW this process doesn't need to copy the arr variable, 
# since it hasn't modified any shared values it can continue reading 
# from the same memory location as the parent process. 
	p arr
end
```

```ruby
arr = [1,2,3] fork do 
# At this point the child process has been initialized. 
# Because of CoW the arr variable hasn't been copied yet. 
	arr << 4 
# The above line of code modifies the array, so a copy of 
# the array will need to be made for this process before 
# it can modify it. The array in the parent process remains 
# unchanged. 
end
```


> MRI's garbage collector uses a `mark-and-sweep` algorithm. In a nutshell this means that when the GC is invoked it must traverse the graph of live objects, and for each one the GC must `mark` it as alive.
> 
> In MRI <= 1.9, this `mark` step was implemented as a modification to that object in memory. So when the GC was invoked right after a `fork`, all live objects were modified, forcing the OS to make copies of all live Ruby objects and foregoing any benefit from CoW semantics.
> 
> MRI >= 2.0 still uses a `mark-and-sweep` GC, but preserves CoW semantics by storing all of the `marks` in a small data structure in a disparate region of memory. So when the GC runs after a `fork`, this small region of memory must be copied, but the graph of live Ruby objects can be shared between parent and child until your code modifies an object.

#### Processes can wait
- _fire and forget_ is useful when you want a child process to handle something asynchrously, but the parent process still has its own work to do.

```ruby
message = 'Good Morning'
recipient = 'tree@mybackyard.com'

fork do 
	# In this contrived example the parent process forks a child to take 
	# care of sending data to the stats collector. Meanwhile the parent 
	# process has continued on with its work of sending the actual payload. 
	# The parent process doesn't want to be slowed down with this task, and 
	# it doesn't matter if this would fail for some reason. 
	StatsCollector.record message, recipient
end
# send message to recipient
```

- `Process.wait`

```ruby
fork do 
	5.times do
		sleep 1 
		puts "I am an orphan!"
	end 
end
Process.wait
abort "Parent process died..."
```

```console
I am an orphan! 
I am an orphan! 
I am an orphan! 
I am an orphan! 
I am an orphan! 
Parent process died...
```

Control will not be returned to the terminal until all of the output has been printed.

**`Process.wait` is a blocking call instructing the parent process to wait for one of its child processes to exit before continuing.**

```ruby
# We create 3 child processes. 
3.times do 
	fork do 
		# Each one sleeps for a random amount of number less than 5 seconds. 
		sleep rand(5) 
	end
end 
3.times do 
	# We wait for each child process to exit and print the pid that 
	# gets returned. 
	puts Process.wait 
end
```

##### Race Conditions
```ruby
# We create two child processes. 
2.times do 
	fork do 
		# Both processes exit immediately. 
		abort "Finished!" 
	end 
end 
# The parent process waits for the first process, then sleeps for 5 seconds. 
# In the meantime the second child process has exited and is no 
# longer running. 
puts Process.wait 
sleep 5 
# The parent process asks to wait once again, and amazingly enough, the second # process' exit information has been queued up and is returned here. 
puts Process.wait
```


The kernel queues up information about exited processes so that parent always receives the information in order that children exited.

##### In the real World
- _babysitting processes_

At the core of this pattern is the concept that you have one process that forks serveral child processes, for concurrency, and then spends ti

- `waitpid(2)`

#### Zoombie Process

- `fire and forget manner`

Kernel queues up information about child processes that have exited. So even if you `Process.wait` long after process has exited its information is still available.

The kernel will retain the status of exited child process until the parent process requests that status using `Process.wait`.
If the parent never requests the status t hen the kernel will never _reap_ that status information. So creating fire and forget child process without collecting their status information is poor use of kernel resources.
