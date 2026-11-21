# First Binary

## Disassembled content

```nasm
080483fb <main>:
 80483fb:       8d 4c 24 04             lea    ecx,[esp+0x4]
 80483ff:       83 e4 f0                and    esp,0xfffffff0
 8048402:       ff 71 fc                push   DWORD PTR [ecx-0x4]
 8048405:       55                      push   ebp
 8048406:       89 e5                   mov    ebp,esp
 8048408:       51                      push   ecx
 8048409:       83 ec 04                sub    esp,0x4
 804840c:       83 ec 0c                sub    esp,0xc
 804840f:       68 b0 84 04 08          push   0x80484b0
 8048414:       e8 b7 fe ff ff          call   80482d0 <puts@plt>
 8048419:       83 c4 10                add    esp,0x10
 804841c:       b8 00 00 00 00          mov    eax,0x0
 8048421:       8b 4d fc                mov    ecx,DWORD PTR [ebp-0x4]
 8048424:       c9                      leave  
 8048425:       8d 61 fc                lea    esp,[ecx-0x4]
 8048428:       c3                      ret    
 8048429:       66 90                   xchg   ax,ax
 804842b:       66 90                   xchg   ax,ax
 804842d:       66 90                   xchg   ax,ax
 804842f:       90                      nop
```

## Key Concepts

### Registers (x86_64)

   `ecx` -> extended counter register -> usually used for storing the fourth parameter of a function or as a counter.

   `esp` -> extended stack pointer. -> points to the top of the stack.

   `ebp` -> extended base pointer. -> points to the bottom of the stack frame.

   `eax` -> holds function return values
   
   `ax` -> lower 2 bytes of eax.  

   `eip` -> extended instruction pointer

### Main Instructions Found

#### x86 Assembly Instructions

- **`lea destination, [calculated_address]`**  
`lea` calculates an address and stores it inside `destination`.

- **`and firstNumber, secondNumber`**  
  Applies an `AND` operation between two numbers. It compares the two numbers bit by bit and gives `1` if both bits are `1`, otherwise `0`.

- **`push x`**  
  `x` is pushed into the stack. If `x` occupies `y` bytes, the stack pointer register is subtracted by `y` positions. Then `y` bytes will be occupied by `x`.

- **`mov destination, source`**  
  Moves `source` content into `destination`. Don't be tricked by the command name. `source`'s content remains intact. This is a sort of copy command rather than a move command.

- **`sub destination, source`**  
  `destination` will be equal to `destination - source`.

- **`call function_address`**  
  This command calls a function by specifying its address in memory.

- **`add destination, source`**  
  `destination` will be equal to `destination + source`.

- **`leave`**  
  It does two things:
  1. `mov esp, ebp` -> discarding all local variables from the function.
  2. `pop ebp` -> restore the caller's base pointer.

- **`ret`**  
  It takes the last 4 bytes from the stack and stores them inside `eip`.

- **`xchg dest, src`**  
  Stands for exchange. It swaps the `src` operand with the `dest` operand. It's like doing three `mov` operations:
  1. Moving `dest` to a temporary space (another register).
  2. Moving `src` to `dest`.
  3. Moving the temporary register content to `src`.

- **`nop`**  
  Stands for "no operation". It tells the CPU to literally do nothing and advance to the next instruction.

### Memory Alignment Note
Data in memory is aligned when its address in memory is divisible by its dimension in bytes. A 4-byte integer is aligned in memory if its address is divisible by `4`. Aligned data is read by the CPU faster. 

The same thing happens for functions. The CPU usually reads stuff in memory in chunks (usually 16 bytes). If a function occupies 37 bytes and the set alignment size is 16 bytes, it might add 11 bytes of `nop` instructions so that the next function starts at an address that is divisible by `16` (`48`).
 

## Sample Assembly Breakdown

**Objective:**

- **`lea ecx,[esp+0x4]`**

  It calculates the address esp+0x4 and then moves it inside ecx (general purpose register).

  Before main is called the stack looks basically like this:

    ARGV

    ARGC

    Next Instruction Address (return address)

  esp correspond to the current position in the stack. esp+0x4 points to the address of the first Byte of the 4 Bytes set containing argc value.

