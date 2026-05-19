# 005 — Account update and closure

As an authenticated customer I want to update and close (delete) my individual checking or savings accounts so I can manage nicknames and close accounts I no longer use.

Summary

- Two related flows: UPDATE account metadata (nickname/display name) and CLOSE (soft-delete) a single account.
- Builds on: 001 (auth, balances, audit), 003 (multi-account retrieval/endpoints), 004 (rules for last-account deletion when customer deletes profile).
- Out of scope: account creation, inter-account transfers, customer profile auth changes, hard-deleting transaction history.

Key rules

- UPDATE:
  - Customers may set or change an optional display name (nickname) for an account (e.g. "Emergency fund").
  - Immutable fields: account number, account type, creation_date, currency. These cannot be changed via update API.
  - Only the account owner may update. Requests require authentication and authorization checks.
  - Validate display name: length limits and allowed characters (see Validation section).
  - Clearing a previously-set display name is allowed but must be explicit: use `clear_display_name=true` or set `display_name=null` in the update payload. An empty string will be treated as invalid unless `clear_display_name=true` is provided.

- CLOSE (account soft-delete):
  - "Delete account" means close: set status -> CLOSED, set `closed_date`, and mark not available for deposits/withdrawals/transfers.
  - Closure is blocked when any of the following are true:
    - account balance != 0 (non-zero balance)
    - pending transfers exist (outgoing or incoming not settled)
    - unresolved holds (e.g. pending authorizations/holds)
    - account is the customer's only remaining OPEN account (unless the customer is doing full customer deletion via spec 004)
  - Customer must confirm closure by re-entering password or other step (MFA) per platform rules. The API requires an explicit confirmation credential in the request.
  - After closure the account remains visible in read-only lists and retrieval endpoints with status CLOSED and `closed_date` and final balance preserved.
  - Audit log events generated: `ACCOUNT_UPDATED` for updates, `ACCOUNT_CLOSED` for closure.

Security & Authorization

- All endpoints require authentication.
- Only the owning customer (account.owner_id) may perform update or closure.
- For closure the customer must provide confirmation credential: either current password (best-effort) or a short-lived confirmation token produced by an MFA flow. The server verifies credential and rejects if not valid.

Validation

- display_name:
  - Optional string.
  - Allowed characters: letters, numbers, spaces, hyphen, underscore, apostrophe. (Regex: `^[A-Za-z0-9 \-_'’]{1,64}$`)
  - Min length: 1 if setting; Max length: 64 characters.
  - If account already has a display_name and user intends to clear it, client must send `clear_display_name=true` or `display_name=null` explicitly. Sending `display_name: ""` (empty string) is invalid and rejected with 400.

Audit events

- ACCOUNT_UPDATED: includes account_id, owner_id, fields_changed (e.g. display_name old/new), actor_id, timestamp.
- ACCOUNT_CLOSED: includes account_id, owner_id, closed_balance, closed_date, reason (optional), actor_id, timestamp.

- ACCOUNT_REOPENED: includes account_id, owner_id, actor_id, reopened_date, prior_closed_date, timestamp.

API contract (suggested)

PATCH /api/v1/accounts/{account_id}

- Description: Update mutable account metadata (display name).
- Auth: required. Caller must be account owner.
- Request JSON:
  - display_name?: string | null
  - clear_display_name?: boolean (optional, default false). If true, clears any existing display name.

- Rules:
  - If `display_name` present and non-null: validate per rules and set/overwrite.
  - If `display_name` === null or `clear_display_name=true`: clear the display name (set to null).
  - If any immutable field is present in payload, return 400 with details.

- Responses:
  - 200 OK: returns updated account resource (read-only fields unchanged).
  - 400 Bad Request: validation errors (illegal characters, length, immutable field sent, empty-string without explicit clear flag).
  - 401 Unauthorized: not authenticated.
  - 403 Forbidden: authenticated but not owner.

Example request (set nickname):
{
"display_name": "Emergency fund"
}

Example response 200:
{
"account_id": "acc_123",
"owner_id": "cust_456",
"display_name": "Emergency fund",
"account_number": "\*\*\*\*1234",
"account_type": "checking",
"currency": "USD",
"status": "OPEN",
"balance": "1250.00",
"created_date": "2024-07-10T12:34:56Z"
}

POST /api/v1/accounts/{account_id}/close

- Description: Close (soft-delete) a single account.
- Auth: required. Caller must be account owner.
- Request JSON:
  - confirmation_password?: string (customer password for confirmation)
  - confirmation_token?: string (optional, for MFA flows)
  - reason?: string (optional free text)

