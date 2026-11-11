LogicC Language Reference Specification and Manual

Overview and Philosophy
LogicC is a low-level interpreted programming language implemented in Steel Bank Common Lisp (SBCL). It provides transparent assembly-like memory control and structured execution statements.

Instruction Syntax Formula
Instructions follow the pattern of an optional variable assignment, an action verb, a target or operand, optional flag attributes prefixed with a hyphen, and mandatory parser punctuation at the end of the line.

Punctuation and Line Termination Rules
Line termination is strictly dictated by punctuation:
A period (.) strictly terminates atomic execution lines and variable assignments.
A colon (:) opens code blocks, function definitions, system labels, entry points, or conditional checks.
Single-line comments begin with two slashes (//). Everything following slashes until the end of the line is ignored by the parser.

Indentation Rules
Indentation dictates block scoping for check and else structures. One tab character is processed as equivalent to four spaces.

Virtual Machine Architecture and RAM Limits
The interpreter maintains an internal contiguous byte array of 16,777,216 bytes (16 Megabytes) representing system memory. Memory below address 1024 is reserved for low-level addressing, while the dynamic heap allocation pointer starts at address 1024.
All multi-byte integer values written to or read from memory use 32-bit (4-byte) little-endian byte ordering.
Calculations operate exclusively on signed and unsigned integers, as floating-point numbers are not supported. Division or modulo operations with a zero divisor return zero to prevent crashes.

Command and Attribute Reference

entry:
Defines the main execution entry point of the program.

exit
Halts machine execution. Supports the attribute -code:N to return a status code (default is 0).
Example:
exit -code:0.

out
Outputs string literals or variable contents to an output stream.
Supported attributes:
-target specifies the output stream, such as console.
-raw suppresses standard trailing line breaks.
-spec enables escape sequence parsing, including \n for newline, \r for carriage return, \t for tab, and \e for ANSI escape sequences.
-mode sets output modes.
Example:
out "Welcome\n" -spec -target:console.

read
Reads input from standard console input, streams binary file contents directly into RAM, or reads 32-bit integers from a RAM address.
Supported attributes:
-source specifies standard console input, a file path, or a memory address.
-offset specifies byte displacement relative to the base address.
-type specifies data size.
-until sets input termination flags.
-else_error specifies a fallback label to jump to if file opening or reading fails.
Examples:
nick = read -source:console.
val = read -source:buffer -offset:4.
read -source:"data.bin" -target:bufferAddress.

alloc
Reserves dynamic memory on the heap and returns the starting base address.
Supported attributes:
-size sets allocation length in bytes.
-heap targets heap space.
-else_error specifies a fallback label to jump to if memory allocation exceeds the 16 Megabyte RAM limit.
Example:
ptr = alloc -size:1024 -heap -else_error:mem_error.

free
Releases dynamic heap handles and supports the -heap attribute.
Example:
free ptr -heap.

fill
Fills a range of memory bytes with a constant byte value.
Supported attributes:
-value specifies the byte value (supports hex notation like 0x00).
-bytes specifies total range length in bytes.
Example:
fill ptr -value:0x00 -bytes:1024.

write
Writes a 32-bit integer into memory across four bytes in little-endian order at a target address plus an optional offset.
Supported attributes:
-target specifies the base memory address.
-offset specifies byte displacement.
-type specifies data type representation (such as i32).
-else_error specifies a fallback label if the write address falls outside the range of 0 to 16,777,215.
Example:
write 42 -target:ptr -offset:4 -type:i32.

wait
Pauses execution for a specified duration in milliseconds.
Supported attributes:
-ms specifies duration in milliseconds.
-target sets execution context.
-status tracks return status.
Example:
wait -ms:500.

jump
Performs an unconditional transfer of execution to a target label.
Example:
jump label_name.

check and else
Evaluates conditions using binary operators (==, !=, >, <, >=, <=). Supports single-line redirection via -jump:Label or indented block structures with an optional else: branch.
Examples:
check A == 1 -jump:label_win.

check A == 10:
out "A is 10" -target:console.
else:
out "A is not 10" -target:console.

fn and return
Defines subroutines and returns execution back to the caller.
Supported fn attributes:
-in specifies formal input parameters.
-out specifies output parameter buffers.
-macro designates inline macro handling.
Subroutines are called using the call command, passing input arguments via -in attributes. The return command pops the call stack.
Examples:
fn PrintValue -in:val:
out val -target:console.
return.

entry:
call PrintValue -in:100.
exit -code:0.

repeat
Loops subroutine execution either a fixed number of times using -count or continuously while a condition remains true using -while.
Supported attributes:
-count specifies total iteration count.
-while specifies conditional predicate.
Examples:
repeat PrintDot -count:10.
repeat PrintDot -while:A < 5.

include
Textually embeds external .lc files during pre-processing.
Example:
include "utils.lc"

Mathematical and Bitwise Operator Precedence

Infix expressions evaluate across four precedence levels:
Level 1 (Highest): Multiplication (*), Division (/), Modulo (%).
Level 2: Addition (+), Subtraction (-).
Level 3: Left Shift (<<), Right Shift (>>).
Level 4 (Lowest): Bitwise AND (&), Bitwise XOR (^), Bitwise OR (|).

Example:
entry:
A = 10.
B = 20.
C = A + B * 2.
Masked = C & 0xFF.
Shifted = Masked << 2.
exit -code:0.

Complete Practical RAM and Functions Example

fn DisplayResult -in:res:
out "Read value: " -raw.
out res.
return.

entry:
MemBlock = alloc -size:1024 -heap -else_error:label_mem_err.
fill MemBlock -value:0x00 -bytes:1024.
write 42 -target:MemBlock -offset:0 -type:i32.
write 100 -target:MemBlock -offset:4 -type:i32.
Val = read -source:MemBlock -offset:4.
call DisplayResult -in:Val.
free MemBlock -heap.
exit -code:0.

label_mem_err:
out "Memory allocation error!" -target:console.
exit -code:1.