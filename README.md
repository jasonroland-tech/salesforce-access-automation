# Salesforce Access Automation

Salesforce access used to depend on which department someone was in. When people changed divisions, Okta could shut off their Salesforce account by accident. This project gives Salesforce access its own switch so HR changes no longer break it.

**Tools:** Okta, Okta Workflows, Jira Service Management
**My role:** Designed, tested and rolled out the solution with the Salesforce admin

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

1. Tested the group rule on one user, then ran a five user pilot across different roles. The Salesforce admin confirmed nothing changed on their accounts.
2. Built an Okta Workflows flow that tags existing employees from a table. It skips anyone already done, so rerunning it is safe.
3. Next: tag everyone, connect the Jira field for new hires, then remove the old group assignments in small batches, smallest groups first.

## Status

| Step | State |
|---|---|
| Design, Okta setup, pilot | ✅ Done |
| Tagging flow | ✅ Built and tested |
| Tag existing employees | ⏳ Next |
| New hire automation | 📋 Planned |
| Remove old department group assignments | 📋 Planned |

## Key takeaways

- **Access should be granted on purpose**, not as a side effect of HR data.
- **Test small first.** Every surprise showed up in a one or five user test, not in production at scale.
- **Direct assignments are a trap.** Removing a user's individual Salesforce assignment deactivated them even though they were in the new group. The fix is Okta's Convert assignments option, one user at a time.
- **Check the defaults.** Okta Workflows' Update User card needs **Partial** mode, or it can wipe other profile fields.

Want the details? See [technical notes](docs/technical-notes.md).

*No employee data is stored in this repository.*
