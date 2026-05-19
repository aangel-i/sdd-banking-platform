# Feature Specification: Multiple Account Types Management

**Feature Branch**: `003-multiple-accounts`

**Created**: 2026-05-19

**Status**: Draft

**Input**: The customer is able to open up an account (either checking or savings). The customer is able to have more than 1 account and it can be of either type

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Create New Checking Account (Priority: P1) 🎯

Customers must be able to open new checking accounts. A checking account is a transactional account for daily deposits and withdrawals with no withdrawal limits.

**Why this priority**: Checking accounts are the fundamental transaction vehicle for customers; enabling account creation is foundational.

**Independent Test**: Customer can create a new checking account and immediately deposit/withdraw funds from it.

**Acceptance Scenarios**:

1. **Given** authenticated customer, **When** customer navigates to "Create Account" page, **Then** system displays account type selection (Checking or Savings)
2. **Given** account type options shown, **When** customer selects "Checking", **Then** system displays checking account details (no withdrawal limits, transaction-focused) and confirmation button
3. **Given** checking account details displayed, **When** customer confirms account creation, **Then** new checking account is created with $0 opening balance
4. **Given** checking account created, **When** customer views account list, **Then** new account appears with account number, type (Checking), balance, and creation date
5. **Given** new checking account, **When** customer performs deposit, **Then** system credits funds to correct checking account (not other accounts)
6. **Given** multiple accounts exist, **When** customer selects checking account, **Then** dashboard displays checking account details and transaction history for that account only

---

### User Story 2 - Create New Savings Account (Priority: P1) 🎯

Customers must be able to open new savings accounts. A savings account is designed for long-term deposits with interest accrual and withdrawal restrictions.

**Why this priority**: Savings accounts enable customers to grow funds with interest; supporting multiple account types is core to the platform expansion.

**Independent Test**: Customer can create a new savings account and system displays it distinctly from checking accounts.

**Acceptance Scenarios**:

1. **Given** authenticated customer, **When** customer selects "Create Account", **Then** system displays account type options
2. **Given** account types available, **When** customer selects "Savings", **Then** system displays savings account details (interest accrual, withdrawal limits if applicable) and confirmation
3. **Given** savings account details confirmed, **When** customer completes creation, **Then** new savings account created with $0 opening balance and interest rate displayed
4. **Given** savings account created, **When** customer views account details, **Then** system displays account type as "Savings" with interest rate, minimum balance requirement (if any), and interest accrual schedule
5. **Given** multiple account types exist, **When** customer initiates transfer, **Then** system allows transfer between checking and savings accounts
6. **Given** savings account, **When** customer views account, **Then** interest earnings displayed separately from principal balance

---

### User Story 3 - Switch Between Multiple Accounts (Priority: P1) 🎯

Customers must be able to easily switch between their multiple accounts to view balances, transaction history, and perform operations on specific accounts.

**Why this priority**: Account switching is essential UI/UX for managing multiple accounts; direct impact on usability.

**Independent Test**: Customer can switch accounts and all displayed data updates correctly to show only selected account's information.

**Acceptance Scenarios**:

1. **Given** customer with 2+ accounts, **When** customer views dashboard, **Then** system displays account selector/dropdown showing all accounts with type and balance
2. **Given** account list available, **When** customer selects different account, **Then** dashboard updates to show selected account details, balance, and transaction history
3. **Given** account switched, **When** customer views transaction history, **Then** history shows only transactions for selected account, not other accounts
4. **Given** multiple accounts, **When** customer performs deposit/withdrawal, **Then** system applies transaction to selected account
5. **Given** customer viewing account, **When** customer navigates to statements, **Then** system generates statement for currently selected account
6. **Given** 3+ accounts visible, **When** account list displays, **Then** accounts clearly distinguished by type (icon, label, or color) and balance highlighted

---

### User Story 4 - View All Accounts Summary (Priority: P2)

Customers must be able to view a summary of all their accounts at once to understand total assets and account distribution.

**Why this priority**: Summary view provides portfolio visibility; valuable but secondary to individual account management.

**Independent Test**: Customer can view a dashboard showing all accounts with balances and combined total.

**Acceptance Scenarios**:

