# Setting Up Your Environment

---

> Before you write a single line of kernel code, your environment needs to be right. The wrong git configuration will corrupt your patches. The wrong email setup will make them unreadable. Building against the wrong tree will get your patch rejected before anyone reads the code. The tools you skip installing will catch you later when a maintainer asks why you did not run them.

**This file gets your environment right once, so it never gets in the way again.**

---

## What's Inside This File

```
08-setting-up-your-environment/
│
├── Getting the Kernel Source — which tree and why
│   ├── Linus's mainline tree
│   ├── linux-next for new features
│   └── Subsystem trees for targeted work
├── Build Dependencies — exact packages
├── Your First Kernel Build
│   ├── Minimal config for fast iteration
│   └── Building and verifying it works
├── Git Configuration for Kernel Work
│   ├── Identity — name and email
│   ├── send-email configuration
│   └── Useful aliases and settings
└── Tools You Need Before Writing a Line
    ├── checkpatch.pl
    ├── sparse
    ├── smatch
    └── coccinelle
```

---

## Getting the Kernel Source

There is no single "the kernel source." There are multiple trees and you need the right one for your type of work. Getting this wrong is the most common cause of patches that cannot be applied.

### Linus's Mainline Tree

```
What it is:
└── The official kernel tree maintained by Linus
└── The reference point for all other trees
└── What becomes the official release

Clone it:
git clone \
  https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
  linux

This is large (~5GB with full history).
For faster cloning, use --depth to limit history:

git clone --depth=1 \
  https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git \
  linux

Warning: shallow clone limits some git operations.
For serious development, clone the full history.

When to use mainline:
└── Fixing a bug that is in the current release
└── Understanding the current state of something
└── When you are not sure which tree to use
```

### linux-next — For New Features

```
What it is:
└── Integration tree — all subsystem -next trees
    merged together daily
└── Represents what the NEXT kernel release will look like
└── Where integration conflicts are caught early

Clone it:
git clone \
  https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git \
  linux-next

Or add it as a remote to your existing clone:
cd linux
git remote add linux-next \
  https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
git fetch linux-next
git checkout -b my-feature linux-next/master

When to use linux-next:
├── Developing a new feature
├── Adding a new driver
├── Any change that is not a bug fix
└── When the maintainer asks you to rebase on it
```

### Subsystem Trees — For Targeted Work

```
What they are:
└── Each subsystem maintainer has their own git tree
└── Your patch lands here before going to Linus
└── Base your work on the subsystem tree when
    the maintainer specifically asks you to

Examples:
Networking:
  git.kernel.org/pub/scm/linux/kernel/git/netdev/net-next.git

USB:
  git.kernel.org/pub/scm/linux/kernel/git/gregkh/usb.git

Memory management:
  git.kernel.org/pub/scm/linux/kernel/git/akpm/mm.git

When to use subsystem tree:
└── Maintainer told you to rebase on their tree
└── Your patch depends on unreleased work in their tree
└── You are doing targeted work in one subsystem
    and want the most up-to-date base for it
```

### Which Tree to Pick — Decision Guide

```
Decision guide:

"I found a bug in a released kernel"
└── Base on: stable tree for that release
    OR mainline if already fixed there

"I want to add a new feature"
└── Base on: linux-next
└── Or the subsystem's -next tree

"I want to clean up code in drivers/staging/"
└── Base on: Greg's staging tree
    git.kernel.org/.../gregkh/staging.git
└── Or linux-next

"I am not sure"
└── Base on: linux-next
└── Ask the maintainer if you are wrong,
    they will tell you

"Maintainer said rebase on my tree"
└── Find their tree in the MAINTAINERS file
└── T: entries list the tree URLs
```

---

## Build Dependencies

Install these before trying to build the kernel. Missing dependencies produce confusing error messages that waste time.

### On Debian / Ubuntu

```
sudo apt-get install \
  build-essential \
  libncurses-dev \
  bison \
  flex \
  libssl-dev \
  libelf-dev \
  bc \
  dwarves \
  git \
  python3 \
  pahole

# For sparse (static analysis):
sudo apt-get install sparse

# For smatch (static analysis):
# Not in apt — build from source (see below)

# For coccinelle (semantic patches):
sudo apt-get install coccinelle
```

### On Fedora / RHEL / CentOS

```
sudo dnf install \
  gcc \
  make \
  ncurses-devel \
  bison \
  flex \
  openssl-devel \
  elfutils-libelf-devel \
  bc \
  dwarves \
  git \
  python3 \
  pahole

# For sparse:
sudo dnf install sparse

# For coccinelle:
sudo dnf install coccinelle
```

