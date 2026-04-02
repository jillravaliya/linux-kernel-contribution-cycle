# After You Send

---

> You sent the patch. Now what? Most new contributors experience the same thing — they send a patch and then have no idea what is happening. Is anyone reading it? Did it get lost? Does silence mean rejection? What does it mean when someone says "Reviewed-by"? How do you respond to feedback without making the maintainer regret reviewing it? When do you follow up? How do you send v2 so the thread history stays intact?

**This file covers everything that happens after you send — tracking, responding, following up, interpreting silence, understanding review culture, and what it means when your patch finally gets accepted.**

---

## What's Inside This File

```
14-after-you-send/
│
├── Tracking Your Patch
│   ├── lore.kernel.org — finding your patch
│   └── Patchwork — understanding the status system
├── What Silence Means — and What to Do
├── Understanding Review Feedback
│   ├── Types of feedback
│   └── How to read harsh feedback
├── How to Respond to Review Correctly
│   ├── Inline reply — the only acceptable format
│   ├── Addressing each comment
│   └── What to say when you disagree
├── Sending v2 After Review
│   ├── Incorporating feedback
│   └── Writing the changelog
├── Review Culture — The Unwritten Rules
│   ├── Why Linus is blunt — structural not personal
│   ├── The CoC change of 2018
│   ├── Famous review exchanges with context
│   └── How to handle criticism without quitting
├── Realistic Timelines — By Subsystem
└── What Acceptance Looks Like
    ├── How to know your patch was merged
    └── What happens next
```

---

## Tracking Your Patch

### Finding Your Patch on lore.kernel.org

Within minutes of sending, your patch should appear in the mailing list archive at lore.kernel.org. If it does not appear within 24 hours, your email may not have been formatted or routed correctly.

```
Find your patch:

1. Go to lore.kernel.org
2. Navigate to the relevant list:
   lore.kernel.org/linux-usb/       (USB patches)
   lore.kernel.org/netdev/          (networking)
   lore.kernel.org/linux-mm/        (memory management)
   lore.kernel.org/linux-kernel/    (LKML)

3. Your patch should appear near the top
   sorted by date

4. Click on it to see the full thread:
   ├── Your patch
   ├── Any review responses
   └── Your replies

5. The permanent URL for your patch:
   lore.kernel.org/r/<message-id@example.com>
   Save this — use it in your v2 --in-reply-to
   and in your commit's Link: tag

If your patch does not appear after 24 hours:
├── Check your sent mail — did it actually send?
├── Check the email was plain text (not HTML)
├── Check you sent to the right list address
└── If you sent to a moderated list (linux-arm-kernel,
    alsa-devel), it may be waiting for approval
    Wait 48 hours before concluding it was lost
```

### Patchwork — Understanding the Status System

Patchwork automatically tracks patches from the mailing list and shows their current state. Check it before sending a ping.

```
Find your patch in Patchwork:

Main kernel patchwork:
  patchwork.kernel.org

Subsystem-specific:
  patchwork.kernel.org/project/linux-usb/list/
  patchwork.kernel.org/project/netdevbpf/list/
  patchwork.kernel.org/project/linux-input/list/

Search by your email address or patch subject.

Patchwork states and what they mean for you:

New
└── Patch arrived, not yet acted on
└── Normal — this is where all patches start
└── No action needed from you yet

Under Review
└── A maintainer or reviewer is actively looking at it
└── Response coming — do not resend
└── Do not ping — you will interrupt the review

Accepted
└── Maintainer applied it to their tree
└── Your patch will go into the next release cycle
└── You may not receive an explicit confirmation email
    Check the maintainer's git tree to verify

Superseded
└── A newer version of this patch exists
└── Your v2 replaced your v1 — this is correct
└── The v2 should show as New or Accepted

Changes Requested
└── Maintainer wants changes before accepting
└── Check the thread for their comments
└── Address feedback and send v2

Rejected
└── Maintainer will not accept this patch as-is
└── Read the rejection reason carefully
└── Ask what the correct approach would be
└── Do not resubmit the same patch

RFC
└── Request For Comments — seeking feedback
└── Not yet ready for merge
└── You would have set this yourself with [RFC]

Deferred
└── Patch is good but held for a future cycle
└── Common near the end of a merge window
└── Will be reconsidered next cycle

Not Applicable
└── Patch is not relevant to this project
└── Wrong list — resend to correct subsystem
```

