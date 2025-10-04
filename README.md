# Stack Virtual Machine

A minimal stack-based virtual machine with its own assembler, demonstrating fundamental computer architecture concepts.

## Overview

This project consists of two components:

1. **Stack VM** - A simple virtual machine that executes bytecode using a stack-based architecture
2. **SASM** (Stack Assembler) - An assembler that compiles assembly-like source code into VM bytecode

## Architecture

### Stack VM

The virtual machine implements a classic fetch-decode-execute cycle:

- **Memory**: Unified memory space for code and data
- **Program Counter (PC)**: Points to the current instruction (starts at 100)
- **Stack Pointer (SP)**: Points to the top of the stack
- **Instruction Format**: 32-bit instructions
  - Bits 30-31: Type header (00 = positive integer, 01 = primitive operation, 10 = negative integer)
  - Bits 0-29: Data payload

### Instruction Set

| Instruction | Opcode | Description |
|------------|---------|-------------|
| `+` | 0x40000001 | Add two top stack values |
| `-` | 0x40000002 | Subtract top from second |
| `*` | 0x40000003 | Multiply two top values |
| `/` | 0x40000004 | Divide second by top |
| `halt` | 0x40000000 | Stop execution |

### SASM Assembler

The assembler performs two phases:

1. **Lexical Analysis** (lexer.cpp)
   - Tokenizes source code
   - Handles whitespace, comments (`//`), strings, and special characters
   - Uses a state machine for parsing

2. **Code Generation** (sasm.cpp)
   - Converts tokens to binary instructions
   - Maps operators to opcodes
   - Outputs bytecode to `out.bin`

## Building

### Prerequisites

- C++ compiler with C++11 support (g++, clang++)
- Make

### Build Instructions

```bash
# Build the assembler
cd sasm
make

# Build the virtual machine
cd ../stack-vm
make
```

Clean build artifacts:
```bash
make clean  # in either directory
```

## Usage

### Complete Workflow

1. **Write assembly code** (`.sasm` file):
```
3 4 + 2 * 2 + 4 /
```

2. **Compile to bytecode**:
```bash
./sasm/sasm myprogram.sasm
```
This generates `out.bin` in the current directory.

3. **Run on the VM**:
```bash
./stack-vm/stack-vm out.bin
```

### Example

```bash
# Using the included test file
cd /home/stack-vm
./sasm/sasm sasm/test.sasm
./stack-vm/stack-vm out.bin
```

**test.sasm content:**
```
3 4 + 2 * 2 + 4 /
```

**Execution trace:**
```
add 3 4
tos: 7
mul 7 2
tos: 14
add 14 2
tos: 16
div 16 4
tos: 4
```

**Result:** 4 (which is (3 + 4) * 2 + 2 / 4 = 16 / 4 = 4)

## Execution Flow

```
┌─────────────┐
│  Source     │
│  .sasm file │
└──────┬──────┘
       │
       ├─────────────────────────────┐
       │  SASM Assembler             │
       │                             │
       │  ┌──────────┐               │
       │  │  Lexer   │               │
       │  │  Parse   │               │
       │  │  tokens  │               │
       │  └────┬─────┘               │
       │       │                     │
       │  ┌────▼─────────────┐       │
       │  │  Compiler        │       │
       │  │  Map to opcodes  │       │
       │  └────┬─────────────┘       │
       │       │                     │
       └───────┼─────────────────────┘
               │
       ┌───────▼──────┐
       │  out.bin     │
       │  (bytecode)  │
       └───────┬──────┘
               │
       ┌───────▼──────────────────┐
       │  Stack VM                │
       │                          │
       │  ┌──────────┐            │
       │  │  Load    │            │
       │  │  Program │            │
       │  └────┬─────┘            │
       │       │                  │
       │  ┌────▼─────┐            │
       │  │  Fetch   │◄───┐       │
       │  └────┬─────┘    │       │
       │       │          │       │
       │  ┌────▼─────┐    │       │
       │  │  Decode  │    │       │
       │  └────┬─────┘    │       │
       │       │          │       │
       │  ┌────▼─────┐    │       │
       │  │ Execute  │────┘       │
       │  └──────────┘            │
       │                          │
       └──────────────────────────┘
```

## How It Works

### 1. Assembly Phase

**Input**: `3 4 + 2 *`

**Lexer** breaks it into tokens:
```
["3", "4", "+", "2", "*"]
```

**Compiler** converts to instructions:
```
[3, 4, 0x40000001, 2, 0x40000003]
```

**Output**: Binary file with 5×4 bytes = 20 bytes

### 2. Execution Phase

VM loads bytecode and executes:

| Step | Instruction | Stack State | Action |
|------|------------|-------------|--------|
| 1 | 3 | [3] | Push 3 |
| 2 | 4 | [3, 4] | Push 4 |
| 3 | + | [7] | Pop 4,3; Push 7 |
| 4 | 2 | [7, 2] | Push 2 |
| 5 | * | [14] | Pop 2,7; Push 14 |

## File Structure

```
stack-vm/
├── sasm/
│   ├── lexer.h          - Lexer class definition
│   ├── lexer.cpp        - Tokenization logic
│   ├── sasm.cpp         - Main assembler & compiler
│   ├── makefile         - Build configuration
│   └── test.sasm        - Example program
│
└── stack-vm/
    ├── stack-vm.h       - VM class definition
    ├── stack-vm.cpp     - VM implementation
    ├── main.cpp         - VM entry point
    └── makefile         - Build configuration
```

## Educational Value

This project demonstrates:

- **Compiler Design**: Lexical analysis, tokenization, code generation
- **Computer Architecture**: Fetch-decode-execute cycle, stack-based computing
- **Binary I/O**: Reading/writing machine code
- **State Machines**: Lexer implementation
- **Virtual Machines**: Software-based instruction execution

## Limitations

- No labels or jump instructions
- Limited error handling
- No negative number support in assembly syntax
- Fixed instruction set (5 operations only)
- No system calls or I/O operations

## Future Enhancements

- Add control flow (jumps, conditionals)
- Implement labels and symbolic addresses
- Add more operations (comparison, logical)
- Memory access instructions
- Better error reporting
- Debugger interface

## License

Educational project - free to use and modify.

