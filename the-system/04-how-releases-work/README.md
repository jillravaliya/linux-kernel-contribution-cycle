# How Releases Work

---

> You write a patch. It gets reviewed. It gets accepted by a subsystem maintainer. Then what? How does it actually get into a release that ships to millions of machines? What is a release candidate? Why does your patch sometimes take three months to reach users even after it was accepted? What is the difference between mainline, stable, and LTS — and why does it matter which one your fix lands in?

**You're about to find out — the full release machinery, from merge window to stable backport.**

---

## What's Inside This File

```
04-how-releases-work/
│
├── The Big Picture — the full release pipeline
├── The Merge Window — what it is and how it works
├── Release Candidates — rc1 through rc8 explained
├── The Final Release — what happens when Linus tags it
├── Mainline vs Stable vs LTS — the three trees
├── linux-next — the integration testing tree
├── A Real Release Timeline with example dates
└── What this means for your patch
```

---

## The Big Picture — The Full Release Pipeline

Before going deep on each piece, here is the entire pipeline at once. Everything below explains one part of this diagram.

```
DEVELOPER                SUBSYSTEM             LINUS
WRITES PATCH             MAINTAINER            TORVALDS
     |                       |                     |
     |---patch to list------->|                     |
     |                       |                     |
     |<--review feedback------|                     |
     |                       |                     |
     |---v2 patch------------>|                     |
     |                       |                     |
     |        patch sits in subsystem tree          |
     |                       |                     |
     |              MERGE WINDOW OPENS              |
     |                       |                     |
     |                       |---pull request------>|
     |                       |                     |
     |                       |       Linus merges   |
     |                       |       into mainline  |
     |                       |                     |
     |              MERGE WINDOW CLOSES (2 weeks)   |
     |                       |                     |
     |                  rc1 released                |
     |                  rc2 released (1 week later) |
     |                  rc3...rc4...rc5...rc6...rc7 |
     |                       |                     |
     |                  FINAL RELEASE               |
     |                  (6.x tagged on kernel.org)  |
     |                       |                     |
     |           STABLE TEAM picks it up            |
     |           Backports to 6.(x-1), 6.(x-2)...  |
     |                       |                     |
     v                       v                     v
  YOUR PATCH IS NOW IN A RELEASE THAT SHIPS
  TO MILLIONS OF MACHINES
```

The whole cycle takes approximately **9 to 10 weeks** from merge window open to final release. Your patch may have been accepted by a subsystem maintainer weeks before the merge window even opened — meaning the total time from submission to release can be anywhere from 6 weeks to 6 months depending on timing.

---

## The Merge Window — What It Is and How It Works

The merge window is a **two-week period** that opens immediately after each major release. It is the only time new features and significant changes can enter Linus's mainline tree.

```
Timeline example using Linux 6.8:

  March 10, 2024  -- Linux 6.7 released
                     Merge window for 6.8 OPENS immediately

  March 10-24     -- Two-week merge window
                     Subsystem maintainers send pull requests
                     Linus merges subsystem trees into mainline
                     ~10,000-14,000 changesets merged
                     This is the busiest two weeks of the cycle

  March 24, 2024  -- Merge window CLOSES
                     Linux 6.8-rc1 released
                     No new features accepted after this point
```

During the merge window, Linus is merging at a pace of hundreds of commits per hour at peak. He is not reviewing individual patches — he is pulling entire subsystem trees that maintainers have already reviewed and curated. The individual patch review happened weeks or months earlier at the subsystem level.

### What Subsystem Maintainers Do During the Merge Window

Each subsystem maintainer has been collecting patches in their tree throughout the previous development cycle. When the merge window opens, they send Linus a pull request — a message saying "here is my tree, please pull these N commits."

```
Subsystem maintainer during merge window:

  From: Jakub Kicinski <kuba@kernel.org>
  To: Linus Torvalds <torvalds@linux-foundation.org>

  Hi Linus,

  Here is the networking pull request for 6.8.
  Please pull from:
    git://git.kernel.org/pub/.../net-next.git

  This contains:
  - 847 commits from 183 authors
  - New driver for Mediatek MT7988 SoC
  - BPF improvements for tc offload
  - Various fixes and cleanups

  ...shortlog...
  ...diffstat...
```

