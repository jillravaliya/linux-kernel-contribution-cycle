# Sending the Patch

---

> Everything is ready. Your patch is correct, your commit message is complete, your checks all pass, you know exactly who to send to. Now the moment where most new contributors make their biggest mistake — the actual sending. Wrong email client corrupts the patch. Missing flags produce malformed threads. Forgetting a cover letter makes a 5-patch series confusing to review. Sending v2 incorrectly breaks the thread history.

**This file covers the complete sending process — from configuring git send-email correctly to sending a patch series with a cover letter to handling v2 and v3 revisions properly.**

---

## What's Inside This File

```
13-sending-the-patch/
│
├── Why git send-email — and not your email client
├── git send-email Setup — complete from scratch
│   ├── Gmail
│   ├── Outlook / Office 365
│   └── Custom SMTP
├── Sending a Single Patch
│   ├── The exact command
│   └── What to verify before confirming
├── Sending a Patch Series
│   ├── What a series is and when you need one
│   ├── The cover letter — when and how to write it
│   └── Threading — how the series stays together
├── What a Correct Patch Email Looks Like
│   ├── Headers
│   └── Body and diff format
├── Sending v2, v3 — Revised Patches
│   ├── When to send a new version
│   ├── The changelog — what changed between versions
│   └── How to reference the previous version
└── Common Sending Mistakes
```

---

## Why git send-email — Not Your Email Client

This is not optional. Use `git send-email`. Do not use Gmail, Outlook, Thunderbird, Apple Mail, or any other email client to send patches directly.

```
What email clients do wrong:

Gmail web interface:
├── Converts tabs to spaces (breaks the diff)
├── Wraps long lines (breaks the diff)
├── Adds HTML parts (breaks the patch format)
└── Sometimes modifies whitespace invisibly

Outlook:
├── Always adds HTML/RTF formatting
├── Converts indentation
├── Adds signature formatting that breaks patches
└── Most reliable way to have your patch rejected

Thunderbird:
├── Can work if configured perfectly
├── Easy to accidentally send HTML
└── git send-email is safer and simpler

Apple Mail:
├── Adds rich text formatting
└── Converts indentation in code blocks

git send-email:
├── Sends exactly what git format-patch produces
├── Always plain text
├── Preserves all whitespace exactly
├── Handles threading automatically
├── Knows the exact format maintainers expect
└── Used by every serious kernel contributor
```

---

## git send-email Setup — Complete From Scratch

You need to configure `git send-email` once. After that it works for every patch you ever send.

### Installing git send-email

```
On some systems git send-email is a separate package:

Debian / Ubuntu:
sudo apt-get install git-email

Fedora:
sudo dnf install git-email

Arch:
sudo pacman -S git   (included in main package)

Verify it is installed:
git send-email --help
(if you see the help page, it is installed)
```

### Gmail Setup

```
Step 1 — Enable App Passwords in your Google account:
└── Go to myaccount.google.com/security
└── Enable 2-Step Verification (required for app passwords)
└── Go to myaccount.google.com/apppasswords
└── Create a new app password
    App: Mail
    Device: Other (custom name) → "git send-email"
└── Copy the 16-character password shown

Step 2 — Add to ~/.gitconfig:

[sendemail]
    smtpServer = smtp.gmail.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = youraddress@gmail.com

Step 3 — Store credentials (optional but convenient):
git config --global \
  sendemail.smtpPass "your-16-char-app-password"

Or use git credential store:
git config --global credential.helper store
(git will prompt once and remember)

Step 4 — Test:
git send-email \
  --dry-run \
  --to="yourself@gmail.com" \
  --subject="test" \
  --body="test body"
```

### Outlook / Office 365 Setup

```
[sendemail]
    smtpServer = smtp.office365.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = youraddress@company.com

Note: Many corporate Outlook setups block
third-party SMTP access. If this does not work,
ask your IT department if SMTP AUTH is enabled.

Alternative: use a personal email for kernel work.
Many contributors use a separate Gmail or
personal domain for open source contributions.
```

### Custom SMTP / Personal Domain

```
[sendemail]
    smtpServer = mail.yourdomain.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = you@yourdomain.com
    smtpPass = yourpassword

For port 465 (SMTPS instead of STARTTLS):
    smtpServerPort = 465
    smtpEncryption = ssl
```

### Additional Recommended Settings

```
Add these to the [sendemail] section:

[sendemail]
    # ... SMTP settings above ...

    # Always ask for confirmation before sending:
    confirm = always

    # Do not thread patches in a series as
    # replies to each other (thread under cover letter):
    chainreplyto = false

    # Do not CC yourself on patches you send:
    suppresscc = self

    # Suppress CC to addresses already in To:
    suppresscc = bodycc

    # Show header of each email before sending:
    annotate = yes
```

