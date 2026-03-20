# How a Change Moves Through the System

---

> You found a bug. You fixed it. You have a patch. Now what? Where does it go? Who reads it? How does it get from your laptop to the official kernel that ships on millions of machines? Why does it sometimes take three months? Why do some patches get ignored and others get merged in a week? What actually happens between "I submitted this" and "this is in the release"?

**You're about to find out — the complete lifecycle of a kernel change, every stage, every person, every decision point.**

---

## What's Inside This File

```
03-how-a-change-moves-through-the-system/
│
├── The Full Lifecycle — big picture first
├── Stage 1 — Writing and Preparing the Patch
├── Stage 2 — Sending to the Mailing List
├── Stage 3 — Community Review
├── Stage 4 — Subsystem Maintainer Decision
├── Stage 5 — Sitting in the Subsystem Tree
├── Stage 6 — The Merge Window
├── Stage 7 — The rc Cycle
├── Stage 8 — Final Release
├── Stage 9 — Stable Backport (if applicable)
├── What Can Go Wrong at Each Stage
└── Realistic Timelines — what to actually expect
```

---

## The Full Lifecycle — Big Picture First

Before going deep on each stage, here is the entire journey of a patch from idea to shipped release. Every section below is one part of this picture.

```
YOUR LAPTOP                MAILING LIST           SUBSYSTEM TREE
     |                          |                       |
  Write patch                   |                       |
  Test it locally               |                       |
  Run checkpatch.pl             |                       |
     |                          |                       |
     |----send patch----------->|                       |
     |                          |                       |
     |         Community reads, reviews, discusses      |
     |<---review comments-------|                       |
     |                          |                       |
     |----send v2 patch-------->|                       |
     |<---"looks good"----------|                       |
     |                          |                       |
     |              Maintainer picks it up              |
     |                          |--------add to tree--->|
     |                          |                       |
     |                   Sits in subsystem tree         |
     |                   Goes into linux-next           |
     |                   Gets tested                    |
     |                          |                       |
     |                  MERGE WINDOW OPENS              |
     |                          |                       |
     |                          |                  LINUS'S TREE
     |                          |                       |
     |                          |       Maintainer----->|
     |                          |       sends pull      |
     |                          |       request         |
     |                          |                       |
     |                          |       Linus merges    |
     |                          |       subsystem tree  |
     |                          |                       |
     |                  MERGE WINDOW CLOSES             |
     |                  rc1, rc2...rc7 testing          |
     |                          |                       |
     |                  FINAL RELEASE                   |
     |                  Your patch ships                |
     |                  to millions of machines         |
```

The full journey from first submission to shipped release is typically **2 to 6 months** depending on when in the cycle you submitted and how much review your patch needed. This is normal. Do not panic if your patch is not merged in a week.

---

## Stage 1 — Writing and Preparing the Patch

Everything starts on your laptop. Before a single line goes to the mailing list, several things need to happen.

### Finding the Right Base

You do not write your patch against an arbitrary version of the kernel. You write it against the correct tree for your type of change.

```
Which tree to base your patch on:

Bug fix for a current release:
└── Base on the stable tree for that release
    e.g. linux-6.8.y stable tree
    Maintainer can backport it easily

New feature or driver:
└── Base on the subsystem's -next tree
    e.g. net-next for networking features
    This is where new work goes

Not sure?
└── Base on linux-next
    It contains all subsystem trees merged together
    Your patch will be tested against future integration
```

Getting this wrong is one of the most common new contributor mistakes. A feature patch based on the stable tree will be rejected immediately — it cannot be applied cleanly to the -next tree where new work lands.

### Making the Change

You make your change, commit it with git. The commit message is not optional decoration — it is part of the patch and gets reviewed as carefully as the code itself.

```
Anatomy of a kernel commit message:

  subsystem: short description under 72 chars

  Longer explanation of what problem this fixes and why
  this approach was chosen. Write for a reviewer who
  has never seen this code before.

  Explain what the bug was, what the symptoms were,
  and how this patch addresses the root cause.
  Do not just describe what the code does — explain
  WHY the code does it.

  If this fixes a specific reported bug:
  Fixes: abcdef123456 ("previous commit that caused it")
  Link: https://lore.kernel.org/link-to-bug-report/

  Signed-off-by: Your Name <your@email.com>
```

