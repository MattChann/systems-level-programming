# Notes for Systems Level Programming Class

------------------------------------------------------------
### Monday, January 6, 2020

[Beej's Guide](https://beej.us/guide/bgnet/)

#### Socket
- a piece of memory that your program is going to use to transfer data
- unlike a pipe it will need a lot more information on the operating system
- there are 3 things associated with a socket that makes it unique
	1. Protocol (2)
		- 2 main protocols you will deal with
			1. Transmission Control Protocol (TCP) or just Stream Protocol
				- if you send data to another machine using TCP
					- either the data will be sent in entirety in the order you sent
					- or you will get an error
				- 2 packets might take different paths to the same place
				- if packets dont get there, they can ask to send the packet again
			2. User Datagram Protocol (UDP)
				- not verified (no handshake)
				- maybe someone listening, maybe not
				- just sends data out
				- does not worry about rearranging packets (not in order)
				- WONT request packets missing
				- **much faster**
				- used all the time for streaming media and games
	2. Port (65,536)
		- that is what allows you to have multiple sockets associated with the same IP Address
		- ex: every different web browser tab is a socket with different ports
		- some are designated for specific usage
			- port 80 is for web traffic
	3. Incoming/Outgoing Addresses (I.P. Addresses)
		- externally provided
		- generally each computer has just 1

------------------------------------------------------------
### Monday, January 6, 2020

#### Networking Hurdles to Overcome:
- Operating System
- Endian-ness (Integer size)
- Somehow connected

#### OSI 7 Layer Model
7)Application - most abstract

...

1)Physical Hardware - most concrete

------------------------------------------------------------
### Friday, January 3, 2020

#### Server/Client Design Patterns
-1) **Peer-To-Program** (not exactly Server/Client)
- C_0  <--->  C_1
- Direct connection between clients
- Good for well-structured interactions
0. **Single Server**
	- 1 server
		- Handles all connections
		- Handles all communication
1. **Forking Server**
	- 1 main server
		- Handles all connections
		- Creates subservers to handle all communication
2. **Dispatch Server**
	- 1 main server
		- Handles all connections
		- Handles all incoming data
		- Subservers handle all outgoing data
		- Main server routes data to subservers
		
------------------------------------------------------------
### Wednesday, December 18, 2019

#### How do we flag down a resource?
- Semaphore operations
	- Create a semaphore
	- Set an initial value
	- Remove a semaphore
	- `Up(S)` / `V(S)` - atomic
		- Release the semaphore to signal you are done with its associates resource
		- Pseudocode
			- `S++`
	- `Down(S)` / `P(S)` - atomic
		- Attempt to take the semaphore
		- If the semaphore is 0, wait for it to be available
		- Pseudocode
			- `while (S == 0) { block } S--;`

```c
```

------------------------------------------------------------
### Monday, December 16, 2019

#### Sharing is caring
- Shared Memory
	- `<sys/shm.h>`, `<sys/ipc.h>`, `<sys/types.h>`
	- A segment of heap memory that can be accessed by multiple processes
	- Shared memory is accessed via a key that is known by any process that needs to access it
	- Shared memory does not get released when a program exits
	- 5 Shared memory operations
		- Create the segment (happens once) - `shmget`
		- Access the segment (happens once per process) - `shmget`
		- Attach the segment to a variable (once per process) - `shmat`
		- Detach the segment from a variable (once per process) - `shmdt`
		- Remove the segment (happens once) - `shmctl`

------------------------------------------------------------
### Friday, December 13, 2019

```c
int main() {
	int fd;
	char line[100];
	
	mkfifo("mario", 0640);
	
	fd = open("mario", O_RDONLY);
	printf("fifo open!\n");
	
	remove("mario);
	
	while(read(fd, line, sizeof(line)))
		printf("read: [%s]\n", line);
}
```

------------------------------------------------------------
### Wednesday, November 27, 2019

#### What the fork?
- Managing Sub-Processes
	- `fork() - <unistd.h>`
		- Creates a separate process based on the current one, the new process is called a child, the original is the parent
		- The child process is a duplicate of the parent process
		- All parts of the parent process are copied, including stack and heap memory, and the file table
		- Returns `0` to the child and the child's pid, or `-1` (errno), to the parent

------------------------------------------------------------
### Tuesday, November 26, 2019

#### Executive Decisions (Cont.)
- `strsep - <string.h>`
	- Parse a string with a common delimiter

			strsep( source, delimiters)
		- Locates the first occurence of any of the specified `delimiters` in a string and replaces it with `NULL`
		- `delimiters` is a string, each character is interpreted as a distinct delimiter
		- Returns the beginning of the original string, sets `source` to the string starting at 1 index past the location of the new `NULL`
	- Example:
```c
char line[100] = "woah-this-is-cool";
char *curr = line;
char * token;
token = strsep(&curr, "-");
```
- replaces the `-` after `woah` with `NULL`
- returns a pointer to the `w` in `woah`
- sets `curr` to point to the `t` in `this is cool`

