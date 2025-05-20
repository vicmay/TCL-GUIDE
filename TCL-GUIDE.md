# TCL Cheatsheet

## Table of Contents
- [Basic Syntax](#basic-syntax)
- [Variables](#variables)
- [Control Structures](#control-structures)
- [Procedures](#procedures)
- [Data Structures](#data-structures)
- [String Operations](#string-operations)
- [File I/O](#file-io)
- [Error Handling](#error-handling)
- [Common Commands](#common-commands)
- [Best Practices](#best-practices)
- [Package Management](#package-management)
- [Common Pitfalls](#common-pitfalls)
- [Resources](#resources)
- [Namespaces](#namespaces)
- [Object-Oriented Programming](#object-oriented-programming)
- [Event Loop and Asynchronous Programming](#event-loop-and-asynchronous-programming)
- [Threading](#threading)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Internationalization and Localization](#internationalization-and-localization)
- [Testing Frameworks](#testing-frameworks)
- [Debugging Tools and Techniques](#debugging-tools-and-techniques)
- [Code Style and Documentation](#code-style-and-documentation)
- [Build Systems and Deployment](#build-systems-and-deployment)
- [Database Integration](#database-integration)
- [Network Programming](#network-programming)
- [Text Encodings and Character Sets](#text-encodings-and-character-sets)
- [Array Commands](#array-commands)
- [Regular Expressions](#regular-expressions)

## Basic Syntax

```tcl
# Comments start with #
set var value;        # Set a variable
puts "Hello, World!";  # Print to stdout
```

## Variables

```tcl
set var 5;           # Set a variable
set str "Hello";     # String variable
set lst {a b c};     # List variable
set arr(index) 10;   # Array element

# Variable substitution
set x 10
puts "Value is $x";  # => Value is 10
puts [expr $x * 2];  # Command substitution => 20
```

## Namespaces

Namespaces in Tcl provide a way to organize code and avoid naming conflicts. This section covers namespace management and best practices.

### Basic Usage

#### Creating and Using Namespaces

```tcl
# Create a namespace
namespace eval ::mylib {
    # Variables in namespace
    variable counter 0
    variable config [dict create]
    
    # Procedures in namespace
    proc init {} {
        variable counter
        set counter 0
    }
    
    proc increment {} {
        variable counter
        incr counter
        return $counter
    }
}

# Access namespace procedures
::mylib::init
set value [::mylib::increment]
```

#### Namespace Variables

```tcl
namespace eval ::mylib {
    # Declare variables
    variable x 10
    variable y 20
    
    # Access variables
    proc get_sum {} {
        variable x
        variable y
        return [expr {$x + $y}]
    }
}
```

### Advanced Usage

#### Nested Namespaces

```tcl
namespace eval ::mylib {
    namespace eval utils {
        proc helper {} {
            return "Helper function"
        }
    }
    
    namespace eval db {
        proc connect {} {
            return "Connected to database"
        }
    }
}

# Access nested namespaces
::mylib::utils::helper
::mylib::db::connect
```

#### Importing and Exporting

```tcl
namespace eval ::mylib {
    # Export procedures
    namespace export init increment
    
    # Define procedures
    proc init {} { ... }
    proc increment {} { ... }
    proc internal {} { ... }  # Not exported
}

# Import specific procedures
namespace import ::mylib::init
namespace import ::mylib::increment

# Use imported procedures
init
increment
```

#### Namespace Management

```tcl
# List all namespaces
namespace children ::

# List procedures in namespace
info procs ::mylib::*

# Check if namespace exists
namespace exists ::mylib

# Delete namespace
namespace delete ::mylib
```

### Best Practices

- Use `::` prefix for global namespace
- Use meaningful namespace names
- Export only necessary procedures
- Use `variable` command to declare namespace variables
- Use `namespace eval` for namespace creation
- Use `namespace import` carefully
- Use `namespace export` to control visibility

### Common Pitfalls

- Forgetting to declare namespace variables
- Confusing global and namespace variables
- Overusing namespace imports
- Not using proper namespace qualification

### Resources

- [Tcl Documentation on namespace](https://www.tcl.tk/man/tcl/TclCmd/namespace.htm)
- [Tclers Wiki on namespace](https://wiki.tcl-lang.org/page/namespace)

## Object-Oriented Programming

TclOO (Tcl Object-Oriented) is Tcl's built-in object system, introduced in Tcl 8.6. This section covers the comprehensive usage of TclOO.

### Basic Class Definition

```tcl
# Create a basic class
oo::class create Animal {
    # Constructor
    constructor {name} {
        my variable name
        set name $name
    }
    
    # Method
    method speak {} {
        my variable name
        return "$name makes a sound"
    }
}

# Create an instance
set dog [Animal new "Rex"]
puts [$dog speak];  # => Rex makes a sound
```

### Class Features

#### Variables and Methods

```tcl
oo::class create Person {
    # Class variable (shared by all instances)
    variable classVar
    common classVar 0
    
    # Instance variables
    variable name age
    
    # Constructor
    constructor {n a} {
        set name $n
        set age $a
        incr classVar
    }
    
    # Instance method
    method getInfo {} {
        return "Name: $name, Age: $age"
    }
    
    # Class method
    classmethod getCount {} {
        return $classVar
    }
}
```

#### Inheritance

```tcl
# Base class
oo::class create Animal {
    variable name
    
    constructor {n} {
        set name $n
    }
    
    method speak {} {
        return "$name makes a sound"
    }
}

# Derived class
oo::class create Dog {
    superclass Animal
    
    method speak {} {
        my variable name
        return "$name barks"
    }
    
    method wag {} {
        return "Tail wagging"
    }
}

# Usage
set dog [Dog new "Rex"]
puts [$dog speak];  # => Rex barks
puts [$dog wag];   # => Tail wagging
```

### Advanced Features

#### Mixins

```tcl
# Define a mixin
oo::class create Logger {
    method log {message} {
        puts "[clock format [clock seconds]]: $message"
    }
}

# Use mixin in a class
oo::class create LoggedPerson {
    mixin Logger
    variable name
    
    constructor {n} {
        set name $n
        my log "Created person: $n"
    }
}
```

#### Filters

```tcl
oo::class create SecureClass {
    variable password
    
    constructor {pwd} {
        set password $pwd
    }
    
    # Filter to check password before method execution
    filter checkPassword
    
    method checkPassword {method args} {
        if {[lindex $args 0] ne $password} {
            error "Invalid password"
        }
        return [next {*}[lrange $args 1 end]]
    }
    
    method secret {pwd data} {
        return "Secret data: $data"
    }
}
```

#### Forwarding

```tcl
oo::class create Wrapper {
    variable obj
    
    constructor {object} {
        set obj $object
    }
    
    # Forward all unknown methods to the wrapped object
    forward unknown $obj
}

# Usage
set list [list 1 2 3]
set wrapper [Wrapper new $list]
puts [$wrapper llength];  # => 3
```

### Object Lifecycle

```tcl
oo::class create Resource {
    variable handle
    
    constructor {} {
        set handle [open "file.txt" w]
    }
    
    # Destructor
    destructor {
        if {[info exists handle]} {
            close $handle
        }
    }
    
    method write {data} {
        puts $handle $data
    }
}
```

### Best Practices

- Use meaningful class and method names
- Keep classes focused and single-purpose
- Use proper access control (public/private methods)
- Implement proper error handling
- Use destructors for cleanup
- Document class interfaces
- Use mixins for cross-cutting concerns
- Use filters for method interception
- Use forwarding for composition

### Common Pitfalls

- Forgetting to call `next` in filters
- Not properly initializing variables
- Circular inheritance
- Not cleaning up resources
- Overusing inheritance
- Not using proper access control
- Forgetting to call superclass constructors

### Next Scripting Framework (NX)

NX is an advanced object system for Tcl that provides additional features beyond TclOO. It's particularly useful for complex applications and when you need more sophisticated object-oriented capabilities.

#### Key Features

```tcl
# Basic NX class definition
nx::Class create Person {
    :property name:required
    :property age:integer
    
    :method getInfo {} {
        return "Name: ${:name}, Age: ${:age}"
    }
}

# Create instance
set person [Person new -name "John" -age 30]
puts [$person getInfo]
```

#### Advanced Features

```tcl
# Multiple inheritance
nx::Class create Employee {
    :superclass Person
    :property salary:integer
    
    :method getInfo {} {
        return "[next] Salary: ${:salary}"
    }
}

# Mixins with NX
nx::Class create Logger {
    :method log {message} {
        puts "[clock format [clock seconds]]: $message"
    }
}

nx::Class create LoggedEmployee {
    :superclass Employee
    :mixin Logger
    
    :method work {} {
        :log "Employee ${:name} is working"
    }
}
```

#### NX vs TclOO

- NX provides more sophisticated property management
- Better support for multiple inheritance
- More powerful mixin system
- Built-in validation and constraints
- Better support for method chaining
- More intuitive syntax for common operations
- Better performance for complex object hierarchies

#### When to Use NX

- Complex applications requiring sophisticated OO features
- When you need property validation and constraints
- When working with complex inheritance hierarchies
- When you need better performance for large object systems
- When you need more intuitive syntax for common operations

### Resources

- [Tcl Documentation on TclOO](https://www.tcl.tk/man/tcl/TclCmd/class.htm)
- [Tclers Wiki on TclOO](https://wiki.tcl-lang.org/page/TclOO)
- [TclOO Tutorial](https://wiki.tcl-lang.org/page/TclOO+tutorial)
- [Next Scripting Framework Documentation](https://next-scripting.org/)
- [NX GitHub Repository](https://github.com/aplsimple/nx)

## Event Loop and Asynchronous Programming

Tcl's event loop is a powerful feature that enables non-blocking I/O, timers, and event-driven programming. This section covers the comprehensive usage of Tcl's event system.

### Basic Event Loop

```tcl
# Start the event loop
vwait forever

# Or with a specific variable
set done 0
vwait done

# Exit the event loop
set done 1
```

### Timers and Delayed Execution

```tcl
# Basic timer
after 1000 {
    puts "This runs after 1 second"
}

# Cancel a timer
set timer [after 1000 {
    puts "This won't run"
}]
after cancel $timer

# Repeating timer
proc repeat_timer {} {
    puts "Tick"
    after 1000 repeat_timer
}
repeat_timer
```

### File Events

```tcl
# Set up file event handling
set sock [socket $host $port]
fconfigure $sock -buffering line

# Read event
fileevent $sock readable [list handle_read $sock]

# Write event
fileevent $sock writable [list handle_write $sock]

# Remove event handler
fileevent $sock readable {}

# Example handlers
proc handle_read {sock} {
    if {[eof $sock]} {
        close $sock
        return
    }
    set line [gets $sock]
    puts "Received: $line"
}

proc handle_write {sock} {
    puts $sock "Hello, server!"
    fileevent $sock writable {}  # Remove write handler after sending
}
```

### Asynchronous I/O

```tcl
# Non-blocking file operations
set fh [open "large_file.txt" r]
fconfigure $fh -blocking 0
fileevent $fh readable [list read_chunk $fh]

proc read_chunk {fh} {
    if {[eof $fh]} {
        close $fh
        return
    }
    set data [read $fh 8192]  # Read 8KB chunks
    process_data $data
}
```

### Coroutines

```tcl
# Basic coroutine
coroutine counter apply {{} {
    set i 0
    while 1 {
        yield $i
        incr i
    }
}}

# Use coroutine
puts [counter]  # => 0
puts [counter]  # => 1
puts [counter]  # => 2

# Coroutine with state
coroutine state_machine apply {{} {
    set state "idle"
    while 1 {
        set cmd [yield $state]
        switch $cmd {
            "start" { set state "running" }
            "stop"  { set state "stopped" }
            "reset" { set state "idle" }
        }
    }
}}

# Use state machine
puts [state_machine]     # => idle
puts [state_machine start]  # => running
puts [state_machine stop]   # => stopped
```

### Event Loop Best Practices

```tcl
# Proper event loop shutdown
proc shutdown {} {
    global timers sockets
    # Cancel all timers
    foreach timer $timers {
        after cancel $timer
    }
    # Close all sockets
    foreach sock $sockets {
        catch {close $sock}
    }
    # Exit event loop
    set ::done 1
}

# Error handling in event handlers
proc safe_handler {args} {
    if {[catch {
        eval $args
    } err]} {
        puts "Error in event handler: $err"
        puts $::errorInfo
    }
}

# Resource cleanup
proc cleanup {sock} {
    fileevent $sock readable {}
    fileevent $sock writable {}
    catch {close $sock}
}
```

### Advanced Event Loop Features

#### Event Loop Control

```tcl
# Update the event loop
update

# Update without processing events
update idletasks

# Process pending events
update

# Check if event loop is running
if {[info exists tcl_interactive]} {
    puts "Running in interactive mode"
}
```

#### Custom Event Sources

```tcl
# Create a custom event source
proc create_custom_source {} {
    set source [list]
    trace add variable source write [list handle_source_change]
    return $source
}

proc handle_source_change {name1 name2 op} {
    # Handle source changes
    after 0 [list process_source_changes]
}
```

### Best Practices

- Always use non-blocking I/O for network operations
- Clean up resources properly (timers, sockets, file handlers)
- Handle errors in event callbacks
- Use coroutines for complex state machines
- Avoid long-running operations in event handlers
- Use proper buffering settings for I/O
- Implement proper shutdown procedures
- Use timeouts for operations that might hang

### Common Pitfalls

- Blocking the event loop with long operations
- Not cleaning up resources
- Forgetting to handle errors in callbacks
- Using blocking I/O in event-driven code
- Not implementing proper shutdown
- Memory leaks from unclosed resources
- Race conditions in event handlers
- Deadlocks in coroutines

### Resources

- [Tcl Documentation on event loop](https://www.tcl.tk/man/tcl/TclCmd/vwait.htm)
- [Tcl Documentation on coroutines](https://www.tcl.tk/man/tcl/TclCmd/coroutine.htm)
- [Tclers Wiki on event loop](https://wiki.tcl-lang.org/page/event+loop)
- [Tclers Wiki on coroutines](https://wiki.tcl-lang.org/page/coroutine)

## Threading

Tcl provides built-in support for multi-threading through the Thread package. This section covers thread creation, management, and inter-thread communication.

### Basic Thread Operations

```tcl
package require Thread

# Create a new thread
set thread [thread::create]

# Create thread with initialization script
set thread [thread::create {
    package require http
    proc fetch_url {url} {
        set token [http::geturl $url]
        set data [http::data $token]
        http::cleanup $token
        return $data
    }
}]

# Send command to thread
thread::send $thread {fetch_url "http://example.com"}

# Wait for result
set result [thread::send -async $thread {fetch_url "http://example.com"}]
vwait $result

# Get thread ID
set tid [thread::id]

# Check if running in thread
if {[thread::is_main]} {
    puts "Running in main thread"
} else {
    puts "Running in worker thread"
}
```

### Thread Safety

```tcl
# Thread-safe variable access
thread::mutex create mutex
thread::rwlock create rwlock

# Using mutex
thread::mutex lock mutex
# Critical section
thread::mutex unlock mutex

# Using read-write lock
thread::rwlock rdlock rwlock
# Read operations
thread::rwlock unlock rwlock

thread::rwlock wrlock rwlock
# Write operations
thread::rwlock unlock rwlock
```

### Thread Pools

```tcl
# Create thread pool
set pool [thread::pool create -min 2 -max 4]

# Submit work to pool
thread::pool post $pool {
    # Work to be done
    after 1000
    return "Work done"
}

# Wait for all work to complete
thread::pool wait $pool

# Get pool statistics
set stats [thread::pool stats $pool]
```

### Inter-Thread Communication

```tcl
# Create message queue
set queue [thread::queue create]

# Send message
thread::queue put $queue "Hello from thread [thread::id]"

# Receive message
set message [thread::queue get $queue]

# Non-blocking receive
if {[thread::queue size $queue] > 0} {
    set message [thread::queue get $queue]
}
```

### Thread-Safe Data Structures

```tcl
# Thread-safe dictionary
set dict [thread::shared create]

# Thread-safe list
set list [thread::shared create]

# Thread-safe array
set array [thread::shared create]

# Access shared data
thread::shared lock $dict
dict set $dict key value
thread::shared unlock $dict
```

### Advanced Threading Patterns

#### Producer-Consumer

```tcl
# Create shared queue
set queue [thread::queue create]

# Producer thread
set producer [thread::create {
    proc produce {queue} {
        for {set i 0} {$i < 10} {incr i} {
            thread::queue put $queue "Item $i"
            after 100
        }
    }
}]

# Consumer thread
set consumer [thread::create {
    proc consume {queue} {
        while 1 {
            set item [thread::queue get $queue]
            puts "Consumed: $item"
        }
    }
}]

# Start producer and consumer
thread::send $producer [list produce $queue]
thread::send $consumer [list consume $queue]
```

#### Thread Pool with Work Queue

```tcl
# Create work queue and thread pool
set queue [thread::queue create]
set pool [thread::pool create -min 2 -max 4]

# Worker procedure
proc worker {queue} {
    while 1 {
        set work [thread::queue get $queue]
        # Process work
        after 1000
    }
}

# Submit work
for {set i 0} {$i < 10} {incr i} {
    thread::queue put $queue "Work $i"
}

# Start workers
for {set i 0} {$i < 4} {incr i} {
    thread::pool post $pool [list worker $queue]
}
```

### Best Practices

- Use thread pools for CPU-bound tasks
- Use message queues for thread communication
- Always use mutexes for shared data access
- Keep critical sections small
- Use read-write locks when appropriate
- Clean up resources properly
- Handle thread errors gracefully
- Use thread-safe data structures
- Avoid thread starvation
- Implement proper shutdown procedures

### Common Pitfalls

- Deadlocks from improper mutex usage
- Race conditions in shared data access
- Memory leaks from unclosed resources
- Thread starvation
- Improper error handling
- Not cleaning up threads
- Using non-thread-safe commands
- Blocking the main thread
- Forgetting to unlock mutexes
- Not handling thread exceptions

### Resources

- [Tcl Documentation on Thread](https://www.tcl.tk/man/tcl/ThreadCmd/thread.htm)
- [Tclers Wiki on Thread](https://wiki.tcl-lang.org/page/thread)
- [Thread Package Documentation](https://www.tcl.tk/man/tcl/ThreadCmd/contents.htm)

## Error Handling and Debugging

Tcl provides robust error handling and debugging capabilities. This section covers comprehensive error handling, debugging techniques, and testing strategies.

### Basic Error Handling

```tcl
# Basic try-catch
try {
    # Code that might fail
    set result [expr {1 / 0}]
} on error {errMsg} {
    puts "Error occurred: $errMsg"
} finally {
    # Cleanup code
    puts "Cleanup executed"
}

# Multiple catch blocks
try {
    # Code that might fail
    set result [expr {1 / 0}]
} on error {errMsg} {
    puts "Error: $errMsg"
} on return {value} {
    puts "Returned: $value"
} on break {} {
    puts "Break encountered"
} on continue {} {
    puts "Continue encountered"
}
```

### Error Information

```tcl
# Get detailed error information
try {
    # Code that might fail
    set result [expr {1 / 0}]
} on error {errMsg} {
    puts "Error message: $errMsg"
    puts "Error code: $::errorCode"
    puts "Error info: $::errorInfo"
    puts "Error stack: [info stack]"
}

# Custom error codes
proc custom_error {msg} {
    return -code error -errorcode {CUSTOM ERROR} $msg
}

# Using custom error
try {
    custom_error "Something went wrong"
} on error {errMsg} {
    puts "Error code: $::errorCode"
}
```

### Debugging Techniques

#### Tracing

```tcl
# Trace variable changes
trace add variable myVar write {apply {args {
    puts "Variable changed: $args"
}}}

# Trace command execution
trace add execution myProc enter {apply {args {
    puts "Entering myProc with args: $args"
}}}

# Trace procedure calls
proc traced_proc {args} {
    puts "Called with: $args"
    # Procedure body
}
trace add execution traced_proc enter {apply {args {
    puts "Entering traced_proc: $args"
}}}
```

#### Debugging Tools

```tcl
# Print variable information
puts "Variable exists: [info exists myVar]"
puts "Variable value: [set myVar]"
puts "Variable type: [info tclversion]"

# Print procedure information
puts "Procedure exists: [info procs myProc]"
puts "Procedure body: [info body myProc]"
puts "Procedure args: [info args myProc]"

# Print stack trace
puts "Stack trace: [info stack]"
puts "Call stack: [info frame]"
```

### Logging

```tcl
# Basic logging procedure
proc log {level message} {
    set timestamp [clock format [clock seconds] -format "%Y-%m-%d %H:%M:%S"]
    puts "$timestamp \[$level\] $message"
}

# Log levels
proc log_debug {message} { log DEBUG $message }
proc log_info {message}  { log INFO $message }
proc log_warn {message}  { log WARN $message }
proc log_error {message} { log ERROR $message }

# File logging
proc log_to_file {filename level message} {
    set fh [open $filename a]
    puts $fh "[clock format [clock seconds]] \[$level\] $message"
    close $fh
}
```

### Testing and Assertions

```tcl
# Basic assertion
proc assert {condition message} {
    if {![expr $condition]} {
        error "Assertion failed: $message"
    }
}

# Test framework
proc test {name body} {
    puts "Running test: $name"
    try {
        uplevel $body
        puts "Test passed: $name"
    } on error {errMsg} {
        puts "Test failed: $name"
        puts "Error: $errMsg"
    }
}

# Using tests
test "basic addition" {
    assert {[expr {1 + 1}] == 2} "1 + 1 should equal 2"
}

# Test suite
proc run_tests {} {
    test "addition" {
        assert {[expr {1 + 1}] == 2} "Basic addition"
    }
    test "subtraction" {
        assert {[expr {2 - 1}] == 1} "Basic subtraction"
    }
}
```

### Error Recovery Strategies

```tcl
# Retry mechanism
proc with_retry {max_attempts body} {
    set attempts 0
    while {$attempts < $max_attempts} {
        try {
            return [uplevel $body]
        } on error {errMsg} {
            incr attempts
            if {$attempts >= $max_attempts} {
                error "Max retries exceeded: $errMsg"
            }
            after 1000;  # Wait before retry
        }
    }
}

# Fallback mechanism
proc with_fallback {primary fallback} {
    try {
        return [uplevel $primary]
    } on error {errMsg} {
        puts "Primary failed: $errMsg"
        return [uplevel $fallback]
    }
}
```

### Best Practices

- Use try-catch for error handling
- Provide meaningful error messages
- Clean up resources in finally blocks
- Use appropriate log levels
- Implement proper error recovery
- Use assertions for debugging
- Write comprehensive tests
- Document error conditions
- Handle all possible error cases
- Use proper error codes

### Common Pitfalls

- Not handling all error cases
- Swallowing errors silently
- Not cleaning up resources
- Using wrong error codes
- Not providing enough context
- Not logging errors properly
- Not testing error conditions
- Not handling timeouts
- Not handling resource exhaustion
- Not handling concurrent errors

### Resources

- [Tcl Documentation on error handling](https://www.tcl.tk/man/tcl/TclCmd/try.htm)
- [Tcl Documentation on debugging](https://www.tcl.tk/man/tcl/TclCmd/trace.htm)
- [Tclers Wiki on error handling](https://wiki.tcl-lang.org/page/error+handling)
- [Tclers Wiki on debugging](https://wiki.tcl-lang.org/page/debugging)

## Performance Optimization

Tcl provides various mechanisms for optimizing performance. This section covers techniques for improving execution speed, memory usage, and overall efficiency.

### Profiling and Measurement

```tcl
# Basic timing
set start [clock microseconds]
# Code to measure
set end [clock microseconds]
set duration [expr {$end - $start}]

# Using time command
set result [time {
    # Code to measure
} 1000]  # Run 1000 times

# Memory usage
proc memory_usage {} {
    set mem [memory info]
    puts "Total: [dict get $mem total]"
    puts "Used: [dict get $mem used]"
    puts "Free: [dict get $mem free]"
}
```

### String Operations

```tcl
# Efficient string concatenation
set result ""
foreach item $list {
    append result $item
}

# vs. inefficient
set result ""
foreach item $list {
    set result "$result$item"
}

# Using string map for multiple replacements
set text "Hello World"
set replacements [list "Hello" "Hi" "World" "Tcl"]
set result [string map $replacements $text]

# Using string repeat for padding
set padded [string repeat " " 10]$text
```

### List Operations

```tcl
# Efficient list building
set result [list]
foreach item $items {
    lappend result $item
}

# vs. inefficient
set result ""
foreach item $items {
    set result "$result $item"
}

# Using lmap for transformations
set doubled [lmap x $list {expr {$x * 2}}]

# Using lsearch with -all for filtering
set matches [lsearch -all -inline $list "pattern"]
```

### Dictionary Operations

```tcl
# Efficient dictionary updates
dict set data key value
dict unset data key
dict update data key {
    set key [expr {$key + 1}]
}

# Batch dictionary operations
set data [dict create {*}$pairs]

# Dictionary filtering
set filtered [dict filter $data value {* > 10}]
```

### Regular Expressions

```tcl
# Compile regexp for repeated use
set pattern [regexp -expanded {
    ^
    [A-Za-z0-9._%+-]+
    @
    [A-Za-z0-9.-]+
    \.
    [A-Za-z]{2,}
    $
}]

# Using -inline for multiple matches
set matches [regexp -all -inline $pattern $text]

# Using -start for incremental matching
set pos 0
while {[regexp -start $pos $pattern $text match]} {
    set pos [expr {$pos + [string length $match]}]
}
```

### File I/O Optimization

```tcl
# Efficient file reading
set fh [open $file r]
fconfigure $fh -buffersize 65536
set data [read $fh]
close $fh

# Line-by-line processing
set fh [open $file r]
fconfigure $fh -buffering line
while {[gets $fh line] >= 0} {
    # Process line
}
close $fh

# Binary file handling
set fh [open $file rb]
fconfigure $fh -translation binary
set data [read $fh]
close $fh
```

### Memory Management

```tcl
# Efficient memory usage
proc process_large_file {filename} {
    set fh [open $filename r]
    fconfigure $fh -buffering line
    while {[gets $fh line] >= 0} {
        # Process one line at a time
    }
    close $fh
}

# Using unset for cleanup
proc cleanup {var} {
    unset -nocomplain $var
}

# Using after idle for garbage collection
after idle {update}
```

### Caching Strategies

```tcl
# Simple caching
proc cached_proc {key args} {
    variable cache
    if {[info exists cache($key)]} {
        return $cache($key)
    }
    set result [eval $args]
    set cache($key) $result
    return $result
}

# Time-based caching
proc timed_cache {key ttl args} {
    variable cache
    variable cache_time
    set now [clock seconds]
    if {[info exists cache($key)] && 
        [expr {$now - $cache_time($key)}] < $ttl} {
        return $cache($key)
    }
    set result [eval $args]
    set cache($key) $result
    set cache_time($key) $now
    return $result
}
```

### Best Practices

- Use appropriate data structures
- Minimize string concatenation
- Use compiled regular expressions
- Implement proper caching
- Use efficient file I/O
- Clean up resources promptly
- Use appropriate buffering
- Profile before optimizing
- Use built-in commands when possible
- Implement proper error handling

### Common Pitfalls

- Excessive string concatenation
- Inefficient list operations
- Unnecessary variable creation
- Memory leaks
- Inefficient file I/O
- Over-caching
- Premature optimization
- Not using built-in commands
- Inefficient regular expressions
- Not cleaning up resources

### Resources

- [Tcl Documentation on performance](https://www.tcl.tk/man/tcl/TclCmd/time.htm)
- [Tclers Wiki on performance](https://wiki.tcl-lang.org/page/performance)
- [Tcl Performance Tips](https://wiki.tcl-lang.org/page/Performance+Tips)

## Security Best Practices

Security is crucial in Tcl applications. This section covers essential security practices, input validation, and protection against common vulnerabilities.

### Input Validation

```tcl
# Validate numeric input
proc validate_number {input} {
    if {![string is integer -strict $input]} {
        error "Invalid number: $input"
    }
    return $input
}

# Validate string input
proc validate_string {input max_length} {
    if {[string length $input] > $max_length} {
        error "String too long"
    }
    # Remove potentially dangerous characters
    regsub -all {[^a-zA-Z0-9_-]} $input "" cleaned
    return $cleaned
}

# Validate file path
proc validate_path {path} {
    set normalized [file normalize $path]
    if {![string match "/safe/dir/*" $normalized]} {
        error "Access denied to path: $path"
    }
    return $normalized
}
```

### Command Injection Prevention

```tcl
# Safe command execution
proc safe_exec {cmd args} {
    # Validate command
    if {![string match {[a-zA-Z]*} $cmd]} {
        error "Invalid command"
    }
    # Use list for arguments
    eval [list $cmd] $args
}

# Safe file operations
proc safe_open {filename mode} {
    set normalized [file normalize $filename]
    if {![string match "/safe/dir/*" $normalized]} {
        error "Access denied"
    }
    return [open $normalized $mode]
}

# Safe eval
proc safe_eval {script} {
    # Validate script content
    if {[regexp {[^a-zA-Z0-9_]} $script]} {
        error "Invalid characters in script"
    }
    return [eval $script]
}
```

### Secure File Operations

```tcl
# Safe file reading
proc safe_read_file {filename} {
    set normalized [file normalize $filename]
    if {![string match "/safe/dir/*" $normalized]} {
        error "Access denied"
    }
    set fh [open $normalized r]
    fconfigure $fh -translation binary
    set data [read $fh]
    close $fh
    return $data
}

# Safe file writing
proc safe_write_file {filename data} {
    set normalized [file normalize $filename]
    if {![string match "/safe/dir/*" $normalized]} {
        error "Access denied"
    }
    set fh [open $normalized w]
    fconfigure $fh -translation binary
    puts -nonewline $fh $data
    close $fh
}
```

### Network Security

```tcl
# Secure socket connection
proc secure_connect {host port} {
    set sock [socket -async $host $port]
    fconfigure $sock -buffering line -encoding utf-8
    
    # Set timeout
    fconfigure $sock -timeout 5000
    
    # Enable TLS if available
    if {[catch {package require tls}] == 0} {
        tls::import $sock
    }
    
    return $sock
}

# Secure HTTP request
proc secure_http_get {url} {
    set token [http::geturl $url -timeout 5000]
    set status [http::status $token]
    if {$status ne "ok"} {
        http::cleanup $token
        error "HTTP request failed: $status"
    }
    set data [http::data $token]
    http::cleanup $token
    return $data
}
```

### Password Handling

```tcl
# Secure password hashing
proc hash_password {password} {
    if {[catch {package require sha256}] == 0} {
        return [sha256::sha256 $password]
    }
    error "SHA256 package not available"
}

# Password verification
proc verify_password {password hash} {
    set hashed [hash_password $password]
    return [string equal $hashed $hash]
}

# Secure password storage
proc store_password {username password} {
    set hashed [hash_password $password]
    set salt [binary format H* [string repeat "0" 32]]
    set stored "$salt$hashed"
    # Store in secure location
    return $stored
}
```

### Secure Configuration

```tcl
# Secure configuration loading
proc load_config {filename} {
    set normalized [file normalize $filename]
    if {![string match "/safe/dir/*" $normalized]} {
        error "Access denied"
    }
    
    # Set safe interpreter
    set interp [interp create -safe]
    $interp eval [read [open $normalized r]]
    return [$interp eval {array get config}]
}

# Secure environment variables
proc get_env_var {name} {
    if {![regexp {^[A-Z_]+$} $name]} {
        error "Invalid environment variable name"
    }
    return $::env($name)
}
```

### Best Practices

- Always validate input
- Use safe interpreters when possible
- Implement proper access control
- Use secure file operations
- Handle sensitive data carefully
- Implement proper error handling
- Use secure network protocols
- Follow principle of least privilege
- Implement proper logging
- Regular security audits

### Common Pitfalls

- Command injection vulnerabilities
- Path traversal attacks
- Insecure file operations
- Unsafe eval usage
- Insecure network operations
- Plain text password storage
- Insufficient input validation
- Insecure configuration handling
- Missing access controls
- Insecure error handling

### Resources

- [Tcl Documentation on safe interps](https://www.tcl.tk/man/tcl/TclCmd/interp.htm)
- [Tclers Wiki on security](https://wiki.tcl-lang.org/page/security)
- [OWASP Tcl Security](https://owasp.org/www-community/attacks/Command_Injection)

## Internationalization and Localization

Tcl provides robust support for internationalization (i18n) and localization (l10n). This section covers message translation, character encoding, and locale handling.

### Basic Message Translation

```tcl
# Using msgcat package
package require msgcat

# Define messages
::msgcat::mcset en "greeting" "Hello"
::msgcat::mcset fr "greeting" "Bonjour"
::msgcat::mcset de "greeting" "Hallo"

# Set locale
::msgcat::mclocale "fr"

# Get translated message
puts [::msgcat::mc "greeting"];  # => Bonjour

# With parameters
::msgcat::mcset en "welcome" "Welcome, %s!"
::msgcat::mcset fr "welcome" "Bienvenue, %s!"
puts [::msgcat::mc "welcome" "John"];  # => Bienvenue, John!
```

### Message Catalog Management

```tcl
# Load message catalog from file
proc load_catalog {locale} {
    set catalog_file "msgs/$locale.msg"
    if {[file exists $catalog_file]} {
        ::msgcat::mcload [file dirname $catalog_file]
    }
}

# Define message catalog structure
# msgs/en.msg
::msgcat::mcset en "app.title" "My Application"
::msgcat::mcset en "app.menu.file" "File"
::msgcat::mcset en "app.menu.edit" "Edit"

# msgs/fr.msg
::msgcat::mcset fr "app.title" "Mon Application"
::msgcat::mcset fr "app.menu.file" "Fichier"
::msgcat::mcset fr "app.menu.edit" "Éditer"
```

### Character Encoding

```tcl
# Set system encoding
encoding system utf-8

# Convert between encodings
proc convert_encoding {text from to} {
    return [encoding convertfrom $to [encoding convertto $from $text]]
}

# Handle BOM (Byte Order Mark)
proc read_with_bom {filename} {
    set fh [open $filename r]
    fconfigure $fh -encoding unicode
    set content [read $fh]
    close $fh
    return $content
}

# Write with specific encoding
proc write_with_encoding {filename text encoding} {
    set fh [open $filename w]
    fconfigure $fh -encoding $encoding
    puts -nonewline $fh $text
    close $fh
}
```

### Locale Handling

```tcl
# Set locale
proc set_locale {locale} {
    ::msgcat::mclocale $locale
    load_catalog $locale
}

# Get current locale
proc get_locale {} {
    return [::msgcat::mclocale]
}

# Format numbers according to locale
proc format_number {number locale} {
    set old_locale [::msgcat::mclocale]
    ::msgcat::mclocale $locale
    set formatted [::msgcat::mc "number.format" $number]
    ::msgcat::mclocale $old_locale
    return $formatted
}

# Format dates according to locale
proc format_date {timestamp locale} {
    set old_locale [::msgcat::mclocale]
    ::msgcat::mclocale $locale
    set formatted [::msgcat::mc "date.format" [clock format $timestamp]]
    ::msgcat::mclocale $old_locale
    return $formatted
}
```

### Advanced Translation Features

```tcl
# Plural forms
::msgcat::mcset en "items" {[llength $n] == 1 ? "1 item" : "%d items"}
::msgcat::mcset fr "items" {[llength $n] == 1 ? "1 élément" : "%d éléments"}

# Context-based translation
::msgcat::mcset en "save" "Save"
::msgcat::mcset en "save.file" "Save File"
::msgcat::mcset en "save.document" "Save Document"

# Nested translations
proc translate_nested {key args} {
    set result [::msgcat::mc $key]
    foreach {k v} $args {
        set result [string map [list "%$k%" $v] $result]
    }
    return $result
}
```

### Text Direction and Layout

```tcl
# Handle right-to-left text
proc is_rtl {text} {
    # Check if text contains RTL characters
    return [regexp {[\u0591-\u07FF\u200F\u202B\u202E\uFB1D-\uFDFD\uFE70-\uFEFC]} $text]
}

# Adjust layout based on text direction
proc adjust_layout {text} {
    if {[is_rtl $text]} {
        return "right"
    }
    return "left"
}

# Handle mixed direction text
proc handle_mixed_direction {text} {
    # Insert direction markers if needed
    if {[is_rtl $text]} {
        return "\u202B$text\u202C"
    }
    return $text
}
```

### Best Practices

- Use msgcat for all user-visible strings
- Support UTF-8 encoding
- Handle text direction properly
- Use proper number and date formatting
- Implement proper plural forms
- Use context-based translations
- Handle mixed direction text
- Support locale-specific formats
- Use proper character encoding
- Implement fallback translations

### Common Pitfalls

- Hardcoded strings
- Incorrect character encoding
- Missing translations
- Improper number formatting
- Incorrect date formatting
- Missing plural forms
- Text direction issues
- Mixed direction text problems
- Locale-specific format issues
- Missing fallback translations

### Resources

- [Tcl Documentation on msgcat](https://www.tcl.tk/man/tcl/TclCmd/msgcat.htm)
- [Tclers Wiki on internationalization](https://wiki.tcl-lang.org/page/internationalization)
- [Unicode in Tcl](https://wiki.tcl-lang.org/page/Unicode)

## Testing Frameworks

Tcl provides several testing frameworks and tools for writing and running tests. This section covers the most commonly used testing approaches and best practices.

### tcltest Framework

```tcl
# Basic test setup
package require tcltest
namespace import tcltest::*

# Simple test
test simple-1.1 {basic addition} {
    expr {1 + 1}
} 2

# Test with setup and cleanup
test math-1.1 {complex calculation} -setup {
    set x 5
    set y 3
} -body {
    expr {$x * $y + 2}
} -cleanup {
    unset x y
} -result 17

# Test with constraints
test constraints-1.1 {platform specific} -constraints {unix} {
    file exists /dev/null
} 1

# Test with multiple constraints
test constraints-1.2 {multiple constraints} -constraints {unix tcl8.6} {
    # Test code
} result
```

### Test Organization

```tcl
# Test file structure
package require tcltest
namespace import tcltest::*

# Test suite setup
proc setup {} {
    # Global setup code
}

# Test suite cleanup
proc cleanup {} {
    # Global cleanup code
}

# Test cases
test case-1.1 {test description} {
    # Test code
} result

# Run all tests
tcltest::runAllTests
```

### Advanced Testing Features

```tcl
# Custom test constraints
tcltest::customMatch float {expr abs($expected - $actual) < 0.0001}

# Test with custom matching
test float-1.1 {floating point comparison} {
    expr {3.14159}
} -match float 3.14159265359

# Test with error conditions
test error-1.1 {error handling} -body {
    error "test error"
} -returnCodes error -result "test error"

# Test with cleanup
test cleanup-1.1 {resource cleanup} -setup {
    set fh [open test.txt w]
} -body {
    puts $fh "test"
} -cleanup {
    close $fh
    file delete test.txt
}
```

### Test Fixtures

```tcl
# Test fixture setup
proc setup_fixture {} {
    # Create test environment
    file mkdir testdir
    set fh [open testdir/test.txt w]
    puts $fh "test data"
    close $fh
}

# Test fixture cleanup
proc cleanup_fixture {} {
    # Clean up test environment
    file delete -force testdir
}

# Using fixtures
test fixture-1.1 {file operations} -setup setup_fixture -cleanup cleanup_fixture {
    set fh [open testdir/test.txt r]
    set data [read $fh]
    close $fh
    return $data
} "test data"
```

### Test Reporting

```tcl
# Custom test output
proc customOutput {numTests numPassed numSkipped numFailed} {
    puts "Test Summary:"
    puts "Total: $numTests"
    puts "Passed: $numPassed"
    puts "Skipped: $numSkipped"
    puts "Failed: $numFailed"
}

# Configure test output
tcltest::configure -verbose bps
tcltest::configure -outputFile test.log
tcltest::configure -outputChannel stdout
tcltest::configure -testdir [file dirname [info script]]
```

### Mocking and Stubbing

```tcl
# Simple mock
proc mock_proc {name args body} {
    set original [info procs $name]
    if {$original ne ""} {
        rename $name ${name}_original
    }
    proc $name $args $body
    return $original
}

# Restore original
proc restore_proc {name original} {
    if {$original ne ""} {
        rename $name {}
        rename ${name}_original $name
    }
}

# Using mocks
test mock-1.1 {mocked function} {
    set original [mock_proc math::add {a b} {return 42}]
    set result [math::add 1 2]
    restore_proc math::add $original
    return $result
} 42
```

### Best Practices

- Write independent tests
- Use meaningful test names
- Include setup and cleanup
- Use appropriate constraints
- Handle error conditions
- Clean up resources
- Use test fixtures
- Implement proper assertions
- Document test cases
- Maintain test organization

### Common Pitfalls

- Missing cleanup code
- Dependent tests
- Unclear test names
- Missing error handling
- Resource leaks
- Incomplete test coverage
- Unnecessary test dependencies
- Missing test constraints
- Improper test isolation
- Inadequate test documentation

### Resources

- [Tcl Documentation on tcltest](https://www.tcl.tk/man/tcl/TclCmd/tcltest.htm)
- [Tclers Wiki on testing](https://wiki.tcl-lang.org/page/testing)
- [Tcl Test Suite](https://core.tcl-lang.org/tcl/file?name=unix/tclUnixTest.c)

## Debugging Tools and Techniques

Tcl provides various debugging tools and techniques to help diagnose and fix issues in your code. This section covers both built-in and external debugging capabilities.

### Built-in Debugging Commands

```tcl
# Print variable information
puts "Variable exists: [info exists myVar]"
puts "Variable value: [set myVar]"
puts "Variable type: [info tclversion]"

# Print procedure information
puts "Procedure exists: [info procs myProc]"
puts "Procedure body: [info body myProc]"
puts "Procedure args: [info args myProc]"

# Print stack trace
puts "Stack trace: [info stack]"
puts "Call stack: [info frame]"

# Print namespace information
puts "Namespace children: [namespace children ::]"
puts "Namespace variables: [info vars ::mynamespace::*]"
```

### Tracing and Monitoring

```tcl
# Trace variable changes
trace add variable myVar write {apply {args {
    puts "Variable changed: $args"
}}}

# Trace command execution
trace add execution myProc enter {apply {args {
    puts "Entering myProc with args: $args"
}}}

# Trace procedure calls
proc traced_proc {args} {
    puts "Called with: $args"
    # Procedure body
}
trace add execution traced_proc enter {apply {args {
    puts "Entering traced_proc: $args"
}}}
```

### Interactive Debugging

```tcl
# Set breakpoint
proc debug_break {} {
    puts "Breakpoint hit. Type 'continue' to proceed."
    while 1 {
        gets stdin cmd
        if {$cmd eq "continue"} break
        if {[catch {eval $cmd} result]} {
            puts "Error: $result"
        } else {
            puts $result
        }
    }
}

# Debug console
proc debug_console {} {
    puts "Debug console started. Type 'exit' to quit."
    while 1 {
        puts -nonewline "debug> "
        gets stdin cmd
        if {$cmd eq "exit"} break
        if {[catch {eval $cmd} result]} {
            puts "Error: $result"
        } else {
            puts $result
        }
    }
}
```

### Memory Debugging

```tcl
# Memory usage tracking
proc memory_usage {} {
    set mem [memory info]
    puts "Total: [dict get $mem total]"
    puts "Used: [dict get $mem used]"
    puts "Free: [dict get $mem free]"
}

# Track variable creation
proc track_vars {} {
    set vars [info vars]
    puts "Current variables: $vars"
    after 1000 [list track_vars]
}

# Monitor procedure calls
proc monitor_calls {proc_name} {
    set count 0
    trace add execution $proc_name enter {apply {args {
        global count
        incr count
        puts "Call #$count to $proc_name"
    }}}
}
```

### Logging and Diagnostics

```tcl
# Logging levels
proc log {level message} {
    set timestamp [clock format [clock seconds] -format "%Y-%m-%d %H:%M:%S"]
    puts "$timestamp \[$level\] $message"
}

# Log levels
proc log_debug {message} { log DEBUG $message }
proc log_info {message}  { log INFO $message }
proc log_warn {message}  { log WARN $message }
proc log_error {message} { log ERROR $message }

# File logging
proc log_to_file {filename level message} {
    set fh [open $filename a]
    puts $fh "[clock format [clock seconds]] \[$level\] $message"
    close $fh
}
```

### Performance Profiling

```tcl
# Basic profiling
proc profile {script} {
    set start [clock microseconds]
    set result [eval $script]
    set end [clock microseconds]
    set duration [expr {$end - $start}]
    puts "Execution time: $duration microseconds"
    return $result
}

# Command profiling
proc profile_command {cmd args} {
    set start [clock microseconds]
    set result [eval $cmd $args]
    set end [clock microseconds]
    set duration [expr {$end - $start}]
    puts "$cmd took $duration microseconds"
    return $result
}

# Memory profiling
proc profile_memory {script} {
    set before [memory info]
    set result [eval $script]
    set after [memory info]
    puts "Memory before: [dict get $before used]"
    puts "Memory after: [dict get $after used]"
    return $result
}
```

### Debugging Tools Integration

```tcl
# Integration with external debugger
proc debugger_attach {port} {
    package require debug
    debug::attach $port
}

# Remote debugging
proc remote_debug {host port} {
    package require debug
    debug::connect $host $port
}

# Debug session management
proc debug_session {script} {
    set session [debug::session create]
    debug::session eval $session $script
    debug::session destroy $session
}
```

### Best Practices

- Use appropriate logging levels
- Implement proper error handling
- Use tracing for complex issues
- Monitor memory usage
- Profile performance bottlenecks
- Use interactive debugging when needed
- Implement proper cleanup
- Document debugging procedures
- Use version control for debugging
- Maintain debugging logs

### Common Pitfalls

- Insufficient error handling
- Missing cleanup code
- Memory leaks
- Performance bottlenecks
- Inadequate logging
- Missing stack traces
- Improper variable tracking
- Incomplete debugging information
- Resource leaks
- Debug code in production

### Resources

- [Tcl Documentation on debugging](https://www.tcl.tk/man/tcl/TclCmd/trace.htm)
- [Tclers Wiki on debugging](https://wiki.tcl-lang.org/page/debugging)
- [Tcl Debugging Tools](https://wiki.tcl-lang.org/page/Debugging+Tools)

## Code Style and Documentation

Good code style and documentation are essential for maintainable Tcl code. This section covers best practices, coding standards, and documentation techniques.

### Code Style Guidelines

```tcl
# Variable naming
set userName "John";        # Use camelCase for variables
set MAX_RETRIES 3;         # Use UPPER_CASE for constants
set _privateVar "hidden";  # Use _prefix for private variables

# Procedure naming
proc calculateTotal {items} {  # Use camelCase for procedures
    # Procedure body
}

# Namespace naming
namespace eval ::myapp::utils {  # Use ::prefix for namespaces
    # Namespace contents
}

# Indentation and spacing
if {$condition} {
    # Use 4 spaces for indentation
    set result [expr {
        $value1 + $value2
    }]
}

# Line length and wrapping
set longString "This is a very long string that needs to be \
    wrapped for better readability"
```

### Documentation Standards

```tcl
# Procedure documentation
proc calculateTotal {items} {
    # Calculate the total value of items
    #
    # @param items List of items with price values
    # @return Total value of all items
    # @throws Error if items list is empty
    
    if {[llength $items] == 0} {
        error "Items list cannot be empty"
    }
    
    set total 0
    foreach item $items {
        incr total [dict get $item price]
    }
    return $total
}

# Class documentation
oo::class create Calculator {
    # Calculator class for mathematical operations
    #
    # This class provides basic mathematical operations
    # and maintains calculation history.
    
    variable history
    
    constructor {} {
        # Initialize calculator with empty history
        set history [list]
    }
    
    method add {a b} {
        # Add two numbers
        #
        # @param a First number
        # @param b Second number
        # @return Sum of a and b
        
        set result [expr {$a + $b}]
        lappend history [list add $a $b $result]
        return $result
    }
}
```

### Code Organization

```tcl
# File structure
# mypackage.tcl

# 1. Package declaration
package provide mypackage 1.0

# 2. Required packages
package require Tcl 8.6
package require some::other::package 1.0

# 3. Namespace declaration
namespace eval ::mypackage {
    # 4. Variable declarations
    variable config [dict create]
    variable state [dict create]
    
    # 5. Procedure declarations
    proc init {} {
        # Initialization code
    }
    
    proc cleanup {} {
        # Cleanup code
    }
    
    # 6. Class definitions
    oo::class create MyClass {
        # Class implementation
    }
}

# 7. Package initialization
::mypackage::init
```

### Commenting Guidelines

```tcl
# File header
# mypackage.tcl
# Author: John Doe
# Date: 2024-03-20
# Description: This module provides functionality for...

# Section headers
# ==============
# Configuration
# ==============

# Inline comments
set result [expr {$value * 2}];  # Double the value

# Block comments
# This is a block comment that explains
# a complex algorithm or implementation
# details that span multiple lines

# TODO comments
# TODO: Implement error handling for edge cases
# FIXME: This needs to be optimized for large datasets
```

### Code Examples

```tcl
# Good example: Clear and well-documented
proc calculateDiscount {price discount} {
    # Calculate final price after discount
    #
    # @param price Original price
    # @param discount Discount percentage (0-100)
    # @return Final price after discount
    
    if {$discount < 0 || $discount > 100} {
        error "Discount must be between 0 and 100"
    }
    
    return [expr {$price * (100 - $discount) / 100.0}]
}

# Bad example: Unclear and poorly documented
proc calc {p d} {
    return [expr {$p * (100 - $d) / 100.0}]
}
```

### Best Practices

- Use consistent naming conventions
- Follow proper indentation rules
- Write clear and concise comments
- Document all procedures and classes
- Use meaningful variable names
- Keep procedures focused and small
- Use proper error handling
- Follow DRY (Don't Repeat Yourself)
- Use proper code organization
- Maintain consistent style

### Common Pitfalls

- Inconsistent naming
- Poor indentation
- Missing documentation
- Unclear variable names
- Overly complex procedures
- Insufficient comments
- Inconsistent style
- Poor code organization
- Missing error handling
- Code duplication

### Resources

- [Tcl Style Guide](https://wiki.tcl-lang.org/page/Tcl+Style+Guide)
- [Tcl Documentation Guidelines](https://wiki.tcl-lang.org/page/Documentation+Guidelines)
- [Tcl Best Practices](https://wiki.tcl-lang.org/page/Best+Practices)

## Build Systems and Deployment

Tcl applications can be built and deployed using various tools and techniques. This section covers build systems, packaging, and deployment strategies.

### TEA (Tcl Extension Architecture)

```tcl
# configure.ac
AC_INIT([myextension], [1.0])
AC_CONFIG_SRCDIR([generic/myextension.c])
AC_CONFIG_HEADER([config.h])

# Check for Tcl
TEA_INIT([3.9])
AC_PROG_CC
AC_PROG_INSTALL

# Configure extension
TEA_ADD_SOURCES([generic/myextension.c])
TEA_ADD_HEADERS([generic/myextension.h])
TEA_ADD_LIBS([-lm])
TEA_ADD_TCL_SOURCES([tcl/myextension.tcl])

AC_CONFIG_FILES([Makefile])
AC_OUTPUT

# Makefile.in
TCL_DEFS = @TCL_DEFS@
INCLUDES = @TCL_INCLUDE_SPEC@
LIBS = @TCL_LIB_SPEC@ @LIBS@

SRCS = @TEA_ADD_SOURCES@
OBJS = $(SRCS:.c=.o)

all: myextension.so

myextension.so: $(OBJS)
    $(CC) -shared -o $@ $(OBJS) $(LIBS)

install: myextension.so
    $(INSTALL) $< $(libdir)
```

### Build Tools

#### tclapp (Tcl Dev Kit)

```tcl
# Build script for tclapp
package require tclapp

# Create application
tclapp::create myapp {
    # Add source files
    tclapp::add file main.tcl
    tclapp::add file utils.tcl
    
    # Add packages
    tclapp::add package Tcl 8.6
    tclapp::add package tcllib
    
    # Set options
    tclapp::set option -name "My Application"
    tclapp::set option -version 1.0
    tclapp::set option -company "My Company"
    
    # Build
    tclapp::build
}
```

#### tclkit

```tcl
# Create tclkit executable
package require tclkit

# Build standalone executable
tclkit::wrap myapp.kit {
    # Add files
    tclkit::add file main.tcl
    tclkit::add file utils.tcl
    
    # Add packages
    tclkit::add package Tcl 8.6
    tclkit::add package tcllib
    
    # Set metadata
    tclkit::set name "My Application"
    tclkit::set version 1.0
}
```

### Deployment Strategies

#### Standalone Applications

```tcl
# Create deployment directory structure
proc create_deployment {appname version} {
    set deploy_dir [file join deploy $appname-$version]
    file mkdir $deploy_dir
    
    # Copy executable
    file copy myapp.kit [file join $deploy_dir $appname]
    
    # Copy resources
    file mkdir [file join $deploy_dir resources]
    file copy resources/* [file join $deploy_dir resources]
    
    # Create launcher script
    set launcher [open [file join $deploy_dir run.sh] w]
    puts $launcher "#!/bin/sh"
    puts $launcher "cd \[dirname \$0\]"
    puts $launcher "./$appname"
    close $launcher
    file attributes [file join $deploy_dir run.sh] -permissions 0755
}
```

#### Package Deployment

```tcl
# Create package for distribution
proc create_package {pkgname version} {
    set pkg_dir [file join packages $pkgname-$version]
    file mkdir $pkg_dir
    
    # Create pkgIndex.tcl
    set pkgindex [open [file join $pkg_dir pkgIndex.tcl] w]
    puts $pkgindex "package ifneeded $pkgname $version \[list source \[file join \$dir $pkgname.tcl\]\]"
    close $pkgindex
    
    # Copy package files
    file copy $pkgname.tcl [file join $pkg_dir $pkgname.tcl]
    
    # Create README
    set readme [open [file join $pkg_dir README] w]
    puts $readme "Package: $pkgname"
    puts $readme "Version: $version"
    puts $readme "Description: ..."
    close $readme
}
```

### Installation Scripts

```tcl
# Unix installation script
proc install_unix {prefix} {
    # Create directories
    file mkdir [file join $prefix bin]
    file mkdir [file join $prefix lib]
    
    # Install executable
    file copy myapp [file join $prefix bin]
    
    # Install libraries
    file copy lib/* [file join $prefix lib]
    
    # Set permissions
    file attributes [file join $prefix bin myapp] -permissions 0755
}

# Windows installation script
proc install_windows {installdir} {
    # Create directories
    file mkdir [file join $installdir bin]
    file mkdir [file join $installdir lib]
    
    # Install executable
    file copy myapp.exe [file join $installdir bin]
    
    # Install libraries
    file copy lib/* [file join $installdir lib]
    
    # Create start menu shortcut
    set shortcut [open [file join $installdir "MyApp.lnk"] w]
    puts $shortcut "... shortcut content ..."
    close $shortcut
}
```

### Version Control Integration

```tcl
# Git integration
proc git_version {} {
    set git_cmd "git describe --tags --always"
    if {[catch {exec $git_cmd} version]} {
        return "unknown"
    }
    return $version
}

# Update version information
proc update_version {filename} {
    set content [read_file $filename]
    set version [git_version]
    regsub {VERSION\s+[0-9.]+} $content "VERSION $version" new_content
    write_file $filename $new_content
}
```

### Best Practices

- Use TEA for C extensions
- Implement proper versioning
- Create reproducible builds
- Include installation scripts
- Document build requirements
- Test deployment process
- Handle platform differences
- Include license information
- Provide update mechanism
- Maintain build documentation

### Common Pitfalls

- Missing dependencies
- Platform-specific issues
- Incorrect permissions
- Missing documentation
- Incomplete installation
- Version conflicts
- Resource path issues
- Missing cleanup
- Incomplete testing
- Poor error handling

### Resources

- [Tcl Extension Architecture](https://www.tcl.tk/doc/tea/)
- [Tcl Dev Kit Documentation](https://www.tcl.tk/software/tclpro/)
- [Tclkit Documentation](https://www.tcl.tk/software/tclkit/)

## Database Integration

Tcl provides various ways to interact with databases. This section covers database connectivity, query execution, and data management using different database systems.

### SQLite Integration

```tcl
# Basic SQLite usage
package require sqlite3

# Open database
sqlite3 db "mydatabase.db"

# Create table
db eval {
    CREATE TABLE users (
        id INTEGER PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT UNIQUE,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
}

# Insert data
db eval {
    INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com')
}

# Query data
db eval {SELECT * FROM users} {
    puts "User: $name, Email: $email"
}

# Parameterized queries
set name "John Doe"
db eval {SELECT * FROM users WHERE name = $name} {
    puts "Found user: $name"
}

# Transaction handling
db transaction {
    db eval {INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')}
    db eval {INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com')}
}
```

### MySQL/MariaDB Integration

```tcl
# Using mysqltcl
package require mysqltcl

# Connect to database
set handle [mysqlconnect -host localhost -user root -password secret -db mydb]

# Execute query
set result [mysqlexec $handle "SELECT * FROM users"]
while {[mysqlnext $result]} {
    puts "User: [mysqlresult $result name], Email: [mysqlresult $result email]"
}

# Parameterized queries
set name "John Doe"
set query "SELECT * FROM users WHERE name = ?"
set result [mysqlexec $handle $query $name]

# Transaction handling
mysqlexec $handle "START TRANSACTION"
if {[catch {
    mysqlexec $handle "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')"
    mysqlexec $handle "INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com')"
    mysqlexec $handle "COMMIT"
} err]} {
    mysqlexec $handle "ROLLBACK"
    error $err
}
```

### PostgreSQL Integration

```tcl
# Using pgtcl
package require pgtcl

# Connect to database
set conn [pg_connect -conninfo "host=localhost dbname=mydb user=postgres password=secret"]

# Execute query
set result [pg_exec $conn "SELECT * FROM users"]
set ntuples [pg_result $result -numTuples]
for {set i 0} {$i < $ntuples} {incr i} {
    puts "User: [pg_result $result -getTuple $i name], Email: [pg_result $result -getTuple $i email]"
}

# Parameterized queries
set name "John Doe"
set result [pg_exec $conn "SELECT * FROM users WHERE name = \$1" $name]

# Transaction handling
pg_exec $conn "BEGIN"
if {[catch {
    pg_exec $conn "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')"
    pg_exec $conn "INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com')"
    pg_exec $conn "COMMIT"
} err]} {
    pg_exec $conn "ROLLBACK"
    error $err
}
```

### Database Connection Management

```tcl
# Connection pool
package require pool

# Create connection pool
set pool [pool::create -max 10 -idle 5]

# Get connection from pool
set conn [pool::get $pool]

# Use connection
db eval {SELECT * FROM users} {
    puts "User: $name"
}

# Return connection to pool
pool::put $pool $conn

# Connection wrapper
proc with_db {pool script} {
    set conn [pool::get $pool]
    try {
        uplevel $script
    } finally {
        pool::put $pool $conn
    }
}

# Usage
with_db $pool {
    db eval {SELECT * FROM users} {
        puts "User: $name"
    }
}
```

### Query Building

```tcl
# SQL query builder
proc build_query {table args} {
    set query "SELECT * FROM $table"
    set where [list]
    foreach {field value} $args {
        lappend where "$field = :$field"
    }
    if {[llength $where] > 0} {
        append query " WHERE [join $where { AND }]"
    }
    return $query
}

# Usage
set query [build_query users name "John Doe" email "john@example.com"]
db eval $query {
    puts "Found user: $name"
}
```

### Data Validation

```tcl
# Input validation
proc validate_user {name email} {
    if {![string is ascii $name]} {
        error "Name contains invalid characters"
    }
    if {![regexp {^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$} $email]} {
        error "Invalid email format"
    }
}

# Safe insertion
proc insert_user {db name email} {
    validate_user $name $email
    db eval {
        INSERT INTO users (name, email)
        VALUES ($name, $email)
    }
}
```

### Error Handling

```tcl
# Database error handling
proc safe_db_exec {db query args} {
    if {[catch {
        db eval $query {*}$args
    } err]} {
        puts "Database error: $err"
        puts "Error code: [db errorcode]"
        puts "Error message: [db errormsg]"
        error $err
    }
}

# Usage
safe_db_exec $db {
    INSERT INTO users (name, email)
    VALUES ($name, $email)
} name "John Doe" email "john@example.com"
```

### Best Practices

- Use parameterized queries
- Implement connection pooling
- Handle transactions properly
- Validate input data
- Use proper error handling
- Close connections properly
- Use appropriate indexes
- Implement proper logging
- Use prepared statements
- Follow security guidelines

### Common Pitfalls

- SQL injection vulnerabilities
- Connection leaks
- Missing error handling
- Improper transaction management
- Inefficient queries
- Missing input validation
- Resource exhaustion
- Deadlocks
- Data type mismatches
- Missing indexes

### Resources

- [SQLite Documentation](https://www.sqlite.org/docs.html)
- [MySQL Tcl Documentation](https://www.xdobry.de/mysqltcl/)
- [PostgreSQL Tcl Documentation](https://www.postgresql.org/docs/current/libpq.html)

## Network Programming

Tcl provides robust networking capabilities for both client and server applications. This section covers socket programming, protocols, and network utilities.

### Socket Programming

#### TCP Server

```tcl
# Basic TCP server
proc start_server {port} {
    socket -server accept $port
    puts "Server listening on port $port"
    vwait forever
}

proc accept {sock addr port} {
    puts "Connection from $addr:$port"
    fconfigure $sock -buffering line -encoding utf-8
    fileevent $sock readable [list handle_client $sock]
}

proc handle_client {sock} {
    if {[eof $sock] || [catch {gets $sock line}]} {
        close $sock
        return
    }
    puts "Received: $line"
    puts $sock "Echo: $line"
    flush $sock
}

# Start server
start_server 8080
```

#### TCP Client

```tcl
# Basic TCP client
proc connect_to_server {host port} {
    set sock [socket $host $port]
    fconfigure $sock -buffering line -encoding utf-8
    return $sock
}

proc send_message {sock message} {
    puts $sock $message
    flush $sock
    set response [gets $sock]
    return $response
}

# Usage
set sock [connect_to_server localhost 8080]
set response [send_message $sock "Hello, server!"]
puts "Server response: $response"
close $sock
```

#### UDP Communication

```tcl
# UDP server
proc start_udp_server {port} {
    set sock [socket -server accept_udp $port]
    fconfigure $sock -buffering none
    vwait forever
}

proc accept_udp {sock} {
    fconfigure $sock -buffering none
    fileevent $sock readable [list handle_udp $sock]
}

proc handle_udp {sock} {
    set data [read $sock]
    set peer [fconfigure $sock -peer]
    puts "Received from $peer: $data"
    puts -nonewline $sock "Echo: $data"
}

# UDP client
proc send_udp {host port message} {
    set sock [socket -datagram $host $port]
    fconfigure $sock -buffering none
    puts -nonewline $sock $message
    set response [read $sock]
    close $sock
    return $response
}
```

### Protocol Implementation

#### HTTP Client

```tcl
# Basic HTTP client
package require http

proc http_get {url} {
    set token [http::geturl $url]
    set data [http::data $token]
    set status [http::status $token]
    http::cleanup $token
    return [list $status $data]
}

proc http_post {url data} {
    set headers [list Content-Type "application/json"]
    set token [http::geturl $url -method POST -headers $headers -query $data]
    set response [http::data $token]
    set status [http::status $token]
    http::cleanup $token
    return [list $status $response]
}

# Usage
set result [http_get "http://example.com"]
lassign $result status data
puts "Status: $status"
puts "Data: $data"
```

#### Custom Protocol

```tcl
# Custom protocol implementation
proc send_protocol_message {sock type data} {
    set message [list $type $data]
    puts $sock $message
    flush $sock
    return [gets $sock]
}

proc handle_protocol_message {sock} {
    if {[catch {gets $sock message}]} {
        close $sock
        return
    }
    lassign $message type data
    switch $type {
        "ECHO" { puts $sock "ECHO: $data" }
        "TIME" { puts $sock "TIME: [clock seconds]" }
        default { puts $sock "ERROR: Unknown message type" }
    }
    flush $sock
}
```

### Network Utilities

#### Port Scanner

```tcl
# Simple port scanner
proc scan_port {host port} {
    if {[catch {
        set sock [socket -async $host $port]
        fconfigure $sock -blocking 1
        close $sock
        return 1
    }]} {
        return 0
    }
}

proc scan_range {host start_port end_port} {
    set open_ports [list]
    for {set port $start_port} {$port <= $end_port} {incr port} {
        if {[scan_port $host $port]} {
            lappend open_ports $port
        }
    }
    return $open_ports
}
```

#### Network Monitor

```tcl
# Network connection monitor
proc monitor_connections {} {
    set connections [list]
    foreach file [glob /proc/*/fd/*] {
        if {[catch {
            set target [file readlink $file]
            if {[string match "socket:*" $target]} {
                lappend connections $target
            }
        }]} {
            continue
        }
    }
    return $connections
}

# Usage
set connections [monitor_connections]
puts "Active connections: $connections"
```

### Security Features

#### SSL/TLS Support

```tcl
# SSL/TLS client
package require tls

proc connect_secure {host port} {
    set sock [socket $host $port]
    fconfigure $sock -buffering line
    tls::import $sock
    tls::handshake $sock
    return $sock
}

# SSL/TLS server
proc start_secure_server {port cert key} {
    tls::init -certfile $cert -keyfile $key
    socket -server accept_secure $port
}

proc accept_secure {sock addr port} {
    tls::import $sock
    tls::handshake $sock
    fconfigure $sock -buffering line
    fileevent $sock readable [list handle_secure_client $sock]
}
```

### Best Practices

- Use non-blocking I/O
- Implement proper error handling
- Use appropriate buffering
- Handle connection timeouts
- Implement proper cleanup
- Use secure protocols
- Handle large data transfers
- Implement proper logging
- Use connection pooling
- Follow protocol standards

### Common Pitfalls

- Blocking the event loop
- Memory leaks
- Connection leaks
- Improper error handling
- Buffer overflows
- Security vulnerabilities
- Resource exhaustion
- Protocol violations
- Timeout issues
- Encoding problems

### Resources

- [Tcl Documentation on socket](https://www.tcl.tk/man/tcl/TclCmd/socket.htm)
- [Tcl Documentation on http](https://www.tcl.tk/man/tcl/TclCmd/http.htm)
- [Tclers Wiki on networking](https://wiki.tcl-lang.org/page/networking)

## Control Structures

### If-Else
```tcl
if {$x > 0} {
    puts "Positive"
} elseif {$x < 0} {
    puts "Negative"
} else {
    puts "Zero"
}
```

### Loops
```tcl
# While loop
set i 0
while {$i < 5} {
    puts $i
    incr i
}

# For loop
for {set i 0} {$i < 5} {incr i} {
    puts $i
}

# Foreach loop
foreach item {a b c} {
    puts $item
}
```

## Procedures

```tcl
proc greet {name {greeting "Hello"}} {
    return "$greeting, $name!"
}

greet "World";      # => Hello, World!
greet "World" "Hi"; # => Hi, World!
```

## Data Structures

### Lists
```tcl
set colors [list red green blue]
lappend colors yellow;    # Add to list
set first [lindex $colors 0];  # Get first element
set len [llength $colors];     # Get length
```

### Arrays
```tcl
array set person {
    name    John
    age     30
    city    "New York"
}

set name $person(name);  # Get value
set person(age) 31;      # Update value
array names person;      # Get all keys
```

### Dictionaries (Tcl 8.5+)
```tcl
dict set user name "John"
dict set user age 30
dict get $user name;  # Get value
```

## String Operations

```tcl
set str "Tcl Programming"
string length $str;     # Get length
string toupper $str;    # Convert to uppercase
string first "Pro" $str; # Find substring
string trim $str;       # Trim whitespace
split "a,b,c" ,;        # Split string
join {a b c} "->";      # Join list to string
```

## File I/O

### Basic File Operations
```tcl
# Reading files
set fh [open "file.txt" r];     # Open for reading
set content [read $fh];         # Read entire file
set lines [split [read $fh] "\n"];  # Read as lines
close $fh

# Writing to files
set fh [open "output.txt" w];   # Overwrite
puts $fh "Line 1"
puts $fh "Line 2"
close $fh

# File information
file exists $path
file size $filename
file mtime $filename
file type $filename
file normalize $path;      # Get absolute path
file nativename $path;    # Platform-specific path

# Directory operations
file mkdir $dirPath
file delete $fileOrDir
file copy $source $dest
file rename $old $new
file link $linkname $target;  # Create symlink
```

### File Encoding and Binary Data
```tcl
# Text files with specific encoding
set fh [open "file.txt" r]
fconfigure $fh -encoding utf-8;  # Set encoding
fconfigure $fh -translation auto;  # Handle line endings
set content [read $fh]
close $fh

# Binary files
set fh [open "image.jpg" rb];  # 'b' for binary mode
fconfigure $fh -translation binary
set binary_data [read $fh]
close $fh

# Binary data manipulation
binary scan $binary_data H8 first_four_bytes;  # Read as hex
binary format H* "48656c6c6f";  # Create binary from hex
```

## Network Programming

### TCP Server
```tcl
# Basic TCP server
proc start_server {port} {
    socket -server accept $port
    puts "Server listening on port $port"
    vwait forever
}

proc accept {sock addr port} {
    puts "Connection from $addr:$port"
    fconfigure $sock -buffering line -encoding utf-8
    fileevent $sock readable [list handle_client $sock]
}

proc handle_client {sock} {
    if {[eof $sock] || [catch {gets $sock line}]} {
        close $sock
        return
    }
    puts "Received: $line"
    puts $sock "Echo: $line"
    flush $sock
}

# Start server
start_server 8080
```

### TCP Client
```tcl
# Basic TCP client
proc connect_to_server {host port} {
    set sock [socket $host $port]
    fconfigure $sock -buffering line -encoding utf-8
    return $sock
}

proc send_message {sock message} {
    puts $sock $message
    flush $sock
    set response [gets $sock]
    return $response
}

# Usage
set sock [connect_to_server localhost 8080]
set response [send_message $sock "Hello, server!"]
puts "Server response: $response"
close $sock
```

### UDP Communication
```tcl
# UDP server
proc start_udp_server {port} {
    set sock [socket -server accept_udp $port]
    fconfigure $sock -buffering none
    vwait forever
}

proc accept_udp {sock} {
    fconfigure $sock -buffering none
    fileevent $sock readable [list handle_udp $sock]
}

proc handle_udp {sock} {
    set data [read $sock]
    set peer [fconfigure $sock -peer]
    puts "Received from $peer: $data"
    puts -nonewline $sock "Echo: $data"
}

# UDP client
proc send_udp {host port message} {
    set sock [socket -datagram $host $port]
    fconfigure $sock -buffering none
    puts -nonewline $sock $message
    set response [read $sock]
    close $sock
    return $response
}
```

### HTTP and Web Requests
```tcl
package require http

# Simple GET request
set token [http::geturl "https://api.example.com/data"]
set data [http::data $token]
http::cleanup $token

# POST request with headers and data
set headers [list \
    Content-Type "application/json" \
    Authorization "Bearer $token"
]
set response [http::geturl "https://api.example.com/endpoint" \
    -method POST \
    -headers $headers \
    -query $post_data
]
set status [http::status $response]
set data [http::data $response]
http::cleanup $response
```

## Text Encodings and Character Sets

### Common Encodings
```tcl
# Common encodings in Tcl
set common_encodings {
    utf-8       "Unicode (UTF-8)"
    iso8859-1   "Western European"
    cp1252      "Windows Western"
    shiftjis    "Japanese"
    euc-jp      "Japanese (EUC)"
    gb2312      "Simplified Chinese"
    big5        "Traditional Chinese"
    koi8-r      "Russian"
    ascii       "ASCII only"
}
```

### Encoding Conversion
```tcl
# Set system default encoding (affects new channels)
encoding system utf-8

# Convert between encodings
set utf8_text "Hello, "
set sjis_text [encoding convertto shiftjis $utf8_text]
set back_to_utf8 [encoding convertfrom shiftjis $sjis_text]

# Check if string is valid in encoding
proc is_valid_encoding {string encoding} {
    return [string equal [encoding convertfrom $encoding \
                         [encoding convertto $encoding $string]] $string]
}
```

### File Encoding Detection
```tcl
# Simple encoding detection
proc detect_encoding {filename} {
    set fh [open $filename r]; # Open file for reading
    fconfigure $fh -translation binary; # Read as binary
    set data [read $fh 4096];  # Read first 4KB
    close $fh
    
    # Common encodings to try (in order of likelihood)
    foreach enc {utf-8 cp1252 iso8859-1} {
        if {![catch {
            set test [encoding convertfrom $enc [encoding convertto $enc $data]]
            if {[string equal $test $data]} {
                return $enc
            }
        }]} {
            continue
        }
    }
    return "unknown"
}
```

### Common Encoding Pitfalls
```tcl
# 1. Not specifying encoding for text files
set fh [open "file.txt" r]; # Uses system encoding

# 2. Mixing binary and text modes
set fh [open "data.bin" r]; # Text mode can corrupt binary data

# 3. Assuming default encoding for external data
proc process_file {filename} {
    set fh [open $filename r]; # No encoding specified
    # ...
}

# 4. Not handling BOM (Byte Order Mark)
proc read_with_bom {filename} {
    set fh [open $filename r]
    fconfigure $fh -encoding unicode
    set content [read $fh]
    close $fh
    return $content
}

# 5. Incorrect line ending handling
fconfigure $fh -translation auto;  # Handles CR, LF, CRLF
fconfigure $fh -translation lf;    # Force Unix line endings
fconfigure $fh -translation crlf;  # Force Windows line endings
```

### Network Encoding Best Practices
```tcl
# Always set encoding for text protocols
socket -server accept $port
proc accept {sock addr port} {
    fconfigure $sock -encoding utf-8 -buffering line; # Set UTF-8 for text
    # ...
}

# For binary protocols
set sock [socket $host $port]
fconfigure $sock -translation binary -buffering none; # Binary mode

# Handle encoding in HTTP headers
proc send_http_response {sock content type} {
    puts $sock "HTTP/1.1 200 OK";
    puts $sock "Content-Type: $type; charset=utf-8";
    puts $sock "Content-Length: [string length [encoding convertto utf-8 $content]]";
    puts $sock "";
    puts -nonewline $sock $content;
    flush $sock;
}
```

### Working with Different Character Sets
```tcl
# Convert between character sets
proc convert_charset {string from to} {
    set temp [encoding convertto $from $string];
    return [encoding convertfrom $to $temp];
}

# Example: Convert from Windows-1252 to UTF-8
set win1252_text [encoding convertto cp1252 $utf8_text];
set back_to_utf8 [encoding convertfrom cp1252 $win1252_text];

# Handle BOM (Byte Order Mark)
proc read_unicode_file {filename} {
    set fh [open $filename r];
    fconfigure $fh -encoding unicode;
    set content [read $fh];
    close $fh;
    return $content;
}

```
### Resources
- [Official Tcl Documentation](https://www.tcl.tk/doc/)
- [Tcl/Tk for Real Programmers](https://www.beedub.com/book/)
- [Tclers Wiki](https://wiki.tcl-lang.org/)

## Array Commands

Tcl arrays are associative arrays (hash tables) that store key-value pairs. This section covers the comprehensive usage of array commands.

### Basic Array Operations

```tcl
# Creating arrays
array set colors {
    red    #FF0000
    green  #00FF00
    blue   #0000FF
}

# Accessing elements
set red_hex $colors(red);     # Get value
set colors(yellow) #FFFF00;   # Set value

# Checking existence
if {[info exists colors(red)]} {
    puts "Red exists"
}

# Removing elements
unset colors(red)
```

### Array Commands

```tcl
# Get all array names
array names colors;                    # Returns all keys
array names colors "r*";              # Returns keys matching pattern
array names colors -exact "red";      # Exact match

# Get array size
array size colors;                    # Returns number of elements

# Get array statistics
array statistics colors;              # Returns array statistics

# Check if variable is an array
if {[array exists colors]} {
    puts "colors is an array"
}

# Get array get/set
array get colors;                     # Returns list of key-value pairs
array set new_colors [array get colors];  # Copy array
```

### Iterating Over Arrays

```tcl
# Using array names
foreach key [array names colors] {
    puts "$key: $colors($key)"
}

# Using array get
foreach {key value} [array get colors] {
    puts "$key: $value"
}

# Using array startsearch
set search [array startsearch colors]
while {[array anymore colors $search]} {
    set key [array nextelement colors $search]
    puts "$key: $colors($key)"
}
array donesearch colors $search
```

### Array Operations

```tcl
# Copying arrays
array set new_colors [array get colors]

# Merging arrays
array set colors [array get new_colors]

# Clearing arrays
array unset colors

# Checking if array is empty
if {[array size colors] == 0} {
    puts "Array is empty"
}
```

### Advanced Usage

#### Nested Arrays

```tcl
# Creating nested arrays
array set config {
    database.host     localhost
    database.port     5432
    database.user     admin
    database.pass     secret
}

# Accessing nested values
set db_host $config(database.host)
```

#### Array Patterns

```tcl
# Using patterns with array names
set red_keys [array names colors "red*"]
set color_keys [array names colors "c*"]

# Case-insensitive matching
set keys [array names colors -nocase "RED*"]
```

### Best Practices

- Use meaningful array names
- Use consistent key naming conventions
- Use array get/set for copying arrays
- Use array names with patterns for filtering
- Use array exists to check if variable is an array
- Use array unset to clear arrays
- Use array statistics for debugging

### Common Pitfalls

- Forgetting to check if array exists
- Using array names without patterns when needed
- Not using array get/set for copying
- Using unset instead of array unset
- Not checking array size before iteration

### Resources

- [Tcl Documentation on array](https://www.tcl.tk/man/tcl/TclCmd/array.htm)
- [Tclers Wiki on array](https://wiki.tcl-lang.org/page/array)

## Regular Expressions

Tcl provides powerful regular expression capabilities through the `regexp` and `regsub` commands. This section covers the most common uses and best practices.

### Basic Usage

#### regexp Command

```tcl
# Basic pattern matching
set str "Hello, World!"
regexp {World} $str;  # Returns 1 if pattern matches

# Capturing groups
regexp {(\w+), (\w+)} $str -> greeting name
puts "Greeting: $greeting, Name: $name";  # => Greeting: Hello, Name: World

# Named captures
regexp -named {^(?<greeting>\w+), (?<name>\w+)} $str
puts "Greeting: $greeting, Name: $name"

# Case-insensitive matching
regexp -nocase {world} $str;  # Returns 1

# Line anchors
regexp {^Hello} $str;  # Start of string
regexp {!$} $str;      # End of string
```

#### regsub Command

```tcl
# Basic substitution
set str "Hello, World!"
regsub {World} $str "Tcl" new_str
puts $new_str;  # => Hello, Tcl!

# Global substitution
regsub -all {l} $str "L" new_str
puts $new_str;  # => HeLLo, WorLd!

# Using captured groups in replacement
regsub {(\w+), (\w+)} $str "\\2, \\1" new_str
puts $new_str;  # => World, Hello!
```

### Common Patterns

```tcl
# Word boundaries
regexp {\bword\b} "a word here";  # Matches whole words only

# Character classes
regexp {[aeiou]} "hello";  # Matches any vowel
regexp {[^aeiou]} "hello"; # Matches any non-vowel
regexp {\d} "123";         # Matches any digit
regexp {\s} "a b";         # Matches any whitespace
regexp {\w} "a1";          # Matches word character

# Quantifiers
regexp {a+} "aaa";     # One or more
regexp {a*} "aaa";     # Zero or more
regexp {a?} "a";       # Zero or one
regexp {a{2,4}} "aaa"; # Between 2 and 4

# Alternation
regexp {cat|dog} "cat";  # Matches either cat or dog
```

### Advanced Usage

#### Complex Patterns

```tcl
# Email validation
set email "user@example.com"
regexp {^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$} $email

# Phone number
set phone "(123) 456-7890"
regexp {^\(\d{3}\)\s*\d{3}-\d{4}$} $phone

# URL parsing
set url "https://example.com/path?param=value"
regexp {^(https?)://([^/]+)(/.*)?$} $url -> protocol host path
```

#### Performance Optimization

```tcl
# Compile regexp for repeated use
set pattern [regexp -expanded {
    ^                   # Start of string
    [A-Za-z0-9._%+-]+  # Username
    @                  # @ symbol
    [A-Za-z0-9.-]+     # Domain
    \.                 # Dot
    [A-Za-z]{2,}       # TLD
    $                  # End of string
}]
```

### Best Practices

- Use `-expanded` flag for complex patterns to allow comments and whitespace
- Use `-line` for line-by-line matching
- Use `-all` for global matching
- Use `-nocase` for case-insensitive matching
- Use `-inline` to return all matches
- Use `-start` to specify starting position
- Use `-indices` to get match positions

### Common Pitfalls

- Tcl uses `{` and `}` for pattern grouping, not `(` and `)`
- Use `\\` for literal backslash in patterns
- Remember that `*` and `+` are greedy by default
- Use `-expanded` for complex patterns to improve readability

### Resources

- [Tcl Documentation on regexp](https://www.tcl.tk/man/tcl/TclCmd/regexp.htm)
- [Tcl Documentation on regsub](https://www.tcl.tk/man/tcl/TclCmd/regsub.htm)
