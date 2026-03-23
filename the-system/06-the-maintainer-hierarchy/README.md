# The Maintainer Hierarchy

---

> The Linux kernel has no CEO, no product manager, no engineering manager telling people what to work on. And yet 4,000 contributors coordinate successfully every release cycle without chaos. How? Through a hierarchy of maintainers — people who have earned the trust of the community over years and now hold authority over specific parts of the codebase. Who are these people? How did they get there? What do they actually do every day? And critically — how do you reach them?

**You're about to find out — every level of the hierarchy, every major maintainer, their contact details, and how authority actually works in the world's largest collaborative software project.**

---

## What's Inside This File

```
06-the-maintainer-hierarchy/
│
├── How Authority Works — earned not assigned
├── The Four Levels of the Hierarchy
├── Level 1 — Linus Torvalds
│   ├── What he personally reviews
│   ├── What he delegates entirely
│   ├── How he communicates
│   └── What gets his personal attention
├── Level 2 — The Lieutenants
│   └── Every major subsystem owner with full details
├── Level 3 — Subsystem Maintainers
├── Level 4 — Reviewers and Contributors
├── How Someone Becomes a Maintainer
├── Maintainer Burnout — the real problem
└── How to Read the MAINTAINERS File
```

---

## How Authority Works — Earned Not Assigned

There are no job titles in the kernel hierarchy. No one is promoted to "Senior Kernel Maintainer." Authority in the kernel community comes from one source: demonstrated technical judgment over time.

```
How kernel authority actually accumulates:

Year 1: You submit patches
        Community reviews them
        Some get accepted, some get rejected
        You learn what the standards are

Year 2: Your patches are consistently good
        Maintainers start trusting your judgment
        You begin reviewing other people's patches
        Your Reviewed-by carries some weight

Year 3+: You know a subsystem deeply
         Existing maintainer acknowledges your expertise
         Community starts directing questions to you
         You are added to MAINTAINERS file as reviewer

Eventually: Maintainer steps back or asks for help
            Community consensus forms around you
            You are acknowledged as co-maintainer
            Then maintainer

No formal process. No interview. No promotion committee.
The community decides — by continuing to take your
patches and trusting your judgment — that you
have earned the role.
```

This system has a significant advantage: you cannot fake your way into a maintainer role. Years of visible, reviewable work on a public mailing list is the only path. There is no politics, no seniority, no networking your way in. The code and the reviews speak for themselves.

It also has a significant disadvantage: the barrier to entry is high and the path is long. This contributes to diversity problems in the kernel community — more on that later.

---

## The Four Levels of the Hierarchy

```
Level 1:  Linus Torvalds
          Final authority over mainline
          Pulls from Level 2 only

Level 2:  Lieutenants (major subsystem owners)
          Authority over large areas
          Pull from Level 3
          Send pull requests to Linus

Level 3:  Subsystem maintainers
          Authority over specific subsystems
          Accept patches from contributors
          Send to Level 2 or directly to Linus

Level 4:  Reviewers and contributors
          No merge authority
          Reviews carry weight
          Submitting patches
```

Most patches never reach Linus directly. They flow upward through the hierarchy — contributor to subsystem maintainer to lieutenant to Linus. Linus pulls entire trees, not individual patches.

---

## Level 1 — Linus Torvalds

```
Name:     Linus Torvalds
Email:    torvalds@linux-foundation.org
Role:     Creator and lead maintainer of the Linux kernel
Since:    1991
Location: Portland, Oregon, USA
Employer: Linux Foundation Fellow (funded by LF)
Tree:     git.kernel.org/torvalds/linux.git
```

### What Linus Actually Does Day to Day

Many people assume Linus reads every patch that goes into the kernel. He does not. The kernel receives ~70 patches per day. Reading each one carefully would take a full team of engineers.

What Linus actually does:

```
Linus's actual workload:

During merge window (2 weeks per cycle):
├── Reads ~100 pull request messages from lieutenants
├── Reviews the summary, shortlog, diffstat for each
├── Pulls trees that look right
├── Occasionally digs into specific areas he is
│   concerned about
└── Merges ~10,000-14,000 commits over 2 weeks

During rc cycle (6-8 weeks per cycle):
├── Reviews patches that are escalated to him
│   (important fixes, regressions, conflicts)
├── Reads daily reports from kernel testing systems
├── Makes judgment calls on disputed technical questions
├── Writes weekly rc announcement emails
└── Decides when the kernel is ready to release

What he does NOT do:
├── Review individual driver patches
├── Review staging tree patches
├── Review documentation patches
└── Review anything that has a clear subsystem owner
```