1. **Given** customer with multiple accounts, **When** customer navigates to "All Accounts" or summary view, **Then** system displays table/grid of all accounts with account number, type, balance, and last transaction date
2. **Given** summary view displayed, **When** customer reviews data, **Then** accounts clearly segregated by type (Checking accounts listed together, Savings accounts listed together)
3. **Given** account summary, **When** customer views totals, **Then** system displays total balance across all accounts and breakdown by account type (total checking, total savings)
4. **Given** summary view, **When** customer clicks on any account, **Then** system navigates to account detail view for that account

---

### User Story 5 - Transfer Between Own Accounts (Priority: P2)

Customers must be able to transfer funds between their own checking and savings accounts for account management and strategy.

**Why this priority**: Inter-account transfers are valuable for account management; secondary to core operations but important for multi-account experience.

**Independent Test**: Customer can transfer $100 from checking to savings and balances update correctly on both accounts.

**Acceptance Scenarios**:

1. **Given** authenticated customer with 2+ accounts, **When** customer initiates transfer, **Then** system displays transfer form with "From Account" and "To Account" dropdowns showing eligible accounts
2. **Given** transfer form displayed, **When** customer selects checking as source and savings as destination, **Then** system allows selection and displays transfer amount input
3. **Given** transfer amount entered, **When** customer reviews transfer details, **Then** system shows source account, destination account, amount, and estimated completion time
4. **Given** transfer details confirmed, **When** customer submits, **Then** transfer completes and both accounts updated immediately
5. **Given** transfer between own accounts completed, **When** customer views account transactions, **Then** transfer appears in "From" account as withdrawal and "To" account as deposit

---

### Edge Cases

- What happens when customer attempts to create 100th account?
  - **Answer**: System allows creation; no limit specified on account quantity (assume reasonable limit like 100 or 1000 enforced in implementation)
- Can customer transfer between checking and savings when insufficient balance in source?
  - **Answer**: System rejects transfer with "Insufficient funds in source account" message
- What if customer closes one account while owning multiple accounts?
  - **Answer**: Covered in separate feature; this feature assumes all accounts remain open
- Can customer rename accounts (e.g., "My Emergency Fund" vs "Savings Account")?
  - **Answer**: Out of scope for this feature; naming/labeling covered in profile features
- What if savings account has withdrawal restrictions or minimum balance?
  - **Answer**: Display restrictions clearly; validate at withdrawal/transfer time; reject if violates constraints

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST allow authenticated customers to create new checking accounts from profile/account management page
- **FR-002**: System MUST allow authenticated customers to create new savings accounts from profile/account management page
- **FR-003**: System MUST assign unique account number to each new account (format: consistent, e.g., 10-digit or UUID)
- **FR-004**: System MUST display account type (Checking or Savings) for all accounts
- **FR-005**: System MUST initialize new account with $0 opening balance
- **FR-006**: System MUST record account creation timestamp for each account
- **FR-007**: System MUST allow customer to view list of all owned accounts with account number, type, balance, and creation date
- **FR-008**: System MUST allow customer to select/switch between accounts for transaction operations
- **FR-009**: System MUST ensure all transactions (deposit/withdraw) apply to currently selected account only
- **FR-010**: System MUST display transaction history filtered to currently selected account (no cross-account transaction mixing)
- **FR-011**: System MUST generate statements for specific account when customer requests (not combined statements)
- **FR-012**: System MUST display interest rate for savings accounts (if applicable to savings account type)
- **FR-013**: System MUST allow customer to transfer funds between their own accounts (e.g., checking to savings)
- **FR-014**: System MUST validate transfer source account has sufficient balance before completing inter-account transfer
- **FR-015**: System MUST update both source and destination account balances immediately after inter-account transfer completes
- **FR-016**: System MUST record inter-account transfer as withdrawal in source account and deposit in destination account
- **FR-017**: System MUST prevent transfer from account to same account (no self-transfers)
- **FR-018**: System MUST display summary view showing all accounts with totals by type (total checking balance, total savings balance)
- **FR-019**: System MUST calculate and display total balance across all accounts
- **FR-020**: System MUST enforce account ownership (customers can only view/access their own accounts)
- **FR-021**: System MUST prevent operation on closed or inactive accounts (if applicable)
- **FR-022**: System MUST audit log account creation, closure, and transfers for compliance