- **`and esp,0xfffffff0`**

  This is an and operation.
  The and operator compare bit by bit the two numbers. It gives 1 if both bits are 1, otherwise 0. 

  0xfffffff0 in binary is -> 11111111 11111111 11111111 11110000

  This operation is gonna make the last 4 bits from esp 0 no matter what bit it's, and will maintain intact the others bits.

  This operation is needed for ensuring a 16Byte data alignment, meaning that esp, after the and operation, by having the last 4 bits zero will have an address that is divisible by 16.

  In order to understand we can make an example using base-10 numbers. In order to make a base-10 number, like 37, a new number that is divisible by 10 you can just remove the last unit (in this case 7) and put a 0 instead.

  The same thing happens for hexadecimal numbers. Let's suppose:

  ```text
  esp = 0x00000078
  in binary -> 00000000 00000000 00000000 0111 1000
  ```

  To make the `esp` address divisible by 16, you can just replace `8` (the last character) with `0` by doing an `AND` operation that targets the last 4 bits (1 hex character = 4 bits).

  `and esp, 0xfffffff0`:

  ```text
  -> 0x00000078: 00000000 00000000 00000000 0111 1000
  -> 0xfffffff0: 11111111 11111111 11111111 1111 0000
  -> and       : 00000000 00000000 00000000 0111 0000 -> 0x00000070
  ```

The address stored in `esp` is now `0x00000070`, which is divisible by 16.
 
- **`push   DWORD PTR [ecx-0x4]`**

  WORD: 2 Bytes
  DWORD: DoubleWord : 4 Bytes

  4 Bytes of data is pushed into the stack. That data is the return address, meaning the address of the instruction to execute after main function ends.

- **`push ebp`**

  The previous stack frame base pointer is pushed into the stack. We need to store info regarding the previous stack frame.

- **`mov ebp, esp`**

  esp address is moved (copied, actually, as esp content remains intact) from esp into ebp. At this point both ebp and esp points to the same point in stack.

  Everything that was done till this point is all basically setup. ebp indicates the current stack frame base, so usually it stays fixed, while esp tends to move constantly. 

- **`push ecx`**

  ecx (contains argc) is pushed into the stack

- **`sub esp, 0x4`**  and **`sub esp, 0xc`**

  esp decreased by 16 positions

- **`push 0x80484b0`**

  The address that points to the string that is gonna be read by puts function is pushed onto the stack.

   (Note: In GDB the command `x/s 0x80484b0` ask GDB to examine memory at the specified address and interpret it as a string)

- **`call   80482d0 <puts@plt>`**

  call does two things:
  1) The next instruction is located at address 0x8048419. So this value is pushed onto the stack, so that when the called function ends the CPU just pop the address and continues from there.
  2) The function at address 0x80482d0 is executed.

  Modern compilers like GCC replace `printf()` with `puts()` whenever you print a plain string with no format specifiers, like %d or %s. Puts is much faster compared to printf.

  Why <puts@plt> ? 

  The operating system randomizes where shared libaries libraries sit in RAM every time you launch the program. (because of a security feature called ASLR, the Address Space Layout Randomization).

  NOte: Shared libraries are files containing pre-compiled machine code that multiple programs can load and share in memory at runtime. On Windows these have the dll extension (Dynamic Link Libraries). On linux its .so, meaning Shared Object Files(for example libc.so is the shared library containing puts function)

  PLT stands for Procedure Linkage Table. It acts as a sort of intermediary between your executable and GOT (Global Offset Table), the table that actually contains the addresses for each function.

  PLT is just a small section of actual machine code inside the program that access the GOT and select the address for the function we need.
  
  <puts@plt> might look like this when called for the first time:

  ```nasm
  0x80482d0 <puts@plt>:
  jmp    DWORD PTR [0x804a00c]   ; 1. Jump to whatever address is written inside GOT[puts]
  push   0x0                     ; 2. Push the ID number for "puts" onto the stack (0x0 refers to puts)
  jmp    0x80482c0               ; 3. Jump to the dynamic linker to find the real address (the dynamic linker will select the previously pushed value (0x0) as the ID for the function we're searching for (puts) and search it's real address.
  ```
  The dynamic linker is a helper program whose main job is to connect your program to external shared libraries. After finding the real address for puts it will overwrite the current addresss (0x804a00c) with the real address for puts.

- **`mov eax,0x0`**

  Moves 0 into eax. eax is the register tipically used for the return value. The main function, indeed, returns 0 if everything was fine. 

- **`mov ecx,DWORD PTR [ebp-0x4]`**

  Moves the previously pushed ecx value (that contained the return address) into ecx.

- **`leave`**

  It does two things:
  1. `mov esp, ebp` -> It discards all local variables from the function.
  2. `pop ebp` -> Then it restores the caller's base pointer.

- **`lea esp,[ecx-0x4]`**

  The address ecx-0x4 (contains the address for the next instruction, the return address) is moved into esp.

