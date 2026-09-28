# compile_tool

Turn C programs into memory images for a bare-metal CPU. There is no OS, no
libc and no syscalls.

You give it C files. It gives you:

- one instruction image per memory bank (`imem_bank<N>.hex` and `.bin`)
- one data image (`dmem.hex` and `.bin`)
- the ELF, a disassembly, the linker script and a small I/O header

It is the program builder for the
[Carolyne](https://github.com/Tanawin1701d/Carolyne) out-of-order CPU
generator, and it also works on its own.

## Targets

| target   | compiler prefix        | notes                                          |
| -------- | ---------------------- | ---------------------------------------------- |
| `rv32im` | `riscv64-unknown-elf-` | RISC-V 32-bit with multiply and divide         |
| `mips32` | `mipsel-linux-gnu-`    | MIPS32r2, little-endian, every delay slot is a `nop` |

To use another compiler, set `CAROLYNE_RISCV_PREFIX` or `CAROLYNE_MIPS_PREFIX`.

## Quick start

```bash
sudo apt install gcc-riscv64-unknown-elf gcc-mipsel-linux-gnu
git clone https://github.com/Tanawin1701d/compile_tool.git
python3 -m compile_tool build compile_tool/programs/hello.c --name hello
```

Run it from the folder that contains `compile_tool/`. The images go to
`generated/programs/hello/`.

New here? Read the [tutorial](TUTORIAL.md).

## Commands

```bash
python3 -m compile_tool build a.c b.c --target rv32im --imem 16K --dmem 8K
python3 -m compile_tool layout --target mips32
python3 -m compile_tool verify generated/programs/hello/hello.elf
```

## Verification (optional)

If [Carolyne](https://github.com/Tanawin1701d/Carolyne) is installed, the tool
decodes every instruction with Carolyne's own ISA description. If the CPU
cannot run a word, the build fails and names its address.

Without Carolyne the build still works and prints `not verified`.

## Requirements

- Python 3.10 or newer (standard library only)
- a cross GCC for your target

## License

Apache License 2.0 with an added no-military-use condition. See
[LICENSE](LICENSE).

The programs in `programs/` that come from RIDECORE keep their own license:
[programs/LICENSE.ridecore](programs/LICENSE.ridecore).
