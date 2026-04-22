# The Process — How to Actually Contribute

---

> You understand the system. You know who runs it. Now the part most guides try to cover without the foundation — the actual step-by-step process of contributing a patch to the Linux kernel. Every step in order. Every command explained. Every mistake documented so you do not have to make it yourself.

**This section is the complete contribution workflow. Seven files, seven stages. Read them in order the first time.**

---

## Why This Section Exists

The Linux kernel contribution process has more failure points than almost any other open source project. Wrong email client corrupts the patch before anyone reads the code. Wrong commit message format signals you did not read the documentation. Wrong tree base means the patch cannot be applied. Wrong mailing list means the right person never sees it.

None of these are deliberate traps. They are the accumulated conventions of a project that has been running since 1991, that processes 70 patches per day, and that cannot afford to lower the bar for a codebase running on medical devices and satellites.

Every file in this section eliminates one category of failure.

---

## What Is in This Section

---

### [08 — Setting Up Your Environment](./08-setting-up-your-environment/README.md)

> The right tree, the right tools, the right git config, the right send-email setup. Done once, correctly, before writing any code.

```
The decisions that matter before you start:

Which tree to clone:
  Bug fix for current release  → stable tree
  New feature                  → linux-next
  Not sure                     → linux-next

  Cloning the wrong tree = "please rebase" from maintainer

Build dependencies (Debian/Ubuntu):
  sudo apt-get install build-essential libncurses-dev \
    bison flex libssl-dev libelf-dev bc dwarves git

Git config that matters:
  user.name   = Your Real Name (must match Signed-off-by)
  user.email  = your@email.com (must match Signed-off-by)
  sendemail.smtpServer     = smtp.yourdomain.com
  sendemail.confirm        = always
  sendemail.chainreplyto   = false  ← critical for threading

Tools to install before writing a line:
  checkpatch.pl  (already in kernel tree at scripts/)
  sparse         (apt-get install sparse)
  smatch         (build from source)
  coccinelle     (apt-get install coccinelle)
```

---

### [09 — Finding What to Work On](./09-finding-what-to-work-on/README.md)

> The staging tree as an entry point, TODO files, checkpatch warnings in real drivers, syzkaller bugs, CVEs, and how to find a subsystem you will actually care about long-term.

```
The entry points ranked by difficulty:

Easiest — drivers/staging/ cleanup:
  └── Drivers with explicit TODO files
  └── checkpatch.pl always finds issues
  └── Greg KH reviews patiently
  └── find drivers/staging -name "TODO"

Medium — kernel janitor tasks:
  └── Coccinelle-identified patterns across the kernel
  └── API modernisation (kmalloc+memset → kzalloc)
  └── devm_ conversion in probe functions

Harder — real bugs from syzkaller:
  └── syzkaller.appspot.com/upstream
  └── Live bugs with C reproducers
  └── Real impact — these affect real systems

What to avoid first:
  ├── Trivial one-word typo patches sent in bulk
  │   (signals gaming contribution counts)
  ├── Patches sent during merge window
  │   (maintainers are busy — patch gets lost)
  └── "I'll clean it up if you think it's worth it"
      (clean it up first — always)
```

---

### [10 — Writing the Patch](./10-writing-the-patch/README.md)

> The commit message format that maintainers expect — every field, every rule, good vs bad examples. Review tags. git format-patch. What makes a patch reviewable vs what makes it immediately suspicious.

```
The commit message format:

  subsystem: short description under 72 chars

  Body explaining WHAT problem exists, WHY it exists,
  HOW this patch fixes it, and HOW it was tested.
  Wrap at 75 characters. Plain prose, no bullet points.

  Fixes: 3f8a2d1b9e4c ("commit that introduced the bug")
  Link: https://lore.kernel.org/r/bug-report-message-id
  Cc: stable@vger.kernel.org
  Signed-off-by: Your Name <your@email.com>

Rules that trip up new contributors:
  ├── Lowercase after the colon:
  │   Wrong: "usb: Fix null pointer"
  │   Right: "usb: fix null pointer"
  ├── No period at end of subject line
  ├── Blank line between subject and body — required
  ├── Body explains WHY — not just what the diff does
  └── Signed-off-by is not optional — ever

The Signed-off-by means:
  └── You certify this is your code to give
      under the kernel's license (the DCO)
      Your real name — no pseudonyms
```

---

### [11 — Finding the Right Maintainer](./11-finding-the-right-maintainer/README.md)

> get_maintainer.pl — the script that reads the MAINTAINERS file and tells you exactly who gets To: and who gets Cc:. Real output examples. Edge cases. What happens if you send to the wrong place.

