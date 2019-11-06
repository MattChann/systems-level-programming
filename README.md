# Notes for Systems Level Programming Class

------------------------------------------------------------
### Wednesday, November 6, 2019

#### File Permissions
- **File Permissions**
	- 3 types of permissions
		- read, write, execute
	- Permissions can be represented as 3-digit binary numbers, or 1-digit octal numbers
		- `100` <==> `4` => read only
		- `111` <==> `7` => read, write, execute
	- There are 3 permission "areas"
		- user, group, others
			- Membership in each "area" is mutually exclusive
			- The creator of the file is the default setting for the user and group of a file
	
	- `chmod permissions file`
		- Command line utility to change file permissions
		- The owner of a file (or root) can always change permissions
		- File ownership and group can be changed with the `chown` and `chgrp` command line utilities

- **File Table**
	- A list of all files being used by a program while it is running
	- Contains basic information like the file's location and size
	- The file table has limited space, which is a power of 2 and commonly 256
	- `getdtablesize()` will return the file table size
	- Each file is given an integer index, starting at 0, this is the file descriptor
	- There are 3 files always open in the table:
		- 0 or 

------------------------------------------------------------
### Monday, November 4, 2019

#### Binary, Octal, and Hexadecimal Integers
- Other base formatting characters for `printf`:
	- `%o`: octal integer
	- `%x`: hexadecimal integer
- You can define native integers in bases 2, 8, and 16 by using the following prefixes
	- `0b`: binary
	- `0`: octal
	- `0x`: hexadecimal

_Note: Why do programmers always mix up Holloweeen and Christmas? ... Because 31 Oct is the same as 25 Dec :P_

#### Bitwise Operators
- Evaluated on every bit of a value
- `~x`
	- Negation
	- Flip every bit of `x`
- `a | b`
	- Bitwise or
	- Perform logical or for each pair of bits in (a, b)
- `a & b`
	- Bitwise and
	- Perform logical and for each pair of bits in (a, b)
- `a ^ b`
	- Bitwise xor
	- Perform logical xor for each pair of bits in (a, b)
```c
char i = 13
i: 00001101

!i: 0 
~i: 11110010

char x = 8;
x: 00001000
~i | x : 11111010
~i & x : 00000000

// swapping a and b
a = a ^ b // a contains bits not in common
b = a ^ b // b now contains a 
a = b ^ a // a now contains b 

// swapping cont.
r = a ^ b 
b = r ^ b => a ^ b ^ b = 0 ^ a 
a = r ^ b => a ^ b ^ a = 0 ^ b
```
- **Swapping Bits Using Only Bitwise Operators**

| a | b | a = a ^ b | b = a ^ b | a = b ^ a | 
|---|---|-----------|-----------|-----------|
| T | T | F         | T         | T         |
| T | F | T         | T         | F         |
| F | T | T         | F         | T         |
| F | F | F         | F         | F         |

------------------------------------------------------------
### Monday, October 28, 2019

#### Makefile:
```make
ifeq ($(DEBUG), true)
	CC = gcc -g
else
	CC = gcc
endif


all: main.o definitions.o
	$(CC) -o program main.o definitions.o

main.o: main.c headers.h
	$(CC) -c main.c

definitions.o: definitions.c headers.h
	$(CC) -c definitions.c

run:
	./program

memcheck:
	valgrind --leak-check=yes ./program

clean:
	rm *.o
	rm program
```
- can now run this command to make with `-g` flags in order to debug:
		
		$ make DEBUG=true
- convention to use all caps variables in makefile
- `$(<variable>)` is how you use variables in make
- can now easily run valgrind using:

		$ make memcheck

------------------------------------------------------------
### Thursday, October 24, 2019

#### GDB - GNU DeBugger
- to use `gdb`, you must compile using the `-g` flag with gcc
- basic usage:

		$ gdb program
	- starts a `gdb` shell from which you can run your program
- **Commands from in the `gdb` shell**
	- `run`: runs the program until it ends/crashes/gets a signal
	- `list`: shows the lines of code run around a crash
	- `print var`: prints the value of of `var` at the time
	- `backtrace`: shows the current stack
	- `break lineNum`: creates breakpoint at a line
- **Running a program in pieces**
	- `run`: restarts the program
	- `continue`: run the program until the next breakpoint/crash/end
	- `next`: run the next line of the program only
	- `step`: run the next line of the program, if that is a function call, run only the next line of that function

#### Valgrind
- tool for debugging memory issues in C programs
- you must compile with `-g` in order to use valgring (and similar tools)
- basic usage:

		$ valgrind --leak-check=yes ./program

------------------------------------------------------------
### Wednesday, October 23, 2019

#### Dynamic Memory Allocation _(cont.)_
```c
calloc(size_t n, size_t x)
```
- allocates `n * x` bytes of memory
- **ensures every bit is 0**

```c
realloc(void *p, size_t x)
```
- changes the amount of memory allocated for a block to `x` bytes
- `p` must point to the beginning of a block
- returns a pointer to the beginning of the block (this is not always the same as `p`)
- if `x` is smaller than the original size of the allocation, the extra bytes will be released
- if `x` is larger than the original size then either:
	1. if there is enough space at the end of the original allocation, the original allocation will be updated
	2. if there is not enough space, a new allocation will be created, containing all the original values; the original allocation will be freed

------------------------------------------------------------
### Tuesday, October 22, 2019

