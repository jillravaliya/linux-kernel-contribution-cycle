# Linux Kernel Contribution Cycle

![Sections](https://img.shields.io/badge/Sections-03-000000?style=for-the-badge&logo=github&logoColor=white&labelColor=grey)
![Files](https://img.shields.io/badge/Files-14-000000?style=for-the-badge&logo=files&logoColor=white&labelColor=grey)
![Focus](https://img.shields.io/badge/Focus-Kernel%20Contribution-EE4444?style=for-the-badge&logo=linux&logoColor=white&labelColor=grey)
![Status](https://img.shields.io/badge/Status-Working-13AA52?style=for-the-badge&logo=buffer&logoColor=white&labelColor=grey)
![License](https://img.shields.io/badge/License-CC--BY--SA--4.0-1D96E8?style=for-the-badge&logo=creativecommons&logoColor=white&labelColor=grey)

> Most resources about kernel contribution either explain the mechanics without the context, or explain the culture without the mechanics. This repository covers both — in the order that actually makes sense.

---

## What This Is

The Linux kernel receives ~70 patches per day from ~4,000 contributors. It has no issue tracker, no pull requests, no sprint planning. Contributions happen through email, reviewed by a hierarchy of maintainers who earned their authority through years of visible, public work.

Contributing to it for the first time is genuinely difficult — not because the community is hostile, but because the system has specific expectations and very little tolerance for sloppiness. A patch sent to the wrong maintainer gets ignored. A patch with checkpatch.pl errors signals you did not do basic preparation. A patch with a vague commit message tells the reviewer nothing useful.

This repository covers everything needed to navigate that — not as a checklist, but as a deep explanation of how the system works, who runs it, and exactly what to do at each step.

```
What you will be able to do after reading this:

├── Understand what the kernel is and how it is organised
├── Know the full maintainer hierarchy by name and email
├── Find the right person and list for any patch you write
├── Write a commit message that passes review
├── Run every pre-submission check correctly
├── Send a patch or patch series with git send-email
├── Respond to review feedback correctly
└── Track your patch from submission to mainline
```

---

## Repository Structure

### **[the-system/](./the-system/README.md)**
> Understand the machine before you touch it — what the kernel is, how it is organised, how changes move through it, how releases work, and who funds it.

**Covers:**
- Kernel vs OS vs distribution — the real difference
- Source tree layout — every major directory explained
- How a patch moves from laptop to release (full lifecycle)
- Merge windows, rc1-rc8, mainline, stable, LTS
- Intel, Google, Red Hat, Samsung, Microsoft — why they all contribute
- GSoC, Linux Kernel Mentorship, Outreachy

---

### **[the-people/](./the-people/README.md)**
> Know who runs it and how to reach them — the full hierarchy, every major maintainer with contact details, and the communication infrastructure that holds it together.

**Covers:**
- Linus Torvalds — what he personally reviews vs delegates
- Greg Kroah-Hartman, Andrew Morton, Jakub Kicinski, Ingo Molnar, Thomas Gleixner, Will Deacon, Paolo Bonzini, Kees Cook — and more
- Every major subsystem owner with name, email, tree URL
- How someone becomes a maintainer (years, not weeks)
- Why the MAINTAINERS file exists and how to read it
- Every major mailing list with address
- lore.kernel.org, Patchwork, inline reply culture

---

### **[the-process/](./the-process/README.md)**
> The complete step-by-step contribution workflow — from setting up your environment to seeing your patch ship in a release.

**Covers:**
- Which kernel tree to clone and why it matters
- The staging tree as an entry point for new contributors
- Commit message format — every field, every rule
- get_maintainer.pl — finding the right maintainer every time
- checkpatch.pl, sparse, smatch — what each catches
- git send-email — full setup, single patch, series, v2, v3
- Tracking, responding to review, review culture, timelines

---

## How to Read This

```
New to kernel contribution — read everything in order:

the-system/01  →  02  →  03  →  04  →  05
                                          │
                                          v
                              the-people/06  →  07
                                                  │
                                                  v
          the-process/08 → 09 → 10 → 11 → 12 → 13 → 14
                                                        │
                                                        v
                                              Your first merged patch

Looking for something specific:

  Who maintains the networking subsystem?
  └── the-people/06-the-maintainer-hierarchy/

  What is a merge window?
  └── the-system/04-how-releases-work/

  How do I write a commit message?
  └── the-process/10-writing-the-patch/

  How do I use git send-email?
  └── the-process/13-sending-the-patch/

  What does silence from a maintainer mean?
  └── the-process/14-after-you-send/

  Why does the kernel use email instead of GitHub?
  └── the-people/07-the-mailing-list-system/
```

---

## Author

> Built by **Jill Ravaliya** while going deep on the Linux kernel contribution process — further than any single course covers.

**Focus Areas:**
- Linux kernel development and contribution
- Open source ecosystem and governance
- Systems programming
- Supply chain security

**Currently Exploring:**
- Upstream kernel patch submission
- Kernel subsystem internals
- Open source contribution at scale

---

## Connect With Me

I'm actively learning and building in the **Linux kernel** and **systems programming** space.

- **Email:** jillahir9999@gmail.com
- **LinkedIn:** [linkedin.com/in/jill-ravaliya-684a98264](https://linkedin.com/in/jill-ravaliya-684a98264)
- **GitHub:** [github.com/jillravaliya](https://github.com/jillravaliya)

**Open to:**
- Kernel development discussions
- Open source contribution collaboration
- Systems programming conversations
- Feedback on these notes

---

### ⭐ Star this repository if it helped you understand how Linux kernel contribution actually works.

---

> **Start here → [the-system/01-what-the-kernel-is/](./the-system/01-what-the-kernel-is/README.md)**