### What Gets Linus's Personal Attention

```
Things Linus personally reviews:

Core architecture changes:
└── Changes to how the kernel fundamentally works
    New system calls, major scheduler changes,
    memory model changes, security architecture

Cross-subsystem conflicts:
└── When two subsystem maintainers disagree
└── When a change affects multiple subsystems badly
└── Linus makes the final call

Reversions:
└── If a merged patch causes widespread breakage
└── Linus may personally revert it with explanation

Anything unusual in a pull request:
└── If a pull request is much larger than expected
└── If it contains something surprising
└── Linus asks the maintainer to explain

Release decisions:
└── Every final release decision is his alone
└── He decides when rc7 is good enough to release
└── No committee, no vote
```

### How Linus Communicates

Linus communicates entirely through the kernel mailing list. He does not use Slack, Discord, GitHub issues, or any other platform for kernel work. His emails are direct, often blunt, and always technically substantive.

```
Linus communication style — real examples:

When something is technically wrong:
  "This is complete and utter garbage."
  "What were you thinking?"
  "NAK. This is wrong for the following reasons..."

When something is right:
  "Looks good to me."
  "Applied, thanks."
  [No response — patch just appears in his tree]

When explaining a decision:
  Long, detailed technical explanation of why a
  particular approach is wrong and what the
  correct approach would be.

The 2018 Code of Conduct change:
└── After community pressure, Linus took a break
└── Came back with commitment to be more measured
└── Still direct, still technically blunt
└── But the personal attacks have reduced significantly
```

---

## Level 2 — The Lieutenants

These are the people who hold authority over the largest subsystems and who send pull requests directly to Linus. Knowing who these people are and what they own is essential for any kernel contributor.

---

### Greg Kroah-Hartman

```
Name:     Greg Kroah-Hartman
Email:    gregkh@linuxfoundation.org
IRC:      gregkh on various kernel IRC channels
Employer: Linux Foundation Fellow
Trees:    kernel.org/pub/scm/linux/kernel/git/gregkh/

Subsystems maintained:
├── Stable kernel releases (ALL stable branches)
│   └── The most important: he decides what gets
│       backported to stable and LTS kernels
├── USB subsystem (drivers/usb/)
├── Driver core (drivers/base/)
├── char/misc drivers (drivers/char/, drivers/misc/)
├── drivers/staging/ (the entry point for new drivers)
├── tty/serial layer (drivers/tty/)
└── sysfs, kobject infrastructure

Mailing lists:
├── stable@vger.kernel.org (stable backports)
├── linux-usb@vger.kernel.org (USB)
└── linux-kernel@vger.kernel.org (general)

Why Greg matters more than almost anyone:
└── He maintains ALL stable kernels simultaneously
└── Linux 6.8.x, 6.6.x (LTS), 6.1.x (LTS),
    5.15.x (LTS), 5.10.x (LTS), 4.19.x (LTS)
└── Every security fix that reaches Android phones,
    enterprise servers, and embedded devices
    goes through Greg's stable tree
└── He reviews hundreds of backport patches per week
    on top of all his other subsystems
```

Greg is also well-known for being welcoming to new contributors through the staging tree. He has explicitly stated that drivers/staging/ exists as an entry point and that he will review patches from beginners patiently.

---

### Andrew Morton

```
Name:     Andrew Morton
Email:    akpm@linux-foundation.org
Employer: Linux Foundation Fellow
Tree:     kernel.org/pub/scm/linux/kernel/git/akpm/mm.git

Subsystems maintained:
├── mm/ — memory management (entire subsystem)
│   └── Page allocator, slab allocator, vmalloc
│   └── OOM killer, swap, mmap, huge pages
│   └── One of the most complex and critical subsystems
├── -mm tree — a staging area for misc patches
│   └── Patches that do not have a clear subsystem home
│   └── Various core kernel improvements
└── Various core kernel files

Mailing list:
└── linux-mm@kvack.org

Why Andrew matters:
└── Memory management is at the core of everything
└── mm/ bugs can cause data loss, system crashes,
    security vulnerabilities
└── Andrew reviews mm patches with extraordinary care
└── He is known for extremely detailed technical review
└── Getting a patch into mm/ requires high quality
    and strong justification
```

