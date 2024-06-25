+++
title = "What Is Memory"
date = 2024-06-24T18:58:32-03:00
draft = true
tags = ["memory", "low-level", "programming", "C", "Assembly"]
categories = "programming"
+++

# Memory and pointers for the curious coder…

## Memory, what is it really?

Most programmers have a view of memory as a list of “things” these “things” might be whatever you are working with in your code, and for most purposes that view is fine, but if you want to really understand what is all those terms read on!

## What I assume you know

This article will assume you know programming, more than a simple hello world, but not much beyond flow control and arrays.

For this article I will use C and gas syntax in intel mode (fancy way of saying it has a logical operands order) x86_64 assembly, don’t worry if that sounds scary, all the code examples will be commented to explain what they are doing.

I also assume you know about binary and hexadecimal notation, if you are not sure, check these:

- https://en.wikipedia.org/wiki/Hexadecimal
- https://en.wikipedia.org/wiki/Binary_number

If you care, all code was compiled in GCC 14.1 using the command:
```sh
$ gcc -c $FILE -Wall -Wpedantic -Werror
```

Check [Further reading](#further-reading) at the end of the page for more useful links.

# Pointers, bytes and Turing machines

## Turing machines and memory

I you are familiar with the concept of a Turing machine, it helps to imagine the RAM as the machine’s tape, if you don’t, here’s a brief summary.

A Turing machine is a theoretical machine devised by [Alan Turing](https://en.wikipedia.org/wiki/Alan_Turing), the idea is that this is a machine that has a tape of unlimited length, this tape is divided into cells, such cells can hold a symbol from a finite alphabet of symbols it can write to and read from  each cell, and a finite number of states, it also has a head that at any point in time is over a cell in that tape, it will read the cell’s contents and based on the machine’s current state and the read symbol it will move the head left, right or halt the whole machine. Despite simple this machine is what we refer to as `Turing complete` meaning it can implement and run any algorithm that a computer can.

Memory in this context is that tape, with the difference that in the real world it’s finite, long but finite, however the RA in RAM stands for Random Access, this just means that if the computer wants to look at the cell at the very end of memory it can get that value directly instead of reading the whole tape to get there. The cells in this tape hold a byte each, in this article we will assume the byte is the smallest addressable unit, while that might not be true for all computers it works for this.

# If everything is bytes, what are types?

Types are an abstraction, something that exists in the source code solely for the benefit of the humans looking at it and the compiler making machine code out of it. A `struct` in C is merely a wrapping over a span of memory so the compiler knows the offsets to the “fields” and since it knows their types, it also knows how wide they are. This also means that if there’s some data in RAM, and you know its structure, you can cast a pointer to the start of that structure to a pointer to a struct in C. Now, magically, you have a struct of that type!

In short, only the compiler and you care about types, the hardware does not! It has different instructions for different widths of numbers, and that is why CPUs care about the width of the numbers, but that is the closest it gets to caring about types. Oh, and pointers, it cares if a value is a pointer or a value.

## Pointers

So with that in mind, what is a pointer? You might have heard of pointers as scary complex things that cause bugs and pain to developers, but a pointer is nothing more than taking a number like `10000` and saying: Hey computer, this is an address into memory (that tape from before), look at that please!, you can then tell the computer to grab the number in that address, or put a number there, this is what we call a pointer dereferencing operation, because it takes the or places a value at the location r**eferenced** by the pointer, as simple as that!

Example code in C:

```c
#include <stdio.h>

void sum_a_b_result_in_b(int a,int *b) {
	// if we do instead: b = a + b the compiler complains
	*b = a + *b;
}

int main(void) {
    int a = 1;
    int b = 2;

    sum_a_b_result_in_b(a, &b);

    printf("%d\n", b);
    return 0;
}

```

```c
#include <stdio.h>

void sum_a_b_result_in_b(int a,int b) {
	// if we do instead: b = a + b the compiler complains
	b = a + b;
}

int main(void) {
    int a = 1;
    int b = 2;

    sum_a_b_result_in_b(a, &b);

    printf("%d\n", b);
    return 0;
}

```

# Why we count arrays from 0?

Have you ever asked yourself why, with very few exceptions arrays start at 0? Spoiler: It has to do with pointers. 

For this let’s define an array, from the low level programmer’s point of view everything can be seen as being: bytes, pointers or a contiguous runs of bytes (span), an array is a span, an array in most compiled languages (with some variation as to how they store their length information) is a pointer, and the nice thing with pointers is, they are just a number that the computer is told to treat as an address in RAM, this means you can do maths on them!

Lets start with basics:

```s
some_array:
.8byte 10 ; This allocates 10*8 bytes of memory

save_rdi:
mov r8, [some_array + 8 * 8] ; This is a pointer in assembly, this will save r8 in some_array[8]
mov r9, [some_array] ; This is equivalent of saying some_array + 0 * 8
```

Based on the code above, you can then see that when we index into lists in programming, when you write `some_array[0]` ,`some_array[1]` or `some_array[i] where i is an integer` you are in effect saying the same as `*(some_array + i * sizeof(type))` , now if you never had to worry about memory before, you might be asking, what do you mean `sizeof(type)`?

Well, it just so happens that since the byte is the smallest addressable unit, pointers hold the address of an individual byte, this is fine if your number is a byte, however since one byte can only hold 256 distinct values `0 to (2^8) - 1` any numbers larger than that need multiple bytes to be stored, so if you are given a pointer to some data that’s all the same  type and thus each item in that sequence is the same width in their span, then you have yourself an array, even if no one says it’s an array, you can tell the C compiler, hey this is a pointer of type `x` , the compiler will then be helpful and as long as it can determine the width of the span of a single value of type `x` it will allow you to write `some_x_pointer[i]` which becomes `*(some_x_pointer + i)` which further boils down to:

`*((byte *)some_x_pointer + i * sizeof(x))`

So in code:

```c
/**
8 bytes in most compilers */
typedef unsigned long int x;

extern x *ptr;

x sum_x_array(x *arr) {
	x sum = 0;
	for (int i = 0; i < 10; i++) {
        sum += arr[i];
    }
    return sum;
}
```

This compiles down to this assembly:

```s
sum_x_array:
        push    rbp
        mov     rbp, rsp
        mov     QWORD PTR [rbp-24], rdi
        mov     QWORD PTR [rbp-8], 0
        mov     DWORD PTR [rbp-12], 0
        jmp     .L2
.L3: ; This whole block is roughly the body of the for loop in the C code above
        mov     eax, DWORD PTR [rbp-12]
        cdqe
        lea     rdx, [0+rax*8] ; rax here is holding the address of arr in the function above
												       ; and as I explained the compiler boiled it down into 
												       ; *((byte *)some_x_pointer + i * sizeof(x))
												       ; which is *(rax + i * 8) where 8 is sizeof(x)
        mov     rax, QWORD PTR [rbp-24]
        add     rax, rdx
        mov     rax, QWORD PTR [rax]
        add     QWORD PTR [rbp-8], rax
        add     DWORD PTR [rbp-12], 1
.L2: ; This is the i < 10 part
        cmp     DWORD PTR [rbp-12], 9 ; note how i < 10 becomes compare i to 9
        jle     .L3 ; and while it is less than or equal to 9, jumb back to the loop body
        mov     rax, QWORD PTR [rbp-8]
        pop     rbp
        ret
```

# Conclusion

This article hopefully helped clear up some of the mysteries of memory and low level programming, these dark arts, that seem so inaccessible, to all but the most experienced grey bearded wizards or the sorcerers like the iconic Margaret Hamilton (seen bellow).

![Margaret Hamilton in 1969, standing next to listings of the software she and her MIT team produced for the Apollo project, public domain[1].](magaret-hamilton.jpg)

Margaret Hamilton in 1969, standing next to listings of the software she and her MIT team produced for the Apollo project, public domain[1].

# Next up

Soon I plan to release another article about numeric bases and some other in depth more math heavy details on that aspect. Also some bit manipulation dark arts can be talked about.

If there’s demand I can talk about how a CPU works and how one can be emulated in software!

# Further reading

1. https://www.encyclopediaofmath.org/index.php?title=Turing_machine
2. https://en.wikipedia.org/wiki/Turing_machine

# Image credits
* https://commons.wikimedia.org/wiki/File:Margaret_Hamilton_-_restoration.jpg?uselang=en#Licensing
* https://commons.wikimedia.org/wiki/File:KL_CoreMemory.jpg#Licensing
