# Variable Types and Memory Management in C/C++

#static #extern #variableTypes #memorySegments

## Understanding Variable Types

Variables in C/C++ differ across four fundamental characteristics:
- **Scope** - Where the variable can be accessed
- **Initial Value** - What value the variable starts with
- **Memory Segment** - Which part of memory stores the variable
- **Lifetime** - How long the variable exists

## Scope Resolution

**Scope resolution** is the process by which a programming language determines which variable, function, or class member to use when multiple definitions share the same name across different scopes (local, global, class, namespace, etc.).

### C++ Scope Resolution Example:
```cpp
#include <iostream>
using namespace std;

int x = 100;  // Global variable

int main() {
    int x = 50;               // Local variable
    cout << x << endl;        // Prints 50 (local x)
    cout << ::x << endl;      // Prints 100 (global x using scope resolution operator)
}
```

```cpp
class A {
public:
    static int value;
};
int A::value = 10;  // Define the static member variable
```

### Static vs Dynamic Scoping

| Feature          | Static Scoping (Lexical)          | Dynamic Scoping                    |
| ---------------- | --------------------------------- | ---------------------------------- |
| **When decided** | Compile time                      | Runtime                            |
| **How resolved** | Based on **code structure**       | Based on **call stack**            |
| **Common in**    | C, C++, Java, Python, JavaScript  | Lisp (early), Perl, Bash (partial) |
| **Efficiency**   | Faster (resolved at compile time) | Slower (runtime lookup required)   |
| **Behavior**     | Always consistent                 | Depends on calling context         |

#### Static Scoping (Lexical Scoping)
In static scoping, variable resolution follows the lexical structure of the code. The compiler searches for variables in the containing block or function first, then in outer scopes, and finally in global scope. This approach looks at **where the variable is defined in the source code**, not where the function is called. Static scoping is used by **C, C++, Java, Python, and JavaScript**.

#### Dynamic Scoping
In dynamic scoping, variable resolution follows the execution call stack. The program searches for variables in the current function, then in the calling function, and continues up the call chain. This approach looks for the most recent variable definition in the **function call sequence**, not in the lexical code structure. **C and C++ do not support dynamic scoping**. Some older languages like **LISP and early BASIC** supported it.

### Example: Static vs Dynamic Scoping
```c
#include <stdio.h>

int x = 0;  // Global variable

int f() {
    return x;
}

int g() {
    int x = 1;  // Local variable
    return f();
}

int main() {
    printf("%d\n", g());
    return 0;
}
```
This prints `0` because C uses static scoping. With dynamic scoping, it would print `1`.

## The extern Keyword

The `extern` keyword extends a variable's scope from file scope to project scope, allowing access across multiple source files.

Consider two files in the same project:

**file1.c:**
```c
#include <stdio.h>

int x = 5;  // Global variable with file scope

int main() {
    return 0;
}
```

**file2.c:**
```c
#include <stdio.h>

extern int x;  // Declares x as externally defined

int main() {
    printf("%d\n", x);  // Can now access x from file1.c
    return 0;
}
```

**Important:** You cannot initialize a variable when declaring it with `extern`. The linker handles the connection between files during compilation.

## Local Variables

### Characteristics:
- **Scope:** Block scope (within curly braces)
- **Initial Value:** Garbage value (undefined)
- **Memory Segment:** Stack
- **Lifetime:** Function execution duration

### Examples of Local Variable Scope:

**Function scope:**
```c
#include <stdio.h>

int main() {
    int x;  // Local to main function
    return 0;
}
```

**Block scope:**
```c
#include <stdio.h>

int main() {
    {
        int x;  // Local to this block
    }
    return 0;
}
```

**Parameter scope:**
```c
#include <stdio.h>

void fun(int x) {  // x is local to function fun
    return;
}

int main() {
    return 0;
}
```

## Global Variables

