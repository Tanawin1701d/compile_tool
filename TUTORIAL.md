# Tutorial

This tutorial builds a small C program into memory images, step by step. It
takes about five minutes.

## 1. Install

You need Python 3.10 or newer and a cross compiler.

```bash
sudo apt install gcc-riscv64-unknown-elf     # for rv32im
sudo apt install gcc-mipsel-linux-gnu        # for mips32
git clone https://github.com/Tanawin1701d/compile_tool.git
```

Nothing else to install. The tool uses only the Python standard library.

Run every command below from the folder that CONTAINS `compile_tool/`.

## 2. Look at the memory map

```bash
python3 -m compile_tool layout --target rv32im
```

```
memory layout
  code  0x00000000 .. 0x00002000    8192 B  2 bank(s)
  data  0x10000000 .. 0x10000ff0    4080 B  data, bss, stack
  io    0x10000ff0 .. 0x10001000      16 B  putchar putint exit
```

There are two separate memories:

- **code** holds the instructions. It starts at the ISA's reset address.
- **data** holds globals, strings and the stack.
- **io** is the top 16 bytes of data. A store there prints a character, prints
  a number, or stops the program.

## 3. Write a program

Save this as `sum.c`:

```c
#include "carolyne_io.h"

int main(void)
{
    int total = 0;
    for (int i = 1; i <= 10; i++)
        total += i;

    print_str("sum = ");
    print_int(total);
    print_char('\n');
    return 0;
}
```

You do not write `carolyne_io.h` yourself. The tool generates it from the
memory map. It gives you four calls:

| call              | what it does                   |
| ----------------- | ------------------------------ |
| `print_char(c)`   | print one character            |
| `print_int(n)`    | print a signed number          |
| `print_str(s)`    | print a string                 |
| `finish(code)`    | stop the machine               |

Returning from `main` also stops the machine.

## 4. Build it

```bash
python3 -m compile_tool build sum.c --name sum
```

```
sections
  .text        imem  0x00000000    208 B
  .rodata      dmem  0x10000000      8 B

images
  imem_bank0      26 / 1024  words used
  imem_bank1      26 / 1024  words used
  dmem             2 / 1024  words used

images written to generated/programs/sum
```

Look in `generated/programs/sum/`:

| file                 | what it is                                    |
| -------------------- | --------------------------------------------- |
| `imem_bank0.hex`     | instruction bank 0, one 32-bit word per line  |
| `imem_bank1.hex`     | instruction bank 1                            |
| `dmem.hex`           | the data memory                               |
| `*.bin`              | the same images in binary                     |
| `sum.elf`            | the linked program                            |
| `sum.dump`           | the disassembly                               |
| `link.ld`            | the generated linker script                   |

Load the `.hex` files into your simulator with `$readmemh`.

## 5. Change the machine

Match the images to your hardware with a few options:

```bash
python3 -m compile_tool build sum.c --name sum --imem 16K --dmem 8K --lanes 4
```

- `--imem` and `--dmem` set the memory sizes. Each must be a power of two.
- `--lanes` sets the number of instruction banks. Word `w` goes to bank
  `w % lanes`.
- `--opt` sets the optimization level, for example `--opt=-O0`.

## 6. Try MIPS

```bash
python3 -m compile_tool build sum.c --name sum_mips --target mips32
```

The same C code runs on both targets. For MIPS, the program starts at
`0xBFC00000`, and every branch delay slot holds a `nop`.

## 7. Verify against the CPU (optional)

Install [Carolyne](https://github.com/Tanawin1701d/Carolyne). The build then
checks every instruction against the CPU's ISA description:

```
verified 63 instructions against rv32im: ok
```

If a word is not supported, the build fails and prints its address:

```
0x000000ac  02e787b3  [.text]  matches no µop in the ISA
```

## 8. Use it from Python

```python
from compile_tool import build_program, target_named

mem     = target_named("rv32im").machine_mem(imem_bytes=8192, dmem_bytes=4096, banks=2)
program = build_program(["sum.c"], mem, name="sum")
print(program.describe())
```

## More programs

`programs/` has 17 test programs: recursion, sorting, a sieve and more. Build
any of them the same way:

```bash
python3 -m compile_tool build compile_tool/programs/hanoi.c --name hanoi
```

`cprime.c` needs `--dmem 16K`.