Linus reviews the pull request message, looks at the diffstat, and if satisfied, runs `git pull`. The entire subsystem tree lands in mainline in one operation.

### What Gets Accepted During the Merge Window

Only changes that were already in a subsystem tree before the merge window opens get merged. You cannot submit a new patch during the merge window and expect it to land in the current release. The merge window is for pulling pre-reviewed work, not for new submissions.

```
Common mistake from new contributors:

  Release 6.7 ships on March 10
  New contributor submits a patch on March 12
  "The merge window just opened, my patch will
   make it into 6.8!"

  Reality:
  └── Their patch has not been reviewed yet
  └── Has not been in any subsystem tree
  └── Will not make it into 6.8
  └── Best case: lands in subsystem tree in April
      Gets pulled in 6.9 merge window in May
      6.9 ships in July
  └── Total time from submission to release: ~4 months
```

This is not a problem with the process — it is the process working correctly. Patches need review time. Subsystem trees need testing time. The merge window pulls code that has already gone through that process.

---

## Release Candidates — rc1 Through rc8 Explained

When the merge window closes, Linus releases the first release candidate: `-rc1`. From this point until the final release, **only bug fixes are accepted**. No new features.

```
What each release candidate means:

rc1  -- Merge window just closed
        All the new code just landed
        This is the most unstable point
        Lots of new bugs, regressions, conflicts
        "Do not use rc1 on anything important"

rc2  -- First round of fixes
        The most obvious breakage fixed
        Still rough around the edges
        Build failures eliminated

rc3  -- Stabilising
        Serious regressions being addressed
        Driver conflicts resolved
        Still finding problems from new code

rc4  -- Midpoint
        Things are calming down
        Good developers use rc4 for daily driving
        Tells Linus if the release is on track

rc5  -- Getting stable
        Only targeted fixes for known bugs
        Large patch series no longer accepted
        Focus narrows to critical issues

rc6  -- Late stabilisation
        Only fixes for regressions introduced
        in this cycle accepted
        Serious kernel developers using this daily

rc7  -- Near final
        Most releases ship after rc7
        If things are calm, Linus releases here

rc8  -- Extended stabilisation
        Only needed when rc7 found new problems
        Rare — most cycles do not reach rc8
        Signals a difficult release cycle
```

Linus sends a weekly email announcing each rc. These announcements are worth reading — they tell you the health of the release cycle and what areas are causing problems.

Real example of a Linus rc announcement:

```
Subject: Linux 6.8-rc4 released

So things continue to look fairly normal for this release,
and rc4 is out there for your testing pleasure.

The biggest thing here is probably the networking fixes,
with some driver fixes (gpu, sound, usb, etc.) being the
other bulk. And then there's misc fixes all over.

Go test,
Linus
```

### How Long Do rc Cycles Take?

Each rc is released approximately one week after the previous one. A typical cycle:

```
rc1     Week 3 of cycle   (merge window just closed)
rc2     Week 4
rc3     Week 5
rc4     Week 6
rc5     Week 7
rc6     Week 8
rc7     Week 9    <-- most common final release point
rc8     Week 10   <-- if needed

Final   Week 9 or 10
```

The total time from rc1 to final release is 6-8 weeks. Combined with the 2-week merge window, the full cycle is 8-10 weeks.

---

## The Final Release

When Linus decides the kernel is ready, he tags the release on kernel.org and sends an announcement to the kernel mailing list. There is no committee vote. No product management approval. One person decides.

```
What Linus looks at before releasing:

├── Patch volume in the final weeks
│   └── If still getting many patches: not ready
│   └── If patch volume has dropped to trickle: ready
│
├── Nature of remaining patches
│   └── Only small isolated fixes: ready
│   └── Still fixing regressions from new features: wait
│
├── Reports from kernel testers
│   └── 0-day bot results (Intel's automated testing)
│   └── Reports from distributions testing rc kernels
│   └── Syzkaller fuzzer findings
│
└── His own judgment
    └── He has been doing this since 1991
    └── He knows what a ready kernel feels like
```

