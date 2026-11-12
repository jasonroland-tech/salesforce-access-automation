# Technical notes

The short version is in the [README](../README.md). This page keeps the details someone would need to repeat or maintain the setup.

## Okta setup

- **Custom attribute:** `salesforceAccess` (string). `Yes` means access; anything else means none.
- **Group rule:** `user.salesforceAccess == "Yes"` adds users to `App - Salesforce`.
- **Baseline access for that group:** Minimum Access profile, no role, no permission sets. The Salesforce admin handles everything beyond that.
- **Provisioning settings were left as they were:** Create Users on, Update User Attributes off, Deactivate Users on. Because updates are off, moving someone into the new group does not change an existing account.

## Rollout plan

1. Load the list of current Salesforce users into an Okta Workflows table (never into this repo).
2. Run the tagging flow on one user, verify, then a few, then everyone.
3. Remove Salesforce from old department groups one group at a time, smallest first, with the Salesforce admin checking after each batch.
4. Add the Jira field to the existing onboarding flow so new hires are tagged automatically.

## Tagging flow

For each row: skip it if already done, look up the user, set `salesforceAccess = Yes`, then mark the row `Yes` (or `ERROR` if something failed). The skip step makes reruns safe.

## Gotchas worth knowing

| Gotcha | What to do |
|---|---|
| A group assignment with a blank Salesforce Profile fails to create accounts | Always set a Profile on the group assignment |
| Removing a user's direct (Individual) assignment deactivates them, even if they are in the group | Use **Convert assignments** on that specific user. Never "Convert all" |
| Update User in Okta Workflows defaults to settings that can erase profile fields | Use **Partial** mode, untick Pass Empty Values and Username, and set the field under Inputs |
| A flow can show "Success" even when its error branch ran | Check the table status, not just the run status |
| Removing Salesforce from a group removes it for everyone in that group | Confirm every member is tagged first |
