# Feature Specification: Customer Deletion (Self-service)

**Feature Branch**: `004-customer-deletion`

**Created**: 2026-05-19

**Status**: Draft

**Summary**: As a customer, I want to permanently delete my customer profile and close my relationship with the bank platform, so my personal data is removed where legally allowed while required banking records are retained for compliance.

## Scope

- Authenticated customer initiates "Delete my account" from profile/settings in the web UI.
- Customer must explicitly confirm intent by a multi-step confirmation: type a confirmation keyword (e.g. "DELETE"), re-enter password, and complete MFA if the platform's rules require it.
- System blocks self-service deletion if any account has non-zero balance, pending transfers, unresolved holds, or active legal/regulatory holds. Customer must resolve or transfer funds before deletion proceeds.
- On successful deletion:
  - All customer checking/savings accounts are closed (status updated, balances zeroed and reconciled prior to close).
  - Customer login is disabled; credentials no longer allow access.
  - Personally Identifiable Information (PII) is erased or anonymized/pseudonymized consistent with GDPR/CCPA and platform policies (see Requirements).
  - Transaction and audit records required for compliance are retained for 7 years (per `001-banking-platform`); where possible customer identifiers are pseudonymized.
  - Customer receives email confirmations: one acknowledging the deletion request and one on successful completion.
- Admin-initiated deletion, bulk purge tools, or deletion of another customer's account are out of scope.

## Out of scope

- Deleting individual accounts only (account closure flows are covered by `003-multiple-accounts` and related account-closure stories).
- Admin bulk purge or legal/forensic tools to permanently remove records beyond retention requirements.
- Deleting another customer's account or admin-driven self-service UI for deletion.

## Builds on

- `001-banking-platform` (authentication, transfers, GDPR/CCPA baseline, 7-year retention requirement)
- `002-profile-update` (profile/settings UI and re-auth patterns)
- `003-multiple-accounts` (multi-account considerations: all accounts must be closable for deletion to complete)

## User Scenarios & Testing (mandatory)

### User Story 1 - Self-service Account Deletion (Priority: P1)

Customers must be able to permanently delete their profile from the platform, subject to business and regulatory constraints.

Independent Test: An authenticated customer with zero balances, no pending transfers, and no holds can delete their profile end-to-end and receive confirmation; records are retained and PII is pseudonymized.

Acceptance Scenarios:

1. Given authenticated customer, When customer navigates to Settings -> Delete my account, Then UI displays clear warnings about irreversible consequences, a summary of blocked conditions, and an explicit multi-step confirmation flow
2. Given delete flow initiated, When customer types the confirmation keyword (e.g. "DELETE"), re-enters their password, and completes any required MFA challenge, Then system proceeds to validation checks
3. Given any account has a non-zero balance, pending transfer, or unresolved hold, When customer attempts to confirm deletion, Then system blocks the deletion and displays actionable instructions to resolve the issue (transfer funds, clear pending transfers, contact support for holds)
4. Given validation checks pass, When deletion proceeds, Then:
   - All customer accounts are closed and set to status CLOSED (or equivalent) after any required settlements
   - Customer credentials are disabled and login attempts fail
   - PII is erased or pseudonymized according to the data protection rules in this spec
   - Transaction and audit records are retained for 7 years and include pseudonymized customer identifiers where legally appropriate
   - Customer receives email confirming completion
5. Given deletion completes, When customer attempts to login, Then authentication fails and a customer-facing message explains account no longer exists and to contact support if needed

### Edge Cases

- Pending disputes or chargebacks: If a dispute is open against an account, deletion is blocked until the dispute is resolved or escalated according to legal hold policies.
- Legal/regulatory hold: If a legal hold or subpoena requires preservation of records in their original form, deletion must be blocked and the customer informed that the request cannot be completed for legal reasons.
- Multiple sessions: Deletion must invalidate all active sessions immediately after completion.
- Partially completed funds movement: The system must detect in-flight transfers and either wait for settlement or block deletion until such transfers complete or are cancelled.
- Cross-border data restrictions: If deletion would conflict with jurisdictional data residency or cross-border transfer rules, follow `001-banking-platform` compliance rules and inform the customer.

## Requirements (mandatory)

### Functional Requirements

