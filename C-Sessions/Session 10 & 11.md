# **Note 1: " GOTO "**
If I want the code to jump from a part to another part I can use the `goto` keyword.
It acts like a teleporting for the compiler, It needs a label in the beginning of the line that we want the compiler to teleport to.
```c
#include <stdio.h>

int main() {
	double cash;
L1: printf("Enter the cash"); //that L1: is a label for the goto keyword
	scanf("%lf", cash);
	getchar(); //eats the '\n'
	if(cash<0) 
		goto L1;
	return 0;
}
```

It is not recommended to use `goto` as it makes it not maintainable or even readable it is really hard to trace, you can use loops instead.
`goto` is compiled into `jump` assembly order, where `jump` jumps to an address which is the label, after it jumps to that address and executes it, the program counter starts to act sequentially again.

# **Note 2: " Functions Pros & Cons"**
The main advantage of using a function is that it saves memory, If I have a sum function like this:
```c
int getSum() {
	int num1, num2;
	printf("Enter two Numbers to be summed");
	scanf("%i%i", &num1, &num2);
	return num1+num2;
}
```
if this code takes 60 bytes from the flash, then if I don't use the function and used the whole code block 3 times, I will consume 180 bytes for it, while using the function I only use the 60 and jump to them when I need it, then go back.

So Functions save memory, but it decreases the execution time, as the processor has to stop execution then jump to where the function is stored then starts to execute again, then stops again to get back to the rest of the code to complete sequentially

## The name of the function is the address of the first instruction in the flash.

# **Note 3: " Start up code "**
What calls the main function is the start-up code, In micro-controllers, the manufacturing company is the one that writes the start up code. When you burn an executable to the controller it gets combined with the startup code. It makes some initialization tasks, and the last task it does that it loads the address of the main function in the program counter.
The startup code operates when you run the code, not build it.
## Run on codeblocks or VScode is equivalent to giving power to the micro-controller. 

# **Note 4: " MISRA C "**
MISRA C is a set of rules, made to make sure that your c language code is fully reliable, it is made for the automotive cars industry specially, One of its rules that your function has only one return and the return is the last statement of your code.

****
# **Note 5: " Modularity "** 
Modularity is dividing your code into many source files and header files, Imagine we are making a project that we use LCD and Keypad in it, after we have understood what is LCD and keypad and how do they interface, We need to write a driver for them, we can make source and header file for the LCD, and we can make also source and header file for the keypad, If we didn't do that, all the code for everything will be in the main file which is spaghetti code.
This makes the code **unreadable**, **un-reusable**, **unmaintainable** and **unreliable**, So we need to separate the code of each module.
## What is the difference between header files and source files?
**Header files (.h/.hpp)** are for the declarations like (function prototypes, class/struct definitions, constant/macros definitions, Inline functions sometimes). They are included in source files using `#include`, They are blueprints or contracts that tell the compiler what exists but not how it works.
**Source files (.c/.cpp)** are for definitions like (actual function implementations, logic/algorithms, variable definitions). They are compiled into object code (.o/.obj) and then linked to form the final program. They are the implementation of the above blueprint.

## What is the difference between <> and "" ?

```c
#include <stdio.h> //standard C files header file
#inlcude "max.h" //User defined header file 
```
The `<>` makes the compiler search inside the standard C header files
The `""` makes the compiler search inside the directory of the project, if doesn't find it then it searches for it inside the standard header files.
So, We can say that `""` can be used for all, but we don't do that to make it more readable.
Anyone that sees `""` will know that it is user defined and `<>` is standard 
# **Note 6: " Printing Octal or Hexa "**
I can use `printf("%o", num)` to print the octal numbers, and `printf("%p", num)` for the hexadecimal but printed as an address so it is formatted to be 8 bits. I can print hexadecimal also in another way which is `printf("%x", num)` which will print the number without formatting it in address form
# **Note 7: " Address Operator "**
When you try to access the memory with an address of `01` , you are typing in the address lines of the processor `01` but the actual RAM memory doesn't have this address **This is a logical address**. What really occurs is that the `01` address is passed to the decoder into only 1 line is ON, and the rest is OFF, and the one line ON is the second one, so we access the second address.
The decoder's inputs are the address lines, and the decoder outputs are the ENABLE pins of each memory line.
![pic11_1](Attachment/Session-11/pic11_1.png)

