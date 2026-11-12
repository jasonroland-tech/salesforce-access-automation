# Problem and design

## Incident

A user moved from one division to another. Paylocity updated their department and division, Okta removed them from the old division group, and Salesforce deactivated their account about nine seconds later. The old group was their only Salesforce assignment, and the Salesforce app has *Deactivate Users* enabled.

Overlapping assignments would have hidden the problem, but relying on overlap is not a design.

## Root cause

Salesforce entitlement was an accidental side effect of HR-driven group membership. Nothing expressed "this person needs Salesforce" directly.

## Existing Salesforce provisioning (unchanged)

| Setting | Value |
|---|---|
| Create Users | On |
| Update User Attributes | Off |
| Deactivate Users | On |
| Sync Password | Off |
| Attribute mappings | All "Apply on: Create" |

Because Update User Attributes is off, changing a user's group assignment values does not overwrite an existing Salesforce account. Profile, role and permission sets stay under the Salesforce admin's control.

## Decisions

| Decision | Choice | Alternatives considered |
|---|---|---|
| Source of truth | Custom Okta attribute `salesforceAccess` (`Yes`) | Direct group membership by hand; a new Paylocity field |
| Intake | New Jira field "Salesforce Access Required" (Yes/No) on the onboarding request | Free-text request; Slack |
| Assignment | Group rule `user.salesforceAccess == "Yes"` into the existing `App - Salesforce` group | New group `APP - Salesforce - Users` (rejected: the existing group already had direct members and was assigned to the app) |
| Existing employees | One-time Okta Workflows helper flow that tags each person | Editing 150 profiles by hand or CSV import |
| Profile Sources | None configured, so no import can overwrite the attribute | n/a |
| Baseline access | Minimum Access profile, no role, no permission sets | Copying a division profile (rejected: over-grants) |

## What the group rule does and does not do

A group rule only manages members it added. Users who were added directly to `App - Salesforce` stay in it. Removing the attribute value removes only rule-added members.
