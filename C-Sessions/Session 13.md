# Pre-processors directives
The pre-processor replaces every `#` function with text that is equivalent for that order, and after that the compilation process continues.
As an example, It will replace the `#include` with the header file it tries to include.

## `#include`
### It has two types:
- **#include <>** :
  - Tells the pre-processor that the file to be included is in the standard libraries (such as string.h)
- **#include ""** :
  - Search for the file to be included in the current directory (user defined library) else search for it in standard libraries.
To be readable we make the user defined between `""`, while the standard libraries are between `<>`.

## `#define`
This pre-processor directives tells the compiler to replace a text with another in all the `.c` file, so it replaces the text during compilation before runtime.
As an example:
```c
#include <stdio.h>

#define x 5                        //  replace x with 5 

int main () {
	int y=x;
	
	printf("%i\n, y);
	printf("%i\n, x);
	
	return 0;
}
```
If you see the `.i` file after the pre-processor stage has finished, you will find the code like this:
```c
#include <stdio.h>              // this line is replaced with the library

int main () {
	int y=5;
	
	printf("%i\n, y);
	printf("%i\n, 5);
	
	return 0;
}
```

#### Trick 1
What is the output of this code?
```c
#include <stdio.h>

#define x 5                        

int main () {
	int x=6;
	
	printf("%i\n, x);
	
	return 0;
}
```
If you tried to view the `.i` file of this code after replacing the `x` with `5`, you will notice that you are trying to assign a rValue with another rValue which is inconvenient.
```c
#include <stdio.h>              // this line is replaced with the library

int main () {
	int 5=6;                    // Compilation error::Syntax stage
	printf("%i\n, 5);
	
	return 0;
}
```

To resolve this issue by regulation, programmers when naming any **macro** they type them in uppercase.
```c 
#define X 5
#define SUM_SIZE 19
```
Anything that is `#defined` is considered a macro, and macros are not considered as variables as they end after the pre-processing while a variable condition is to be accessible to read or write during runtime.
So, `#define` is a pre-processor directive, while `SUM_SIZE` is a macro.

#### Trick 2
What is the output of this code?
```c
#include <stdio.h>

#define X 5                        

int main () {	
	printf("%i\n, X);
	#define X 6
	printf("%i\n", X);
	
	return 0;
}
```
It prints `5` then `6`, as the macros can be redefined.

#### Trick 3
What is the output of this code?
```c
#include <stdioh>

void f(void);

int main () {
	
	f();
	#define X 7
	
	return 0;
}

void f(void) {
	printf("%i\n, x);
}
```
It runs normally and prints out 7, as the pre-processor text replaces all Xs bellow the `#define`.

#### Trick 4
What is the output of this code?
```c
#include <stdio.h>

#define X \
99
int main () {
	printf("%i", X);
	return 0;
}
```
The output will be `99` as the backslash is a continuity operator. It makes the compiler replace the backslash with the line below it. 

Another Example:
```c
#include <stdio.h>

#define X\
56 8

int main () {
    printf("%i", X56);           // prints 8
    return 0;
}
```
The output  of this code is `6`, Notice here that the backslash first replaces the `56` beside the `X` macro so that the whole macro be `X56`.

Another Example:
```c
#include <stdio.h>

#define X\
56 \
8

int main () {
    printf("%i", X56);           // prints 8
    return 0;
}
```
You can notice here that you can use the continuity operator over more than one line.
### A huge difference:
The `const` variable are variables stored in the memory and can be accessed (read) during runtime, while `macro` are not, Macros are replaced by their value as a part of the code.
Macros are preferred if you are trying to use a constant value, as it doesn't consume memory much as const variables, but sometimes it's preferred to use const vars over macros for better readability.

