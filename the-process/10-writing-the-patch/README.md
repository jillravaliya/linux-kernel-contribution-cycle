# Writing the Patch

---

> You found something to fix. You made the change in your editor. Now what? A kernel patch is not just a diff — it is a commit with a specific message format, a certification line, tags that carry legal meaning, and a diff that must be formatted precisely. Get any of these wrong and the patch gets rejected before the code is even read. Get them right and the reviewer can focus entirely on whether the code itself is correct.

**This file covers every part of a kernel patch — from the first line of the commit message to the last line of the diff — with real examples of what good and bad look like.**

---

## What's Inside This File

```
10-writing-the-patch/
│
├── The Anatomy of a Kernel Patch
├── The Commit Message — every field explained
│   ├── The subject line — format and rules
│   ├── The body — what to write and what not to
│   ├── The Fixes: tag
│   ├── The Link: tag
│   ├── The Cc: stable tag
│   └── The Signed-off-by line — what it means legally
├── Review Tags — what they mean
│   ├── Reviewed-by
│   ├── Acked-by
│   └── Tested-by
├── git format-patch — what it produces
├── Good Patch vs Bad Patch — real examples
└── Common Patch Mistakes
```

---

## The Anatomy of a Kernel Patch

A kernel patch has two distinct parts — the commit message and the diff. Both are reviewed. Both matter.

```
Complete patch structure:

+------------------------------------------+
|  COMMIT MESSAGE                          |
|                                          |
|  Subject line                            |
|  (blank line)                            |
|  Body — explanation of what and why      |
|  (blank line)                            |
|  Tags — Fixes:, Link:, Cc: stable       |
|  Signed-off-by: Your Name <email>        |
+------------------------------------------+
|  ---  (three dashes — separator)         |
+------------------------------------------+
|  Diffstat (files changed summary)        |
+------------------------------------------+
|  DIFF                                    |
|                                          |
|  The actual code change                  |
+------------------------------------------+

Everything above --- is the commit message.
Everything below --- is the diff metadata.
The --- line and below does NOT go into git history.
Only the commit message above --- is permanent.
```

---

## The Commit Message — Every Field Explained

The commit message is the most important part of your patch. Code can be changed. A commit message is permanent — it lives in the git history of every kernel installation forever. Maintainers know this and they review commit messages carefully.

### The Subject Line

```
Format:
subsystem: short description of what the patch does

Rules:
├── 72 characters maximum (including subsystem prefix)
├── subsystem: comes from the affected directory/component
├── No capital letter after the colon
│   └── Wrong: "usb: Fix null pointer dereference"
│   └── Right: "usb: fix null pointer dereference"
├── No period at the end
├── Present tense, imperative mood
│   └── Wrong: "fixed the bug" / "fixes the bug"
│   └── Right: "fix the bug"
├── Describe what the patch DOES, not what the bug IS
│   └── Wrong: "usb: null pointer dereference in hub"
│   └── Right: "usb: fix null pointer dereference in hub"
└── No "PATCH" in the subject — git send-email adds [PATCH]
```

Subsystem prefix examples:

```
Change in drivers/usb/core/hub.c:
  usb: fix null pointer in hub_disconnect

Change in net/ipv4/tcp.c:
  tcp: fix off-by-one in receive buffer calculation

Change in drivers/staging/vt6655/device_main.c:
  staging: vt6655: remove trailing whitespace

Change in mm/slab.c:
  mm/slab: fix memory leak on error path

Change in arch/x86/kernel/cpu/bugs.c:
  x86/bugs: update Spectre v2 mitigation documentation

Change in fs/ext4/inode.c:
  ext4: fix extent corruption on punch hole

Multiple subsystems (use the primary one):
  usb: fix race condition in USB hub driver
  (even if it also touches drivers/base/)
```

### The Body

The body explains what problem exists and why this patch fixes it. This is not a description of the diff — a maintainer can read the diff. The body explains what is not obvious from the diff.

