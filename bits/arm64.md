# arm64 assembly crash course

_Note: May or may not be specific to iOS._

### Registers

-   General purpose registers 0 through 30 with two addressing modes:
    -   `w0` = 32-bit
    -   `x0` = 64-bit
-   Zero register `wzr` or `xzr`. Write to = discard, read from = `0`.
-   Stack pointer `sp` - unlike other instruction sets, never modified implicitly (e.g. no `push`/`pop`).
-   Instruction pointer `pc`, not modifiable directly.
-   A lot of float, vector and system registers, look up as needed.
-   First register in assembly is usually destination, rest are source (except for `str`).

### Register manipulation

-   `mov` to copy one register to another, e.g. `mov x0, x1` -> `x0 = x1`.
-   Constant `0` loaded from `wzr`/`xzw`.
-   Small constants usually OR'ed with zero register, e.g. `orr x0, xzr, 5`.
-   Big constants usually loaded with `movz`+`movk`, e.g.:
    ```asm
    movz x0, 0x1234, lsl 32
    movk x0, 0x5678, lsl 16
    movk x0, 0x9abc
    ```
    -> `x0 = 0x123456789abc`.
-   `movn` for negative values, e.g. `movn x0, 1` -> `x0 = -1`.
-   `lsl` and `lsr` instructions = logic-shift-left and logic-shift-right, e.g. `lsl x0, x0, 8` -> `x0 <<= 8`.
    -   `lsl` and `lsr` not only used as instructions, but also as operands to other instructions (see `movz` above).
    -   `asl` for arithmetic shift also exists, but less frequently used.
-   Lots of arithmetic, logic and bitwise instructions, look up as needed.

### Memory

-   `ldr` and `str` with multiple variations and addressing modes:
    -   `ldr x0, [x1]` -> `x0 = *x1`
    -   `str x0, [x1]` -> `*x1 = x0`
    -   `ldr x0, [x1, 0x10]` -> `x0 = *(x1 + 0x10)`
    -   `ldp`/`stp` to load/store two registers at once behind each other, e.g.:  
        `stp x0, x1, [x2]` -> `*x2 = x0; *(x2 + 8) = x1;`
    -   Multiple variations for load/store size:
        -   Register names `xN` for 64-bit, `wN` for 32-bit
        -   `ldrh`/`srth` for 16-bit
        -   `ldrb`/`strb` for  8-bit
    -   Multiple variations for sign-extending registers smaller than 64-bit:
        -   `ldrsw x0, [x1]` -> load 32-bit int, sign extend to 64-bit
        -   `ldrsh x0, [x1]` -> load 16-bit int, sign extend to 64-bit
        -   `ldrsb x0, [x1]` -> load  8-bit int, sign extend to 64-bit
        -   (No equivalent `str` instructions)
-   Three register addressing modes:
    -   Normal: `ldr x0, [x1, 0x10]`
    -   Pre-indexing: `ldr x0, [x1, 0x10]!` (notice the `!`) -> `x1 += 0x10; x0 = *x1;`
    -   Post-indexing: `ldr x0, [x1], 0x10` -> `x0 = *x1; x1 += 0x10;`
-   Memory addresses usually computed by PC-relative instructions:
    -   `adr x0, 0x12345` (only works for small offset from PC)
    -   Bigger ranges use `adrp`+`add`:
        ```asm
        adrp x0, 0xffffff8012345000 ; "address of page", last 12 bits are always zero
        add x0, x0, 0x678
        ```
    -   Even bigger ranges usually stored as pointers in data segment, offset by linker and loaded with `ldr`.

### Calling convention

_Note: Only dealing with integral types here. The rules change when floating-point is involved._

-   `x0`-`x7` first 8 arguments, rest on the stack (low address to high) with natural alignment (as if they were members of a struct)
-   `x8` pointer to where to write the return value if >128 bits, otherwise scratch register
-   `x9`-`x17` scratch registers
-   `x18` platform register (reserved, periodically zeroed by XNU)
-   `x19`-`x28` callee-saved
-   `x29` frame pointer (basically also just callee-saved)
-   `x30` return address
-   Functions that save anything in `x19`-`x28` usually start like this:
    ```asm
    stp x24, x23, [sp, -0x40]!
    stp x22, x21, [sp, 0x10]
    stp x20, x19, [sp, 0x20]
    stp x29, x30, [sp, 0x30]
    add x29, sp, 0x30
    ```
    and end like this:
    ```asm
    ldp x29, x30, [sp, 0x30]
    ldp x20, x19, [sp, 0x20]
    ldp x22, x21, [sp, 0x10]
    ldp x24, x23, [sp], 0x40
    ret
    ```
    The stack for local variables is usually managed separately though, with `add sp, sp, 0x...` and `sub sp, sp, 0x...`.
