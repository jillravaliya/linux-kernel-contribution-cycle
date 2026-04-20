# The System — How the Linux Kernel Actually Works

---

> Before you send a single patch, you need to understand what you are contributing to. Not just "it is an OS kernel" — the actual architecture, the actual numbers, the actual flow of how 4,000 contributors working across every timezone coordinate without a central office, a project manager, or a sprint planning meeting.

**This section explains the machine. Read it before touching anything in the-process/.**

---

## Why This Section Exists

Most kernel contribution guides start with "here is how to send a patch." They skip the part that actually matters — understanding why the process is the way it is.

The kernel uses email because of how patches work. It has a merge window because of how Linus coordinates with 100+ maintainers. It has stable trees because of how distributions ship software. It has subsystems because 36 million lines cannot be managed by one person.

Every process rule in the-process/ has a reason rooted in how this system works. If you understand the system first, the rules stop feeling arbitrary.

---

## What Is in This Section

---

### [01 — What the Kernel Is](./01-what-the-kernel-is/README.md)

> The foundation. What the kernel actually does at runtime, how it differs from the operating system sitting above it, and why this project operates differently from every other software project on earth.

```
What you will understand after reading:

Kernel vs OS vs Distribution:
  Ubuntu / Fedora / Android
  └── Distribution (bundles everything)
      └── GNU tools + package manager + desktop
          └── Linux kernel  ← this is what we work on
              └── Hardware (CPU, RAM, disk, NIC)

What the kernel does every millisecond:
  ├── Scheduler — decides which of 200 processes
  │   gets the next 4ms of CPU time
  ├── Memory manager — maintains the illusion that
  │   every process owns all of RAM
  ├── Device drivers — translates "write to disk"
  │   into the 47 hardware-specific steps required
  └── System calls — the only legal gateway between
      your program and the hardware

The numbers:
  ├── ~36 million lines of code
  ├── ~4,000 contributors per release cycle
  ├── ~70 patches merged per day
  └── Runs on 97% of web servers, all 500 top
      supercomputers, and 3 billion phones
```

---

### [02 — How the Kernel Is Organised](./02-how-the-kernel-is-organised/README.md)

> 36 million lines. How is it not chaos? Where does networking live? Where do drivers live? What is a subsystem and why does it matter for your patch?

```
The top-level source tree:
  arch/        CPU architectures (x86, ARM, RISC-V)
  drivers/     ~24 million lines — 67% of the kernel
  fs/          50+ filesystem implementations
  mm/          Memory management
  net/         Networking stack
  kernel/      Core: scheduler, locking, timers
  security/    SELinux, AppArmor, hardening
  crypto/      Cryptographic algorithms

What a subsystem is:
  └── A self-contained area with its own:
      ├── Maintainer — one person ultimately responsible
      ├── Mailing list — where patches are discussed
      ├── Git tree — where patches land before mainline
      └── Rules — what gets accepted and what does not

Why it matters for you:
  └── Your patch goes to the subsystem, not to Linus
      The subsystem maintainer reviews it
      They send it to Linus during the merge window
      You never interact with Linus directly
```

---

### [03 — How a Change Moves Through the System](./03-how-a-change-moves-through-the-system/README.md)

> The complete lifecycle of a kernel patch — every stage from idea to the release that ships on millions of machines. What can go wrong at each stage. How long it actually takes.

```
The full journey — from your laptop to a release:

  YOU WRITE THE PATCH
       │
       v
  Send to mailing list
       │
       v
  Community review (days to weeks)
  ├── Automated bots find build failures
  ├── Other developers review the code
  └── Subsystem maintainer decides
       │
       v
  Accepted into subsystem tree
       │
       v
  linux-next integration testing (daily)
       │
       v
  MERGE WINDOW OPENS (2 weeks)
  Maintainer sends pull request to Linus
       │
       v
  In Linus's mainline tree
       │
       v
  rc1 through rc7 testing (6-8 weeks)
       │
       v
  FINAL RELEASE — ships to millions of machines

Realistic timelines:
  Simple cleanup:    2-6 weeks
  Bug fix:           3-8 weeks
  New driver:        2-6 months
  New feature:       3-6 months
```

---

### [04 — How Releases Work](./04-how-releases-work/README.md)

> The merge window, release candidates, stable trees, LTS kernels — the full release machinery with real dates from a real release cycle.

```
The 10-week cycle:

  Weeks 1-2:   MERGE WINDOW
               New features only — ~10,000 commits
               Maintainers send pull requests to Linus
               Linus pulls entire trees, not patches

  Weeks 3-10:  STABILISATION — bug fixes only
               rc1 → rc2 → rc3 → rc4 → rc5 → rc6 → rc7
               Each rc is one week, more stable than last

  Week 9-10:   FINAL RELEASE
               Linus decides — no committee, no vote

The three trees:
  Mainline   Linus's tree — cutting edge
  Stable     Greg KH's tree — backports, 6 months
  LTS        2 to 6 years support
             Android phones run LTS kernels

Real example — Linux 6.8:
  Jan 7  2024  6.7 released, merge window opens
  Jan 21       6.8-rc1 released
  Mar 10       6.8 final released (62 days total)
```

---

### [05 — Who Contributes and Why](./05-who-contributes-and-why/README.md)

> Intel, Google, Red Hat, Samsung, Microsoft — why do competing companies all fund work on the same project? What do independent contributors do? Where do you fit in?

```
Top corporate contributors (approximate):
  Intel        ~11%   Drivers, CPU performance, security
  Red Hat/IBM  ~8%    Enterprise reliability, KVM, SELinux
  Google       ~6%    Android, BPF, cloud performance
  Samsung      ~5%    ARM SoCs, display, camera drivers
  Microsoft    ~2%    Hyper-V, WSL, Azure (yes, Microsoft)

The business logic in one sentence:
  Maintaining a private fork of 36M lines that
  changes 70 times per day costs MORE than
  contributing upstream and letting everyone
  else help maintain it.

Entry paths for new contributors:
  ├── Google Summer of Code (paid, 12-22 weeks)
  ├── Linux Kernel Mentorship (3 cohorts/year)
  └── Outreachy ($7,000 stipend)
```

---

## How These Five Files Connect

```
01-what-the-kernel-is
    └── What it is and what it does at runtime

02-how-the-kernel-is-organised
    └── How 36M lines is structured into subsystems

03-how-a-change-moves-through-the-system
    └── How a patch travels through that structure

04-how-releases-work
    └── The timing and machinery of releases

05-who-contributes-and-why
    └── The humans and companies behind all of it
          │
          v
    Ready for: the-people/
```

---

> **Next: [the-people/](../the-people/README.md) — now that you understand the machine, meet the people who run it.**
