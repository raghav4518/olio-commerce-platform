# Backend 05 — Identity, Authentication, Sessions and Authorization

## 1. Identity model

Use one canonical user identity system across the three panels, with roles and policy scopes determining access. Do not create separate customer, admin and support credential stores unless the repository audit identifies a compelling security/compliance requirement and an approved ADR is written.

A single account may have more than one role only under an explicit owner policy. A user's customer role must not imply any staff capability. Staff accounts must be provisioned/approved through an authorized process; public registration cannot create staff roles.

## 2. Authentication baseline

Recommended browser model:

- Server-managed, opaque sessions.
- Session identifier delivered using a `Secure`, `HttpOnly` cookie with a deliberate `SameSite` policy and restrictive path/domain.
- Store only session lookup/hash material server-side where feasible; never log raw session cookie values.
- Rotate session on login, privilege change, password reset and step-up authentication.
- Revoke session on logout, account suspension, credential reset and administrator revocation.
- Enforce idle and absolute expiry through documented configurable policy; staff sessions should be shorter and more strongly protected than customer sessions.
- Use CSRF protection for state-changing cookie-authenticated requests plus Origin/Referer validation.
- Use exact CORS allowlists; no wildcard credentialed CORS.

If the repository uses an established identity provider or token-based architecture, the agent must compare it against this baseline and document cookie/token storage, rotation, refresh, revocation, audience and CSRF/XSS implications before retaining or replacing it.

## 3. Credential and account flows

### Registration

Registration policy is an owner decision. If self-registration is enabled:

1. Validate email and password policy.
2. Create a pending account (or account with unverified email according to design).
3. Send a single-use verification token; store only its hash.
4. Apply rate limits and generic responses that do not reveal whether an email exists where enumeration is a risk.
5. Activate/verify only after a valid, unexpired, single-use token is consumed.
6. Record consent/notice version and timestamp where legally required.

### Login

- Rate-limit and monitor credential stuffing/brute force.
- Use generic authentication errors; do not disclose whether account or password was incorrect.
- Use an adaptive password hashing algorithm such as Argon2id through a maintained library; parameters must be reviewed for production hardware and rehash migration.
- Require MFA for owner and staff accounts as the proposed baseline. Support customer MFA/passkeys as a product decision; never block accessible recovery.
- Create/rotate session only after all required authentication factors pass.
- Detect suspicious sign-in patterns without collecting excessive device data.

### Password reset

- Return generic request responses.
- Store only token hashes, expire tokens, use single-use semantics, rate-limit sending and invalidate prior tokens according to policy.
- After reset, revoke existing sessions and notify the account owner.
- Do not email a password or reveal whether an address has an account.

### Account closure and recovery

Support account export, closure, correction and recovery flows subject to legally approved retention. Do not hard-delete financial/order records that must be preserved. Anonymize or pseudonymize personal details only in accordance with documented legal and operational requirements.

## 4. Staff authentication and step-up controls

- MFA mandatory for Owner/Admin and Support/Operations accounts under the proposed baseline.
- Re-authentication or step-up authentication required for sensitive actions: refund approval/execution, changing gateway credentials, changing staff roles, disabling MFA, exporting bulk personal data, changing storage/security policies and revoking/restoring customer access.
- Step-up proof must be recent and bound to the action/session; a previously authenticated browser tab must not authorize a sensitive action indefinitely.
- Support staff cannot elevate their own role or grant permissions.
- Owner-level account recovery must have a documented emergency procedure and audit trail.

## 5. Authorization model

Use deny-by-default RBAC with explicit permissions and object-level policies. RBAC alone is insufficient: every request must verify the user can act on the specific object and its current state.

### Permission naming convention

`domain.resource.action`, for example:

- `catalog.ebooks.read`, `catalog.ebooks.create`, `catalog.ebooks.edit`, `catalog.ebooks.publish`, `catalog.ebooks.archive`
- `orders.read`, `payments.read`, `refunds.read`, `refunds.approve`, `refunds.execute`
- `customers.read`, `customers.suspend`, `entitlements.suspend`, `entitlements.restore`
- `support.queue.read`, `support.tickets.assign`, `support.messages.send`, `support.notes.create`
- `security.staff.manage`, `security.roles.manage`, `audit.read`, `settings.payments.manage`

Use permission strings consistently in API policy, tests and audit events. Do not derive security from frontend route or menu visibility.

## 6. Proposed role matrix (must be owner-approved)

| Capability | Owner | Operations manager | Catalog manager | Finance manager | Marketing manager | Support agent | Analyst |
|---|---:|---:|---:|---:|---:|---:|---:|
| Full configuration | Yes | No | No | No | No | No | No |
| Catalog read/edit | Yes | Limited | Yes | No | Read | Read | Read |
| Publish/unpublish | Yes | Optional | Configurable | No | No | No | No |
| Orders read | Yes | Yes | No | Yes | Aggregated | Limited lookup | Aggregated |
| Payment read | Yes | Limited | No | Yes | No | Masked status | Aggregated |
| Refund approve/execute | Yes | No by default | No | Yes, policy/threshold | No | Request only | No |
| Customer data | Yes | Limited | No | Limited | Segmented/consented | Minimum necessary | Aggregated |
| Support tickets | Yes | Yes | No | Limited related cases | No | Assigned/queue | No |
| Campaigns/coupons | Yes | Read | No | Financial visibility | Yes | No | Read metrics |
| Roles/permissions | Yes | No | No | No | No | No | No |
| Audit logs | Yes | Limited | No | Finance subset | No | Ticket subset | No |
| Bulk export | Yes, audited | Scoped | No | Finance subset | Consented segments | No | Aggregated |

This is a starting matrix only. The owner must approve exact role permissions, scopes and approval thresholds. Prefer custom roles assembled from atomic permissions rather than hard-coded broad roles.

## 7. Support access safeguards

- Support staff receive only the minimum customer/order data needed for the ticket.
- Sensitive customer lookups should require a ticket or reason code and be audit logged.
- Mask payment instruments and provider data; support must never see card PAN/CVV or gateway secrets.
- Staff-only notes are never included in customer API responses.
- Access to a ticket does not automatically grant access to every order/account resource.
- Bulk exports and unrestricted customer search are not support permissions.

## 8. Object authorization requirements

Automated tests must prove:

- Customer A cannot read Customer B's order, library, download grant, reading progress, profile or support ticket by changing an ID.
- Support Agent A cannot read an unassigned/restricted ticket unless queue policy permits.
- Catalog Manager cannot issue refunds or change payment gateway credentials.
- Finance Manager cannot publish or edit ebook source files unless explicitly granted.
- A suspended account cannot create a new session or use existing sessions.
- Revoked sessions stop working promptly.
- An inactive entitlement cannot generate reader/download grants.
- Admin APIs reject missing or insufficient permissions regardless of UI state.

## 9. Session and security lifecycle

Record: authenticated user, session creation, authentication strength, MFA completion, last activity, revocation, reason and relevant coarse context. Avoid persistent invasive fingerprinting. Provide customers with session logout controls; staff should have administrative session revocation under permission and audit.

## 10. Standards alignment

Use NIST SP 800-63B-4 as an authentication reference and OWASP ASVS 5.0.0 for verification planning. These references do not automatically confer compliance. Confirm applicable requirements and identity-provider capabilities before release.
