---
layout: post
title: C Arrays in the Heap vs The Stack
published: true
author: Brett Settle
---

I started reading [Understanding and Using C Pointers](https://www.amazon.com/Understanding-Using-Pointers-Techniques-Management/dp/1449344186/ref=pd_lpo_sbs_14_img_0?_encoding=UTF8&psc=1&refRID=KGNVW6ZJWYR5TSCNJBZS) and thought I would write my first blog post about something neat I learned. Even though it's pretty obvious looking back, I hadn't really thought about pointer arithmetic on static vs. dynamic arrays in C.

<!--more-->
If you’ve dealt with arrays in C, you know that arrays are stored as pointers to their first element.  You probably also know that pointers store information about the objects they point to.  In the case of an array, the pointer needs to know which type of object it contains so that it can successfully step through its objects. In the code below, we can use pointer arithmetic or bracket notation to get an item at a specific index in the array.

```c
int arr[5]; //create an array of length 5
*arr = 1; // set the first value in the array to 1
arr[1] = 2; //set the second value to 2
*(arr+2) = 3; //set the third value to 3
// and so on
```

Pointer arithmetic makes sense because arrays are stored as a single block of memory.  For example, `*(arr+2)`  increments the array pointer by two indices, using the `sizeof` function to determine how many bytes in each cell.  Iterating through the array is like a game of Candyland, except you get to cheat and move several spaces at once:

![contiguous array image](https://i.stack.imgur.com/qXo36.png)
###### Image source: [Stack Overflow Question](https://stackoverflow.com/questions/40364357/are-array-element-values-stored-in-one-location-or-in-separate-locations)

An important note about memory in C (and other languages) is that dynamic memory and static memory are stored differently.  When a variable is declared globally or statically, it is placed on the stack, which grows downwards. When a variable is declared dynamically, it is added to the top of the heap, which grows upward.  Both use pointer arithmetic to keep track of where frames start and end.

![memory in c programs](https://www.geeksforgeeks.org/wp-content/uploads/Memory-Layout.gif)
###### Image source: [Memory Layout of C Program](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

When an array is initialized, its elements are added to the stack or heap accordingly.

![c array stack vs heap](http://www.bogotobogo.com/cplusplus/images/pointers2/array_stack_heapA.png)
###### Image source: [C++ Tutorial: Pointers II - 2018](http://www.bogotobogo.com/cplusplus/pointers2_voidpointers_arrays.php)

In order to iterate via pointer arithmetic, arrays are stored as a contiguous block of memory, _in the same order_ whether declared statically or dynamically. That means, despite the memory blocks growing in opposite directions, the first element will always have the lowest memory address, and the last element will always have the highest memory address. The only contradiction is the arrays order in the direction that the memory block grows, and so, where the stack pointer points.

Here's a short block of C code to demonstrate this:

```c
#include <stdio.h>
#include <stdlib.h>

int stack[5];
int stack2;
int main(int argc, char* argv[]){
	printf("Stack[0]: %p\n", stack);
	printf("Stack[1]: %p\n", (stack+1));
	printf("Stack[2]: %p\n", &(stack[2]));

	char comp_s = &stack2 > stack ? '>' : '<';
	printf("Stack2: %p %c Stack[0] %p\n", &stack2, comp_s, stack);

	int *heap = malloc(5 * sizeof(int));
	printf("Heap[0]: %p\n", heap);
	printf("Heap[1]: %p\n", (heap+1));
	printf("Heap[2]: %p\n", &(heap[2]));

	int *heap2 = malloc(sizeof(int));
	char comp_h = heap2 > heap ? '>' : '<';
	printf("Heap2: %p %c Heap[0] %p\n", heap2, comp_d, heap);

	free(heap2);
	free(heap);

	/*
	Output: addresses will differ for you
	Stack[0]: 0x10e186030
	Stack[1]: 0x10e186034
	Stack[2]: 0x10e186038
	Stack2: 0x10e186044 > Stack[0] 0x10e186030

	Heap[0]: 0x7f7f34c027d0
	Heap[1]: 0x7f7f34c027d4
	Heap[2]: 0x7f7f34c027d8
	Heap2: 0x7f7f34c00370 < Heap[0] 0x7f7f34c027d0
	*/

	return 0;
}
```

The static variables are stacked on top of each other with decreasing memory addresses, while the heap variables are declared with increasing memory addresses. However, the order of the items in the array are always stored in the same order.  After all, an array is a single object.  Nevertheless, I thought it was pretty neat to print out the addresses and see it for myself.
