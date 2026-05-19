# Requirements checklist — 005 Account update and closure

Status: Draft

Functional requirements

- [ ] FR-001: Authenticated customer can update account display name (nickname).
- [ ] FR-002: Immutable fields cannot be modified via update API (account_number, account_type, created_date, currency).
- [ ] FR-003: Only account owner can update or close account; API enforces ownership.
- [ ] FR-004: Display name validation: allowed characters and max length 64.
- [ ] FR-005: Clearing a previously-set display name must be explicit (clear_display_name=true or display_name=null accepted).
- [ ] FR-006: Customer can close (soft-delete) an account when preconditions met.
- [ ] FR-007: Closure blocked for non-zero balance, pending transfers, unresolved holds, or when it's the customer's only open account (unless via customer deletion flow 004).
- [ ] FR-007b: Closure blocked when product minimum_balance prevents zeroing balance (savings minimums).
- [ ] FR-007c: If account is default settlement account, require reassignment or explicit reassign_default_account_id in close request.
- [ ] FR-007d: Return list of linked standing instructions or scheduled payments that target the closing account so client can reconfigure.
- [ ] FR-008: Closure requires confirmation (password or MFA confirmation token).
- [ ] FR-009: After closure account.status == CLOSED and closed_date is set; account remains visible in read-only lists and retrieval endpoints.
- [ ] FR-010: Closed accounts cannot be used for deposits/withdrawals/transfers; transactions return 403/409.
- [ ] FR-012: Support configurable cooling-off (reopen) window; reopening requires owner confirmation and only allowed within the configured window.
- [ ] FR-013: Reopening emits ACCOUNT_REOPENED and sets reopened_date; outside window return 409.
- [ ] FR-011: Audit events emitted: ACCOUNT_UPDATED and ACCOUNT_CLOSED with required metadata.

API and error handling

- [ ] API: PATCH /api/v1/accounts/{account_id} for updates.
- [ ] API: POST /api/v1/accounts/{account_id}/close for closure.
- [ ] Error codes: 400 for validation/immutable field, 401 for unauthenticated, 403 for unauthorized, 409 for blocked closure.

Security & privacy

- [ ] Ensure confirmation credential is not logged. Audit events must not contain plaintext passwords.

Acceptance tests (high level)

- [ ] Update: set new nickname -> 200, display in detail view.
- [ ] Update: change nickname -> 200, audit event ACCOUNT_UPDATED with fields_changed.
- [ ] Update: explicit clear -> 200, display_name null.
- [ ] Update: send empty-string without clear flag -> 400.
- [ ] Close: attempt with non-zero balance -> 409.
- [ ] Close: attempt with pending transfers/holds -> 409.
- [ ] Close: attempt as non-owner -> 403.
- [ ] Close: successful close with zero balance and confirmation -> 200 and ACCOUNT_CLOSED event.
- [ ] Retrieval endpoints return closed account details including closed_date and final balance.
- [ ] Close: attempt when product enforces minimum balance -> 409 and reason referencing minimum_balance.
- [ ] Reopen: successful reopen within cooling-off window -> 200 and ACCOUNT_REOPENED event.
- [ ] Close: closing one of multiple accounts preserves other accounts; default-account reassignment enforced when needed.

Notes

- Coordinate with spec 003 to ensure closed accounts remain included in retrievals.
- Coordinate with spec 004 for the last-account deletion rule and cross-link behaviors.
