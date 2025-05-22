# TCL Bytecode Reference

This document provides a comprehensive reference for TCL bytecode, including instruction set, examples, and practical usage.

## Table of Contents
1. [Introduction](#introduction)
2. [Bytecode Basics](#bytecode-basics)
3. [Instruction Set](#instruction-set)
4. [Bytecode Examples](#bytecode-examples)
5. [Working with Bytecode](#working-with-bytecode)
6. [Advanced Topics](#advanced-topics)
7. [Tools and Utilities](#tools-and-utilities)
8. [References](#references)

## Introduction

TCL (Tool Command Language) compiles scripts into bytecode before execution. This bytecode runs on a stack-based virtual machine. Understanding TCL bytecode can help with debugging, optimization, and creating custom tools.

## TCL Assembly Language Tutorial

TCL bytecode is the low-level representation of TCL code that runs on the TCL Virtual Machine (TVM). This section provides a hands-on tutorial for working with TCL assembly.

### Basic Concepts

1. **Stack-Based VM**: TCL uses a stack-based virtual machine where most operations work by pushing and popping values from a stack.
2. **Registers**: While primarily stack-based, TCL bytecode does use some virtual registers (like %v0, %v1) for local variables.
3. **Instruction Format**: Most instructions consist of an opcode followed by zero or more operands.

### Getting Started with TCL Assembly

Let's start with a simple example. Here's how to create and execute a simple TCL procedure using bytecode:

```tcl
# This is equivalent to: proc double x {expr {$x * 2}}
set bytecode {
    # Procedure header: name, numArgs, numLocals
    proc double 1 0
    
    # Procedure body
    load %v0    # Load first argument (x)
    push1 0     # Push constant 2 (index 0 in constants table)
    mult        # Multiply top two stack items
    return      # Return the result
}

# Define the constants table
set ::tcl::unsupported::constants(double) {2}

# Create the procedure
proc double x [list bytecode [tcl::unsupported::assemble $bytecode]]

# Test it
puts [double 5]  ;# Outputs: 10
```

### Detailed Opcode Reference

#### Stack Manipulation

| Opcode | Operands | Description | Example |
|--------|----------|-------------|---------|
| push1  | uint1    | Push 1-byte constant index | `push1 0` |
| push4  | uint4    | Push 4-byte constant index | `push4 1000` |
| dup    | -        | Duplicate top stack item | `dup` |
| pop    | -        | Remove top stack item | `pop` |
| load   | uint1    | Load local variable | `load %v0` |
| store  | uint1    | Store to local variable | `store %v0` |
| loadStk | -       | Load from stack | `loadStk` |
| storeStk | -      | Store to stack | `storeStk` |


#### Arithmetic Operations

| Opcode | Operands | Description | Example |
|--------|----------|-------------|---------|
| add    | -        | Add top two stack items | `add` |
| sub    | -        | Subtract | `sub` |
| mult   | -        | Multiply | `mult` |
| div    | -        | Divide | `div` |
| mod    | -        | Modulo | `mod` |
| incr   | int1     | Increment variable by 1 | `incr %v0` |
| incr1  | -        | Increment top stack item by 1 | `incr1` |


#### Control Flow

| Opcode | Operands | Description | Example |
|--------|----------|-------------|---------|
| jump   | int1/int4 | Unconditional jump | `jump +10` |
| jumpTrue | int1/int4 | Jump if true | `jumpTrue +5` |
| jumpFalse | int1/int4 | Jump if false | `jumpFalse +5` |
| invokeStk | int1   | Call command | `invokeStk1 2` (2 args) |
| return | -        | Return from procedure | `return` |
| done   | -        | End of bytecode | `done` |


#### String Operations

| Opcode | Operands | Description | Example |
|--------|----------|-------------|---------|
| strcat | int1     | Concatenate strings | `strcat 3` (3 items) |
| strcmp | -        | Compare strings | `strcmp` |
| strlen | -        | Get string length | `strlen` |
| strindex | -      | Get character at index | `strindex` |
| strrange | -      | Get substring | `strrange` |


### Step-by-Step Example: Factorial Function

Let's implement a factorial function in TCL assembly:

```tcl
# Factorial function in TCL assembly
# Equivalent to: proc fact n {expr {$n <= 1 ? 1 : $n * [fact [expr {$n-1}]]}}


set bytecode {
    # Procedure header: name, numArgs, numLocals
    proc fact 1 1
    
    # if {$n <= 1} {return 1}
    load %v0        # Load n
    push1 0          # Push constant 1 (index 0 in constants)
    le              # Less than or equal
    jumpFalse1 +10   # Jump to recursive case if false
    
    # Base case: return 1
    push1 0          # Push constant 1
    return           # Return it
    
    # Recursive case: return $n * [fact [expr {$n-1}]]
    load %v0        # Load n
    load %v0        # Load n again
    push1 0          # Push constant 1
    sub              # n-1
    
    # Call fact(n-1)
    push1 1          # Push command "fact" (index 1 in constants)
    invokeStk1 1     # Call with 1 argument
    
    # Multiply n * fact(n-1)
    mult
    return
}

# Define constants: [1, "fact"]
set ::tcl::unsupported::constants(fact) {1 fact}

# Create the procedure
proc fact n [list bytecode [tcl::unsupported::assemble $bytecode]]

# Test it
puts [fact 5]  ;# Outputs: 120
```

### Debugging TCL Bytecode

To debug TCL bytecode, you can use the `tcl::unsupported::disassemble` command:

```tcl
# Disassemble a simple procedure
puts [tcl::unsupported::disassemble {
    proc test {a b} {
        set c [expr {$a + $b}]
        return [expr {$c * 2}]
    }
}]
```

This will show you the bytecode generated for the procedure, which can help you understand how TCL compiles your code.

### Common Patterns

1. **Conditional Execution**:
   ```
   # if {$x > 0} {set y 10} else {set y 20}
   load %v0        # Load x
   push1 0          # Push 0
   gt               # Compare
   jumpFalse1 +5    # Jump to else branch if false
   push1 1          # Push 10 (true branch)
   jump1 +3         # Jump past else branch
   push1 2          # Push 20 (false branch)
   store %v1        # Store to y
   ```

2. **Loop Pattern**:
   ```
   # for {set i 0} {$i < 10} {incr i} {puts $i}
   push1 0          # 0 (initial value of i)
   store %v0        # Store in i
   # Start of loop condition
   load %v0         # Load i
   push1 1          # Push 10
   lt               # Compare i < 10
   jumpFalse1 +10    # Exit loop if false
   # Loop body
   push1 2          # Push "puts"
   load %v0         # Load i
   invokeStk1 2      # Call puts with 1 argument
   # Loop increment
   incr %v0 1       # Increment i
   jump1 -12         # Jump back to condition
   ```

## Bytecode Basics

### Bytecode Structure

TCL bytecode consists of:
- **Header**: Metadata about the bytecode
- **Instruction Stream**: Sequence of opcodes and operands
- **Constant Pool**: Literal values used in the code
- **Local Variables**: Storage for procedure variables
- **Exception Ranges**: For error handling

### Viewing Bytecode

Use `tcl::unsupported::disassemble` to view bytecode:

```tcl
puts [tcl::unsupported::disassemble {
    proc add {a b} {
        return [expr {$a + $b}]
    }
}]
```

## Instruction Set

### Stack Manipulation

| Opcode | Example | Description |
|--------|---------|-------------|
| push1  | `push1 0` | Push 1-byte constant |
| push4  | `push4 1000` | Push 4-byte constant |
| load   | `load %v0` | Load variable |
| store  | `store %v0` | Store to variable |
| dup    | `dup` | Duplicate top stack item |
| pop    | `pop` | Remove top stack item |

### Arithmetic Operations

| Opcode | Example | Description |
|--------|---------|-------------|
| add    | `add` | Add top two stack items |
| sub    | `sub` | Subtract |
| mult   | `mult` | Multiply |
| div    | `div` | Divide |
| mod    | `mod` | Modulo |
| incr   | `incr %v0` | Increment variable |
| incrStk | `incrStk` | Increment stack value |

### Control Flow

| Opcode | Example | Description |
|--------|---------|-------------|
| jump   | `jump +10` | Unconditional jump |
| jumpTrue | `jumpTrue +5` | Jump if true |
| jumpFalse | `jumpFalse +5` | Jump if false |
| invokeStk | `invokeStk1 2` | Call command with 2 args |
| return | `return` | Return from procedure |
| done   | `done` | End of bytecode |

### String Operations

| Opcode | Example | Description |
|--------|---------|-------------|
| strcat | `strcat 2` | Concatenate strings |
| strcmp | `strcmp` | Compare strings |
| strlen | `strlen` | Get string length |
| strindex | `strindex` | Get character at index |
| strrange | `strrange` | Get substring |

## Bytecode Examples

### Simple Arithmetic

```tcl
# Source: set x [expr {10 + 20}]
ByteCode 0x00000000026F0E10:
  Source "set x [expr {10 + 20}]"
  Cmds 1, src 24, inst 10, litObjs 1, aux 0, stkDepth 2, code/src 0.00
  Commands 1:
      1: pc 0-8, src 0-23
  Command 1: "set x [expr {10 + 20}]"
    (0) push1 0     # "10"
    (2) push1 1     # "20"
    (4) add 
    (5) store %v0   # var "x"
    (7) push1 2     # ""
    (9) done 
```

### If-Else Statement

```tcl
# Source:
# if {$x > 0} {
#     set result "Positive"
# } else {
#     set result "Non-positive"
# }
ByteCode 0x00000000026F1E10:
  Source "if {$x > 0} {\n    set result \"Positive\"\n} else {\n    set result \"Non-positive\"\n}"
  Cmds 3, src 85, inst 26, litObjs 4, aux 0, stkDepth 3, code/src 0.00
  Commands 3:
      1: pc 0-24, src 0-84
      2: pc 0-7, src 13-35
      3: pc 9-16, src 44-84
  Command 1: "if {$x > 0} {\n    set result \"Positive\"\n} else {\n    set result \"Non-positive\"\n}"
    (0) load %v0    # var "x"
    (2) push1 0     # "0"
    (4) gt 
    (5) jumpFalse1 +9  # pc 14
  Command 2: "set result \"Positive\""
    (7) push1 1     # "Positive"
    (9) store %v1   # var "result"
    (11) jump1 +8   # pc 19
  Command 3: "set result \"Non-positive\""
    (14) push1 2    # "Non-positive"
    (16) store %v1  # var "result"
    (18) nop 
    (19) push1 3    # ""
    (21) done 
```

### For Loop

```tcl
# Source:
# for {set i 0} {$i < 10} {incr i} {
#     puts "i = $i"
# }
ByteCode 0x00000000026F2E10:
  Source "for {set i 0} {$i < 10} {incr i} {\n    puts \"i = $i\"\n}"
  Cmds 4, src 48, inst 28, litObjs 6, aux 0, stkDepth 4, code/src 0.00
  Commands 4:
      1: pc 0-26, src 0-47
      2: pc 0-2, src 5-12
      3: pc 4-6, src 13-22
      4: pc 8-13, src 23-46
  Command 1: "for {set i 0} {$i < 10} {incr i} {\n    puts \"i = $i\"\n}"
  Command 2: "set i 0"
    (0) push1 0     # "0"
    (2) store %v0   # var "i"
  Command 3: "$i < 10"
    (4) load %v0    # var "i"
    (6) push1 1     # "10"
    (8) lt 
    (9) jumpFalse1 +12  # pc 21
  Command 4: "puts \"i = $i\""
    (11) push1 2    # "i = "
    (13) load %v0   # var "i"
    (15) strcat 2 
    (17) push1 3    # "puts"
    (19) invokeStk1 2 
    (21) jump1 -18  # pc 4
    (23) push1 4    # ""
    (25) done 
```

## Working with Bytecode

### Generating Bytecode

```tcl
# Generate bytecode from Tcl code
set bytecode [tcl::unsupported::disassemble {
    proc square {x} {
        return [expr {$x * $x}]
    }
}]
puts $bytecode
```

### Executing Bytecode

```tcl
# Create a procedure from bytecode
proc run_bytecode {bytecode} {
    set cmd [tcl::unsupported::assemble $bytecode]
    uplevel #0 $cmd
}

# Example bytecode for: set x 42
set bc {
    push1 0     # "42"
    store %v0   # var "x"
    push1 1     # ""
    return
}

run_bytecode $bc
puts "x = $x"  ;# Outputs: x = 42
```

## Advanced Topics

### Custom Bytecode Generation

You can generate custom bytecode for optimization:

```tcl
# This is equivalent to: proc double x {expr {$x * 2}}
set bytecode {
    # Procedure header
    proc double 1 0
    
    # Body
    load %v0    # Load argument x
    push1 0     # Push constant 2
    mult        # Multiply
    return      # Return result
}

# Create the procedure
proc double x [list bytecode [tcl::unsupported::assemble $bytecode]]

# Test it
puts [double 5]  ;# Outputs: 10
```

### Bytecode Optimization

TCL performs several optimizations when generating bytecode:

1. Constant folding
2. Dead code elimination
3. Inlining of simple procedures
4. Common subexpression elimination

## Tools and Utilities

### tcl::unsupported::disassemble

Disassembles Tcl code into bytecode:

```tcl
puts [tcl::unsupported::disassemble {set x [expr {1 + 2 * 3}]}]
```

### tcl::unsupported::getbytecode

Gets the raw bytecode for a command:

```tcl
proc test {} { return "Hello" }
set bc [tcl::unsupported::getbytecode test]
binary scan $bc H* hex
puts $hex
```

### tcl::unsupported::assemble

Assembles bytecode into executable code:

```tcl
# Bytecode to set x to 42
set bc {
    push1 0     # "42"
    store %v0   # var "x"
    push1 1     # ""
    return
}
set cmd [tcl::unsupported::assemble $bc]
eval $cmd
puts $x  ;# Outputs: 42
```

## References

1. [Tcl Compile API](https://www.tcl.tk/man/tcl/CompileProc.html)
2. [Tcl Bytecode Engine](https://www.tcl.tk/man/tcl/ByteCompilation.html)
3. [Tcl Source Code](https://core.tcl-lang.org/tcl/)
4. [Tclers Wiki - Bytecode Engineering](https://wiki.tcl-lang.org/page/Bytecode+Engineering)

## License

This document is provided as-is under the MIT License. Use at your own risk.
