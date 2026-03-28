# Finding What to Work On

---

> You have a working build environment. You want to contribute to the Linux kernel. Now the hardest question: what do you actually work on? The kernel is 36 million lines. There is no issue tracker with "good first issue" labels. No one is going to assign you a task. How do you find something real to fix, at a difficulty level that will not get you crushed by a maintainer telling you the approach is fundamentally wrong?

**This file gives you concrete, specific places to look — from the absolute beginner entry point all the way to finding real bugs that need fixing.**

---

## What's Inside This File

```
09-finding-what-to-work-on/
│
├── The Staging Tree — why every beginner starts here
│   ├── What drivers/staging/ is
│   ├── How to find TODO items
│   └── What kinds of fixes are appropriate
├── Kernel Janitor Tasks — cleanup work
│   ├── checkpatch.pl warnings in real drivers
│   └── Coccinelle-identified patterns
├── Syzkaller — real bugs found by fuzzing
│   └── How to read and reproduce them
├── CVEs and Security Fixes
│   └── When and how to work on these
├── Finding a Subsystem You Care About
│   └── How to pick something you will stay with
└── What to Avoid as a New Contributor
```

---

## The Staging Tree — Why Every Beginner Starts Here

`drivers/staging/` is a directory in the kernel tree that exists specifically as an entry point for code that is not yet ready for the main tree. Greg Kroah-Hartman created it and maintains it personally.

### What drivers/staging/ Is

```
What staging contains:

drivers/staging/
├── Drivers that work but violate coding standards
├── Drivers with known TODO items (explicitly listed)
├── Drivers being cleaned up before graduation
│   to the main tree
└── Experimental code being evaluated for inclusion

What staging is NOT:
├── Abandoned code (it gets removed if nobody works on it)
├── A dumping ground (drivers must be useful)
└── Easy to graduate from (standards for leaving
    staging are the same as for the main tree)

Current staging drivers (examples):
├── drivers/staging/media/     -- media drivers
├── drivers/staging/vt6655/   -- WiFi driver
├── drivers/staging/wfx/      -- WiFi driver
├── drivers/staging/rtl8723bs/ -- Realtek WiFi
└── many others
```

### Why Beginners Start Here

Greg has explicitly and publicly stated that staging exists partly as an on-ramp for new contributors. He reviews patches to staging with more patience than elsewhere. He will point out what is wrong and explain why. He will give you multiple chances to get it right.

```
The staging advantage for new contributors:

Lower barrier:
└── Code already has known problems (the TODO files)
└── You are not finding bugs — they are listed for you
└── Fixing known issues is clearer than finding new ones

Explicit guidance:
└── TODO files tell you exactly what needs doing
└── checkpatch.pl on staging drivers always finds issues
└── You know what "done" looks like

Greg's review style for staging:
└── Patient with beginners
└── Will explain what is wrong clearly
└── Will not reject your patch without telling you why
└── Has said publicly he wants more contributors here

Path to mainline:
└── Once a staging driver is clean enough,
    Greg or the subsystem maintainer moves it
    to the main tree
└── Your cleanup contributions help that happen
```

### How to Find TODO Items

Every staging driver has a TODO file listing known problems.

```
Find TODO files:
find drivers/staging -name "TODO" | head -20

Example TODO file (drivers/staging/vt6655/TODO):
  TODO:
  - Fix sparse warnings
  - Fix checkpatch.pl warnings
  - Review against current coding standards
  - Convert to use kernel coding style
  - Test with current kernels

That is a task list. Pick one item. Fix it. Submit.
```

### Checking for checkpatch Warnings

Every staging driver has checkpatch warnings. Find them:

```
# Run checkpatch on an entire staging driver directory:
./scripts/checkpatch.pl --strict -f \
  drivers/staging/vt6655/*.c \
  drivers/staging/vt6655/*.h

# This outputs every warning in those files
# Pick one type of warning
# Fix all instances of that warning in the driver
# Submit as one patch

Good first staging patches:
├── Fix all "trailing whitespace" in a driver
├── Fix all "line over 80 characters" in a driver
│   (where the fix does not reduce readability)
├── Replace uint8_t with u8, uint32_t with u32
├── Add missing blank lines after declarations
├── Fix spacing around operators
└── Remove unnecessary parentheses
```

### What Makes a Good Staging Patch

```
Good staging patch:
├── Fixes ONE type of issue across ONE driver
├── Does NOT mix different fix types
│   └── Bad: "Fix whitespace AND rename variables"
│   └── Good: "Fix trailing whitespace in vt6655"
├── Does NOT change functionality
│   └── Pure style/cleanup only
│   └── No logic changes in cleanup patches
└── Passes checkpatch.pl with zero warnings
    on the files you touched

Subject line format:
Staging: vt6655: fix trailing whitespace issues
Staging: rtl8723bs: use kernel integer types

NOT:
"clean up code"       <- too vague
"fix issues"          <- what issues?
"improve vt6655"      <- means nothing
```

---

## Kernel Janitor Tasks — Cleanup Work

Beyond staging, the entire kernel has cleanup opportunities. These are called "kernel janitor" tasks — maintenance work that improves code quality without changing functionality.

### Finding checkpatch Warnings in Real Drivers

```
Run checkpatch on any driver directory:
./scripts/checkpatch.pl --strict -f drivers/net/ethernet/intel/e1000/*.c

Most real drivers have minor warnings.
These are legitimate patches for mainline
(not just staging) if done carefully.

Important distinction:
└── Staging: any cleanup welcome, Greg is patient
└── Mainline: cleanup patches must be clean and
    purposeful — do not send trivial one-line
    whitespace fixes to mainline drivers unless
    they are part of a meaningful cleanup

Good cleanup patch for mainline:
└── Convert all proc_create() calls in a driver
    to use the modern proc_ops interface
└── Replace all open-coded container_of() patterns
    with proper struct embedding
└── Modernise an old driver to use devm_ allocators
    (device-managed resources — cleaner cleanup paths)
```

### Coccinelle-Identified Patterns

The kernel ships semantic patch scripts in `scripts/coccinelle/` that identify code patterns that should be modernised.

```
Find coccinelle warnings across staging:
make coccicheck \
  MODE=report \
  M=drivers/staging/vt6655/

Find coccinelle warnings across a specific subsystem:
make coccicheck \
  COCCI=scripts/coccinelle/api/kzalloc-simple.cocci \
  MODE=report \
  M=drivers/net/

What this finds:
└── kmalloc() + memset(0) patterns
    that should be kzalloc()
└── copy_from_user() return value not checked
└── Various API misuse patterns
└── Each one is a potential patch

Example fix:
Wrong:
  buf = kmalloc(size, GFP_KERNEL);
  if (!buf)
      return -ENOMEM;
  memset(buf, 0, size);

Right:
  buf = kzalloc(size, GFP_KERNEL);
  if (!buf)
      return -ENOMEM;
```

---

## Syzkaller — Real Bugs Found by Fuzzing

Syzkaller is a coverage-guided kernel fuzzer developed by Google. It runs continuously against the Linux kernel and finds real bugs — memory corruption, use-after-free, null pointer dereferences, deadlocks.

These are not cleanup tasks. These are real bugs that can crash systems or cause security vulnerabilities. Working on them is significantly harder than staging cleanup but significantly more impactful.

### The Syzkaller Dashboard

```
Live bug reports:
https://syzkaller.appspot.com/upstream

What you see there:
├── List of open bugs found by syzkaller
├── Each entry has:
│   ├── Bug title (e.g. "KASAN: use-after-free in...")
│   ├── How many times it has been reproduced
│   ├── A C reproducer (code to trigger the bug)
│   ├── Kernel crash log (the oops/KASAN report)
│   └── The kernel config needed to reproduce it

Filtering for approachable bugs:
└── Look for bugs with a C reproducer
    (much easier to work with than syzprog)
└── Look for bugs in subsystems you know
└── Look for bugs with clear crash logs
    (stack trace points to the problem area)
└── Avoid bugs marked "corrupted" or with
    very long open time (may be very hard)
```