- **FR-023**: System MUST provide an authenticated API/UI endpoint to retrieve a single account's details by `account_id`.
- **FR-024**: The account detail response MUST include, at minimum, the following fields:
  - `id` (UUID)
  - `account_number` (human-readable)
  - `account_type` (CHECKING or SAVINGS)
  - `currency_code` (ISO-4217)
  - `balance` (current ledger balance in cents)
  - `available_balance` (balance available for withdrawal after holds in cents)
  - `opening_balance` (in cents)
  - `status` (Active/Inactive/Closed)
  - `interest_rate` (nullable, percent)
  - `created_date` (UTC timestamp)
  - `last_transaction_date` (UTC timestamp, nullable)
  - `closed_date` (UTC timestamp, nullable)
  - `pending_transfers_count` (integer)
  - `unresolved_holds_count` (integer)
  - `transactions_url` (link or href to fetch paginated transactions for this account)
  - `owner_customer_id` (UUID) — returned only when the caller is authorized to view this field (i.e., the account owner or authorized admin)
  - `display_name` (optional customer-provided label)
  - NOTE: Sensitive fields MUST NOT be returned (e.g., `password_hash`, full unredacted internal tokens).

- **FR-025**: If the requested `account_id` does not exist, the system MUST return a 404 Not Found (error code: ACCOUNT_NOT_FOUND) to the caller.
- **FR-026**: If the requested account exists but does not belong to the authenticated customer, the system MUST return a 403 Forbidden (error code: FORBIDDEN) and must not disclose whether the account exists beyond the standard error.
- **FR-027**: Closed account behaviour: The detail endpoint MUST return account details for CLOSED accounts (status = CLOSED) for read-only and historical purposes. The response MUST include `closed_date` and the final `balance` at time of closure. Any write/transaction attempts against closed accounts are prevented by other APIs and should return 409 Conflict (see FR-021 for prevention of operations on closed accounts).
- **FR-028**: The account detail response MUST include indicators useful for blocking/resolution flows (e.g., `pending_transfers_count`, `unresolved_holds_count`, `available_balance`) so the UI can present actionable guidance (for example, blocking deletion when balances/pending items exist).
- **FR-029**: The account detail endpoint MUST support conditional requests (ETag/Last-Modified) and return appropriate cache headers to allow safe short-lived caching of static fields while ensuring balances/holds are fresh.

### Non-Functional Requirements

- **NFR-001**: Account creation MUST complete within 2 seconds
- **NFR-002**: Account switching MUST update dashboard within 1 second
- **NFR-003**: Account list display MUST load within 1 second for customers with up to 50 accounts
- **NFR-004**: System MUST support minimum 50 accounts per customer without performance degradation
- **NFR-005**: Transfer between own accounts MUST complete within 2 seconds
- **NFR-006**: All account views MUST display correctly on desktop and mobile devices (responsive design)
- **NFR-007**: Account data MUST be encrypted at rest and in transit
- **NFR-008**: Account ownership validation MUST occur on every request to prevent unauthorized access

- **NFR-009**: Account detail retrieval MUST return within 500ms for 95% of requests and within 2 seconds for 99.9% of requests under typical production load (defined as customers with up to 50 accounts and normal transaction throughput).
- **NFR-010**: Account detail endpoint MUST support conditional GETs (ETag/Last-Modified) and include appropriate Cache-Control headers for short-lived caching of non-volatile fields. Dynamic financial fields (e.g., `balance`, `available_balance`) MUST either be marked as non-cacheable or updated frequently with short TTLs.

## Success Criteria _(mandatory)_

- Customers can create new accounts in under 1 minute with clear account type selection
- Account switching is seamless with dashboard updating within 1 second
- Account list display is intuitive and clearly segregates checking vs. savings accounts
- Customers can transfer funds between own accounts within 2 seconds
- All account balances display accurately and update in real-time after transactions
- Zero unauthorized access to other customers' accounts (100% ownership enforcement)
- Account creation form provides clear guidance on account type differences and features
- Summary view accurately calculates and displays total balances by type
- Responsive design renders account selector and details correctly on 320px-2560px widths
- 99.9% of inter-account transfers complete successfully without errors
- Audit logs capture 100% of account creation and transfer operations for compliance