---

## What Silence Means — and What to Do

Silence from a maintainer after submitting a patch is the most common experience for new contributors and the most misunderstood.

```
What silence usually means:

Week 1-2:
└── Normal. Maintainers have full inboxes.
    Do nothing.

Week 2-3:
└── Still normal for most subsystems.
    Do nothing.

Week 3-4:
└── Reasonable to consider a polite ping.
    Check Patchwork first — is it "Under Review"?
    If yes: wait longer. If still "New": ping.

Month 2+:
└── Ping again. Clearly and politely.
    Something may have changed — the maintainer
    may have missed it or been too busy.

What silence almost never means:
└── "Your patch is rejected"
    Rejection gets an explicit response
└── "You did something wrong"
    Wrong sends get redirected, not ignored
└── "Nobody cares about this"
    Maintainers are just busy
```

### How to Send a Polite Ping

```
Reply to your original patch thread on lore.kernel.org.
Do NOT send a new email. Thread under the original.

Good ping:
From: Your Name <you@example.com>
Subject: Re: [PATCH] usb: fix null pointer in hub

Gentle ping. Is there anything needed to move
this forward?

Thanks,
Your Name

Bad ping:
"Why haven't you merged this? I sent it 3 weeks ago."
"This is an important fix, please apply it."
"Bump."

Ping timing:
├── First ping: 3-4 weeks after sending
├── Second ping: 2-3 weeks after first ping
└── Third ping: consider asking on the list
    "Is this the right approach? I've pinged twice
    with no response — happy to rework if needed."
```

---

## Understanding Review Feedback

### Types of Feedback

```
Nit (minor suggestion):
└── "Nit: this variable name could be clearer"
└── Small style or naming preference
└── Usually preceded by "nit:" or "minor:"
└── Fix it in v2 — shows you pay attention to detail

Request for clarification:
└── "Why is this needed?"
└── "What hardware does this affect?"
└── "Can you explain the locking here?"
└── Respond with explanation in the thread
    If the explanation is important, also add it
    as a comment in the code or to the commit message

Request for changes:
└── "This approach is wrong because..."
└── "Use X instead of Y here"
└── "This needs a null check before dereferencing"
└── Fix it in v2 — do not argue without good reason

Reviewed-by (positive):
└── "Reviewed-by: Name <email>"
└── Reviewer has read the patch and it looks correct
└── Collect these — they help the maintainer decide

Tested-by (positive):
└── "Tested-by: Name <email>"
└── Someone tested it on real hardware
└── Very valuable — thank the tester

Acked-by (positive):
└── "Acked-by: Name <email>"
└── They agree with the approach
└── May not have reviewed every line

NAK (rejection):
└── "NAK" or "Nacked-by" on a patch
└── Formal objection to the patch
└── Must be addressed before merging
└── If you get a NAK, understand why before responding
```

### Automated Bot Feedback

```
kernel-bot@kernel.org (Intel 0-day bot):
└── Automated build testing across 100+ configs
└── If your patch breaks a build, you get an email
└── Fix it immediately — send v2 with the fix
└── Do not be embarrassed — even experienced
    contributors get 0-day reports

Example 0-day report:
  tree: git://git.kernel.org/...
  head: abc123def456
  commit: 789xyz (your commit hash)
  config: x86_64-allmodconfig
  compiler: gcc-12

  error: 'struct hub' has no member named 'kref'

This means your patch compiles on your config
but breaks on allmodconfig. Fix it.

syzbot:
└── Automated kernel fuzzer from Google
└── If your patch introduced a crash:
    syzbot will report it against your commit
└── The report includes a C reproducer
└── Fix it promptly — syzbot reports are public
└── Maintainer may revert your patch if unfixed
```

---

## How to Respond to Review Correctly

The way you respond to review feedback determines whether reviewers want to review your next patch. A good response builds trust. A defensive response burns it.

### Inline Reply — The Only Acceptable Format

