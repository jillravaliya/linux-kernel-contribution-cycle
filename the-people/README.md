# The People — Who Runs the Linux Kernel

---

> The kernel has no CEO, no sprint planning, no engineering manager. It has maintainers — people who earned authority over specific areas through years of visible, reviewable work on a public mailing list. You cannot fake your way into a maintainer role. You cannot network your way in. The code and the reviews speak for themselves.

**This section covers everyone who runs the kernel — their names, their email addresses, their subsystems, and the communication infrastructure that holds 4,000 contributors together.**

---

## Why This Section Exists

You can write a perfect patch and have it ignored because you sent it to the wrong person. You can have a legitimate technical disagreement and escalate it incorrectly. You can misread a blunt email response as hostility when it is just directness.

Knowing who runs what, how they communicate, and why the culture is the way it is prevents all of these mistakes before they happen.

---

## What Is in This Section

---

### [06 — The Maintainer Hierarchy](./06-the-maintainer-hierarchy/README.md)

> The full hierarchy from Linus to subsystem maintainers — every major name, their email, their subsystem, and what they actually do day to day. Plus how someone becomes a maintainer and why burnout is a real structural problem.

```
The four levels:

  Level 1:  Linus Torvalds
            Final authority over mainline
            Pulls from lieutenants only
            Does NOT review individual patches

  Level 2:  Lieutenants (major subsystem owners)
            Send pull requests directly to Linus
            Authority over large areas of the kernel

  Level 3:  Subsystem maintainers
            Accept patches from contributors
            Send to Level 2 or directly to Linus

  Level 4:  Reviewers and contributors
            No merge authority
            Reviews carry weight

Key lieutenants — every one covered in full detail:

  Greg Kroah-Hartman  gregkh@linuxfoundation.org
  └── Stable trees, USB, driver core, staging
      The person whose work reaches the most
      devices — every stable backport goes through him

  Andrew Morton        akpm@linux-foundation.org
  └── Memory management — the most complex subsystem
      mm/ bugs can cause data loss and security issues

  Jakub Kicinski       kuba@kernel.org
  └── Networking — net/, drivers/net/
      200-400 patch emails per day on netdev

  Ingo Molnar          mingo@redhat.com
  └── x86, scheduler, perf, locking
      The scheduler determines how every workload runs

  Thomas Gleixner      tglx@linutronix.de
  └── x86, interrupts, timers, PREEMPT_RT
      Wrote the Spectre/Meltdown mitigations

  Will Deacon          will@kernel.org
  └── ARM64 — modern phones, Apple Silicon, Graviton

  Paolo Bonzini        pbonzini@redhat.com
  └── KVM — the hypervisor inside AWS, GCP, Azure

  Kees Cook            keescook@chromium.org
  └── Kernel security hardening, seccomp, CFI

  ... and 8 more covered in full detail
```

---

### [07 — The Mailing List System](./07-the-mailing-list-system/README.md)

> Why email — not GitHub, not Slack, not Jira. What LKML is and what it is actually used for today. Every major mailing list with its address. How to read threads on lore.kernel.org. Patchwork states. The inline reply rule that trips up every new contributor.

```
Why email works for this project:

  Problem: 4,000 contributors, every timezone,
           70 patches per day, all coordinating

  Email solves:
  ├── Asynchronous — no need to be online together
  ├── Permanently archived — lore.kernel.org has
  │   emails back to the early 1990s
  ├── Native patch format — git send-email produces
  │   exactly what git am consumes. The email IS
  │   the patch. No conversion needed.
  └── Naturally sharded — subscribe only to
      the lists relevant to your subsystem

The lists you need to know:

  linux-kernel@vger.kernel.org        LKML — CC everything
  netdev@vger.kernel.org              Networking
  linux-usb@vger.kernel.org           USB (good for beginners)
  linux-mm@kvack.org                  Memory management
  linux-arm-kernel@lists.infradead.org  ARM + ARM64
  linux-staging@lists.linux.dev       Staging drivers
  kvm@vger.kernel.org                 Virtualisation
  linux-hardening@vger.kernel.org     Security hardening
  alsa-devel@alsa-project.org         Audio
  dri-devel@lists.freedesktop.org     GPU/graphics

  ... 20+ more covered with full addresses

The one rule that trips up every new contributor:

  WRONG (top-posting — what corporate email does):
  Your reply at the top
  > Reviewer's comment buried at the bottom

  CORRECT (inline reply — what kernel email does):
  > Reviewer's comment quoted here
  Your response directly below it
  > Next reviewer comment
  Your response directly below it
```

---

## How These Two Files Connect

```
06-the-maintainer-hierarchy
    └── WHO runs things — names, emails, authority

07-the-mailing-list-system
    └── HOW they communicate — lists, threads, culture

Together they answer:
"Where does my patch need to go and
 how does communication actually work here?"
          │
          v
    Ready for: the-process/
```

---

## The Culture Question

The kernel has a reputation for harsh communication. Before you encounter it, understand why it exists:

```
Why kernel review is direct:

The stakes are real:
├── Medical devices, satellites, 3 billion phones
├── A bug that ships cannot be silently patched
└── Soft feedback that misses the severity
    causes bad code to get merged

The volume is real:
└── Maintainers review hundreds of patches per week
    Writing diplomatic responses to everything
    would be a full-time job by itself

After 2018:
└── Linus took a break, the kernel adopted a CoC
    Personal attacks reduced significantly
    Technical bluntness remained — and should remain
    "This is wrong because X" is useful information
    "You are an idiot" is not — and is no longer acceptable

What this means for you:
└── Separate the tone from the content
    If the technical point is correct, fix it
    If you disagree, explain your technical reasoning
    Your patch is not your identity
```

---

> **Next: [the-process/](../the-process/README.md) — you understand the system and the people. Now the step-by-step process of actually participating.**