---

### Jakub Kicinski

```
Name:     Jakub Kicinski
Email:    kuba@kernel.org
Employer: Meta
Tree:     kernel.org/pub/scm/linux/kernel/git/netdev/

Subsystems maintained:
├── Networking stack (net/ — current primary maintainer)
│   └── Core network stack (net/core/)
│   └── IPv4 and IPv6 (net/ipv4/, net/ipv6/)
│   └── TCP, UDP, SCTP protocols
│   └── Netfilter (iptables, nftables)
│   └── Wireless stack (net/wireless/, net/mac80211/)
└── Network device drivers (drivers/net/)

Co-maintained with:
└── Paolo Abeni <pabeni@redhat.com>

Mailing list:
└── netdev@vger.kernel.org

Note on networking history:
└── David S. Miller <davem@davemloft.net> maintained
    networking for over 20 years before stepping back
└── Jakub took over as primary maintainer
└── David still participates and his review carries weight

Networking trees:
├── net.git — fixes for current release
└── net-next.git — new features for next release
    This distinction matters: send fixes to net,
    features to net-next
```

---

### Ingo Molnar

```
Name:     Ingo Molnar
Email:    mingo@redhat.com
Employer: Red Hat
Tree:     kernel.org/pub/scm/linux/kernel/git/tip/tip.git
          (the "tip" tree — tip of development)

Subsystems maintained:
├── x86 architecture (arch/x86/) — co-maintained
├── Scheduler (kernel/sched/)
│   └── The CFS (Completely Fair Scheduler)
│   └── Real-time scheduler
│   └── Load balancing across CPUs
├── Locking subsystem (kernel/locking/)
│   └── Mutex, spinlock, rwlock, seqlock
│   └── Lock correctness tooling (lockdep)
├── Performance counters (kernel/events/, tools/perf/)
│   └── The perf tool is largely his creation
└── Timers and timekeeping (kernel/time/)

Mailing list:
└── linux-kernel@vger.kernel.org (tip tree patches)

Why Ingo matters:
└── The scheduler determines how well every workload
    runs on Linux
└── Scheduler bugs cause performance regressions
    that affect millions of users
└── Ingo has maintained this for 15+ years
└── His lockdep tool has prevented countless
    kernel deadlocks from shipping
```

---

### Thomas Gleixner

```
Name:     Thomas Gleixner
Email:    tglx@linutronix.de
Employer: Linutronix (his own company, real-time Linux)
Tree:     kernel.org/pub/scm/linux/kernel/git/tip/tip.git
          (shares tip tree with Ingo Molnar)

Subsystems maintained:
├── x86 architecture (arch/x86/) — co-maintained with Ingo
├── Interrupt subsystem (kernel/irq/)
│   └── Generic interrupt handling framework
│   └── IRQ chip drivers
├── High-resolution timers (kernel/time/hrtimer.c)
├── clocksource and clockevent subsystems
├── PREEMPT_RT — real-time Linux patches
│   └── Long effort to merge RT patches into mainline
│   └── Largely complete as of kernel 6.x
└── x86 CPU vulnerabilities
    └── Spectre, Meltdown, MDS mitigations
    └── Thomas was central to writing these

Mailing list:
└── linux-kernel@vger.kernel.org

Why Thomas matters:
└── Real-time Linux is critical for industrial,
    medical, and robotics applications
└── The PREEMPT_RT merge was a decade-long effort
└── x86 security mitigations for Spectre/Meltdown
    affected every Intel CPU user in the world
```

---

### Russell King

```
Name:     Russell King
Email:    linux@armlinux.org.uk
Website:  armlinux.org.uk
Tree:     kernel.org/pub/scm/linux/kernel/git/rmk/

Subsystems maintained:
├── ARM architecture (arch/arm/)
│   └── 32-bit ARM — the original ARM Linux support
│   └── Maintained this since the early 1990s
├── MCI/MMC subsystem (drivers/mmc/)
└── Serial/UART drivers

Mailing list:
└── linux-arm-kernel@lists.infradead.org

Why Russell matters:
└── ARM Linux support started with him in 1993
└── Before Linaro consolidated things, Russell was
    the single point of contact for all 32-bit ARM
└── Still active and reviewing ARM patches
└── One of the longest-serving kernel maintainers
```

---

### Will Deacon