------------------------------------------------------------
### Monday, November 25, 2019

#### Executive Decisions
- the `exec` family - `<unistd.h>`
	- A group of c functions that can be used to run other programs
	- Replaces the current process with the new program
	- `execl`

			execl(path, command, arg0, arg1, ... NULL)
		- `path`
			- The path to the program (ex `"/bin/ls"`)
		- `command`
			- The name of the program (ex: `"ls"`)
		- `arg0 ...`
			- Each command line argument you wish to give the program (ex: `"-a", "-l"`)
			- The last argument must be NULL
	- `execlp`

			execlp(path, command, arg0, arg1, ... NULL)
		- Works like `excel`,except it uses $PATH environment variable for command
	- `execvp`
	
			execvp(path, argument_array)
		- `argument_array`
			- Array of strings containing the arguments to the command
			- `argument_array[0]` must be the name of the program
			- Last entry must be NULL
	- `strsep - <string.h>`
		- Parse a string with a common delimiter

------------------------------------------------------------
### Friday, November 22, 2019

```c
static void sighandler() {
	if (signo == SIGINT) {
		printf("haha! Can't touch this!\n");
	}
	if (signo == SIGSEGV) {
		printf("nothing too see here..\n");
		exit(0);
	}
}

int main() [
	//...
	
	signal(SIGINT, sighandler);
	signal(SIGSEGV, sighandler);
	
	//...
}
```
#### Sending Mixed Signals (Cont.)
- Signal handling in c program `<signal.h>`
	- `sighandler`
		- To intercept signals in a c program you must create a signal handling function
		- Some signals (like SIGKILL, SIGSTOP) cannot be caught
		- `static void sighandler(int signo)`
			- Must be `static`
			- Must be `void`
			- Must take a single `int` parameter

------------------------------------------------------------
### Thursday, November 21, 2019

### Sending Mixed Signals
- Signals
	- Limited way of sending information to a process
	- Sends an integer value to a process
	- `$ kill`
		- Command line utility to send a signal to a process
		- `$ kill pid`
			- Sends signal 15 (SIGTERM) to `pid`
		- `$ kill -signal pid`
			- Sends `signal` to `pid`
	- `$ killall [-signal] process_name`
_Note: killing one of the init processes shuts down your computer_
- Signals in c programs `<signal.h>`
	- kill
		- `kill(pid, signal)`
		- Can use to catch and change behavior when signals recieved

------------------------------------------------------------
### Wednesday, November 20, 2019
#### Are your processes runing? - Then go out and catch them!
- Processes
	- Every running program is a process
	- A process can create subprocesses, but these are no different from regular processes
	- A processor can handle 1 process per cycle (per core)
	- "Multitasking" appears to happen because the processor switches between all the active processes quickly

------------------------------------------------------------
### Tuesday, November 19, 2019

#### You want Input? fget(s) about it! (Cont.)
- `sscanf - <stdio.h>`
	- Reads in data from a string using a format string to determine types
	- `sscanf(char *s, char * format, void * var0, void * var1, ...)`
	- Copies the data into each variable
	- Example usage:
```c
int x; float f; double d;
sscanf(s, "%d %f %lf", &x, &f, &d);
```

------------------------------------------------------------
### Monday, November 18, 2019

#### You want Input? fget(s) about it!
- Command Line Arguments:
	- `int main (int argc, char * argv[])`
	- Program name is considered the first command line argument 
	- `argc`
		- number of command line arguments
	- `argv` (argument vector)
		- array of command line arguments as strings

- `fgets - <stdio.h>`
	- Read in data from a file stream and store it in a string.
	- `fgets( char * s, int n, FILE * f);`
		- Reads at most `n - 1` characters from file stream `f` and stores it in `s`, appends `NULL` to the end. 
	- Stops at newline, end of file, or the byte limit. 
	- File stream
		- `File *` type, more complex than a file descriptor, allows for buffered input. 
		- `stdin` is a `FILE *` variable
	- Example:
  		- `fgets(s, 100, stdin)`

------------------------------------------------------------
### Thursday, November 14, 2019

#### Where do compsci clergy keep their files? - In d'rectory!
All directories are 4096 bytes and are executable. 

Directories 
- A linux directory is a file containing the names of the files within the directory along with basic information, like file type.
- Linux will increase the directory size if needed.

`opendir - <dirent.h>`
  - open a directory file
  - This will ***not*** change the current working directory (cwd), it only allows your program to read the contents of the directory file
  - `opendir(path)`
    - Returns a pointer to a directory stream (`DIR *`)

`readdir - <dirent.h>`
  - `readdir(dir_stream)`
    - Returns a pointer to the next entry in a directory steam, or NULL if all entries have alreadty been returned. 
    -`struct dirent - <sys/types.h>`

------------------------------------------------------------
### Wednesday, November 13, 2019

