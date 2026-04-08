# Table of Contents# Table of Contents
- [Overview](#overview)
- [The Full Compilation Pipeline](#the-full-compilation-pipeline)
- [Setting Up a Simple Project](#setting-up-a-simple-project)
- [Stage 1 Deep Dive: The Preprocessor](#stage-1-deep-dive-the-preprocessor)
- [Stage 2 Deep Dive: The Compiler](#stage-2-deep-dive-the-compiler)
- [Stage 3 Deep Dive: Assembly and Object Files](#stage-3-deep-dive-assembly-and-object-files)
- [Stage 4 Deep Dive: The Linker](#stage-4-deep-dive-the-linker)
- [Working with External Libraries: Compiler Flags](#working-with-external-libraries-compiler-flags)
- [Practical Example: The SFML Graphics Library](#practical-example-the-sfml-graphics-library)
- [Useful Tools Reference](#useful-tools-reference)
## Overview

When you write a C or C++ program and hit "compile," it might feel like magic, you give the compiler a text file and you get back a runnable program. But this apparent simplicity hides a multi-stage pipeline, each stage with its own job, its own output, and its own set of errors that can go wrong.

Understanding this pipeline is not just an academic exercise. When something goes wrong - and it will - knowing which stage produced the error tells you exactly where to look.
A **syntax error** comes from the compiler stage. An **undefined reference** comes from the linker stage. Treating these the same way wastes a lot of time.

The concepts covered here apply broadly. Although the discussion is anchored in C and C++, the same fundamental ideas govern Java (compiling to bytecode and linking JAR files), compiled languages like D and Pascal, and to varying degrees even scripting languages. The specific tool names change; the pipeline logic does not.

---
## The Full Compilation Pipeline

Here is the complete pipeline at a glance:
![Image Description: A horizontal flowchart showing 5 boxes connected by arrows from left to right: (1) source.cpp — the plain text source file, (2) Preprocessor — outputs source.i after handling #include, #define, #if, (3) Compiler — outputs source.s assembly after parsing and code generation, (4) Assembler — outputs source.o binary object file, (5) Linker (Static + Dynamic) — combines all .o files and shared libraries into the final executable. Each box should show the file extension produced below it.](./attachments/image1.png)
### Stage 0 - The Source File

This is just a plain text file you write. In C++ it typically ends in `.cpp`; in C it ends in `.c`. There is nothing special about it from the computer's perspective - it is just characters. All the real work begins in Stage 1.
### Stage 1 - The Preprocessor

The preprocessor handles every line in your source file that begins with the `#` (pound / hash) symbol. These are called **preprocessor directives**. The three most common are:

| Directive                               | Purpose                                                                       |
| --------------------------------------- | ----------------------------------------------------------------------------- |
| `#include`                              | Copy-pastes another file's contents into the current file                     |
| `#define`                               | Creates a symbolic constant or macro                                          |
| `#if` / `#ifdef` / `#ifndef` / `#endif` | Conditional compilation - includes or excludes blocks of code at compile time |

The preprocessor produces a file conventionally called `source.i`. This file is still plain text - it is an expanded version of your source with all directives resolved.
### Stage 2 - The Compiler

The compiler takes `source.i` (the preprocessed text) and transforms it into **assembly code**, producing `source.s`. Internally, to do this it:
1. Parses your code into an internal tree structure (an Abstract Syntax Tree or Concrete Syntax Tree)
2. Walks that tree to detect syntax errors and validate the grammar of the language
3. Generates assembly instructions corresponding to the operations the tree represents

Assembly is a **textual representation of machine code** - it is still human-readable (with some effort), but it is very close to the actual binary instructions your CPU will execute.
### Stage 3 - The Assembler

The assembler (for example, the GAS assembler bundled with GCC/G++) takes `source.s` and converts it into actual **machine code** - the ones and zeros that your CPU directly executes. The output is an **object file**, `source.o`.

An object file is a binary blob. It is not yet a complete runnable program; it may reference functions or variables that are defined elsewhere and that have not been resolved yet.
### Stage 4 - The Linker

The linker's job is to take one or more `.o` object files and **glue them together** into a single executable. It also brings in external library code. There are two flavors of linking:

**Static linking** resolves references by copying the necessary library code directly into the executable at build time. The result is a self-contained binary. Library files involved here are typically `.a` files (archives) on Linux/Mac, or `.lib` files on Windows.

**Dynamic linking** defers resolution until runtime. The executable records *which* shared library it needs and which symbols it expects from it, but the actual library code stays in a separate file. When the program starts, the operating system's dynamic linker loads the required shared libraries into memory. These files are:

- `.so` — Shared Object (Linux)
- `.dll` — Dynamic Link Library (Windows)
- `.dylib` — Dynamic Library (macOS)
### The Final Executable

After the linker finishes, you have your final program:

| OS      | How you name it              | How you run it             |
| ------- | ---------------------------- | -------------------------- |
| Linux   | `prog` (no extension needed) | `./prog`                   |
| Windows | `prog.exe`                   | `prog.exe` or double-click |
| macOS   | `prog` or `prog.app`         | `./prog` or double-click   |

---
## Setting Up a Simple Project

Before going deep into each stage, it helps to have a concrete project to experiment with. The project used throughout this guide is minimal on purpose - it is small enough to understand completely, but structured enough to show all the interesting compilation mechanics.
### Project File Structure

Three files are used:
```
project/
├── source.cpp    ← the implementation (the actual add function)
├── source.hpp    ← the header / interface (declaration of add)
└── main.cpp      ← the entry point (calls add, uses cout)
```

This is a typical real-world layout. The `.hpp` (header) file declares **what** functions exist. The `.cpp` (source) file defines **how** those functions work. The `main.cpp` file ties everything together.
### Writing the Files

`source.hpp` - the header file, just a declaration:
```cpp
// source.hpp
// Declares the add function. Any file that includes this
// knows the function exists and what its signature looks like.

int add(int a, int b);
```

`source.cpp` - the implementation:
```cpp
// source.cpp
// Defines how add actually works.

#include "source.hpp"

int add(int a, int b) {
    return a + b;
}
```

`main.cpp` - the entry point:
```cpp
// main.cpp
// The program starts here. We include source.hpp so the
// compiler knows about the add function.

#include "source.hpp"
#include <iostream>

int main() {
    std::cout << add(2, 3) << std::endl;
    return 0;
}
```

### Compiling Everything in One Shot

The simplest way to compile this project is to give both `.cpp` files to the compiler together:
```bash
g++ main.cpp source.cpp -o prog
```

You can also use a wildcard (though being explicit about file names is generally better practice because you know exactly what is being compiled):
```bash
# Wildcard approach - convenient but less explicit
g++ *.cpp -o prog
```

Running the program:
```bash
./prog
# Output:
# 5
```
This works, but the interesting question is: *what exactly happened between writing the source and getting that executable?*

---
## Stage 1 Deep Dive: The Preprocessor

### What the Preprocessor Actually Does

The most important thing to understand about the preprocessor is that `#include` is literally a **copy-paste** operation. When the preprocessor sees:
```cpp
#include "source.hpp"
```

it opens `source.hpp`, reads its entire contents, and inserts those contents at exactly that line in the file being processed. The `#include` directive itself disappears. What replaces it is the full text of the included file.

This is not a link, not a reference, not a pointer - it is a textual substitution.
### Inspecting Preprocessor Output with `-E`

You can ask `g++` to stop after the preprocessing stage and print the result to the terminal using the `-E` flag:
```bash
g++ -E main.cpp
```

The output will be a large dump of text. Look through it and you will see the contents of `source.hpp` pasted in exactly where the `#include "source.hpp"` line used to be. You will see the declaration:
```cpp
int add(int a, int b);
```
sitting right there in the middle of what used to be `main.cpp`.

If you also include `<iostream>`:
```bash
g++ -E main.cpp   # with #include <iostream> in main.cpp
```
the output becomes huge - thousands of lines - because the entire standard library header gets copied in. This is normal. It is also a hint about why large C++ projects can be slow to compile: every `.cpp` file that includes heavy headers has to process all that text.
### 4.3 The Danger of Including Files Twice

Because `#include` is a copy-paste, including the same header twice gives you two copies of everything in that header. Consider what happens when both `main.cpp` and `source.cpp` include `source.hpp`, and `source.hpp` contains a function *definition* (not just a declaration):

```cpp
// source.hpp — DANGEROUS if it contains a definition
int add(int a, int b) {   // ← full definition inside a header
    return a + b;
}
```

When the linker combines `main.o` and `source.o`, it finds *two definitions* of `add` - one copied into each translation unit from the header. The result is:
```
error: redefinition of 'int add(int a, int b)'
```

This is one of the most common errors beginners see in C++ projects. The rule is:
> **Headers should contain declarations (the function signature), not definitions (the function body).** Definitions belong in `.cpp` files.

The only exception is templates and certain `inline` functions, which must be defined in headers, but that is a more advanced topic.
### Header Guards - The Fix

Even when headers contain only declarations, including the same header twice can still cause trouble in larger projects (for example, when two different headers both include a third common header). The standard solution is the **header guard**:
```cpp
// source.hpp — with header guard

#ifndef SOURCE_HPP        // "if not defined SOURCE_HPP"
#define SOURCE_HPP        // "define SOURCE_HPP (now it is defined)"

int add(int a, int b);

#endif                    // "end of the conditional block"
```

Here is what happens on the first inclusion:
1. The preprocessor asks: is `SOURCE_HPP` already defined? No.
2. So it enters the block, defines `SOURCE_HPP`, and processes the content.

On a second inclusion of the same file:
1. The preprocessor asks: is `SOURCE_HPP` already defined? Yes.
2. So it skips the entire block.

The result is that no matter how many times this header is included - directly or transitively - its content only ever appears once in the translation unit.

You can verify this with `-E`:
```bash
g++ -E main.cpp   # with header guard in source.hpp
```
Even if `main.cpp` includes `source.hpp` twice explicitly, the declaration of `add` will appear only once in the output.

A modern alternative to the traditional header guard is:
```cpp
#pragma once
```

This achieves the same effect and is supported by all major compilers (GCC, Clang, MSVC), though it is technically not part of the C++ standard.

### What Happens When You Include a Standard Library

Including `<iostream>` with `-E` to see the output is a useful exercise:
```bash
g++ -E main.cpp
```

You will see definitions for `std::cout`, `std::endl`, and many other utilities pasted directly into the preprocessed output. This is the same mechanism - the standard library is just a large collection of header files that get copied into your translation unit. The **implementation** of these functions lives in pre-compiled shared libraries that the linker brings in later.

### 4.6 The `extern` Keyword

When a function is declared in a header:
```cpp
int add(int a, int b);
```

it is implicitly `extern` - meaning the compiler assumes its definition will be found somewhere else (another translation unit, a library, etc.) and defers the resolution to the linker.

You can write this explicitly:
```cpp
extern int add(int a, int b);
```

but you normally do not need to. The explicit `extern` is most commonly used for global variables that need to be shared across translation units. For functions, the implicit extern behavior is the default.

The distinction between a compile-time error and a link-time error is important here:

- If you call `add()` without any declaration and without including the header, the compiler says: `add` was not declared in this scope - a **compile error**.
- If you call `add()` with the declaration present (via the header) but without providing the implementation `.cpp` file to the linker, the compiler is satisfied but the linker says: undefined reference to `add(int, int)` - a **link error**.

These are two completely different problems with two completely different solutions.

---
## Stage 2 Deep Dive: The Compiler

### Building the Internal Tree (AST / CST)

When the compiler processes your code, it does not work on it as raw text. It parses the text and builds a tree data structure that represents the logical structure of your program. This tree is called an **Abstract Syntax Tree (AST)** or, in some contexts, a **Concrete Syntax Tree (CST)**.

Consider this simple statement:

```cpp
int x = 2 + 2;
```

The compiler sees this and builds something like:
![Image Description: A tree diagram showing the AST for 'int x = 2 + 2'. The root node is labeled 'Assignment'. It has two children: left child labeled 'Variable x (int)', and right child labeled 'BinaryOp (+)'. The BinaryOp node has two children, both labeled 'Literal 2'. Arrows connect parents to children.](./attachments/image2.png)

The compiler then **traverses** this tree. During the traversal it:
- Checks that variables are declared before use
- Checks that types are compatible (you cannot add a string and an integer without a conversion)
- Validates that the grammar is correct (correct syntax for statements, expressions, blocks, etc.)
- Detects re-declarations of the same variable in the same scope

Line number information is attached to each node in the tree. That is how the compiler tells you "syntax error on line 7" - it looks at the line number stored in the relevant tree node.

### How Errors Are Located

If `x` were already declared above this statement, the compiler would walk down the tree, see the assignment to `x`, check against its symbol table, find that `x` already exists in this scope, and report:
```
error: redeclaration of 'int x'
```
along with the line number stored in the AST node. This is not heuristic guessing - it is a direct result of the tree structure and the attached metadata.

### Dumping the Syntax Tree with GCC

GCC provides a way to dump visual representations of the internal control flow graph that it generates. It is not something you need to memorize or use day-to-day, but it is useful for understanding what is happening under the hood, and occasionally useful for low-level analysis:
```bash
g++ -fdump-tree-all-graph -g main.cpp source.cpp
```

This generates a collection of `.dot` files in the current directory - one for each internal representation of each function. These `.dot` files can be rendered as diagrams using tools like **Graphviz** (`dot`, `xdot`) on Linux/Mac or similar tools on other platforms.

Opening one of these files, you might see something like this for `main()`:
```
main function entry
  → call add(2, 3)
    → store result in temporary _3
  → return
```

Again, this is not something you need to read fluently. The point is that *your code has a structured internal representation inside the compiler*, and that representation is what drives both error detection and code generation.
### Code Generation: From Tree to Assembly

After parsing and validation, the compiler walks the tree one more time to **generate assembly instructions**. Each node in the tree corresponds to one or a few assembly operations. An addition node becomes an `add` instruction. An assignment node becomes a `mov` instruction. A function call node becomes a `call` instruction.

This is the **code generation** step. The result is the `.s` assembly file.

---
## Stage 3 Deep Dive: Assembly and Object Files

### Generating the Assembly File with `-S`

You can ask `g++` to stop after the compiler stage - after generating assembly but before assembling it into machine code - using the `-S` flag:
```bash
g++ -S main.cpp
```
This produces `main.s` in the current directory.

### Reading Assembly Output

Opening `main.s`, you will see something like this (exact output depends on your platform, compiler version, and optimization level):
```asm
    .file   "main.cpp"
    .text
    .globl  main
    .type   main, @function
main:
    pushq   %rbp
    movq    %rsp, %rbp
    ...
    call    _Z3addii       ; call to add(int, int)
    ...
    movl    $0, %eax
    popq    %rbp
    ret
```

A few things to notice:
- **`.text`** - this section marker tells the assembler that what follows is executable code (as opposed to `.data` for initialized global variables, or `.bss` for uninitialized ones).
- **Labels** like `main:` - these are named positions in the code. The linker uses them to resolve function calls.
- **`call _Z3addii`** - this is the mangled name of the `add(int, int)` function. C++ compilers mangle function names to encode type information (since C++ supports overloading). `_Z3addii` means: a function named `add` (`3add` because "add" is 3 characters) that takes two `int` arguments (`ii`).
- Each line is **one operation**. This is much more granular than C++ source code.

Assembly is a textual representation of machine code. The assembler's job is to convert this text into actual binary bytes that the CPU can execute.

### Compiling to Object Files with `-c`

You can stop the pipeline just after the assembler stage - producing an object file but not yet linking - using the `-c` flag:
```bash
# Compile main.cpp to main.o (object file, not executable)
g++ -c main.cpp

# Compile source.cpp to source.o
g++ -c source.cpp
```

After these two commands, you have `main.o` and `source.o`. Neither is executable on its own. `main.o` has a call to `add`, but `add` is not resolved yet. `source.o` has the definition of `add`, but no `main` entry point.

The `-c` flag is how you compile large projects efficiently: compile each `.cpp` file separately into its own `.o`, then link them all together at the end. In a large project, if you change one `.cpp` file, you only need to recompile that file to a new `.o` - you do not have to recompile everything. The linker then re-links all the `.o` files into a new executable. This is exactly what build systems like `make`, `cmake`, and `ninja` do for you.

Linking the two object files together into an executable:
```bash
g++ main.o source.o -o prog
```

### Inspecting Object Files with `objdump`

Object files are binary. You cannot just open them in a text editor and read them. The `objdump` tool knows how to parse the internal structure of these binary files and display the information in a readable form.
```bash
# Show the symbol table of source.o
objdump -t source.o
```

The `-t` flag requests the **symbol table** — a list of all the named things (functions, global variables) that this object file defines or references. Sample output:

```
source.o:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*  0000000000000000 source.cpp
0000000000000000 l    d  .text  0000000000000000 .text
0000000000000000 g     F .text  0000000000000016 _Z3addii
```

The last line tells you that this object file contains a function (`F`) named `_Z3addii` (the mangled form of `add(int, int)`), and it lives at offset 0 in the `.text` section.

After linking, you can inspect the final executable similarly:
```bash
objdump -t prog
```

Now you will see symbols from both `main.cpp` and `source.cpp`, plus any symbols pulled in from external libraries. If you compiled with debugging information (`-g`), you will see even more detail including source file names.

Platform-equivalent tools:

| Platform | Tool                         |
| -------- | ---------------------------- |
| Linux    | `objdump`                    |
| macOS    | `otool`                      |
| Windows  | Dependency Walker (for DLLs) |

---
## Stage 4 Deep Dive: The Linker

### Static Linking - Gluing Object Files Together

The linker's primary job is to take all the `.o` files your project produced, resolve all the cross-references between them, and produce a single executable.

"Resolving a cross-reference" means: `main.o` has a `call _Z3addii` instruction that currently points to a placeholder address. The linker looks through all the object files, finds that `source.o` has a symbol `_Z3addii` at a known offset, and patches the `call` instruction in `main.o` to point to the correct address. This process happens for every unresolved symbol across all your object files.

When you run:
```bash
g++ main.o source.o -o prog
```
the linker is doing all of this gluing behind the scenes.

### Dynamic Linking - Shared Libraries at Runtime

When a program uses a shared library (`.so`, `.dll`, `.dylib`), the linker does not copy the library's code into the executable. Instead, it records a **dependency**: "this program needs `libstdc++.so.6`, and it uses the symbol `_ZSt4cout` from it."

When you run the program, the operating system's **dynamic linker** (also called the dynamic loader) reads this dependency list, finds the required shared library files on disk, loads them into memory, and patches the executable's reference table so the calls go to the right place.

The advantage of dynamic linking:
- Multiple executables on the system can share a single copy of the library in memory — saving RAM.
- You can update the shared library (for example, a security patch) without recompiling all the programs that use it.

The disadvantage:
- If the required shared library is not present on the user's machine (wrong version, not installed), the program fails at launch rather than at build time.

### Checking Dynamic Dependencies with `ldd`

The `ldd` command (Linux) lists all the shared libraries that a given executable needs at runtime, and shows where those libraries are located on the current system:
```bash
ldd prog
```

For the simple `hello world` style program that uses `cout`:
```
    linux-vdso.so.1 (0x00007fff...)
    libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f...)
    libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f...)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f...)
    /lib64/ld-linux-x86-64.so.2 (0x00007f...)
```

After linking in the SFML library (see Section 9), `ldd prog` will show several additional entries like `libsfml-system.so`, `libsfml-window.so`, and `libsfml-graphics.so`.

macOS users use `otool -L prog` for equivalent information.
### Common Linker Errors

| Error Message                  | What It Means                                                         | How to Fix It                                                                                                 |
| ------------------------------ | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `undefined reference to 'foo'` | The linker cannot find the implementation of `foo`                    | Add the `.cpp` file or `.o` file that implements `foo`; or add the `-l` flag for the library that contains it |
| `multiple definition of 'foo'` | The linker found more than one definition of `foo`                    | Remove duplicate definitions; check header guards; do not define functions in header files                    |
| `cannot find -lbar`            | The linker was told to link against `libbar` but cannot find the file | Install the library, or use `-L` to point to the directory where it lives                                     |

> **Key insight for debugging:** When you see `undefined reference`, do not look at your syntax. Look at your *linker invocation* — which `.cpp` or `.o` files and `-l` flags are you passing? Something is missing there.

---

## Working with External Libraries: Compiler Flags

When using third-party libraries, you need to tell the compiler and linker three things:
1. Where to find the **header files** for the library (so the compiler knows the function signatures)
2. Where to find the **library files** (`.so`, `.a`, `.dll`, `.lib`, `.dylib`) at link time
3. **Which** library files to link against

These three needs correspond to three flags.

### The `-l` Flag: Specifying a Library

```bash
g++ main.cpp -o prog -lsfml-system -lsfml-window -lsfml-graphics
```

The `-l` flag (lowercase L) tells the linker to look for a library named `lib<name>.<ext>`. So `-lsfml-system` tells the linker to find `libsfml-system.so` (or `libsfml-system.a` for static). You drop the `lib` prefix and the extension — the linker assumes the conventional naming.

Multiple `-l` flags are used for multiple libraries. Order can matter in some cases: if library A depends on library B, then `-lA` should come before `-lB` in the command.

### The `-L` Flag: Specifying a Library Search Directory

```bash
g++ main.cpp -o prog -L/path/to/library/directory -lsfml-system
```

The `-L` flag (uppercase L) tells the linker to also look in the specified directory when searching for libraries named with `-l`. Without this, the linker only searches the default system paths (like `/usr/lib`, `/usr/local/lib`).

If you installed a library in a non-standard location (for example, you compiled it yourself and put it in `/home/user/mylibs/`), you need `-L/home/user/mylibs` to tell the linker to look there.

### The `-I` Flag: Specifying an Include Directory

```bash
g++ main.cpp -o prog -I/usr/include/sfml -lsfml-graphics
```

The `-I` flag (uppercase I) adds a directory to the list of places the **preprocessor** looks when it resolves `#include <...>` directives (the angle-bracket form used for system headers).

When you write:
```cpp
#include <SFML/Graphics.hpp>
```
the preprocessor looks through its list of include directories for a subdirectory named `SFML` containing `Graphics.hpp`. If your SFML headers are installed in `/usr/include/` (which would make the full path `/usr/include/SFML/Graphics.hpp`), you might need `-I/usr/include` (though this is usually already in the default path).

If they were installed somewhere unusual like `/opt/sfml/include/`, you would need `-I/opt/sfml/include`.

### Default System Search Paths

You do not always need `-L` and `-I` because the compiler and linker have default search paths. On Linux, the `PATH`-like variable and system configuration tell the linker to look in directories like `/usr/lib`, `/usr/local/lib`, etc. The preprocessor similarly has default include paths that typically include `/usr/include` and `/usr/local/include`.

On Linux, you can see the default include search paths by running:
```bash
g++ -v -E - < /dev/null 2>&1 | grep -A 20 "#include <...>"
```

When you install a library with your package manager (`apt`, `brew`, `pacman`, etc.), the package manager typically installs the library files and headers into these default paths, so you do not need `-L` or `-I`. You only need `-l` to name the library.

When you install a library manually or into a custom location, you need all three flags.

---

## Types of Object Files

GCC can produce three distinct types of object files:

| Type | Description | Executable? |
|---|---|---|
| **Relocatable** | Output of the assembler stage (`.o` files). Has unresolved symbol references. Must be processed by the linker before it can run. | No |
| **Executable** | A fully linked binary that can be run directly. | Yes |
| **Shared Object** | A shared library (`.so` file). Designed to live on the target system so the dynamic linker can link into it at runtime. Not executable on its own. | No |

---
## The ELF Format

On Linux, all object files — relocatable, executable, and shared objects alike - use the **ELF** format.

> **ELF** stands for **Executable and Linkable Format**. It is the standard binary format on Linux (and many other Unix-like systems).

ELF files carry a rich set of metadata that tools like `file` and `readelf` can parse and display. This metadata tells you things like:

- Whether the file is 32-bit or 64-bit
- The target CPU architecture (`x86-64`, `ARM`, etc.)
- The byte order (**little-endian** = LSB, **big-endian** = MSB)
- Whether the binary is dynamically or statically linked
- The path to the dynamic linker/loader the binary expects to find on the target system
- Whether the binary still has its symbol table ("`not stripped`") or has had it removed ("`stripped`")

**Stripping** refers to removing the symbol table from a binary. The symbol table is useful for debugging (tools like `gdb` use it), but it takes up space and is not needed for the program to actually execute. Before distributing a binary, you would typically strip it.

![Image Description: An annotated diagram of the output of the `file` command run on an ELF executable. Each field in the output is highlighted and labelled: ELF type (executable/shared object), bit-width (64-bit), endianness (LSB = little-endian), dynamic/static linkage, interpreter path, build ID, and stripped/not-stripped status.](./attachments/image3.png)

---
### Inspecting the Binary with `file`

The `file` command reads the metadata embedded in a file and tells you what it is:
```bash
file main
```

Example output:
```bash
main: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked,
interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=<hash>, for GNU/Linux 3.2.0, not stripped
```

Let's walk through each field:

| Field | Meaning |
|---|---|
| `ELF 64-bit` | This is an ELF file targeting a 64-bit processor |
| `LSB` | **Least Significant Byte** first — this is a **little-endian** binary. If it said `MSB` it would be big-endian. |
| `shared object` | See note on PIE below — this is not as alarming as it sounds |
| `x86-64` | Target CPU architecture |
| `dynamically linked` | The binary has references to external shared libraries; those must exist on the target system |
| `interpreter /lib64/ld-linux-x86-64.so.2` | The path to the **dynamic linker/loader** the OS must find in order to run this binary |
| `BuildID[sha1]=<hash>` | A SHA-1 fingerprint of this build. Re-running the same compiler command produces the same hash. |
| `not stripped` | The symbol table is still present — useful for debugging, but takes up extra space |

---
### Position Independent Executables (PIE)
You may notice the `file` output says `shared object` even though you compiled an executable. This is because the **default behaviour on modern Linux is to compile executables as Position Independent Executables (PIE)**.

A PIE binary can be **loaded at any address in memory** — the OS can place it wherever it wants, which is the basis for **Address Space Layout Randomization (ASLR)**, a security feature. Because the binary can be loaded anywhere, the compiler formats it similarly to a shared object (`.so`) internally - hence the `shared object` label in `file` output.

To disable PIE and produce a "traditional" executable (one with a fixed load address), pass `-no-pie`:
```bash
gcc main.c -lm -no-pie -o main
```

Now `file hello` will say `executable` instead of `shared object`.
> In everyday practice, for a general Linux application, you should leave PIE enabled and let GCC do what it wants. The `-no-pie` flag is shown here purely for illustration — so that the `file` output displays the more intuitive `executable` label.

---

### 7.6 Inspecting Shared Library Dependencies with `readelf`

`readelf` is a utility from the **binutils** package. It reads and parses the internal structure and metadata of ELF files. Passing it `-a` dumps everything:
```bash
readelf -a main
```

That produces a lot of output. A practical way to focus on specific things is to pipe through `grep`.

**Finding shared library dependencies:**

```bash
readelf -a mai | grep shared
```

Output (example):

```
0x0000000000000001 (NEEDED)   Shared library: [libm.so.6]
0x0000000000000001 (NEEDED)   Shared library: [libc.so.6]
```

This tells you exactly which shared libraries the binary requires at runtime. If either of those `.so` files does not exist on the target system, the binary will refuse to run.

**Finding the expected dynamic linker path:**
```bash
readelf -a hello | grep interpreter
```

Output (example):
```
[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```
This is the same interpreter path shown by `file`. The dynamic linker at that exact path must exist on the target system. If it does not, the executable will not start.

---
### Keeping Intermediate Files with `--save-temps`

By default, GCC deletes the intermediate files (`.i`, `.s`, `.o`) once compilation is done. If you want to inspect them, use `--save-temps`:
```bash
gcc main.c -lm -no-pie --save-temps -o main
```

After this command, you will see all of these files in the directory:
```
main.c       # original source
main.i       # pre-processed output (expanded source)
main.s       # compiled assembly
main.o       # assembled object file (relocatable, not linked)
main         # final linked executable
```
You can then inspect each file to understand what happened at each stage of the pipeline.


---
## Practical Example: The SFML Graphics Library

SFML (Simple and Fast Multimedia Library) is a cross-platform multimedia/game development framework. It provides windows, 2D graphics, audio, and networking. Using it as an example is helpful because it is a real, non-trivial library that requires all the compiler flags discussed above.

### Installing SFML

On Ubuntu/Debian Linux:
```bash
sudo apt-get install libsfml-dev
```

On macOS with Homebrew:
```bash
brew install sfml
```

On Windows, download from the official SFML website and follow the installation guide.

### Compiling Without the Flags — What Goes Wrong

Here is the SFML "hello world" program (a window showing a green circle):
```cpp
// sfml_hello.cpp
#include <SFML/Graphics.hpp>

int main() {
    sf::RenderWindow window(sf::VideoMode(200, 200), "SFML works!");
    sf::CircleShape shape(100.f);
    shape.setFillColor(sf::Color::Green);

    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
        }
        window.clear();
        window.draw(shape);
        window.display();
    }
    return 0;
}
```

Trying to compile with just:
```bash
g++ -g sfml_hello.cpp -o prog
```

produces a flood of linker errors:
```
/tmp/ccXXXXXX.o: In function `main':
sfml_hello.cpp:(.text+0x...): undefined reference to `sf::RenderWindow::RenderWindow(...)'
sfml_hello.cpp:(.text+0x...): undefined reference to `sf::CircleShape::CircleShape(float, unsigned int)'
sfml_hello.cpp:(.text+0x...): undefined reference to `sf::Color::Green'
... many more ...
collect2: error: ld returned 1 exit status
```

Notice: these are **linker errors**, not compiler errors. The compiler found the headers (because SFML was installed in a default include path), so it knows the function signatures. But the linker cannot find the library files containing the actual function implementations.

### Compiling Correctly with All Flags

```bash
g++ -g sfml_hello.cpp -o prog \
    -lsfml-system \
    -lsfml-window \
    -lsfml-graphics
```

This compiles and links successfully. SFML is split into multiple sub-libraries, so you need to name each one you use. The main ones are:

| Flag              | What It Provides                     |
| ----------------- | ------------------------------------ |
| `-lsfml-system`   | Core utilities (time, threads, etc.) |
| `-lsfml-window`   | Window creation, input handling      |
| `-lsfml-graphics` | 2D rendering, shapes, sprites, text  |
| `-lsfml-audio`    | Sound and music playback             |
| `-lsfml-network`  | TCP/UDP networking                   |

If you needed all of them:
```bash
g++ -g sfml_hello.cpp -o prog \
    -lsfml-system \
    -lsfml-window \
    -lsfml-graphics \
    -lsfml-audio \
    -lsfml-network
```

After linking, check the dynamic dependencies:
```bash
ldd prog
```
You will now see `libsfml-system.so`, `libsfml-window.so`, and `libsfml-graphics.so` listed among the dependencies, along with their full paths on disk.

### Finding Where Libraries Are Installed

If you installed a library and cannot figure out where the files ended up:
```bash
# Find library files (.so) by name
find / -name "libsfml*" 2>/dev/null

# Find header files by name
find / -name "Graphics.hpp" 2>/dev/null
```

A more targeted approach for package-manager-installed libraries on Debian/Ubuntu:
```bash
dpkg -L libsfml-dev   # lists all files installed by this package
```

On macOS with Homebrew:
```bash
brew info sfml   # shows where Homebrew installed SFML
```

#### A Complete Command-Line Invocation with All Flags Explicit

Just to see everything spelled out at once, even for a system where the defaults would work fine:
```bash
g++ -g sfml_hello.cpp -o prog \
    -I/usr/include \
    -L/usr/lib/x86_64-linux-gnu \
    -lsfml-system \
    -lsfml-window \
    -lsfml-graphics
```

Breaking this down:

| Flag | Role |
|---|---|
| `-g` | Include debugging symbols in the output (useful with `objdump`, `gdb`, etc.) |
| `sfml_hello.cpp` | The source file to compile |
| `-o prog` | Name the output executable `prog` |
| `-I/usr/include` | Look here for `#include <SFML/Graphics.hpp>` and similar |
| `-L/usr/lib/x86_64-linux-gnu` | Look here for `libsfml-*.so` files |
| `-lsfml-system` | Link against `libsfml-system.so` |
| `-lsfml-window` | Link against `libsfml-window.so` |
| `-lsfml-graphics` | Link against `libsfml-graphics.so` |

---
## Useful Tools Reference

| Option         | Effect                                                                                                                  |
| -------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `-o <file>`    | Set the name of the output file                                                                                         |
| `--save-temps` | Do not delete intermediate files (`.i`, `.s`, `.o`) — leave them on disk for inspection                                 |
| `--static`     | Produce a statically linked executable                                                                                  |
| `-no-pie`      | Disable Position Independent Executable (PIE) — produces a traditional executable rather than a shared-object-style PIE |
	
| Tool                           | Platform      | Purpose                                                       |
| ------------------------------ | ------------- | ------------------------------------------------------------- |
| `g++ -E`                       | Linux/Mac     | Stop after preprocessing; print the expanded source           |
| `g++ -S`                       | Linux/Mac     | Stop after compilation; produce the `.s` assembly file        |
| `g++ -c`                       | Linux/Mac     | Stop after assembling; produce the `.o` object file           |
| `g++ -fdump-tree-all-graph -g` | Linux/Mac     | Dump internal AST/CFG graphs as `.dot` files                  |
| `objdump -t`                   | Linux         | Print the symbol table of an object file or executable        |
| `otool -L` / `otool -t`        | macOS         | macOS equivalent of `objdump`                                 |
| `ldd`                          | Linux         | List the shared libraries an executable depends on at runtime |
| `nm`                           | Linux/Mac     | Another way to list symbols in object files                   |
| `find`                         | Linux/Mac     | Locate library or header files on the filesystem              |
| `dpkg -L <package>`            | Debian/Ubuntu | List all files installed by a package                         |

---

## Complete Compilation Command Reference

### Basic compilation
```bash
# Compile and link in one step, output named 'prog'
g++ main.cpp source.cpp -o prog

# Compile only (produce object file, no linking)
g++ -c main.cpp          # produces main.o
g++ -c source.cpp        # produces source.o

# Link object files into an executable
g++ main.o source.o -o prog
```
### Inspecting each stage
```bash
# See preprocessor output
g++ -E main.cpp

# See assembly output
g++ -S main.cpp          # produces main.s

# Dump internal AST graphs
g++ -fdump-tree-all-graph -g main.cpp source.cpp
```

### Compiling with debugging symbols
```bash
g++ -g main.cpp source.cpp -o prog
```

### Using an external library
```bash
g++ main.cpp -o prog \
    -I/path/to/headers \           # -I: header search directory
    -L/path/to/libraries \         # -L: library search directory
    -lname_of_library              # -l: name of library to link
```

### Inspecting binary files
```bash
objdump -t source.o           # symbol table of an object file
objdump -t prog               # symbol table of the executable
ldd prog                      # shared library dependencies (Linux)
otool -L prog                 # shared library dependencies (macOS)
```

---

## Concept Summary and Key Takeaways

Here is a consolidated recap of every major concept in this guide.

**The Pipeline:**
The compilation of a C/C++ program passes through four distinct stages: preprocessing → compilation → assembly → linking. Running `g++ main.cpp -o prog` performs all four stages invisibly in sequence.

**The Preprocessor (`-E`):**
- Handles all `#` directives
- `#include` is a literal copy-paste of file contents
- `#define` creates text-substitution macros
- `#if`/`#ifdef`/`#ifndef`/`#endif` enables conditional compilation
- Header guards (`#ifndef HEADER_HPP` / `#define HEADER_HPP` / `#endif`) prevent multiple-inclusion errors
- Use `-E` to see exactly what the preprocessor produces

**The Compiler (`-S`):**
- Parses preprocessed code into an Abstract Syntax Tree
- Validates grammar, types, and scoping rules
- Detects syntax errors and reports them with line numbers
- Generates assembly code from the tree
- Use `-S` to see the assembly output

**The Assembler (`-c`):**
- Converts assembly text (`.s`) into binary machine code (`.o`)
- The resulting object file may have unresolved symbol references
- Use `-c` to stop at this stage and inspect the object file
- Use `objdump -t` to see the symbol table

**The Linker:**
- Combines multiple `.o` files, resolving cross-references
- Performs static linking (copying library code into the executable)
- Records dependencies for dynamic linking (shared libraries loaded at runtime)
- `undefined reference` errors come from here, not from the compiler
- Use `ldd` (Linux) or `otool -L` (macOS) to check dynamic dependencies

**The Flags:**
- `-l<name>` — name a library to link against (e.g., `-lsfml-graphics`)
- `-L<path>` — add a directory to the library search path
- `-I<path>` — add a directory to the header search path
- `-c` — compile to object file only (no linking)
- `-S` — compile to assembly only
- `-E` — preprocess only
- `-o <name>` — set the output file name
- `-g` — include debugging symbols

**Error Diagnosis Guide:**

| Error Pattern | Stage | Root Cause |
|---|---|---|
| `'foo' was not declared in this scope` | Compiler | Missing `#include` or missing declaration |
| `redefinition of 'int foo'` | Compiler/Linker | Function or variable defined multiple times |
| `undefined reference to 'foo'` | Linker | Implementation not provided; missing `.cpp` or `-l` flag |
| `cannot find -lfoo` | Linker | Library file not in any search path; need `-L` or install the library |
| Program crashes at launch with "shared library not found" | Runtime (dynamic linker) | `.so`/`.dll`/`.dylib` file not present on the system at runtime |

---

*This guide was fully generated by an LLM using some source materials*

---