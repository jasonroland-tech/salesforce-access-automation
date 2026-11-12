# Okta configuration

## Custom attribute

| Field | Value |
|---|---|
| Display name | Salesforce Access |
| Variable name | `salesforceAccess` |
| Type | string |
| Values used | `Yes`. Blank or any other value means no access. |

Added in Directory, Profile Editor, Okta user profile. No Profile Source writes to it.

## Group rule

```
user.salesforceAccess == "Yes"
```

Target group: `App - Salesforce`. The rule only adds and removes users it manages. Direct members are untouched.

## Group assignment on the Salesforce app

`App - Salesforce` is a least-privilege baseline:

- Salesforce Profile: Minimum Access
- Role: none
- Permission sets: none
- Feature licenses: none

New hires therefore start with minimal access until the Salesforce admin configures their account. The admin owns everything beyond that.

> **A blank Profile on a group assignment breaks provisioning.** Okta returns `Blank is an invalid value for profile field (code blank)` and the account is not created. Every group assignment that can create users needs a Profile.

## Group priority

When a user belongs to several groups assigned to the app, Okta applies the assignment values from the group with the highest priority (lowest number). This affects only *new* accounts here, because Update User Attributes is off.

## Provisioning behaviour to remember

- Removing a user's last assignment triggers Deactivate Users within seconds.
- Assignment values are applied on Create only. They never overwrite a configured account.