The `Signed-off-by` line is the Developer Certificate of Origin (DCO). By adding it you are certifying that you have the right to submit this code under the kernel's license. It is required. Patches without it are rejected.

### Running the Checks

Before sending, you run the kernel's own checking tools:

```
Minimum checks before sending:

scripts/checkpatch.pl --strict -f changed_file.c
  └── Checks coding style, formatting, common mistakes
  └── Fix ALL warnings before sending
  └── "CHECK" level can sometimes be ignored
  └── "WARNING" and "ERROR" must be fixed

make C=1 (sparse static analysis)
  └── Catches type errors, lock imbalances,
      incorrect memory access patterns

Build the kernel with your change applied
  └── Must compile with zero new warnings
  └── Test with allmodconfig to catch config issues
```

Sending a patch that fails checkpatch.pl is one of the fastest ways to get dismissed by maintainers. It signals that you did not do basic preparation.

---

## Stage 2 — Sending to the Mailing List

The kernel has no pull request system. No GitHub. No web interface for submitting patches. Everything goes through email.

### Finding Who to Send To

The script `scripts/get_maintainer.pl` reads the MAINTAINERS file and tells you exactly who to send your patch to based on which files you changed.

```
Example: you changed drivers/usb/core/hub.c

$ ./scripts/get_maintainer.pl --file drivers/usb/core/hub.c

Greg Kroah-Hartman <gregkh@linuxfoundation.org> (maintainer)
Alan Stern <stern@rowland.harvard.edu> (maintainer)
linux-usb@vger.kernel.org (open list)
linux-kernel@vger.kernel.org (open list)
```

The output tells you:
- `To:` the maintainer(s)
- `Cc:` the mailing list(s)

### The Email Itself

Your patch email looks like this when sent correctly with `git send-email`:

```
From: Your Name <your@email.com>
To: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
Cc: linux-usb@vger.kernel.org, linux-kernel@vger.kernel.org
Subject: [PATCH] usb: hub: fix null pointer dereference on disconnect

The hub driver dereferences hub->dev before checking if hub
is NULL when a device disconnects during port reset. This
causes a kernel oops on hot-unplug under load.

Fix this by checking hub for NULL before dereferencing.

Fixes: 3f8a2d1b9e4c ("usb: hub: refactor port reset logic")
Cc: stable@vger.kernel.org
Signed-off-by: Your Name <your@email.com>
---
 drivers/usb/core/hub.c | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)

diff --git a/drivers/usb/core/hub.c b/drivers/usb/core/hub.c
index 7a3f9c1..2b84e5d 100644
--- a/drivers/usb/core/hub.c
+++ b/drivers/usb/core/hub.c
@@ -1823,6 +1823,9 @@ static void hub_disconnect(
        struct usb_hub *hub = usb_get_intfdata(intf);
        struct usb_device *hdev;

+       if (!hub)
+               return;
+
        hdev = hub->hdev;
```

The patch is the email. The diff is literally inline in the email body. This is how it has worked since the beginning and it still works today.

---

## Stage 3 — Community Review

Once your patch hits the mailing list, it is public. Anyone subscribed to the list can read it, test it, and respond. This is where the real work happens.

### Who Reviews

Reviews come from multiple sources:

```
Who might review your patch:

Subsystem maintainer:
└── The most important reviewer
└── Their Acked-by or Reviewed-by is usually needed
    before a patch gets merged

Other developers in the subsystem:
└── Developers who know this code well
└── May spot problems the maintainer missed
└── Their Reviewed-by carries weight

Automated bots:
└── kernel-bot@kernel.org (Intel 0-day bot)
    Automatically builds and tests your patch
    against many configurations
    Replies with build errors or warnings you missed

└── syzbot (Google's Syzkaller fuzzer)
    If your patch is in linux-next, syzbot may
    find kernel crashes triggered by your change

Other contributors:
└── Anyone who finds the patch interesting
└── Someone who has a device to test it on
└── Someone who ran into the same bug
```

