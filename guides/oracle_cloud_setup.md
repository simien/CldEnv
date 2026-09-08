# Oracle Cloud Setup Guide

Everything between "no Oracle account" and "an Ubuntu ARM64 server you can SSH into." Once you're at that point, go back to the main [README](../README.md)'s Quick Start to deploy CldEnv itself.

---

## 1. Always Free Limits (Read This First)

Oracle's Always Free Ampere A1 (ARM) allocation was cut in 2026. Per [Oracle's own current documentation](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier.htm):

> To continue using your existing Arm-based instances as an Always Free user... ensure that you have no more than **2 OCPUs and 12 GB of memory in total** across all Ampere A1 instances in your tenancy.

This is down from the 4 OCPU / 24 GB figure still widely quoted around the internet (and still shown in some parts of the Oracle console, inconsistently). The limit applies **per account, not per instance** -- provisioning beyond it doesn't get billed on an Always Free account, but the instance gets disabled, then deleted after 30 days if you don't resize or upgrade to a paid account.

**What this means for CldEnv:**
*   Provision at **2 OCPU / 12 GB** to stay unambiguously inside Always Free, with zero billing risk.
*   12 GB is noticeably tighter than 24 GB for this stack's ~14 services, especially `ollama` (a few GB once a model is loaded), `browserless` (a full headless Chrome instance), and Windmill's three containers if you kept those over n8n. If you're tight on memory, the first things worth dropping are `changedetection`/`browserless` (a self-contained pair, remove both together) -- see the Architecture section in the README for what everything actually is.
*   If your account still shows the option to provision 4 OCPU / 24 GB, that's real (enforcement has been inconsistent), but it's not guaranteed to stay that way, and Oracle's own support has given conflicting answers about billing for accounts that exceed 2/12. Don't build a plan around keeping it.

---

## 2. Create an Oracle Cloud Account

1.  Go to [oracle.com/cloud/free](https://www.oracle.com/cloud/free/) and sign up. Requires a phone number and a card for identity verification -- the card is not charged for Always Free resources.
2.  **Pick your region carefully during signup -- it cannot be changed afterward** without creating a new account. Region choice matters for step 4 below (capacity availability varies a lot by region).
3.  Once your account is provisioned, log into the OCI Console.

---

## 3. Create the Instance

1.  In the Console, go to **Compute > Instances > Create Instance**.
2.  **Image**: Canonical Ubuntu 24.04 (ARM). Confirm "Always Free Eligible" is checked/shown for your selection.
3.  **Shape**: click **Change Shape** > **Ampere** > `VM.Standard.A1.Flex`. Set **2 OCPUs** and **12 GB memory** (see Section 1 -- don't default to whatever the slider's maximum is).
4.  **Boot volume**: up to 200 GB is included free; the default is smaller. Set it to 200 GB now if you want the headroom (Docker images, logs, backups) -- resizing later is possible but more hassle than setting it correctly here.
5.  **SSH keys**: let the console generate a key pair and **download the private key immediately** -- it's only offered once. Save it somewhere real; you'll need it for every SSH connection from here on.
6.  Click **Create**. If this succeeds immediately, skip to Section 5.

---

## 4. If You Get "Out of Host Capacity"

This is the single most common blocker for a new Always Free ARM instance -- Oracle's free Ampere capacity is genuinely oversubscribed in popular regions. It's not a mistake on your end.

**What actually helps, in order of effort:**

1.  **Try a different Availability Domain.** If your region has more than one (visible in the instance-creation form's Placement section), cycle through all of them on each retry -- capacity opens and closes per-AD, independently.
2.  **Region matters a lot, and you already picked yours in Section 2.** US regions (Ashburn, Phoenix) are frequently congested for hours or days at a time. EU/APAC regions (Frankfurt, Singapore, Tokyo) have historically provisioned within minutes far more often. If you're stuck and haven't started using the account for anything else yet, a fresh signup in a less-congested region is a legitimate option.
3.  **Just retry.** Capacity frees up when other users terminate instances; clicking Create again periodically works, especially outside US business hours.
4.  **Automation exists.** Several community scripts poll the OCI API and create the instance automatically the moment capacity appears (search "oracle cloud out of host capacity script" for current options). Useful if you're going to be retrying for more than a few minutes, but read whatever script you use before running it against your account -- it needs your API credentials.

None of this is CldEnv-specific -- it's the same capacity lottery every Oracle free-tier ARM tutorial runs into.

---

## 5. Connect and Verify

```bash
ssh -i /path/to/your-key.key ubuntu@<your-instance-public-ip>
```

Once connected, confirm you actually got what you asked for:

```bash
nproc          # should show 2 (or whatever you provisioned)
free -h        # should show ~12 GB total (or whatever you provisioned)
```

From here, continue with the main [README](../README.md)'s Quick Start (Requirements section onward) -- including the Security List / Network Security Group step, which is separate from anything covered here and easy to miss.