In micro-controllers, as an example on the **atmega processors** the RAM memory inside it is 2KByte, and the processor interfaces with it in 16 address lines instead of 11 as needed, so the rest is ignored no matter it is zeros or ones.
But in normal computers, the processors has a virtual memory (taking a part from the hard disk and considering it as a RAM) which needs virtual memory address, so the location addresses may be `32 bits`,`48 bits` or `64 bits` and we can check it.
So we can say that in micro-controllers, the processor address lines are related to the RAM memory size, while in normal computer processors it is not related to its size. The address width is determined according to how many address line does the processor interfaces them with the memory.

If I have an `int` variable named x, then it uses 4 bytes of the memory, If I wanted to print its address, so the printed address will be the address of the first byte.

When trying to display the address, do not use `%i`
```c
#include <stdio.h>

int main () {
	int x=50;
	printf("%i", &x);
	return 0;
}
// this will print the decimal value of the address, which is not helping much 
```
you should print it in hexadecimal
```c
#include <stdio.h>

int main () {
	int x=50;
	printf("%p", &x);
	return 0;
}
```
Writing it in decimal will be long and visually very large and hard to read, The hexadecimal saves some space as **1 hex digit = exactly 4 binary bits.**  Hexadecimal is compact, human-friendly, and directly tied to binary, which is why it’s the standard format for addresses.
# **Note 9: " Stack Pointer "**
First we need to know some properties of the local variables like:
- its scope is in between curly brackets.
- Having initialized garbage value.
- Having function runtime lifetime..
- It is named also as automatic variable (it is created automatic, de-allocated automatic).
- It is stored in stack segment.

The stack is a memory segment that has some characters, such as it is L.I.F.O. (Last in First Out) or  F.I.L.O. (First in Last out). It is like putting dishes above each other.
One of the registers of the micro-processor is the **Stack Pointer**, At first the stack pointer is pointing to the bottom of the Stack. The Stack pointer every time I am pushing a value to the stack the pointer decreases by an address, so it can point to the designated memory address. If I am poping a value from the stack, the pointer is incrementing to get the address below.
Sometimes (depending on release or debug) the stack may be inverted so the stack pointer is pointing to the top, what matters is that it is always pushing in a way opposite to poping.
**On debug mode the stack is pointing to bottom and goes upwards. On release mode the stack is pointing to the top and goes downwards.**

# **Note 10: " Context Switching "**
Context switching is the process where the CPU stops running one process, saves its current state, and loads the saved state of another process of interrupt or higher priority  and when it finishes it, it goes back to the task that previously been executed, so that multiple processes can share the CPU effectively. It is essential in multitasking systems where many processes need CPU time.
# **Note 11: " Execution Process "**
The process of executing a code is divided into some stages:
## 1st: Pre-Processor
- It gets input from all the project files.
- It searches in the source files about any orders that begin with `#` ( e.g., `#include`, `#define`, `#ifdef`) (any order begins with `#` is named as preprocessor directive)
- Removes comments from the code.
- Expands macros.
- Includes header files by copying their content into the source file.
- Produces a **modified source code** file that is ready for compilation (often called a _translation unit_).
- It exports a file its extension is `.i` or `.pre` (equivalent to `.c` files) if I have 3 source files then I will have 3 preprocessor files
## 2nd: Compiler & Assembler
### Compiler
- Translates the pre-processed C/C++ code into **assembly language** for the target machine.
- Performs syntax analysis, semantic checks, and optimizations. 
- Ensures code correctness at the language level.
### Assembler
- Converts the assembly code into **machine code (object code)**.
- Produces an **object file** (`.o` or `.obj`) containing binary instructions, but still incomplete (not fully executable).
## 3rd: Linker
It is a program that takes one or more object files generated by a compiler and combines them into a single executable file, library, or another object file. It resolves symbolic references and relocates code and data so the final program can run correctly.


# **Note 12: " Recursion "**
First there is the direct and indirect recursion, The direct is that the function calls itself repeats itself, The indirect is that function 1 calls function 2 and function 2 calls function 1 and so on...
## Recursive Function is restricted in embedded systems, due to:
- **Non determinism**, I can not know how specified memory consumption and execution time.
- **Exit condition bug**, Sometimes if the exit function doesn't operate well, It may go in an infinite loop pushing much functions in the stack till it overflows. 


# **Note 13: " Inline Function "**
Not all compilers support inline functions, the inline function is implemented by putting the `inline` keyword before the implementation, and this makes the compiler before running during compiling remove the code each function calling and puts the implementation of the function instead.
This makes the program lose its saved memory that we talked about earlier at note 2, otherwise, It makes the program has less execution time as it saves the time of pushing and pulling.
If you try to apply this on a recursive function, then the compiler will ignore your request as it is meaningless.