- FR-001: System MUST expose a "Delete my account" option in the authenticated profile/settings UI
- FR-002: System MUST require explicit confirmation: typing a confirmation keyword (e.g. "DELETE") and re-entering current password
- FR-003: System MUST require MFA for deletion when the platform rules require MFA for sensitive actions (consistent with `001-banking-platform` MFA rules)
- FR-004: System MUST validate preconditions and block deletion if any of the following are true:
  - Any account has non-zero balance
  - Any pending transfers exist (outgoing or incoming pending settlement)
  - Any unresolved holds or disputes exist
  - Any active legal/regulatory hold applies
- FR-005: System MUST provide clear UI messaging and steps for the customer to resolve blocking conditions (transfer funds, close accounts, contact support)
- FR-006: On successful deletion, system MUST close all checking/savings accounts, set account statuses to CLOSED/TERMINATED, and record the closure in audit logs
- FR-007: On successful deletion, system MUST disable customer login and revoke all active sessions/tokens
- FR-008: On successful deletion, system MUST anonymize or erase PII as allowed by GDPR/CCPA; where erasure is not permitted by law, system MUST pseudonymize identifiers
- FR-009: System MUST retain transaction and audit records for 7 years (per `001-banking-platform`) and ensure those records remain available for compliance and audit; any customer identifiers in retained records MUST be pseudonymized where legally allowed
- FR-010: System MUST send an email acknowledging receipt of the deletion request and a second email upon completion
- FR-011: System MUST create an immutable audit entry for each deletion request and its final result (requested, blocked, completed, failed) including timestamps and operator (self-service or admin)
- FR-012: System MUST expose an administrative escalation path for customers who cannot complete self-service due to holds or disputes (note: admin-side deletion tools are out of scope for this feature)

### Non-Functional Requirements

- NFR-001: Deletion request acknowledgement email MUST be sent within 5 minutes of request
- NFR-002: Deletion completion confirmation email MUST be sent within 30 minutes of completion (normally immediate after processing completes)
- NFR-003: All operations within the deletion flow (validation, account closures, pseudonymization) MUST be logged and complete within 30 seconds for the happy path (subject to backend settlement times)
- NFR-004: All deletion-related operations MUST be performed over HTTPS and authenticated channels
- NFR-005: Audit logs and retention storage MUST meet the platform's security and immutability requirements from `001-banking-platform`
- NFR-006: Data erasure/pseudonymization operations MUST be reversible only under controlled, auditable admin processes where permitted by law (e.g., for legal investigations); reversal is out of scope for this self-service flow and requires documented justification

## Success Criteria (mandatory)

- Customers with no blocking conditions can complete account deletion self-service end-to-end and receive confirmation emails.
- Deletion requests are acknowledged by email within 5 minutes for 99% of requests.
- Successful deletions result in disabled logins and pseudonymized retained records consistent with compliance rules.
- Blocking conditions correctly prevent deletion and provide actionable guidance to the customer in 100% of tested cases.
- All deletion requests produce immutable audit entries retained for 7 years.

## Key Entities

- **Account** (existing entity)
  - `status`: New allowed values include CLOSED, PENDING_DELETION, DELETED (internal) to track lifecycle
  - `closed_date`: When account was closed
  - `balance`: Must be zero prior to deletion

- **CustomerDeletionRequest** (new entity)
  - `id`: UUID
  - `account_id`: Account requesting deletion
  - `requested_by`: Subject (self-service)
  - `requested_at`: Timestamp
  - `confirmation_keyword`: The literal keyword typed by customer (stored only as verification that step completed, not retained plaintext longer than necessary)
  - `auth_verified`: Boolean (password verification passed)
  - `mfa_verified`: Boolean
  - `status`: REQUESTED / BLOCKED / PROCESSING / COMPLETED / FAILED
  - `blocked_reasons`: List of strings explaining why request is blocked (non-zero balance, pending transfers, holds, legal_hold)
  - `completed_at`: Timestamp when deletion completed

- **DeletionAuditLog** (new entity or reuse of ProfileAuditLog)
  - `id`: UUID
  - `deletion_request_id`: Foreign key
  - `account_id`: Account being deleted
  - `action`: REQUESTED / VALIDATION_FAILED / COMPLETED / ESCALATED
  - `notes`: Free-text for operator or system reasons
  - `timestamp`: UTC

## Assumptions

- The platform enforces single-customer accounts (no joint accounts). Joint-account deletion or shared account models are out of scope.
- Account balances must be zero before deletion; customers can transfer remaining balances to external accounts or internal beneficiaries using existing transfer flows.
- Legal/regulatory holds override self-service deletion and must be detected by the compliance/holds subsystem implemented in `001-banking-platform`.
- Pseudonymization approaches (hashing, tokenization) are considered acceptable where full erasure is prohibited by law; the implementation will follow legal counsel and platform policy.
- Email is the primary communication channel for confirmations; if email delivery is blocked, the deletion request will be queued and retried per platform email retry policies.

