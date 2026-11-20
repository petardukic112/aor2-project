I’ll inspect the repository structure and key files, then add a README with setup and usage details.# AOR2 KVM Virtualization Project

A C-based Linux/KVM project that creates and runs virtual machines in 64-bit long mode. It builds a minimal guest program and uses `/dev/kvm` to configure VCPUs, memory, page tables, registers, and I/O exits.

## Features

- Linux KVM API integration
- 64-bit x86 long mode
- 4 KB and 2 MB guest page support
- 2 MB, 4 MB, and 8 MB guest memory options
- Multiple guest execution using POSIX threads
- Minimal guest binary loaded directly into guest memory
- Guest output through the `0xE9` debug I/O port
- Basic guest HLT and VM shutdown handling
- MIT licensed

## Project Structure

| File | Description |
|---|---|
| `kvm_zadatak3.c` | KVM host implementation |
| `guest.c` | Minimal freestanding guest program |
| `guest.ld` | Linker script for creating the guest binary |
| `Makefile` | Build rules for the host and guest |
| `LICENSE` | MIT license |

## Requirements

- Linux with KVM support
- A KVM-compatible CPU
- `/dev/kvm` available and accessible
- GCC
- GNU `ld`
- Make
- POSIX threading support

Check whether virtualization is enabled:

```bash
ls /dev/kvm
```

## Building

Build the KVM runner and guest image:

```bash
make
```

This creates:

- `kvm_zadatak3`
- `guest.o`
- `guest.img`

Remove generated files:

```bash
make clean
```

## Running

The host program currently accepts memory, page-size, and guest-image arguments:

```bash
./kvm_zadatak3 \
  --memory 4 \
  --page 4 \
  --guest guest.img
```

Supported argument values:

- `--memory` / `-m`
  - `2` → 2 MB memory
  - `4` → 4 MB memory
  - `8` → 8 MB memory
- `--page` / `-p`
  - `2` → 2 MB pages
  - `4` → 4 KB pages
- `--guest` / `-g` → guest binary image

Example guest output:

```text
Hello, world!
KVM_EXIT_HLT
```

## How It Works

### Host VM

The host program:

1. Opens `/dev/kvm`.
2. Creates a VM and virtual CPU.
3. Allocates guest memory with `mmap`.
4. Configures the guest memory region.
5. Initializes the PML4, PDPT, PD, and page tables.
6. Enables PAE, protected mode, paging, and 64-bit long mode.
7. Loads `guest.img` at guest address `0x8000`.
8. Starts the CPU with `KVM_RUN`.
9. Handles guest I/O and halt exits.

### Guest Program

The guest is a freestanding C program placed at the `.start` section. It writes `Hello, world!\n` to I/O port `0xE9`, then executes `hlt` repeatedly.

This project was created as an operating systems/virtualization exercise focused on low-level Linux KVM usage and x86-64 paging.
