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

An important note about memory in C (and other languages) is that dynamic memory and static memory are stored differently.  When a variable is declared globally or statically, it is placed on the stack, which grows downwards. When a variable is declared dynamically, it is added to the top of the heap, which grows upward.  Both use pointer arithmetic to jump around in memory.

![memory in c programs](https://www.geeksforgeeks.org/wp-content/uploads/Memory-Layout.gif)
###### Image source: [Memory Layout of C Program](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

When a variable is declared, it is pushed onto the stack or heap accordingly. Declaring a static array will push `sizeof(type) * size` bytes onto the stack and return the location of `arr[0]`, which will be the bottom of the stack.  When a dynamic variable/array is declared, the pointer is pushed onto the stack pointing to `nullptr`. Running `malloc` allocates space on the heap and returns the first address of that block.  So, when allocating space for an array on the heap or the stack, the lowest address of the block is always the first element.

In this snippet, I create 2 static arrays, then 2 dynamic arrays, all of length 2;
```c
int a[2] = {1, 2};
int a2[2] = {1, 2};

int *b = malloc(2 * sizeof(int));
int *b2 = malloc(2 * sizeof(int));
```
Here is a visual representation of how memory is being allocated.  Notice that the elements are pushed onto the bottom of the stack in the order they are declared, and pushed onto the top of the heap in the order they are allocated.  However, the array elements are always in the same order, with the first element at the lowest memory address.

![snippet image](https://raw.githubusercontent.com/TeamBabylon/teambabylon.github.io/master/images/C_Arrays_Heap_vs_Stack.png)

In order to iterate via pointer arithmetic, arrays are stored as a contiguous block of memory, _in the same order_ whether declared statically or dynamically. That means, despite the memory blocks growing in opposite directions, the first element will always have the lowest memory address, and the last element will always have the highest memory address.

For a test, I printed out the memory addresses of the variables.
```
a[1]: 0x7ffeec2fb7b4
a[0]: 0x7ffeec2fb7b0
a2[1]: 0x7ffeec2fb7ac
a2[0]: 0x7ffeec2fb7a8
*b: 0x7ffeec2fb790
*b2: 0x7ffeec2fb788
-----------
b2[1]: 0x7ff5a0c027d4
b2[0]: 0x7ff5a0c027d0
b[1]: 0x7ff5a0c00374
b[0]: 0x7ff5a0c00370
```

Looks like everything is laid out as expected.  There is a large gap between b[1] and b2[0], but that is expected with `malloc`.  Most of this is pretty basic knowledge, but I still thought it was interesting, and would make a good first blog post. Thanks for reading!
