# Finding the Right Maintainer

---

> Your patch is written, checked, and ready. Now it needs to reach the right person. Send it to the wrong maintainer and it gets ignored. Send it to the wrong mailing list and nobody relevant sees it. Send it directly to Linus and you will either get no response or a blunt redirect. The kernel has a precise routing system — the MAINTAINERS file and the script that reads it. Use them every time, without exception.

**This file shows you exactly how to find the right maintainer and mailing list for any patch you write — with real output examples, edge cases, and what to do when the answer is not obvious.**

---

## What's Inside This File

```
11-finding-the-right-maintainer/
│
├── The MAINTAINERS File — what it is and how it works
├── get_maintainer.pl — the script that does the work
│   ├── Basic usage
│   ├── Reading the output
│   └── Real examples with actual output
├── To: vs Cc: — who gets what
├── Edge Cases — when the answer is not obvious
│   ├── Multiple subsystems in one patch
│   ├── New files with no MAINTAINERS entry
│   ├── Orphaned subsystems
│   └── When get_maintainer.pl returns nothing useful
├── What Happens if You Send to the Wrong Place
└── Double-Checking Before You Send
```

---

## The MAINTAINERS File — What It Is

The MAINTAINERS file lives at the root of the kernel tree. It is the single source of truth for who maintains what. At over 25,000 lines, it covers every subsystem, every driver category, every architecture, and many individual drivers.

You should never need to read it manually. But understanding its structure helps you understand what `get_maintainer.pl` is doing and why it gives you the output it does.

```
Structure of a MAINTAINERS entry:

USB SUBSYSTEM
M: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
R: Alan Stern <stern@rowland.harvard.edu>
L: linux-usb@vger.kernel.org
S: Maintained
W: http://www.linux-usb.org
T: git git://git.kernel.org/.../gregkh/usb.git
T: git git://git.kernel.org/.../gregkh/usb-next.git
F: Documentation/usb/
F: drivers/usb/
F: include/linux/usb.h
F: include/linux/usb/

Field reference:
M: Maintainer     — primary contact, send patches here
R: Reviewer       — CC this person, not primary
L: List           — mailing list, always CC
S: Status         — Maintained / Odd Fixes / Orphan
W: Website        — project page
B: Bug tracker    — where bugs are filed
C: Patchwork      — patchwork project URL
T: Tree           — git tree where patches land
F: Files          — file patterns this entry covers
X: Excluded files — exceptions to F: patterns
N: Name pattern   — matches filenames by regex
K: Keyword        — matches commit messages by regex
```

The `F:` fields are what `get_maintainer.pl` matches against your changed files. It reads your diff, extracts every changed file path, matches those paths against all `F:` patterns in the MAINTAINERS file, and returns the corresponding `M:` and `L:` values.

---

## get_maintainer.pl — The Script That Does the Work

`get_maintainer.pl` is a Perl script located at `scripts/get_maintainer.pl` in the kernel tree. Run it on every patch before sending. No exceptions.

### Basic Usage

```
Three ways to run it:

1. On a specific file you changed:
./scripts/get_maintainer.pl --file drivers/usb/core/hub.c

2. On a patch file from git format-patch:
./scripts/get_maintainer.pl 0001-usb-fix-null-ptr.patch

3. On a diff from git diff:
git diff HEAD~1 | ./scripts/get_maintainer.pl

The third form is most useful during development
before you have committed — pipe your working diff
directly into the script.
```

### Reading the Output

The script output tells you exactly who to put in `To:` and `Cc:`.

```
Example — running on drivers/usb/core/hub.c:

$ ./scripts/get_maintainer.pl --file drivers/usb/core/hub.c

Greg Kroah-Hartman <gregkh@linuxfoundation.org> (maintainer)
Alan Stern <stern@rowland.harvard.edu> (reviewer)
linux-usb@vger.kernel.org (open list)
linux-kernel@vger.kernel.org (open list)

Output fields explained:
(maintainer)  — put in To:
(supporter)   — put in Cc:
(reviewer)    — put in Cc:
(open list)   — put in Cc:
(moderated list) — put in Cc: (posts may be held for approval)
```

How to use this output in your send-email command:

```
git send-email \
  --to="gregkh@linuxfoundation.org" \
  --cc="stern@rowland.harvard.edu" \
  --cc="linux-usb@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  0001-usb-fix-null-pointer-in-hub.patch
```