```
What the body MUST answer:

1. What is the problem?
   └── What can go wrong? Under what conditions?
   └── What does the user or system experience?

2. Why does the problem exist?
   └── What is the root cause?

3. How does this patch fix it?
   └── What approach was chosen and why?
   └── Why not the alternative approach?

4. How was it tested?
   └── What hardware? What kernel version?
   └── What test confirmed the fix works?
```

Real example — good commit message body:

```
The hub driver may call hub_disconnect() while a
concurrent hub_event() is still running. When the
disconnect path frees hub->dev and hub_event() then
dereferences it to log an error message, a use-after-
free occurs.

This is reproducible by hot-unplugging a USB hub while
heavy I/O is in progress on a connected device. The
kernel oops shows a null pointer dereference at the
dev_err() call inside hub_event().

Fix this by taking hub->kref before entering the event
handler and releasing it on exit, ensuring the hub
structure is not freed while the event handler runs.
Tested on x86-64 with a 4-port USB 3.0 hub running
stress-ng against connected storage.
```

What makes this good:
- States the race condition clearly
- Explains when it happens (hot-unplug under load)
- Explains the symptom (kernel oops with null deref)
- Explains the fix approach (kref)
- States how it was tested

Bad commit message body:

```
Fix a bug in the USB hub driver.

This patch fixes the issue by adding a null check.
```

What makes this bad:
- Does not explain what the bug is
- Does not explain when it happens
- Does not explain why a null check fixes it
- No testing information
- A reviewer learns nothing from reading this

### Body Formatting Rules

```
Formatting:
├── Wrap lines at 75 characters
├── One blank line between the subject and body
├── One blank line between paragraphs
├── Do not use bullet points or numbered lists
│   └── Plain prose paragraphs only
├── Write in English
│   └── Native English not required
│   └── Clear and correct is more important than fluent
└── Do not use first person ("I fixed...")
    └── Use neutral description ("Fix the...")
    └── Exception: "Reported-by" tags imply first person
        context — that is fine
```

### The Fixes: Tag

If your patch fixes a regression — a bug introduced by a specific earlier commit — add a Fixes: tag.

```
Format:
Fixes: <12 characters of commit hash> ("<commit subject>")

Example:
Fixes: 3f8a2d1b9e4c ("usb: hub: refactor port reset logic")

How to generate it correctly:
git log --abbrev=12 --pretty=format:"%h (\"%s\")" \
  -1 <full-commit-hash>

Why it matters:
├── Tells the stable team exactly which commit broke things
├── Allows automated tools to identify regression fixes
├── Helps bisect investigations
└── The stable team uses it to decide backport priority

When to use Fixes:
└── Only when you know the SPECIFIC commit that
    introduced the bug
└── Do not guess — if you are not sure, leave it out
└── A wrong Fixes: tag is worse than no Fixes: tag
```

### The Link: Tag

Link to relevant external resources — bug reports, mailing list discussions, syzkaller reports.

```
Format:
Link: https://url-to-relevant-resource

Examples:
Link: https://lore.kernel.org/r/message-id@kernel.org
Link: https://syzkaller.appspot.com/bug?id=XXXXXXXX
Link: https://bugzilla.kernel.org/show_bug.cgi?id=XXXXX

When to use it:
├── When you are fixing a reported bug
│   └── Link to the bug report
├── When your patch was discussed on a mailing list
│   └── Link to the discussion thread
├── When there is a syzkaller report
│   └── Link to the syzkaller dashboard entry
└── When there is relevant external documentation

Multiple Link: tags are allowed:
Link: https://first-resource
Link: https://second-resource
```

### The Cc: stable Tag

If your patch fixes a bug that affects users on stable kernels — not just the latest mainline — add the stable tag.

