# Validation and pilots

Every step was tested on real accounts before the next one, starting with the smallest possible population.

## Tests run

| # | Test | Result |
|---|---|---|
| 1 | Test group with the rule; attribute `Yes` adds the user, clearing it removes them | Passed |
| 2 | Create a Salesforce account via a test group (to confirm baseline provisioning) | Failed first with a blank-Profile error, passed after setting a Profile on the group assignment |
| 3 | Overlap test (one admin): move from a group with a different Profile, Role and Permission Sets to `App - Salesforce` | Passed. The Salesforce admin confirmed nothing changed on the configured account. |
| 4 | Five-user pilot: same move, five real users across different roles | Passed in Okta and Salesforce |
| 5 | Helper flow on one individually-assigned user | Tagged correctly, then see the incident below |
| 6 | Helper flow guard: re-run the same row | Passed. The flow stopped at Continue If and changed nothing. |

## What the pilots proved

- Moving a user from a division group to `App - Salesforce` leaves a configured Salesforce account unchanged (Profile, Role, Permission Sets).
- The attribute and rule add users to the group without touching other assignments.
- Re-running a finished row is safe.

## Incident during testing: individual assignments

One user had a direct (Individual) assignment to Salesforce rather than a group assignment. After tagging them and adding them to `App - Salesforce`, removing the Individual assignment deactivated the Salesforce account. The account was reactivated within minutes and no data was lost.

**Cause (from the Okta system log):** tagging added the user to the group, but Okta never created a separate group-based assignment because the user already had an Individual one. Per Okta's documentation, unassigning an Individual assignment does not automatically reassign the app, even if the user belongs to a group that has it.

**Correct procedure:** use **Applications, Salesforce, Assignments, Convert assignments** and convert only the specific user (never "Convert all"). Convert changes the type from Individual to Group without removing anything. The user must already be in a group assigned to the same app.

Reference: Okta Help Center, "Convert an individual assignment to a group assignment".
