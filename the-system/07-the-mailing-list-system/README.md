# The Mailing List System

---

> You want to contribute to the Linux kernel. You go to GitHub. You look for the repository. You find torvalds/linux. You try to open a pull request. GitHub tells you pull requests are disabled. There is no issue tracker. No discussions tab. Nothing. Just a read-only mirror of the code. Where does the actual work happen? How do 4,000 contributors coordinate without any of the modern collaboration tools everyone uses?

**The answer is email. And after this, you will understand not just why — but exactly how the system works, which lists exist, how to read threads, and how maintainers actually process the flood of patches they receive every day.**

---

## What's Inside This File

```
07-the-mailing-list-system/
│
├── Why Email — the real reasons, not just tradition
├── What LKML Is — history, volume, what it is for today
├── The Major Mailing Lists — every important list
│   with address and what it covers
├── How to Read Kernel Mailing List Threads
│   ├── lore.kernel.org — the archive
│   └── How threads are structured
├── Mailing List Etiquette
│   ├── Plain text only
│   ├── Inline reply — never top-post
│   └── Threading correctly
├── Patchwork — how maintainers track patches
│   └── Every state explained
└── How Linus Actually Reads Mail
```

---

## Why Email — The Real Reasons

Every new contributor asks this question. The answer is not "because old people are used to it." There are genuine technical and social reasons why email works better than any alternative for this specific use case.

### It Is Asynchronous and Distributed

The kernel has contributors in every timezone. A maintainer in Germany, a contributor in Japan, a reviewer in California. Synchronous tools — Slack, Discord, real-time chat — require people to be online simultaneously. Email does not.

```
Timezone reality for a patch review:

Monday 9am PST:   Contributor in California submits patch
Monday 6pm CET:   Maintainer in Germany reads it, responds
Tuesday 9am JST:  Reviewer in Japan adds comments
Tuesday 1am PST:  California contributor wakes up, reads both
                  Sends v2

Total clock time: ~36 hours
All three people worked during their normal hours
No one scheduled a meeting
No one waited for anyone to be online
```

With Slack or GitHub discussions, you get notification noise, pressure to respond immediately, and timezone-driven exclusion. Email naturally accommodates a globally distributed volunteer team.

### It Is Permanently Archived and Searchable

Every email sent to a kernel mailing list is archived permanently at lore.kernel.org. Every discussion, every review, every technical argument, every decision — all searchable, all linkable, all permanently accessible.

```
Value of permanent email archives:

A bug appears in Linux 6.9.
Bisection points to a commit from 2019.
Why was this change made?

With email archives:
└── Find the patch email from 2019
└── Read the full review discussion
└── See what alternatives were considered
└── Understand the reasoning
└── Fix the bug correctly because you understand
    why the original code was written that way

Without email archives:
└── "Why was this written this way?"
└── No record exists
└── Guess. Probably wrong. Introduce another bug.
```

GitHub issues and PR comments exist — until a repository is deleted, made private, or GitHub shuts down. lore.kernel.org has emails going back to the early 1990s and is independently operated. The archive does not depend on any single company's continued existence.

### It Works With the Patch Format Natively

A kernel patch is a diff — a text file showing what changed. Email is a text format. A patch sent by email is:

```
Exactly what gets sent:

From: Developer Name <dev@example.com>
To: maintainer@kernel.org
Cc: linux-subsystem@vger.kernel.org
Subject: [PATCH] subsystem: fix the thing

Commit message explaining what and why.

Signed-off-by: Developer Name <dev@example.com>
---
 file.c | 5 +++--
 1 file changed, 3 insertions(+), 2 deletions(-)

diff --git a/file.c b/file.c
index abc123..def456 100644
--- a/file.c
+++ b/file.c
@@ -42,7 +42,8 @@ static int do_thing(void)
-       old code
+       new code
```

The maintainer receives this email. They run:

```
git am < the_email.eml
```