-   Variadic arguments are passed on the stack (low address to high), each promoted to 8 bytes. Structs that don't fit into 8 bytes have a pointer passed instead.  
    Fixed arguments that don't fit into `x0`-`x7` come before variadic arguments on the stack, naturally aligned.
-   The return value is passed as follows:
    -   If it fits into 64 bits, in `x0`.
    -   If it fits into 128 bits, the first/lower half in `x0`, the second/upper half in `x1`.
    -   If it is larger than 128 bits, the caller passes a pointer in `x8` to where the result is written.

### Conditions

-   System register `nzcv` holds condition flags (Negative, Zero, Carry, oVerflow).  
    Set by one instruction and acted upon by a subsequent one, the latter using condition codes.  
    (Could be accessed as normal system register, but usually isn't.)
-   Some instructions use condition codes as suffixes (`instr.cond`), others as source operands (`instr ..., cond`). List of condition codes:
    -   `eq`/`ne` = equal/not equal
    -   `lt`/`le`/`gt`/`ge` = less than/less or equal/greater than/greater or equal (signed)
    -   `lo`/`ls`/`hi`/`hs` = lower/lower or same/higher/higher or same (unsigned)
    -   A few more weird flags, seldom used.
    -   Unlike many other instruction sets, arm64 sets carry on no-borrow rather than borrow.  
        Thus, `cs`/`cc` = carry set/carry clear are aliases of `hs`/`lo`.
-   `cmp` = most common/basic compare instruction, sets condition flags. Examples:
    ```asm
    cmp x0, x1
    cmp x0, 3
    ```
-   Other instructions that set condition flags:
    -   `cmn` = compare negative
    -   `tst` = bitwise test
    -   `adds`/`adcs` = add/add with carry
    -   `subs`/`sbcs` = subtract/subtract with carry
    -   `negs`/`ngcs` = negate/negate with carry
    -   Some more bitwise and float instructions.
-   Some instructions that act on condition flags:
    -   `cset` = conditional set, e.g.:
        ```asm
        cmp x0, 3
        cset x0, lo
        ```
        -> `x0 = (x0 < 3)`
    -   `csel` = conditional select, e.g.:
        ```asm
        cmp x0, 3
        csel x0, x1, x2, lo
        ```
        -> `x0 = (x0 < 3) ? x1 : x2`  
        (Translates nicely to ternary conditional.)
    -   `ccmp` = conditional compare, e.g.:
        ```asm
        cmp x0, 3
        ccmp x0, 7, 2, hs
        b.hi 0xffffff8012345678
        ```
        -> `hi` condition will be true if `x0 < 3 || x0 > 7` (third `ccmp` operand is raw `nzcv` data).  
        Often generated by compiler for logical and/or of two conditions.
    -   Many, many more.

### Branches

-   `b` = simple branch, jump to PC-relative address.  
    Can be unconditional:
    ```asm
    b 0xffffff8012345678
    ```
    or conditional:
    ```asm
    cmp x0, 3
    b.lo 0xffffff8012345678 ; jump to 0xffffff8012345678 if x < 3
    ```
    Used primarily within function for flow control.
-   Shortcuts `cbz`/`cbnz` = compare-branch-zero and compare-branch-non-zero.  
    Just shorter ways to write
    ```asm
    cmp xN, 0
    b.eq 0x...
    ```
    or
    ```asm
    cmp xN, 0
    b.ne 0x...
    ```
    (Translate nicely to C `if(x)` or `if(!x)`.)
-   Shortcurts `tbz`/`tbnz` = test single bit and branch if zero/non-zero.  
    E.g. `tbz x0, 3, ...` translates to `if((x0 & (1 << 3)) == 0) goto ...`.
-   `bl` = branch-and-link (e.g. `bl 0xffffff8012345678`)  
    Store return address to `x30` and jump to PC-relative address. Used for static function calls.
-   `blr` = branch-and-link to register (e.g. `blr x8`)  
    Store return address to `x30` and jump to address in `x8`. Used for calls with function pointers or C++ virtual methods.
-   `br` = branch to register (e.g. `br x8`)  
    Jump to address in `x8`. Used for tail calls.
-   `ret` = return to address in register, default: `x30`  
    Can in theory use registers other than `x30` (e.g. `ret x8`), but compiler doesn't usually generate that.
