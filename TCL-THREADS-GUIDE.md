# Complete TCL Threads Programming Guide

## Table of Contents
1. [Introduction to TCL Threads](#introduction)
2. [Thread Package Overview](#package-overview)
3. [Creating and Managing Threads](#creating-threads)
4. [Thread Communication](#communication)
5. [Synchronization Primitives](#synchronization)
6. [Thread-Safe Programming](#thread-safety)
7. [Advanced Features](#advanced-features)
8. [Performance Considerations](#performance)
9. [Common Pitfalls and Solutions](#pitfalls)
10. [Best Practices](#best-practices)
11. [Debugging and Troubleshooting](#debugging)
12. [Complete Examples](#examples)

## Introduction to TCL Threads {#introduction}

TCL's thread extension provides preemptive threading capabilities, allowing multiple threads of execution within a single TCL process. Each thread maintains its own interpreter and variable space, providing isolation while enabling controlled communication between threads.

### Key Concepts
- **Thread Isolation**: Each thread has its own interpreter instance
- **Message Passing**: Primary communication mechanism between threads
- **Shared Variables**: Optional shared memory mechanism
- **Thread Pools**: Efficient management of worker threads
- **Event-Driven**: Integration with TCL's event loop

### Prerequisites
```tcl
package require Thread
```

## Thread Package Overview {#package-overview}

The Thread package provides several core commands:

### Core Commands
- `thread::create` - Create new threads
- `thread::send` - Send messages between threads
- `thread::wait` - Wait for thread completion
- `thread::join` - Join threads and retrieve results
- `thread::exit` - Terminate threads
- `thread::id` - Get current thread ID
- `thread::names` - List all thread IDs

### Synchronization Commands
- `thread::mutex` - Mutex operations
- `thread::cond` - Condition variables
- `thread::rwmutex` - Reader-writer locks

### Shared Variables

## Thread Extension API Reference {#api-reference}

### Thread Management

```tcl
# Create a new thread with optional script
thread::create ?-joinable? ?-preserved? ?script?

# Get the ID of the current thread
thread::id

# Get IDs of all threads
thread::names

# Wait for a thread to exit
thread::join threadId ?resultVarName?

# Exit the current thread with optional result
thread::exit ?result?

# Get thread's Tcl interpreter
thread::interp
```

### Thread Communication

```tcl
# Send a script to another thread for evaluation
thread::send ?-async? ?-head? threadId script ?varname?

# Create a new thread-shared queue
thread::queue create ?queueName?

# Put data into a queue
thread::queue put queueName item ?item ...?

# Get data from a queue (blocks if empty)
thread::queue get queueName

# Get data from a queue with timeout (ms)
thread::queue get -unbuffered queueName ?timeout?

# Get the number of items in the queue
thread::queue size queueName

# Destroy a queue
thread::queue destroy queueName
```

### Mutex Operations

```tcl
# Create a new mutex
thread::mutex create

# Lock a mutex (blocks if already locked)
thread::mutex lock mutexId

# Unlock a mutex
thread::mutex unlock mutexId

# Attempt to lock a mutex (returns 1 if successful, 0 if not)
thread::mutex trylock mutexId

# Destroy a mutex
thread::mutex destroy mutexId
```

### Condition Variables

```tcl
# Create a new condition variable
thread::cond create

# Wait on a condition variable
thread::cond wait condId mutexId ?ms?

# Signal one thread waiting on the condition
thread::cond notify condId

# Signal all threads waiting on the condition
thread::cond broadcast condId

# Destroy a condition variable
thread::cond destroy condId
```

### Thread-Shared Variables (TSV)

```tcl
# Set a thread-shared variable
tsv::set varName ?key? value

# Get a thread-shared variable
tsv::get varName ?key?

# Get all keys of a thread-shared array
tsv::array names arrayName ?pattern?

# Unset a thread-shared variable
tsv::unset varName ?key?
```

### Thread Pools

```tcl
# Create a thread pool
thread::pool create ?-minworkers count? ?-maxworkers count? ?-initcmd script?

# Add work item to the pool
thread::pool post ?-detached? poolId script ?varname?

# Wait for all tasks in the pool to complete
thread::pool wait poolId

# Get pool statistics
thread::pool stats poolId

# Destroy a thread pool
thread::pool destroy poolId
```

### Shared Variables
- `tsv::set`, `tsv::get` - Thread-shared variables
- `tsv::array` - Shared array operations
- `tsv::lock`, `tsv::unlock` - Shared variable locking

## Creating and Managing Threads {#creating-threads}

### Basic Thread Creation

```tcl
# Create a simple thread
set tid [thread::create {
    # This code runs in the new thread
    set result "Hello from thread [thread::id]"
    # Thread continues running until explicitly terminated
}]

puts "Created thread: $tid"
```

### Thread with Script File

```tcl
# Create thread and source a script
set tid [thread::create]
thread::send $tid {source worker_script.tcl}
```

### Thread with Return Value

```tcl
# Create thread that returns a value
set tid [thread::create {
    # Perform some computation
    set result [expr {2 * 21}]
    thread::wait  ;# Wait for join
}]

# Join thread and get result
set result [thread::join $tid]
puts "Thread result: $result"
```

### Thread Termination

```tcl
# Graceful termination
thread::send $tid {thread::exit}

# Forced termination (use with caution)
thread::release $tid

# Wait for thread to finish
thread::join $tid
```

## Thread Communication {#communication}

### Message Passing with thread::send

```tcl
# Synchronous message sending
set result [thread::send $tid {
    expr {$x + $y}
} -x 10 -y 20]

# Asynchronous message sending
thread::send -async $tid {
    puts "Async message received"
}

# Send with callback
thread::send -async $tid {
    return "callback result"
} [list apply {{result} {
    puts "Callback received: $result"
}}]
```

### Message Queue Pattern

```tcl
# Worker thread with message queue
set worker_tid [thread::create {
    proc process_message {msg} {
        # Process the message
        puts "Processing: $msg"
        return "Processed: $msg"
    }

    # Main message loop
    while {1} {
        set msg [thread::receive]
        if {$msg eq "QUIT"} break
        set result [process_message $msg]
        thread::send [thread::parent] [list result_callback $result]
    }
}]

# Send messages to worker
thread::send $worker_tid {thread::queue_put "task1"}
thread::send $worker_tid {thread::queue_put "task2"}
```

### Bidirectional Communication

```tcl
# Parent thread
proc handle_response {response} {
    puts "Received response: $response"
}

set worker_tid [thread::create [format {
    set parent_tid %s

    proc send_to_parent {msg} {
        global parent_tid
        thread::send -async $parent_tid [list handle_response $msg]
    }

    # Worker logic here
    send_to_parent "Worker started"
} [thread::id]]]
```

## Synchronization Primitives {#synchronization}

### Mutexes

```tcl
# Create and use mutex
set mutex [thread::mutex create]

proc critical_section {} {
    global mutex shared_counter

    thread::mutex lock $mutex
    try {
        # Critical section code
        incr shared_counter
        after 100  ;# Simulate work
    } finally {
        thread::mutex unlock $mutex
    }
}

# Cleanup
thread::mutex destroy $mutex
```

### Condition Variables

```tcl
set mutex [thread::mutex create]
set condition [thread::cond create]
set ready 0

# Producer thread
set producer [thread::create [format {
    set mutex %s
    set condition %s

    thread::mutex lock $mutex
    # Produce data
    set ready 1
    thread::cond notify $condition
    thread::mutex unlock $mutex
} $mutex $condition]]

# Consumer thread
thread::mutex lock $mutex
while {!$ready} {
    thread::cond wait $condition $mutex
}
# Consume data
thread::mutex unlock $mutex
```

### Reader-Writer Locks

```tcl
set rwlock [thread::rwmutex create]

# Reader
proc read_data {} {
    global rwlock
    thread::rwmutex rlock $rwlock
    try {
        # Read operation
        set data [get_shared_data]
    } finally {
        thread::rwmutex unlock $rwlock
    }
    return $data
}

# Writer
proc write_data {new_data} {
    global rwlock
    thread::rwmutex wlock $rwlock
    try {
        # Write operation
        set_shared_data $new_data
    } finally {
        thread::rwmutex unlock $rwlock
    }
}
```

## Thread-Safe Programming {#thread-safety}

### Thread-Shared Variables (TSV)

```tcl
# Initialize shared variable
tsv::set shared counter 0

# Atomic increment
proc safe_increment {} {
    tsv::lock shared
    try {
        set current [tsv::get shared counter]
        tsv::set shared counter [incr current]
    } finally {
        tsv::unlock shared
    }
}

# Shared arrays
tsv::array set shared_config {
    timeout 30
    retries 3
    debug 1
}

# Access shared array
set timeout [tsv::array get shared_config timeout]
```

### Thread-Local Storage

```tcl
# Each thread maintains its own copy
set thread_local_data [dict create]

proc set_thread_data {key value} {
    global thread_local_data
    dict set thread_local_data $key $value
}

proc get_thread_data {key} {
    global thread_local_data
    return [dict get $thread_local_data $key]
}
```

### Safe Resource Management

```tcl
proc safe_file_operation {filename} {
    set mutex [thread::mutex create file_$filename]

    thread::mutex lock $mutex
    try {
        set fd [open $filename r]
        set data [read $fd]
        close $fd
        return $data
    } finally {
        thread::mutex unlock $mutex
        thread::mutex destroy $mutex
    }
}
```

## Advanced Features {#advanced-features}

### Thread Pools

```tcl
# Create thread pool
proc create_thread_pool {size} {
    set pool {}
    for {set i 0} {$i < $size} {incr i} {
        set tid [thread::create {
            proc worker_loop {} {
                while {1} {
                    set task [thread::queue_get]
                    if {$task eq "SHUTDOWN"} break

                    # Execute task
                    set result [eval $task]

                    # Send result back if needed
                    if {[info exists ::callback]} {
                        thread::send $::parent_tid [list $::callback $result]
                    }
                }
            }

            set ::parent_tid [thread::parent]
            worker_loop
        }]
        lappend pool $tid
    }
    return $pool
}

# Submit task to pool
proc submit_task {pool task {callback ""}} {
    set tid [lindex $pool [expr {int(rand() * [llength $pool])}]]
    if {$callback ne ""} {
        thread::send $tid [list set ::callback $callback]
    }
    thread::send $tid [list thread::queue_put $task]
}

# Shutdown pool
proc shutdown_thread_pool {pool} {
    foreach tid $pool {
        thread::send $tid {thread::queue_put "SHUTDOWN"}
        thread::join $tid
    }
}
```

### Thread-Safe Event Processing

```tcl
# Event dispatcher thread
set dispatcher [thread::create {
    set event_handlers [dict create]

    proc register_handler {event handler} {
        global event_handlers
        dict set event_handlers $event $handler
    }

    proc dispatch_event {event data} {
        global event_handlers
        if {[dict exists $event_handlers $event]} {
            set handler [dict get $event_handlers $event]
            eval [list $handler $data]
        }
    }

    # Event loop
    while {1} {
        set event [thread::queue_get]
        if {$event eq "QUIT"} break

        dispatch_event [lindex $event 0] [lindex $event 1]
    }
}]

# Register event handlers
thread::send $dispatcher [list register_handler "data_received" handle_data_event]

# Usage example
proc handle_data_event {data} {
    puts "Received data in thread [thread::id]: $data"
}

# Post events to the dispatcher
thread::send -async $dispatcher [list thread::queue_put [list "data_received" "Test data"]]

# Cleanup
thread::send -async $dispatcher {thread::queue_put "QUIT"}
thread::join $dispatcher
```

### Producer-Consumer Pattern

```tcl
# Buffer with size limit
set buffer_size 10
set buffer {}
set buffer_mutex [thread::mutex create]
set not_full [thread::cond create]
set not_empty [thread::cond create]

# Producer
proc produce_item {item} {
    global buffer buffer_size buffer_mutex not_full not_empty

    thread::mutex lock $buffer_mutex
    while {[llength $buffer] >= $buffer_size} {
        thread::cond wait $not_full $buffer_mutex
    }

    lappend buffer $item
    thread::cond notify $not_empty
    thread::mutex unlock $buffer_mutex
}

# Consumer
proc consume_item {} {
    global buffer buffer_mutex not_full not_empty

    thread::mutex lock $buffer_mutex
    while {[llength $buffer] == 0} {
        thread::cond wait $not_empty $buffer_mutex
    }

    set item [lindex $buffer 0]
    set buffer [lrange $buffer 1 end]
    thread::cond notify $not_full
    thread::mutex unlock $buffer_mutex

    return $item
}
```

## Performance Considerations {#performance}

### Minimizing Thread Creation Overhead

```tcl
# Bad: Creating threads for each task
foreach task $tasks {
    set tid [thread::create "process_task {$task}"]
    thread::join $tid
}

# Good: Reuse thread pool
set pool [create_thread_pool 4]
foreach task $tasks {
    submit_task $pool "process_task {$task}"
}
shutdown_thread_pool $pool
```

### Efficient Message Passing

```tcl
# Bad: Many small messages
for {set i 0} {$i < 1000} {incr i} {
    thread::send $tid "process_item $i"
}

# Good: Batch processing
set batch {}
for {set i 0} {$i < 1000} {incr i} {
    lappend batch $i
    if {[llength $batch] >= 50} {
        thread::send $tid "process_batch {$batch}"
        set batch {}
    }
}
if {[llength $batch] > 0} {
    thread::send $tid "process_batch {$batch}"
}
```

### Memory Management

```tcl
# Proper cleanup in long-running threads
proc worker_thread {} {
    while {1} {
        set task [thread::queue_get]
        if {$task eq "QUIT"} break

        # Process task
        set result [process_task $task]

        # Clean up large variables
        unset task
        unset result

        # Periodic garbage collection
        if {[incr ::cycle_count] % 100 == 0} {
            # Force garbage collection if available
            catch {gc}
        }
    }
}
```

## Common Pitfalls and Solutions {#pitfalls}

### Deadlock Prevention

```tcl
# Problem: Potential deadlock
proc bad_example {} {
    thread::mutex lock $mutex1
    thread::mutex lock $mutex2  ;# Potential deadlock
    # ... critical section ...
    thread::mutex unlock $mutex2
    thread::mutex unlock $mutex1
}

# Solution: Consistent lock ordering
set global_lock_order [list $mutex1 $mutex2]

proc acquire_locks {mutexes} {
    set sorted_mutexes [lsort $mutexes]
    foreach mutex $sorted_mutexes {
        thread::mutex lock $mutex
    }
}

proc release_locks {mutexes} {
    set sorted_mutexes [lsort -decreasing $mutexes]
    foreach mutex $sorted_mutexes {
        thread::mutex unlock $mutex
    }
}
```

### Race Condition Avoidance

```tcl
# Problem: Race condition
set shared_counter 0
proc unsafe_increment {} {
    global shared_counter
    set temp $shared_counter
    after 1  ;# Simulate delay
    set shared_counter [expr {$temp + 1}]
}

# Solution: Atomic operations
set counter_mutex [thread::mutex create]
proc safe_increment {} {
    global shared_counter counter_mutex
    thread::mutex lock $counter_mutex
    incr shared_counter
    thread::mutex unlock $counter_mutex
}

# Better: Use TSV for atomic operations
tsv::set counters main 0
proc atomic_increment {} {
    tsv::incr counters main
}
```

### Memory Leaks in Thread Communication

```tcl
# Problem: Accumulating messages
thread::send -async $tid {
    # Long-running operation
    heavy_computation
} ;# Result is lost, may accumulate

# Solution: Proper async handling
thread::send -async $tid {
    heavy_computation
} [list apply {{result} {
    # Handle result properly
    log_result $result
}}]
```

### Thread Synchronization Issues

```tcl
# Problem: Lost wakeups
set ready 0
set condition [thread::cond create]
set mutex [thread::mutex create]

# Producer sets ready before consumer waits
set ready 1
thread::cond notify $condition  ;# Lost wakeup

thread::mutex lock $mutex
while {!$ready} {
    thread::cond wait $condition $mutex  ;# Will wait forever
}
thread::mutex unlock $mutex

# Solution: Always check condition under lock
thread::mutex lock $mutex
if {!$ready} {
    set ready 1
    thread::cond notify $condition
}
thread::mutex unlock $mutex

thread::mutex lock $mutex
while {!$ready} {
    thread::cond wait $condition $mutex
}
thread::mutex unlock $mutex
```

## Best Practices {#best-practices}

### Thread Design Principles

1. **Single Responsibility**: Each thread should have a clear, single purpose
2. **Minimize Shared State**: Use message passing over shared variables when possible
3. **Fail Fast**: Detect and handle errors early in thread execution
4. **Resource Cleanup**: Always clean up resources in thread termination

### Error Handling in Threads

```tcl
# Robust error handling pattern
proc safe_worker_thread {} {
    try {
        # Main thread logic
        while {1} {
            set task [thread::queue_get]
            if {$task eq "QUIT"} break

            try {
                process_task $task
            } trap {TASK_ERROR} {msg opts} {
                log_error "Task processing failed: $msg"
                # Continue with next task
            }
        }
    } trap {FATAL_ERROR} {msg opts} {
        log_error "Fatal error in worker thread: $msg"
        # Notify parent thread of failure
        thread::send [thread::parent] [list worker_failed [thread::id] $msg]
    } finally {
        # Cleanup resources
        cleanup_thread_resources
    }
}
```

### Monitoring and Logging

```tcl
# Thread monitoring utilities
proc monitor_thread {tid name} {
    set start_time [clock seconds]

    # Check if thread is responsive
    if {[catch {
        thread::send $tid {return "ping"} 5000  ;# 5 second timeout
    } result]} {
        log_warning "Thread $name ($tid) is unresponsive: $result"
        return 0
    }

    return 1
}

# Thread-safe logging
set log_mutex [thread::mutex create]
proc thread_log {level message} {
    global log_mutex

    thread::mutex lock $log_mutex
    try {
        set timestamp [clock format [clock seconds]]
        set tid [thread::id]
        puts "\[$timestamp\] \[Thread $tid\] \[$level\] $message"
        flush stdout
    } finally {
        thread::mutex unlock $log_mutex
    }
}
```

### Configuration Management

```tcl
# Thread-safe configuration
tsv::array set app_config {
    worker_threads 4
    queue_size 100
    timeout 30
    debug_level 1
}

proc get_config {key {default ""}} {
    if {[tsv::array exists app_config $key]} {
        return [tsv::array get app_config $key]
    }
    return $default
}

proc set_config {key value} {
    tsv::array set app_config $key $value
}
```

### Graceful Shutdown

```tcl
# Shutdown coordinator
proc initiate_shutdown {} {
    global worker_threads shutdown_flag

    # Set shutdown flag
    tsv::set app_state shutdown_requested 1

    # Signal all worker threads
    foreach tid $worker_threads {
        thread::send -async $tid {
            if {[info exists ::shutdown_handler]} {
                $::shutdown_handler
            }
        }
    }

    # Wait for graceful shutdown with timeout
    set timeout_ms 10000
    set start_time [clock milliseconds]

    while {[llength $worker_threads] > 0} {
        set current_time [clock milliseconds]
        if {($current_time - $start_time) > $timeout_ms} {
            # Force shutdown remaining threads
            foreach tid $worker_threads {
                catch {thread::release $tid}
            }
            break
        }

        # Check which threads have finished
        set remaining {}
        foreach tid $worker_threads {
            if {[catch {thread::send $tid {return "alive"} 100}]} {
                # Thread has terminated
                catch {thread::join $tid}
            } else {
                lappend remaining $tid
            }
        }
        set worker_threads $remaining

        after 100
    }
}
```

## Debugging and Troubleshooting {#debugging}

### Thread State Inspection

```tcl
proc inspect_thread {tid} {
    puts "Thread ID: $tid"

    # Check if thread exists and is alive
    if {[lsearch [thread::names] $tid] == -1} {
        puts "  Status: Thread does not exist"
        return
    }

    # Test responsiveness
    if {[catch {
        set response [thread::send $tid {return "alive"} 1000]
        puts "  Status: Alive and responsive"
    } error]} {
        puts "  Status: Unresponsive or dead - $error"
        return
    }

    # Get thread information
    if {[catch {
        set info [thread::send $tid {
            list \
                [thread::id] \
                [info exists ::tcl_platform] \
                [array size ::env] \
                [info commands]
        }]
        puts "  Internal ID: [lindex $info 0]"
        puts "  Platform info exists: [lindex $info 1]"
        puts "  Environment variables: [lindex $info 2]"
        puts "  Available commands: [llength [lindex $info 3]]"
    } error]} {
        puts "  Error getting thread info: $error"
    }
}

# Monitor all threads
proc monitor_all_threads {} {
    set threads [thread::names]
    puts "Active threads: [llength $threads]"

    foreach tid $threads {
        inspect_thread $tid
        puts ""
    }
}
```

### Deadlock Detection

```tcl
# Simple deadlock detection utility
set lock_owners [dict create]
set lock_waiters [dict create]
set detection_mutex [thread::mutex create]

proc acquire_tracked_lock {lock_id} {
    global lock_owners lock_waiters detection_mutex
    set tid [thread::id]

    thread::mutex lock $detection_mutex
    dict set lock_waiters $lock_id $tid
    thread::mutex unlock $detection_mutex

    # Actual lock acquisition
    thread::mutex lock $lock_id

    thread::mutex lock $detection_mutex
    dict set lock_owners $lock_id $tid
    dict unset lock_waiters $lock_id
    thread::mutex unlock $detection_mutex
}

proc release_tracked_lock {lock_id} {
    global lock_owners detection_mutex

    thread::mutex unlock $lock_id

    thread::mutex lock $detection_mutex
    dict unset lock_owners $lock_id
    thread::mutex unlock $detection_mutex
}

proc check_deadlock {} {
    global lock_owners lock_waiters detection_mutex

    thread::mutex lock $detection_mutex
    set owners [dict keys $lock_owners]
    set waiters [dict keys $lock_waiters]
    thread::mutex unlock $detection_mutex

    # Simple cycle detection (can be enhanced)
    foreach lock $waiters {
        set waiter [dict get $lock_waiters $lock]
        # Check if waiter owns any locks that others are waiting for
        # Implementation depends on specific requirements
    }
}
```

### Performance Profiling

```tcl
# Thread performance profiler
proc profile_thread {tid duration} {
    set samples {}
    set start_time [clock milliseconds]
    set end_time [expr {$start_time + $duration}]

    while {[clock milliseconds] < $end_time} {
        set sample_start [clock microseconds]

        if {[catch {
            thread::send $tid {
                return [list \
                    [clock microseconds] \
                    [info level] \
                    [catch {info frame 0} frame_info] \
                ]
            } 100
        } result]} {
            lappend samples [list $sample_start "ERROR" $result]
        } else {
            lappend samples [list $sample_start "OK" $result]
        }

        after 10  ;# Sample every 10ms
    }

    # Analyze samples
    set responsive 0
    set unresponsive 0

    foreach sample $samples {
        if {[lindex $sample 1] eq "OK"} {
            incr responsive
        } else {
            incr unresponsive
        }
    }

    puts "Thread $tid profiling results:"
    puts "  Total samples: [llength $samples]"
    puts "  Responsive: $responsive"
    puts "  Unresponsive: $unresponsive"
    puts "  Response rate: [expr {$responsive * 100.0 / [llength $samples]}]%"
}
```

## Troubleshooting Common Issues {#troubleshooting}

### Thread Hangs

**Symptom**: Threads stop responding or appear to hang.

**Possible Causes and Solutions**:
1. **Deadlock**: Check for circular wait conditions
   ```tcl
   # Use the deadlock detection utility periodically
   after 60000 detect_deadlocks  ;# Check every minute
   ```

2. **Infinite Wait**: Ensure all conditions are properly signaled
   ```tcl
   # Always use condition variables with a timeout
   if {![thread::cond wait $condition $mutex 5000]} {
       log_thread "WARNING" "Timeout waiting for condition"
   }
   ```

### Memory Leaks

**Symptom**: Memory usage grows over time.

**Diagnosis and Prevention**:
1. **Track object creation/destruction**:
   ```tcl
   proc track_object {obj} {
       incr ::object_count
       trace add command $obj delete [list incr ::object_count -1]
       return $obj
   }
   ```

2. **Periodic cleanup**:
   ```tcl
   proc cleanup_resources {} {
       # Clean up unused resources
       cleanup_temporary_files
       cleanup_idle_connections
       
       # Schedule next cleanup
       after 300000 cleanup_resources  ;# Every 5 minutes
   }
   ```

### Performance Bottlenecks

**Symptom**: Poor performance under load.

**Optimization Techniques**:
1. **Profile critical sections**:
   ```tcl
   proc profile {script name} {
       set start [clock microseconds]
       set result [uplevel 1 $script]
       set elapsed [expr {[clock microseconds] - $start}]
       log_thread "PROFILE" "$name took ${elapsed}µs"
       return $result
   }

   # Usage
   profile {
       # Code to profile
   } "expensive_operation"
   ```

2. **Reduce lock contention**:
   ```tcl
   # Fine-grained locking
   set mutexes [lmap i {1 2 3 4 5} {thread::mutex create}]
   proc get_mutex {key} {
       global mutexes
       return [lindex $mutexes [expr {[string length $key] % [llength $mutexes]}]]
   }
   ```

## Testing Threaded Applications {#testing}

### Unit Testing Threaded Code

```tcl
# Test framework setup
package require tcltest

# Test case for thread-safe counter
proc test_thread_safe_counter {} {
    set counter_mutex [thread::mutex create]
    set counter 0

    # Worker procedure
    set worker_script [format {
        global counter counter_mutex
        for {set i 0} {$i < 1000} {incr i} {
            thread::mutex lock $counter_mutex
            incr counter
            thread::mutex unlock $counter_mutex
        }
    }]

    # Create worker threads
    set threads {}
    for {set i 0} {$i < 10} {incr i} {
        lappend threads [thread::create $worker_script]
    }

    # Wait for all threads to complete
    foreach tid $threads {
        thread::join $tid
    }

    # Verify counter
    tcltest::test counter-test-1.1 "Thread-safe counter test" -body {
        return $counter
    } -result 10000
}

# Run tests
test_thread_safe_counter
tcltest::cleanupTests
```

### Stress Testing

```tcl
proc stress_test {worker_count iterations} {
    set workers {}
    set results [dict create]

    # Create workers
    for {set i 0} {$i < $worker_count} {incr i} {
        set tid [thread::create {
            proc stress_work {iterations} {
                set start [clock milliseconds]
                for {set i 0} {$i < $iterations} {incr i} {
                    # Simulate work
                    after 1
                }
                return [expr {[clock milliseconds] - $start}]
            }
        }]
        lappend workers $tid
    }

    # Distribute work
    set i 0
    foreach tid $workers {
        thread::send -async $tid [list stress_work $iterations] [list apply {
            {tid result} {
                global results
                dict set results $tid $result
            }
        } $tid]
    }

    # Wait for completion
    while {[dict size $results] < $worker_count} {
        after 100
    }

    # Report results
    set total_time 0
    dict for {tid time} $results {
        puts "Worker $tid completed in ${time}ms"
        set total_time [expr {$total_time + $time}]
    }

    puts "Average time per worker: [expr {$total_time / $worker_count}]ms"
}
```

## Complete Examples {#examples}

### Example 1: Web Server with Thread Pool

```tcl
package require Thread
package require socket

# Configuration
set server_port 8080
set thread_pool_size 4
set max_connections 100

# Create thread pool for request handling
proc create_request_handler {} {
    return [thread::create {
        proc handle_http_request {client_socket} {
            try {
                # Read HTTP request
                set request [read $client_socket]

                # Simple HTTP response
                set response "HTTP/1.1 200 OK\r\n"
                append response "Content-Type: text/plain\r\n"
                append response "Content-Length: 13\r\n"
                append response "\r\n"
                append response "Hello, World!"

                puts -nonewline $client_socket $response
                flush $client_socket

            } catch {error} {
                puts "Error handling request: $error"
            } finally {
                catch {close $client_socket}
            }
        }

        # Worker loop
        while {1} {
            set task [thread::queue_get]
            if {$task eq "SHUTDOWN"} break

            handle_http_request $task
        }
    }]
}

# Initialize thread pool
set thread_pool {}
for {set i 0} {$i < $thread_pool_size} {incr i} {
    lappend thread_pool [create_request_handler]
}

# Connection dispatcher
set current_thread 0
proc dispatch_connection {client_socket} {
    global thread_pool current_thread

    set tid [lindex $thread_pool $current_thread]
    thread::send -async $tid [list thread::queue_put $client_socket]

    set current_thread [expr {($current_thread + 1) % [llength $thread_pool]}]
}

# Start server
set server_socket [socket -server dispatch_connection $server_port]
puts "Server listening on port $server_port"

# Graceful shutdown handler
proc shutdown_server {} {
    global server_socket thread_pool

    # Stop accepting connections
    close $server_socket

    # Shutdown thread pool
    foreach tid $thread_pool {
        thread::send -async $tid {thread::queue_put "SHUTDOWN"}
    }

    # Wait for threads to finish
    foreach tid $thread_pool {
        catch {thread::join $tid}
    }

    puts "Server shutdown complete"
    exit 0
}

# Handle SIGINT for graceful shutdown
signal trap SIGINT shutdown_server

# Enter event loop
vwait forever
```

### Example 2: Parallel File Processing

```tcl
package require Thread

# Parallel file processor
proc parallel_file_processor {directory pattern worker_count} {
    # Find all files matching pattern
    set files [glob -directory $directory -type f $pattern]
    puts "Found [llength $files] files to process"

    # Create result collection
    tsv::array set results {}
    set results_mutex [thread::mutex create]

    # Worker thread template
    set worker_script [format {
        set results_mutex %s

        proc process_file {filename} {
            # Example: count lines in file
            set line_count 0
            set word_count 0

            try {
                set fd [open $filename r]
                while {[gets $fd line] >= 0} {
                    incr line_count
                    incr word_count [llength [split $line]]
                }
                close $fd

                return [list $line_count $word_count]

            } catch {error} {
                return [list -1 -1 $error]
            }
        }

        proc store_result {filename result} {
            global results_mutex
            thread::mutex lock $results_mutex
            tsv::array set results $filename $result
            thread::mutex unlock $results_mutex
        }

        # Worker main loop
        while {1} {
            set task [thread::queue_get]
            if {$task eq "DONE"} break

            set result [process_file $task]
            store_result $task $result

            # Report progress
            thread::send [thread::parent] [list report_progress $task]
        }
    } $results_mutex]

    # Create worker threads
    set workers {}
    for {set i 0} {$i < $worker_count} {incr i} {
        lappend workers [thread::create $worker_script]
    }

    # Progress tracking
    set processed_count 0
    proc report_progress {filename} {
        global processed_count files
        incr processed_count
        if {$processed_count % 10 == 0 || $processed_count == [llength $files]} {
            puts "Processed $processed_count/[llength $files] files"
        }
    }

    # Distribute work
    set file_index 0
    foreach file $files {
        set worker_id [expr {$file_index % $worker_count}]
        set worker_tid [lindex $workers $worker_id]
        thread::send -async $worker_tid [list thread::queue_put $file]
        incr file_index
    }

    # Signal completion
    foreach tid $workers {
        thread::send -async $tid {thread::queue_put "DONE"}
    }

    # Wait for all workers to complete
    foreach tid $workers {
        thread::join $tid
    }

    # Collect and display results
    puts "\nProcessing Results:"
    puts [format "%-40s %10s %10s" "File" "Lines" "Words"]
    puts [string repeat "-" 62]

    set total_lines 0
    set total_words 0

    foreach filename [tsv::array names results] {
        set result [tsv::array get results $filename]
        set lines [lindex $result 0]
        set words [lindex $result 1]

        if {$lines >= 0} {
            puts [format "%-40s %10d %10d" [file tail $filename] $lines $words]
            incr total_lines $lines
            incr total_words $words
        } else {
            puts [format "%-40s %10s %10s" [file tail $filename] "ERROR" "ERROR"]
        }
    }

    puts [string repeat "-" 62]
    puts [format "%-40s %10d %10d" "TOTAL" $total_lines $total_words]

    # Cleanup
    thread::mutex destroy $results_mutex
    tsv::array unset results
}

# Usage example
if {$argc >= 1} {
    set directory [lindex $argv 0]
    set pattern [expr {$argc >= 2 ? [lindex $argv 1] : "*"}]
    set workers [expr {$argc >= 3 ? [lindex $argv 2] : 4}]

    parallel_file_processor $directory $pattern $workers
} else {
    puts "Usage: $argv0 <directory> \[pattern\] \[worker_count\]"
    exit
}
```

## Further Reading {#further-reading}

### Official Documentation
- [Tcl Thread Extension Documentation](https://tcl.tk/doc/thread/)
- [Tcl Thread Extension Source](https://core.tcl-lang.org/thread/)
- [Tcl Thread Extension Wiki](https://wiki.tcl-lang.org/page/Thread)

### Books
- "Tcl/Tk 8.5 Programming Cookbook" by Bert Wheeler
- "Practical Programming in Tcl and Tk" by Brent Welch
- "Tcl and the Tk Toolkit" by John K. Ousterhout and Ken Jones

### Online Resources
- [Tcl Developer Xchange](https://www.tcl.tk/)
- [Tcl'ers Wiki](https://wiki.tcl-lang.org/)
- [Stack Overflow Tcl Tag](https://stackoverflow.com/questions/tagged/tcl)
- [Tcl/Tk Documentation](https://www.tcl.tk/doc/)

### Related Tools and Extensions
- [Tcllib](https://core.tcl-lang.org/tcllib/doc/tcllib-1-20/embedded/index.html) - Standard Tcl Library
- [TclX](https://sourceforge.net/projects/tclx/) - Extended Tcl
- [TclKit](https://www.equi4.com/tclkit/) - Tcl runtime environment
- [ActiveState Tcl](https://www.activestate.com/products/tcl/) - Commercial Tcl distribution

### Community and Support
- [Tcl Chat Room](https://chat.stackoverflow.com/rooms/19830/tcl) - Real-time help on Stack Overflow
- [Tcl Mailing Lists](https://www.tcl.tk/community/tcl-moderated.html) - Mailing lists for Tcl developers
- [Tcl Core Team](https://core.tcl-lang.org/index) - Core development team and resources

### Related Standards
- [Tcl Improvement Proposals (TIPs)](https://core.tcl-lang.org/tips/doc/trunk/index.md)
- [Tcl Language Reference](https://www.tcl.tk/man/tcl8.6/TclCmd/contents.htm)
- [Thread Extension Reference](https://www.tcl.tk/man/tcl/ThreadCmd/thread.htm)
