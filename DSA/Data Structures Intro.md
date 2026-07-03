This part is a documentation for the data structures session in c. We are covering only the linear data structures.
A Data Structure is a collection of data stored in memory in a certain way and accessed in memory through a certain way. Each data structure has its ways of accessing data and storing them.

Array is one of the basic data structures known in C. It is stored in memory in a contiguous way, and it is accessed in random way (random access (sequentially)).

We have two implementations for each data structure, one is array implementation and the other is the linked list implementation.
Array Stack implementation differs from the normal array in accessing memory, but the storing is the same. The same rule applies to Linked Stack.

I am not going to explain the data structures logic from scratch, I am just migrating from data structures in Cpp to data structures in C. 

We will use generic with `void *` and also generic prints
## genericPrint.h
```c
#pragma once
#include <stdio.h>

typedef enum {
	TYPE_BYTE,
	TYPE_SHORT,
	TYPE_INT,
	TYPE_LONG,
	TYPE_LONG_LONG,
	TYPE_FLOAT,
	TYPE_DOUBLE,
	TYPE_LONGDOUBLE,
	TYPE_CHAR,
	TYPE_STRING
}Datatype;

void genericPrint(const void *data, Datatype type) {
    if (data == NULL) {
        printf("(null)\n");
        return;
    }

    switch (type) {
        case TYPE_BYTE:
            // Printed as a hex byte or raw integer value
            printf("0x%02X\n", *(const unsigned char*)data);
            break;
            
        case TYPE_SHORT:
            printf("%d\n", *(const short*)data);
            break;
            
        case TYPE_INT:
            printf("%d\n", *(const int*)data);
            break;
            
        case TYPE_LONG:
            printf("%ld\n", *(const long*)data);
            break;
            
        case TYPE_LONG_LONG:
            printf("%lld\n", *(const long long*)data);
            break;
            
        case TYPE_FLOAT:
            printf("%f\n", *(const float*)data);
            break;
            
        case TYPE_DOUBLE:
            printf("%lf\n", *(const double*)data);
            break;
            
        case TYPE_LONGDOUBLE:
            printf("%Lf\n", *(const long double*)data);
            break;
            
        case TYPE_CHAR:
            printf("%c\n", *(const char*)data);
            break;
            
        case TYPE_STRING:
            // For strings, data itself IS the pointer to the characters
            printf("%s\n", (const char*)data);
            break;
            
        default:
            fprintf(stderr, "Error: Unknown data type token.\n");
            break;
    }
}
```