---

## Sending a Single Patch

### Generating the Patch File

```
# Create patch for your last commit:
git format-patch -1 HEAD

# Output: 0001-subsystem-fix-the-thing.patch

# Verify it looks right:
cat 0001-subsystem-fix-the-thing.patch

# Run checkpatch one more time on the patch file:
./scripts/checkpatch.pl --strict \
  0001-subsystem-fix-the-thing.patch
```

### The Exact Send Command

```
git send-email \
  --to="maintainer@example.com" \
  --cc="subsystem-list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  0001-subsystem-fix-the-thing.patch
```

### What Happens When You Run It

```
git send-email prompts you with:

Who should the emails appear to be from? [Your Name <you@email.com>]
(Press Enter to accept)

Who should the emails be sent to? maintainer@example.com
(Press Enter to accept, or add more)

Message-ID to be used as In-Reply-To for the
first email? (Leave blank unless sending v2)
(Press Enter to leave blank for a new thread)

(Patch display)
Send this email? ([y]es|[n]o|[q]uit|[a]ll)

Type 'y' to send.
```

### What to Verify Before Typing 'y'

```
Before confirming the send:

[ ] From: shows your real name and correct email
[ ] To: shows the maintainer (not a list)
[ ] Cc: shows the subsystem list AND linux-kernel
[ ] Subject: starts with [PATCH] not [PATCH 0/1]
    (0/1 would mean it is a cover letter)
[ ] The message body shows your commit message
[ ] The diff is present below the --- separator
[ ] No obvious corruption (tabs look like tabs)

If anything looks wrong: type 'q' to quit
Fix the issue and run again.
```

---

## Sending a Patch Series

A patch series is multiple related patches sent together. They are threaded in email so reviewers can see them as a unit.

### When You Need a Series

```
Use a series when:
├── Your fix requires more than one commit
│   (e.g., first patch adds infrastructure,
│    second patch uses it)
├── You are adding a new feature that touches
│   multiple subsystems or files logically
├── You are doing a cleanup that has multiple steps
│   (e.g., patch 1: rename functions,
│    patch 2: update callers)
└── A maintainer asked you to split your patch

Do NOT use a series when:
└── You have one logical change
    Even if it touches many files, one logical
    change = one patch
```

### Generating a Series

```
# Last 3 commits as a series with cover letter:
git format-patch --cover-letter -3 HEAD

# Output:
0000-cover-letter.patch
0001-subsystem-first-change.patch
0002-subsystem-second-change.patch
0003-subsystem-third-change.patch

# The cover letter is where you explain the series.
# Edit it before sending:
$EDITOR 0000-cover-letter.patch
```

### The Cover Letter — When and How to Write It

```
The cover letter template looks like:

From: Your Name <you@email.com>
Subject: [PATCH 0/3] subsystem: series description

*** BLURB HERE ***

Your Name (3):
  subsystem: first change description
  subsystem: second change description
  subsystem: third change description

 file_one.c | 5 +++++
 file_two.c | 3 +--
 3 files changed, 6 insertions(+), 2 deletions(-)

Replace *** BLURB HERE *** with:
```

What to write in the cover letter body:

```
A good cover letter explains:

1. What problem this series solves
   (the big picture — individual patches explain details)

2. Why multiple patches are needed
   (not just one patch)

3. How to review the series
   (read patch 1 first, it sets up for patch 2)

4. Any dependencies or prerequisites
   (does this series depend on another series?)

5. Testing summary for the whole series

Example cover letter body:

This series fixes the race condition in the USB hub
driver that causes use-after-free on hot-unplug.

Patch 1 adds the kref-based lifetime tracking that
the fix depends on. Patch 2 uses it to fix the race.
They are split because the kref infrastructure is
independently useful and keeping them separate makes
each patch easier to review.

Tested on x86-64 and ARM64 with USB 2.0 and 3.0
hubs. Hot-unplug under load no longer produces oops.
```

### When a Cover Letter Is Optional

```
Cover letter is REQUIRED:
├── Series of 3 or more patches
├── New driver or significant new feature
└── Complex multi-subsystem changes

Cover letter is OPTIONAL:
└── Two closely related patches that are self-evident

Cover letter is NOT needed:
└── Single patch (never use a cover letter for one patch)
```

### Threading — How the Series Stays Together

```
How a correctly threaded series looks in email:

[PATCH 0/3] subsystem: fix the race condition  ← cover letter
  [PATCH 1/3] subsystem: add kref tracking      ← patch 1
  [PATCH 2/3] subsystem: fix the race           ← patch 2
  [PATCH 3/3] subsystem: add documentation      ← patch 3

All patches are replies to the cover letter.
The cover letter is the top-level thread.
Reviewers can see the full series in one thread.
```