### Advantage of using `#define`:
Imagine having an array of size 5 and you have a source file for some functions for it.
In this code you have 5 functions that process an array, if you want to change your array size, you will have to change 6lines, it may seem easy but in real world applications this will be hard to trace or maintain. 
```c
#include <stdio.h>

void scanArr(unsigned int size, int arr[]);
void printArr(unsigned int size, int arr[]);
int sumArr(unsigned int size, int arr[]);
int maxNumArr(unsigned int size, int arr[]);
int minNumArr(unsigned int size, int arr[]);

int main () {
	int myArr[5]={0};                                      // change this
	scanArr(5, myArr);                                     // change this
	printf(5, myArr);                                      // change this
	printf("Sum is %i", sumArr(5,myArr));                  // change this
	printf("Max number is %i\n", maxNumArr(5,myArr));      // change this
	printf("Min number is %i\n", minNumArr(5,myArr));      // change this
	
	return 0;
}

void scanArr(unsigned int size, int arr[]) {
	for(int i=0; i<size; i++) {
		scanf("%i", &arr[i]);
	}
}

void printArr(unsigned int size, int arr[]) {
	for(int i=0; i<size; i++) {
		printf("%i ", arr[i]);
	}
	printf("\n");
}

int sumArr(unsigned int size, int arr[]) {
	unsigned int sum=0;
	for(int i=0; i<size; i++) {
		sum+=arr[i];
	}
	
	return sum;
}

int maxNumArr(unsigned int size, int arr[]) {
	int max = arr[0];
	for(int i=1; i<size; i++) {
		max = max > arr[i] ? max : arr[i];
	}
	
	return max;
}

int minNumArr(unsigned int size, int arr[]) {
	int min = arr[0];
	for(int i=1; i<size; i++) {
		min = min < arr[i] ? min : arr[i];
	}
	
	return min;
}
```
So, we define a macro with the size of the array and when we need to change the size we only change the macro.
```c
#include <stdio.h>

#define ARR_SIZE 5                                  // only change this

void scanArr(unsigned int size, int arr[]);
void printArr(unsigned int size, int arr[]);
int sumArr(unsigned int size, int arr[]);
int maxNumArr(unsigned int size, int arr[]);
int minNumArr(unsigned int size, int arr[]);

int main () {
	int myArr[ARR_SIZE]={0};                               
	scanArr(ARR_SIZE, myArr);                                
	printf(ARR_SIZE, myArr);                                  
	printf("Sum is %i", sumArr(ARR_SIZE5,myArr));              
	printf("Max number is %i\n", maxNumArr(ARR_SIZE,myArr));     
	printf("Min number is %i\n", minNumArr(ARR_SIZE,myArr));      
	
	return 0;
}

void scanArr(unsigned int size, int arr[]) {
	for(int i=0; i<size; i++) {
		scanf("%i", &arr[i]);
	}
}

void printArr(unsigned int size, int arr[]) {
	for(int i=0; i<size; i++) {
		printf("%i ", arr[i]);
	}
	printf("\n");
}

int sumArr(unsigned int size, int arr[]) {
	unsigned int sum=0;
	for(int i=0; i<size; i++) {
		sum+=arr[i];
	}
	
	return sum;
}

int maxNumArr(unsigned int size, int arr[]) {
	int max = arr[0];
	for(int i=1; i<size; i++) {
		max = max > arr[i] ? max : arr[i];
	}
	
	return max;
}

int minNumArr(unsigned int size, int arr[]) {
	int min = arr[0];
	for(int i=1; i<size; i++) {
		min = min < arr[i] ? min : arr[i];
	}
	
	return min;
}
```
## **DO NOT Define a macro with a semi-colon**
****
# `#undef`
The `#undef` pre-processor directive is made for cancelling out the defined macros.
```c
#include <stdio.h>
#define X 9
int main () {
	printf("%i", X);
	#undef X
	printf("%i", X);               // error X is undefined
	return 0;
}
```

# `#if` & `#else`
## `#if`
The `#if` and `#else` directives in C are **preprocessor directives** used for conditional compilation. Unlike regular `if` statements that execute at runtime, these directives are processed before compilation begins, allowing you to include or exclude entire sections of code based on compile-time conditions.
The basic syntax for conditional preprocessing directives is:
```c
#if condition
    // Code to include if condition is true (non-zero)
#else
    // Code to include if condition is false (zero)
#endif
```
**Important**: Every `#if` must be closed with `#endif`.
### Example 1: Basic Conditional Compilation
```c
#include <stdio.h>

#define DEBUG_MODE 1

int main() {
    printf("Program started\n");
    
#if DEBUG_MODE
    printf("Debug: This is a debug message\n");
#else
    printf("Running in release mode\n");
#endif
    
    printf("Program ended\n");
    return 0;
}
```
**Output when DEBUG_MODE is 1:**
```
Program started
Debug: This is a debug message
Program ended
```
### Example 2: Using Numeric Values
```c
#include <stdio.h>

#define VERSION 2

int main() {
#if VERSION == 1
    printf("This is version 1\n");
#else
    printf("This is version 2 or higher\n");
#endif
    
    return 0;
}
```
Output:
```bash
This is version 2 or higher
```

### Example 3: `#elif` Directive