- Rules & behavior:
  - Server checks: ownership, account status (must be OPEN), balance == 0, no pending transfers, no unresolved holds, not customer's last open account (unless full customer deletion flow applies).
  - If checks pass and confirmation credential validated, set account.status = CLOSED, set closed_date = now, retain final balance and transaction history.
  - Emit `ACCOUNT_CLOSED` audit event.
  - Closed accounts are excluded from operations that allow deposits/withdrawals/transfers; attempts return 409 Conflict or 403 as appropriate.

- Responses:
  - 200 OK: returns account resource with status CLOSED and closed_date.
  - 400 Bad Request: missing confirmation credential or validation error.
  - 401 Unauthorized: not authenticated.
  - 403 Forbidden: not owner or confirmation failed.
  - 409 Conflict: closure blocked (non-zero balance, pending transfers, unresolved holds, or only open account left). Response body explains block reason.

Example close request:
{
"confirmation_password": "current-password-here",
"reason": "No longer need this account"
}

Notes on read-only visibility

- Retrieval endpoints (spec 003 FR-023+) must continue to return closed accounts in account lists and detail views, with `status=CLOSED` and `closed_date` present. Closed accounts must be read-only through update/transaction endpoints.

Edge cases and decision notes

- Clearing display name: we require explicit intent (`clear_display_name` or null) to prevent accidental clearing by empty payloads.
- Final balance on close: require balance == 0 by default. If product offers auto-payout on close, that is out of scope and must be implemented as a separate flow.
- Last open account: closing the customer's only open account is blocked here; if the customer intends to delete their entire profile, they must follow spec 004's deletion flow.

- Savings minimum-balance products: many savings products have product or regulatory minimum balance rules. The general platform rule remains: an account must be brought to a zero balance before it can be closed. If a savings product enforces a minimum balance that prevents the balance reaching zero (for example, a product that does not allow withdrawals below a configured `minimum_balance`), closure must be blocked and the API should return 409 with a clear reason (e.g. "product_minimum_balance prevents closure"). Product-specific remediation (allowing an exception, auto-payout of remaining funds, or administrative close) is out of scope and must be handled by a separate product flow or support process.

- Cooling-off period & reopening closed accounts: platforms may offer a configurable cooling-off (grace) period during which a recently-closed account can be reopened rather than creating a new account. Suggested default: 30 days. Rules:
  - Reopen allowed only within the cooling-off window and only by the account owner (authentication + confirmation required).
  - Preconditions for reopen: no regulatory blocks, ledger/account identifier still available, and no conflicting reuse of the account number or internal identifiers.
  - Reopen operation should set status -> OPEN, set `reopened_date`, clear or preserve `closed_date` (implementation choice — preserve closed_date and add reopened_date is recommended), and emit `ACCOUNT_REOPENED` audit event.
  - If the account number or ledger mapping was recycled or purged after closure, reopening is not possible and the customer must open a new account.

- Reopen API (suggested):

POST /api/v1/accounts/{account_id}/reopen

- Auth: required (owner only). Request JSON: `confirmation_password` or `confirmation_token`.
- Responses: 200 OK (account reopened), 400/401/403 for auth/validation errors, 409 Conflict if outside cooling-off window or if ledger/account mapping unavailable.

- Behavior when closing one of several accounts:
  - Closing one account must not implicitly affect other open accounts. Other accounts remain OPEN and fully operational.
  - Side effects to consider and document:
    - Scheduled payments, standing orders, direct deposits, or external payment instructions that target the closing account must be identified and either cancelled or re-pointed. The close endpoint should return a list of linked standing instructions that require attention, and clients should be instructed to update them before or after closure.
    - If the closing account is the customer's configured default for settlements/external transfers, the system must block closure until the customer sets another default or the UI/flow forces selection of a new default. Optionally, the API may accept `reassign_default_account_id` to atomically reassign the default and close the old account (implementation detail).
    - Internal references (e.g., scheduled transfers between the customer's accounts) should be validated — scheduled outgoing transfers targeting the closing account must be cancelled or failed.

Acceptance criteria

- Unit/integration tests cover:
  - Successful display_name set, change, and explicit clear.
  - Rejection when attempting to change immutable fields.
  - Validation errors for invalid characters/too long names.
  - Successful close when preconditions met and confirmation provided.
  - Blocked close when balance non-zero, pending transfers, unresolved holds, or last open account.
  - Audit events produced for update and close with required metadata.

Open implementation tasks (developer notes)

- Wire PATCH/POST endpoints into existing account service and authorization layers.
- Add validation layer for display_name.
- Implement closure precondition checks using balances, transfer queue, holds store, and customer's account list.
- Ensure retrieval endpoints include closed accounts and display `closed_date` and final balance.
- Add audit producers for ACCOUNT_UPDATED and ACCOUNT_CLOSED.