### On Arch Linux

```
sudo pacman -S \
  base-devel \
  ncurses \
  bison \
  flex \
  openssl \
  libelf \
  bc \
  pahole \
  git \
  python

# sparse:
sudo pacman -S sparse

# coccinelle:
sudo pacman -S coccinelle
```

### Verifying Your Build Tools

```
# Check gcc version (need 5.1+, prefer 10+):
gcc --version

# Check make:
make --version

# Check that kernel scripts work:
cd linux
./scripts/checkpatch.pl --version

# Check sparse:
sparse --version

# Check coccinelle:
spatch --version
```

---

## Your First Kernel Build

Building the kernel confirms your toolchain works and gives you a kernel you can test your patches against. Do this before writing any code.

### Step 1 — Get a Config

Building with a full distribution config takes 30-60 minutes and produces a huge kernel. For development, use a minimal config.

```
Option A — Use your running kernel's config (easiest):
cp /boot/config-$(uname -r) .config
make olddefconfig
# Updates config for any new options, using defaults

Option B — Minimal local config (fastest builds):
make localmodconfig
# Generates a config with only the modules
# currently loaded on your system
# Results in much smaller, faster builds

Option C — Absolute minimum for testing (fastest):
make tinyconfig
# Extremely minimal — may not boot your system
# Good for compile testing specific subsystems
# Not good for functional testing
```

For development work, Option B (localmodconfig) is the best balance — it builds quickly and produces a kernel that boots on your machine.

### Step 2 — Build

```
# Use all CPU cores for parallel build:
make -j$(nproc)

# Or specify explicitly:
make -j8   # for 8 cores

# Build only a specific subsystem (faster iteration):
make -j$(nproc) drivers/usb/   # only USB drivers
make -j$(nproc) net/           # only networking
make -j$(nproc) fs/ext4/       # only ext4

# Check for warnings (important — treat as errors):
make -j$(nproc) 2>&1 | grep -E "warning:|error:"
```

A successful build produces:
```
arch/x86/boot/bzImage   <- compressed kernel image
vmlinux                 <- uncompressed kernel (for debugging)
System.map              <- symbol table
```

### Step 3 — Install and Test (Optional)

```
# Install modules:
sudo make modules_install

# Install kernel:
sudo make install

# Reboot into your new kernel:
sudo reboot

# Verify version after reboot:
uname -r
```

For kernel development, you often do not need to install. You can:
- Use QEMU/KVM to test the kernel without rebooting
- Use `make -j$(nproc)` to verify it compiles
- Boot with kexec to switch kernels without full reboot

### Testing in QEMU (Recommended for Development)

Rebooting your development machine every time you test a kernel change is slow and risky. Use QEMU:

```
# Install QEMU:
sudo apt-get install qemu-system-x86   # Debian/Ubuntu
sudo dnf install qemu-system-x86      # Fedora

# Build a minimal initramfs (for testing):
# (details depend on your distro and setup)

# Boot your kernel in QEMU:
qemu-system-x86_64 \
  -kernel arch/x86/boot/bzImage \
  -initrd /path/to/initramfs.img \
  -append "console=ttyS0 root=/dev/sda" \
  -m 512M \
  -nographic

# This gives you a kernel running in a VM
# Safe to crash, safe to test risky changes
# Exit QEMU: Ctrl-a x
```

The kernel project maintains a tool called `virtme-ng` specifically for this:

```
# Install virtme-ng:
pip install virtme-ng

# Boot current directory kernel in QEMU:
virtme-ng --run

# This handles initramfs automatically
# Gives you a shell inside your test kernel
# Very fast iteration cycle
```

---

## Git Configuration for Kernel Work

Your git configuration directly affects your patches. Wrong settings produce patches that are rejected for technical reasons before anyone reads the code.

### Identity — Name and Email

```
# Set your identity:
git config --global user.name "Your Real Name"
git config --global user.email "you@yourdomain.com"

Requirements:
├── Use your real name — pseudonyms are not accepted
│   Your Signed-off-by must use your real identity
├── Use an email you check regularly
│   Review feedback comes here
│   Bot reports come here
└── This email appears permanently in git history
    for every patch you contribute

Important: the email in git config must match
the email in your Signed-off-by line.
Mismatches cause confusion and look sloppy.
```

### send-email Configuration

`git send-email` is how you send patches. Configure it once correctly and it works forever.