The `#elif` (else if) directive allows you to chain multiple conditions:
```c
#include <stdio.h>

#define OPTIMIZATION_LEVEL 2

int main() {
#if OPTIMIZATION_LEVEL == 0
    printf("No optimization\n");
#elif OPTIMIZATION_LEVEL == 1
    printf("Basic optimization\n");
#elif OPTIMIZATION_LEVEL == 2
    printf("Advanced optimization\n");
#else
    printf("Unknown optimization level\n");
#endif
    
    return 0;
}
```
Output:
```
Advanced optimization
```
### Example 4: `#ifdef` & `#ifndef`
#### `#ifdef` or `#if defined()`
It stands for **"if defined"** and is a specialized preprocessor directive that checks whether a specific macro has been defined, regardless of its value.
```c
#ifdef MACRO_NAME
    // Code to include if MACRO_NAME is defined
#else
    // Code to include if MACRO_NAME is not defined
#endif
```
The key difference between `#if` and `#ifdef`:
- **`#if MACRO_NAME`**: Checks if `MACRO_NAME` evaluates to a non-zero value
- **`#ifdef MACRO_NAME`**: Checks if `MACRO_NAME` exists at all (has been defined)
#### `#ifndef`
```c
#ifndef DEBUG_MODE
    printf("Debug mode is NOT defined - running in release mode\n");
#else
    printf("Debug mode is defined\n");
#endif
```
****
### Example 5: File guard
You can not include the same header file more than one time in the same source file.
```c
#include <stdio.h>
#include <stdio.h> // error
```
So to resolve this issue, we use file guard.
#### What is header file guard?
Header guards are preprocessor directives that ensure a header file is only included once, even if it's referenced multiple times through different `#include` statements.
```c
#ifndef MAX_H
#define MAX_H
printf("OK");
#endif
```
- **First inclusion**: `MYHEADER_H` is not defined, so the content is processed
- **Subsequent inclusions**: `MYHEADER_H` is already defined, so the preprocessor skips everything until `#endif`
Guard macros typically follow this pattern:
- `FILENAME_H` or `FILENAME_H_`
- `__FILENAME_H__` (though double underscores are reserved)
- Include the full path for uniqueness: `PROJECT_MODULE_FILENAME_H`
##### Modern Alternative: `#pragma once`
```c
// myheader.h
#pragma once

// Your header content goes here
void my_function(void);
int my_variable;
```
****
### Example 6: Platform detection code
```c
#include <stdio.h>

int main() {
    printf("Detected platform: ");
    
#ifdef _WIN32
    printf("Windows\n");
#elif defined(__linux__)
    printf("Linux\n");
#elif defined(__APPLE__)
    printf("macOS\n");
#else
    printf("Unknown\n");
#endif

    return 0;
}
```
Output on a windows system:
```
Detected platform: Windows
```
`_WIN32` : Defined on all Windows systems (both 32-bit and 64-bit).
`_WIN64` : Defined only on 64-bit Windows.
`_MSC_VER` : Defined when using Microsoft Visual C++.
****
`__Linux__` : Defined on Linux systems.
`__unix__` : Defined on Unix-like systems.
`__GNUC__` : Defined when using GCC compiler.
****
`__APPLE__` : Defined on all Apple platforms.
`__MACH__` : Defined on Mach kernel systems.
### What is the advantage of using it?
In LCD, We have two modes to control it using our MCU, The first is named as 8-bit mode which uses all the pins and is better in speed, and the second is named as 4-bit mode which optimizes the number of used pins in the MCU, Each mode of them has a unique code, so rather than loading the code 2 times in the memory, I can make a `#if` condition to switch between them depending on a predefined macro before the runtime.
```c
#define LCD_BITS 4

#if LCD_BITS == 4
// code of 4-bit mode
#else 
// code of 8-bit mode  
#endif
```
### What is the great difference between it and normal conditions?
Normal conditions can check variables or macros, it will be during runtime. But `#if` and `#else` can be used only for macros, it will be before the runtime.
Checking a macro is better in `#if` as it saves flash memory, as it deletes the false condition.
```c
#define DEBUG 1

#if DEBUG
// code of debug mode
#else 
// code of release mode 
// ( will be deleted by the pre-processor as the condition is false ) 
// ( won't be uploaded to flash so it saves memory)
#endif
```
And it also saves the runtime, as the normal conditions wastes some time in checking the conditions.