```
Name:     Will Deacon
Email:    will@kernel.org
Employer: Google
Tree:     kernel.org/pub/scm/linux/kernel/git/arm64/linux.git

Subsystems maintained:
├── ARM64 architecture (arch/arm64/)
│   └── 64-bit ARM — modern phones, Apple Silicon,
│       AWS Graviton, Raspberry Pi 4+
├── ARM64 memory model
│   └── The memory ordering model for ARM64 is complex
│   └── Will is one of the world's experts on it
└── ARM IOMMU drivers

Co-maintained with:
└── Catalin Marinas <catalin.marinas@arm.com>

Mailing list:
└── linux-arm-kernel@lists.infradead.org

Why Will matters:
└── ARM64 is now the dominant architecture
    for phones, tablets, cloud, and embedded
└── Apple Silicon Macs run ARM64 Linux
└── AWS Graviton (ARM64) powers significant cloud workloads
└── Getting ARM64 right affects more devices
    than almost any other architecture
```

---

### Paolo Bonzini

```
Name:     Paolo Bonzini
Email:    pbonzini@redhat.com
Employer: Red Hat
Tree:     kernel.org/pub/scm/linux/kernel/git/kvm/kvm.git

Subsystems maintained:
├── KVM — Kernel-based Virtual Machine
│   └── The hypervisor inside the Linux kernel
│   └── Powers AWS EC2, Google Compute Engine,
│       Azure VMs, and most cloud infrastructure
├── KVM for x86 (arch/x86/kvm/)
├── KVM for ARM64 (arch/arm64/kvm/)
└── KVM for other architectures (S/390, PowerPC, MIPS)

Mailing list:
└── kvm@vger.kernel.org

Why KVM matters:
└── When you launch an EC2 instance or a GCP VM,
    you are almost certainly running inside KVM
└── KVM bugs can cause VM escapes — security
    vulnerabilities where guest code breaks out
    and accesses the host
└── Paolo's work directly affects cloud security
    for hundreds of millions of users
```

---

### Rafael J. Wysocki

```
Name:     Rafael J. Wysocki
Email:    rafael@kernel.org
Employer: Intel
Tree:     kernel.org/pub/scm/linux/kernel/git/rafael/

Subsystems maintained:
├── ACPI subsystem (drivers/acpi/)
│   └── Advanced Configuration and Power Interface
│   └── How the OS controls hardware power states
│   └── Critical for laptops and power management
├── Power management framework (drivers/base/power/)
│   └── System sleep states (suspend, hibernate)
│   └── Runtime PM — device power management
├── CPUfreq — CPU frequency scaling
│   └── How CPUs throttle up and down for power saving
└── Thermal management framework

Mailing list:
└── linux-acpi@vger.kernel.org
└── linux-pm@vger.kernel.org

Why Rafael matters:
└── Battery life on Linux laptops depends on ACPI
└── Sleep and resume behaviour depends on his subsystems
└── Poor power management was a famous Linux laptop
    problem for years — Rafael's work has
    dramatically improved this
```

---

### Takashi Iwai

```
Name:     Takashi Iwai
Email:    tiwai@suse.de
Employer: SUSE
Tree:     kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git

Subsystems maintained:
├── ALSA — Advanced Linux Sound Architecture
│   └── The entire Linux audio subsystem
│   └── sound/ directory
├── HD Audio (HDA) codec drivers
│   └── Realtek, Conexant, IDT/Sigmatel codecs
│   └── The drivers that make laptop audio work
├── USB audio drivers
└── FireWire audio

Mailing list:
└── alsa-devel@alsa-project.org

Why Takashi matters:
└── Audio on Linux has a reputation for being
    complicated — PulseAudio, PipeWire, ALSA...
└── The kernel layer is Takashi's domain
└── He maintains support for thousands of audio
    device variants across all laptop manufacturers
└── Has been doing this since the early 2000s
```

---

### Kees Cook

```
Name:     Kees Cook
Email:    keescook@chromium.org
Employer: Google
Tree:     kernel.org/pub/scm/linux/kernel/git/kees/linux.git

Subsystems maintained:
├── Kernel security hardening (broadly)
├── seccomp — system call filtering
│   └── Used by Chrome sandbox, Docker, Flatpak
├── KSPP — Kernel Self Protection Project
│   └── Stack canaries, ASLR improvements
│   └── Control Flow Integrity (CFI)
│   └── Hardened usercopy
└── Various security-sensitive APIs

Mailing list:
└── linux-hardening@vger.kernel.org
└── linux-security-module@vger.kernel.org

Why Kees matters:
└── Modern kernel security is largely his work
└── CFI prevents entire classes of exploit techniques
└── seccomp is how Chrome, Firefox, and containers
    sandbox themselves — a Kees contribution
└── The kernel is significantly harder to exploit
    today than it was 10 years ago, partly because
    of his sustained effort
```