### Types of Review Feedback

```
Feedback you will receive:

Reviewed-by: Name <email>
└── Reviewer read the code carefully
└── Believes the approach is correct
└── Does not imply testing

Tested-by: Name <email>
└── Someone tested the patch on real hardware
└── The bug is fixed or the feature works
└── Very valuable for driver patches

Acked-by: Name <email>
└── Maintainer of affected code agrees with approach
└── Not necessarily a full review
└── "I looked at this and I'm ok with it"

Nit comments:
└── "This variable name could be clearer"
└── "Missing blank line here"
└── Fix these and send v2

Substantive objections:
└── "This approach is wrong because..."
└── "There is a race condition here"
└── "This will break X when Y"
└── Must be addressed before patch can be merged

Request for more information:
└── "What hardware did you test this on?"
└── "Can you add a comment explaining why?"
└── "Can you split this into two patches?"
```

### How to Respond to Feedback

Responding correctly to review feedback is a skill. The kernel community has specific expectations:

```
How to respond to review comments:

1. Reply inline to each comment
   └── Quote the reviewer's comment
   └── Write your response immediately below it
   └── Never top-post (never put your reply above
       the quoted text)

2. For each substantive comment, either:
   └── Fix it in your next version, OR
   └── Explain clearly why you disagree
       Be polite but direct
       "I considered this but did not do it because..."

3. Send a new version (v2) with all fixes applied
   └── Subject: [PATCH v2] subsystem: description
   └── Add a changelog after the --- marker:
       ---
       Changes in v2:
       - Fixed null check as suggested by Greg
       - Added comment explaining the lock ordering
       - Split USB and HID changes into separate patches

4. Never:
   └── Argue aggressively with a maintainer
   └── Ignore feedback and resend unchanged
   └── Take criticism of your code personally
```

---

## Stage 4 — Subsystem Maintainer Decision

After review, the subsystem maintainer makes a decision. This is the most opaque stage for new contributors because it often happens without explicit communication.

### The Four Outcomes

```
What a maintainer can do with your patch:

1. Apply it to their tree
   └── Best outcome
   └── You may get no notification other than
       seeing it appear in their git tree
   └── Check: git log --oneline subsystem-tree | head

2. Ask for changes
   └── Explicit: "Please fix X and resend"
   └── Implicit: Review comments that clearly need
       addressing before it can be applied

3. Ignore it (for now)
   └── Does not mean rejection
   └── Maintainer may be busy
   └── Patch may be queued in their mental backlog
   └── Ping politely after 2-4 weeks of silence

4. Explicitly reject it
   └── "This is not the right approach"
   └── "We already fixed this differently"
   └── "This subsystem does not accept this type of change"
   └── Rare for well-prepared patches
   └── Ask what the correct approach would be
```

### Why Maintainers Sometimes Go Silent

Kernel maintainers are often overwhelmed. Greg Kroah-Hartman manages the stable tree alone. Some subsystem maintainers maintain their code on top of a full-time job. Silence is not rejection — it is usually a queue building up.

```
Realistic maintainer workload:

Greg Kroah-Hartman (stable + USB + driver core):
└── Receives hundreds of patch emails per week
└── Maintains multiple stable branches simultaneously
└── Each stable release requires reviewing ~50-200 patches
└── Also manages driver core for the entire kernel

Andrew Morton (mm):
└── Receives the most complex patches in the kernel
└── Memory management bugs can cause data loss
└── Every patch requires careful understanding
└── Some patches take months of back-and-forth

What this means for you:
└── 2 weeks silence: normal
└── 4 weeks silence: reasonable to send polite ping
└── 8 weeks silence: ping again, ask if more work needed
└── Never: "why haven't you merged my patch yet?"
```

---

## Stage 5 — Sitting in the Subsystem Tree

Once a maintainer applies your patch, it lives in their git tree. This is a staging area between the mailing list and Linus's mainline.

### The Two Trees Most Subsystems Maintain

