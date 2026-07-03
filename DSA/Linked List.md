It is a linear collection of nodes connected by pointer links accessed via a pointer to the first node of the list. The link pointer in the last node is set to null to mark the list's end.    

| **Features** | **Array**                                       | **Linked List**                                     |
| ------------ | ----------------------------------------------- | --------------------------------------------------- |
| **Strength** | Random Access<br>Less memory needed per element | Dynamic size<br>Faster in insertion and deletion    |
| **Weakness** | Fixed size<br>Slower in insertion and deletion  | Sequential access<br>More memory needed per element |
We will need generic print in this 
## LinkedList.h
```c
#pragma once
#include <stdio.h>
#include <stdlib.h>
#include "genericPrint.h"

typedef struct point{
	void *data;
	DataType datatype;
	struct point *next;
}Node;

void print_list(Node *current);
void push_head(Node **head, void *data, Datatype datatype);
void push_end(Node *current, Node **head, void *data, Datatype datatype);
void* pop_head(Node **head);
void* pop_tail(Node *current, Node **head);
void insert_by_index(Node *current, Node **head, void *data, int idx, Datatype datatype);
int remove_by_index(Node **head, int idx);
```
### First Function:
`void print_list(Node *current);` 
It prints the data of each node in the linked list.

### Second Function:
`void push_head(Node **head, void *data);`
It pushes a new node containing data given to the head of the linked list.

### Third Function:
`void push_end(Node *current, Node **head, void *data);`
It pushes a new node containing data give to the end of the linked list.

### Fourth Function:
`void* pop_head(Node **head);`
It returns the head of the linked list then remove it and assigns the head pointer to the next node.

### Fifth Function:
`void* pop_tail(Node *current, Node **head);`
It returns the tail of the linked list then remove it and assigns the tail pointer to the preceding node.

### Sixth Function:
`void insert_by_index(Node *current, Node **head, void *data, int idx);`
It inserts a new node containing the data given to the linked list at a specific index given.

### Seventh Function:
`int remove_by_index(Node **head, int idx)l`
It removes a specific node at a specific index from the linked list.   

## LinkedList.c
```c
#include "LinkedList.h"

void print_list(Node *head) {
	if(head == nullptr) {
		fprintf(stderr, "Can not print an empty list");	
		return;
	}
	
	while(head->next != nullptr) {
		print_generic(head->data, head->datatype);
	}
}
```

### First Function:
```c
void print_list(Node *current) {
	if(current == nullptr) {
		fprintf(stderr, "Can not print an empty list");	
		return;
	}
	
	while(current != nullptr) {
		genericPrint(current->data);
		current = current->next;
	}
}

// Usage inside main
int main(void) {
	Node* myList = mullptr;
	print_list(myList);
	
	retrun 0;
}
```
In `main.c`, you will have the linked list stored as a head pointer to it very first element, so we can say that head is the linked list.
The function here receives the linkedlist (head) inside a pointer to node variable named head as well.
If the head is not null, then the linked list is not empty, so I can go on.
It starts by Looping all over all nodes. at each node it prints the data.
The loop will stops only when you have finished the linked list which means it will only stop when you have reached null.
The data can be `int`, `short`, `double`, `char`, etc. , and this makes a problem with the format specifiers in `printf` function, so one solution is using a user-defined function that is overloaded with each `printf` of each type.
To move to the next node you have to make the head pointer points to the next of it.

### Second Function:
```c
void push_head(Node **head, void *data, Datatype datatype) {
	Node* newNode = (Node *) malloc(sizeof(Node));
	if(newNode == nullptr) {
		return;
	}
	newNode->data = data;
	newNode->datatype = datatype;
	newNode->next = *head;
	*head = newNode;
}

int main(void) {
	Node* myList = nullptr;
	push_head(&myList, 10, 2);
	
	return 0;
}
```
First it creates a new node with the data given, then it will make its next pointer points to the same address the head pointer points to then makes the head pointer points to it.

#### Why are we passing the head as a pointer to pointer?
As we desire to change it, and in C, everything is passed **by value** (as a copy). If you only pass a single pointer (`node *current`), the function creates a local copy of that memory address. Any changes you make to that local pointer inside the function are completely lost when the function returns.

