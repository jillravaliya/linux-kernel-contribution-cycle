# How the Kernel Is Organised

---

> You just learned what the kernel does. Now the real question: how is 36 million lines of code actually structured? How do you open the source tree and not feel completely lost? Where does the networking code live? Where do drivers live? What is a subsystem and why does it matter for contributing?

**You're about to find out — and after this, you will open the kernel source tree and know exactly where to look.**

---

## What's Inside This File

```
02-how-the-kernel-is-organised/
│
├── The Top-Level Source Tree — what every folder is
├── What a Subsystem Is
├── The Major Subsystems — deep look at each
├── How Subsystems Relate to Each Other
└── The MAINTAINERS File — how to read every field
```

---

## The Top-Level Source Tree

Clone the kernel and this is what you see:

```
linux/
│
├── arch/        CPU architecture-specific code
├── block/       Block device layer (storage I/O)
├── certs/       Kernel signing certificates
├── crypto/      Cryptographic algorithms
├── Documentation/ Kernel documentation
├── drivers/     Device drivers (largest directory)
├── fs/          Filesystem implementations
├── include/     Header files used across the kernel
├── init/        Kernel startup and init code
├── ipc/         Inter-process communication
├── kernel/      Core kernel (scheduler, locking...)
├── lib/         Common library functions
├── mm/          Memory management
├── net/         Networking stack
├── samples/     Example code
├── scripts/     Build and utility scripts
├── security/    Security modules (SELinux, AppArmor)
├── sound/       Audio subsystem (ALSA)
├── tools/       Userspace tools (perf, bpftool...)
├── usr/         Early userspace (initramfs)
└── virt/        Virtualisation support (KVM)
```

The three largest directories by line count:

```
drivers/     ~24 million lines  (~67% of the kernel)
arch/        ~4 million lines   (~11%)
net/         ~1.5 million lines (~4%)
everything else combined: ~18%
```

Most of the kernel is drivers. If you want to contribute and do not know where to start, `drivers/` is where the most work happens and where most contributors begin.

---

## What a Subsystem Is

A subsystem is a self-contained area of the kernel with its own maintainer, its own mailing list, its own git tree, and its own rules for accepting patches.

```
Think of the kernel like a city:

+------------------------------------------+
|              LINUX KERNEL                |
|                                          |
|  +----------+  +----------+  +--------+ |
|  | Memory   |  | Network  |  | Files  | |
|  | Mgmt     |  | Stack    |  | ystem  | |
|  | (mm/)    |  | (net/)   |  | (fs/)  | |
|  +----------+  +----------+  +--------+ |
|                                          |
|  +----------+  +----------+  +--------+ |
|  | Drivers  |  | Security |  | Arch   | |
|  | (drivers)|  | (security|  | (arch/)| |
|  +----------+  +----------+  +--------+ |
|                                          |
+------------------------------------------+

Each box = one subsystem
Each has its own maintainer
Each has its own mailing list
Each has its own git tree
Patches go to the subsystem first
```

A patch for a USB driver goes to the USB subsystem maintainer. A patch for the TCP stack goes to the networking maintainer. A patch for memory allocation goes to the mm maintainer. They do not all go to Linus. Linus only pulls from subsystem maintainers.

---

## The Major Subsystems — Deep Look at Each

---

### arch/ — CPU Architectures

```
arch/
├── x86/      Intel and AMD (desktops, servers, laptops)
├── arm/      32-bit ARM (older phones, embedded)
├── arm64/    64-bit ARM (modern phones, Apple Silicon)
├── riscv/    RISC-V (open ISA, growing fast)
├── mips/     MIPS (routers, older embedded)
├── powerpc/  IBM PowerPC (servers, older Macs)
├── s390/     IBM mainframes
└── ...       20+ architectures total
```

Architecture code handles everything that is CPU-specific — how to boot, how to handle interrupts, how to do context switches, how page tables are structured. If you are not writing CPU-specific code, you almost never touch `arch/`.

---

### drivers/ — Device Drivers

The largest directory in the kernel. Almost everything hardware-related lives here.

