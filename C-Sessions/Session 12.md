# **Note 1: " Global VS Local Variables types "**
Variables have many types that differs in 4 main points:
- Scope
- Initial Value
- Memory segment
- Lifetime

### Scope Resolving:
The scope resolving is (or **scope resolution**) is the process by which a programming language determines **which variable, function, or class member to use** when there are multiple definitions with the same name in different scopes (local, global, class, namespace, etc.).
##### In C++:
```cpp
#include <iostream>
using namespace std;

int x = 100;

int main() {
    int x = 50; 
    cout << x << endl;    // Prints 50 (local x)
    cout << ::x << endl;  // Prints 100 (global x, resolved using ::)
}
```

```cpp
class A {
public:
    static int value;
};
int A::value = 10; // The variable value that belongs to the namespace A
```

#### Static Scoping VS Dynamic Scoping:
| Feature              | Static Scoping (Lexical)      | Dynamic Scoping                   |
| -------------------- | ----------------------------- | --------------------------------- |
| **When decided**     | Compile time                  | Runtime                           |
| **How resolved**     | Based on **code structure**   | Based on **call stack**           |
| **Common in**        | C, C++, Java, Python, JS      | Lisp (early), Perl, Bash (partly) |
| **Efficiency**       | Faster (compiler knows ahead) | Slower (runtime lookup needed)    |
| **Example behavior** | Always consistent             | Depends on caller                 |
##### Static Scoping (Lexical Scoping)
In Static scoping (or lexical scoping), definition of a variable is resolved by searching its containing block or function. If that fails, then searching the outer containing block and so on... The compiler looks at **where the variable is defined in the source code**, not where the function is called. This is the **most common scoping rule** in modern languages like **C, C++, Java, Python, JavaScript**.

##### Dynamic Scoping
**In dynamic scoping,** definition of a variable is resolved by searching its containing block and if not found, then searching its calling function and if still not found then the function which called that calling function will be searched and so on... The program looks for the most recent variable definition in the **function call chain**, not in the lexical code structure. C, C++, and Java **do not use dynamic scoping** Some older languages (like **LISP, early versions of BASIC, Perl**) supported it.
Both C and C++ supports only static scoping and doesn’t support dynamic scoping.

```c
# include <stdio.h>
int x = 0;
int f()
{
	return x;
}
int g()
{
	int x = 1;
	return f();
}
int main()
{
	printf("%d", g());
	printf("\n");
}
```
It prints `0` as C only supports static scoping, but if it supports dynamic scoping, it will print `1`.

### Extern Keyword:
Imagine you have 2 source files in the same project, One named as **file1.c** and the other is named as **file2.c**, In the **file1.c** You have declared an `int x=5;` so if you tried to use it in **file2.c**. As the variable is declared with only **file scope**,
So how can we extend its scope to be **Software scope** (accessible through every source file in the same project), this can be done using the `extern` keyword.
```c
//file1.c
#include <stdio.h>

int main () {
	return 0;
}
```

```c
//file2.c
#include <stdio.h>

extern int x;

int main () {
	return 0;
}
```
You can not assign a value during the initialization, **The linker** is responsible for linking all these files during compilation
# **Note 2: " Global VS Local "**

## Local Variable:
- **Scope** 
It scope is only block scope (between the curly brackets). It may be inside main or a function:
```c
	#include <stdio.h
	
	int main() {
		int x;
		
		return 0;
	}
```
or may be inside a bracket:
```c
	#include <stdio.h
	
	int main() {
		{
			int x;
		}
		
		return 0;
	}
```
or may be a function argument:
```c
	#include <stdio.h
	
	void fun(int x) {
		return;
	}
	
	int main() {
		
		return 0;
	}
```