- **`ret`**

  Takes the top 4 bytes (data pointed by esp) and store it inside eip (extended instruction pointer). The CPU now continues from the correct instruction address. 

- 
  ```nasm
  8048429:       66 90                   xchg   ax,ax 
  804842b:       66 90                   xchg   ax,ax
  804842d:       66 90                   xchg   ax,ax
  804842f:       90                      nop
  ```
  The last instructions are used for aligning the code into memory. They aren't part of the actaul logic of main. 0x804842f, the last address, is indeed divisible by 16.

**Putting everything together. General View**

The main function just prints "hello world!" into output. We can see which is the parameter passed to puts function by giving this command to GDB: `x/s 0x80484b0` while analizing the binary file.

# Second Binary

## Disassembled Content

```nasm
080483fb <main>:
 80483fb:       8d 4c 24 04             lea    ecx,[esp+0x4]
 80483ff:       83 e4 f0                and    esp,0xfffffff0
 8048402:       ff 71 fc                push   DWORD PTR [ecx-0x4]
 8048405:       55                      push   ebp
 8048406:       89 e5                   mov    ebp,esp
 8048408:       51                      push   ecx
 8048409:       83 ec 14                sub    esp,0x14
 804840c:       c7 45 f4 0a 00 00 00    mov    DWORD PTR [ebp-0xc],0xa
 8048413:       83 7d f4 0a             cmp    DWORD PTR [ebp-0xc],0xa
 8048417:       75 10                   jne    8048429 <main+0x2e>
 8048419:       83 ec 0c                sub    esp,0xc
 804841c:       68 c0 84 04 08          push   0x80484c0
 8048421:       e8 aa fe ff ff          call   80482d0 <puts@plt>
 8048426:       83 c4 10                add    esp,0x10
 8048429:       b8 00 00 00 00          mov    eax,0x0
 804842e:       8b 4d fc                mov    ecx,DWORD PTR [ebp-0x4]
 8048431:       c9                      leave  
 8048432:       8d 61 fc                lea    esp,[ecx-0x4]
 8048435:       c3                      ret    
 8048436:       66 90                   xchg   ax,ax
 8048438:       66 90                   xchg   ax,ax
 804843a:       66 90                   xchg   ax,ax
 804843c:       66 90                   xchg   ax,ax
 804843e:       66 90                   xchg   ax,ax
```

## Sample Assembly Breakdown

  This binary code is similar to the previous one. 

### The only main difference is that there is an if / then statement: 

```nasm
mov DWORD PTR [ebp-0xc], 0xa
cmp DWORD PTR [ebp-0xc], 0xa
jne 8048429 <main+0x2e>
```
  Firstly 0xa (10 in base-10) is moved into the address ebp-0xc
  The values 0xa and ebp-0xc are compared. 

- **`cmp DWORD PTR [ebp-0xc], 0xa`**

  The `cmp a, b`command basically does this: `a - b = result`. If result is positive then the first value is greater than the second one. If result is negative the second value is greater than the first one and if the result is 0 then the values are equal.

  **Regarding EFLAGS**

  Eflags are bits that live inside the EFLAGS register (4B register for x86 systems). Each bit from this register have a specific meaning. The important ones to know just to start are these:
  • CF (Carry Flag, Bit 0): Unsigned out of bounds
    If in the addition the unsigned number go past its max :\
    `cmp a, b` -> CF=1 if a+b go past the max value the ceiling (for example the max value for a 1 Byte number is 2^8-1, meaning 255) or below the floor (unsigned integers cannot be negative, 0 is the minimum value) 
  • ZF (Zero Flag, Bit 6): The result is zero (or two compared values are equal).
    After `cmp a, b` if result is 0 then ZF = 1. Otherwise ZF = 0\
    After `sub` or `and` ZF = 1 if the result hit 0.
  • SF (Sign Flag, Bit 7): The result is negative (the leftmost bit is 1)).
    SF=1 means the leftmost bit is 1, then the number is negative.\
    SF=0 means the leftmost bit is 0, then the number is positive (or zero).
  
- **`jne 8048429 <main+0x2e>`**

  If the previously compared value are not equal the CPU will jump to the address 0x8048429 (main address + 0x2e). Otherwise it will just continue with the next instructions.

### SECOND MAIN SECTION

```nasm
push 0x80484c0
call 80482d0 <puts@plt>
```
  The address 0x80484c0 is pushed onto the stack. puts function will take the value pointed from that address and use it as a parameter.
  
