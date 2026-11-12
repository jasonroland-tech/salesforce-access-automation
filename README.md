# Salesforce Access Automation

Decouple Salesforce access from Paylocity-driven department/division groups, using a Jira intake field, Okta Workflows, a custom Okta attribute and a dedicated Okta group.

**Owner:** IT Support / Identity (Okta, Okta Workflows, Jira Service Management)
**Status:** Design validated and piloted in production. Helper flow built and tested. Bulk tagging, new-hire wiring and group clean-up are in progress (see [Status](#status)).

## The problem

Salesforce access was granted indirectly: users received the Salesforce app through the Okta groups that Paylocity drives (department and division). When someone changed division, Okta removed them from their old group. If that was their only Salesforce assignment, Okta's *Deactivate Users* setting deactivated the Salesforce account within seconds, even though the person still needed Salesforce.

## The solution

```
Jira onboarding ticket                         Existing employees
 "Salesforce Access Required" (Yes/No)          (one-time bulk tag)
        |                                              |
        v                                              v
 Okta Workflows (existing onboarding flow)      Okta Workflows helper flow
        |                                              |
        +----------------> salesforceAccess = "Yes" <--+     (custom Okta attribute)
                                   |
                                   v
              Group rule: user.salesforceAccess == "Yes"
                                   |
                                   v
                   Okta group: App - Salesforce  (long-term group)
                                   |
                                   v
                       Salesforce app assignment
```

Salesforce access now depends on one attribute and one group instead of whatever department or division someone sits in.

## Design principles

- Reuse the existing onboarding automation. No new onboarding workflow.
- Preserve the current Salesforce provisioning settings (Create Users on, Update User Attributes off, Deactivate Users on, Sync Password off).
- The Salesforce admin keeps ownership of roles, profiles and permission sets. Okta only controls *whether* a user has access.
- The long-term group is a least-privilege baseline (Minimum Access profile, no role, no permission sets).
- Migrate existing users gradually, verify each step, and never remove an old assignment until the new one is proven.

## Repository layout

| File | Contents |
|---|---|
| [docs/01-problem-and-design.md](docs/01-problem-and-design.md) | Incident, root cause, decisions and alternatives |
| [docs/02-okta-configuration.md](docs/02-okta-configuration.md) | Attribute, group rule, app assignment settings |
| [docs/03-validation-and-pilots.md](docs/03-validation-and-pilots.md) | Tests run and what they proved |
| [docs/04-migration-runbook.md](docs/04-migration-runbook.md) | Bulk tagging and group-removal procedure |
| [docs/05-helper-flow-spec.md](docs/05-helper-flow-spec.md) | Okta Workflows helper flow, card by card |
| [docs/06-new-hire-integration-plan.md](docs/06-new-hire-integration-plan.md) | Jira to Workflows to Okta mapping for new hires |
| [docs/07-lessons-learned.md](docs/07-lessons-learned.md) | Gotchas worth knowing |

## Status

| Phase | State |
|---|---|
| Design and Okta configuration (attribute, rule, group) | Done |
| Overlap test and five-user pilot | Done, passed |
| Helper flow (set `salesforceAccess = Yes`) | Built, single-user tests passed |
| Bulk tagging of existing employees | Next |
| Jira field to Create User mapping for new hires | Planned |
| Remove old department/division Salesforce assignments, in batches | Planned |

## Data handling

No employee names, emails or exports are stored in this repository. Migration lists contain personal data and live only in the Okta Workflows table used for the run. `*.csv` is git-ignored.