```
drivers/
├── net/         Network interface card drivers
│   ├── ethernet/  Wired NIC drivers (Intel, Realtek...)
│   └── wireless/  WiFi drivers (iwlwifi, ath9k...)
├── usb/         USB host controllers and device drivers
├── gpu/         Graphics drivers
│   ├── i915/    Intel GPU driver
│   ├── amdgpu/  AMD GPU driver
│   └── nouveau/ NVIDIA open source driver
├── block/       Storage drivers (NVMe, SCSI, SATA)
├── input/       Keyboard, mouse, touchscreen
├── i2c/         I2C bus drivers
├── spi/         SPI bus drivers
├── sound/       Audio device drivers
├── bluetooth/   Bluetooth drivers
├── staging/     Drivers not yet meeting quality bar
└── ...          100+ subdirectories
```

`drivers/staging/` deserves special attention. It contains drivers that are functional but not yet clean enough for the main tree. Coding style issues, TODO comments, known problems. This is where most new contributors start — the bar is lower, the feedback is gentler, and the path to mainline is clear.

---

### fs/ — Filesystems

```
fs/
├── ext4/      The standard Linux filesystem
├── btrfs/     Modern copy-on-write filesystem
├── xfs/       High-performance filesystem (SGI origin)
├── tmpfs/     RAM-based temporary filesystem
├── proc/      /proc virtual filesystem (process info)
├── sysfs/     /sys virtual filesystem (device info)
├── nfs/       Network File System (remote mounts)
├── fat/       FAT filesystem (USB drives, SD cards)
├── ntfs3/     NTFS driver (Windows drives)
└── ...        50+ filesystem implementations
```

Every `mount` you do uses code from here. `/proc/cpuinfo`, `/sys/class/net` — all virtual filesystems implemented in `fs/`.

---

### mm/ — Memory Management

```
mm/
├── page_alloc.c    Physical page allocator (buddy system)
├── slab.c          Slab allocator (kmalloc internals)
├── vmalloc.c       Virtual memory allocator
├── mmap.c          Memory mapping (mmap syscall)
├── swap.c          Swap space management
├── oom_kill.c      Out-of-memory killer
├── page_fault.c    Page fault handler
└── ...
```

When your system runs low on RAM and starts swapping, or when the OOM killer terminates a process to free memory — that is all `mm/`. This is one of the most complex subsystems. Patches here get very close scrutiny.

---

### net/ — Networking Stack

```
net/
├── core/       Core networking (socket layer, skbuff)
├── ipv4/       IPv4 protocol implementation
├── ipv6/       IPv6 protocol implementation
├── tcp/        TCP implementation
├── udp/        UDP implementation
├── netfilter/  Packet filtering (iptables/nftables)
├── wireless/   WiFi stack (mac80211)
├── bluetooth/  Bluetooth protocol stack
└── ...
```

When your application calls `connect()` or `send()`, the data travels through `net/core/` → `net/ipv4/` → `net/tcp/` before reaching the driver in `drivers/net/`. The networking stack is separate from the networking drivers.

---

### kernel/ — Core Kernel

```
kernel/
├── sched/      Process scheduler (CFS, RT, deadline)
├── locking/    Mutex, spinlock, rwlock implementations
├── time/       Timers, timekeeping, POSIX clocks
├── irq/        Interrupt handling framework
├── printk.c    The kernel's logging system (dmesg)
├── fork.c      Process creation (fork, clone)
├── exit.c      Process termination
├── signal.c    Signal handling
└── ...
```

This is the heart of the kernel. The scheduler in `kernel/sched/` decides what runs. `kernel/locking/` prevents race conditions across the entire kernel. These files are touched by almost every other subsystem.

---

### security/ — Security Modules

```
security/
├── selinux/    SELinux (mandatory access control)
├── apparmor/   AppArmor (profile-based MAC)
├── smack/      Simplified Mandatory Access Control
├── tomoyo/     TOMOYO Linux
├── landlock/   Landlock (sandboxing for unprivileged)
└── keys/       Kernel key management
```

The Linux Security Module (LSM) framework hooks into kernel operations — file open, process exec, network connect — and lets security modules make allow/deny decisions. SELinux uses this. AppArmor uses this.

---

### crypto/ — Cryptographic Algorithms

```
crypto/
├── aes_generic.c    AES block cipher
├── sha256_generic.c SHA-256 hash
├── rsa.c            RSA asymmetric crypto
├── hmac.c           HMAC construction
├── gcm.c            GCM authenticated encryption
└── ...
```