```
# Full send-email configuration in ~/.gitconfig:

[sendemail]
    # Your SMTP server settings:
    smtpServer = smtp.yourdomain.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = you@yourdomain.com

    # Confirm before sending:
    confirm = always

    # Thread patches in a series automatically:
    chainreplyto = false

    # Suppress CC to self:
    suppresscc = self

    # Annotate with 'From' in body for git am:
    annotate = no

# For Gmail specifically:
[sendemail]
    smtpServer = smtp.gmail.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = you@gmail.com
    # Note: Gmail requires an App Password
    # not your regular Gmail password
    # Generate at: myaccount.google.com/apppasswords
```

### Testing Your send-email Setup

```
# Test without actually sending:
git send-email \
  --dry-run \
  --to="test@example.com" \
  0001-your-patch.patch

# Send to yourself first:
git send-email \
  --to="you@yourdomain.com" \
  0001-your-patch.patch

# Check the received email:
# ├── Is it plain text? (not HTML)
# ├── Are the tabs preserved? (not converted to spaces)
# ├── Does the diff apply cleanly?
# └── Is the subject line correct?

# Verify the patch applies:
git am < received_email.eml
```

### Useful Git Aliases for Kernel Work

```
# Add to ~/.gitconfig:

[alias]
    # Show log with one line per commit:
    lol = log --oneline

    # Show files changed in last N commits:
    files = diff --name-only HEAD~1

    # Create patch for last N commits:
    fp = format-patch

    # Check what changed since a base:
    since = log --oneline origin/master..HEAD

    # Find which commit introduced a string:
    pickaxe = log -S

    # Show commit with full diff:
    show-patch = show --format=fuller

    # Interactive rebase for cleaning up commits:
    fixup = rebase -i origin/master
```

### Other Important Git Settings

```
[core]
    # Use vim for commit messages (or your preference):
    editor = vim

    # Warn on whitespace errors:
    whitespace = fix

[diff]
    # Show function context in diffs:
    funcContext = true

[format]
    # Add cover letter for patch series:
    coverLetter = auto

    # Numbered patches:
    numbered = true

    # Thread patches in a series:
    thread = true

[log]
    # Show dates in a useful format:
    date = relative
```

---

## Tools You Need Before Writing a Line

These tools are not optional. Maintainers expect you to run them before submitting. Submissions that fail these checks get immediate negative feedback.

---

### checkpatch.pl — Coding Style Checker

`checkpatch.pl` is a Perl script in the kernel tree at `scripts/checkpatch.pl`. It checks for violations of the kernel coding style and common mistakes.

```
How to run it:

# On a specific file:
./scripts/checkpatch.pl --strict -f drivers/usb/core/hub.c

# On a patch file:
./scripts/checkpatch.pl --strict 0001-my-patch.patch

# On all your staged changes:
git diff HEAD | ./scripts/checkpatch.pl --strict -

# Output levels:
ERROR:   Must fix before submitting — no exceptions
WARNING: Should fix — explain if you cannot
CHECK:   Consider fixing — sometimes ignore is ok

# Example output:
ERROR: space prohibited before open square bracket '['
#42: FILE: drivers/usb/core/hub.c:42:
+       array [index] = value;

WARNING: line over 80 characters
#67: FILE: drivers/usb/core/hub.c:67:
+       very_long_function_name(argument_one,
+               argument_two, argument_three);

CHECK: Prefer kernel type 'u8' over 'uint8_t'
#89: FILE: drivers/usb/core/hub.c:89:
+       uint8_t value;
```

Fix every ERROR. Fix every WARNING unless you have a specific documented reason not to. CHECKs are suggestions — use your judgment.

```
Common checkpatch errors and fixes:

"trailing whitespace"
└── Remove spaces/tabs at end of lines
└── Most editors can do this automatically

"line over 80 characters"
└── Break long lines
└── Exception: if breaking would make it less readable
    Add a comment explaining why

"space before open parenthesis"
└── Wrong: function_name (arg)
└── Right: function_name(arg)
└── Exception: if/while/for/switch DO have a space
    Correct: if (condition)

"missing blank line after declaration"
└── Add a blank line between variable declarations
    and the first statement

"should be using kzalloc"
└── kzalloc zero-initialises — safer than kmalloc
└── Use kzalloc unless you have a performance reason

"use of __DATE__ or __TIME__"
└── These make builds non-reproducible
└── Remove them
```

---

### sparse — Static Analysis

sparse is a semantic checker for C code developed specifically for the Linux kernel. It catches type errors that the compiler misses — particularly around kernel-specific type annotations.