```
Most subsystems have two trees:

fixes tree (e.g. net, usb, mm):
└── Bug fixes for the current release cycle
└── Gets sent to Linus quickly
└── May be pulled mid-cycle for serious fixes

next tree (e.g. net-next, usb-next, mm-next):
└── New features for the NEXT release
└── Accumulated during current cycle
└── Sent to Linus during the next merge window

Example for networking:
├── net.git     -- fixes, goes to Linus quickly
└── net-next.git -- new features, waits for merge window
```

Your patch lands in one of these depending on what it is. A bug fix goes into `net`. A new feature goes into `net-next`. If you sent a feature to the wrong tree, the maintainer will either move it or ask you to rebase.

### linux-next Integration

Every day, a bot maintained by Stephen Rothwell pulls all the subsystem -next trees together into `linux-next`. This is where integration problems are caught early.

```
What happens to your patch in linux-next:

Day 1:  Your patch enters subsystem tree
Day 2:  linux-next pulls subsystem tree
        Your patch is now in linux-next

Automated testing begins:
├── Intel 0-day bot: builds against 100+ configs
│   └── Finds build failures, sparse warnings
│   └── Emails you if it finds problems
│
├── Syzkaller: fuzzes the kernel
│   └── Finds crash bugs and memory corruption
│   └── Files bugs against the patches that caused them
│
└── Various distro CI systems test linux-next
```

If linux-next testing finds a problem with your patch, you need to fix it. The maintainer may ask you to resend, or may fix it themselves if it is trivial.

---

## Stage 6 — The Merge Window

When Linus releases a new kernel, the merge window opens immediately. For two weeks, subsystem maintainers send pull requests to Linus containing everything that has been building up in their -next trees.

```
What the merge window looks like from the outside:

Linus's inbox during merge window:

  From: Jakub Kicinski — "net-next pull request for 6.9"
  From: Greg KH        — "USB pull request for 6.9"
  From: Andrew Morton  — "mm patches for 6.9"
  From: Ingo Molnar    — "x86 pull request for 6.9"
  From: Will Deacon    — "arm64 pull request for 6.9"
  ... ~100 pull requests over 2 weeks

Linus reviews each pull request:
├── Reads the summary the maintainer wrote
├── Looks at the diffstat (what changed, how much)
├── Checks for anything unexpected or alarming
└── Runs: git pull <maintainer-tree-url>
```

Your patch, which has been sitting in the subsystem -next tree since it was accepted, is included in the pull request. When Linus pulls the tree, your patch enters mainline.

This is the moment your patch officially becomes part of the Linux kernel.

---

## Stage 7 — The rc Cycle

After the merge window closes, Linus releases rc1 and the stabilisation period begins. Your patch is already in mainline. But the release is not final yet.

```
What happens during rc cycle after your patch lands:

rc1 released:
└── Your patch is in the tree
└── Millions of testers and developers update
└── Some run it on unusual hardware configs

If your patch causes a regression:
├── Someone reports it on the mailing list
├── Bisect points to your commit
├── Maintainer or Linus may revert it
│   └── "Reverted due to regression on X hardware"
│   └── You need to fix the underlying problem
└── Better: you find the bug yourself and send a fix
    before anyone reports it

If your patch is fine:
└── It stays in the tree through all rc cycles
└── Ships in the final release
```

The rc cycle is where your patch is battle-tested. Most patches survive without incident. But if something breaks, the kernel community finds out quickly because many developers run rc kernels daily.

---

## Stage 8 — Final Release

When Linus decides the rc cycle has stabilised, he releases the final version. Your patch is now in an officially released kernel.

```
What final release means for your patch:

kernel.org tags the release:
└── git tag v6.9 at the specific commit

Tarballs appear on kernel.org:
└── linux-6.9.tar.gz downloadable by anyone

Distributions begin integration:
├── Fedora: usually ships mainline quickly
├── Ubuntu: evaluates for next LTS
├── Debian: testing branch gets it
└── Android: considers for next Android release

Your commit is now permanent:
└── In the git history forever
└── git log shows your name and email
└── Traceable via git blame to any line you changed
└── Part of the project's permanent record
```

---

## Stage 9 — Stable Backport