The announcement looks like this:

```
From: Linus Torvalds <torvalds@linux-foundation.org>
Subject: Linux 6.8 released

So the last week was fairly quiet, and I have no
excuses to delay the v6.8 release.

So here it is. The full shortlog is obviously enormous,
but the changes since rc7 are tiny, and just the usual
last-minute fixes.

Go get it,
Linus
```

After this, the version is tagged in the git tree, tarballs appear on kernel.org, and distributions begin integrating it.

---

## Mainline vs Stable vs LTS — The Three Trees

This is one of the most confusing aspects of the kernel release process for new contributors. There are three different trees, each serving a different purpose.

---

### Mainline — Linus's Tree

```
Tree:    git.kernel.org/torvalds/linux.git
Owner:   Linus Torvalds
Purpose: Development — new features land here first
Users:   Developers, testers, bleeding-edge distros
Cadence: New release every ~10 weeks
```

Mainline is where everything starts. Every patch must eventually pass through mainline. When people say "the kernel," they usually mean mainline. Fedora and Arch Linux track mainline relatively closely. Ubuntu and Debian do not — they use LTS.

---

### Stable — Greg Kroah-Hartman's Tree

```
Tree:    git.kernel.org/stable/linux.git
Owner:   Greg Kroah-Hartman (and Sasha Levin)
Purpose: Bug fixes and security patches for released kernels
Users:   Distributions, production systems
Cadence: Releases every 1-2 weeks per maintained version
```

After Linus releases 6.8, the stable team takes over maintaining it. They backport important bug fixes and security patches from mainline into 6.8.x. These are called **stable releases** or **point releases**.

```
Example stable release sequence:

  6.8    -- Linus releases mainline
  6.8.1  -- Greg releases stable (1-2 weeks later)
  6.8.2  -- Greg releases stable (1-2 weeks later)
  6.8.3  -- Greg releases stable
  ...
  6.8.12 -- Final stable release before EOL
             (next major release is stable by now)
```

### What Qualifies for Stable Backport?

Not every patch gets backported to stable. The stable team has strict rules:

```
Stable patch rules (from Documentation/process/stable-kernel-rules.rst):

MUST meet ALL of these:
├── It must fix a real bug that users can encounter
├── It cannot be larger than 100 lines (with context)
├── It must already be in Linus's mainline tree
├── It must not introduce new features
├── It must not contain "theoretical" fixes
└── It must fix something that could cause data loss,
    security issues, or hardware damage

To request stable backport, add to commit message:
  Cc: stable@vger.kernel.org
```

If you fix a real bug in mainline and want it in stable, you add `Cc: stable@vger.kernel.org` to your commit message. The stable team reviews it and decides whether to include it.

---

### LTS — Long Term Support

```
Owner:   Greg Kroah-Hartman (selects LTS versions)
Purpose: Extended support for embedded, enterprise, Android
Users:   Android devices, embedded Linux, enterprise distros
Support: 2 years minimum, some kernels up to 6 years
```

Not every kernel version becomes LTS. Greg selects specific versions — roughly one per year — for long-term maintenance. These receive backported fixes for years after mainline has moved on.

```
Current and recent LTS kernels (as of 2024):

Version   Released    EOL (End of Life)
--------------------------------------
6.6       Oct 2023    Dec 2026
6.1       Dec 2022    Dec 2026
5.15      Oct 2021    Dec 2026
5.10      Dec 2020    Dec 2026
5.4       Nov 2019    Dec 2025
4.19      Oct 2018    Dec 2024

Android uses 5.10 and 5.15 heavily.
Many embedded devices run 4.19 or older.
```

LTS kernels matter enormously for the real world. The Android phone in your pocket is probably running a 5.10 or 5.15 LTS kernel. A security fix in mainline 6.8 needs to be backported all the way to 5.10 to protect those devices.

---

## linux-next — The Integration Testing Tree

Before your patch enters a subsystem tree and before the merge window, there is an intermediate step that many new contributors do not know about: **linux-next**.

