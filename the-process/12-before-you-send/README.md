# Before You Send

---

> Your patch is written. Your commit message is correct. You know who to send it to. But there is one more stage that separates patches that get reviewed cleanly from patches that get immediate pushback before anyone reads the code. This is the quality gate — the checks that catch problems you missed, the tools that find bugs before a maintainer does, and the verification that your patch actually applies and builds correctly.

**Do these checks every time. Not most times. Every time. A patch that fails checkpatch.pl costs you the reviewer's goodwill before they read a single line of your code.**

---

## What's Inside This File

```
12-before-you-send/
│
├── The Pre-Send Mindset
├── checkpatch.pl — complete guide
│   ├── How to run it correctly
│   ├── Every output level explained
│   ├── Common errors and exact fixes
│   └── When it is acceptable to ignore a warning
├── sparse — static type checking
│   ├── How to run it
│   └── What its warnings mean
├── smatch — deeper analysis
│   ├── How to run it
│   └── What it catches that sparse misses
├── Building — what to verify
│   ├── Zero new warnings rule
│   └── Build configurations to test
├── Testing — what is the minimum bar
│   ├── By type of change
│   └── How to document what you tested
└── The Complete Pre-Send Checklist
```

---

## The Pre-Send Mindset

Before going through the specific tools, understand the standard you are working to.

The Linux kernel runs on hundreds of millions of devices. A bug that ships in a kernel release cannot be fixed with a silent app update — it requires a kernel update, which requires distribution packaging, which requires users actually updating. On embedded systems and Android phones, some kernel bugs never get patched because the device never receives kernel updates.

This is why the pre-submission bar is high. It is not gatekeeping. It is the natural consequence of the stakes.

```
The standard you are working to:

"Would a senior kernel developer be embarrassed
 to have submitted this patch?"

Signs the answer is NO (good):
├── checkpatch.pl passes with zero errors
├── All warnings either fixed or documented
├── sparse shows no new warnings
├── Kernel builds with zero new warnings
├── You have tested that the fix actually works
├── You can explain every line of the diff
└── The commit message tells a complete story

Signs the answer is YES (fix before sending):
├── "I'll fix the checkpatch warning if they ask"
├── "The build warning was already there"
├── "I haven't tested it but the logic looks right"
├── "I'm not sure why that line is there but
    removing it seemed to help"
└── "It compiles, that's good enough"
```

---

## checkpatch.pl — Complete Guide

`checkpatch.pl` is the first thing every maintainer mentally checks when they see a patch. Patches with checkpatch errors signal that the contributor did not do basic preparation. Some maintainers will not even read the code until checkpatch issues are resolved.

### How to Run It Correctly

```
On a generated patch file (most common):
./scripts/checkpatch.pl --strict \
  0001-subsystem-fix-the-thing.patch

On a specific source file:
./scripts/checkpatch.pl --strict -f \
  drivers/usb/core/hub.c

On your current uncommitted changes:
git diff | ./scripts/checkpatch.pl --strict -

On all files in a directory:
./scripts/checkpatch.pl --strict -f \
  drivers/staging/vt6655/*.c

The --strict flag enables additional checks.
Always use it. If you plan to submit to mainline,
your code should pass even the strict checks.
```

### Every Output Level Explained

```
Three severity levels:

ERROR:
└── Must fix before submitting — no exceptions
└── Indicates a definitive coding standard violation
└── Maintainers will reject the patch for these
└── Example: "do not use assignment in if condition"

WARNING:
└── Should fix — explain clearly if you cannot
└── Indicates probable style violation or potential issue
└── Some warnings are legitimate in specific contexts
└── If you leave a warning, add a comment in the code
    explaining why and mention it in the cover letter

CHECK:
└── Consider fixing — sometimes acceptable to ignore
└── Suggestions that improve code quality
└── Not required for patch acceptance
└── Fixing CHECKs shows attention to detail
```

### Common Errors and Exact Fixes

These are the errors you will encounter most often. Know them and how to fix them.

---

**Trailing whitespace:**

```
ERROR: trailing whitespace
#12: FILE: drivers/usb/core/hub.c:12:
+       value = get_value();   $
                              ^-- space before newline

Fix:
Remove all spaces and tabs at the end of lines.

In vim: :%s/\s\+$//g
In sed: sed -i 's/[[:space:]]*$//' file.c

Prevention:
Set your editor to show trailing whitespace.
Most editors have this option — enable it.
git diff shows trailing whitespace as red blocks.
```

---

**Line over 80 characters:**