---

### Dmitry Torokhov

```
Name:     Dmitry Torokhov
Email:    dmitry.torokhov@gmail.com
Employer: Google
Tree:     kernel.org/pub/scm/linux/kernel/git/dtor/input.git

Subsystems maintained:
├── Input subsystem (drivers/input/)
│   └── Every keyboard, mouse, touchpad, touchscreen
│   └── The evdev interface that X11 and Wayland use
│   └── Gamepad and joystick drivers
└── Serio — serial I/O for PS/2 devices

Mailing list:
└── linux-input@vger.kernel.org

Why Dmitry matters:
└── Every time you type on a Linux machine,
    your keystrokes travel through his subsystem
└── Touchpad support — one of the historically
    painful areas of Linux laptop support —
    lives in his domain
└── Has maintained this since 2005
```

---

### Krzysztof Kozlowski

```
Name:     Krzysztof Kozlowski
Email:    krzk@kernel.org
Employer: Linaro
Tree:     Various ARM and device tree trees

Subsystems maintained:
├── Samsung ARM SoC (arch/arm/mach-s3c/, mach-s5p/)
├── ARM64 device trees for Samsung hardware
├── Device tree bindings (Documentation/devicetree/)
│   └── Co-maintainer of DT bindings for many subsystems
│   └── Extremely active reviewer of DT bindings
└── Qualcomm ARM SoC (shared maintenance)

Mailing list:
└── linux-arm-kernel@lists.infradead.org
└── linux-samsung-soc@vger.kernel.org

Why DT bindings matter:
└── Device tree is how ARM hardware describes itself
    to the kernel
└── Every new ARM board needs DT bindings reviewed
└── Krzysztof reviews a huge volume of DT patches
    across many different subsystems
```

---

## Level 3 — Subsystem Maintainers

Below the lieutenants are hundreds of subsystem maintainers responsible for more specific areas. A few important ones:

```
Selected Level 3 maintainers:

Filesystem maintainers:
├── Ted Ts'o <tytso@mit.edu> — ext4
├── Dave Chinner <david@fromorbit.com> — XFS
├── Chris Mason / Josef Bacik — Btrfs (Facebook)
└── Steve French <sfrench@samba.org> — CIFS/SMB

GPU/Graphics:
├── Dave Airlie <airlied@redhat.com> — DRM subsystem
├── Daniel Vetter <daniel@ffwll.ch> — DRM core
└── Alex Deucher <alexander.deucher@amd.com> — amdgpu

Storage:
├── Jens Axboe <axboe@kernel.dk> — block layer, io_uring
│   └── io_uring is entirely his creation
│   └── One of the most significant recent kernel additions
└── Martin K. Petersen <martin.petersen@oracle.com> — SCSI

Security:
├── James Morris <jmorris@namei.org> — LSM framework
└── Paul Moore <paul@paul-moore.com> — SELinux, audit

BPF (one of the fastest-growing subsystems):
├── Alexei Starovoitov <ast@kernel.org> — BPF (Meta)
└── Daniel Borkmann <daniel@iogearbox.net> — BPF (Isovalent)
```

---

## How Someone Becomes a Maintainer

There is no formal process. But there is a clear pattern:

```
The path to becoming a maintainer:

Phase 1 — Contributor (months to years):
├── Submit quality patches consistently
├── Fix your review feedback promptly
├── Learn the subsystem deeply
└── Build a track record the community can see

Phase 2 — Reviewer (1-3 years):
├── Start reviewing other people's patches
├── Your review comments are technically sound
├── Maintainer starts pointing you at patches
│   "Can you take a look at this?"
├── Added to MAINTAINERS as R: (reviewer)
└── Your Reviewed-by is respected

Phase 3 — Co-maintainer (1-3 more years):
├── Maintainer trusts you to pick up patches
├── You have merge rights to a subsystem tree
├── You handle the routine work
└── Maintainer focuses on architectural decisions

Phase 4 — Maintainer:
├── Previous maintainer steps back or moves on
├── Community consensus forms around you
├── You are acknowledged in MAINTAINERS as M:
└── You now receive all patch emails for the subsystem

Total time: 3-8 years is typical
            Some people get there faster
            Many never get there — and that is fine
            Being a respected contributor is valuable
            without being a maintainer
```