```
The script that does the work:

  # On a specific file you changed:
  ./scripts/get_maintainer.pl --file drivers/usb/core/hub.c

  # Output:
  Greg Kroah-Hartman <gregkh@linuxfoundation.org> (maintainer)
  Alan Stern <stern@rowland.harvard.edu> (reviewer)
  linux-usb@vger.kernel.org (open list)
  linux-kernel@vger.kernel.org (open list)

  # Use it:
  To:  maintainer(s)
  Cc:  mailing lists + linux-kernel@vger.kernel.org

Run this on every patch. No exceptions.
It takes 2 seconds and prevents the most
common new contributor mistake.

Edge cases covered in the file:
  ├── Patch touches multiple subsystems
  ├── New file with no MAINTAINERS entry
  ├── Orphaned subsystem (no active maintainer)
  └── Script returns too many people
```

---

### [12 — Before You Send](./12-before-you-send/README.md)

> The quality gate. checkpatch.pl every error explained and fixed, sparse, smatch, build verification, testing minimum bar, and the complete pre-send checklist.

```
The checks in order:

  ./scripts/checkpatch.pl --strict 0001-your-patch.patch
  └── Fix every ERROR — no exceptions
  └── Fix every WARNING or document why you cannot

  make C=1 drivers/your-subsystem/
  └── sparse static analysis
  └── Catches __user/__iomem pointer misuse
  └── Catches endianness errors in network drivers

  make -j$(nproc) 2>&1 | grep "warning:"
  └── Zero new warnings compared to clean tree

  Test the fix:
  └── Bug is reproducible before your patch
  └── Bug is gone after your patch
  └── Document what you tested in the commit body

The rule that matters most:
  └── Your patch must not introduce any new
      compiler warnings — compared against the
      clean tree before your change
```

---

### [13 — Sending the Patch](./13-sending-the-patch/README.md)

> git send-email complete setup from scratch (Gmail, Outlook, custom SMTP), the exact send command, patch series with cover letters, v2 and v3 with changelogs, and the 10 most common sending mistakes.

```
Why git send-email — not your email client:
  Gmail:    converts tabs to spaces → breaks diff
  Outlook:  adds HTML → patch is corrupt
  Mail.app: adds rich text → unreadable

Setup once (Gmail example):
  [sendemail]
      smtpServer      = smtp.gmail.com
      smtpServerPort  = 587
      smtpEncryption  = tls
      smtpUser        = you@gmail.com
      confirm         = always
      chainreplyto    = false

The send command:
  git send-email \
    --to="maintainer@kernel.org" \
    --cc="list@vger.kernel.org" \
    --cc="linux-kernel@vger.kernel.org" \
    0001-your-patch.patch

Sending v2 after review:
  git format-patch --subject-prefix="PATCH v2" -1
  # Edit patch to add "Changes in v2:" below ---
  git send-email \
    --in-reply-to="<message-id-of-v1@example.com>" \
    ... *.patch
```

---

### [14 — After You Send](./14-after-you-send/README.md)

> Tracking on lore.kernel.org and Patchwork, what silence means, how to respond to review feedback (inline — always), sending v2, review culture, the CoC change of 2018, realistic timelines by subsystem, and what acceptance actually looks like.

```
Patchwork states and what they mean:

  New              → Normal — wait 2-4 weeks
  Under Review     → Response coming — do not ping
  Accepted         → Applied to subsystem tree
  Superseded       → Your v2 replaced v1 — correct
  Changes Requested→ Read the thread, send v2
  Rejected         → Read why — ask what is right

What silence means:
  Week 1-2:   Normal. Do nothing.
  Week 3-4:   Send polite ping if Patchwork shows "New"
  Month 2+:   Ping again. Not rejection — just busy.

When your patch is accepted:
  Day 1:     In subsystem tree
  Week 1:    In linux-next (daily integration)
  Week 4-8:  Pulled into mainline at merge window
  Week 10:   Ships in final release
  After:     Stable backports reach enterprise
             distros, LTS kernels, Android devices

Your name is now permanent in:
  git log --author="Your Name" linux/
```

---

## The Complete Workflow in One View

```
08 Setup environment
    │
    v
09 Find something to fix
    │
    v
10 Write the patch (commit message + diff)
    │
    v
11 Find the right maintainer (get_maintainer.pl)
    │
    v
12 Run all checks (checkpatch, sparse, build, test)
    │
    v
13 Send with git send-email
    │
    v
14 Track → respond to review → send v2 → accepted
    │
    v
Merge window → mainline → release → millions of machines
```

---

> **Start here: [08-setting-up-your-environment/](./08-setting-up-your-environment/README.md)**