```
WARNING: line over 80 characters
#34: FILE: drivers/usb/core/hub.c:34:
+       ret = some_function(argument_one, argument_two,
+                           argument_three, arg4);

Fix option 1 — break the line:
        ret = some_function(argument_one, argument_two,
                            argument_three, arg4);

Fix option 2 — use a local variable:
        int local = some_function(argument_one,
                                  argument_two);
        ret = use_result(local, argument_three, arg4);

When you can ignore this warning:
└── The line contains a URL that cannot be broken
└── The line is a string that must not be split
└── Breaking it would genuinely reduce readability
└── In these cases: add a comment in your cover letter
    "checkpatch warns about line length on line X,
    breaking it would make it less readable"
```

---

**Space before open parenthesis in function call:**

```
ERROR: space prohibited before open parenthesis '('
#45: FILE: drivers/usb/core/hub.c:45:
+       result = function_name (argument);
                              ^-- space here

Fix:
        result = function_name(argument);

Exception — these SHOULD have a space:
        if (condition)
        while (condition)
        for (i = 0; i < n; i++)
        switch (value)
        return (value);  -- optional but common

Function CALLS never have a space.
Keywords always have a space.
```

---

**Missing space after keyword:**

```
ERROR: space required after ',' keyword
#56: FILE: drivers/usb/core/hub.c:56:
+       if(condition)
          ^-- missing space

Fix:
        if (condition)
        while (condition)
        for (i = 0; i < n; i++)
```

---

**Braces on wrong line:**

```
WARNING: braces {} are not necessary for single
statement branches
#67: FILE: drivers/usb/core/hub.c:67:
+       if (condition) {
+               do_thing();
+       }

Fix (single statement — no braces needed):
        if (condition)
                do_thing();

Exception — keep braces when:
└── The if has an else that has multiple statements
└── Another branch of the same if/else has braces
    (be consistent — all branches or none)
└── The single statement is complex enough
    that future readers might accidentally add a
    second statement without adding braces
```

---

**Use of deprecated or wrong function:**

```
WARNING: __DATE__ and __TIME__ are not appropriate
WARNING: use of deprecated function X
WARNING: please use Y instead of X

Examples:
├── kfree_skb() → dev_kfree_skb() in most driver code
├── kmalloc() + memset(0) → kzalloc()
├── pci_alloc_consistent() → dma_alloc_coherent()
└── These are API migrations the kernel is doing

Fix:
Use the suggested replacement.
If you are unsure why, read the kernel documentation
for the new API before converting.
```

---

**Missing blank line after declaration:**

```
WARNING: Missing a blank line after declarations
#78: FILE: drivers/usb/core/hub.c:78:
+       int ret;
+       int value;
+       ret = get_value();

Fix:
        int ret;
        int value;

        ret = get_value();

Blank line separates declarations from statements.
This is a kernel coding style requirement.
```

---

**Wrong integer type:**

```
CHECK: Prefer kernel type 'u8' over 'uint8_t'
CHECK: Prefer kernel type 'u32' over 'unsigned int'

Fix:
uint8_t  → u8
uint16_t → u16
uint32_t → u32
uint64_t → u64
int8_t   → s8
int16_t  → s16
int32_t  → s32
int64_t  → s64

These are defined in <linux/types.h>
Use them consistently throughout kernel code.
```

---

**Commit message issues:**

```
ERROR: Please use git commit description style
       "do not" instead of "don't"
WARNING: commit message subject line too long
WARNING: missing commit description

Fix:
├── Use "do not" not "don't"
├── Use "cannot" not "can't"
├── Keep subject under 72 characters
└── Add a body that explains what and why
```

---

### When It Is Acceptable to Ignore a Warning

Very few cases. Document them when you do.

```
Legitimate reasons to leave a WARNING:

Long line containing a URL:
└── // See https://very-long-url-that-cannot-be-split.com
└── Leave it. Add note to cover letter.

String that cannot be split:
└── pr_err("This error message must stay on one line\n");
└── Splitting it changes the output format users search for.
└── Leave it if genuinely necessary.

Alignment that checkpatch disagrees with:
└── Sometimes readable alignment conflicts with
    checkpatch's preferred style
└── If your version is genuinely more readable,
    document why in your commit message or cover letter

What is NEVER acceptable to ignore:
└── ERROR: level issues — fix them, no exceptions
└── Missing Signed-off-by — always required
└── Trailing whitespace — always fixable
└── Wrong brace style — always fixable
```

---

## sparse — Static Type Checking

sparse is a semantic C checker that understands Linux-specific type annotations. It catches mistakes the compiler cannot see.

### How to Run It

```
Build with sparse checking enabled:

# Check specific files (C=1 = check compiled files):
make C=1 drivers/usb/core/hub.o

# Check all files including dependencies (C=2):
make C=2 drivers/usb/core/hub.o

# Check an entire directory:
make C=1 drivers/usb/core/

# Output goes to stderr alongside normal build output.
# Pipe to file to read later:
make C=1 drivers/usb/core/ 2>sparse_output.txt
```

