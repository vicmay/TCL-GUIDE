# Complete NX Programming Guide
## Next Scripting Framework - Object-Oriented Extension for TCL

### Table of Contents
1. [Introduction to NX](#introduction-to-nx)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Concepts](#basic-concepts)
4. [Classes and Objects](#classes-and-objects)
5. [Methods and Variables](#methods-and-variables)
6. [Inheritance](#inheritance)
7. [Mixins](#mixins)
8. [Filters and Guards](#filters-and-guards)
9. [Metaclasses](#metaclasses)
10. [Advanced Features](#advanced-features)
11. [Best Practices](#best-practices)
12. [Complete Examples](#complete-examples)

## Introduction to NX

NX (Next Scripting Framework) is a modern object-oriented extension for TCL that provides a clean, flexible, and powerful programming model. It was developed as a successor to XOTcl and offers improved performance, better syntax, and enhanced features for object-oriented programming.

### Key Features
- Class-based object orientation
- Multiple inheritance
- Mixins for flexible code reuse
- Method filters and guards
- Introspection capabilities
- Serialization support
- Integration with TCL's namespace system

## Installation and Setup

### Installing NX
```tcl
# Most common installation methods:
# 1. Through package manager (if available)
# 2. Download from https://next-scripting.org/
# 3. Compile from source

# Loading NX in your script
package require nx
```

### Basic Setup
```tcl
#!/usr/bin/env tclsh
package require nx

# Your NX code starts here
```

## Basic Concepts

### Objects vs Classes
In NX, everything is an object. Classes are special objects that can create other objects (instances).

```tcl
# Creating a simple object
nx::Object create myObject

# Creating a class
nx::Class create MyClass

# Creating an instance of a class
MyClass create myInstance
```

### Namespaces and NX
NX integrates well with TCL's namespace system:

```tcl
namespace eval ::myapp {
    nx::Class create MyClass {
        :method hello {} {
            puts "Hello from [self]"
        }
    }
}

::myapp::MyClass create obj
obj hello
```

## Classes and Objects

### Creating Classes

#### Basic Class Definition
```tcl
nx::Class create Person {
    # Class variables (shared among all instances)
    :variable population 0
    
    # Constructor
    :method init {name age} {
        set :name $name
        set :age $age
        incr :population
    }
    
    # Instance methods
    :method getName {} {
        return ${:name}
    }
    
    :method getAge {} {
        return ${:age}
    }
    
    :method introduce {} {
        puts "Hi, I'm ${:name} and I'm ${:age} years old"
    }
    
    # Class method
    :class method getPopulation {} {
        return ${:population}
    }
}
```

#### Creating and Using Objects
```tcl
# Create instances
Person create john "John Doe" 30
Person create jane "Jane Smith" 25

# Use instance methods
john introduce
puts "John's name: [john getName]"
puts "Jane's age: [jane getAge]"

# Use class method
puts "Total population: [Person getPopulation]"
```

### Object Properties and Variables

#### Property Definition
```tcl
nx::Class create Car {
    # Properties with default values
    :property brand "Unknown"
    :property model "Unknown"
    :property {year 2000}
    :property {color "white"}
    
    :method init {args} {
        # Properties can be set during initialization
        puts "Created car: ${:brand} ${:model} (${:year})"
    }
    
    :method describe {} {
        return "${:color} ${:year} ${:brand} ${:model}"
    }
}

# Creating cars with properties
Car create car1 -brand Toyota -model Camry -year 2020 -color blue
Car create car2 -brand Honda -model Civic

puts [car1 describe]
puts [car2 describe]
```

#### Variable Access and Modification
```tcl
nx::Class create Counter {
    :property {count 0}
    
    :method increment {} {
        incr :count
    }
    
    :method decrement {} {
        incr :count -1
    }
    
    :method getValue {} {
        return ${:count}
    }
    
    :method setValue {value} {
        set :count $value
    }
}

Counter create counter1
counter1 increment
counter1 increment
puts "Counter value: [counter1 getValue]"
counter1 setValue 10
puts "New counter value: [counter1 getValue]"
```

## Methods and Variables

### Method Visibility

In NX, methods have explicit visibility modifiers:
- `:method` - **Private method** (default, only accessible within the class hierarchy)
- `:public method` - **Public method** (accessible from outside the object)
- `:protected method` - **Protected method** (accessible within class hierarchy but not from outside)

#### Public Methods (External Interface)
```tcl
nx::Class create Calculator {
    # Public methods - accessible from outside
    :public method add {a b} {
        return [:performOperation + $a $b]
    }
    
    :public method multiply {a b} {
        return [:performOperation * $a $b]
    }
    
    :public method divide {a b} {
        if {$b == 0} {
            error "Division by zero"
        }
        return [:performOperation / $a $b]
    }
    
    # Private method - internal implementation
    :method performOperation {op a b} {
        return [expr "$a $op $b"]
    }
    
    # Protected method - accessible in subclasses
    :protected method validateInput {value} {
        if {![string is double $value]} {
            error "Invalid numeric input: $value"
        }
    }
}

Calculator create calc
puts "5 + 3 = [calc add 5 3]"
puts "4 * 7 = [calc multiply 4 7]"

# This would work - public method
puts "10 / 2 = [calc divide 10 2]"

# This would fail - private method not accessible
# calc performOperation + 1 2  ;# Error!
```

#### Public vs Private Class Methods
```tcl
nx::Class create MathUtils {
    # Public class method - accessible from outside
    :public class method pi {} {
        return [:getPrecisePi 5]
    }
    
    :public class method factorial {n} {
        if {$n <= 1} {
            return 1
        } else {
            return [expr {$n * [:factorial [expr {$n - 1}]]}]
        }
    }
    
    # Private class method - internal helper
    :class method getPrecisePi {precision} {
        switch $precision {
            3 { return 3.142 }
            5 { return 3.14159 }
            default { return 3.14159265359 }
        }
    }
}

puts "Pi = [MathUtils pi]"
puts "5! = [MathUtils factorial 5]"

# This would fail - private class method
# puts [MathUtils getPrecisePi 3]  ;# Error!
```

#### Forward Methods (Method Delegation)
```tcl
nx::Class create Logger {
    :method log {message} {
        puts "\[LOG\] $message"
    }
}

nx::Class create Application {
    :variable logger
    
    :method init {} {
        set :logger [Logger new]
    }
    
    # Forward log calls to the logger object
    :forward log ${:logger} log
}

Application create app
app log "Application started"
```

### Variable Scoping

#### Encapsulation Example
```tcl
nx::Class create BankAccount {
    :property {balance 0}
    :property accountNumber
    
    :method init {accNum initialBalance} {
        set :accountNumber $accNum
        set :balance $initialBalance
    }
    
    # Public methods - external interface
    :public method deposit {amount} {
        if {[:validateAmount $amount]} {
            set :balance [expr {${:balance} + $amount}]
            :logTransaction "DEPOSIT" $amount
            puts "Deposited $amount. New balance: ${:balance}"
        }
    }
    
    :public method withdraw {amount} {
        if {[:validateAmount $amount] && [:hasSufficientFunds $amount]} {
            set :balance [expr {${:balance} - $amount}]
            :logTransaction "WITHDRAW" $amount
            puts "Withdrew $amount. New balance: ${:balance}"
        } else {
            puts "Insufficient funds or invalid amount"
        }
    }
    
    :public method getBalance {} {
        return ${:balance}
    }
    
    :public method getAccountNumber {} {
        return ${:accountNumber}
    }
    
    # Private methods - internal implementation
    :method validateAmount {amount} {
        if {$amount <= 0} {
            puts "Amount must be positive"
            return 0
        }
        return 1
    }
    
    :method hasSufficientFunds {amount} {
        return [expr {$amount <= ${:balance}}]
    }
    
    :method logTransaction {type amount} {
        puts "LOG: $type of $amount for account ${:accountNumber}"
    }
}

BankAccount create account1 "12345" 1000
account1 deposit 500
account1 withdraw 200
puts "Final balance: [account1 getBalance]"

# These would fail - private methods not accessible:
# account1 validateAmount 100        ;# Error!
# account1 hasSufficientFunds 50     ;# Error!
# account1 logTransaction "TEST" 10  ;# Error!
```

#### Class Variables
```tcl
nx::Class create IDGenerator {
    # Class variable shared by all instances
    :variable nextID 1
    
    :method init {} {
        set :id ${:nextID}
        incr :nextID
    }
    
    :method getID {} {
        return ${:id}
    }
    
    :class method getNextID {} {
        return ${:nextID}
    }
}

IDGenerator create obj1
IDGenerator create obj2
IDGenerator create obj3

puts "Object 1 ID: [obj1 getID]"
puts "Object 2 ID: [obj2 getID]"
puts "Object 3 ID: [obj3 getID]"
puts "Next available ID: [IDGenerator getNextID]"
```

## Inheritance

### Single Inheritance
```tcl
nx::Class create Animal {
    :property name
    :property species
    
    :method init {n s} {
        set :name $n
        set :species $s
    }
    
    :method speak {} {
        puts "${:name} makes a sound"
    }
    
    :method describe {} {
        puts "${:name} is a ${:species}"
    }
}

nx::Class create Dog -superclass Animal {
    :property breed
    
    :method init {n b} {
        # Call parent constructor
        next $n "Dog"
        set :breed $b
    }
    
    # Override parent method
    :method speak {} {
        puts "${:name} barks: Woof!"
    }
    
    :method fetch {} {
        puts "${:name} fetches the ball"
    }
}

Dog create buddy "Buddy" "Golden Retriever"
buddy describe
buddy speak
buddy fetch
```

### Multiple Inheritance
```tcl
nx::Class create Flyable {
    :method fly {} {
        puts "[self] is flying"
    }
}

nx::Class create Swimmable {
    :method swim {} {
        puts "[self] is swimming"
    }
}

nx::Class create Duck -superclass {Animal Flyable Swimmable} {
    :method init {name} {
        next $name "Duck"
    }
    
    :method speak {} {
        puts "${:name} quacks: Quack!"
    }
}

Duck create donald "Donald"
donald describe
donald speak
donald fly
donald swim
```

### Method Resolution and super/next
```tcl
nx::Class create Vehicle {
    :property {speed 0}
    
    :method start {} {
        puts "Vehicle started"
        set :speed 10
    }
    
    :method stop {} {
        puts "Vehicle stopped"
        set :speed 0
    }
}

nx::Class create Car -superclass Vehicle {
    :method start {} {
        puts "Starting car engine..."
        # Call parent method
        next
        puts "Car is ready to drive"
    }
    
    :method honk {} {
        puts "Beep beep!"
    }
}

Car create myCar
myCar start
myCar honk
myCar stop
```

## Mixins

Mixins provide a way to add functionality to classes or objects without inheritance.

### Class Mixins
```tcl
nx::Class create Timestamped {
    :method timestamp {} {
        return [clock seconds]
    }
    
    :method getCreationTime {} {
        if {![info exists :creationTime]} {
            set :creationTime [:timestamp]
        }
        return ${:creationTime}
    }
}

nx::Class create Serializable {
    :method serialize {} {
        set data {}
        foreach var [:info vars] {
            if {[info exists :$var]} {
                lappend data $var [set :$var]
            }
        }
        return $data
    }
    
    :method deserialize {data} {
        foreach {var value} $data {
            set :$var $value
        }
    }
}

nx::Class create Document {
    :property title
    :property content
    
    # Add mixins to the class
    :mixins add {Timestamped Serializable}
    
    :method init {t c} {
        set :title $t
        set :content $c
    }
}

Document create doc "My Document" "This is the content"
puts "Creation time: [doc getCreationTime]"
puts "Serialized: [doc serialize]"
```

### Object Mixins
```tcl
nx::Class create Debuggable {
    :method debug {message} {
        puts "DEBUG \[[self]\]: $message"
    }
}

nx::Class create Service {
    :property name
    
    :method init {n} {
        set :name $n
    }
    
    :method process {} {
        puts "Processing in ${:name}"
    }
}

Service create service1 "Database Service"
service1 process

# Add mixin to specific object
service1 mixins add Debuggable
service1 debug "Service initialized"
service1 process
```

## Filters and Guards

### Method Filters
Filters allow you to intercept method calls for logging, debugging, or modifying behavior.

```tcl
nx::Class create LoggingFilter {
    :method filter {args} {
        set method [lindex $args 0]
        puts "ENTER: [self] -> $method"
        set result [next {*}$args]
        puts "EXIT:  [self] <- $method"
        return $result
    }
}

nx::Class create BusinessService {
    :property name
    
    :method init {n} {
        set :name $n
    }
    
    :method processOrder {orderID} {
        puts "Processing order $orderID in ${:name}"
        return "Order $orderID processed"
    }
    
    :method cancelOrder {orderID} {
        puts "Cancelling order $orderID in ${:name}"
        return "Order $orderID cancelled"
    }
}

BusinessService create orderService "Order Service"

# Add filter to log all method calls
orderService filters add LoggingFilter

orderService processOrder "12345"
orderService cancelOrder "67890"
```

### Method Guards
Guards provide conditional execution of methods.

```tcl
nx::Class create AccessControl {
    :property {authenticated 0}
    
    :method login {username password} {
        # Simplified authentication
        if {$username eq "admin" && $password eq "secret"} {
            set :authenticated 1
            puts "Login successful"
        } else {
            puts "Login failed"
        }
    }
    
    :method logout {} {
        set :authenticated 0
        puts "Logged out"
    }
    
    # Guard method - only execute if authenticated
    :method sensitiveOperation {} -guard {${:authenticated}} {
        puts "Performing sensitive operation"
    }
    
    :method publicOperation {} {
        puts "Performing public operation"
    }
}

AccessControl create system
system publicOperation
system sensitiveOperation  ;# This will fail - not authenticated

system login "admin" "secret"
system sensitiveOperation  ;# This will succeed

system logout
system sensitiveOperation  ;# This will fail again
```

## Metaclasses

Metaclasses allow you to customize class creation and behavior.

```tcl
nx::Class create SingletonMeta -superclass nx::Class {
    :property instances {}
    
    :method create {name args} {
        if {[llength ${:instances}] > 0} {
            error "Singleton class can only have one instance"
        }
        set instance [next $name {*}$args]
        lappend :instances $instance
        return $instance
    }
}

SingletonMeta create DatabaseConnection {
    :property {host "localhost"}
    :property {port 5432}
    
    :method connect {} {
        puts "Connected to ${:host}:${:port}"
    }
}

# First instance creation succeeds
DatabaseConnection create db1
db1 connect

# Second instance creation fails
try {
    DatabaseConnection create db2
} on error {msg} {
    puts "Error: $msg"
}
```

## Advanced Features

### Introspection
```tcl
nx::Class create IntrospectionDemo {
    :property prop1 "value1"
    :property prop2 "value2"
    
    :method method1 {} {
        puts "Method 1"
    }
    
    :method method2 {arg} {
        puts "Method 2: $arg"
    }
}

IntrospectionDemo create demo

# Get class information
puts "Class: [demo info class]"
puts "Methods: [demo info methods]"
puts "Variables: [demo info vars]"

# Check if method exists
if {[demo info methods method1] ne ""} {
    puts "method1 exists"
}

# Get method parameters
puts "method2 parameters: [demo info method parameters method2]"
```

### Serialization
```tcl
nx::Class create SerializableClass {
    :property data
    :property timestamp
    
    :method init {d} {
        set :data $d
        set :timestamp [clock seconds]
    }
    
    :method getData {} {
        return ${:data}
    }
}

SerializableClass create obj "Important data"

# Serialize object
set serialized [obj serialize]
puts "Serialized: $serialized"

# Create new object and deserialize
SerializableClass create newObj
newObj deserialize $serialized
puts "Deserialized data: [newObj getData]"
```

### Namespaced Classes
```tcl
namespace eval ::myapp::model {
    nx::Class create User {
        :property username
        :property email
        
        :method init {u e} {
            set :username $u
            set :email $e
        }
        
        :method getInfo {} {
            return "${:username} <${:email}>"
        }
    }
}

namespace eval ::myapp::controller {
    nx::Class create UserController {
        :method createUser {username email} {
            return [::myapp::model::User new $username $email]
        }
    }
}

::myapp::controller::UserController create controller
set user [controller createUser "john_doe" "john@example.com"]
puts [user getInfo]
```

## Best Practices

### 1. Use Public Methods for External Interface
```tcl
nx::Class create Person {
    :property name
    :property age
    :property email
    
    # Public interface
    :public method getName {} { return ${:name} }
    :public method setName {n} { 
        if {[:validateName $n]} {
            set :name $n
        }
    }
    
    :public method getAge {} { return ${:age} }
    
    # Private validation
    :method validateName {name} {
        return [expr {[string length $name] > 0}]
    }
}
```

### 2. Initialize Objects Properly
```tcl
nx::Class create BankAccount {
    :property accountNumber
    :property {balance 0}
    
    :method init {accNum initialBalance} {
        if {$initialBalance < 0} {
            error "Initial balance cannot be negative"
        }
        set :accountNumber $accNum
        set :balance $initialBalance
    }
}
```

### 3. Use Mixins for Cross-Cutting Concerns
```tcl
nx::Class create Auditable {
    :method audit {action details} {
        puts "\[AUDIT\] [clock format [clock seconds]]: $action - $details"
    }
}

nx::Class create Customer {
    :mixins add Auditable
    :property name
    
    :method updateName {newName} {
        set oldName ${:name}
        set :name $newName
        :audit "Name Change" "From '$oldName' to '$newName'"
    }
}
```

### 4. Use Guards for Conditional Behavior
```tcl
nx::Class create FileProcessor {
    :property {fileExists 0}
    
    :method setFile {filename} {
        set :filename $filename
        set :fileExists [file exists $filename]
    }
    
    :method process {} -guard {${:fileExists}} {
        puts "Processing ${:filename}"
    }
}
```

## Complete Examples

### Example 1: Simple Web Server Framework
```tcl
package require nx

nx::Class create HTTPRequest {
    :property method
    :property path
    :property headers
    :property body
    
    :method init {m p} {
        set :method $m
        set :path $p
        set :headers [dict create]
        set :body ""
    }
    
    :method addHeader {name value} {
        dict set :headers $name $value
    }
    
    :method getHeader {name} {
        if {[dict exists ${:headers} $name]} {
            return [dict get ${:headers} $name]
        }
        return ""
    }
}

nx::Class create HTTPResponse {
    :property {status 200}
    :property {headers {}}
    :property {body ""}
    
    :method setStatus {code} {
        set :status $code
    }
    
    :method addHeader {name value} {
        dict set :headers $name $value
    }
    
    :method setBody {content} {
        set :body $content
    }
    
    :method toString {} {
        set response "HTTP/1.1 ${:status} OK\n"
        dict for {name value} ${:headers} {
            append response "$name: $value\n"
        }
        append response "\n${:body}"
        return $response
    }
}

nx::Class create Route {
    :property pattern
    :property handler
    
    :method init {p h} {
        set :pattern $p
        set :handler $h
    }
    
    :method matches {path} {
        return [string match ${:pattern} $path]
    }
    
    :method handle {request response} {
        {*}${:handler} $request $response
    }
}

nx::Class create WebServer {
    :property {routes {}}
    :property {port 8080}
    
    :method addRoute {pattern handler} {
        lappend :routes [Route new $pattern $handler]
    }
    
    :method handleRequest {method path} {
        set request [HTTPRequest new $method $path]
        set response [HTTPResponse new]
        
        foreach route ${:routes} {
            if {[$route matches $path]} {
                $route handle $request $response
                return [$response toString]
            }
        }
        
        $response setStatus 404
        $response setBody "Not Found"
        return [$response toString]
    }
}

# Usage example
proc homeHandler {request response} {
    $response addHeader "Content-Type" "text/html"
    $response setBody "<h1>Welcome to NX Web Server</h1>"
}

proc apiHandler {request response} {
    $response addHeader "Content-Type" "application/json"
    $response setBody {{"message": "Hello from API"}}
}

WebServer create server
server addRoute "/" homeHandler
server addRoute "/api/*" apiHandler

puts [server handleRequest "GET" "/"]
puts "---"
puts [server handleRequest "GET" "/api/users"]
```

### Example 2: Event System
```tcl
package require nx

nx::Class create EventEmitter {
    :property listeners
    
    :method init {} {
        set :listeners [dict create]
    }
    
    :method on {event callback} {
        if {![dict exists ${:listeners} $event]} {
            dict set :listeners $event {}
        }
        dict lappend :listeners $event $callback
    }
    
    :method emit {event args} {
        if {[dict exists ${:listeners} $event]} {
            foreach callback [dict get ${:listeners} $event] {
                {*}$callback {*}$args
            }
        }
    }
    
    :method off {event {callback ""}} {
        if {[dict exists ${:listeners} $event]} {
            if {$callback eq ""} {
                dict unset :listeners $event
            } else {
                set callbacks [dict get ${:listeners} $event]
                set index [lsearch $callbacks $callback]
                if {$index >= 0} {
                    set callbacks [lreplace $callbacks $index $index]
                    dict set :listeners $event $callbacks
                }
            }
        }
    }
}

nx::Class create Timer -superclass EventEmitter {
    :property {interval 1000}
    :property {running 0}
    
    :method start {} {
        if {!${:running}} {
            set :running 1
            :emit "start"
            :tick
        }
    }
    
    :method stop {} {
        if {${:running}} {
            set :running 0
            :emit "stop"
        }
    }
    
    :method tick {} {
        if {${:running}} {
            :emit "tick" [clock seconds]
            after ${:interval} [list [self] tick]
        }
    }
}

# Usage
proc onTick {timestamp} {
    puts "Tick at [clock format $timestamp]"
}

proc onStart {} {
    puts "Timer started"
}

proc onStop {} {
    puts "Timer stopped"
}

Timer create timer -interval 2000
timer on "tick" onTick
timer on "start" onStart
timer on "stop" onStop

timer start
after 6000 [list timer stop]

# Keep the event loop running
vwait forever
```

### Example 3: Database ORM Pattern
```tcl
package require nx

nx::Class create DatabaseConnection {
    :property host
    :property database
    :property {connected 0}
    
    :method init {h d} {
        set :host $h
        set :database $d
    }
    
    :method connect {} {
        # Simulate database connection
        set :connected 1
        puts "Connected to ${:database} at ${:host}"
    }
    
    :method execute {query} {
        if {!${:connected}} {
            error "Not connected to database"
        }
        puts "Executing: $query"
        # Return simulated result
        return "Query executed successfully"
    }
    
    :method disconnect {} {
        set :connected 0
        puts "Disconnected from database"
    }
}

nx::Class create Model {
    :property tableName
    :property attributes
    :property connection
    
    :method init {table conn} {
        set :tableName $table
        set :attributes [dict create]
        set :connection $conn
    }
    
    :method set {attr value} {
        dict set :attributes $attr $value
    }
    
    :method get {attr} {
        if {[dict exists ${:attributes} $attr]} {
            return [dict get ${:attributes} $attr]
        }
        return ""
    }
    
    :method save {} {
        set fields {}
        set values {}
        dict for {attr value} ${:attributes} {
            lappend fields $attr
            lappend values '$value'
        }
        set query "INSERT INTO ${:tableName} ([join $fields ,]) VALUES ([join $values ,])"
        return [${:connection} execute $query]
    }
    
    :method find {id} {
        set query "SELECT * FROM ${:tableName} WHERE id = $id"
        return [${:connection} execute $query]
    }
}

nx::Class create User -superclass Model {
    :method init {conn} {
        next "users" $conn
    }
    
    :method setName {name} {
        :set "name" $name
    }
    
    :method setEmail {email} {
        :set "email" $email
    }
    
    :method getName {} {
        return [:get "name"]
    }
    
    :method getEmail {} {
        return [:get "email"]
    }
}

# Usage
DatabaseConnection create db "localhost" "myapp"
db connect

User create user1 $db
user1 setName "John Doe"
user1 setEmail "john@example.com"
user1 save

puts "User: [user1 getName] <[user1 getEmail]>"

db disconnect
```

This comprehensive guide covers all major aspects of NX programming, from basic concepts to advanced features. The examples demonstrate real-world usage patterns and best practices for building object-oriented applications with NX.