### Characteristics:
- **Scope:** File scope
- **Initial Value:** Default value for the data type (0 for integers, NULL for pointers)
- **Memory Segment:** `.bss` (if uninitialized or initialized to default value) or `.data` (if initialized to non-default value)
- **Lifetime:** Program execution duration

## Local vs Global: Practical Examples

### Example 1: Variable Shadowing
```c
#include <stdio.h>

int x = 5;  // Global variable

int main() {
    int x = 10;  // Local variable shadows global
    printf("%i\n", x);  // Prints 10
    return 0;
}
```
The local variable takes precedence due to smaller scope.

### Example 2: Function Access
```c
#include <stdio.h>

int x = 5;  // Global variable
void f(void);

int main() {
    int x = 10;  // Local variable
    printf("%i\n", x);  // Prints 10
    f();                 // Prints 5
    return 0;
}

void f(void) {
    printf("%i\n", x);  // Accesses global x (no local x in scope)
}
```

### Example 3: Block Modification
```c
#include <stdio.h>

int x = 5;  // Global variable

int main() {
    int x = 10;  // Local variable
    {
        x = 20;      // Modifies the local variable
        printf("%i\n", x);  // Prints 20
    }
    printf("%i\n", x);      // Prints 20
    return 0;
}
```

### Example 4: Nested Block Variables
```c
#include <stdio.h>

int x = 5;  // Global variable

int main() {
    int x = 10;  // Local variable
    {
        int x;       // New local variable in inner scope
        x = 20;
        printf("%i\n", x);  // Prints 20
    }
    printf("%i\n", x);      // Prints 10 (outer local variable)
    return 0;
}
```

### Example 5: Self-Assignment Issue
```c
#include <stdio.h>

int x;  // Global variable (initialized to 0)

int main() {
    int x = x;  // Local variable assigned to itself (garbage value)
    printf("%i\n", x);  // Prints garbage value
    return 0;
}
```

In C++, you can use the scope resolution operator to access the global variable:
```cpp
#include <iostream>

int x;  // Global variable

int main() {
    int x = ::x;  // Local variable assigned global variable's value
    std::cout << x << std::endl;  // Prints 0
    return 0;
}
```

## Static Variables

### Static Global Variables
- **Scope:** File scope (cannot be accessed from other files)
- **Initial Value:** Default value for the data type
- **Memory Segment:** `.bss` or `.data` (same as regular global variables)
- **Lifetime:** Program execution duration

Static global variables are identical to regular global variables except they cannot be accessed using `extern` from other files.

### Static Local Variables
- **Scope:** Block scope (where declared)
- **Initial Value:** Default value (0 for integers)
- **Memory Segment:** `.bss` or `.data`
- **Lifetime:** Program execution duration

Static local variables are initialized only once before runtime, so we can say that all variable types are initialized before runtime except for normal local variables which are initialized during runtime.
### Static Variable Examples

#### Example 1: Regular vs Static Local
```c
#include <stdio.h>

void print(void);

int main() {
    print();  // Prints 0
    print();  // Prints 0
    print();  // Prints 0
    return 0;
}

void print(void) {
    int x = 0;       // Regular local variable
    printf("x=%i\n", x);
    x++;
}
```

```c
#include <stdio.h>

void print(void);

int main() {
    print();  // Prints 0
    print();  // Prints 1
    print();  // Prints 2
    return 0;
}

void print(void) {
    static int x = 0;  // Static local variable
    printf("x=%i\n", x);
    x++;
}
```

#### Example 2: Initialization Restrictions
This code causes a compilation error:
```c
#include <stdio.h>

int print2(void);
void print(void);

int main() {
    print();
    return 0;
}

void print(void) {
    static int x = print2();  // ERROR: Cannot initialize static with function call
    printf("x=%i\n", x);
}

int print2(void) {
    printf("not ok\n");
    return 5;
}
```

Static variables must be initialized with compile-time constants only.
## Key Insight: Static Keyword Effects
**Does the `static` keyword affect scope or lifetime?**
- For **global variables:** Affects scope (limits to file scope)
- For **local variables:** Affects lifetime (extends to program lifetime)
## Static Functions
By default, functions have software scope and can be used across files. The `static` keyword limits function scope to the current file.