## Key Entities

- **Account** (existing entity, extended)
  - `id`: Unique account identifier (UUID)
  - `customer_id`: Customer who owns account (foreign key) — **NEW: explicitly links account to customer**
  - `account_number`: Unique human-readable account number (e.g., 10-digit code)
  - `account_type`: Account type (CHECKING or SAVINGS) — **NEW**
  - `currency_code`: Currency (ISO-4217 code, e.g., USD) — **NEW: enables multi-currency support if needed**
  - `balance`: Current account balance in cents (integer)
  - `opening_balance`: Balance when account created (for audit trail)
  - `status`: Active/Inactive/Closed (allows closed accounts without deletion)
  - `interest_rate`: Interest rate for savings accounts (nullable, percent, e.g., 2.5 for 2.5%)
  - `created_date`: Account creation timestamp
  - `last_transaction_date`: When last transaction occurred (nullable)
  - `closed_date`: When account was closed, if applicable (nullable)
  - `password_hash`: Bcrypt hash (moved from legacy model if single-account)
  - `email`: Email address (moved from legacy model if single-account)
  - `full_name`: Account holder name (moved from legacy model if single-account)
  - Other profile fields (address, etc.)

- **InterAccountTransfer** (new entity)
  - `id`: Unique transfer identifier (UUID)
  - `customer_id`: Customer performing transfer (foreign key)
  - `from_account_id`: Source account (foreign key, must belong to customer)
  - `to_account_id`: Destination account (foreign key, must belong to customer)
  - `amount`: Transfer amount in cents (integer, positive)
  - `description`: Optional transfer description/reference
  - `timestamp`: When transfer initiated (UTC)
  - `completion_timestamp`: When transfer completed (UTC)
  - `status`: Initiated/Completed/Failed/Reverted
  - `from_account_balance_after`: Source account balance after transfer
  - `to_account_balance_after`: Destination account balance after transfer

- **AccountAuditLog** (new entity)
  - `id`: Unique log entry identifier (UUID)
  - `customer_id`: Customer performing action (foreign key)
  - `account_id`: Account being acted upon (foreign key, nullable for customer-level actions)
  - `action_type`: Action (ACCOUNT_CREATED, ACCOUNT_CLOSED, TRANSFER_INITIATED, TRANSFER_COMPLETED, ACCOUNT_ACCESSED)
  - `account_type`: Account type if action is creation (CHECKING or SAVINGS)
  - `details`: Additional details (JSON, e.g., transfer amount, account number)
  - `ip_address`: IP address from which action originated
  - `user_agent`: Browser/client user agent
  - `timestamp`: When action occurred (UTC)

## Assumptions

- **Account ownership model**: One customer can own multiple accounts; each account belongs to exactly one customer (no joint accounts)
- **Account types**: Two account types supported (Checking and Savings); no other types (Business, Money Market, etc.) in scope
- **Account creation permission**: Any authenticated customer can create new accounts (no approval process)
- **Account limits**: No specified maximum accounts per customer; implementation enforces reasonable limit (assume 50-100)
- **Account isolation**: Transactions, statements, and data are strictly isolated per account; customers see only their own accounts
- **Inter-account transfers**: Transfers between own accounts are immediate and atomic; treated as two linked transactions
- **Account closure**: Out of scope for this feature; closed account handling (data retention, etc.) handled separately
- **Currency**: Single currency per customer/platform (USD assumed); multi-currency transfers out of scope
- **Interest accrual**: Savings account interest calculation and accrual mechanism out of scope; system displays rate but doesn't calculate interest (separate feature)
- **Withdrawal restrictions**: Savings account withdrawal limits/restrictions out of scope; customers can withdraw at any time
- **Account balances**: Always represent most recent state; no cached/stale balance display
- **Compliance**: Account creation logged for audit trail; customer data privacy maintained per GDPR/CCPA (from main platform)
- **Performance**: Database indexes on customer_id, account_id, account_number enable fast lookups; account list queries optimized for 50+ accounts

## Dependencies

- **Depends on**: 001-banking-platform (authentication, transaction model, compliance framework)
- **Depends on**: 002-profile-update (customer profile management)
- **Supports**: Future features (business accounts, joint accounts, account linking, fee management)