```
Format:
Cc: stable@vger.kernel.org

Place it with the other tags, before Signed-off-by:

Fixes: 3f8a2d1b9e4c ("usb: hub: refactor port reset logic")
Cc: stable@vger.kernel.org
Signed-off-by: Your Name <your@email.com>

When to add stable tag:
├── Bug causes data loss, corruption, or security issue
├── Bug causes kernel crash or oops
├── Bug causes hardware malfunction
└── Bug is a regression from a recent release

When NOT to add stable tag:
├── New features (never go to stable)
├── Cleanup or style fixes
├── Performance improvements
└── Theoretical bugs that have never been observed

Stable version targeting:
If the bug only affects specific versions, specify:
Cc: stable@vger.kernel.org # 6.1+

The stable team will evaluate whether to include it.
Adding the tag is a request, not a guarantee.
```

### The Signed-off-by Line

Every kernel patch must have a Signed-off-by line. Without it, the patch will be rejected immediately.

```
Format:
Signed-off-by: Your Real Name <your@email.com>

What it means:
└── By adding this line, you certify under the
    Developer Certificate of Origin (DCO) that:

    (a) The contribution was created in whole or in
        part by you, and you have the right to submit
        it under the open source license indicated; OR

    (b) The contribution is based upon previous work
        that is covered under an appropriate open source
        license, and you have the right to submit
        modifications of that work; OR

    (c) The contribution was provided to you by someone
        who certified (a) or (b), and you have not
        modified it; OR

    (d) You understand and agree that this project and
        the contribution are public and that a record of
        the contribution (including your identity) is
        maintained indefinitely.

In plain language:
└── You are certifying this code is yours to give
└── You understand it is permanently public
└── Your real name must be used — not a pseudonym
└── The email must match your git config user.email

Multiple Signed-off-by lines:
└── If someone else wrote the patch and sent it to you,
    their SOB appears first, yours appears last
└── This creates a chain of custody

From: Original Author <author@example.com>
...patch...
Signed-off-by: Original Author <author@example.com>
Signed-off-by: Maintainer Who Applied <maintainer@example.com>
```

---

## Review Tags — What They Mean

When other people review your patch and add tags, it signals different levels of endorsement. Understanding these is important both for reading others' patches and for knowing what a maintainer expects before merging yours.

### Reviewed-by

```
Reviewed-by: Reviewer Name <reviewer@example.com>

Meaning:
└── The reviewer read the patch carefully
└── They believe the approach is technically sound
└── They checked the logic, the error handling,
    the locking, the memory management
└── They are willing to stand behind it technically

Does NOT mean:
└── They tested it on hardware
└── They agree with the overall direction
└── It will definitely be merged

Weight:
└── A Reviewed-by from a known subsystem expert
    significantly increases merge likelihood
└── A maintainer may merge a patch with a strong
    Reviewed-by from a trusted reviewer without
    reviewing every line themselves
```

### Acked-by

```
Acked-by: Maintainer Name <maintainer@example.com>

Meaning:
└── The person acknowledges the patch is OK
└── Often used by maintainers of affected subsystems
    that do not own the primary subsystem
└── "I looked at this, I'm fine with it going in"

Example:
└── A networking patch touches a small piece of the
    crypto subsystem
└── The crypto maintainer adds Acked-by to indicate
    the crypto portion is acceptable
└── The networking maintainer merges based on their
    own review plus the crypto Acked-by

Does NOT mean:
└── Full code review was done
└── It was tested
```

### Tested-by

```
Tested-by: Tester Name <tester@example.com>

Meaning:
└── The person ran the patch on real hardware or
    a real workload
└── The bug is fixed or the feature works as described
└── Their specific setup is documented in their email

Extremely valuable for:
└── Hardware-specific driver patches
└── Race condition fixes (hard to test without the hardware)
└── Fixes for specific platform bugs

How to get Tested-by:
└── Post your patch with a clear description of
    what hardware it fixes
└── Developers with that hardware may test and respond
└── On netdev, there are automated testers that add
    Tested-by automatically for networking patches
```

---

## git format-patch — What It Produces

`git format-patch` takes your commits and turns them into patch files ready to send.

