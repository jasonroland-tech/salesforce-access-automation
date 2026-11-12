# Lessons learned

1. **Entitlement should be explicit.** Access that is a side effect of HR-driven groups breaks the moment HR data changes.
2. **Test with real accounts, smallest population first.** One user, then five, then batches. Every surprise so far surfaced in a small test.
3. **A group assignment needs a Profile.** A blank Salesforce Profile on a group assignment fails provisioning with `Blank is an invalid value for profile field`.
4. **Individual assignments are not covered by group membership.** Adding someone to the group does not create a group assignment while an Individual one exists. Removing the Individual assignment deactivates the account. Use **Convert assignments** on specific users. Convert cannot be undone, so never use "Convert all".
5. **Okta Workflows Update User defaults are dangerous.**
   - Update Semantics must be set to **Partial**. Strict replaces the whole profile.
   - Untick Pass Empty Values and Username.
   - The card lists inputs and outputs separately. Tick the field under Inputs.
6. **Build in idempotency.** A Continue If on `done` made reruns safe.
7. **Treat caught errors as a signal.** The flow showed "Success" even when its If Error branch ran. Check the table status, not only the execution status.
8. **Keep personal data out of project artefacts.** Migration lists belong in the run table, not in a repo or deck.
9. **A blast radius check before group removals.** Removing Salesforce from a rule-driven group removes it for everyone in the group.