How to achieve this threading automatically:

```
git send-email handles this with chainreplyto = false
in your gitconfig (set this earlier).

With chainreplyto = false:
└── Patches 1, 2, 3 are all replies to patch 0
└── Correct — flat threading under cover letter

With chainreplyto = true (wrong):
└── Patch 2 replies to patch 1
└── Patch 3 replies to patch 2
└── Deep nesting — harder to review
└── Make sure this is set to false in gitconfig
```

### Sending the Complete Series

```
git send-email \
  --to="maintainer@example.com" \
  --cc="linux-list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  0000-cover-letter.patch \
  0001-subsystem-first-change.patch \
  0002-subsystem-second-change.patch \
  0003-subsystem-third-change.patch

Or use a glob:
git send-email \
  --to="maintainer@example.com" \
  --cc="linux-list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  *.patch
```

---

## What a Correct Patch Email Looks Like

A patch email received by the maintainer should look exactly like this:

```
From: Your Name <you@example.com>
To: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
Cc: linux-usb@vger.kernel.org,
    linux-kernel@vger.kernel.org
Subject: [PATCH] usb: fix null pointer in hub_disconnect
Date: Mon, 10 Mar 2025 14:23:45 +0000
Message-ID: <20250310142345.12345-1-you@example.com>

The hub driver dereferences hub->dev in hub_event()
after hub_disconnect() may have freed the hub
structure. Fix this by checking hub for NULL before
dereferencing.

Fixes: 3f8a2d1b9e4c ("usb: hub: rework disconnect")
Cc: stable@vger.kernel.org
Signed-off-by: Your Name <you@example.com>
---
 drivers/usb/core/hub.c | 4 ++++
 1 file changed, 4 insertions(+)

diff --git a/drivers/usb/core/hub.c b/drivers/usb/core/hub.c
index 7a3f9c1..2b84e5d 100644
--- a/drivers/usb/core/hub.c
+++ b/drivers/usb/core/hub.c
@@ -1823,6 +1823,10 @@ static void hub_event(
        struct usb_hub *hub =
                container_of(work, struct usb_hub, events);

+       if (!hub)
+               return;
+
        /* process hub events */
```

What the maintainer does with this:

```
git am < the_email.eml

Result: patch applied to their local tree
        commit message preserved exactly
        your authorship preserved
        ready to push to their tree
```

---

## Sending v2, v3 — Revised Patches

After receiving review feedback, you will fix your patch and send a new version. This is normal. Most patches go through at least v2.

### When to Send a New Version

```
Send v2 when:
├── Reviewer asked you to change something
├── You found a bug in your own patch after sending
├── Another reviewer raised a valid concern
└── The maintainer asked for changes

Do NOT send v2 when:
├── You are waiting for a response (be patient)
├── You want to "bump" your patch for attention
│   └── Wait 2-4 weeks, then send a polite ping
└── Nothing has changed
```

### Preparing v2

```
Step 1 — Make the changes to your code:
git add -p   (stage changes carefully)
git commit --amend   (amend the commit)

Or for multiple commits:
git rebase -i HEAD~3   (interactive rebase)
(edit the relevant commits)

Step 2 — Generate new patch files:
git format-patch --cover-letter -1 HEAD
# or -3 for a series

Step 3 — Update the version number:
# The subject line now needs [PATCH v2]:
# git format-patch does NOT do this automatically
# You need to specify it:

git format-patch \
  --subject-prefix="PATCH v2" \
  --cover-letter \
  -3 HEAD

# Output:
0000-cover-letter.patch   (subject: [PATCH v2 0/3]...)
0001-...patch             (subject: [PATCH v2 1/3]...)
```

### The Changelog — What Changed Between Versions

The changelog explains to the reviewer what changed since the last version. It goes in the cover letter (for a series) or below the `---` separator (for a single patch).

```
Where the changelog goes:

For a single patch — after the --- separator:
  Signed-off-by: Your Name <you@example.com>
  ---
  Changes in v2:
  - Fixed null check as suggested by Greg Kroah-Hartman
  - Added comment explaining why lock is taken here
  - Fixed checkpatch.pl WARNING on line 234

  drivers/usb/core/hub.c | 4 ++++

For a series — in the cover letter body:
  [The cover letter explanation...]

  Changes in v2:
  - Patch 1: Renamed kref_get to hub_get as suggested
  - Patch 2: Added the missing unlock on error path
  - Patch 3: Added Reviewed-by from Alan Stern

  Changes in v1:
  - Initial submission
```