#### Dynamic Memory Allocation
```c
malloc(size_t x)
```
- allocates `x` bytes of **heap** memory
- returns the address at the beginning of the allocation
- returns a `void *`
- can be used like an array:
		
		int *p;
		p = malloc(5 * sizeOf(int));

```c
free(void * p)
```
- releases dynamically allocated memory
- has one parameter, a pointer to the beginning of a dynamically allocated block of memory
	- cannot free a portion, only the entire thing
- every call to `malloc`/`calloc` should have a corresponding call to `free`

```c
calloc(size_t n, size_t x)
```
- allocates `n * x` bytes of memory
- **ensures every bit is 0**
------------------------------------------------------------
### Monday, October 21, 2019

#### Stack Memory
- as a data structure
	- first in, last out
	- terminology: push, pop
- associated with function calls
- stores all normally declared variables (including pointers and structs), arrays, and function calls
- functions are pushed onto the stack in the order they are called, and popped off when completed
- when a function is popped off the stack, the staclk memory associated with it is released
- only **one** stack

#### Heap Memory
- stores dynamically allocated memory
	- **allocated at runtime**
- data will remain in the eap until it is manually released (or the program terminates)
	- hence no garbage collector like in Java


------------------------------------------------------------
### Friday, October 18, 2019

```c
struct login u = new_account(4190);
```
Order of operations when run:
1. allocates memory necessary for `u`
2. runs `new_account(4190)`
3. copies the values of the returned struct into space allocated for `u`
4. returned struct goes away and `u` persists because it is in the `main()` function

Using struct as a parameter vs. pointer:
- copies argument into parameter variable
- edits the copy
- only in scope of the function

------------------------------------------------------------
### Tuesday, October 15, 2019

#### `struct` - creates a new type that is a collection of values
```c
struct {int a; char x;} s;
```
- (kind of like objects, but not advised to think of it this way)
- in this example, `s` is a variable of type `struct {int a; char x;}`
- we use the `.` operator to access values inside a struct

		s.a = 10;
		s.x = '@';
- size of `struct` may not necessarily equal the sum of sizes of the values, might be a little more
- "anonymous" struct, no name
```c
struct foo {int a; char x;};
```
```c
struct foo {
	int a;
	char x;
};
```
- `foo` is a **prototype** for the `struct`

		struct foo s;
- gcc will appropriately copy values between same non-anonymous `struct`s
- common practice (in standard C libraries) to not `typedef` a `struct` because it hides stuff
- normally don't declare `struct`s in main, typically outside all functions
- can also declare `struct`s in header files
- `.` binds before `*`
- to access data from a `struct` pointer you would do:
```c
struct foo *p;
p = &s;
		
(*p).x;
		
p->x; //c shorthand for (*p).x
```

------------------------------------------------------------
### Thursday, October 10, 2019

[Mr.Zamansky and Hunter Daedalus Program Talk](https://zamansky.github.io/presentations/navigating-hs-to-college-in-tech/index.html)

------------------------------------------------------------
### Thursday, October 10, 2019

Google Mentorship Talk

------------------------------------------------------------
### Tuesday, October 8, 2019

#### makefiles
- used to make executable
- works for not only C
- Java Compiler and JVM do stuff for you, so it doesn't give you executables (gives machine code instead)
- good practices
	- separate compilations steps for each \*.c file
	- run
	- clean
- only recompiles modified files
- dependencies should only be one \*.c file and other header \*.h files
- example makefile:

		all: main.o definitions.o
			gcc -o program main.o definitions.o
		
		main.o: main.c headers.h
			gcc -c main.c
		
		definitions.o: definitions.c
			gcc -c definitions.c
		
		run:
			./program

		clean:
			rm *.o

- make will stop at first error

In `char *strncpy(char *dest, const char *src, size_t n);` what is the type `size_t`?

#### `typedef` - provides a new name for an existing data type
Usage:
```c
typedef real_type new_name;
```
Examples:
```c
typedef unsigned long size_t;
size_t x = 139; //x is really an unsigned long
```
Why?
- allows code to work on different machines (known as portability)
- defines appropriately for every machine
- technically: can `typedef` a `typedef`

------------------------------------------------------------
### Friday, October 4, 2019

#### Pointers
- variable type for storing memory addresses
    - unsigned integer type
- 8 bytes large

#### Pointers in Functions
- when used as parameter
    - pass by value
        - address copied into the parameter and points to the same thing
    - argument remains **unchanged**
    - parameter's memory allocated in space allocated for the associated function


#### Primitive Types
|Types  |Size    |
|-------|--------|
|char   |1 byte  |
|int    |4 bytes |
|short  |2 bytes |
|long   |8 bytes |
|float	|4 bytes |
|double |8 bytes |


#### CPU
- Instructions go in (in bits)
- Stuff happens in the middle
- Out comes the result
- 2GHz = 2 billion input/output cycles per second
    - Gets hot because electrical signals all go through the same pathway
- Reads bits instruction, all 64-bits **at once**
    - Cannot break up memory addresses
    - This is why a memory address can only be 8 bytes at most (8 * 8 = 64)

------------------------------------------------------------
### Thursday, October 3, 2019

#### Java
.java --> .class --> JVM --> OS --> Hardware


#### C
.c --> Executable --> OS --> Hardware


#### Protected Memory
- OS allocates memory for programs and keeps track
- Programs cannot access memory outside of their allocation
    - If the program tries to --> Segmentation Fault
 
------------------------------------------------------------