### What sparse Catches

**Incorrect address space usage:**

```
__user — pointer to userspace memory
__iomem — pointer to I/O memory
__percpu — per-CPU pointer
__rcu — RCU-protected pointer

sparse enforces these do not mix:

drivers/usb/hub.c:523: warning:
  incorrect type in assignment (different address spaces)
  expected unsigned char *buf
  got unsigned char [noderef] __user *ubuf

This means:
└── ubuf is a __user pointer (from userspace)
└── buf is a regular kernel pointer
└── You cannot assign one to the other directly
└── You must use copy_from_user() or copy_to_user()
└── This is a real bug — fix it
```

**Endianness errors:**

```
__le16, __le32, __le64 — little-endian values
__be16, __be32, __be64 — big-endian values

sparse warns when you mix them:

drivers/net/mydriver.c:89: warning:
  incorrect type in assignment (different base types)
  expected unsigned int [usertype] value
  got restricted __be32 [usertype] be_value

This means:
└── be_value is a big-endian 32-bit value
└── value is a plain unsigned int (host endian)
└── Mixing them is a bug on big-endian machines
└── Use le32_to_cpu() or be32_to_cpu() to convert

Network protocols use big-endian.
x86 CPUs are little-endian.
This mismatch is a common source of bugs
that only appear on ARM or PowerPC systems.
```

**Lock annotation issues:**

```
__must_hold(lock) — function requires lock held
__acquires(lock)  — function acquires the lock
__releases(lock)  — function releases the lock

sparse checks that your code respects these:

warning: context imbalance in 'my_function' -
  unexpected unlock

This means:
└── You released a lock that was not held
└── Or held a lock you should not have
└── Real deadlock or use-after-unlock bug
└── Fix the locking logic
```

### Zero New sparse Warnings Rule

```
Before sending, compare sparse output:

# Sparse output on clean tree:
git stash
make C=1 drivers/usb/core/ 2>sparse_before.txt
git stash pop

# Sparse output with your patch:
make C=1 drivers/usb/core/ 2>sparse_after.txt

# Difference:
diff sparse_before.txt sparse_after.txt

Any new warnings in sparse_after.txt are your fault.
Fix them before sending.
```

---

## smatch — Deeper Analysis

smatch performs more sophisticated analysis than sparse — it traces execution paths and finds bugs that only appear under specific conditions.

### How to Run It

```
Build smatch from source (not in most package managers):

git clone https://repo.or.cz/smatch.git
cd smatch
make
export PATH=$PATH:$(pwd)

Run against specific files:
make CHECK="smatch -p=kernel" \
  C=1 \
  drivers/usb/core/hub.o 2>&1 | grep smatch

Run with full kernel database (slower, more accurate):
# First build the smatch database:
smatch --kerneldir=/path/to/linux \
  --project=kernel \
  --call-tree \
  drivers/usb/core/*.c
```

### What smatch Catches That sparse Misses

**Null pointer dereferences:**

```
smatch output:
drivers/usb/hub.c:892 hub_get_status()
  error: potential null dereference 'hub->dev'

This means:
└── hub->dev might be NULL at this point
└── sparse did not catch it (no annotation)
└── smatch traced the execution path and found
    a code path where hub->dev can be NULL
└── Add a null check or prove it cannot be NULL
    (if it cannot be NULL, add a WARN_ON or comment)
```

**Use after free:**

```
smatch output:
drivers/usb/hub.c:445 hub_disconnect()
  error: dereferencing freed memory 'hub'

This means:
└── hub was freed on an earlier code path
└── The code continues to dereference it
└── Classic use-after-free bug
└── Fix the lifetime management
```

**Resource leaks:**

```
smatch output:
drivers/usb/hub.c:234 usb_hub_probe()
  error: 'hub' was allocated on line 198
  but not freed on line 234 error path

This means:
└── hub = kzalloc() on success path
└── On error path at line 234, hub is not freed
└── Memory leak — every failed probe leaks memory
└── Add kfree(hub) on the error path
    Or better: use devm_kzalloc() for automatic cleanup
```

---

## Building — What to Verify

### The Zero New Warnings Rule

```
The rule is simple:
Your patch must not introduce any new compiler warnings.

How to verify:

Step 1 — Record warnings before your patch:
git stash
make -j$(nproc) 2>&1 | grep "warning:" > before.txt
git stash pop

Step 2 — Record warnings with your patch:
make -j$(nproc) 2>&1 | grep "warning:" > after.txt

Step 3 — Compare:
diff before.txt after.txt

Any lines in after.txt that are not in before.txt
are new warnings your patch introduced. Fix them.
```

### Build Configurations to Test

Your patch may compile fine with your current config but break with a different config. Test at minimum:

```
Your current config (always):
make -j$(nproc)

With W=1 (extra warnings):
make -j$(nproc) W=1 drivers/usb/core/

This enables additional GCC warnings that are
disabled by default. Your patch should pass W=1
in the files you modified.

With ARCH=arm64 (cross-compilation):
sudo apt-get install gcc-aarch64-linux-gnu
make ARCH=arm64 \
  CROSS_COMPILE=aarch64-linux-gnu- \
  -j$(nproc) drivers/usb/core/

Catches x86-specific assumptions in your code.
Common source of bugs in drivers.

With allmodconfig (everything enabled):
make allmodconfig
make -j$(nproc) 2>&1 | grep "warning:"

This enables every config option.
Catches code that only compiles with specific options.
Takes longer but catches real issues.
```

---

## Testing — What Is the Minimum Bar

### By Type of Change

```
Bug fix:
Minimum:
├── The bug is reproducible before your patch
├── The bug is NOT reproducible after your patch
└── Basic functionality still works after the patch

Ideal:
└── Automated test case added (if the subsystem
    has a test framework)
└── Tested on multiple hardware configurations
└── Tested under stress conditions

Driver fix:
Minimum:
├── The driver loads without errors (dmesg clean)
├── The hardware it targets works correctly
└── No regressions in basic device operations

New driver:
Minimum:
├── Driver loads and unloads cleanly (modprobe/rmmod)
├── Hardware works for its primary function
├── No kernel warnings or oops during operation
└── Tested with CONFIG_DEBUG_KERNEL=y

Cleanup / style fix:
Minimum:
└── Kernel compiles
└── No functional change (verify with diff of .o files
    before and after)

For no-functional-change patches:
objdump -d file.o > before.txt
# apply patch
objdump -d file.o > after.txt
diff before.txt after.txt
# Should be empty or only line number changes
```

### How to Document What You Tested

This goes in the commit message body:

```
Sufficient testing documentation:

For a bug fix:
"Tested on x86-64 with Linux 6.8-rc4. Reproduced
the null pointer dereference with the original code
by hot-unplugging a USB hub under load. Confirmed
the oops does not occur with this patch applied."

For a driver patch:
"Tested with a Realtek RTL8153 USB Ethernet adapter
on x86-64. Device enumerates correctly, link comes
up, sustained 900 Mbps transfer without errors.
Driver unloads cleanly."

For a cleanup:
"Compile-tested only. No functional change."
(this is acceptable for pure style cleanups)

For a new driver:
"Tested on Raspberry Pi 4 (ARM64) with the XYZ sensor
connected via I2C. Driver loads, device enumerates,
readings are accurate. No warnings in dmesg under
continuous operation for 24 hours."
```

---

## The Complete Pre-Send Checklist

Print this. Run through it for every patch.

```
CODE QUALITY:
[ ] checkpatch.pl --strict: zero ERRORs
[ ] checkpatch.pl --strict: all WARNINGs addressed
    (fixed or documented with clear justification)
[ ] sparse C=1: zero new warnings vs clean tree
[ ] smatch: checked files you modified
[ ] No new compiler warnings (compared with before.txt)
[ ] W=1 build passes on files you modified

BUILD VERIFICATION:
[ ] Full kernel build completes successfully
[ ] Cross-compilation tested (arm64 at minimum)
[ ] allmodconfig build passes (if changing core code)
[ ] Module loads and unloads cleanly (if driver change)

TESTING:
[ ] Bug is reproduced before patch
[ ] Bug is not present after patch
[ ] Basic functionality verified after patch
[ ] Testing documented in commit message body

COMMIT MESSAGE:
[ ] Subject: subsystem: lowercase description
[ ] Subject under 72 characters
[ ] Blank line between subject and body
[ ] Body explains what problem and why this fix
[ ] Body documents what was tested
[ ] Fixes: tag if fixing a regression
[ ] Link: tag if there is a bug report
[ ] Cc: stable@vger.kernel.org if appropriate
[ ] Signed-off-by: with real name and correct email

PATCH FILE:
[ ] git format-patch produces clean output
[ ] checkpatch.pl on .patch file: zero errors
[ ] Diffstat shows only files you intended to change
[ ] No whitespace corruption in the diff
    (tabs are tabs, not spaces)

RECIPIENTS:
[ ] get_maintainer.pl run on the patch
[ ] All (maintainer) entries in To:
[ ] All mailing lists in Cc:
[ ] linux-kernel@vger.kernel.org in Cc:
[ ] git send-email --dry-run reviewed

READY:
[ ] You can explain every line of the diff
[ ] You are confident this is correct
[ ] Send it
```

---

> **Everything checks out. Your patch is ready to go. Next: how to actually send it — git send-email setup, the exact commands, cover letters for patch series, and how v2 and v3 work.**
> **→ [13-sending-the-patch](../13-sending-the-patch/README.md)**