Or let git send-email read the patch headers:

```
# Add recipients to the patch file itself using --to and --cc
# git send-email will pick them up automatically

git send-email \
  --annotate \
  0001-usb-fix-null-pointer-in-hub.patch
```

### Real Examples With Actual Output

**Example 1 — USB driver patch:**

```
$ ./scripts/get_maintainer.pl --file drivers/usb/serial/ch341.c

Johan Hovold <johan@kernel.org> (maintainer)
Greg Kroah-Hartman <gregkh@linuxfoundation.org> (maintainer)
linux-usb@vger.kernel.org (open list)
linux-kernel@vger.kernel.org (open list)

Action:
To:  Johan Hovold (primary maintainer for USB serial)
Cc:  Greg Kroah-Hartman (USB subsystem maintainer)
Cc:  linux-usb@vger.kernel.org
Cc:  linux-kernel@vger.kernel.org
```

**Example 2 — Networking driver patch:**

```
$ ./scripts/get_maintainer.pl \
  --file drivers/net/ethernet/intel/e1000e/netdev.c

Jesse Brandeburg <jesse.brandeburg@intel.com> (maintainer)
Tony Nguyen <anthony.l.nguyen@intel.com> (maintainer)
intel-wired-lan@lists.osuosl.org (open list)
netdev@vger.kernel.org (open list)
linux-kernel@vger.kernel.org (open list)

Action:
To:  Jesse Brandeburg and Tony Nguyen
Cc:  intel-wired-lan@lists.osuosl.org
Cc:  netdev@vger.kernel.org
Cc:  linux-kernel@vger.kernel.org

Note: Two maintainers — both go in To:
```

**Example 3 — Memory management patch:**

```
$ ./scripts/get_maintainer.pl --file mm/slab.c

Christoph Lameter <cl@linux.com> (maintainer)
Pekka Enberg <penberg@kernel.org> (maintainer)
David Rientjes <rientjes@google.com> (maintainer)
Joonsoo Kim <iamjoonsoo.kim@lge.com> (maintainer)
Andrew Morton <akpm@linux-foundation.org> (maintainer)
Roman Gushchin <roman.gushchin@linux.dev> (maintainer)
Hyeonggon Yoo <42.hyeyoo@gmail.com> (maintainer)
linux-mm@kvack.org (open list)
linux-kernel@vger.kernel.org (open list)

Action:
To:  All maintainers (yes, all of them)
Cc:  linux-mm@kvack.org
Cc:  linux-kernel@vger.kernel.org

Note: mm/ has many co-maintainers — this is normal
Note: linux-mm uses kvack.org server, not vger
```

**Example 4 — ARM64 architecture patch:**

```
$ ./scripts/get_maintainer.pl --file arch/arm64/kernel/signal.c

Catalin Marinas <catalin.marinas@arm.com> (maintainer)
Will Deacon <will@kernel.org> (maintainer)
linux-arm-kernel@lists.infradead.org (moderated list)
linux-kernel@vger.kernel.org (open list)

Action:
To:  Catalin Marinas and Will Deacon
Cc:  linux-arm-kernel@lists.infradead.org (moderated)
Cc:  linux-kernel@vger.kernel.org

Note: linux-arm-kernel is moderated — your first post
      may be held for approval. This is normal.
      It will appear after a moderator approves it.
```

**Example 5 — staging driver patch:**

```
$ ./scripts/get_maintainer.pl \
  --file drivers/staging/vt6655/device_main.c

Greg Kroah-Hartman <gregkh@linuxfoundation.org> (maintainer)
linux-staging@lists.linux.dev (open list)
linux-kernel@vger.kernel.org (open list)

Action:
To:  Greg Kroah-Hartman
Cc:  linux-staging@lists.linux.dev
Cc:  linux-kernel@vger.kernel.org
```

---

## To: vs Cc: — Who Gets What

The distinction matters. Kernel developers filter their email heavily. The wrong field means your patch ends up in someone's low-priority folder or gets missed entirely.

