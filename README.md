# Salesforce Access Automation

Salesforce access used to depend on which department someone was in. When people changed divisions, Okta could shut off their Salesforce account by accident. This project gives Salesforce access its own switch so HR changes no longer break it.

**Tools:** Okta, Okta Workflows, Jira Service Management

## The problem

Users got Salesforce through department and division groups driven by HR data. When someone moved divisions, Okta removed them from the old group, and if that was their only Salesforce assignment, the account was deactivated within seconds.

## The fix

```
Jira request: "Salesforce Access Required?"  ──►  salesforceAccess = Yes
                                                         │
                                                         ▼
                                           Okta group: App - Salesforce
                                                         │
                                                         ▼
                                                 Salesforce access
```

One custom Okta attribute and one group now decide who gets Salesforce. Department changes no longer matter.

## How it was rolled out

1. Tested on one user, then a five user pilot, before touching anyone else
2. Built an Okta Workflows flow to tag existing employees safely (rerunning it changes nothing)
3. Next: tag everyone, connect the Jira field for new hires, then remove the old group assignments in small batches

## Status

| Step | State |
|---|---|
| Design, Okta setup, pilot | ✅ Done |
| Tagging flow | ✅ Built and tested |
| Tag existing employees | ⏳ Next |
| New hire automation and old group cleanup | 📋 Planned |

## Key takeaways

- Access should be granted on purpose, not as a side effect of HR data.
- Test small first. Every surprise showed up in a one or five user test.
- Okta Workflows' Update User card needs **Partial** mode, or it can wipe profile fields.

Want the details? See [technical notes](docs/technical-notes.md).

*No employee data is stored in this repository.*