```
What linux-next is:

  Every day, Stephen Rothwell builds linux-next by
  pulling together:
  ├── Linus's current mainline tree
  ├── ~200 subsystem trees
  └── Merging them all together

  Result: a tree that represents what the NEXT
  kernel release will roughly look like

  Purpose: catch merge conflicts and integration
  problems BEFORE the merge window opens

  If two subsystems both modified the same file
  in incompatible ways, linux-next reveals it now
  rather than during the merge window
```

Why this matters for contributors:

```
Your patch journey through linux-next:

  You submit patch to subsystem maintainer
          |
          v
  Maintainer accepts into their tree
          |
          v
  Stephen's script pulls subsystem tree into linux-next
          |
          v
  Automated testing runs on linux-next
  (0-day bot, kernel test robot)
          |
          v
  If your patch breaks something in another subsystem:
  └── You get an automated email
  └── Maintainer may ask you to fix before merge window
          |
          v
  Merge window opens
  Maintainer sends clean pull request to Linus
```

linux-next is also useful for contributors who want to test their patch against the most recent code. Always develop patches against linux-next if you are targeting a new feature, or against the relevant stable tree if you are fixing a bug.

---

## A Real Release Timeline — Linux 6.8

Here is the complete timeline for the Linux 6.8 release to make all of this concrete:

```
Linux 6.8 release timeline:

Jan 7,  2024  -- Linux 6.7 released by Linus
                 6.8 merge window OPENS

Jan 7-21      -- Merge window (2 weeks)
                 13,000+ commits pulled from subsystem trees
                 New features: EEVDF scheduler improvements,
                 FUSE passthrough, network improvements...

Jan 21, 2024  -- Merge window closes
                 Linux 6.8-rc1 released

Jan 28         -- Linux 6.8-rc2
Feb 4          -- Linux 6.8-rc3
Feb 11         -- Linux 6.8-rc4
Feb 18         -- Linux 6.8-rc5
Feb 25         -- Linux 6.8-rc6
Mar 3          -- Linux 6.8-rc7

Mar 10, 2024  -- Linux 6.8 FINAL RELEASE
                 "things have been quiet, here we go"
                 -- Linus Torvalds

Mar 10        -- 6.9 merge window opens immediately
Mar ~15        -- Linux 6.8.1 stable released by Greg KH
Mar ~22        -- Linux 6.8.2 stable
...continues for ~6 months until 6.9 stable...
```

Total time from 6.7 release to 6.8 release: **62 days**.

---

## What This Means for Your Patch

Understanding releases tells you how to time your contributions:

```
Scenarios and what to expect:

Scenario 1: You submit a bug fix
├── Submit to subsystem mailing list
├── Gets reviewed and accepted into subsystem tree
├── If marked Cc: stable@vger.kernel.org:
│   └── Backported to stable within weeks
│   └── Users get it in the next 6.x.y release
└── Gets into mainline at next merge window

Scenario 2: You submit a new feature
├── Submit to subsystem mailing list
├── Review may take weeks or months
├── Accepted into subsystem's -next tree
│   (separate from the fixes tree)
├── Goes into linux-next for integration testing
├── Pulled into mainline during next merge window
└── Ships in the next major release (~10 weeks later)
    Total time: potentially 3-6 months

Scenario 3: You submit right before merge window
├── Patch submitted 3 days before merge window opens
├── Maintainer has no time to review properly
├── Does NOT get pulled in this merge window
└── Sits in subsystem tree for next cycle
    Total time: 4-6 months

Best timing for new features:
└── Submit early in the cycle — right after a release
    This gives maximum review time before the next
    merge window opens
```

The release process is not bureaucracy — it is what allows 4,000 contributors to work on the same codebase simultaneously without catastrophic breakage. Every piece of it has a reason.

---

> **Now you understand how releases work and how your patch gets from a subsystem tree to a shipped kernel. Next: who are the actual people running all of this — the companies funding kernel development and the individuals who do it for free?**
> **→ [05-who-contributes-and-why](../05-who-contributes-and-why/README.md)**