```
To: field — direct recipients:
├── The primary maintainer(s) for the subsystem
├── Anyone marked (maintainer) in get_maintainer output
└── Co-maintainers if there are multiple (all in To:)

Cc: field — copied recipients:
├── Mailing lists (open list, moderated list)
├── Anyone marked (reviewer) or (supporter)
├── linux-kernel@vger.kernel.org (always)
├── The person who reported the bug
│   └── If you are fixing something they reported
│   └── Format: Reported-by: Name <email@example.com>
│       in the commit message, and CC them on the email
└── stable@vger.kernel.org if applicable
    (though this goes in the commit message as
    Cc: stable@vger.kernel.org, not in the email Cc:)

Who never goes in To: unless get_maintainer says so:
└── Linus Torvalds — almost never for regular patches
└── Random people you found in the git log
    (use get_maintainer.pl, not git log --follow)
└── Everyone on the mailing list —
    the list handles distribution
```

---

## Edge Cases — When the Answer Is Not Obvious

### Patch Touches Multiple Subsystems

```
Situation: Your patch modifies files in two different
subsystems — for example, net/ and crypto/.

What to do:
└── Run get_maintainer.pl on ALL changed files
    and combine the results

git diff HEAD~1 | ./scripts/get_maintainer.pl

This reads your entire diff and returns maintainers
for ALL changed files in one pass.

If maintainers are from different subsystems:
├── Both maintainer lists go in To:
├── All mailing lists go in Cc:
└── Consider: should this be two separate patches?
    If the changes are logically independent — split them
    If they are tightly coupled — keep them together
    but explain in the cover letter why they are combined

If subsystem maintainers disagree about the approach:
└── This escalates to a cross-subsystem discussion
└── Both maintainers need to agree before it merges
└── Be patient — this takes longer
```

### New Files With No MAINTAINERS Entry

```
Situation: You are adding a new driver.
The file does not exist yet.
get_maintainer.pl has nothing to match against.

What to do:

Step 1: Identify which directory it belongs in
        drivers/usb/   → USB maintainer
        drivers/net/   → Networking maintainer
        drivers/input/ → Input maintainer
        etc.

Step 2: Run get_maintainer on the parent directory:
./scripts/get_maintainer.pl --file drivers/usb/

Step 3: Add a MAINTAINERS entry for your new driver
        as part of your patch series:

        MY NEW DRIVER
        M: Your Name <your@email.com>
        L: linux-usb@vger.kernel.org
        S: Maintained
        F: drivers/usb/mydriver/

This is expected for new drivers.
The MAINTAINERS addition is usually the last patch
in a series, or included with the driver patch.
```

### Orphaned Subsystems

```
Situation: get_maintainer.pl returns no maintainer,
only mailing lists. MAINTAINERS shows:
S: Orphan

What this means:
└── No one is actively maintaining this subsystem
└── Patches still go to the mailing list
└── Someone from the broader community will review
└── May take longer to get a response

What to do:
├── Send to the mailing list
├── CC linux-kernel@vger.kernel.org
└── Be patient — reviews may come from anyone
    with knowledge of that code area

If you want to become the maintainer:
└── Submit patches, fix things, build familiarity
└── After sustained contributions, email the list:
    "Is anyone maintaining X? I've been working on it
    and would like to take over maintenance."
└── The community will generally welcome this
```

### When get_maintainer.pl Returns Too Many People

```
Situation: Script returns 15 names for a simple patch.

This happens with:
├── Core files touched by many subsystems
├── Header files included everywhere
└── Files with many historical contributors

What to do:
└── Prioritise (maintainer) over (reviewer)
└── Always include ALL mailing lists
└── For people: include all (maintainer) entries
    You can trim (reviewer) entries if there are many
└── When in doubt, include everyone
    It is better to over-CC than under-CC

The maintainer will not be annoyed by being CCed.
They WILL be annoyed if they should have been CCed
and were not.
```

### get_maintainer.pl Returns Nothing Useful

```
Situation: Script returns only:
linux-kernel@vger.kernel.org (open list)

This means the file is not covered by any specific
MAINTAINERS entry.

What to do:

1. Check git log for recent contributors:
git log --oneline -20 -- path/to/file.c

2. Check who wrote the code you are modifying:
git blame path/to/file.c | head -30
Look for email addresses in the commit hashes

3. Search lore.kernel.org:
Search for the file name to find past patch discussions
Look at who reviewed those patches

4. Ask on the closest relevant mailing list:
"I have a patch for X, who should I send this to?"
This is acceptable — the community will redirect you
```

---

## What Happens if You Send to the Wrong Place

Understanding the consequences helps you understand why getting this right matters.