**Example: Regular function (accessible across files):**

**file1.c:**
```c
void hello(void) {
    printf("hello");
}
```

**file2.c:**
```c
#include <stdio.h>

void hello(void);  // Function declaration

int main() {
    hello();  // Works fine
    return 0;
}
```

**Example: Static function (file scope only):**

**file1.c:**
```c
static void hello(void) {
    printf("hello");
}
```

**file2.c:**
```c
#include <stdio.h>

static void hello(void);  // Declaration

int main() {
    hello();  // Linker error - function not accessible
    return 0;
}
```

## Const Variables

### Const Local Variables
- Have the same properties as regular local variables
- Cannot be modified after initialization
- Can be changed indirectly using pointers (undefined behavior)

### Const Global Variables
- Have the same properties as regular global variables except:
- Cannot be modified after initialization
- Stored in `.rodata` (read-only data) segment instead of `.data` in initialized non zero values, and stored in `.bss` in initialized zero values, or uninitialized.
- Attempting to modify results in runtime error
## Memory Segments

### RAM Segments:

#### Stack
- Stores function contexts (things function stores before calling another function) and local variables
- Grows and shrinks during program execution
- Fast access but limited size

#### Heap
- Used for dynamic memory allocation at runtime
- Grows and shrinks based on memory requests
- Managed by `malloc()`, `free()`, etc.

#### .bss (Block Started by Symbol)
- Stores uninitialized global and static variables
- Variables automatically initialized to zero
- Consumes no space in the executable file

#### .data
- Stores initialized global and static variables with non-zero values
- Takes up space in the executable file

### Flash Memory Segments:

#### .rodata (Read-Only Data)
- Stores string literals, const global variables, and lookup tables
- Read-only during program execution
- Located in non-volatile memory

#### Interrupt Vector Table (IVT)
- Contains addresses of interrupt handler functions
- Each interrupt type has a corresponding entry
- Read-only, located in flash memory

#### .text
	- Contains compiled program instructions (machine code) (exe)
- All executable code resides here
- Read-only, located in flash memory

### Dual Location: .data Segment

The `.data` segment exists in both Flash and RAM:

1. **Flash Memory:** Contains initial values of initialized global/static variables
2. **RAM:** Working copy created at program startup

**Startup Process:**
1. Program loads into Flash memory
2. Startup code (in `.text`) executes first
3. Startup code copies initialized data from Flash `.data` to RAM `.data`
4. Startup code zeros out the `.bss` segment in RAM
5. Program execution begins

![pic1](Attachments/Session-12/pic1.png)

| Segment       | Content                          | Flash Location | RAM Location | Writable? | Notes                                    |
| ------------- | -------------------------------- | -------------- | ------------ | --------- | ---------------------------------------- |
| `.text`       | Machine code instructions        | Yes            | No           | No        | Executes directly from Flash            |
| `.rodata`     | Constants, string literals       | Yes            | No           | No        | Read-only data in Flash                  |
| `.data`       | Initialized globals/statics      | Yes (initial)  | Yes (runtime)| Yes       | Copied from Flash to RAM at startup     |
| `.bss`        | Uninitialized globals/statics    | No             | Yes (zeroed) | Yes       | Zero-initialized in RAM                  |
| Heap          | Dynamic memory                   | No             | Yes          | Yes       | Grows upward in RAM                      |
| Stack         | Local vars, return addresses     | No             | Yes          | Yes       | Grows downward in RAM                    |
| IVT           | Interrupt vectors                | Yes (fixed)    | No           | No        | Contains stack pointer and ISR addresses |

## Checking Code Size

You can view your program's memory usage in two ways:

**Method 1: Command line**
![pic2](Attachments/Session-12/pic2.png)

**Method 2: Check the `.lss` file in your project directory**

## Storage Class Keywords