```
Install:
sudo apt-get install sparse   # Debian/Ubuntu
sudo dnf install sparse       # Fedora

Run:
make C=1 drivers/usb/core/hub.o
# C=1 runs sparse on files being compiled

make C=2 drivers/usb/core/hub.o
# C=2 runs sparse on ALL files including dependencies
# Much slower but catches more issues

What sparse catches:
├── __user pointer used in kernel context
│   └── Data from userspace must be copied with
│       copy_from_user() — not accessed directly
├── __iomem pointer used as regular pointer
│   └── I/O memory has special access requirements
├── Endianness errors
│   └── __le32 (little-endian) mixed with __be32
│       (big-endian) — common in networking drivers
├── Lock imbalances (simple cases)
└── Integer size mismatches

Example sparse output:
drivers/usb/core/hub.c:523:14: warning:
  incorrect type in assignment (different address spaces)
  expected struct usb_hub *hub
  got struct usb_hub [noderef] __rcu *hub
```

sparse errors involving `__user`, `__iomem`, and endianness annotations are always worth fixing — they indicate real potential bugs.

---

### smatch — Deeper Static Analysis

smatch is a more powerful static analysis tool that catches bugs sparse misses — null pointer dereferences, out-of-bounds access, resource leaks.

```
smatch is not in most package managers.
Build it from source:

git clone https://repo.or.cz/smatch.git
cd smatch
make
sudo make install

Run against the kernel:
make CHECK="smatch -p=kernel" C=1 \
  drivers/usb/core/hub.o

What smatch catches:
├── NULL dereference paths
│   └── "hub->dev could be NULL here"
├── Use after free
│   └── "dev was freed on line 423,
│       used again on line 431"
├── Integer overflow
│   └── "len * sizeof(int) can overflow"
├── Resource leaks
│   └── "mutex locked on line 234 but
│       not unlocked on error path"
└── Off-by-one errors in buffer access

smatch is slower than sparse.
Run it on files you modified, not the whole kernel.
It is particularly valuable for new drivers
where you might have introduced resource leaks.
```

---

### coccinelle — Semantic Patches

coccinelle is a program transformation tool. It applies "semantic patches" — patches that match code patterns rather than exact text. The kernel uses it to apply systematic changes across the entire codebase.

```
Install:
sudo apt-get install coccinelle   # Debian/Ubuntu
sudo dnf install coccinelle       # Fedora

Run the kernel's own coccinelle scripts:
make coccicheck \
  COCCI=scripts/coccinelle/api/kzalloc-simple.cocci \
  MODE=report \
  M=drivers/usb/core/

Run all coccinelle checks:
make coccicheck MODE=report M=drivers/usb/core/

What coccinelle does:
├── Finds patterns that should be modernised
│   └── "This should use kzalloc instead of
│       kmalloc + memset"
├── Finds patterns the API guidelines prohibit
│   └── "Use devm_kzalloc instead of kzalloc
│       in probe functions"
├── Finds common bug patterns
│   └── "sizeof applied to pointer, not struct"
└── Systematically applies cleanups across subsystems
    One coccinelle script can generate hundreds
    of patches touching thousands of files

Why coccinelle matters for beginners:
└── drivers/staging/ has known coccinelle warnings
└── Fixing them is a common first contribution
└── The fix is mechanical but must still be correct
└── Good practice for understanding patterns
```

---

## Your Environment Checklist

Before writing your first patch, verify everything:

```
Environment checklist:

Source:
[ ] Cloned the right tree for your type of work
[ ] git log shows recent commits (tree is up to date)
[ ] git remote -v shows the upstream correctly

Build:
[ ] make -j$(nproc) completes without errors
[ ] Zero new warnings compared to clean tree
[ ] Kernel boots in QEMU or real hardware

Git config:
[ ] git config user.name shows your real name
[ ] git config user.email shows your real email
[ ] git send-email --dry-run works without errors
[ ] Test email to yourself passes (plain text, no corruption)

Tools:
[ ] ./scripts/checkpatch.pl --version works
[ ] sparse --version works
[ ] spatch --version works (coccinelle)
[ ] smatch --version works (if built from source)

Test run:
[ ] Run checkpatch.pl on an existing file — no errors
    (the existing code should pass — if it does not,
    check that you are on the right tree)
[ ] Run sparse on an existing file — understand output
[ ] Run coccinelle on drivers/staging/ — find something

Ready:
[ ] You can build the kernel
[ ] You can check your code
[ ] You can send email correctly
[ ] Now you can write a patch
```

---

> **Your environment is ready. Next: what should you actually work on? Where do beginners find their first patch — and how do you find something that matches your skill level?**
> **→ [09-finding-what-to-work-on](../09-finding-what-to-work-on/README.md)**
