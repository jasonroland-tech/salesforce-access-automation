# Technical notes

The short version is in the [README](../README.md). This page keeps the details someone would need to repeat or maintain the setup.

## Okta setup

- **Custom attribute:** `salesforceAccess` (string). `Yes` means access; anything else means none.
- **Group rule:** `user.salesforceAccess == "Yes"` adds users to `App - Salesforce`.
- **Baseline access for that group:** Minimum Access profile, no role, no permission sets. The Salesforce admin handles everything beyond that.
- **Provisioning settings were left as they were:** Create Users on, Update User Attributes off, Deactivate Users on, Sync Password off. Because updates are off, moving someone into the new group does not change an existing account.
- **The weekly HR profile sync can't overwrite the attribute.** It uses Partial updates and never sends `salesforceAccess`.

## Migration approach

1. Reconciled Okta's Salesforce assignments against the Salesforce admin's user list and loaded the result into an Okta Workflows table (never into this repo).
2. Ordered the old groups from smallest to largest and gave each person a batch number, so everyone in a group was tagged before that group's turn came.
3. Ran the tagging flow one batch at a time and checked the table after each run.
4. Removed Salesforce from each old group, smallest first. The groups themselves were kept, since HR data still manages them.

| Batch | What it covered |
|---|---|
| 1 | People whose groups were removed later (tagged first) |
| 2 | 11 of the smallest groups (1 to 2 members each) |
| 3 | 5 groups (3 to 6 members) |
| 4 | 2 groups (about 11 members each) |
| 5 | 2 groups (about 27 members each) |
| 6 | 1 group (about 31 members) |

## Workflows

**Tagging flow:** for each row, skip it if already done, look up the user, set `salesforceAccess = Yes`, then mark the row `Yes` (or `ERROR` if something failed). The skip step makes reruns safe.

**Driver flow:** finds all rows for one batch number and runs the tagging flow for each. The largest batch (41 rows) took about five minutes.

**New hires:** Jira field → inbound onboarding flow saves it to the Flow State table → account creation flow sets the attribute on Create User → the group rule grants access.

## Gotchas worth knowing

| Gotcha | What to do |
|---|---|
| A group assignment with a blank Salesforce Profile fails to create accounts | Always set a Profile on the group assignment |
| Removing a user's direct (Individual) assignment deactivates them, even if they are in the group | Use **Convert assignments** on that specific user. Never "Convert all" |
| Update User in Okta Workflows defaults to settings that can erase profile fields | Use **Partial** mode, untick Pass Empty Values and Username, and set the field under Inputs |
| A flow can show "Success" even when individual runs failed | Check the table status, not just the run status |
| Removing Salesforce from a group removes it for everyone in that group | Confirm every member is tagged first |
| Exports taken right after a change can show old data | Verify on the group's live People tab |
| Importing the same CSV twice doubles the table rows | Check the row count after every import |
| An incomplete test ticket fails the trigger and looks like your change broke it | Fill in every field on test tickets |
| Renaming a table column breaks the cards that use it | Leave column names alone once they're mapped |
| Unnamed Workflows versions expire after 30 days | Name a version before editing a live flow |

## Remaining follow ups

- Test the "No" answer on a new hire ticket (the "Yes" path is tested)
- Check the first real new hires after go live
- Fix a failing Slack notification step on the account creation flow
- Delete legacy groups with no app assignments after a cooling off period
- Review whether the two contractor groups are still needed