What makes a good changelog entry:

```
Good changelog:
- Fixed null check before hub->dev dereference
  (suggested by Greg Kroah-Hartman)
- Moved the kref_get() call earlier in the function
  to avoid the race window (suggested by Alan Stern)
- Added documentation comment explaining lock ordering

Bad changelog:
- Fixed issues
- Updated code
- v2
```

The changelog tells the reviewer what to look at in the diff. Without it, they review everything again from scratch. With a clear changelog, they can focus on the changed parts.

### Referencing the Previous Version

When you send v2, reference v1 so the thread history is connected:

```
# Find the Message-ID of your v1 cover letter
# (or single patch if no series):
# Check your sent mail or find it on lore.kernel.org

# Send v2 as a reply to v1:
git send-email \
  --in-reply-to="<20250301120000.12345-0-you@example.com>" \
  --to="maintainer@example.com" \
  --cc="linux-list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  *.patch

The --in-reply-to value is the Message-ID from your
v1 cover letter (the 0/N patch, or the single patch).

Finding the Message-ID:
├── Your sent mail folder → look at email headers
│   Message-ID: <this-is-what-you-want@example.com>
└── lore.kernel.org → find your v1 → the URL path
    contains the message ID

Result in the thread:
[PATCH 0/3] original series          ← v1
  [PATCH 1/3] patch 1
  [PATCH 2/3] patch 2
  [PATCH v2 0/3] revised series      ← v2 (reply to v1)
    [PATCH v2 1/3] patch 1 revised
    [PATCH v2 2/3] patch 2 revised
```

---

## Common Sending Mistakes

```
Mistake 1: Sending from an email client instead of
           git send-email
Result: Tabs converted to spaces, patch is corrupt
Fix: Always use git send-email

Mistake 2: Forgetting --in-reply-to for v2
Result: v2 starts a new thread, history is broken
Fix: Always reference v1 with --in-reply-to

Mistake 3: Not running --dry-run first
Result: Sent to wrong people, subject wrong,
        something embarrassing in the patch
Fix: Always dry-run, always read the output

Mistake 4: Sending the cover letter for a single patch
Result: [PATCH 0/1] and [PATCH 1/1] — looks amateur
Fix: Cover letters are for series of 2+ patches only

Mistake 5: chainreplyto = true (wrong threading)
Result: Patches nest inside each other
        Reviewers see a deeply nested thread
Fix: Set chainreplyto = false in gitconfig

Mistake 6: Subject says [PATCH] but has version number
           in the body
Result: Reviewers cannot tell this is v2 from the
        subject line — they may re-review v1 content
Fix: Use --subject-prefix="PATCH v2" for v2 patches

Mistake 7: Changelog missing from v2
Result: Reviewer reviews entire patch again from scratch
        They may miss that you addressed their feedback
Fix: Always include a Changes in vN: section

Mistake 8: Sending to linux-kernel@vger.kernel.org only
Result: Subsystem maintainer does not see your patch
Fix: ALWAYS use get_maintainer.pl to find the right
     subsystem list and maintainer

Mistake 9: CC'ing too many random people
Result: Annoyed developers who get irrelevant email
Fix: Only CC people and lists that get_maintainer
     tells you to — do not add people from git log

Mistake 10: Sending multiple unrelated patches in one email
Result: Confusion — is this a series? Are these related?
Fix: One logical change per patch
     Related changes in a series with a cover letter
```

---

## Quick Reference — The Full Send Command

```
Single patch:
git format-patch -1 HEAD
./scripts/checkpatch.pl --strict *.patch
git send-email \
  --to="maintainer@kernel.org" \
  --cc="list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  0001-*.patch

Patch series:
git format-patch --subject-prefix="PATCH" \
  --cover-letter -N HEAD
# Edit 0000-cover-letter.patch
./scripts/checkpatch.pl --strict [0-9]*.patch
git send-email \
  --to="maintainer@kernel.org" \
  --cc="list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  *.patch

Revised patch (v2):
git format-patch --subject-prefix="PATCH v2" \
  --cover-letter -N HEAD
# Edit 0000-cover-letter.patch (add changelog)
./scripts/checkpatch.pl --strict [0-9]*.patch
git send-email \
  --in-reply-to="<message-id-of-v1@example.com>" \
  --to="maintainer@kernel.org" \
  --cc="list@vger.kernel.org" \
  --cc="linux-kernel@vger.kernel.org" \
  *.patch
```

---

> **Your patch is sent. Now the wait begins — but the work is not done. Next: what happens after you send, how to track your patch, how to respond to review, how to send v2 correctly, and what silence actually means.**
> **→ [14-after-you-send](../14-after-you-send/README.md)**