### **Some Pre-defined Macros:**
```c
#include <stdio.h>
main()
{
    printf("File :%s\n", __FILE__);
    printf("Date :%s\n", __DATE__);
    printf("Time :%s\n", __TIME__);
    printf("Line :%d\n", __LINE__);      // notice the reminder is d
    printf("ANSI :%d\n", __STDC__);      // notice the reminder is d
}
```
The `__FILE__` is a pre-defined macro in the C language that replaces with the path of the source file. 
The `__DATE__` is a pre-defined macro in the C language that replaces the date of the execution.
The `__TIME__` is a pre-defined macro in the C language that replaces the time of the execution.
The `__LINE__` is a pre-defined macro in the C language that replaces the line replacing that macro.
The `__ANSI__` is a pre-defined macro in the C language that replaces `1` if the compiler follows the ANSI C standards, and 0 if not.
```bash
File :test.c
Date :Sep  5 2025
Time :21:26:50
Line :7
ANSI :1
```
You can re-define them and they will work normally.
****
#### **Windows Macros**
##### **`_WIN32`**
- **What it does**: Indicates the code is being compiled for any Windows platform
- **When defined**: On ALL Windows systems (both 32-bit and 64-bit)
- **Use case**: Write Windows-specific code that works on any Windows version
```c
#ifdef _WIN32
    // This runs on Windows XP, 7, 8, 10, 11 (both 32-bit and 64-bit)
    #include <windows.h>
    printf("Running on Windows\n");
#endif
```
##### **`_WIN64`**
- **What it does**: Indicates the code is being compiled specifically for 64-bit Windows
- **When defined**: Only on 64-bit Windows systems
- **Use case**: Code that needs 64-bit specific features or optimizations
```c
#ifdef _WIN64
    // This only runs on 64-bit Windows
    printf("Running on 64-bit Windows - can handle large memory\n");
    // Use 64-bit specific APIs or optimizations
#endif
```
##### **`__CYGWIN__`**
- **What it does**: Indicates compilation under Cygwin (Unix-like environment on Windows)
- **When defined**: When using Cygwin compiler on Windows
- **Use case**: Code that runs on Windows but uses Unix-like APIs through Cygwin
```c
#ifdef __CYGWIN__
    // Unix-like code running on Windows through Cygwin
    #include <unistd.h>  // Unix headers work here
    printf("Running Unix-like code on Windows via Cygwin\n");
#endif
```
****
#### **Linux Macros**
##### **`__linux__`**
- **What it does**: Indicates compilation for Linux operating system
- **When defined**: On any Linux distribution (Ubuntu, CentOS, Debian, etc.)
- **Use case**: Linux-specific system calls, file paths, or features
```c
#ifdef __linux__
    #include <sys/utsname.h>
    printf("Running on Linux\n");
    // Use Linux-specific features like epoll, inotify, etc.
#endif
```
##### **`__gnu_linux__`**
- **What it does**: Indicates compilation for GNU/Linux systems specifically
- **When defined**: On GNU/Linux systems (most common Linux distributions)
- **Use case**: More specific than `__linux__`, ensures GNU C library features
```c
#ifdef __gnu_linux__
    // GNU-specific extensions are available
    #define _GNU_SOURCE
    #include <gnu/lib-names.h>
    printf("Running on GNU/Linux with GNU extensions\n");
#endif
```
****
### **macOS Macros**
##### **`__APPLE__`**
- **What it does**: Indicates compilation for any Apple platform
- **When defined**: On macOS, iOS, iPadOS, watchOS, tvOS
- **Use case**: Apple ecosystem-specific code
```c
#ifdef __APPLE__
    #include <TargetConditionals.h>
    #if TARGET_OS_MAC
        printf("Running on macOS\n");
    #elif TARGET_OS_IPHONE
        printf("Running on iOS device\n");
    #endif
#endif
```
##### **`__MACH__`**
- **What it does**: Indicates the Mach kernel is being used
- **When defined**: On macOS and other systems using Mach kernel
- **Use case**: Mach-specific kernel APIs and featureس
```c
#ifdef __MACH__
    #include <mach/mach.h>
    printf("Mach kernel available - can use Mach APIs\n");
    // Use Mach-specific features like mach ports
#endif
```
****
#### **Unix Macros**
##### **`__unix__`**
- **What it does**: Indicates a Unix or Unix-like operating system
- **When defined**: On Linux, macOS, BSD, Solaris, AIX, etc.
- **Use case**: POSIX-compliant code that works across Unix-like systems
```c
#ifdef __unix__
    #include <unistd.h>
    #include <sys/types.h>
    printf("Running on Unix-like system\n");
    // Use POSIX functions like fork(), exec(), etc.
#endif
```
##### **`__unix`** (without underscores)
- **What it does**: Alternative form of `__unix__`
- **When defined**: Same as `__unix__` but older convention
- **Use case**: Legacy compatibility, same as `__unix__`
****
#### **FreeBSD Macro**
##### **`__FreeBSD__`**
- **What it does**: Indicates compilation for FreeBSD operating system
- **When defined**: On FreeBSD systems
- **Use case**: FreeBSD-specific system calls and features
```c
#ifdef __FreeBSD__
    #include <sys/param.h>
    printf("Running on FreeBSD version %d\n", __FreeBSD_version);
    // Use FreeBSD-specific features like jails, ZFS, etc.
#endif
```
****
#### **Android Macro**
##### **`__ANDROID__`**
- **What it does**: Indicates compilation for Android platform
- **When defined**: When using Android NDK (Native Development Kit)
- **Use case**: Android-specific native code
```c
#ifdef __ANDROID__
    #include <android/log.h>
    __android_log_print(ANDROID_LOG_INFO, "MyApp", "Running on Android");
    // Use Android NDK features
#endif
```
****
#### **Compiler Macros**
##### GCC Macro
###### **`__GNUC__`**
- **What it does**: Indicates compilation with GNU Compiler Collection (GCC)
- **When defined**: When using GCC compiler
- **Use case**: GCC-specific extensions and optimizations
```c
#ifdef __GNUC__
    // Use GCC-specific features
    #define INLINE __attribute__((always_inline)) inline
    #define PACKED __attribute__((packed))
    printf("Compiled with GCC version %d.%d.%d\n", 
           __GNUC__, __GNUC_MINOR__, __GNUC_PATCHLEVEL__);
#endif
```
##### Clang Macro
###### **`__clang__`**
- **What it does**: Indicates compilation with Clang compiler
- **When defined**: When using Clang/LLVM compiler
- **Use case**: Clang-specific features and pragmas
```c
#ifdef __clang__
    // Use Clang-specific features
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Wunused-variable"
    printf("Compiled with Clang version %d.%d.%d\n",
           __clang_major__, __clang_minor__, __clang_patchlevel__);
#endif
```
****
#### **MSVC Macro**
###### **`_MSC_VER`**
- **What it does**: Indicates compilation with Microsoft Visual C++
- **When defined**: When using MSVC compiler
- **Use case**: Microsoft-specific extensions and Windows optimizations
```c
#ifdef _MSC_VER
    // Use MSVC-specific features
    #define FORCE_INLINE __forceinline
    #pragma warning(disable: 4996)  // Disable deprecated function warnings
    printf("Compiled with MSVC version %d\n", _MSC_VER);
#endif
```
****
#### **Intel Macro**
###### **`__INTEL_COMPILER`**
- **What it does**: Indicates compilation with Intel C++ Compiler
- **When defined**: When using Intel's compiler (icc/icl)
- **Use case**: Intel-specific optimizations and vector instructions
```c
#ifdef __INTEL_COMPILER
    // Use Intel-specific optimizations
    #include <immintrin.h>  // Intel intrinsics
    printf("Compiled with Intel Compiler version %d\n", __INTEL_COMPILER);
    // Can use advanced vectorization hints
#endif
```
****
#### **Architecture Macros**
##### x86-64 Macros
###### **`__x86_64__`**
- **What it does**: Indicates 64-bit x86 architecture (AMD64/Intel 64)
- **When defined**: On 64-bit x86 processors (most modern computers)
- **Use case**: 64-bit specific optimizations, memory models
```c
#ifdef __x86_64__
    // 64-bit x86 specific code
    printf("Running on 64-bit x86 architecture\n");
    // Can use 64-bit registers, larger address space
    typedef unsigned long long pointer_sized_int;
#endif
```
###### **`_M_X64`**
- **What it does**: Microsoft's version of x86-64 detection
- **When defined**: On 64-bit x86 with MSVC compiler
- **Use case**: Same as `__x86_64__` but Microsoft-specific
```c
#ifdef _M_X64
    // MSVC on 64-bit x86
    printf("Running on 64-bit x86 (MSVC)\n");
    // Use Microsoft-specific 64-bit intrinsics
#endif
```
##### x86-32 Macros
###### **`__i386__`**
- **What it does**: Indicates 32-bit x86 architecture (i386/i686)
- **When defined**: On 32-bit x86 processors
- **Use case**: 32-bit specific code, memory limitations
```c
#ifdef __i386__
    // 32-bit x86 specific code
    printf("Running on 32-bit x86 architecture\n");
    // Limited to ~4GB address space
    typedef unsigned int pointer_sized_int;
#endif
```
####### **`_M_IX86`**
- **What it does**: Microsoft's version of 32-bit x86 detection
- **When defined**: On 32-bit x86 with MSVC compiler
- **Use case**: Same as `__i386__` but Microsoft-specific
##### ARM64 Macros
###### **`__aarch64__`**
- **What it does**: Indicates 64-bit ARM architecture (ARM64/AArch64)
- **When defined**: On 64-bit ARM processors (Apple M1/M2, newer phones)
- **Use case**: ARM64-specific optimizations, NEON instructions
```c
#ifdef __aarch64__
    #include <arm_neon.h>  // ARM NEON SIMD instructions
    printf("Running on 64-bit ARM (AArch64)\n");
    // Use ARM64-specific features like crypto extensions
#endif
```
###### **`_M_ARM64`**
- **What it does**: Microsoft's version of ARM64 detection
- **When defined**: On ARM64 with MSVC compiler (Windows on ARM)
- **Use case**: Same as `__aarch64__` but Microsoft-specific
##### ARM32 Macros
###### **`__arm__`**
- **What it does**: Indicates 32-bit ARM architecture
- **When defined**: On 32-bit ARM processors (older phones, embedded systems)
- **Use case**: ARM32-specific code, embedded system optimizations
```c
#ifdef __arm__
    printf("Running on 32-bit ARM\n");
    // Use ARM32 assembly or optimizations
    // Common in embedded systems and older mobile devices
#endif
```
###### **`_M_ARM`**
- **What it does**: Microsoft's version of 32-bit ARM detection
- **When defined**: On ARM32 with MSVC compiler
- **Use case**: Same as `__arm__` but Microsoft-specific
#### **Practical Example Using Multiple Macros**
Here's how you might combine these macros in real code:
```c
#include <stdio.h>

void print_system_info() {
    // Operating System
    printf("Operating System: ");
#ifdef _WIN32
    printf("Windows");
    #ifdef _WIN64
        printf(" (64-bit)");
    #else
        printf(" (32-bit)");
    #endif
#elif defined(__APPLE__)
    printf("macOS/iOS");
#elif defined(__linux__)
    printf("Linux");
#elif defined(__FreeBSD__)
    printf("FreeBSD");
#elif defined(__unix__)
    printf("Unix-like");
#else
    printf("Unknown");
#endif
    printf("\n");

    // Compiler
    printf("Compiler: ");
#ifdef __GNUC__
    printf("GCC %d.%d.%d", __GNUC__, __GNUC_MINOR__, __GNUC_PATCHLEVEL__);
#elif defined(__clang__)
    printf("Clang %d.%d.%d", __clang_major__, __clang_minor__, __clang_patchlevel__);
#elif defined(_MSC_VER)
    printf("MSVC %d", _MSC_VER);
#elif defined(__INTEL_COMPILER)
    printf("Intel Compiler %d", __INTEL_COMPILER);
#else
    printf("Unknown");
#endif
    printf("\n");

    // Architecture
    printf("Architecture: ");
#if defined(__x86_64__) || defined(_M_X64)
    printf("x86-64 (64-bit)");
#elif defined(__i386__) || defined(_M_IX86)
    printf("x86 (32-bit)");
#elif defined(__aarch64__) || defined(_M_ARM64)
    printf("ARM64 (64-bit)");
#elif defined(__arm__) || defined(_M_ARM)
    printf("ARM (32-bit)");
#else
    printf("Unknown");
#endif
    printf("\n");
}

int main() {
    print_system_info();
    return 0;
}
```
Output on my system:
```
Operating System: Windows (64-bit)
Compiler: GCC 14.2.0
Architecture: x86-64 (64-bit)
```
Each macro serves as an automatic "flag" that tells your program exactly what environment it's running in, allowing you to write code that adapts automatically to different platforms, compilers, and architectures!
****
# `#error` and `#warning`
In C programming, `#error` and `#warning` are preprocessor directives used to generate compile-time messages. 
They're particularly useful for conditional compilation (printing out errors under certain conditions using `#if`, debugging, and ensuring proper build configurations.