```
Scenario 1: Sent to right maintainer, wrong list
├── Maintainer sees it
├── May apply it anyway
└── May ask you to resend with the correct list CCed
    so the community can review

Scenario 2: Sent to right list, wrong maintainer
├── Someone on the list may redirect you
│   "This should go to X instead"
├── The right maintainer may not see it
└── Resend with corrected recipients

Scenario 3: Sent only to LKML, not the subsystem list
├── The relevant maintainer may miss it entirely
├── LKML volume is high — things get lost
└── Resend to the correct subsystem list
    Reference your original email:
    "Resending to the correct list. Original:
    https://lore.kernel.org/lkml/your-message-id"

Scenario 4: Sent directly to Linus
├── Linus may ignore it (he trusts his maintainers)
├── Linus may redirect: "Send this to Greg"
└── Does not help your patch — adds delay
    Use get_maintainer.pl to avoid this

Scenario 5: Sent to a list that requires subscription
├── Your email is held for moderation
├── Wait — it will appear after a moderator approves
└── Do not resend — it will appear as a duplicate
    when the original is approved

The general rule:
└── If you realise you sent to the wrong place,
    wait 48 hours before resending
└── The original may appear, or someone may redirect you
└── Add a note: "Resending to correct list."
```

---

## Double-Checking Before You Send

Run through this before executing `git send-email`:

```
Pre-send routing checklist:

[ ] Ran get_maintainer.pl on the patch file or diff
    NOT on just one file if patch touches multiple

[ ] All (maintainer) entries are in To:

[ ] All mailing lists are in Cc:

[ ] linux-kernel@vger.kernel.org is in Cc:
    (always, for every patch)

[ ] If fixing a reported bug:
    Original reporter is in Cc:
    Reported-by: tag is in commit message

[ ] If the patch was requested by someone:
    They are in Cc:

[ ] Linus is NOT in To: or Cc:
    (unless get_maintainer.pl explicitly listed him)

[ ] No random people from git log
    (only people get_maintainer.pl returned)

[ ] Subject line starts with [PATCH] or [PATCH v2]
    (git send-email adds this automatically)

[ ] Dry run first:
    git send-email --dry-run *.patch
    Review the output — correct recipients?
    Correct subject? Correct threading?
```

### The Dry Run — Always Do This

```
git send-email --dry-run \
  --to="maintainer@kernel.org" \
  --cc="linux-subsystem@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  0001-your-patch.patch

Dry run output shows:
From: Your Name <your@email.com>
To: maintainer@kernel.org
Cc: linux-subsystem@vger.kernel.org
Cc: linux-kernel@vger.kernel.org
Subject: [PATCH] subsystem: fix the thing
Date: ...

Read this carefully:
├── Is your From: correct? (your real name + email)
├── Is the To: the maintainer, not a list?
├── Are all the right lists in Cc:?
└── Does the Subject look right?

Only run without --dry-run when everything looks correct.
```

---

## Quick Reference — Common Subsystem Routing

```
You changed...              Send to...

drivers/usb/               gregkh@linuxfoundation.org
                           linux-usb@vger.kernel.org

drivers/staging/           gregkh@linuxfoundation.org
                           linux-staging@lists.linux.dev

net/ or drivers/net/       kuba@kernel.org
                           netdev@vger.kernel.org

mm/                        akpm@linux-foundation.org
                           linux-mm@kvack.org

drivers/input/             dmitry.torokhov@gmail.com
                           linux-input@vger.kernel.org

sound/ or drivers/sound/   tiwai@suse.de
                           alsa-devel@alsa-project.org

security/                  linux-security-module@vger.kernel.org

drivers/acpi/ or pm/       rafael@kernel.org
                           linux-acpi@vger.kernel.org
                           linux-pm@vger.kernel.org

arch/arm/                  linux@armlinux.org.uk
                           linux-arm-kernel@lists.infradead.org

arch/arm64/                catalin.marinas@arm.com
                           will@kernel.org
                           linux-arm-kernel@lists.infradead.org

arch/x86/                  mingo@redhat.com
                           tglx@linutronix.de
                           linux-kernel@vger.kernel.org

drivers/gpu/ (DRM)         dri-devel@lists.freedesktop.org

Always CC:                 linux-kernel@vger.kernel.org

This table is a reference — ALWAYS verify with
get_maintainer.pl. Maintainers change. This table
may be out of date. The script never is.
```

---

> **You know where to send your patch. Next: the final checks before it goes out — the pre-send checklist that catches what everything else missed.**
> **→ [12-before-you-send](../12-before-you-send/README.md)**
