---
date: "2016-05-13 20:23:01-06:00"
lastmod: "2016-05-13 20:23:01-06:00"
title: "ARM Assembly: Binary Heaps"
featured: true
categories:
- Programming
- ARM
- Assembly
- Data Structures
---
I got a [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi) for Christmas and I've been teaching myself [ARM assembly](https://en.wikipedia.org/wiki/ARM_architecture#Instruction_set).

It's my first time working with assembly language, as I didn't take an systems architecture or OS fundamentals class in college.

I'm slowly working on a Huffman Encoder, trying to use only native Linux system calls without making calls to other external libraries.

This will be the first in a series of posts about this topic.

The posts probably won't be very long, the stuff I'm doing isn't revolutionary, so there's not a lot to discuss.

Today I'm here to talk about [binary
heaps](https://en.wikipedia.org/wiki/Binary_heap).  When I learned
about binary heaps in my data structures class I didn't really
understand the usefulness of them.  I knew that you could use them to
implement heapsort, and to make priority queues, but because they're
simpler than [red-black
trees](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree) and other
binary search trees, they can't be used for easy ordered
traversal. Working in higher level languages I never really had a need
to implement a heap for anything.

However, I am now in need of a priority queue, and I am trying to work
in a static memory footprint because I'm trying not to allocate any
additional memory beyond the initial memory footprint of the
application. Suddenly binary heaps make a whole lot of sense. (Before
you ask, yes, I did realize afterwards that I could solve the same
problem using the same space and time complexity with a binary search
tree, which is what I will implement next.)

The code I wrote to work with heaps is designed to be
used for a priority queue. As such it stores a 2-word (64 bit) value
in the heap. The first word (32 bits) of the value is the "key" or
"priority", used to sort the entry within the heap. The second word
(32 bits) is the "value". The value is actually optional, so one could
use this same code for sorting an array of integers, for example,
although with a few tweaks it could be made more more efficient for
that purpose. Also, when removing an element I ought to swap it with
the last element rather than just copying the last element into the
first spot. That would make the algorithm effectively do a heapsort
out-of-the-box. Maybe in v2. The code also stores the current size of
the heap and the maximum size of the heap in the first two words of
the heap's memory space so that those values don't need to be passed
around by hand.

I am still learning assembly, but hopefully someone will find this
useful.

The code is located here:
[https://github.com/vaelen/assembly/blob/master/arm/util/heap.s](https://github.com/vaelen/assembly/blob/master/arm/util/heap.s)
The test code is here:
[https://github.com/vaelen/assembly/blob/master/arm/util/test-heap.s](https://github.com/vaelen/assembly/blob/master/arm/util/test-heap.s)