### auto Keyword
- Used for declaring local variables (optional)
- Has no functional effect in modern C
- Cannot be used with global variables

```c
#include <stdio.h>

int main() {
    auto int x = 10;  // Explicitly shows it's a local variable
    return 0;
}
```

Invalid usage:
```c
#include <stdio.h>

auto int x = 10;  // ERROR: auto cannot be used for global variables

int main() {
    return 0;
}
```

Some people use it without using the datatype, but this is not preferred, as the compiler implicitly converts it to the nearest type for it.
```c
#inlcude <stdio.h>

int main () {
	auto x = 10; // Casted into int
	printf("%i", x);
	return 0;
}
```


### register Keyword
- **Suggests** to the compiler that a variable should be stored in CPU registers
- Compiler may ignore this suggestion
- **Restriction:** Cannot use the address operator (`&`) on register variables
- Modern compilers typically handle register allocation better than manual specification

### volatile Keyword
- Tells the compiler that a variable's value may change unexpectedly
- Prevents compiler optimization that might cache the variable's value
- Essential for hardware registers, interrupt handlers, and multi-threaded code
- `volatile` and `const` are called **type qualifiers**

## Storage Class Summary

| Storage Class | Memory Location    | Initial Value | Scope                                      | Lifetime        |
| ------------- | ------------------ | ------------- | ------------------------------------------ | --------------- |
| **auto**      | Stack              | Garbage       | Block scope                                | End of block    |
| **extern**    | .data or .bss      | Zero          | Global (multiple files)                    | Program end     |
| **static**    | .data or .bss      | Zero          | Block (local) or File (global)            | Program end     |
| **register**  | CPU registers/Stack| Garbage       | Block scope                                | End of block    |


## Compilation (Build) process:

### A. Pre-Processor:
The preprocessor takes all the files of the project (all the source files, all built in header files, all user defined header files) and it outputs `.i` files that are equivalent to the C files.
It deals with all commands that starts with `#` which have a name of **preprocessor directives**.
Pre-processor replaces preprocessor directives with a text that is convenient for the rest of stages.
We can say that `.i` file is the same as `.c` but without any `#`.
If you try to `#include` file that doesn't exist or making anything wrong with preprocessor directives, it gives pre-processor error but depends on the compiler version.
### B. Compiler:
Compiler takes every file and performs on each `.i` file. 
#### 1. Checking syntax error.
Checks if there is any syntax error like missing semicolon, or using a variable that is not defined, or anything that irrelevant to the language. 
#### 2. Checking prototypes of called functions:
If there is a function that hasn't prototype, this stage will detect this error. 
If there is a function that has a prototype of 3 arguments and you have called it with 2 only, this stage will detect this error
If there is a function that has an implementation error in the syntax, the checking syntax error stage will detect this error.
If there is a function that hasn't implementation at all, the linker will detect this error.

#### 3. Making optimizations:
During compilation, the compiler makes some changes in the way that your code's logic doesn't change. It makes some changes that could make your execution faster, or memory consumption less.
The optimization differs from a compiler to another, Some IDE licenses more expensive than another IDE licenses due to the code optimization level of their compilers.

##### Some of the optimizations that most compilers do:
###### A. Removing dead code, unreachable code
That is low level of optimization should be found in most compilers.

Example of the unreachable code:
```c
int add(x,y) {
	return x+y;
	int z=x*y; // unreachable
}
```
It is unreachable code as you return before executing it, which means it is never executed.
It takes some of the flash but it is not executed so it won't bother the RAM


Example of the dead code:
```c
int add(x,y) {
	int z = x*y; // Dead
	return x+y;
}
```
It is dead as you will never make a use of it. It is worst than unreachable code as it is executed and has no use. It takes some of the flash and also increases the time of execution and stores a variable on the RAM.

###### B. Deploying `register` keyword
If you have a variable that is used a lot in the program so it will add to a register in the CPU automatically, which will help decreasing the execution time.

###### C. `Inline` expansion
It takes all the function that you've written and apply `inline` keyword to them all.
This can be turned off in the IDE's settings

