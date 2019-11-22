---
date: "2019-10-30 06:30:00+09:00"
lastmod: "2019-11-22 15:30:00+09:00"
title: "ARM Assembly: Sorting"
featured: true
categories:
- Programming
- ARM
- Assembly
- Algorithms
image:
  caption: "xkcd.com/1185"
---
After taking a hiatus for two years, I've started working with ARM assembly language again. I realized that the code I had been working on before had become a sort of utility library, so I rearranged the [git repository](https://github.com/vaelen/assembly/tree/master/arm/util) to reflect that.

While doing so, I noticed that my sorting libraries were in an incomplete state, so I decided to work on finishing them.  The result is that I now have four working sort functions that all operate in place on an existing array of 32bit signed integers.

Below you'll find more details about each of the sorting algorithms I implemented, along with the code.  All of the following code snippets are licensed under the [GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html).

# Bubble Sort

[Bubble sort](https://en.wikipedia.org/wiki/Bubble_sort) is often one of the first sorting algorithms people learn.  It works by iterating through the list of items to be sorted and swapping items that are out of order.  After each iteration, if any swaps were made it iterates again.  This results in the largest (or smallest) value "bubbling" up to the end of the list.

{{< youtube Cq7SMsQBEUw >}}

The algorithm is very simple, but both its average and worst case [time complexities](https://en.wikipedia.org/wiki/Time_complexity#Table_of_common_time_complexities) are O(N²) (quadratic), which make it fairly inefficient.  Because of this, bubble sort is almost never used in real applications. However, one nice thing about bubble sort is that when applied to an already sorted list it only requires a single iteration to verify that the list is sorted.  In this best case scenario the time complexity is O(1) (constant).

{{< youtube lyZQPjUT5B4 >}}

The version provided below includes an optimization that skips the already sorted elements at the end of the array during each successive iteration.  This improves actual performance but does not change the overall time complexity.

```armasm
/*
    Copyright 2019, Andrew C. Young <andrew@vaelen.org>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
*/

// Bubble Sort

// Exported Methods
    .global bsort

// Code Section

bsort:
    // Bubble sort an array of 32bit integers in place
    // Arguments: R0 = Array location, R1 = Array size
    PUSH    {R0-R6,LR}          // Push registers on to the stack
bsort_next:                     // Check for a sorted array
    MOV     R2,#0               // R2 = Current Element Number
    MOV     R6,#0               // R6 = Number of swaps
bsort_loop:                     // Start loop
    ADD     R3,R2,#1            // R3 = Next Element Number
    CMP     R3,R1               // Check for the end of the array
    BGE     bsort_check         // When we reach the end, check for changes
    LDR     R4,[R0,R2,LSL #2]   // R4 = Current Element Value
    LDR     R5,[R0,R3,LSL #2]   // R5 = Next Element Value
    CMP     R4,R5               // Compare element values
    STRGT   R5,[R0,R2,LSL #2]   // If R4 > R5, store current value at next
    STRGT   R4,[R0,R3,LSL #2]   // If R4 > R5, Store next value at current
    ADDGT   R6,R6,#1            // If R4 > R5, Increment swap counter
    MOV     R2,R3               // Advance to the next element
    B       bsort_loop          // End loop
bsort_check:                    // Check for changes
    CMP     R6,#0               // Were there changes this iteration?
    SUBGT   R1,R1,#1            // Optimization: skip last value in next loop
    BGT     bsort_next          // If there were changes, do it again
bsort_done:                     // Return
    POP     {R0-R6,PC}          // Pop the registers off of the stack
```

# Quicksort

[Quicksort](https://en.wikipedia.org/wiki/Quicksort) is often the second sorting algorithm one learns. It works by first choosing a pivot value and then dividing the list of items to be sorted into two buckets based on whether they are smaller than or larger than the pivot value.  It then repeats this process on each bucket until the entire list is sorted.

{{< youtube 8hEyhs3OV1w >}}

The average [time complexity](https://en.wikipedia.org/wiki/Time_complexity#Table_of_common_time_complexities) of quicksort is O(n log n) (loglinear), making it much faster than bubble sort. However, if the pivot selected at each step is either the largest or smallest value in the list, then it effectively becomes a bubble sort and its time complexity becomes O(N²). This can be avoided by improving the pivot selection process so that it never selects either the largest or smallest value.

{{< youtube ywWBy6J5gz8 >}}

The simplest implementation simply chooses either the first value in the list or the last value to use as the pivot.  With this implementation, applying the algorithm to an already sorted list results in the worst case scenario in terms of time complexity. The code below avoids this by using the "median-of-three" pivot selection algorithm. This works by looking at the first, middle, and last values in the list and selecting the one that is the median of these three values as the pivot. In this way the code is guaranteed to never choose either the largest or smallest value in the list.

```armasm
/*
    Copyright 2019, Andrew C. Young <andrew@vaelen.org>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
*/

// Quick Sort

// Exported Methods
    .global qsort

// Code Section
qsort:
    // Quick sort an array of 32bit integers
    // Arguments: R0 = Array location, R1 = Array size
    PUSH    {R0-R10,LR}         // Push registers on to the stack
    MOV     R4,R0               // R4 = Array Location
    MOV     R5,R1               // R5 - Array Size
    CMP     R5,#1               // Check for an array of size <= 1
    BLE     qsort_done          // If array size <= 1, return
    CMP     R5,#2               // Check for an array of size == 2
    BEQ     qsort_check         // If array size == 2, check values
qsort_partition:
    MOV     R1,#2               // Find the middle element
    SDIV    R2,R5,R1            // R2 = The middle element index
    LDR     R6,[R4]             // R6 = Beginning of array value
    LDR     R7,[R4,R2,LSL #2]   // R7 = Middle of array value
    SUB     R8,R5,#1            // R8 = Upper array bound index (len -1 1)
    LDR     R8,[R4,R8,LSL #2]   // R8 = End of the array value
    CMP     R6,R7               // Sort the values
    MOVGT   R9,R6
    MOVGT   R6,R7
    MOVGT   R7,R9
    CMP     R7,R8
    MOVGT   R9,R7
    MOVGT   R7,R8
    MOVGT   R8,R9
    CMP     R6,R7
    MOVGT   R9,R6
    MOVGT   R6,R7
    MOVGT   R7,R9
    MOV     R6,R7               // R6 = Pivot
    MOV     R7,#0               // R7 = Lower array bounds index
    SUB     R8,R5,#1            // R8 = Upper array bounds index (len - 1)
qsort_loop:
    LDR     R0,[R4,R7,LSL #2]   // R0 = Lower value
    LDR     R1,[R4,R8,LSL #2]   // R1 = Upper value
    CMP     R0,R6               // Compare lower value to pivot
    BEQ     qsort_loop_u        // If == pivot, do nothing
    ADDLT   R7,R7,#1            // If < pivot, increment lower index
    STRGT   R0,[R4,R8,LSL #2]   // If > pivot, swap values
    STRGT   R1,[R4,R7,LSL #2]
    SUBGT   R8,R8,#1            // if > pivot, decrement upper index
    CMP     R7,R8               // if indexes are the same, recurse
    BEQ     qsort_recurse
    LDR     R0,[R4,R7,LSL #2]   // R0 = Lower value
    LDR     R1,[R4,R8,LSL #2]   // R1 = Upper value
qsort_loop_u:
    CMP     R1,R6               // Compare upper value to pivot
    SUBGT   R8,R8,#1            // if > pivot, decrement upper index
    STRLT   R0,[R4,R8,LSL #2]   // If < pivot, swap values
    STRLT   R1,[R4,R7,LSL #2]
    ADDLT   R7,R7,#1            // if < pivot, increment lower index
    CMP     R7,R8               // if indexes are the same, recurse
    BEQ     qsort_recurse
    B       qsort_loop          // Continue loop
qsort_recurse:
    MOV     R0,R4               // R0 = Location of the first bucket
    MOV     R1,R7               // R1 = Length of the first bucket
    BL      qsort               // Sort first bucket
    ADD     R8,R8,#1            // R8 = 1 index past final index
    CMP     R8,R5               // Compare final index to original length
    BGE     qsort_done          // If equal, return
    ADD     R0,R4,R8,LSL #2     // R0 = Location of the second bucket
    SUB     R1,R5,R8            // R1 = Length of the second bucket
    BL      qsort               // Sort second bucket
    B       qsort_done          // return
qsort_check:
    LDR     R0,[R4]             // Load first value into R0
    LDR     R1,[R4,#4]          // Load second value into R1
    CMP     R0,R1               // Compare R0 and R1
    BLE     qsort_done          // If R1 <= R0, then we are done
    STR     R1,[R4]             // Otherwise, swap values
    STR     R0,[R4,#4]          //
qsort_done:
    POP     {R0-R10,PC}         // Pop registers off of the stack and return
```

# Heapsort

[Heapsort](https://en.wikipedia.org/wiki/Heapsort) works by building a [heap](https://en.wikipedia.org/wiki/Heap_(data_structure)) out of the array. Once an array has been converted to a heap, the first item is guaranteed to be either the smallest or largest value depending on whether the heap is a minimum or maximum heap, respectively. The heapsort algorithm uses a maximum heap and therefore the first value in the array will always be the largest value after the heap is created. It then swaps this value with the value at the end of the array and re-creates the heap ignoring the now properly sorted last element. The algorithm thus builds the sorted array in place from the end forward. When there are only two items left to be sorted it performs a simple comparison and swap.

{{<youtube _bkow6IykGM>}}

The best, worst, and average [time complexities](https://en.wikipedia.org/wiki/Time_complexity#Table_of_common_time_complexities) of heapsort are all O(N log N).  This is because building the heap has an average time complexity of O(log N) and this process is performed N times as the array is sorted.

``` armasm
/*
    Copyright 2019, Andrew C. Young <andrew@vaelen.org>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
*/

// Heapsort

// Exported Methods
    .global hsort

// Code Section

hsort:
    // R0 = Array location
    // R1 = Array length
    PUSH    {R2-R12,LR}         // Push registers on to the stack
    CMP     R1, #1              // If the array has size 0 or 1, return
    BLE     hsort_done
    CMP     R1, #2              // If the array has size 2, swap if needed
    BLE     hsort_simple
    MOV     R11, #1             // R11 = 1 (constant)
    MOV     R12, #2             // R12 = 2 (constant)
    MOV     R10, #-1            // R10 = -1 (temporarily)
    UDIV    R10, R1, R12        // R10 = (len / 2) - 1 (start heapify index)
    SUB     R10, R10, #1
    SUB     R9, R1, #1          // R9 = len - 1 (start pop index)
    MOV     R2, R10             // R2 = Starting heapify index
heapify:
    // R2 = Index
    MOV     R3, R2              // R3 = Index of largest value
    MLA     R4, R2, R12, R11    // R4 = Left index (2i+1)
    MLA     R5, R2, R12, R12    // R5 = Right index (2i+2)
    LDR     R6, [R0, R2, LSL#2] // R6 = Index value (R0 + (R2 * 4))
    MOV     R7, R6              // R7 = Current largest value
heapify_check_left:
    CMP     R4, R9              // Check if left node is in array bounds
    BGT     heapify_check_right // If not then check the right node
    LDR     R8, [R0, R4, LSL#2] // R8 = Left value (R0 + (R4 * 4)
    CMP     R8, R7              // Compare left value to largest value
    BLE     heapify_check_right // If left <= largest check right
    MOV     R3, R4              // If left > largest, change largest index
    MOV     R7, R8              //    and copy the value
heapify_check_right:
    CMP     R5, R9              // Check if right node is in array bounds
    BGT     heapify_swap        // If not then continue
    LDR     R8, [R0, R5, LSL#2] // R8 = Right value (R0 + (R5 * 4))
    CMP     R8, R7              // Compare right value to largest value
    BLE     heapify_swap        // If right <= largest then continue
    MOV     R3, R5              // If right > largest, change largest index
    MOV     R7, R8              //    and copy the value
heapify_swap:
    CMP     R2, R3              // Check if largest is the initial value
    BEQ     heapify_next        // If so, exit the heapify loop
    STR     R7, [R0, R2, LSL#2] // If not, swap largest and initial values
    STR     R6, [R0, R3, LSL#2]
    MOV     R2, R3              //    and change index to largest index
    B       heapify             //    and recurse
heapify_next:
    CMP     R10, #0             // If last index was 0, heap is finished
    BEQ     heapify_pop
    SUB     R10, R10, #1        // Else, heapify next index
    MOV     R2, R10
    B       heapify
heapify_pop:
    LDR     R3, [R0]            // R3 = Largest value (front of heap)
    LDR     R4, [R0, R9, LSL#2] // R4 = Value from end of heap
    STR     R3, [R0, R9, LSL#2] // Store largest value at end of heap
    STR     R4, [R0]            // Store end value at start of heap
    SUB     R9, #1              // Decrement end of heap index
    CMP     R9, #1              // If there are only two items left, compare
    BEQ     hsort_simple
    MOV     R2, #0              // Else, heapify from the start of the heap
    B       heapify
hsort_simple:
    // Sort a list of exactly 2 elements
    LDR     R2, [R0]            // First value
    LDR     R3, [R0, #4]        // Second value
    CMP     R2, R3              // Compare values
    STRGT   R3, [R0]            // Swap if out of order
    STRGT   R2, [R0, #4]
hsort_done:
    POP     {R2-R12,PC}         // Return
```

# Radix Sort

[Radix sort](https://en.wikipedia.org/wiki/Radix_sort) works a bit differently from other sorting algorithms.  Like quicksort it works by first dividing the unsorted list into two buckets and then iterating over each bucket.  However, rather than using a comparison function to determine whether a given value is larger or smaller than another value it uses the radix (or base) to determine which bucket each value should go in.

{{< youtube Tmq1UkL7xeU >}}

For example, if we were going to sort a deck of 100 cards numbered 0-99, we might perform the sort like this:

1. Divide the cards into piles based on their first digit (0-9), left-to-right. This results in 10 piles.
2. For each pile, divide those cards into piles based on their second digit (0-9), left-to-right.  This results in 10 new piles for each of the first piles, each with only a single card in it.
3. Reassemble the deck by picking up the cards from left to right and the cards will be in order.

As shown by the example, the number of buckets used at any given step depends on the radix (or base) being used.  If the numbers are represented in base 10 (decimal), then each iteration produces 10 buckets.  However, if the numbers are stored in base 2 (binary), then only two buckets are produced. Likewise, the data can be sorted either by the most significant digit (MSD) as in the example above, or by the least significant digit (LSD).

The code below implements an in-place binary MSD radix sort, also called a binary quicksort. Although sorting unsigned integers works as expected, sorting signed integers will result in the negative integers being sorted after the positive integers.  This is due to the fact that negative signed integers have their most significant bit set to `1`. This code could also probably be further optimized by moving the swap logic in-line, but I think it is more readable this way.

```armasm
/*
    Copyright 2019, Andrew C. Young <andrew@vaelen.org>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
*/

// Radix Most Significant Bit In-Place Sorting Algorithm

// Exported Methods
    .global rsort
    .global swap

// Code Section

rsort:
    // Radix MSD sort an array of 32bit integers
    // Arguments: R0 = Array location, R1 = Array size
    PUSH    {R0-R2,LR}          // Push existing registers on to the stack
    CMP     R1,#1               // Check for an empty or single member array
    POPLE   {R0-R7,PC}          // If so, return to where we came from
    ADD     R1,R0,R1,LSL #2     // R1 = End of the array (R0 + (R1*4))
    SUB     R1,R1,#4            //
    MOV     R2,#1               // R2 = Bitmask
    LSL     R2,R2,#31           //   most significant bit
    BL      rsort_recurse       // Begin recursion
    POP     {R0-R2,PC}          // Pop the registers off of the stack

rsort_recurse:
    // Radix MSD sort an array of 32bit integers (recursive helper)
    // Arguments: R0 = Array start, R1 = Array end, R2 = Bitmask
    PUSH    {R0-R8,LR}          // Store previous values
    SUB     R8,R1,R0            // Check array length
    CMP     R8,#4               //
    BLT     rr_done             // Return if array is empty or has 1 entry
    MOV     R3,R0               // R3 = 0s bin pointer (start of array)
    MOV     R4,R1               // R4 = 1s bin pointer (end of array)
    MOV     R5,#0               // R5 = Current value
    MOV     R6,R0               // Set original array start (debug)
    MOV     R7,R1               // Set original array end (debug)
rr_0loop:
    LDR     R5,[R3]             // Load next value from pointer
    TST     R5,R2               // Check bitmask
    BEQ     rr_0loop_next       // If the value is 0, loop
    BL      rr_swap             // If not, swap values
    B       rr_1loop_next       // Switch to the 1s bin
rr_0loop_next:
    ADD     R3,R3,#4            // Increment the pointer
    CMP     R3,R4               // Check if pointers are the same
    BGT     rr_next_bit         // If so, move to the next bit
    B       rr_0loop            // If not, check the next value
rr_1loop:
    ADD     R12,#1              // Increment loop counter (debug)
    LDR     R5,[R4]             // Load current value from pointer
    TST     R5,R2               // Check bitmask
    BNE     rr_1loop_next       // If the value is 1, loop
    BL      rr_swap             // If not, swap values
    B       rr_0loop_next       // Switch to the 0s bin
rr_1loop_next:
    SUB     R4,R4,#4            // Decrement the pointer
    CMP     R3,R4               // Check if pointers are the same
    BGT     rr_next_bit         // If so, move to the next bit
    B       rr_1loop            // If not, check the next value
rr_next_bit:
    LSR     R2,R2,#1            // Update bitmask to next bit
    CMP     R2,#0               // Check for all zeros
    BEQ     rr_done             // If so, do not recurse anymore
    MOV     R0,R6               // Array start
    MOV     R1,R3               // 0s bin array end
    SUB     R1,#4
    BL      rsort_recurse       // Sort 0s bin
    MOV     R0,R4               // 1s bin array start
    ADD     R0,#4
    MOV     R1,R7               // Array end
    BL      rsort_recurse       // Sort 1s bin
    B       rr_done             // All done
rr_swap:
    PUSH    {LR}                // Store LR
    MOV     R0,R3               // End of 0s bin
    MOV     R1,R4               // End of 1s bin
    BL      swap                // Swap values
    POP     {PC}                // Return
rr_done:
    POP     {R0-R8,PC}          // Restore previous values

swap:
    // Swaps the values from 2 memory locations
    // R0 = 1st memory location
    // R1 = 2nd memory location
    PUSH    {R0-R4,LR}          // Store previous values
    LDR     R3,[R0]             // Load value from the first memory location
    LDR     R4,[R1]             // Load value from the second memory location
    STR     R3,[R1]             // Store value 2 in memory location 1
    STR     R4,[R0]             // Store value 1 in memory location 2
    POP     {R0-R4,PC}          // Return
```

# Additional Links and Videos

For more details on common sorting algorithms, including visualizations, I suggest checking out the [Sorting Algorithm Animations](https://www.toptal.com/developers/sorting-algorithms) page by Toptal. I also recommend the [Sound of Sorting](https://panthema.net/2013/sound-of-sorting/) website, which is where I found many of the videos in this article.  They have a [YouTube playlist](https://www.youtube.com/watch?v=8oJS1BMKE64&list=PLZh3kxyHrVp_AcOanN_jpuQbcMVdXbqei&index=1) that includes visualizations of many different sorting algorithms. Finally, check out [AlgoRythmics](https://www.youtube.com/user/AlgoRythmics/videos), who produced the dancing videos.