#### Seek and ye shall find (cont.)
- `stat - <sys/stat.h>`
	- Get information about a file (metadata)
	
			stat(path, stat_buffer)
	- `stat_buffer`
		- Must be a pointer to a `struct stat`
		- All the file information gets put into the stat buffer
		- Some of the fields in `struct stat`:
			- `st_size`
				- file size in bytes
			- `st_uid`, `st_gid`
				- user id, group id
			- `st_mode`
				- file permissions
			- `st_atime`, `st_mtime`
				- last access, last modification
				- these are `time_t` variables, we can use functions in `time.h` to make sense of them
					- `ctime(time)`

------------------------------------------------------------
### Tuesday, November 12, 2019

#### Seek and ye shall find
- `lseek - <unistd.h>`
	- Set the current position in an open file

			lseek(file_descriptor, offset, whence)
	- `offset`
		- Number of bytes to move the position by, can be negative (has to be int)
	- `whence`
		- Where to measure the offset from
		- SEEK_SET
			- offset is evaluated from the beginning of the file
		- SEEK_CUR
			- offset is relative to the current position in the file
		- SEEK_END
			-offset is evaluated from the end of the file
	- Returns the number of bytes the current position is from the beginning of the file , or -1 and sets errno

------------------------------------------------------------
### Friday, November 8, 2019

#### Read your writes!
```c 
  printf("O_RDONLY: \t%d\n", O_RDONLY); // 0
  printf("O_WRONLY: \t%d\n", O_WRONLY); // 1
  printf("O_RDWR:   \t%d\n", O_RDWR);   // 2
  printf("O_CREAT:  \t%d\n", O_CREAT);  // 64
  printf("O_EXCL:   \t%d\n", O_EXCL);   // 128 if a file exists and you want to create it, will return an error 
  printf("O_TRUNC:  \t%d\n", O_TRUNC);  // 512 starts at beginning of file 
  printf("O_APPEND: \t%d\n", O_APPEND); // 1024 starts at end of file
  // each constant in decimal is a 1 with a bunch of 0s after
  // bitwise or (|) will combine them 
```
If you do not set the mode argument when creaing a file, you will get random permissions. 
- ` umask - <sys/stat.h>`
	- set the file creation permission mask
	- By default, created files are not given the exact permissions provided in the mode argument to open. Some permissions are automatically shut off
  ```
    -  mask:         000 000 010 (write for others is shut off)
    - ~mask:         111 111 101 (bit wise negation)
    -  mode:         110 110 110 
    - ~mask & mode : 110 110 101 (bit wise negation of mask, bitwise and mode) 
  - the default linux mask is 002
  - umask(0) //no mask
  ```

- `read - <unistd.h>`
	- read data from a file
	- ` read(fd, buff, n) `
		- read n bytes from fd's file into buffer 
		- returns the number of bytes actually read. Returns -1 and sets errno if unsuccessful. 
		- buff must be a pointer 
```c
  char buff[100];
  int fd = open("foo", 0_RDONLY);
  read(fd, buff, sizeof(buff));
```
  
- `write - <unistd.h>`
	- write data to a file 
	- ` write(fd, buff, n) `
		- write n bytes to fd's file from buffer 
		- returns the number of bytes actually written. Returns -1 and sets errno if unsuccessful. 
		- buff must be a pointer 
    
- `close `
	- closes file 
	- returns -1 and sets errno if unsuccessful.
	- `close(fd)`

------------------------------------------------------------
### Thursday, November 7, 2019

- **File Table (cont.)**
	- Contains where you are in a file
	- Flushing buffers? (smth w/ newlines)
	- ex:

| FD | Name   | Path | Size | ... |
|----|--------|------|------|-----|
| 0  | stdin  |      |      |     |
| 1  | stdout |      |      |     |
| 2  | stderr |      |      |     |
| 3  | goo    |      |      |     |
| 4  | boo    |      |      |     |

`open - <fcntl.h>`
- Add a file to the file table and returns its file descriptor
- If open fails, -1 is returned, extra error information can be found in `errno`
	- `errno` is an int variable that can be found in <errno.h>
	- Use `strerror` in (string.h) on errno to return a string description of the error
- How to use errno example:

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <errno.h>

int main() {
	int fd;
	
	fd = open("tobe", O_RDONLY);
	if ( fd < 0) {
		printf("errno: %d error: %s\n", errno, strerror(errno) );
		return 0;
	}
	printf("fd: %d\n", fd);
	printf("errno: %d error: %s\n", 6, strerror(6) );
	return 0;
}
```
- `open( path, flags, mode)`
	- `mode`
		- Only used when creating a file
		- Set the new file's permissions using a 3 digit octal number
	- `flags`
		- Determine what you plan to do with the rile, use the following constants and combine with |:
			- O_RDONLY
			- O_WRONLY
			- O_RDWR
			- O_APPEND
			- O_TRUNC
			- O_CREAT
			- O_EXCL: when combined with O_CREAT, will return an error if the file exists

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
		- 0 or STDIN_FILENO:  stdin
		- 1 or STDOUT_
FILENO: stdout
		- 2 or STDERR_FILENO: stderr

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
