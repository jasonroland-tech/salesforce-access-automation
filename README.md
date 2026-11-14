# Salesforce Access Automation

Salesforce access used to depend on which department someone was in. When people changed divisions, Okta could shut off their Salesforce account by accident. This project gives Salesforce access its own switch so HR changes no longer break it.

**Tools:** Okta, Okta Workflows, Jira Service Management
**My role:** Designed, tested and rolled out the solution with the Salesforce admin
**Status:** ✅ Complete

## Results

- **148 existing users** moved to the new setup in six verified batches
- **21 department and division groups** removed from Salesforce, plus 3 legacy assignments
- **Zero accounts deactivated** and zero profiles changed during the migration
- **New hires are automatic:** one Yes/No answer on the Jira onboarding request is all it takes
- Salesforce went from dozens of group assignments to **5**: the new group, plus separate groups for external users and contractors

## The problem

Users got Salesforce through department and division groups that are driven by HR data from Paylocity. Nothing actually said "this person needs Salesforce." Access was just a side effect of their department.

That broke the moment HR data changed. One user moved divisions, Okta removed them from their old group, and because that was their only Salesforce assignment, their account was deactivated about nine seconds later. They still needed Salesforce.

## The fix

```
New hires                                   Existing employees
Jira request: "Salesforce Access Required?"  Okta Workflows tagging flow
        │                                           │
        └──────────►  salesforceAccess = Yes  ◄─────┘
                                │
                                ▼
                 Okta group: App - Salesforce
                                │
                                ▼
                        Salesforce access
```

One custom Okta attribute and one group now decide who gets Salesforce. Department and division changes no longer matter.

## Design choices

- **Reuse what exists.** The Jira field plugs into the current onboarding flow instead of building a new one.
- **Don't change Salesforce accounts.** Existing provisioning settings stay the same, so moving someone into the new group leaves their profile, role and permissions untouched.
- **Start with least privilege.** New accounts get a minimal profile, and the Salesforce admin grants anything beyond that.
- **Never remove the old access first.** Each person is moved to the new group and verified before any old assignment comes off.

## How it was rolled out

1. **Pilot.** Tested on one user, then five users across different roles. The Salesforce admin confirmed nothing changed on their accounts.
2. **Tag existing employees.** An Okta Workflows flow tagged 148 users in six batches, planned so everyone in a group was tagged before that group was touched. Each batch was checked before the next one ran.
3. **Remove old access.** Salesforce was removed from the old groups one at a time, smallest first, with the Salesforce admin checking after each stage.
4. **Connect new hires.** Added a required Jira field and mapped it through the existing onboarding flow into Okta, then tested it with a test hire.

## Key takeaways

- **Access should be granted on purpose**, not as a side effect of HR data.
- **Test small first.** Every surprise showed up in a one or five user test, not in production at scale.
- **Direct assignments are a trap.** Removing a user's individual Salesforce assignment deactivated them even though they were in the new group. The fix is Okta's Convert assignments option, one user at a time.
- **Trust the data, not the green checkmark.** Workflows reported success even when individual runs failed, so every batch was verified in the results table.
- **Change one thing at a time when debugging.** A test failure blamed on the new Jira field was really an incomplete test ticket.

Want the details? See [technical notes](docs/technical-notes.md).

*No employee data is stored in this repository.*