The patch applies directly. No conversion, no reformatting, no copy-paste. The email IS the patch. This tight coupling between the communication format and the technical artifact is something no web-based tool has replicated cleanly.

### It Scales to High Patch Volume

The kernel receives ~70 patches per day. A GitHub pull request model at this volume would create an unusable interface — thousands of open PRs, no way to filter by subsystem, no way to delegate review cleanly. Email with mailing lists naturally shards the volume by subsystem. A networking developer subscribes to netdev. A memory management developer subscribes to linux-mm. Nobody has to see everything.

```
Email volume management:

LKML total volume: ~500-1000 emails/day
netdev alone:      ~200-400 emails/day

How developers manage this:
├── Subscribe only to relevant lists
├── Use email filters aggressively
│   └── Filter by subsystem keywords
│   └── Filter by their own name (to catch reviews)
│   └── Mark everything else as low priority
└── Read lore.kernel.org for specific threads
    without subscribing to the full volume
```

---

## What LKML Is — History, Volume, and Purpose

**LKML** stands for Linux Kernel Mailing List. The address is:

```
linux-kernel@vger.kernel.org
```

It is the oldest and highest-volume mailing list in the kernel ecosystem. Understanding what it is for today requires understanding what it was for originally.

### History

LKML was created in 1995 when the kernel was small enough that a single list could cover all development. Back then, a patch for networking and a patch for memory management both went to the same list and the same handful of people read everything.

As the kernel grew, this became unworkable. Subsystem-specific lists were created. By the 2000s, most real technical work had moved to subsystem lists. By 2010, LKML was already more of a coordination point than a primary review venue.

### What LKML Is For Today

```
What LKML is used for today:

CC destination:
└── Most patches are CC'd to LKML in addition to
    the subsystem list
└── Creates a complete public record of all patch activity
└── Anyone can watch the full stream of kernel development

Cross-subsystem discussions:
└── When a change affects multiple subsystems
└── When there is no clear single list to use

Announcements:
└── Linus's release announcements go here
└── Merge window open/close announcements
└── Security vulnerability disclosures (sometimes)

General kernel discussion:
└── Topics that span multiple subsystems
└── Questions about kernel architecture

What LKML is NOT for today:
├── Primary review of most patches
│   └── Those happen on subsystem lists
└── Support questions from users
    └── Use distribution forums for that
```

Sending a patch ONLY to LKML without the subsystem list is one of the most common new contributor mistakes. The relevant maintainer may not read LKML carefully. The patch gets lost.

---

## The Major Mailing Lists

These are the lists you need to know. For each one: what it covers, the address, and how to find more.

---

### Core and General

```
LKML — Linux Kernel Mailing List
Address: linux-kernel@vger.kernel.org
Volume:  Very high (~500-1000 emails/day)
Purpose: General kernel discussion, CC for all patches
Archive: lore.kernel.org/lkml/

linux-kernel-mentees
Address: linux-kernel-mentees@lists.linux.dev
Volume:  Low
Purpose: Kernel Mentorship Program participants
         Good starting point if you are new
```

---

### Architecture Lists

```
ARM (32-bit):
Address: linux-arm-kernel@lists.infradead.org
Maintainer: Russell King, others
Archive: lore.kernel.org/linux-arm-kernel/

ARM64 (64-bit):
Address: linux-arm-kernel@lists.infradead.org
          (same list as ARM)
Maintainer: Will Deacon, Catalin Marinas

x86:
Address: linux-kernel@vger.kernel.org (uses LKML)
Maintainer: Ingo Molnar, Thomas Gleixner, H. Peter Anvin

RISC-V:
Address: linux-riscv@lists.infradead.org
Maintainer: Paul Walmsley, Palmer Dabbelt

MIPS:
Address: linux-mips@vger.kernel.org

PowerPC:
Address: linuxppc-dev@lists.ozlabs.org
```

---

