```cpp
// Created a file named makefile with no extension to contain the following configs

// Optional: Variable initialization & assignment
/* Recursive assignment (=):
 * The value is not calculated when the line is read. Instead Make looks up the value every time the variable is used.
 * Ex:
 * A = $(B)
 * B = hello
 * evaluating $(A) now gives "hello"
 * B = bye
 * evaluating $(A) now gives "bye" */

/* Simple assignment (:= or ::=):
 * The value is calculated once, right at the line where it's defined
 * Ex:
 * B = hello
 * A = $(B)
 * B = bye
 * evaluating &(A) still results in hello */

/* Consditional assignment (?=):
 * This only assigns a value to the variable if it does not already exist 
 * (i.e., it hasn't been set yet in the environment or earlier in the Makefile)
 * Ex.
 * CC ?= gcc
 * If you ran 'CC=clang make', CC stays 'clang'
 * If you just ran 'make', CC becomes 'gcc' */

CXX := g++
CXXFLAGS := -Wall -Wextra -Werror -std=c++20

// Rule Structure
target: dependency1 dependency2 ...
	$(CXX) $(CXXFLAGS) dependency1 -o target
	

/* Below are some pattern rules:
 * %: wild card character
 * $^: the file names of all prerequesites/dependencies
 * $@: the exact file name of the target
 * $<: the file name of the first prerequesite/dependency
 * $*: the text matched by the %
 */
%.o: %.cc
	$(CXX) $(CXXFLAGS) $^ -o $@

// Targets that don't produce actual files are Phony Targets
// .PHONY is a special built-in target name to declare phony targets
// Doesn't need to be declared before use
.PHONY: all clean

// Examples of common phony targets
clean:
	rm -rf *.o main// removes binaries
all:
	$(CXX) $(CXXFLAGS) *.cc -o main
	
	
// command: make <target>
```