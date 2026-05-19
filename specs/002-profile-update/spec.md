# Feature Specification: Customer Profile Update

**Feature Branch**: `002-profile-update`

**Created**: 2026-05-19

**Status**: Draft

**Input**: The customer is able to update their profile such as their email, address and password

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Update Account Email Address (Priority: P1) 🎯

Customers must be able to update their email address on file, with verification to ensure only account holders can change their email.

**Why this priority**: Email is a critical contact method and recovery mechanism; customers need to keep it current.

**Independent Test**: Customer can change their email address, receive verification email, confirm the change, and login with new email.

**Acceptance Scenarios**:

1. **Given** authenticated customer, **When** customer requests email change, **Then** system prompts for new email and current password for verification
2. **Given** new email address provided, **When** system validates format, **Then** system rejects if email already registered to another account
3. **Given** valid new email, **When** customer submits change, **Then** verification email sent to new address with confirmation link (valid for 24 hours)
4. **Given** verification email received, **When** customer clicks confirmation link, **Then** email change completes and customer can login with new email
5. **Given** verification link expired, **When** customer clicks link, **Then** system rejects with "Verification link has expired; request new change"
6. **Given** email changed, **When** customer logs in, **Then** login with old email fails with "Invalid credentials"

---

### User Story 2 - Update Address Information (Priority: P1) 🎯

Customers must be able to update their profile address information (street, city, state, postal code, country).

**Why this priority**: Address is required for compliance, statements, and customer contact; needs to be current and accurate.

**Independent Test**: Customer can update all address fields and system validates and persists the new address.

**Acceptance Scenarios**:

1. **Given** authenticated customer, **When** customer accesses profile edit, **Then** current address displayed with editable fields (or "not provided" if empty)
2. **Given** customer modifies address fields, **When** customer saves changes, **Then** system validates address format (street, city, state, postal code required; country optional)
3. **Given** incomplete address, **When** customer attempts to save, **Then** system rejects with specific field error (e.g., "City is required")
4. **Given** valid address entered, **When** customer saves, **Then** address updates immediately and confirmation message displays
5. **Given** updated address, **When** customer views profile, **Then** new address displays correctly
6. **Given** address changed, **When** next statement generated, **Then** statement reflects updated address

---

### User Story 3 - Update Account Password (Priority: P1) 🎯

Customers must be able to change their account password securely, requiring verification of current password and confirmation of new password.

**Why this priority**: Password management is critical for account security; customers need ability to update periodically or if compromised.

**Independent Test**: Customer can change password, logout, and login successfully with new password.

**Acceptance Scenarios**:

1. **Given** authenticated customer, **When** customer requests password change, **Then** system prompts for current password, new password, and password confirmation
2. **Given** incorrect current password entered, **When** customer submits, **Then** system rejects with "Current password is incorrect"
3. **Given** new password doesn't meet requirements, **When** customer submits, **Then** system rejects with specific error (e.g., "Password must contain uppercase letter")
4. **Given** new password and confirmation don't match, **When** customer submits, **Then** system rejects with "Passwords do not match"
5. **Given** valid current password and matching new passwords meeting requirements, **When** customer submits, **Then** password changes and confirmation displays
6. **Given** password changed, **When** customer logs out and attempts login with old password, **Then** login fails with "Invalid credentials"
7. **Given** password changed, **When** customer logs in with new password, **Then** login succeeds and customer accesses account

---

### User Story 4 - View Profile Summary (Priority: P2)

Customers must be able to view their complete profile information including email, name, address, and account creation date.

**Why this priority**: Profile view provides visibility into account data; secondary to update functionality.

**Independent Test**: Customer can view all profile information currently on file.

**Acceptance Scenarios**:

1. **Given** authenticated customer, **When** customer navigates to profile page, **Then** all profile data displays (email, name, address, created date)
2. **Given** address not yet provided, **When** customer views profile, **Then** profile displays "No address provided" with option to add address
3. **Given** customer reviews profile, **When** profile page displays, **Then** read-only view shows current data without edit mode until edit button clicked

---

### Edge Cases

- What happens if customer attempts to change email to currently registered email (no change)?
  - **Answer**: System allows submission but detects no-op; displays "Email already your current email" message without sending verification
- What if customer is logged in from multiple sessions when changing password?
  - **Answer**: Password change invalidates all existing sessions; customer must login again on all devices with new password
- What if email verification expires before customer completes confirmation?
  - **Answer**: Customer can initiate new email change request; previous verification link becomes inactive