### Major Subsystem Lists

```
Networking:
Address: netdev@vger.kernel.org
Volume:  Very high (~200-400 emails/day)
Owner:   Jakub Kicinski, Paolo Abeni
Archive: lore.kernel.org/netdev/
Note:    One of the most active subsystem lists
         Strict formatting requirements
         Read netdev FAQ before posting first patch

Memory Management:
Address: linux-mm@kvack.org
Volume:  High
Owner:   Andrew Morton
Archive: lore.kernel.org/linux-mm/
Note:    Different server than vger (kvack.org)
         Patches need strong justification
         MM bugs are critical — review is thorough

USB:
Address: linux-usb@vger.kernel.org
Owner:   Greg Kroah-Hartman
Archive: lore.kernel.org/linux-usb/
Note:    Good list for new contributors
         Greg is patient with beginners

Filesystems (general):
Address: linux-fsdevel@vger.kernel.org
Owner:   Various filesystem maintainers
Archive: lore.kernel.org/linux-fsdevel/

ext4:
Address: linux-ext4@vger.kernel.org
Owner:   Ted Ts'o, Andreas Dilger

Btrfs:
Address: linux-btrfs@vger.kernel.org
Owner:   David Sterba, Chris Mason, Josef Bacik

XFS:
Address: linux-xfs@vger.kernel.org
Owner:   Dave Chinner, Darrick J. Wong

Block layer and storage:
Address: linux-block@vger.kernel.org
Owner:   Jens Axboe
Note:    io_uring patches also go here

SCSI:
Address: linux-scsi@vger.kernel.org
Owner:   Martin K. Petersen, James E.J. Bottomley

Graphics (DRM):
Address: dri-devel@lists.freedesktop.org
Owner:   Dave Airlie, Daniel Vetter
Note:    freedesktop.org server, not vger

Input devices:
Address: linux-input@vger.kernel.org
Owner:   Dmitry Torokhov

Sound (ALSA):
Address: alsa-devel@alsa-project.org
Owner:   Takashi Iwai
Note:    alsa-project.org server, not vger

Power Management:
Address: linux-pm@vger.kernel.org
Owner:   Rafael J. Wysocki, Pavel Machek

ACPI:
Address: linux-acpi@vger.kernel.org
Owner:   Rafael J. Wysocki

Clk/Clock framework:
Address: linux-clk@vger.kernel.org
Owner:   Stephen Boyd, Michael Turquette

GPIO:
Address: linux-gpio@vger.kernel.org
Owner:   Linus Walleij, Bartosz Golaszewski

I2C:
Address: linux-i2c@vger.kernel.org
Owner:   Wolfram Sang

SPI:
Address: linux-spi@vger.kernel.org
Owner:   Mark Brown

Regulator:
Address: linux-regulator@vger.kernel.org
Owner:   Mark Brown

Device Tree / Device Tree Bindings:
Address: devicetree@vger.kernel.org
Owner:   Rob Herring, Frank Rowand, Krzysztof Kozlowski
Note:    ALL new device tree bindings must be
         reviewed here before merging anywhere

MMC/SD:
Address: linux-mmc@vger.kernel.org
Owner:   Ulf Hansson

Bluetooth:
Address: linux-bluetooth@vger.kernel.org
Owner:   Marcel Holtmann, Johan Hedberg

WiFi (mac80211/cfg80211):
Address: linux-wireless@vger.kernel.org
Owner:   Johannes Berg
```

---

### Security Lists

```
Kernel Security (LSM framework):
Address: linux-security-module@vger.kernel.org
Owner:   James Morris, Paul Moore

SELinux:
Address: selinux@vger.kernel.org
Owner:   Paul Moore, Stephen Smalley

Kernel Hardening:
Address: linux-hardening@vger.kernel.org
Owner:   Kees Cook
Note:    KSPP (Kernel Self Protection Project) work

Private security reports (vulnerabilities):
Address: security@kernel.org
Note:    For reporting security vulnerabilities
         NOT for patches
         Use for CVE-level issues only
         Embargoed until disclosure date
```