## Putting everything together. General View

By executing `x/s 0x80484c0` on GDB we see:
```nasm
(gdb) x/s 0x80484c0
0x80484c0:      "x = ten"
``` 
The program just prints on output: "x = ten"

# Third Binary

## Disassembly Content For Main

```nasm
0x080483fb <+0>:     lea    ecx,[esp+0x4]
0x080483ff <+4>:     and    esp,0xfffffff0
0x08048402 <+7>:     push   DWORD PTR [ecx-0x4]
0x08048405 <+10>:    push   ebp
0x08048406 <+11>:    mov    ebp,esp
0x08048408 <+13>:    push   ecx
0x08048409 <+14>:    sub    esp,0x14
0x0804840c <+17>:    mov    DWORD PTR [ebp-0xc],0x0
0x08048413 <+24>:    jmp    0x804842c <main+49>
0x08048415 <+26>:    sub    esp,0x8
0x08048418 <+29>:    push   DWORD PTR [ebp-0xc]
0x0804841b <+32>:    push   0x80484c0
0x08048420 <+37>:    call   0x80482d0 <printf@plt>
0x08048425 <+42>:    add    esp,0x10
0x08048428 <+45>:    add    DWORD PTR [ebp-0xc],0x1
0x0804842c <+49>:    cmp    DWORD PTR [ebp-0xc],0x13
0x08048430 <+53>:    jle    0x8048415 <main+26>
0x08048432 <+55>:    mov    eax,0x0
0x08048437 <+60>:    mov    ecx,DWORD PTR [ebp-0x4]
0x0804843a <+63>:    leave
0x0804843b <+64>:    lea    esp,[ecx-0x4]
0x0804843e <+67>:    ret
```

### Main section (loop)

```nasm
0x08048413 <+24>:   jmp    0x804842c <main+49>
0x08048415 <+26>:    sub    esp,0x8
0x08048418 <+29>:    push   DWORD PTR [ebp-0xc]
0x0804841b <+32>:    push   0x80484c0
0x08048420 <+37>:    call   0x80482d0 <printf@plt>
0x08048425 <+42>:    add    esp,0x10
0x08048428 <+45>:    add    DWORD PTR [ebp-0xc],0x1
0x0804842c <+49>:    cmp    DWORD PTR [ebp-0xc],0x13
0x08048430 <+53>:    jle    0x8048415 <main+26>
```

  This section of disassembled code represents a loop. In the first line the CPU jumps to instruction on address 0x804842c. The instruction in that address is `cmp    DWORD PTR [ebp-0xc],0x13`. This instruction compares the value 0x13 (19 in base decimal) with the value inside ebp-0xc address. If those values are equal or the first one is lesser than the second one the cpu will jump to the instruction at address 0x08048415 (The loop will continue).

  In the loop block of code two values are pushed onto the stack:\
```nasm
0x08048418 <+29>:    push   DWORD PTR [ebp-0xc]
0x0804841b <+32>:    push   0x80484c0
```
  The value inside ebp-0xc, that is a 4B value (probably an integer), and the value at address 0x80484c0 are pushed onto the stack. Both of those values will be used by the printf function as parameters. 

  After that 0x1 (1) is added to the value inside the address ebp-0xc.
  This loop continues as long as the value contained inside ebp-0xc is <= 0x13 (19), meaning the loop will repeat itself 20 times (starting from zero, till 19).

  The command `x/s 0x80484c0` prints out:\
```nasm
(gdb) x/s 0x80484c0
0x80484c0:      "%d "
```
  This means that the value at address `0x80484c0` is the format string, and it indicates the format specifier for the second parameter, that is the one that is firstly pushed onto the stack: `push   DWORD PTR [ecx-0xc]`.

  This can feel quite strange because it would be much intuitive pushing onto the stack the format specifier and then the value we wanna print, but in this case it happens the opposite because the calling convention used by printf function here is `cdecl`, where arguments are pushed from right to left.

  The loop execute `printf("%d", i)` for every value inside ebp-ecx address.  The initial value inside that address is 0:\
`0x0804840c <+17>:    mov    DWORD PTR [ebp-0xc],0x0`

  For this reason the loop will start with [ebp-0xc] at 0 and print: 0, 1, ..., 19


# END

These were my first analyzed binary code and i think it wasn't that bad. Once you understand that most of the complexity derives from the CPU setup decisions and can actually select the important parts of the binary data, everything is easier.

My english is not that good :( sorry for that, but it's not my first language. Nonetheless, i hope that whoever read all this understood even only a bit. Bye bye
