# Migration runbook

Goal: tag every existing Salesforce user with `salesforceAccess = Yes`, confirm they are in `App - Salesforce`, and only then remove the old department/division Salesforce assignments, one group at a time.

## Rules

1. Never remove an old assignment until the user is tagged, in `App - Salesforce`, and the Salesforce admin has confirmed their account.
2. Removing Salesforce from a Paylocity rule-driven group removes it for **everyone** in that group. Check every member first.
3. Start with the smallest groups. Do sales and customer-success roles last.
4. Any user with an **Individual** assignment is converted with Convert assignments, not removed.
5. The Salesforce admin checks after every batch.

## Step 1: reconcile the list

Export Okta's Salesforce assignments and the admin's list of active Salesforce users. Reconcile them. Exclude service accounts, external users and anyone already piloted. Keep the result out of source control.

## Step 2: load the Workflows table

Create the table `Salesforce Migration` with two columns:

| Column | Purpose |
|---|---|
| `work_email` | Okta login of the user |
| `done` | Blank until processed, then `Yes` or `ERROR` |

Import the email list as CSV.

## Step 3: run the helper flow

1. Test on a single group-based user. Verify Okta profile, `done`, group membership, and ask the admin to confirm the Salesforce account.
2. Re-run the same row to confirm the guard.
3. Build the driver flow: search rows where `done` is not `Yes`, then For Each (ignore errors, concurrency 5), calling the helper with `row_ID`.
4. Pilot about three users, then run the rest.
5. Verify the count of users with `salesforceAccess = Yes` and review any `ERROR` rows.

## Step 4: remove old group assignments in batches

For each group, smallest first:

1. Confirm every member of the group is tagged and in `App - Salesforce`.
2. Remove Salesforce from that group's assignments only.
3. Have the admin confirm no one was deactivated and no profile changed.
4. Record the result and move to the next group.

## Individually assigned users

Applications, Salesforce, Assignments, Convert assignments. Select the specific users. Do not use "Convert all".