---

### Virtualisation and Containers

```
KVM:
Address: kvm@vger.kernel.org
Owner:   Paolo Bonzini
Note:    Also architecture-specific lists:
         kvm-arm@lists.cs.columbia.edu (ARM KVM)
         kvm-ppc@vger.kernel.org (PowerPC KVM)

Containers / cgroups / namespaces:
Address: linux-fsdevel@vger.kernel.org (namespace patches)
         cgroups@vger.kernel.org (cgroup patches)
```

---

### Driver Staging

```
Staging drivers:
Address: linux-staging@lists.linux.dev
Owner:   Greg Kroah-Hartman
Note:    Where new/incomplete drivers go
         Best list for first-time contributors
         Explicit TODO files tell you what to fix
         Greg reviews with patience for beginners
```

---

## How to Read Kernel Mailing List Threads

### lore.kernel.org

lore.kernel.org is the official, permanent archive of all kernel mailing lists. It is independently operated, search-indexed, and has been running since 2019 (with older archives imported).

```
Using lore.kernel.org:

Browse by list:
└── lore.kernel.org/linux-kernel/  (LKML)
└── lore.kernel.org/netdev/        (networking)
└── lore.kernel.org/linux-usb/     (USB)
└── lore.kernel.org/[list-name]/

Search across all lists:
└── lore.kernel.org/all/?q=search+term

Find a specific patch by message ID:
└── lore.kernel.org/r/message-id@domain

Link to a specific patch thread:
└── Every email has a permanent URL
└── Use this in your commit's Link: tag
└── Link: https://lore.kernel.org/r/your-patch-id
```

### How Threads Are Structured

A patch thread on a kernel mailing list follows a specific structure:

```
Thread structure for a single patch:

[PATCH] subsystem: fix the thing          <- your patch
  Re: [PATCH] subsystem: fix the thing    <- reviewer comment
    Re: [PATCH] subsystem: fix the thing  <- your response
      Re: [PATCH] subsystem: fix the thing <- maintainer
  Re: [PATCH] subsystem: fix the thing    <- another reviewer

Thread structure for a patch series:

[PATCH 0/3] subsystem: fix three things  <- cover letter
  [PATCH 1/3] subsystem: fix first thing <- patch 1
    Re: [PATCH 1/3]...                   <- review of patch 1
  [PATCH 2/3] subsystem: fix second thing <- patch 2
  [PATCH 3/3] subsystem: fix third thing  <- patch 3
```

When you send v2 after review:

```
[PATCH v2 0/3] subsystem: fix three things <- new cover letter
  [PATCH v2 1/3] subsystem: fix first thing <- revised patch 1
  [PATCH v2 2/3] subsystem: fix second thing
  [PATCH v2 3/3] subsystem: fix third thing

Note: v2 starts a new thread
      Reference the v1 thread with:
      Link: https://lore.kernel.org/r/v1-message-id
```

---

## Mailing List Etiquette

This is where many new contributors make mistakes. The kernel mailing list has specific conventions and violating them — even accidentally — creates a bad first impression.

---

### Plain Text Only — No Exceptions

```
Email format rules:

REQUIRED: Plain text (text/plain MIME type)
FORBIDDEN: HTML email
FORBIDDEN: Rich text
FORBIDDEN: Inline images
FORBIDDEN: Word documents or PDFs as patches

Why plain text:
└── Patches must be readable as text
└── HTML adds invisible markup that breaks patches
└── Many kernel developers use terminal email clients
    (mutt, notmuch, gnus) that do not render HTML
└── HTML email signals unfamiliarity with the process
    and gets patches ignored

How to check your email client:
└── Gmail: compose → More options → Plain text mode
└── Outlook: notoriously bad — use git send-email instead
└── Thunderbird: account settings → composition → plain text
└── Best option: use git send-email for patches
    It always sends correctly formatted plain text
```