- **Initial Value** is garbage value.
- **Memory segment** is stack.
- **Memory Lifetime** is function runtime
## Global Variable
- **Scope** is file scope.
- **Initial Value** is default value of each datatype.
- **Memory segment** is `.bss` if it is initialized without assignment or with assignment with the default value, and it is `.data` if it is initialized with assignment of a non-default value. 
- **Memory Lifetime** is program runtime.
### Local VS Global
#### Trick 1:
```C
#include <stdio.h>
int x=5;

int main () {
	int x=10;
	printf("%i\n", x);
	return 0;
}
```
In this code the global variable is declared in the `.data` segment as it is assigned into non default value, while the local variable is declared in the `stack` segment, So, when we try to print the variable it takes the smaller scope which means that the output in the above code is `10`.
#### Trick 2:
```C
#include <stdio.h>
int x=5;
void f(void);

int main () {
	int x=10;
	printf("%i\n", x);
	return 0;
}

void f(void) {
	printf("%i\n", x);
}
```
The scope of the function `f` doesn't see the local variable, which means that `f` function doesn't print `10`, it prints `5` instead.
#### Trick 3:
```C
#include <stdio.h>
int x=5;

int main () {
	int x=10;
	{
		x=20;
		printf("%i\n", x);
	}
	printf("%i\n", x);
	return 0;
}
```
It prints `20` two times, global variable is declared in the `.data` segment as it is assigned into non default value, while the local variable is declared in the `stack` segment, then the local variable is overwritten to `20`.
#### Trick 4:
```C
#include <stdio.h>
int x=5;

int main () {
	int x=10;
	{
		int x;
		x=20;
		printf("%i\n", x);
	}
	printf("%i\n", x);
	return 0;
}
```
It prints `20` then `10`, global variable is declared in the `.data` segment as it is assigned into non default value, while the local variable is declared in the `stack` segment, and another local variable is declared in the `stack` segment.
#### Trick 5:
```c
#include <stdio.h>

int x;

int main () {
	int x=x;
	printf("%i\n", x);
	return 0;
}
```
It prints garbage value, global variable is declared in the `.data` segment as it is assigned into non default value, while the local variable is declared in the `stack` segment, this local variable is then assigned into itself so it stores the garbage value again.
In Cpp this differs depending on compiler, and you can use the `::` operator which makes me get the bigger scope.
```c
#include <iostream>

int x;

int main () {
	int x=::x;
	printf("%i\n", x);
	return 0;
}
```
# **Note 3: " Static Global VS Static Local "**

## Static Global Variable
- **Scope** is Software scope.
- **Initial Value** is default value of each datatype.
- **Memory segment** is `.bss` if it is initialized without assignment or with assignment with the default value, and it is `.data` if it is initialized with assignment of a non-default value. 
- **Memory Lifetime** is program runtime.

Static global is a global variable that can't be used in another source file, can't be used with `extern` keyword, static global is the same as global in every characteristics, the only difference is the `extern`. 
## Static Local Variable
- **Scope** is file scope.
- **Initial Value** is zero as it is stored in `.bss` or the `.data`.
- **Memory segment** is `.bss` if it is initialized without assignment or with assignment with the default value, and it is `.data` if it is initialized with assignment of a non-default value. 
- **Memory Lifetime** is program runtime. 

### Trick 1:
**Static datatypes** are created **before** runtime, the only datatype that is created **during** runtime is the **local datatypes**.
```c
#include <stdio.h>
void print (void);
int main()
{
	print();
	print();
	print();
	return 0;
}
void print (void)
{
	int x=0;
	printf("x=%i\n",x);
	x++;
}
```
The output of the code will be
```c
0
0
0
```
Because the local is created every time in the stack above each other, so it traces which variable is nearest to me.

```c
#include <stdio.h>
void print (void);
int main()
{
	print();
	print();
	print();
	return 0;
}
void print (void)
{
	static int x=0;
	printf("x=%i\n",x);
	x++;
}
```
But the output of this code is:
```c
0
1
2
```
As the static local is initialized only once before the runtime and stored in `.bss`

### Trick 2:
This code is a compilation error due to trying to assign a return value of a function that will be executed **during runtime**, while the static datatypes **only executes before the runtime**.
```c
#include <stdio.h>
int print2(void);
void print(void);
int main()
{
	print();
	return 0;
}
void print(void)
{
	static int x=print2(); // Compilation error
	printf("x=%i\n",x);
}
int print2(void)
{
	printf("not ok\n");
	return 5;
}
```

But this code is not.
```c
#include <stdio.h>
int print2(void);
void print(void);
int main()
{
	print();
	return 0;
}
void print(void)
{
	int x=print2();
	printf("x=%i\n",x);
}
int print2(void)
{
	printf("not ok\n");
	return 5;
}
```

We can also assign a constant value to it.
```c
#include <stdio.h>

int main()
{
	int y=10;
	static int x=6; // Valid
	printf("x=%i\n",x);
	return 0;
}
```
# **Note 4: " Important Question "**
**Does the `static` keyword affects the Scope or Lifetime?**
In global it affects the scope, In local it affects the lifetime.
# **Note 5: "  "**
# **Note 6: "  "**
# **Note 7: "  "**
# **Note 8: "  "**
# **Note 9: "  "**