- What if customer changes email while pending verification from previous change?
  - **Answer**: Previous unconfirmed change is cancelled; new change requires new verification; last change attempt takes precedence

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow authenticated customers to initiate email change from profile page
- **FR-002**: System MUST validate that new email address is not already registered to another account
- **FR-003**: System MUST require current password verification before allowing email change to proceed
- **FR-004**: System MUST send verification email to new email address with clickable confirmation link valid for 24 hours
- **FR-005**: System MUST confirm email change only after customer clicks verification link
- **FR-006**: System MUST prevent login with old email address after email change completes
- **FR-007**: System MUST allow authenticated customers to update their address (street, city, state, postal code)
- **FR-008**: System MUST validate address fields are non-empty and valid format (no special characters beyond standard punctuation)
- **FR-009**: System MUST persist address updates immediately upon successful validation
- **FR-010**: System MUST display updated address on profile page immediately after change
- **FR-011**: System MUST include address on next generated monthly statement after address change
- **FR-012**: System MUST allow authenticated customers to change their account password
- **FR-013**: System MUST require customer to provide current password before accepting new password
- **FR-014**: System MUST validate new password meets strength requirements (8+ characters, mixed case, number, symbol)
- **FR-015**: System MUST require customer to confirm new password (re-enter to verify match)
- **FR-016**: System MUST reject password change if current password is incorrect
- **FR-017**: System MUST reject password change if new password doesn't meet strength requirements
- **FR-018**: System MUST reject password change if new password and confirmation don't match
- **FR-019**: System MUST invalidate all customer sessions after password change (force re-login on all devices)
- **FR-020**: System MUST display profile summary including email, name, address, and account creation date
- **FR-021**: System MUST display "Not provided" for address if not yet set
- **FR-022**: System MUST audit log all profile changes (email, address, password) with timestamp and type of change

### Non-Functional Requirements

- **NFR-001**: Profile update operations MUST complete within 2 seconds
- **NFR-002**: Email verification MUST arrive within 5 minutes of request
- **NFR-003**: Password change MUST invalidate all sessions within 5 seconds
- **NFR-004**: System MUST enforce HTTPS for all profile operations (no unencrypted transmission)
- **NFR-005**: Profile data MUST be displayed only to authenticated account owner
- **NFR-006**: Audit logs MUST be immutable and retained for 7 years per compliance requirements
- **NFR-007**: Password verification MUST use constant-time comparison to prevent timing attacks

## Success Criteria *(mandatory)*

- Customers can change email address in under 2 minutes end-to-end
- Email verification message arrives reliably within 5 minutes
- Address updates persist correctly and reflect on next statement generation
- Password changes complete within 2 seconds with 100% session invalidation
- Zero unauthorized profile changes (100% authentication enforcement)
- All profile modifications logged and auditable for compliance
- Email change verification has zero false positives (no legitimate changes rejected)
- 99% of email verifications successfully delivered and confirmed
- Profile view page loads within 1 second

## Key Entities

- **Account** (existing entity, extended)
  - `id`: Unique account identifier (UUID) — already defined
  - `email`: Customer email address (unique, validated) — updated when email change verified
  - `full_name`: Account holder full name — already defined
  - `address_street`: Street address (optional, max 255 characters)
  - `address_city`: City name (required if address set, max 100 characters)
  - `address_state`: State/province code (required if address set, max 10 characters)
  - `address_postal_code`: Postal code (required if address set, max 20 characters)
  - `address_country`: Country code (optional, defaults to US, ISO-3166-1 alpha-2)
  - `address_updated_date`: When address last modified (timestamp, nullable)
  - `password_hash`: Bcrypt hash of customer password — updated when password changed
  - `password_updated_date`: When password last changed (timestamp, for compliance)

- **EmailVerification** (new entity)
  - `id`: Unique verification token (UUID, cryptographically random)
  - `account_id`: Account requesting email change (foreign key)
  - `new_email`: New email address pending verification (unique index until verified)
  - `verification_code`: Short code sent in email (6-8 alphanumeric characters)
  - `created_date`: When verification request created (timestamp)
  - `expires_date`: When verification link expires (24 hours after creation)
  - `verified_date`: When customer confirmed (timestamp, null until confirmed)
  - `status`: Pending/Confirmed/Expired

- **ProfileAuditLog** (new entity)
  - `id`: Unique log entry identifier (UUID)
  - `account_id`: Account whose profile changed (foreign key)
  - `change_type`: Type of change (EMAIL_CHANGED, EMAIL_CHANGE_REQUESTED, ADDRESS_CHANGED, PASSWORD_CHANGED, EMAIL_VERIFICATION_SENT, EMAIL_VERIFICATION_CONFIRMED)
  - `old_value`: Previous value (nullable for certain changes)
  - `new_value`: New value (nullable for certain changes)
  - `ip_address`: IP address from which change was made
  - `user_agent`: Browser/client user agent string
  - `timestamp`: When change occurred (UTC)
  - `status`: Success/Failed/Reverted

## Assumptions

- **Single account update**: Customers update profiles individually; no batch/admin updates of customer profiles
- **Email uniqueness**: Email addresses are unique across all accounts; changing to email already in use rejected
- **Address format**: Addresses stored as separate fields (street, city, state, postal code, country), not free-form text
- **Email verification**: 24-hour verification window standard; expired verifications require new request
- **Password constraints**: Same password strength rules as registration (8+ chars, mixed case, number, symbol); no history of previous passwords enforced
- **Session invalidation**: Password change invalidates all existing tokens/sessions immediately; customer must re-authenticate
- **Audit immutability**: Audit logs cannot be deleted or modified once created; essential for compliance
- **Address optionality**: Address is optional during account creation; can be added/updated later; not required for banking operations
- **Compliance alignment**: Profile updates comply with GDPR (data accuracy, customer control) and CCPA (customer access to profile data)
- **Security**: All profile changes require re-authentication (current password or MFA if applicable); sensitive operations encrypted end-to-end
- **No blind changes**: Customers see current values before changing; cannot change profile of another account