## Test Cases (suggested)

- Happy path: authenticated customer with zero balances and no holds completes deletion and cannot login afterward; transaction records retained and pseudonymized.
- Blocked path: customer with non-zero balance attempts deletion and receives clear instructions; deletion remains blocked until balance resolved.
- Dispute hold path: customer with open dispute is blocked and receives instructions to contact support; deletion request is logged as BLOCKED.
- MFA-required path: customer required to complete MFA and cannot proceed without it; deletion proceeds after successful MFA verification.

## Notes

- Admin deletion flows and bulk purge tools are intentionally excluded to reduce risk and keep this feature focused on self-service.
- Implementation must coordinate closely with compliance and legal teams to finalize exact pseudonymization/erasure rules.

## Clarifications & Decisions

The following clarifications capture decisions needed to make implementation unambiguous. These are proposals; product, security, and legal should sign off before implementation.

- Cooling-off / pending-deletion period:
  - Default behaviour: Deletion is two-phase. After a successful self-service request the account enters `PENDING_DELETION` for a configurable cooling-off period (recommended default: 14 days). During this window the customer can cancel the request via the UI and normal account access remains disabled.
  - Rationale: prevents accidental or fraudulent permanent deletion and provides time to resolve automated / scheduled payments.
  - Legal exceptions: a legal/regulatory hold MUST override the cooling-off behaviour (i.e., prevent progression to permanent deletion).

- Soft-delete vs hard-delete:
  - Soft-delete (default for self-service): when deletion completes the system pseudonymizes/erases PII in customer-facing stores but retains transactional and audit records in retention storage for 7 years. The account record keeps a minimal administrative row (status = `DELETED`, deletion metadata) so retention and reconciliation systems work.
  - Hard-delete (permanent physical removal) is only allowed after retention periods expire and only via an auditable admin process that requires legal sign-off. Hard-delete is explicitly out of scope for the self-service flow.

- Recovery window and reversal policy:
  - During the `PENDING_DELETION` cooling-off period the customer may cancel and the system will revert to `ACTIVE` (full recovery).
  - After the cooling-off period completes the system performs the pseudonymization/soft-delete steps. For operational/legal reasons, a short admin recovery window (recommended: 7 days) may remain where reversal requires documented justification and an auditable admin process. After that window, reversal is not supported for self-service deletions.
  - All reversals (if any) must be logged in the `DeletionAuditLog` with justification, operator id, and timestamp.

- Behavior when customer is a transfer recipient or beneficiary:
  - Deletion is blocked if there are any pending incoming transfers, scheduled/recurring payments directed to the account, or unsettled incoming settlements. The UI must list these blocking items and provide actionable steps (cancel/redirect recurring payments, wait for settlement, contact payer).
  - Historical transactions where the customer was the recipient remain in the ledger for retention; the recipient identifier in those records is pseudonymized per the pseudonymization rules.
  - If the customer is a named beneficiary on third-party scheduled payments, the platform should attempt to notify counterparties where feasible (implementation detail) or instruct the customer to cancel/redirect such schedules prior to deletion.

- Audit log retention and pseudonymization strategy:
  - Audit and transaction logs are retained for 7 years per `001-banking-platform`. For self-service deletions these records will not contain direct PII after pseudonymization.
  - Recommended approach: generate a pseudonymous deletion token (e.g., deletion_id) that replaces customer-identifying fields in retained records. Store the mapping (deletion_id -> account_id) in a separate, encrypted, access-controlled key-value store accessible only to compliance/legal and only under logged, auditable admin requests.
  - Access to mapping data MUST be strictly controlled and every access must produce an immutable access log entry.

- Interaction with MFA and other high-risk controls:
  - Deletion is high-risk and MUST require re-authentication (password) plus MFA when:
    - The customer's account has MFA enrolled, or
    - The platform's risk engine flags the deletion as high-risk (suspicious location, recent password reset, large recent transfers, etc.).
  - MFA is performed as the final confirmation step in the deletion flow. If MFA cannot be completed (lost device, blocked SMS), the system must offer an administrative escalation path rather than allow bypassing MFA.

These clarifications are intentionally conservative; final numeric values (cooling-off period, admin recovery window) and exact pseudonymization techniques must be approved by legal/compliance and can be made configurable in implementation.