### How to Work on a Syzkaller Bug

```
Step 1 — Reproduce the bug:
├── Get the reproducer C program from the dashboard
├── Get the kernel config from the dashboard
├── Build a kernel with that config + KASAN enabled:
│   make CONFIG_KASAN=y CONFIG_KASAN_INLINE=y \
│     CONFIG_KCSAN=y ...
├── Boot in QEMU
└── Run the reproducer
    If you see the same crash: you can reproduce it

Step 2 — Read the crash:
├── KASAN report tells you:
│   ├── What type of error (use-after-free, OOB...)
│   ├── What address was accessed
│   ├── The allocation stack trace (where it was allocated)
│   ├── The free stack trace (where it was freed)
│   └── The access stack trace (where the bad access was)
└── The combination tells you what happened

Step 3 — Find the bug:
└── Trace the stack through the kernel source
└── Understand why the freed memory was accessed again
└── Or why the out-of-bounds access happened

Step 4 — Fix and test:
└── Apply fix
└── Verify the reproducer no longer crashes
└── Verify no new crashes under normal use

Step 5 — Submit:
└── Normal patch submission process
└── CC: stable@vger.kernel.org if it is a regression
└── Reference the syzkaller report in the commit message:
    Link: https://syzkaller.appspot.com/bug?id=XXXXX
```

---

## CVEs and Security Fixes

CVEs (Common Vulnerabilities and Exposures) are publicly disclosed security vulnerabilities with assigned identifiers. When a CVE affects the Linux kernel, a fix needs to be written and submitted.

### Where to Find Kernel CVEs

```
Sources of kernel CVE information:
├── cve.mitre.org — official CVE database
├── nvd.nist.gov — National Vulnerability Database
│   └── Search "linux kernel"
│   └── Filter by CVSS score for severity
├── kernel.org/docs/CVEs/ — kernel-specific CVE list
└── oss-security@openwall.com mailing list
    └── Where many kernel CVEs are first disclosed
```

### What Working on CVEs Involves

```
Types of kernel CVEs:

Already fixed upstream, needs stable backport:
└── The fix exists in mainline
└── Someone needs to backport it to stable branches
└── CC: stable@vger.kernel.org on the mainline commit
    triggers the stable team to consider it
└── Or submit the backport directly to stable

Not yet fixed:
└── You need to understand the vulnerability
└── Write a fix
└── Submit through normal channels
└── This is harder and more sensitive work

Important for CVE work:
└── Do not publicly discuss an unfixed vulnerability
└── Use security@kernel.org for private disclosure
└── If it is already public (in the CVE database),
    normal public submission is fine
```

---

## Finding a Subsystem You Care About

The most productive contributors work on things they actually care about or use. Random cleanup patches in drivers you never use will get boring quickly.

### Ask Yourself These Questions

```
Questions to find your subsystem:

"What hardware do I use daily?"
├── WiFi driver for your card
├── Sound driver for your audio chip
├── GPU driver for your graphics
└── Storage driver for your NVMe/SATA

"What software stack do I work with?"
├── You write networked applications → net/
├── You work with containers → cgroups, namespaces
├── You do security work → security/, audit
└── You work with databases → io_uring, block layer

"What do I want to learn?"
├── CPU architecture internals → arch/x86 or arch/arm64
├── How filesystems work → fs/ext4 or fs/btrfs
├── How networking works → net/ipv4 or net/core
└── How memory works → mm/

"Where can I have the most impact?"
├── Find subsystems with few maintainers
│   └── Check MAINTAINERS — look for "Odd Fixes"
│       status or single maintainers
└── Find subsystems with hardware you have
    that others might not
```

### Reading the Subsystem Code

