# AOR2 KVM Virtualization Project

A minimal **Linux/KVM-based virtualization project** written in C, implementing a virtual machine capable of executing guest code in **64-bit x86 long mode**. The project uses the Linux KVM API to create and configure virtual machines, allocate guest memory, initialize page tables and CPU registers, and handle guest I/O and VM exits.

## Features

- Linux KVM API integration
- 64-bit x86 long mode execution
- Support for 4 KB and 2 MB guest pages
- Configurable guest memory: 2 MB, 4 MB, and 8 MB
- Multiple guest execution using POSIX threads
- Minimal freestanding guest binary loaded directly into guest memory
- Guest output through the `0xE9` debug I/O port
- Guest `HLT` instruction and VM shutdown handling
- MIT license

## Project Structure

```text
.
├── kvm_zadatak3.c    # KVM host implementation
├── guest.c           # Minimal freestanding guest program
├── guest.ld          # Linker script for the guest binary
├── Makefile          # Build rules for the host and guest
└── LICENSE           # MIT license
```

## Requirements

- Linux operating system with KVM support
- A KVM-compatible CPU with hardware virtualization enabled
- Accessible `/dev/kvm` device
- GCC
- GNU `ld`
- GNU Make
- POSIX threading support

Verify that KVM is available:

```bash
ls -l /dev/kvm
```

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/petardukic112/aor2-kvm.git
cd aor2-kvm
```

### Build the Project

Build the KVM host program and guest image:

```bash
make
```

This generates:

- `kvm_zadatak3` – host-side KVM virtual machine runner
- `guest.o` – compiled guest object file
- `guest.img` – guest binary image

To remove generated files:

```bash
make clean
```

## Running

The host program accepts memory size, page size, and guest image arguments.

Example:

```bash
./kvm_zadatak3 \
  --memory 4 \
  --page 4 \
  --guest guest.img
```

### Command-Line Arguments

| Argument | Description | Supported values |
|---|---|---|
| `--memory`, `-m` | Guest memory size in MB | `2`, `4`, `8` |
| `--page`, `-p` | Guest page size in MB or KB | `2` (2 MB), `4` (4 KB) |
| `--guest`, `-g` | Guest binary image | Path to guest image |

### Example Output

```text
Hello, world!
KVM_EXIT_HLT
```

## How It Works

### Host Virtual Machine

The host program performs the following steps:

1. Opens `/dev/kvm` and initializes the KVM interface.
2. Creates a virtual machine and a virtual CPU (VCPU).
3. Allocates guest physical memory using `mmap`.
4. Registers the guest memory region with KVM.
5. Initializes the guest page tables (PML4, PDPT, and PD).
6. Configures CPU registers and control registers to enable PAE, protected mode, paging, and 64-bit long mode.
7. Loads `guest.img` into guest memory at address `0x8000`.
8. Starts guest execution using `KVM_RUN`.
9. Handles VM exits, including I/O operations and guest halt instructions.

### Guest Program

The guest is a minimal freestanding C program compiled and linked independently of the host application.

It is loaded at guest physical address `0x8000` and executes in 64-bit long mode. The program writes `Hello, world!\n` to the debug I/O port `0xE9`, then executes the `hlt` instruction.

The host handles the resulting VM exit and terminates guest execution.

## Technical Details

- **Virtualization:** Linux Kernel-based Virtual Machine (KVM)
- **Architecture:** x86-64
- **Execution mode:** 64-bit long mode
- **Memory management:** Guest physical memory backed by `mmap`
- **Paging:** 4 KB and 2 MB pages
- **Concurrency:** POSIX threads for multiple guest execution
- **Guest I/O:** Port-mapped I/O using port `0xE9`

## License

This project is licensed under the [MIT License](LICENSE).