If you added `Cc: stable@vger.kernel.org` to your commit message, the stable team reviews your patch for backporting to older kernel versions.

```
Stable backport process:

Your patch lands in mainline 6.9
        |
        v
Stable team reviews it
├── Does it fix a real user-visible bug?
├── Is it under 100 lines?
├── Does it apply cleanly to older kernels?
└── Could it cause a regression in stable?
        |
        v
If accepted: backported to 6.8.x, 6.7.x, 6.6.x (LTS)
        |
        v
Appears in next stable point release:
└── 6.8.9, 6.7.14, 6.6.32 etc.
        |
        v
Ships to users running older kernels
└── Enterprise distros on LTS kernels get your fix
└── Android devices on older kernels may get your fix
└── Embedded systems running 5.15 may get your fix
```

A bug fix with `Cc: stable@vger.kernel.org` has far wider reach than the mainline version number suggests. It reaches every machine running any maintained kernel version.

---

## What Can Go Wrong at Each Stage

```
Stage 1 — Preparation:
├── Patch based on wrong tree
│   └── Maintainer cannot apply it cleanly
│   └── "Please rebase on net-next"
├── checkpatch.pl failures not fixed
│   └── Immediate criticism, fix and resend
└── Commit message too vague
    └── "What problem does this fix?"

Stage 2 — Sending:
├── Sent to wrong maintainer or list
│   └── May get redirected, may get ignored
│   └── Use get_maintainer.pl every time
├── Patch corrupted by email client
│   └── Tabs converted to spaces, lines wrapped
│   └── Use git send-email, never copy-paste
└── HTML email sent instead of plain text
    └── Immediate rejection, no exceptions

Stage 3 — Review:
├── Defensive response to feedback
│   └── "My code is correct, you don't understand"
│   └── Never do this. Ever.
├── Ignoring feedback and resending unchanged
│   └── Maintainer stops responding
└── Taking too long to send v2
    └── Patch gets forgotten
    └── Context is lost
    └── Send v2 within a week if possible

Stage 4 — Maintainer decision:
├── Patch sits unreviewed for weeks
│   └── Polite ping after 2-4 weeks
│   └── "Gentle ping — is there anything needed
│       to move this forward?"
└── Patch needs significant rework
    └── Ask clarifying questions before rewriting
    └── Make sure you understand what is wanted

Stage 5-6 — Trees and merge window:
├── linux-next reveals conflict with another patch
│   └── Coordinate with the other patch author
│   └── Or wait for their patch to land first
└── Build failure found by 0-day bot
    └── Fix it immediately, maintainer may ask for v2
```

---

## Realistic Timelines — What to Actually Expect

```
Type of change        Typical timeline to mainline

Simple cleanup        2-6 weeks
(checkpatch fix,      Fast review, low risk,
 typo, comment)       maintainer accepts quickly

Bug fix               3-8 weeks
                      Depends on severity and subsystem
                      Critical security fix: can be days
                      Minor fix: normal cycle timing

New driver            2-6 months
(staging first)       Goes to staging, gets cleaned up,
                      graduates to mainline over time

New feature           3-6 months
                      Multiple rounds of review typical
                      Architectural discussion may take weeks

Complex subsystem     6-18 months
change                Fundamental changes get heavy review
                      May require RFC (Request for Comments)
                      before proper patch series

Reality check:
└── Your first patch will probably take longer
    than you expect
└── This is normal
└── The process is designed for correctness,
    not speed
└── A patch that takes 3 months and is right
    is better than a patch that merges in 3 days
    and causes a regression on 10 million machines
```

The kernel moves at this pace because it runs on everything — satellites, medical devices, nuclear plant control systems, billions of phones. A bug in the kernel cannot be patched with an app update. It requires a kernel update, which requires distribution packaging, which requires users actually updating. Getting it right the first time is the entire point.

---

> **Now you know the full lifecycle of a kernel change. Next: the release machinery that takes all these merged patches and ships them to the world — merge windows, release candidates, stable trees, and LTS kernels in full detail.**
> **→ [04-how-releases-work](../04-how-releases-work/README.md)**