Used internally by IPsec, dm-crypt, TLS, and anything that needs crypto at kernel level. As covered in the LFD137 notes — this subsystem has export control implications for contributors.

---

## How Subsystems Relate to Each Other

Subsystems are not fully independent. They call into each other constantly.

```
Example: you open a file over NFS (network filesystem)

  Application calls open("/mnt/nfs/file.txt")
          |
          v
  VFS layer (fs/namei.c)
  "which filesystem handles this path?"
          |
          v
  NFS filesystem (fs/nfs/)
  "I need to fetch this over the network"
          |
          v
  Network stack (net/ipv4/, net/core/)
  "I need to send packets"
          |
          v
  Network driver (drivers/net/ethernet/)
  "I need to write to hardware"
          |
          v
  Physical network card
```

A single `open()` call touches four different subsystems. This is why kernel patches sometimes need sign-off from multiple maintainers — if your change touches both `fs/` and `mm/`, both maintainers need to agree.

---

## The MAINTAINERS File

The MAINTAINERS file lives at the root of the kernel tree. It is over 25,000 lines long and lists every subsystem, who maintains it, which mailing list to use, and which files belong to it.

Every entry looks like this:

```
NETWORKING [IPv4/IPv6]
M: Jakub Kicinski <kuba@kernel.org>
M: Paolo Abeni <pabeni@redhat.com>
L: netdev@vger.kernel.org
S: Maintained
W: http://www.linuxfoundation.org/en/Net
T: git git://git.kernel.org/pub/scm/linux/kernel/git/netdev/net.git
T: git git://git.kernel.org/pub/scm/linux/kernel/git/netdev/net-next.git
F: net/
F: include/linux/net*.h
```

What each field means:

```
Field meanings:
│
├── M:  Maintainer — name and email
│       Send patches here (and to the list)
│
├── L:  Mailing list — where discussion happens
│       Always CC this when sending a patch
│
├── S:  Status
│       ├── Maintained   — actively maintained
│       ├── Odd Fixes    — only critical fixes accepted
│       ├── Orphan       — no maintainer, patches welcome
│       └── Obsolete     — do not send patches
│
├── W:  Website — project webpage if any
│
├── T:  Git tree — where patches land before mainline
│       Two trees common:
│       ├── net.git      — fixes for current release
│       └── net-next.git — new features for next release
│
└── F:  Files — which files this entry covers
        Glob patterns, e.g. net/ means all of net/
```

You never need to read this file manually. The script `scripts/get_maintainer.pl` reads it for you and tells you exactly who to send a patch to based on which files you changed. That is covered in detail in the-process section.

But knowing the structure helps you understand what the script is doing — it is just pattern-matching your changed files against the `F:` fields in this file and returning the corresponding `M:` and `L:` values.

---

## Finding Your Way Around

When you are new to the kernel source, the size is overwhelming. Here is a practical approach:

```
Finding code in the kernel:

1. You know the subsystem:
   └── Go directly to the folder
       └── net/ for networking
           mm/ for memory
           drivers/usb/ for USB

2. You know a function name:
   └── git grep "function_name"
       └── Faster than grep, respects .gitignore

3. You know a syscall:
   └── Look in kernel/ or the arch/ syscall table
       └── SYSCALL_DEFINE for the implementation

4. You know a config option (e.g. CONFIG_USB):
   └── grep -r "config USB" --include=Kconfig
       └── Kconfig files define all config options

5. You have no idea:
   └── Start with drivers/staging/
       └── Pick any driver, read it top to bottom
       └── Run checkpatch.pl on it
       └── Fix what it complains about
       └── You now understand the codebase better
           than most people who have never contributed
```

The kernel source is big but it is not random. Once you understand that `drivers/` is hardware, `net/` is protocols, `fs/` is filesystems, `mm/` is memory, and `kernel/` is the core — you can navigate 67% of the codebase by intuition.

---

> **Now you know how the kernel is structured and where everything lives. Next: what actually happens when someone writes a patch? How does a change travel from a developer's laptop to the official release?**
> **→ [03-how-a-change-moves-through-the-system](../03-how-a-change-moves-through-the-system/README.md)**
