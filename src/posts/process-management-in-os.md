---
title: Process management in OS
description: Walk through of how a process works, its componenets, how the CPU interacts with the process. Have also described how this can be used practically.
permalink: posts/{{ title | slug }}/index.html
date: '2025-05-18'
tags: [os, process]
---

## Overview
If you are looking to learn system languages like Rust, C after working on langugaes like Python, JS, you might face lot of challenges. So before directly jumping into using system languages, you should be familiar with lot of operating system concepts. Mainly memory management, system calls, pointer, process management etc.  

I will be start the blog series with process memory and then move to other concepts.
They are the fundament parts of any memory space of a process. I will go over the usage of these data structures and how instructions flow from application to OS to CPU. Also how this knowledge can help you writing better code

The OS manages all its processes using process control block (PCB). The PCB is a doubly linked list of process metadata that are currently running. The metadata contains process state, PID, CPU register and program counter data to the last executed instruction before CPU interrupt, priority of the process, location to the process etc		

```

+----------------+      +----------------+      +----------------+
|    PCB (Proc1) | <--> |    PCB (Proc2) | <--> |    PCB (Proc3) |
+----------------+      +----------------+      +----------------+

Each PCB contains:

+-----------------------------+
| Process State: Running      |
| PID: 101                    |
| CPU Registers: [R1, R2, ...]|
| Program Counter: 0x0040A2F0 |
| Priority: 5                 |
| Memory Location: 0x10000000 |
| Prev PCB: pointer           |
| Next PCB: pointer           |
+-----------------------------+

```


The process itself is stored in memory containing segments mainly: 
- **Text Segment**: Contains the compiled code
- **Data:** Contains global variables and static data
- **Heap:** Can be used to dynamically allocate memory using functions like malloc
- **Stack:** Stores function/local variables, method parameters, return addresses

```

Higher Memory Addresses
+-----------------------+
|                 Stack |  <-- grows downward (toward lower addresses)
+-----------------------+
|                       |
|                       |
|                 Heap  |  <-- grows upward (toward higher addresses)
+-----------------------+
|      Data Segment     |
+-----------------------+
|       Text Segment    |  <-- program code, read-only
+-----------------------+
Lower Memory Addresses

```


**Stack:**
- The memory allocated to stack can vary from few 16 KB to 10 MB. Each function is loaded in the stack along with its args, variables, return address. It is a `contiguous` memory address.
- The stack data is placed at the higher address space in a process and it grows in downward direction in the process memory i.e higher memory address to lower memory address. In cases of uncontrolled recursion or too many local variables, there can be **stack overflow**, due to stack's limited size.
- Each variable allocated on the stack has a size determined at **compile time** based on its type. For example, an `int` might be 4 bytes, a `double` 8 bytes, and so on. This size does not change during execution.
- Some key pointers in a stack are Stack Pointer, Frame Pointer. 
	- **Frame Pointer:** A `frame` represents the memory allocated for an individual `function` call on the stack, known as a `stack frame`. The Frame Pointer points to a fixed base address within this frame, providing a stable reference for accessing function parameters and local variables. This pointer is stored in a CPU register-called **EBP** in 32-bit architectures and **RBP** in 64-bit architectures.
	- **Stack Pointer:**  The Stack Pointer points to the current top of the stack-the location where the next push or pop operation will occur. It is updated frequently as data is pushed onto or popped from the stack. The Stack Pointer is stored in the **ESP** register on 32-bit architectures and **RSP** on 64-bit architectures.


```

Higher Memory Addresses
+-----------------------+
| Function Arguments    |  (EBP + 8, EBP + 12, ...)
+-----------------------+
| Return Address        |  (EBP + 4)
+-----------------------+
| Saved Caller func EBP |  (EBP)  <-- old frame pointer saved on stack
+-----------------------+
| Local Variables       |  (EBP - 4, EBP - 8, ...)
+-----------------------+
| 	TOP OF STACK        |  (ESP)
+-----------------------+
Lower Memory Addresses

```

There are stack components which are useful to know since they are used in various application 
which is explained later.


**Heap:**
- Developers can get memory allocated in the heap dynamically using system calls like malloc/new etc. 
- Initially the OS responds to these system calls responds by providing a chunk of memory, usually larger than the asked size.
- When we store variable on this memory block, that address pointer is stored in the stack. 
- The data stored on heap is not contiguous since the any variable can be stored or removed at any point in time, unlike the stack. This cause memory fragmentation in the memory block. i.e lot of gaps between occupied memory address.

```

|uint8|tempfiledata||empty|string|empty|float64|empty|empty|tempfiledata|empty
			    			HEAP MEMORY BLOCK CONTAINING FRAGMENTED DATA
```

- When we again allocate some memory, the existing memory chunk is used to see if there is space available in it. The function malloc/new first check if the memory block has enough space using **FirstFit**, **BestFit**, **WorstFit** algorithms, if not a system call is made to get more memory.
- Dynamic memory allocation makes the heap risk prone to memory leaks, dangling pointers etc

##### Usage (elaborate below points)
- Based on the above knowledge, in system programming, we should allocate memory at compile time i.e stack for better performance. Heap has penalties of memory allocation like system call and search algorithms.
- Observability tools are built on the knowledge of CPU register like EBP, ESP and data in the stack relative to these pointers. This along with eBPF () hooks
- Static analyzers


#### Example in Rust


## References
- [Reference 1](URL) - Read more about epoll, select, kqueue 
- [OS Notes](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/3_Processes.html) [Tech Caps](https://techcaps.ie/components-of-a-process-in-operating-system/)

---

