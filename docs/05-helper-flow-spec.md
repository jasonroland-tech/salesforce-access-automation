# Helper flow: Salesforce Access - Set Yes

Type: On Demand Helper Flow (kept OFF). It sets `salesforceAccess = Yes` for one row of the `Salesforce Migration` table and records the outcome.

## Cards

1. **On Demand Helper Flow**: input `row_ID`.
2. **Tables, Read Row** (`Salesforce Migration`): Row ID = `row_ID`. Outputs `work_email`, `done`.
3. **Branching, Continue If**: `done` not equal `Yes`. This is the idempotency guard. Finished rows stop here.
4. **Error Handling, If Error**
   - **Try branch**
     1. **Okta, Read User**: ID or Login = `work_email`.
     2. **Okta, Update User**: User ID = Read User's `ID`; Salesforce Access = `Yes`. Update Semantics = **Partial**.
     3. **Tables, Update Row**: Row ID = `row_ID`; `done` = `Yes`.
   - **If error branch**
     1. **Tables, Update Row**: Row ID = `row_ID`; `done` = `ERROR`.

The driver flow (scheduled trigger kept OFF, started with Run) searches for rows where `done` is not `Yes` and calls this helper for each, using For Each with Ignore Errors at concurrency 5.

## Gotchas on the Okta Update User card

| Setting | Required value | Why |
|---|---|---|
| Update Semantics (Options) | **Partial** | Unset gives the error `Invalid type on 'method' input`. **Strict** replaces the whole profile and would wipe every field the card does not send. |
| Input vs Output field lists | Tick **only** `Salesforce Access` in the **Inputs** list | The card lists every field twice. Ticking the Outputs copy makes the card read the value instead of setting it. |
| Pass Empty Values? | Unticked | When ticked, empty fields are sent as blank and can erase attributes. |
| Username | Unticked | Ticked by default after choosing Partial. It is the login and must not be on this card. |
| Mapped inputs | User ID and Salesforce Access only | Re-check after saving Options, because saving can reset the field list. |

Changing the card's Options can reset its field list, so verify the mappings after every Options change.

## Verifying a run

- Execution History shows the Update User card green.
- The user's `salesforceAccess` is `Yes` and every other attribute is unchanged.
- The row's `done` is `Yes`.
- The user appears in `App - Salesforce`.
- The Salesforce admin confirms the account is unchanged.
- A second run of the same row stops at Continue If.