---

## Maintainer Burnout — The Real Problem

Maintainer burnout is one of the most serious structural problems in the Linux kernel project. It is not discussed enough.

```
Why maintainers burn out:

Volume:
├── Popular subsystems receive hundreds of patches/week
├── Each patch requires reading, understanding, responding
├── Review feedback must be written carefully
│   Wrong review = bad code ships
│   Harsh review = contributor quits
└── No limit on how many patches get submitted

No vacation:
├── The kernel never stops
├── A maintainer who takes two weeks off comes back
│   to hundreds of unreviewed patches
├── Some maintainers check email on holidays
└── This is unsustainable long-term

Entitled submitters:
├── "Why haven't you reviewed my patch yet?"
├── Pinging after 3 days
├── Getting defensive about feedback
└── This costs maintainers time and morale

What happens when a maintainer burns out:
├── They go silent — patches pile up unreviewed
├── They step back — subsystem becomes unmaintained
│   └── MAINTAINERS status changes to "Orphan"
├── They quit entirely
└── The subsystem suffers until a replacement emerges

Real examples:
├── Several USB device drivers are in staging because
│   the original maintainer disappeared
├── Some filesystems have had long gaps in maintenance
└── The mm subsystem has historically been
    dependent on very few people
```

### What You Can Do

```
As a contributor, you can help prevent burnout:

Do:
├── Submit clean patches that require minimal iteration
├── Be patient — wait 2-4 weeks before pinging
├── Thank maintainers when they review your work
│   (rare — they notice)
├── Review other contributors' patches yourself
│   Reduces the maintainer's review load
└── Take on reviewer responsibility as you gain expertise

Do not:
├── Ping after 3 days
├── Argue aggressively about review feedback
├── Submit obviously unfinished work
│   "I'll clean it up if you think it's worth merging"
│   → Do not do this. Clean it up first.
└── Treat maintainers as a service
    They are volunteers with full-time jobs
    (or LF fellows with 10 other things to maintain)
```

---

## How to Read the MAINTAINERS File

The MAINTAINERS file at the root of the kernel tree is your directory for reaching the right person. It is 25,000+ lines but its structure is simple.

```
Example entry — USB subsystem:

USB SUBSYSTEM
M: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
L: linux-usb@vger.kernel.org
S: Maintained
W: http://www.linux-usb.org
T: git git://git.kernel.org/pub/.../usb/usb.git
T: git git://git.kernel.org/pub/.../usb/usb-next.git
F: Documentation/usb/
F: drivers/usb/
F: include/linux/usb.h
F: include/linux/usb/

Field meanings:
M: Maintainer — send patches here
R: Reviewer — CC on patches, not primary
L: Mailing list — always CC this
S: Status
   Maintained  = actively maintained, send patches
   Odd Fixes   = only critical fixes considered
   Orphan      = no maintainer, needs one
   Obsolete    = do not bother
W: Website
B: Bug tracker URL (if not lore.kernel.org)
T: Git tree — where patches land
F: Files covered by this entry
   drivers/usb/ means all files under that path
   drivers/usb/*.c means only .c files directly in it
   drivers/usb/**/ means all subdirectories too
X: Excluded files (exceptions to F: patterns)
N: File name pattern match (regex)
K: Keyword in commit message that triggers this entry
```

You do not need to read this manually. The script does it:

```
# Find maintainers for a file you changed:
./scripts/get_maintainer.pl --file drivers/usb/core/hub.c

# Find maintainers for your uncommitted changes:
./scripts/get_maintainer.pl patch.diff

# Output tells you exactly who to send to:
Greg Kroah-Hartman <gregkh@linuxfoundation.org> (maintainer)
linux-usb@vger.kernel.org (open list)
linux-kernel@vger.kernel.org (open list)

# Use this output:
# To:  maintainer
# Cc:  mailing lists
```

Run `get_maintainer.pl` on every patch you send. It is fast, accurate, and prevents the most common mistake new contributors make: sending to the wrong person.

---

> **Now you know who runs the kernel — every level, every major maintainer, every contact. Next: how do these people communicate? The mailing list system that holds everything together.**
> **→ [07-the-mailing-list-system](../07-the-mailing-list-system/README.md)**
