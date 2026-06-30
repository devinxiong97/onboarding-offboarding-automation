# On/Offboarding Runbook (Example)

> Fabricated example data throughout. Real runbook substitutes live org names and tooling.

## Roles

- **HR** submits structured requests via Slack workflow forms.
- **IT/ops** (the operator) reviews agent action-cards and executes privileged steps.
- **Automation agent** detects, parses, triages, and drafts — never executes destructive steps.

---

## Onboarding

**Trigger:** a new-hire form posted to `#newhires-<org>`.

Example form:
```
New Hire Name: Jane Doe
Title: Senior Backend Engineer
Hire Type: Contractor
Location: Remote — Portugal
Start Date: 2026-07-01
Manager: Alex Rivera
Company: OrgA
Personal Email: jane.personal@example.com
Create Google Account: Yes
Invite To Slack: Yes
Send Welcome Email: Yes
```

**Agent produces an action-card:**
1. Parsed summary + ⚠️ flags for any missing required fields (e.g. department, manager email).
2. A drafted welcome email personalized with name + start date.
3. ⚠️ flag if start date is today/past.

**Automated on creation (per company → its own GAM profile/domain/SKU):**
1. Create Google account (`gam create user …`), force password change on first login.
2. Assign the company's Workspace license (SKU per org).
3. Set directory fields for tracking: **title**, **manager** (relation), and **start date** (custom `Employment.StartDate` schema).
4. Add to the company's **all-staff group** (`all@<domain>`) — idempotent.
5. Generate the **welcome-email draft** (work email + temp password → Slack → message manager).
6. **Remote employees:** add a **1Password** setup step to the welcome email; the 1Password invite itself is sent manually (no 1Password API connected).

**Operator finishes:** review/send the welcome draft, send the Slack invite, and (remote) send the 1Password invite.

---

## Offboarding

**Trigger:** an offboarding form posted to `#offboarding`.

Example form:
```
Employee Name: John Smith
Company: OrgA
Termination Date: 2026-06-30
Device: MacBook Pro 14"
Remove Access to Google: Yes
Remove Access to Slack: Yes
Google Account Deletion (requires written approval): Hold
Drive transfer to: Alex Rivera
```

**Agent produces an action-card:**
- Termination date (⚠️ if today/past)
- Access-removal checklist
- Device-recovery + return-address line
- Drive-transfer recipient (or ⚠️ "not specified")
- ⚠️ **APPROVAL REQUIRED** banner if deletion = Yes

**Operator executes — in this order (never delete first):**
1. **Suspend** Google account (reversible access-cut):
   ```
   gam update user john.smith@example.com suspended on
   ```
   > Confirm the exact `primaryEmail` first — watch for look-alike accounts.
2. **Archive Drive** into the Shared Drive (before any deletion):
   ```
   gam user john.smith@example.com transfer drive <recipient> ...
   ```
3. **Slack:** deactivate the seat (manual on standard Business plan; rides on Google SSO if enabled).
4. **Deletion** (only with written approval) — always manual, never automated.

## Guardrails

- Destructive actions require explicit target confirmation.
- Look-alike account detection before suspend/delete (e.g. `j.doe@` vs `j.doer@`).
- Reversible-first: suspend over delete.
- Capability honesty: steps that aren't automatable on the current plan are documented as manual, not faked.
