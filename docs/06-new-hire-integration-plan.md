# New-hire integration plan

Status: **planned, not yet implemented.**

## Data path

```
Jira ticket field "Salesforce Access Required" (Yes/No)
  -> Okta Workflows: Jira New Request trigger (field exposed under Options)
  -> Tables, Create Row: new column salesforce_access_required
       in the Okta Onboarding - Flow State table
  -> Account creation subflow: Okta Create User
       salesforceAccess <- salesforce_access_required
  -> group rule adds the user to App - Salesforce
```

The scheduler hands the creation flow only a `row_ID`, so the Flow State table is the hand-off. No new onboarding flow is created.

## Work items

1. Add the Jira field (single-select Yes/No), add it to the screen, request form and issue layout.
2. In the inbound flow, tick the field on the New Request card.
3. Add the `salesforce_access_required` column to the Flow State table and map it in Create Row.
4. Map `salesforceAccess` from that column in Create User.
5. Back up the flows (Version History), do the first change on a cloned copy that stays OFF.

## Test plan

| Case | Expected |
|---|---|
| Field = Yes | Attribute `Yes`, user in `App - Salesforce`, Salesforce account created with baseline access |
| Field = No | Attribute empty, no Salesforce access |
| Field blank | Needs testing in the clone before production. Create User must not fail on an empty value. |
| End to end | Fake ticket with a far-future start date, then clean up the row, child ticket and staged Okta user |

## Risks

- The inbound flow is live and posts to Slack. Warn the team before changing it.
- New hires get Minimum Access. The Salesforce admin must be told.