```
You received this review:

> +       if (!hub)
> +               return;
>
> This does not handle the case where hub->dev is
> NULL but hub itself is not NULL. Can you add a
> check for hub->dev as well?

WRONG — top-posting your response:
I already checked this case. hub->dev cannot be
NULL if hub is not NULL because of how the
driver initialises it.

> +       if (!hub)
> +               return;
>
> This does not handle...

CORRECT — inline reply:
> This does not handle the case where hub->dev is
> NULL but hub itself is not NULL. Can you add a
> check for hub->dev as well?

hub->dev cannot be NULL when hub is not NULL —
the driver initialises hub->dev before setting hub
in usb_hub_probe(). If hub is valid, hub->dev is
valid. I can add a comment to make this clearer
if that would help.
```

### Addressing Each Comment

```
When a review has multiple comments:

> Comment 1: fix the null check

Response to comment 1 inline.

> Comment 2: rename this variable

Response to comment 2 inline.

> Comment 3: add a comment explaining the locking

Response to comment 3 inline.

---
I'll incorporate these in v2. Thanks for the review.

The reviewer should be able to scan down the thread
and see that each comment was addressed.
Do not batch all responses at the top.
```

### When You Disagree With Feedback

Disagreement is acceptable. Arguing without substance is not.

```
How to disagree constructively:

WRONG:
"I disagree. My approach is fine."
"This is already handled in the driver."
"You don't understand the hardware."

CORRECT:
"I considered this approach but did not use it
because [specific technical reason]. Here is why
the current approach is safe: [explanation].

If I'm missing something, can you point me to it?
Happy to change it if the reasoning is wrong."

Key elements:
├── Acknowledge the comment
├── Give a specific technical reason for your choice
├── Offer to be wrong — show you are open to correction
└── Ask for more detail if you genuinely do not understand

If the maintainer insists after your explanation:
└── Change it. They know the codebase.
└── You can note your concern in a comment:
    /* NOTE: this seems unnecessary but maintainer
     * requested it to protect against future changes */
└── Pick your battles — getting the patch merged
    is more important than being right about one detail
```

---

## Sending v2 After Review

Once you have addressed all feedback, incorporate it into your commit and send v2.

```
The v2 workflow:

Step 1 — Incorporate all feedback:
git add -p                  (stage your changes)
git commit --amend          (amend the commit)
# For a series: git rebase -i to edit specific commits

Step 2 — Verify everything is still correct:
./scripts/checkpatch.pl --strict *.patch
make -j$(nproc) drivers/subsystem/
# Confirm the fix still works

Step 3 — Generate v2 patches:
git format-patch \
  --subject-prefix="PATCH v2" \
  --cover-letter \
  -1 HEAD

Step 4 — Edit the cover letter or patch body:
Add "Changes in v2:" section below the --- separator
(for single patch) or in cover letter (for series)

Step 5 — Find the Message-ID of your v1:
Check lore.kernel.org or your sent mail
Look for: Message-ID: <...@...>

Step 6 — Send v2 as reply to v1:
git send-email \
  --in-reply-to="<v1-message-id@example.com>" \
  --to="maintainer@kernel.org" \
  --cc="list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  *.patch
```

### Writing a Good v2 Changelog

```
The changelog goes below the --- separator
(for single patch) or in the cover letter (for series).

Template:
  ---
  Changes in v2:
  - Fixed null check for hub->dev as suggested by
    Greg Kroah-Hartman
  - Added comment explaining why hub->dev is always
    valid when hub is not NULL
  - Fixed checkpatch.pl warning on line 234

  drivers/usb/core/hub.c | 6 ++++--

What to include:
├── What changed (specific, not vague)
├── Who suggested each change (attribution is polite)
└── If you did NOT make a suggested change,
    explain why here

What NOT to include:
├── "Fixed issues" (too vague)
├── "v2" (obviously)
└── Changes to unrelated things
    (if you fixed something else, mention it but
    be clear it was not from review feedback)
```

---

## Review Culture — The Unwritten Rules

### Why Linus Is Blunt — Structural Not Personal

New contributors are often shocked by the directness of kernel review feedback. Understanding why it exists makes it easier to accept.

```
Why kernel review is blunt:

The stakes:
├── The kernel runs on 3 billion Android devices
├── Medical devices, industrial controllers, satellites
├── A bug that ships cannot be silently patched
└── Soft feedback that does not convey the severity
    of a problem causes bad code to be merged

The volume:
├── Maintainers review hundreds of patches per week
├── Writing diplomatic responses to every bad patch
    would consume their entire day
└── Direct feedback is efficient feedback

The record:
└── Everything is public and permanent
└── "This is wrong" with a clear explanation teaches
    everyone who reads the thread, not just the author
└── Vague feedback helps nobody

This means:
└── "This is completely wrong, it will break on
    big-endian machines for the following reason..."
    is NOT a personal attack
└── It is a specific technical objection
└── Respond with technical substance, not emotion
└── The reviewer took time to explain why — respect that
```

