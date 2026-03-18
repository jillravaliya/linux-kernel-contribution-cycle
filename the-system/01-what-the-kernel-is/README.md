# What the Linux Kernel Is

---

> Ever used a computer and wondered what actually happens when you press a key, open a file, or connect to the internet? Something has to talk to the hardware. Something has to decide which program runs next. Something has to make sure one program cannot read another program's memory. That something is the kernel.

**You're about to find out what it actually is — not the textbook definition, the real thing.**

---

## What's Inside This File

```
01-what-the-kernel-is/
│
├── Kernel vs Operating System — the real difference
├── What the Kernel Actually Does at Runtime
│   ├── Process scheduling
│   ├── Memory management
│   ├── Hardware abstraction
│   └── System calls — the bridge to userspace
├── The Numbers — scale of this project
├── Release Cadence — how often, who decides
└── Why This Project Is Unlike Any Other
```

---

## Kernel vs Operating System — The Real Difference

Most people use "Linux" and "operating system" interchangeably. That is imprecise in a way that matters when you are trying to understand how things work.

```
What you think of as "Linux":

  Ubuntu / Fedora / Debian / Android
  └── This is a DISTRIBUTION
      └── It bundles together:
          ├── The Linux kernel
          ├── GNU tools (bash, ls, cp, gcc...)
          ├── Package manager (apt, dnf, pacman)
          ├── Desktop environment (GNOME, KDE...)
          ├── System services (systemd, networking)
          └── Thousands of applications

  The kernel is ONE component of all of this.
  It is the lowest layer.
  Everything else sits on top of it.
```

The **kernel** is the piece of software that runs directly on the hardware. It is the first thing that loads when a machine boots (after the bootloader), and it stays running until the machine shuts down. Every other program on the system — your browser, your shell, your database — runs on top of the kernel. None of them touch hardware directly. They all go through the kernel.

Think of it this way:

```
Hardware (CPU, RAM, disk, network card, USB...)
        |
        |  Only the kernel talks directly to this
        v
+-----------------------------------+
|         LINUX KERNEL              |
|                                   |
|  Scheduler  Memory  Drivers       |
|  Filesystems  Network  Security   |
+-----------------------------------+
        |
        |  Everything above talks to the kernel
        v
Userspace (your programs, shell, browser, database)
```

When your browser downloads a file:
- Browser calls the kernel: "write this data to disk"
- Kernel talks to the disk controller
- Disk writes the data
- Kernel tells the browser: "done"

The browser never touches the disk directly. It cannot. Only the kernel can.

---

## What the Kernel Actually Does at Runtime

The kernel is not sitting idle. Every millisecond of a running system, it is doing several things simultaneously.

---

### Process Scheduling

Your laptop may have 4 or 8 CPU cores. But it is running hundreds of processes simultaneously — your browser, your music player, system services, background daemons. More processes than cores.

The kernel's scheduler decides which process gets CPU time, for how long, and in what order.

```
4 CPU cores, 200 running processes:

Core 1: [Firefox] → [systemd] → [sshd] → [Firefox] → ...
Core 2: [Spotify] → [bash]    → [vim]  → [Spotify] → ...
Core 3: [kernel]  → [kworker] → [gcc]  → [kernel]  → ...
Core 4: [Python]  → [curl]    → [grep] → [Python]  → ...

Each slice is ~1-4 milliseconds.
Switches happen so fast it feels simultaneous.
The scheduler makes ~1000 decisions per second per core.
```

The Linux scheduler (called CFS — Completely Fair Scheduler) ensures no process starves, real-time processes get priority, and interactive tasks (like your GUI) feel responsive even under heavy load.

---

### Memory Management

Every process thinks it has the entire memory to itself. This is an illusion the kernel creates called **virtual memory**.

```
Reality:                    What each process sees:
+--------------+            +--------------+
|  16 GB RAM   |            |  Full 64-bit |
|              |            |  address     |
|  Firefox     |            |  space       |
|  Spotify     |   kernel   |  (process    |
|  Terminal    |  ------->  |  thinks it   |
|  Python      |  manages   |  owns all    |
|  kernel      |            |  of memory)  |
|  ...         |            |              |
+--------------+            +--------------+
```

The kernel manages:
- Which physical RAM page belongs to which process
- Swapping pages to disk when RAM is full
- Making sure process A cannot read process B's memory
- Mapping files into memory (mmap)
- Handling page faults when a process accesses memory not yet loaded

If a process tries to access memory it does not own, the kernel kills it immediately. That is a segmentation fault. The kernel is enforcing the boundary.

---

### Hardware Abstraction

Your machine might have an Intel NIC or a Realtek NIC or a Broadcom NIC. Your application that sends data over the network does not know or care which one. It just calls `send()`.

The kernel provides a uniform interface. Underneath, a **device driver** handles the hardware-specific details.

```
Application says: "send this data over the network"
        │
        ▼
Kernel network stack (same for every machine)
        │
        ▼
Device driver (hardware-specific)
        │
   ┌────┴────┐
   │         │
Intel NIC  Realtek NIC  ← different hardware,
                          same interface above
```