---

### Inline Reply — Never Top-Post

This is the single biggest culture shock for developers coming from corporate email culture.

```
WRONG — top-posting (what corporate email does):

> Original question from reviewer:
> "Why did you choose this approach instead of X?"

I chose this approach because it avoids the lock
contention that X would introduce in the hot path.

----

RIGHT — inline reply (what kernel email does):

> Why did you choose this approach instead of X?

I chose this approach because it avoids the lock
contention that X would introduce in the hot path.

> Also, the variable name here is confusing.

Agreed — I'll rename it to something clearer in v2.
```

The difference matters because:
- Inline reply keeps context local — you see the question immediately before the answer
- Top-posting buries the original questions at the bottom
- Reviewers comment on multiple separate points in one email — inline replies address each one in context
- Top-posting a reply to a kernel email is an immediate signal that you do not know the culture

```
Inline reply rules:

1. Quote only the relevant part you are responding to
   Not the entire email — trim aggressively

2. Put your response BELOW the quoted text

3. Delete quoted text you are not responding to

4. If reviewer made 5 comments:
   Quote comment 1 → respond
   Quote comment 2 → respond
   ... and so on

5. Never:
   └── Start your email with your response
   └── Put all quoted text at the bottom
   └── Quote the entire email including signatures
```

---

### Correct Email Threading

Email clients thread messages by `In-Reply-To` and `References` headers. When you send v2 of a patch, it should reference the v1 thread. When you respond to a review, your response should thread under the review.

```
git send-email handles threading automatically:

For a patch series, it threads all patches under
the cover letter automatically.

For v2 after receiving reviews:
└── Use --in-reply-to with the message ID of your v1
    cover letter (or single patch if no series)

git send-email \
  --in-reply-to="<message-id-from-v1@mail.kernel.org>" \
  *.patch

Where to find the message ID:
└── lore.kernel.org — the URL contains the message ID
└── The email headers: Message-ID: <...>
└── Your sent mail folder
```

---

## Patchwork — How Maintainers Track Patches

Patchwork is a web application that tracks patches sent to kernel mailing lists. It reads the mailing list archives, identifies patch emails, and maintains a database of their status.

Every major kernel subsystem has a Patchwork instance:

```
Major Patchwork instances:

Main kernel Patchwork:
└── patchwork.kernel.org

Networking:
└── patchwork.kernel.org/project/netdevbpf/list/

USB:
└── patchwork.kernel.org/project/linux-usb/list/

Input:
└── patchwork.kernel.org/project/linux-input/list/
```

### Patchwork States — What Each Means

```
State           What it means for you

New             Your patch arrived, not yet looked at
                Normal — wait 2-4 weeks before pinging

Under Review    Someone is actively reviewing it
                Do not resend — response is coming

Accepted        Maintainer applied it to their tree
                Your patch will go into the next release

Superseded      A newer version replaced this patch
                Your v2 made v1 superseded — correct

RFC             Request For Comments — seeking feedback
                not yet ready for merge consideration
                Set this yourself with [RFC] in subject

Changes         Maintainer wants changes before accepting
Requested       Look at the review comments, send v2

Rejected        Maintainer will not accept this patch
                Read the rejection reason carefully
                Ask what the correct approach would be

Not Applicable  Patch is not relevant to this project
                Wrong list, wrong subsystem

Deferred        Accepted but held for a future cycle
                Common for large features near cycle end
```

### How Maintainers Use Patchwork

Maintainers use Patchwork as their inbox and task tracker for patches. When they apply a patch to their tree, they mark it "Accepted" in Patchwork. When they need changes, they mark it "Changes Requested."

