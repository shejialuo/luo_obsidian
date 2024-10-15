# Chapter 1 Linking and Loading

## 1.1 What do Linkers and Loaders do

The basic job of any linker and loader is simple: It binds more abstract names to more concrete names, which permits programmers to write code using the more abstract names. For example, it takes a name written by a programmer such as `getline` and binds it to the "the location 612 bytes from the beginning of the executable code in module `iosys`".

## 1.2 Address Binding: A Historical Perspective

The earliest computers were programmed entirely in machine language. Programmers would write out the symbolic programs on sheets of paper, hand-assemble them into machine code, and the toggle the machine code into the computer. If the programmer used symbolic addresses and need to add/delete an instruction, the entire program had to be inspected and any addresses affected by this instruction would be adjusted by hand.

The problem was that *the names were bound to addresses too early in the process*. Assemblers solved that problem by letting programers write programs in terms of symbolic names, with the assembler binding the names to machine addresses. If the program changed, the programmer had to reassemble it, but *the work of assigning the addresses is pushed off from the programer to the computer*.

Libraries of code compound the address assignment problem. Computer installations keep a library of pre-written subprograms that programmers can draw upon for use in new programs they write. People at that time wrote about loading programs to do relocation and library search.

With the advent of operating systems, relocating loaders separate from linkers and libraries became necessary. With operating systems, the program had to share the computer's the memory with the operating system and even with other programs. This meant that the actual addresses at which the program would be running weren't known until the operating system loaded the program into memory.

Linkers and loaders now divided up the work, with linkers doing part of the *address binding*, *assigning relative address* with each program, and the loader doing a *final relocation* step to assign actual addresses.

Programs quickly became larger than available memory, so linkers provided overlays, a technique that let *programmers arrange for different parts of a program to share the same memory*, with each overlay loaded on demand when another part of the program called into it.

When different programs are running on a computer, those different programs usually turn out to share al ot of common code. For example, nearly every program written in C users routines such as `fopen` and `printf`. Most systems now provide shared libraries for programs to use, so that all the programs that use a library can share a single copy of it. This both improves run-time performance and saves a lot of disk space.

In the simpler static shared libraries, each library is bound to specific addresses at the time the library is built, and the linker binds program references to library routines to those specific addresses at link time. Static libraries turn out to be inconveniently inflexible, because programs potentially have to *relinked* every time any part of the library changes.

Systems therefore added dynamically linked libraries in which library sections and symbols *aren't bound to actual addresses until the program that uses the library starts running*. Sometimes, the binding is delayed even further than that; with full-fledged dynamic linking, the address of called procedures aren't bound until the first call. Furthermore, programs can bind to libraries as the programs are running, loading libraries in the middle of the program execution.

## 1.3 Linking VS. Loading

Linkers and loaders perform several related but conceptually separate actions.

+ *Program loading*. They copy a program from secondary storage into main memory so it's ready to run.
+ *Relocation*. Compilers and assemblers generally create each file of object code with the program addresses starting at zero. If a program is created from multiple subprograms, all the subprograms have to be loaded at *non-overlapping* addresses. Relocation is the process of assigning load addresses to the various parts of the program and adjusting the code and data in the program to reflect the assigned addresses.
+ *Symbol resolution*. When a program is built from multiple subprograms, the references from one subprogram to another are made using symbols.

Although there's considerable overlap between linking and loading, it's reasonable to define a program that does program loading as a loader, and one that does symbol resolution as a linker.

### 1.3.1 Two-Pass Linking

A linker takes as its input a set of input object files, libraries, and perhaps command files, and produces as its result an *output object file*.

Each input file contains a set of *segments*, contiguous blocks of code or data to be placed in the output file. Each input file also contains at lease one symbol table. Some symbols are *exported*. Other symbols are *imported*.

When a linker runs, it first has to scan the input files to find the sizes of the segments and to collect the definitions and references of all of the symbols. It creates a *segment table* listing all of the segments defined in the input files and a symbol table *with all of the symbols imported or exported*. Using the data from the first pass, the linker assigns numeric locations to symbols, determines the *sizes and location* of the segments in the output address space.

The second pass uses the information collected in the first pass to control the actual linking process. It reads and relocates the object code, substituting numeric addresses for symbol references and adjusting memory addresses in code and data to reflect relocated segment addresses, and writes the relocated code to the output file.

### 1.3.2 Object Code Libraries

All linkers support object code libraries in one form or another, with most also providing support for various kinds of shared libraries. The basic principle of an object code library is simple enough. A library is little more than a set of object code files.

Shared libraries complicate this task a little by moving some of the work from *link time* to *load time*. The linker *identifies* the shared libraries that resolve the undefined names in a linker run, but rather than linking anything into the program, *the linker notes in the output file the names of the libraries in which the symbols were found*, so that the shared library can be found in when the program is loaded.