The kernel ships with drivers for thousands of pieces of hardware. When you plug in a USB device and it "just works," that is the kernel loading the right driver and presenting the device through a standard interface.

---

### System Calls — The Bridge to Userspace

Programs running in userspace cannot directly execute privileged operations — they cannot write to hardware, cannot allocate physical memory, cannot create new processes. They have to ask the kernel.

This asking happens through **system calls** (syscalls).

```
Your program (userspace):
  fd = open("/etc/passwd", O_RDONLY);

What actually happens:
  1. Program puts syscall number in CPU register
     (open = syscall 2 on x86-64)
  2. Program executes SYSCALL instruction
  3. CPU switches from Ring 3 (user) to Ring 0 (kernel)
  4. Kernel handles the request
  5. Kernel returns result
  6. CPU switches back to Ring 3
  7. Your program continues

Total time: microseconds.
This happens millions of times per second on a busy system.
```

There are around 300-400 system calls on Linux. Everything your program does that involves the outside world — files, network, processes, time, memory — goes through a syscall. The kernel is the gatekeeper for all of it.

---

## The Numbers — Scale of This Project

Linux is the largest collaborative software project in human history. These numbers are not marketing — they are the actual scale.

```
Linux kernel by the numbers (as of 2024):

Lines of code:
└── ~36 million lines
    └── For comparison:
        ├── Windows NT (1993): ~4 million lines
        ├── Space Shuttle software: ~400,000 lines
        └── Apollo 11 guidance computer: ~145,000 lines

Contributors per release cycle (~10 weeks):
└── ~4,000 individual contributors
    └── From ~500 different companies and organisations

Patches merged per day:
└── ~70 patches per day on average
    └── That is roughly one every 20 minutes, 24/7

Git history:
└── Over 1,000,000 commits since 2005
    (when git was adopted — git was written by Linus
    specifically for kernel development)

Where it runs:
├── ~97% of the world's top 1 million web servers
├── All 500 of the world's top 500 supercomputers
├── ~74% of smartphones (Android runs on Linux kernel)
├── Most cloud infrastructure (AWS, GCP, Azure VMs)
└── Embedded systems — routers, TVs, cars, satellites
```

This is maintained by a volunteer-driven community, coordinated entirely through email, with no central office, no mandatory working hours, and no single company in control.

---

## Release Cadence — How Often, Who Decides

The Linux kernel follows a predictable release cycle.

```
Linux release cycle (~10 weeks total):

Week 1-2:  MERGE WINDOW
           └── New features accepted into mainline
               Only during this 2-week window

Week 3-9:  STABILISATION (rc1 through rc7/rc8)
           └── Bug fixes only
           └── rc = release candidate
           └── One rc per week
           └── Each rc is more stable than the last

Week 10:   FINAL RELEASE
           └── New version number (e.g. 6.8)
           └── Merge window for next version opens
               immediately after
```

Who decides when to release? **Linus Torvalds**. He personally decides when the kernel is stable enough to release. He looks at the number of patches still coming in, the severity of known bugs, and his own judgment. He announces releases on the kernel mailing list.

```
Typical Linus release announcement:

  "Ok, so I really do not want to add another -rc
   and things have been calm, so here we go with
   the 6.8 release.

   ...the shortlog below is dominated by fixes
   all over..."

  -- Linus Torvalds
```

No product manager. No release committee. One person, reading the mailing list, making a judgment call every 10 weeks.

Version numbering is simple:

```
Linux 6.8.3

6   = major version (increments rarely, mainly symbolic)
8   = release number (increments every ~10 weeks)
3   = stable patch number (bug fixes after release)
```

---

## Why This Project Is Unlike Any Other

Every aspect of how this project runs contradicts conventional software engineering wisdom.

```
Conventional wisdom          Linux kernel reality
--------------------------------------------------
Use a bug tracker            Email only
Use pull requests            Patches on mailing list
Have a roadmap               No formal roadmap
Have a product manager       No product manager
Use CI for everything        Humans review first
Enforce code style with lint  checkpatch.pl, optional
Have a release schedule      Linus decides
Hire a security team         Community reports CVEs

And yet:
├── 36 million lines
├── Runs on everything
├── 30+ years of continuous development
└── Powers 97% of the internet
```

The reason it works is not despite these unusual choices — it is partly because of them. Email is asynchronous, archivable, searchable, and works for contributors in every timezone. Patches reviewed by humans before CI catches the architectural mistakes that automated tools miss. No roadmap means contributors work on what they actually care about, which means they work harder and longer than any employed team would.

This does not mean it is easy to contribute. It is one of the hardest codebases to get a patch accepted into. The standards are high, the review is blunt, and the process is unforgiving of sloppiness.

That is what the rest of this repo covers.

---

> **Now you know what the kernel is and what it does. Next: how is the 36-million-line codebase actually organised — and how do you find your way around it?**
> **→ [02-how-the-kernel-is-organised](../02-how-the-kernel-is-organised/README.md)**