Before submitting anything to a subsystem, read it.

```
How to get familiar with a subsystem:

1. Read the documentation:
   Documentation/networking/   (for net/)
   Documentation/filesystems/  (for fs/)
   Documentation/admin-guide/  (general)
   Documentation/process/      (contribution process)

2. Read the code for a small driver in the subsystem:
   └── Pick a driver under 1000 lines
   └── Read it top to bottom
   └── Understand its structure

3. Subscribe to the mailing list:
   └── Read it for 2-4 weeks before posting
   └── Understand the tone and culture
   └── See what kinds of patches get accepted

4. Build and test the subsystem:
   └── Compile the relevant modules
   └── Test that they work on your hardware
   └── This means you can test your own patches

5. Run the tools on it:
   ./scripts/checkpatch.pl --strict -f subsystem/*.c
   make C=1 subsystem/
   └── See what comes up
   └── Now you have potential patches
```

---

## Specific Beginner Resources

### kernelnewbies.org

```
kernelnewbies.org:
└── Specifically for new kernel contributors
└── Maintains a list of current TODO tasks
└── Links to current "easy" bugs
└── Documentation on getting started
└── IRC channel: #kernelnewbies on irc.libera.chat
   Good place to ask "where do I start" questions
```

### First Kernel Patch Tutorial

The kernel documentation has an official guide:

```
Documentation/process/submitting-patches.rst
└── The official guide to submitting patches
└── Required reading before submitting anything
└── Covers the exact format maintainers expect

Documentation/process/coding-style.rst
└── The kernel coding style guide
└── checkpatch.pl enforces most of this
└── Read it once before writing any code

Documentation/process/kernel-docs.rst
└── Index of all kernel documentation
└── Find docs for specific subsystems here
```

---

## What to Avoid as a New Contributor

Learning what not to do saves you time and protects your reputation on the mailing list.

```
Things to avoid:

Trivial one-word patches:
└── "Fix typo: 'teh' → 'the' in comment"
└── These are legitimate but sending 10 of them
    in a row signals you are gaming contribution
    counts rather than doing real work
└── One or two is fine — do not make it a pattern

Patches that change whitespace for no reason:
└── Not part of fixing a real bug
└── "I reformatted this function because I prefer
    this style" → rejected

Patches sent during the merge window:
└── Maintainers are busy with pull requests
└── Your patch will get lost
└── Wait until rc1 is released

Patches to Linus directly:
└── Unless get_maintainer.pl says to send to him
└── Almost never correct for a new contributor

Patches that say "I'll clean this up later":
└── Clean it up first
└── The kernel does not merge
    "I'll fix this in a follow-up" patches

Asking "is my patch merged yet?" after 3 days:
└── Wait 2-4 weeks minimum
└── Check patchwork for status first
└── If not in patchwork at all after 3 days,
    your email may not have been formatted correctly
```

---

## The Right Mindset

```
What successful new contributors do:

Pick ONE thing:
└── Not "I will contribute to the kernel"
└── "I will fix the checkpatch warnings in
    drivers/staging/vt6655/ this weekend"
└── Specific. Bounded. Achievable.

Do the work properly:
└── Run all the tools
└── Read the subsystem documentation
└── Look at similar patches in the mailing list archive
    to understand expected format and style

Submit and learn from feedback:
└── Your first patch will probably need changes
└── That is normal — not a failure
└── Apply the feedback, send v2
└── The feedback is teaching you the standards

Repeat:
└── Second patch is easier than first
└── Third patch is easier than second
└── After 10 patches you understand the process
└── After 50 patches you know a subsystem
└── After 100 patches maintainers know your name
```

The kernel community does not need more patches — it needs more good patches. Quality over quantity, every time.

---

> **You know what to work on. Next: how do you actually write the patch — the commit message format, the diff format, the Signed-off-by line, and what makes a patch reviewable versus what makes it immediately suspicious.**
> **→ [10-writing-the-patch](../10-writing-the-patch/README.md)**