The `#error` directive causes the preprocessor to emit an error message and halt compilation immediately when encountered.
```c
#error message
```
The message doesn't need quotes and continues to the end of the line. When the preprocessor encounters `#error`, it:
- Outputs the specified error message
- Terminates compilation (the program won't compile)
- Returns a non-zero exit code
```c
#include <stdio.h>

int main() {
	
	#ifndef F_CPU
		#error "CPU Frequency is not defined!"
		#define F_CPU 1500
	#endif
	
	printf("OK\n");
}
```
The code does not continue but prints out ok, due to the `#error` that stops the compilation.
****
```c
#include <stdio.h>

int main() {
	
	#ifndef F_CPU
		#warning "CPU Frequency is not defined!"
		#define F_CPU 1500
	#endif
	
	printf("OK\n");
}
```
The code continues and prints out ok, despite having a `warning` message below.
#### Note: *It does not check the normal conditions*
As the `#error` and `#warning` are pre-processor directives, they are dealt with the pre-processor, and the pre-processor doesn't check the normal if conditions.
```c
#include <stdio.h>
#define X 5
int main() {
	
	if(7==X) 
		#error "X is not 5"
	
	printf("OK\n");
}
```
In this code it won't compile as it gives an error of "X is not 5", despite being inside an if condition, this happens as the pre-processor does not check the normal if condition.
****
## Does it need double quotes or not?
```c
#error This is an error message
#error "This is also an error message"
#warning TODO: fix this later  
#warning "TODO: fix this later"
```
All four examples above work identically.
### When does quotes matter?
#### 1. Messages with special characters:
```c
// These work fine without quotes
#error Cannot find required header file
#warning Using deprecated function

// But quotes can make complex messages clearer
#error "Expected format: CONFIG_SIZE=1024, got something else"
#warning "Performance note: O(n²) algorithm in use"
```
#### 2. Empty or whitespace-only messages:
```c
#error          // Works, but produces empty/minimal message
#error ""       // Also works, clearer intent
```
#### 3. Messages that look like preprocessor directives:
```c
// Potentially confusing without quotes
#error #define was not properly set

// Clearer with quotes  
#error "#define was not properly set"
```
****
## Style Recommendations
Most codebases follow one of these patterns:
#### Style 1: Never use quotes **(common in system headers)**
```c
#error Platform not supported
#warning This feature is experimental
```
#### Style 2: Always use quotes **(common in application code)**
```c
#error "Platform not supported"
#warning "This feature is experimental"
```
#### Style 3: Use quotes for **complex messages only**
```c
#error Platform not supported
#error "Expected CONFIG_VERSION >= 2, got CONFIG_VERSION = 1"
```
# `#pragma`
The `#pragma` directive in C is a preprocessor directive that provides a way to give special instructions to the compiler. Unlike other preprocessor directives that have standardized behavior, `#pragma` directives are largely implementation-specific, meaning different compilers may support different pragmas or interpret them differently.
So, It has some problems like portability as it may run normally on a compiler and on a another compiler it doesn't.
## Standard Pragmas
### `#pragma once`
It is an alternative for the file guard, In VS the file guard is done by `#pragma once`, In Codeblocks the file guard is done by `#ifndef`.
### `#pragma STDC`
This family of pragmas controls certain aspects of the C standard library behavior:
- `#pragma STDC FP_CONTRACT ON/OFF/DEFAULT` - Controls whether floating-point expressions can be contracted (like fused multiply-add operations)
- `#pragma STDC FENV_ACCESS ON/OFF/DEFAULT` - Indicates whether the program accesses the floating-point environment
- `#pragma STDC CX_LIMITED_RANGE ON/OFF/DEFAULT` - Informs the compiler about the range of complex arithmetic operations
## Common Compiler-Specific Pragmas
### GCC Pragmas:
- `#pragma GCC diagnostic` - Controls warning and error messages
- `#pragma GCC optimize` - Specifies optimization options for functions
- `#pragma GCC push_options` / `#pragma GCC pop_options` - Save and restore compiler options
### Microsoft Visual C++ Pragmas:
- `#pragma once` - Ensures a header file is included only once (widely supported beyond MSVC)
- `#pragma warning` - Controls compiler warnings
- `#pragma pack` - Controls structure member alignment
- `#pragma comment` - Embeds comments or directives into object files
### Alignment and Packing:
```c
#pragma pack(1)        // Pack structures with 1-byte alignment
struct packed_struct {
    char a;
    int b;
};
#pragma pack()         // Restore default packing
```
## Memory and Optimization Pragmas
### OpenMP Pragmas:
```c
#pragma omp parallel for
for(int i = 0; i < n; i++) {
    // Parallel execution
}
```
### Loop Optimization:
```c
#pragma unroll 4       // Unroll loop 4 times (compiler-specific)
#pragma vector         // Hint for vectorization
```
## Warning Control
```c
#pragma warning(push)           // Save current warning state
#pragma warning(disable: 4996)  // Disable specific warning
// Code that generates the warning
#pragma warning(pop)            // Restore warning state
```
## Function-Specific Pragmas
```c
#pragma GCC push_options
#pragma GCC optimize ("O3")
void optimized_function() {
    // This function compiled with -O3
}
#pragma GCC pop_options
```
## Memory Sections
```c
#pragma section(".mydata", read, write)
#pragma data_seg(".mydata")
int special_var = 42;
#pragma data_seg()
```
## Unknown Pragma Handling
The C standard requires that unknown pragmas be ignored rather than causing compilation errors. This allows for portable code that uses compiler-specific optimizations when available:
```c
#pragma unknown_pragma_name
// This won't cause an error, just gets ignored
```
## Best Practices
1. **Use sparingly** - Pragmas make code less portable
2. **Document thoroughly** - Always explain why a pragma is needed
3. **Conditional compilation** - Use `#ifdef` to check compiler support
4. **Prefer standard alternatives** when available
```c
#ifdef _MSC_VER
    #pragma warning(disable: 4996)
#elif defined(__GNUC__)
    #pragma GCC diagnostic ignored "-Wdeprecated-declarations"
#endif
```
## Portability Considerations
Since pragmas are largely compiler-specific, code using them may not be portable. When writing portable code, it's often better to use compiler-agnostic approaches like function attributes or command-line compiler options when possible.

The `#pragma` directive is a powerful tool for fine-tuning compiler behavior, but it should be used judiciously to maintain code portability and readability.
# `#line`
The `#line` directive in C is a preprocessor directive that allows you to change the line number and optionally the filename that the preprocessor associates with subsequent lines of code. This is primarily used for debugging, error reporting, and code generation tools.
###### The `#line` directive affects the values of two predefined macros:
- `__LINE__` - Contains the current line number
- `__FILE__` - Contains the current filename
```c
#line 51
#line 51 "origin.c"
```

```c
#include <stdio.h>

int main() {
    printf("Line: %d, File: %s\n", __LINE__, __FILE__);  // Line: 4
    
#line 100
    printf("Line: %d, File: %s\n", __LINE__, __FILE__);  // Line: 100
    
#line 200 "custom.c"
    printf("Line: %d, File: %s\n", __LINE__, __FILE__);  // Line: 200, File: custom.c
    
    printf("Line: %d, File: %s\n", __LINE__, __FILE__);  // Line: 201, File: custom.c
    
    return 0;
}
```
Output:
```
Line: 4, File: main.c
Line: 100, File: main.c
Line: 200, File: custom.c
Line: 202, File: custom.c
```
## Why do we use it?
**Error Mapping:** 
When tools generate C code from other languages or templates, they use `#line` directives to map errors back to the original source:
```c
// Generated from template.txt line 15
#line 15 "template.txt"
int generated_function() {
    return 42;  // If error occurs here, debugger shows template.txt:16
}
```

```c
// Line 1 of actual file: comment
// Line 2 of actual file: #line directive
#line 15 "template.txt"           // Resets to line 15, file "template.txt"
int generated_function() {        // Now considered line 15 of "template.txt"
    return 42;                    // Now considered line 16 of "template.txt"
}                                 // Now considered line 17 of "template.txt"
```

If there's a compilation error on the `return 42;` line, instead of seeing:
```
generated.c:4: error: some error message
```
You will see this:
```
template.txt:16: error: some error message
```
### `gcc -E main.c`
The `gcc -E main.c` command runs **only the C preprocessor** on the file `main.c` and outputs the result to stdout. It stops before compilation, assembly, or linking.
#### What the Preprocessor Does
The `-E` flag tells gcc to:
1. **Process all preprocessor directives** (`#include`, `#define`, `#ifdef`, etc.)
2. **Expand all macros**
3. **Include all header files**
4. **Remove comments** (unless you use `-C` to preserve them)
5. **Insert `#line` directives** to track original source locations
6. **Output the final preprocessed C code**

Example code:
```c
#include <stdio.h>
#define MAX_SIZE 100
#define SQUARE(x) ((x) * (x))

int main() {
    int size = MAX_SIZE;
    printf("Square of 5 is %d\n", SQUARE(5));
    return 0;
}
```
Command:
```bash
gcc -E main.c
```
Output (simplified excerpt):
```c
# 1 "main.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<built-in>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<built-in>" 2
# 1 "main.c"
# 1 "/usr/include/stdio.h" 1 3 4
# 27 "/usr/include/stdio.h" 3 4
# 1 "/usr/include/_ansi.h" 1 3 4
...
[hundreds of lines from stdio.h and related headers]
...
# 2 "main.c" 2

int main() {
    int size = 100;
    printf("Square of 5 is %d\n", ((5) * (5)));
    return 0;
}
```
What You'll See in the Output:
```c
# 1 "main.c"
# 1 "/usr/include/stdio.h" 1 3 4
```
These are `#line` directives showing which file each section came from.
**Macro Expansions:**
- `MAX_SIZE` becomes `100`
- `SQUARE(5)` becomes `((5) * (5))`
**Header File Contents:** 
All the function declarations, type definitions, and macros from `stdio.h` and its dependencies.
**Comments Removed:** 
All comments are stripped out (unless you use `-C` flag).
##### Useful Variations
**Save to file:**
```bash
gcc -E main.c > main.i
```
**Preserve comments:**
```bash
gcc -E -C main.c
```

**Only show your code (skip headers):** This is trickier, but you can search for your filename in the output.
##### Common Use cases:
**1. Debugging Macro Expansions:** See exactly how complex macros expand.
**2. Understanding Include Dependencies:** See which headers are being included and in what order.
**3. Checking Preprocessor Logic:** Verify that `#ifdef` conditions work as expected.
**4. Analyzing Complex Header Issues:** When you get mysterious compilation errors, `-E` helps you see the final code the compiler actually processes.

The preprocessed output can be **very large** (often thousands of lines) because it includes all the header files. A simple "Hello World" program might generate 500+ lines of preprocessed code due to the stdio.h inclusion.
This is why preprocessing is kept as a separate step - it shows you exactly what the compiler sees after all the preprocessor magic is done.
# Parametrized Macros (Macro like a function)
Parametrized macros in C are preprocessor macros that accept arguments, allowing them to behave like functions but with important differences. They're defined using `#define` with parameters and are expanded by the preprocessor before compilation.
*Basic Syntax:*
```c
#define MACRO_NAME(param1, param2, ...) replacement_text
```
## Example 1:
```c
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define PRINT_VAR(var) printf(#var " = %d\n", var)
```
## Example 2:
```c
#include <stdio.h>

#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))

int main() {
    int num = 5;
    printf("Square of %d is %d\n", num, SQUARE(num));
    printf("Max of 10 and 20 is %d\n", MAX(10, 20));
    return 0;
}
```
After Pre-Processing it will look like:
```c
int main() {
    int num = 5;
    printf("Square of %d is %d\n", num, ((num) * (num)));
    printf("Max of 10 and 20 is %d\n", ((10) > (20) ? (10) : (20)));
    return 0;
}
```
## Example 3:
**Text Replacement:** Macros perform literal text substitution, not function calls:
```c
#define DOUBLE(x) (2 * x)
int result = DOUBLE(3 + 4);  // Expands to: (2 * 3 + 4) = 10, not 14!
```
**No Type Checking:**
```c
#define SQUARE(x) ((x) * (x))
int i = SQUARE(5);      // Works with int
float f = SQUARE(2.5);  // Works with float  
char c = SQUARE('A');   // Even works with char!
```
**No Function Call Overhead:** Code is inserted inline, so there's no function call/return overhead.
## Example 4:
**Stringification (`#`):**
Converts macro parameters to string literals:
```c
#define PRINT_VAR(var) printf(#var " = %d\n", var)

int age = 25;
PRINT_VAR(age);  
	// Expands to: printf("age" " = %d\n", age);
    // Which becomes: printf("age = %d\n", age);
```
## Example 5:
Usage in Embedded Systems:
```c
#define REG_SIZE 8

#define SET_BIT(reg, bit) (reg |= (1 << (bit)))
#define CLR_BIT(reg, bit) (reg &= ~(1 << (bit)))
#define TOG_BIT(reg, bit) (reg ^= (1 << (bit)))
#define READ_BIT(reg, bit) ((reg & (1u << (bit))) >> (bit))

#define ROL(reg, number) (reg = ((reg << (number)) | (reg >> (REG_SIZE - (number)))))
#define ROR(reg, number) (reg = ((reg >> (number)) | (reg << (REG_SIZE - (number)))))
```

## Advantages:
The biggest Advantage of micro-like a function is that it don't need to perform context switching so it is faster.

## Disadvantages:
- The biggest Disadvantage of it is the flash code size.
- The second biggest Disadvantage is that we can not debug it.

## Best Practice
If the code of the function is large, you should use normal functions, otherwise use macro like a function.
If you are using a function inside a loop, use macro like a function as by text replacement it will be replaced only one time. Which means that the flash memory is reserved and also the execution time is reserved (due to no context switching).

# Token Concatenation Operator
It only works in macro like a function. It concatenates any parameters.
```c
#include <stdio.h>
#define concat(a, b) a##b

int main(void)
{
    int xy = 30;
    printf("%i", concat(x,y)); 
    // expands to -> printf("%i", xy);
    return 0;
}
```
Output:
```
30
```