#### 4. Converting C to Assembly
This happens on the `.i` files. After the conversion the output will be `.s`
##### Inside the `.s` file there are 4 tables:
###### Assembly table:
This contains the code written in Assembly.
###### Symbol table:
It contains each symbol and its virtual address. The symbol means the name of any variable or any function. It contains a virtual address as during this stage it doesn't know anything about the real memory that this code will go to as it is the last stage at compilation. Later it will replace those virtual addresses by the real ones.
###### Export table and Import table:
These two tables are essential for the linker as it needs them much during linkage, 
The export table has some info about my functions, my global variables.
The import table has some info about called functions, extern global variables.

### C. Assembler:
The assembler takes in the `.s` files that are assembly files and convert them into object files `.o`, which are binary code (machine code)
and some modern compilers has the assembler as a part of its compiler, so it takes in  `.i` files and outputs out the `.o` files.

### D. Linker:
The linker takes the object files  `.o` and the pre-compiled files.
The pre-compiled files are files that are pre-compiled and inserted in the project, they are used when someone makes a program but he doesn't want to share its source code, so he compiles the program and puts it in the rest of the project so that you can call a function that is in the pre-compiled files but you can not view its implementation.
The linker views the **import table** of each object file, when it finds a function call it searches about its implementation in the **export table** if it finds it fine, if doesn't it searches in the other **export tables**, then searches the pre-compiled files, if it finds it in the pre-compiled files but having `static` keyword before, it refuses the request as it can't be externed.
The linker outputs the **re-allocatable files**. It is considered the exe file but it has only one problem.
The problem is having no real addresses it needs real addresses to every symbol.
So, We need to proceed to the final stage of compilation process.

### E. Locator:
The locator takes in the re-allocatable files and another file called **linker script file**.
The linker script file is a file that contains memory classification in details like how many segments in flash and RAM and the starting address and the ending address of each segment in them and the size of each segment. It is written in a script language.
It outputs the `.exe` file after replacing each virtual address in the **re-allocatable files** with the real addresses. It outputs **map file** that is very helpful for debugging and not all the IDE's outputs it.
**In Most compilers the locator is a part from the linker**



# Task
What is the output of this code?
```c
#include <stdio.h>

int fun() {
	static int num = 16;
	return num--;
}

int main() {
	for(fun(); fun(); fun())
		printf("%i ", fun());
	
	return 0;
}
```

The output of this code will be `14 11 8 5 2`

`for(fun(); fun(); fun())` - this is a for loop with three parts
- **Initialization**: `fun()` called → returns 16, `num` becomes 15
- **Condition**: `fun()` called → returns 15 (truthy), `num` becomes 14
- **Loop body**: `printf("%d ", fun());` → returns 14, `num` becomes 13, prints "14 "
- **Update**: `fun()` called → returns 13, `num` becomes 12
- **Condition**: `fun()` called → returns 12 (truthy), `num` becomes 11
- **Loop body**: `printf("%d ", fun());` → returns 11, `num` becomes 10, prints "11 "
- **Update**: `fun()` called → returns 10, `num` becomes 9
- **Condition**: `fun()` called → returns 9 (truthy), `num` becomes 8
- **Loop body**: `printf("%d ", fun());` → returns 8, `num` becomes 7, prints "8 "
- **Update**: `fun()` called → returns 7, `num` becomes 6
- **Condition**: `fun()` called → returns 6 (truthy), `num` becomes 5
- **Loop body**: `printf("%d ", fun());` → returns 5, `num` becomes 4, prints "5 "
- **Update**: `fun()` called → returns 4, `num` becomes 3
- **Condition**: `fun()` called → returns 3 (truthy), `num` becomes 2
- **Loop body**: `printf("%d ", fun());` → returns 2, `num` becomes 1, prints "2 "
- **Update**: `fun()` called → returns 1, `num` becomes 0
- **Condition**: `fun()` called → returns 0 (false), loop terminates