#### What happens if you use a single pointer (`node *current`)?
Let’s look at what goes wrong if the function signature was written like this:
```c
void bad_push_head(node *current, void *val);
```
1. In `main`, you have a variable `node *my_list = NULL;`.
2. When you call `bad_push_head(my_list, val);`, the compiler creates a **copy** of `my_list` and names it `head` inside the function.
3. Inside the function, you allocate a new node and point `current = new_node;`.
4. **The Problem:** Only the local copy `head` shifts to look at the new node. The original pointer back in `main` (`my_list`) is still sitting there pointing to `NULL`. When the function finishes, the local `head` is destroyed, and you've just leaked your new node's memory!

To fix this, you don't pass the value of the head pointer; you pass the **exact memory location where the head pointer itself lives**.
Here is how the double pointer operates step-by-step:
```c
void push_head(node **head, void *data, Datatype datatype) {
}
```
- **`head_ptr`**: The memory address of the pointer variable in `main`.
- **`*head_ptr`**: Dereferencing it once gives you direct, modifying access to the original pointer in `main`. Assigning `*head_ptr = new_node;` forces the pointer in `main` to permanently shift and point to your newly created head node.

### Third Function:
```c
void push_end(Node *current, Node **head, void *data, Datatype datatype) {
	if(current == nullptr) {
		push_head(head, data, datatype);
		return;
	}
	
	Node *newNode = (Node *)malloc(sizeof(Node));
	if(newNode == nullptr) {
		return;
	}
	newNode->data = data;
	newNode->datatype = datatype;
	newNode->next = nullptr; 
	
	while(current->next != nullptr) {
		 current = current->next;
	}
	
	current->next = newNode;	
}

int main(void) {
	Node* myList = nullptr;
	push_head(&myList, 10, 2);
	push_end(myList, &myList, 12, 3);
	
	return 9;
}
```
First, if its linked list's head is `nullptr` - linked list is empty - it will delegate its task to (calls)  `push_head()`. 
Otherwise if the linked list is not empty, first it creates a node, then it loops until it finds the last node, when it finds it, it makes its next pointer pointing to the new node, and make the next pointer of the new node pointing to `nullptr`

The `Node *current` is for looping, while `Node **head` is for changing the head (to be able to call `push_head()` for the case that linked list is empty).

#### Why `current` and `head` together?
A good question to be asked here is why do we need `Node *current` if I can dereference `head` one time to get it.

The answer is simple:
I need to move head variable through the loop, so I will be changing the actual head of the linked list, which is inconvenient.
Another approach is to take a copy inside the function like this:
```c
void push_end(Node **head, void *data, Datatype datatype) {
	Node* current = *head;
	
	/*
		The rest of the logic
	*/
}
```
But lol, it doesn't matter in the two approaches you created a variable to store it whether it is inside the function's signature or inside its body.

### Again the golden rule is:
`Node *current` is a copy, often used for traversing.
`Node **head` is the address of the head, used for editing the head position.

### Fourth Function:
```c
void* pop_head(Node **head) {
	if(*head == nullptr) {
		fprintf(stderr, "Can not pop an empty linked list!");
		return nullptr;
	}
	
	*Node temp = *head;
	void *returnValue = temp->data;
	*head = (*head)->next;
	free(temp);
	
	return retrunVallue;
}

int main(void) {
	Node* myList = nullptr;
	push_head(&myList, 19, 2);
	push_end(myList, &myList, 38, 2);
	pop_head(&myList);
	return 0;
}
```
Before anything it checks if the linked list is not empty. If true:
First it needs to make a temporary variable to store the first node (the one to be deleted).
Second it needs to make the head points to the second node instead of the first, and as it needs to change the head pointer so we need pointer to pointer to head `Node **head`.
Third it needs to return the data of the node to be deleted.
Fourth it needs to deallocate the node to be deleted.

### Fifth Function:
```c
void* pop_tail(Node *current, Node **head) {
	if(*head == nullptr) {
		fprintf(stderr, "Can not pop an empty linked list!");
		return nullptr;
	}
	
	while(current->next->next != nullptr) {
		current = current->next; 
	}
	void *returnValue = current->next->data;
	free(current->next);
	current->next = nullptr;
}

int main(void) {
	
	return 0;
}
```
Before anything it checks if the linked list is not empty, if true:
It checks if there is only 1 node, if true:
it needs to call `pop_head()`, else:
it keeps looping till the node before the last node, then it needs to make a temporary variable to store the last node's data, then make the next pointer of the node before the last node point to `nullptr`, then deallocate the last node, then return the temporary variable.
 
### Sixth Function:
```c
v
```
`01:00:00 in DS 1 part 2`