```
Basic usage:

# Single patch (last commit):
git format-patch -1

# Last 3 commits as 3 separate patches:
git format-patch -3

# Everything since origin/master:
git format-patch origin/master

# With a cover letter for a series:
git format-patch --cover-letter -3

Output files:
0001-subsystem-fix-the-thing.patch
0002-subsystem-fix-another-thing.patch
0003-subsystem-fix-third-thing.patch
0000-cover-letter.patch  (if --cover-letter used)
```

### What the Output Looks Like

```
Contents of 0001-usb-fix-null-pointer-in-hub.patch:

From 7a3f9c1b2d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8 Mon Sep 17 00:00:00 2001
From: Your Name <you@example.com>
Date: Mon, 10 Mar 2025 14:23:45 +0000
Subject: [PATCH] usb: fix null pointer dereference in hub_disconnect

The hub driver may call hub_disconnect() while hub_event()
is still running. If the hub structure is freed while
hub_event() dereferences it, a use-after-free occurs.

Fix this by taking a kref before entering hub_event()
and releasing it on exit.

Fixes: 3f8a2d1b9e4c ("usb: hub: refactor port reset logic")
Cc: stable@vger.kernel.org
Signed-off-by: Your Name <you@example.com>
---
 drivers/usb/core/hub.c | 8 ++++++++
 1 file changed, 8 insertions(+)

diff --git a/drivers/usb/core/hub.c b/drivers/usb/core/hub.c
index 3f8a2d1..7b4c9e2 100644
--- a/drivers/usb/core/hub.c
+++ b/drivers/usb/core/hub.c
@@ -1823,6 +1823,14 @@ static void hub_event(struct work_struct *work)
        struct usb_hub *hub =
                container_of(work, struct usb_hub, events);

+       if (!kref_get_unless_zero(&hub->kref))
+               return;
+
        /* process hub events */
        ...
+
+       kref_put(&hub->kref, hub_release);
 }
```

The file contains everything needed to:
1. Apply the patch with `git am`
2. Read the full context
3. Review the commit message and diff together

### Checking Your Patch File

```
Always run checkpatch on the patch file itself:
./scripts/checkpatch.pl --strict \
  0001-usb-fix-null-pointer-in-hub.patch

This catches:
├── Format issues in the commit message
├── Style issues in the diff
├── Missing Signed-off-by
└── Subject line problems
```

---

## Good Patch vs Bad Patch — Real Examples

### Example 1 — Bug Fix

```
GOOD:

Subject: usb: fix null pointer dereference on hub disconnect

The hub driver dereferences hub->dev in hub_event() after
hub_disconnect() may have already freed the hub structure.
This race is triggerable by disconnecting a hub while I/O
is active on a connected device.

Fix this by checking hub for NULL at the start of
hub_event() before any dereference.

Reported-by: Jane Smith <jane@example.com>
Fixes: 3f8a2d1b9e4c ("usb: hub: rework disconnect path")
Cc: stable@vger.kernel.org
Signed-off-by: Your Name <your@example.com>

Why this is good:
├── Subject explains what the fix does (not the bug)
├── Body explains the race condition clearly
├── Body explains when it happens
├── Fixes: tag identifies the regressing commit
├── Cc: stable because it is a crash bug
├── Reported-by credits the reporter
└── Signed-off-by is present
```

```
BAD:

Subject: Fix USB bug

Fixed a bug where the driver crashes. This was caused
by a null pointer. I added a null check to fix it.

Signed-off-by: Your Name <your@example.com>

Why this is bad:
├── Subject does not say what subsystem
├── Subject does not say what was fixed
├── "Fix USB bug" describes nothing specific
├── Body does not explain the root cause
├── "I added a null check" — use neutral language
├── No Fixes: tag (it is a regression fix)
├── No Cc: stable (it is a crash bug)
└── Reviewer learns nothing from this message
```

### Example 2 — Cleanup Patch