### The CoC Change of 2018

```
What changed in 2018:

Before September 2018:
└── Linus Torvalds was known for extremely blunt,
    sometimes personal attacks on contributors
└── Examples of his emails were used as evidence
    that the kernel community was hostile to newcomers
└── Several maintainers had similar communication styles

September 3, 2018:
└── Linus took a break from kernel development
└── Wrote a public apology for his communication style
└── The kernel adopted a Code of Conduct (CoC)
    based on the Contributor Covenant

What changed:
├── Personal insults are no longer acceptable
│   "This is wrong" is fine
│   "You are an idiot" is not
├── Technical criticism remains direct and sharp
│   The standard for code quality did not change
│   The communication standard changed
└── Linus's own emails became more measured

What did NOT change:
└── The technical bar for patches
└── The directness of technical feedback
└── The expectation that you fix problems when asked
└── The culture of public, detailed technical critique
```

### Famous Review Exchanges — Context and Lesson

**Example 1 — Linus on a bad API:**

```
A developer submitted a patch with what Linus
considered a fundamentally broken API design.

Linus's response (paraphrased):
"This is complete garbage. The API is broken in
multiple ways: [detailed technical list]. This
design will cause problems in X, Y, and Z scenarios.
Here is how it should be done instead: [explanation]."

What to take from this:
├── The response identified specific technical problems
├── The response explained the alternative
└── "Garbage" refers to the code design, not the person
    The developer who submitted it was not stupid —
    Linus was communicating that this specific design
    was unacceptable and why

What the developer did:
└── Rewrote the API using Linus's guidance
└── The patch was eventually merged
└── The developer continued contributing
```

**Example 2 — Maintainer on missing null check:**

```
A driver patch missed a null check on an error path.

Maintainer response:
"You dereference 'dev' on line 45 without checking
if it is NULL. This will oops if dev_get() fails,
which it can under low memory conditions. Fix this
before I can apply it."

What to take from this:
└── Specific: line 45, specific function, specific condition
└── Not vague: "there might be a null pointer issue"
└── Not personal: "you always forget null checks"
└── This is useful feedback — it tells you exactly
    what to fix and why

Response:
"You're right — missed that path. I'll add the
null check and send v2. Thanks for catching it."
```

### How to Handle Criticism Without Quitting

```
The mental model that helps:

Your patch is not you.
Your code is not your identity.
A rejection of your patch is not a rejection of you.

The maintainer reviewed your code.
They found a problem.
They told you what it is.
That is a gift — not an attack.

Practical steps when feedback stings:

1. Read it twice before responding
   The first read activates defensiveness.
   The second read usually reveals the technical point.

2. Ask yourself: are they technically correct?
   Separate the tone from the content.
   If the content is correct, the tone does not matter.

3. If you are not sure if they are correct:
   Ask a clarifying question.
   "Can you help me understand why this is a problem?
   I thought X was safe because Y."

4. If they are correct:
   Say so. Fix it. Send v2.
   "You're right, I missed that. Fixed in v2."

5. If you genuinely disagree:
   Explain your technical reasoning calmly.
   Maintainers are not infallible.
   Sometimes they are wrong.
   Polite, substantive pushback is acceptable.

6. If you are frustrated:
   Do not respond immediately.
   Write the response. Do not send it.
   Wait a few hours. Reread.
   Usually the frustration fades and the
   technical substance remains.

The contributors who succeed long-term:
└── Separate their ego from their patches
└── Treat feedback as information, not judgment
└── Build a reputation for being easy to work with
└── Eventually receive lighter review because
    maintainers trust their judgment
```

---

## Realistic Timelines — By Subsystem

