# Barrelfish OS Capstone

A Spring 2023 CSE 481A operating-systems capstone built on the [Barrelfish](https://barrelfish.org/) research operating system. Our four-person team extended the ARMv8 system through the course milestones, from memory management and process creation to multicore messaging and an interactive shell.

> **Project team:** Andrew Makin · Daniel Rashevsky · David Strupinski · Peter Hale

## What we built

The project brings together several core operating-system mechanisms:

- **Memory management:** capability-backed physical memory allocation, block splitting and coalescing, partial frees, and virtual-memory mapping and unmapping.
- **Process creation:** ELF loading, child address-space and capability-space setup, command-line arguments, and capability transfer.
- **Inter-process communication:** LMP-based RPC for process management, memory requests, and terminal input/output.
- **Multicore support:** booting additional ARM cores and using shared URPC/UMP buffers to exchange messages, acknowledgements, and larger fragmented payloads.
- **Interactive shell:** foreground and background process runs, process listing, timing and memory checks, and RAMFS file and directory commands.

The system was developed and exercised with QEMU and an i.MX8X/Toradex ARMv8 board. The final report describes the design decisions and limitations in detail; the original PDF is not included in this repository history.

## Daniel Rashevsky’s contributions

Daniel’s work focused on connecting the system’s process, capability, and multicore mechanisms:

- Implemented early child-process spawning, including child CSpace and VSpace setup, ELF section allocation, and capability setup and copying.
- Helped establish the LMP channel and RPC flow between `init` and child processes, including endpoint transfer and receive-handler setup.
- Integrated the memory-server path and fixed its handshake.
- Implemented parts of multicore boot setup, including passing memory, module strings, and ELF module capabilities to newly booted cores; fixed related boot-info and alignment issues.
- Added shell help and ELF-module listing commands, and contributed to the shell’s command documentation.

These changes were developed collaboratively and depended on the interfaces and debugging work of the whole team.

## Team credits

The project was a shared effort; these highlights reflect the milestone responsibilities recorded by the team, not exclusive ownership of each subsystem.

| Team member | Project contributions |
| --- | --- |
| **Andrew Makin** | Led much of the memory-management and virtual-memory work; contributed substantially to process spawning and RPC/message-passing implementation, debugging, and integration. |
| **Daniel Rashevsky** | Child-process and capability setup, ELF loading support, LMP/RPC integration, multicore memory and module transfer, and shell help/module-listing features. |
| **David Strupinski** | Virtual-memory and paging work, process-management and IPC debugging, multicore messaging design and fixes, and early shell/terminal integration. |
| **Peter Hale** | Memory allocator coalescing and partial-free work, process and RPC implementation, multicore boot and UMP messaging, UART support, and RAMFS-backed shell commands. |

## How the system fits together

The `init` domain coordinates process management and most of the project’s LMP and UMP services. Child processes receive their own address spaces and capability spaces; `spawn` loads their ELF images and starts their dispatchers. Physical memory allocation and virtual mapping are separate layers, with the RPC and shell interfaces built on top. For multicore operation, the boot core prepares memory and boot information for application cores, while shared URPC frames carry inter-core messages.

This design favored a working course project and simpler integration. As the report notes, keeping many services in `init` reduces modularity and can serialize service work. The shell and virtual-space behavior were also unstable in the final report, and terminal I/O for processes on application cores was incomplete.

## Shell commands

The shell supports:

| Area | Commands |
| --- | --- |
| Processes | `run [cmdline] [&]`, `oncore [coreid] [cmdline] [&]`, `ps` |
| Files and directories | `pwd`, `ls`, `touch`, `rm`, `cat`, `mkdir`, `rmdir`, `cd` |
| Utilities | `echo`, `run_memtest [size]`, `lsmod`, `time [cmd]`, `help` |

The filesystem is an in-memory RAMFS initialized with a sample `test.txt`. Paths are relative for file commands; `cd` takes an absolute path. `oncore` is present, but application-core terminal I/O is not supported in the final report. There is no shell `kill` command.

## Build and run

This is a legacy Barrelfish tree with an older Hake/Haskell and cross-compilation toolchain. The checked-in Nix files describe the intended build dependencies, but current Nix channels may not reproduce that historical environment without adjustment. See [`doc/018-Practical-guide/readme.tex`](doc/018-Practical-guide/readme.tex), [`shell.nix`](shell.nix), and [`default.nix`](default.nix) before setting up a build.

The practical guide’s example for an x86-64 build is:

```sh
mkdir build
cd build
../hake/hake.sh -s ../ -a x86_64
make
make sim
```

The capstone work also targets ARMv8 configurations, including QEMU and i.MX8X; use the corresponding Hake configuration and boot image for the target you intend to run. The repository includes course autograder specifications under [`tools/autograder/tests`](tools/autograder/tests).

## Repository map

- `lib/aos/` — AOS paging, memory allocation, and RPC support
- `lib/spawn/` — ELF loading and child-process startup
- `usr/init/` — process management, core boot, and system coordination
- `usr/shell/` — interactive shell
- `lib/fs/` and `include/fs/` — filesystem interfaces and RAMFS
- `hake/`, `platforms/`, and `tools/` — build, platform, boot, and test support
- `report/` — LaTeX sources for the milestone report

This project retains Barrelfish and third-party components with their own notices. See [`LICENSE`](LICENSE) and the notices in the relevant directories.