```
GOOD:

Subject: staging: vt6655: replace uint types with kernel types

The driver uses C99 uint8_t, uint16_t, uint32_t types
throughout. The kernel coding style prefers u8, u16, u32.
Replace all instances to bring the driver in line with
kernel coding conventions.

No functional change.

Signed-off-by: Your Name <your@example.com>

Why this is good:
├── Clear subject with staging prefix and driver name
├── Explains WHY the change is being made
│   (coding conventions, not personal preference)
├── "No functional change" — important for cleanups
│   Tells reviewer not to look for logic changes
└── No Fixes: tag — correct, this is not a bug fix
```

```
BAD:

Subject: clean up vt6655 driver

Changed uint types to u types because it looks better.

Signed-off-by: Your Name <your@example.com>

Why this is bad:
├── No "staging:" prefix
├── "looks better" is subjective and weak justification
├── Does not mention which types were changed
└── Does not say "No functional change"
```

---

## Common Patch Mistakes

```
Mistake 1: Subject line capitalised after colon
Wrong:  usb: Fix null pointer dereference
Right:  usb: fix null pointer dereference

Mistake 2: Period at end of subject
Wrong:  usb: fix null pointer dereference.
Right:  usb: fix null pointer dereference

Mistake 3: Past tense in subject
Wrong:  usb: fixed null pointer dereference
Right:  usb: fix null pointer dereference

Mistake 4: Describing the diff instead of the problem
Wrong:  "Added a null check before dereference"
Right:  "Fix null pointer dereference when hub
         disconnects during active I/O"

Mistake 5: Missing blank line between subject and body
Wrong:
  usb: fix null pointer
  The hub driver...

Right:
  usb: fix null pointer

  The hub driver...

Mistake 6: Mixing multiple unrelated changes
Wrong:  One patch that fixes a bug AND cleans up style
Right:  Two separate patches — one for each

Mistake 7: No testing information in the body
Add at minimum: what kernel version, what hardware,
what confirmed the fix works.

Mistake 8: Wrong Fixes: hash length
Wrong:  Fixes: 3f8 ("...")
Right:  Fixes: 3f8a2d1b9e4c ("...")
Use 12 characters minimum, 12 is the standard.

Mistake 9: Signed-off-by email does not match git config
Wrong:  git config email: work@company.com
        Signed-off-by: Your Name <personal@gmail.com>
Right:  Both must be the same email address.

Mistake 10: HTML in the patch
Sending from an email client that adds HTML formatting
breaks the patch completely.
Always use git send-email.
```

---

## The Complete Patch Writing Checklist

```
Before you run git format-patch:

Commit message:
[ ] Subject line: subsystem: lowercase description
[ ] Subject line under 72 characters
[ ] No period at end of subject
[ ] Blank line between subject and body
[ ] Body explains WHAT problem exists
[ ] Body explains WHY this is the right fix
[ ] Body explains HOW it was tested
[ ] Lines wrapped at 75 characters
[ ] Fixes: tag if fixing a specific regression
[ ] Link: tag if there is a bug report to reference
[ ] Cc: stable@vger.kernel.org if it is a crash/data
    loss/security bug that affects stable kernels
[ ] Signed-off-by with real name and correct email

Code:
[ ] checkpatch.pl --strict passes with zero ERRORs
[ ] All WARNINGs addressed or documented
[ ] sparse passes (no new warnings)
[ ] Kernel builds with zero new warnings
[ ] Tested — you verified the fix works
[ ] No unrelated changes mixed in

After git format-patch:
[ ] checkpatch.pl on the .patch file itself
[ ] Review the full patch file — read it as a reviewer
[ ] Diffstat looks right (correct files, sensible size)
```

---

> **Your patch is written and checked. Next: finding exactly who to send it to — the right maintainer, the right mailing list, the right CC list — so it lands in front of the right eyes.**
> **→ [11-finding-the-right-maintainer](../11-finding-the-right-maintainer/README.md)**
