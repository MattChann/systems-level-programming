# Notes for Systems Level Programming Class

------------------------------------------------------------
### DayOfWeek, Month Date, Year

#### Topic

------------------------------------------------------------
### DayOfWeek, Month Date, Year

#### Topic

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
Example makefile:
'''
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
'''

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