```
Maintainer workflow using Patchwork:

Morning routine for a busy maintainer:

1. Open Patchwork, filter by "New" status
2. Go through new patches:
   ├── Read patch + commit message
   ├── If obviously wrong: reply with explanation,
   │   mark "Changes Requested" or "Rejected"
   ├── If needs more review: leave as "New"
   │   or mark "Under Review"
   └── If looks good: apply to tree,
       mark "Accepted"
3. Check "Under Review" patches from last week
   └── Has the author responded to feedback?
   └── Have other reviewers commented?
4. Process patches that got Reviewed-by or Tested-by
   └── These are ready to apply

Your patch in Patchwork:
└── Check patchwork.kernel.org for your subsystem
└── Find your patch
└── Status tells you where you stand without
    needing to send a ping email
```

---

## How Linus Actually Reads Mail

This is something many contributors wonder about. Does Linus read every email on LKML? Does he see your patch?

```
Linus's email setup (publicly described):

Email client: custom setup, reportedly evolved from
              older setups over decades
Volume:       Thousands of emails per day
              across all his lists and direct mail

What he reads:
├── Pull request emails from lieutenants
│   └── These go directly to him
│   └── He reads every one during merge window
├── Emails addressed directly to him
│   └── CCed on important discussions
│   └── Escalations from maintainers
└── LKML — selectively, not exhaustively

What he does NOT read:
├── Every patch posted to LKML
│   └── He relies on maintainers to filter
├── Subsystem mailing lists
│   └── That is what maintainers are for
└── Issues or discussions on GitHub
    └── He uses the mirror but does not interact there
```

Your patch should not be going to Linus directly. If you are using `get_maintainer.pl` correctly, Linus will not be in the output for most patches. He is at the top of the hierarchy — your patch needs to travel up through maintainers before it reaches him.

```
The only time your patch reaches Linus:

Your patch → subsystem maintainer tree
subsystem tree → lieutenant pull request
lieutenant → Linus's tree via git pull

Linus never read your individual patch.
He pulled a tree that contains it.
That is exactly how it is supposed to work.

Exception: if your patch is for something
Linus personally maintains — the very core
of the kernel. Almost nobody patches that.
```

---

## Subscribing to Lists

To subscribe to a vger.kernel.org mailing list:

```
Subscribing to vger.kernel.org lists:

Send an email to:
  majordomo@vger.kernel.org

With body:
  subscribe linux-usb
  (replace linux-usb with the list name)

To unsubscribe:
  Send: unsubscribe linux-usb

Managing volume without subscribing:
└── Read lore.kernel.org directly
└── Subscribe with nomail option to be able to post
    without receiving all email:
    send: subscribe linux-usb nomail
```

For lists on other servers (alsa-project.org, infradead.org, freedesktop.org) — each has its own subscription mechanism. Check the list archive page for instructions.

---

## The Full Picture

The mailing list system is not just a communication tool — it is the entire collaboration infrastructure for the world's most important open source project.

```
What the mailing list system provides:

Permanent record:
└── Every decision, every technical argument,
    every review — permanently archived
    and searchable at lore.kernel.org

Global coordination:
└── 4,000 contributors across all timezones
    working asynchronously without chaos

Patch delivery:
└── Email IS the patch format
    No conversion, no upload, no web UI
    The patch goes directly to the maintainer

Scaling through subsystems:
└── Each developer only subscribes to
    relevant lists — nobody reads everything
    Volume is distributed naturally

Quality filter:
└── The process of sending a well-formatted
    plain-text patch to the right list is itself
    a filter — it demonstrates basic competence
    before a single line of code is reviewed
```

Mastering the mailing list system is not optional for kernel contribution. It is the prerequisite for everything else. Once you understand it, the process becomes logical. Before you understand it, every step feels arbitrary.

---

> **Now you understand the communication infrastructure. Next: the-process — how to actually participate, starting with setting up your environment correctly.**
> **→ [08-setting-up-your-environment](../../the-process/08-setting-up-your-environment/README.md)**
