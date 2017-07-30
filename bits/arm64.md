# arm64 assembly crash course

### Registers

- General purpose registers 0 through 30 with two addressing modes:
  - `w0` = 32-bit
  - `x0` = 64-bit
- Zero register `wzr` or `xzr`. Write to = discard, read from = `0`.
- Stack pointer `sp` - unlike other instruction sets, never modified implicitly (e.g. no `push`/`pop`).
- Instruction pointer `pc`, not modifiable directly.
- A lot of float, vector and system registers, look up as needed.
- First register in assembly is usually destination, rest are source (except for `str`).

### Register manipulation

- `mov` to copy one register to another, e.g. `mov x0, x1` -> `x0 = x1`.
- Constant `0` loaded from `wzr`/`xzw`.
- Small constants usually OR'ed with zero register, e.g. `orr x0, xzr, 5`.
- Big constants usually loaded with `movz`+`movk`, e.g.:
    ```asm
    movz x0, 0x1234, lsl 32
    movk x0, 0x5678, lsl 16
    movk x0, 0x9abc
    ```
  -> `x0 == 0x123456789abc`.
- `movn` for negative values, e.g. `movn x0, 1` -> `x0 == -1`.
- `lsl` and `lsr` instructions = logic-shift-left and logic-shift-right, e.g. `lsl x0, x0, 8` -> `x0 <<= 8`.
  - `lsl` and `lsr` not only used as instructions, but also as operands to other instructions (see `movz` above).
  - `asl` for arithmetic shift also exists, but less frequently used.
- Lots of arithmetic, logic and bitwise instructions, look up as needed.

### Memory

- `ldr` and `str` with multiple variations and addressing modes:
  - `ldr x0, [x1]` -> `x0 = *x1`
  - `str x0, [x1]` -> `*x1 = x0`
  - `ldr x0, [x1, 0x10]` -> `x0 = *(x1 + 0x10)`
  - `ldp`/`stp` to load/store two registers at once behind each other, e.g.:  
    `stp x0, x2, [x2]` -> `*x2 = x0; *(x2 + 8) = x1;`
  - Multiple variations for load/store size:
    - Register names `xN` for 64-bit, `wN` for 32-bit
    - `ldrh`/`srth` for 16-bit
    - `ldrb`/`strb` for  8-bit
  - Multiple variations for sign-extending registers smaller than 64-bit:
    - `ldrsw x0, [x1]` -> load 32-bit int, sign extend to 64-bit
    - `ldrsh x0, [x1]` -> load 16-bit int, sign extend to 64-bit
    - `ldrsb x0, [x1]` -> load  8-bit int, sign extend to 64-bit
    - (No equivalent `str` instructions)
- Three register addressing modes:
  - Normal: `ldr x0, [x1, 0x10]`
  - Pre-indexing: `ldr x0, [x1, 0x10]!` (notice the `!`) -> `x1 += 0x10; x0 = *x1;`
  - Post-indexing: `ldr x0, [x1], 0x10` -> `x0 = *x1; x1 += 0x10;`
- Memory addresses usually computed by PC-relative instructions:
  - `adr x0, 0x12345` (only works for small offset from PC)
  - Bigger ranges use `adrp`+`add`:
        ```asm
        adrp x0, 0xffffff8012345000 ; "address of page", last 12 bits are always zero
        add x0, x0, 0x678
        ```
  - Even bigger ranges usually stored as pointers in data segment, offset by linker and loaded with `ldr`.

### Calling convention

- `x0`-`x7` first 8 arguments, rest on the stack (low address to high)
- `x8`-`x17` scratch registers
- `x18` reserved (unused)
- `x19`-`x28` callee-saved
- `x29` frame pointer (basically also just callee-saved)
- `x30` return address
- Functions that save anything in `x19`-`x28` usually start like this:
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

### Conditions and branches

- `cmp` = general compare instruction, sets condition flags.  
  Can be used against registers or small constants:
    ```asm
    cmp x0, x1
    cmp x0, 3
    ```
- `b.cond` = conditional branch instruction, where `cond` can be:
  - `eq`/`ne` = equal/not equal
  - `lt`/`le`/`gt`/`ge` = less than/less or equal/greater than/greater or equal (signed)
  - `lo`/`ls`/`hi`/`hs` = lower/lower or same/higher/higher or same (unsigned)
  - A few more weird flags, virtually unused.
  - Example use:
        ```asm
        cmp x0, 3
        b.ls 0xffffff8012345678
        ```
- Shortcuts `cbz`/`cbnz` = compare-branch-zero and compare-branch-non-zero.  
  Just shorter ways to write `cmp xN, 0`+`b.eq` or `b.ne`.  
  (Translates nicely to C `if(x)` or `if(!x)`.)
- A bunch of specialized instructions exist to conditionally operate on registers (e.g. `ccmp`, `cset`).
- `b` = unconditional branch (e.g. `b 0xffffff8012345678`)  
  Jump to PC-relative address. Used primarily within function for flow control.
- `bl` = branch-and-link (e.g. `bl 0xffffff8012345678`)  
  Store return address to `x30` and jump to PC-relative address. Used for static function calls.
- `blr` = branch-and-link to register (e.g. `blr x8`)  
  Store return address to `x30` and jump to address in `x8`. Used for calls with function pointers or C++ virtual methods.
- `br` = branch to register (e.g. `br x8`)  
  Jump to address in `x8`. Used for tail calls.
- `ret` = return  
  Can in theory use registers other than x30 (e.g. `ret x8`), but compiler doesn't usually generate that.