```
Subsystem           Typical time from send to accepted

drivers/staging/    1-4 weeks
                    Greg is responsive and patient
                    Simple patches often go quickly

USB                 2-6 weeks
                    Greg is busy but consistent

Networking          2-8 weeks
                    High volume — netdev gets 200+ emails/day
                    Automated testing helps speed things up

Memory management   4-12 weeks
                    Andrew is thorough — mm bugs are critical
                    Complex patches take much longer

ARM / ARM64         3-8 weeks
                    Depends on which part of ARM

x86 architecture    4-10 weeks
                    Core changes reviewed very carefully

New drivers         8-24 weeks
                    Especially for staging → mainline graduation
                    Multiple rounds of review expected

Security patches    1-4 weeks (if clearly correct)
                    Critical security fixes can be very fast
                    Controversial security approaches take longer

Factors that speed things up:
├── Clean patch that passes all checks immediately
├── Bug fix for a reproducible, well-documented issue
├── Reviewed-by from a trusted subsystem contributor
├── Tested-by from someone with the affected hardware
└── Patch submitted early in the development cycle

Factors that slow things down:
├── Needs multiple rounds of revision
├── Submitted just before merge window closes
├── Touches multiple subsystems (needs multiple ACKs)
├── Complex or controversial change
└── Maintainer is at a conference or on vacation
    (kernel conferences: LSFMM, Linux Plumbers, etc.)
```

---

## What Acceptance Looks Like

### How to Know Your Patch Was Merged

The kernel community does not always send an explicit "your patch was merged" confirmation. The most common experience is:

```
Your patch is accepted — what you might see:

Option 1 — Maintainer applies silently:
└── No email to you
└── Patch status changes to "Accepted" in Patchwork
└── Your commit appears in the maintainer's git tree

Option 2 — Maintainer confirms:
└── Short email: "Applied, thanks."
└── Or: "Queued for next merge window."

Option 3 — Automated confirmation:
└── Some subsystems (like netdev) send automated
    "patch applied" emails

How to verify your patch was applied:
└── Check the maintainer's public git tree:
    git clone <maintainer-tree-url>
    git log --oneline | grep "your subject"
    or
    git log --author="Your Name" --oneline
```

### What Happens After Acceptance

```
Your patch journey after acceptance:

Day 1-7:   Accepted into subsystem tree
           Appears in linux-next (daily integration)
           Automated testing by 0-day bot

Week 1-8:  Merge window opens
           Subsystem maintainer sends pull request
           Linus pulls subsystem tree into mainline
           Your patch is now in Linus's tree

Week 2-10: rc1 through rc7/rc8 testing cycle
           Millions of developers and testers
           are running a kernel that contains your patch
           If it causes a regression: you will hear about it

Week 8-10: Final release
           linux-6.X.tar.gz on kernel.org
           Your commit is now permanent in the
           official Linux kernel release

After release:
           Distributions begin packaging it
           Fedora, Ubuntu, Debian, Arch...
           Your fix reaches end users

If Cc: stable@vger.kernel.org:
           Backported to 6.(X-1).y, 6.(X-2).y...
           Reaches users on older stable kernels
           Potentially reaches Android devices
           and embedded systems on LTS kernels

Your name in the git log:
git log --oneline --author="Your Name" \
  -- drivers/usb/core/hub.c

You are now a Linux kernel contributor.
Your commit is permanent.
Anyone can see it with:
git log v6.X -- drivers/usb/core/hub.c
```

### The First Merge Is Just the Beginning

```
After your first patch is accepted:

What usually happens:
├── You have learned the process end-to-end
├── The second patch is significantly easier
├── You have a name that maintainers recognise
└── Your Reviewed-by on others' patches carries weight

What to do next:
├── Find the next thing to fix in the same subsystem
├── Review other contributors' patches
│   └── Helps the maintainer, builds your reputation
│   └── You learn what good patches look like
├── Start looking at harder problems
│   └── Not just style fixes — real bugs
└── Stay consistent
    Six patches per year for five years
    is better than sixty patches in one month
    and then disappearing

The contributors who become maintainers:
└── Did not plan to become maintainers
└── Just kept showing up, kept fixing things,
    kept reviewing other people's work
└── Eventually the community trusted them
    with more responsibility

That path starts with your first accepted patch.
```

---

> **You now know the complete Linux kernel contribution cycle — from understanding the system, to knowing who runs it, to every step of the process from finding a bug to seeing your fix ship in a release.**
> **Start from the beginning if anything was unclear, or jump to the specific file you need.**
> **→ [Back to the main index](../../README.md)**
