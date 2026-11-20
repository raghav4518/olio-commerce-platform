# Backend 14 — Standards and Compliance Reference Register

**Verified:** 19 November 2026.  
**Purpose:** Provide an up-to-date standards reference for engineering and QA. This register is not a certification or legal opinion.

## 1. API contracts

- **OpenAPI Specification 3.2.1** — https://spec.openapis.org/oas/v3.2.1.html
- OAS 3.2.1 was published on 10 September 2026. Use it if the repository's parser, validators, code generators and API gateway support it correctly. If tooling support is incomplete, use the latest supported OAS 3.1.x version and record the compatibility reason in an ADR. Do not label a 3.1 document as 3.2.
- **RFC 9457 — Problem Details for HTTP APIs** — https://www.rfc-editor.org/rfc/rfc9457

## 2. Application and API security

- **OWASP ASVS 5.0.0** — https://asvs.owasp.org/
- **OWASP API Security Top 10** — https://owasp.org/API-Security/
- Use ASVS as a verification framework. Select target requirements based on threat model, data sensitivity and risk. Map controls to tests and evidence. Do not claim ASVS compliance merely because this document references it.

## 3. Authentication

- **NIST SP 800-63B-4 — Authentication and Authenticator Management** — https://csrc.nist.gov/pubs/sp/800/63/B/4/final
- Published July 2025. Use as a high-quality authentication reference, while recognizing its scope is digital identity guidance for government systems and it is not automatically a private-sector legal requirement.

## 4. Accessibility and digital publications

- **WCAG 2.2** — https://www.w3.org/TR/WCAG22/
- **EPUB 3.3** — https://www.w3.org/TR/epub-33/
- WCAG 2.2 is a cross-device accessibility recommendation and should guide the later frontend verification work. Backend API states, validation and authentication flows must enable accessible UI implementation.
- EPUB 3.3 is a W3C Recommendation and should inform format validation and reader-engine evaluation if EPUB is approved for launch. Confirm the specific EPUB conformance profile and accessibility requirements appropriate to the product.

## 5. Payments and privacy

- Confirm the payment provider/acquirer integration model and resulting PCI DSS scope with qualified payment/security advisors. Hosted checkout can reduce exposure but does not itself prove compliance.
- Determine the applicable privacy, data-protection, ecommerce, consumer, tax and records laws for every launch jurisdiction. If India is a launch market, obtain current legal review of the DPDP Act, applicable Rules and commencement/notifications in force at launch, alongside other applicable Indian laws.
- Treat privacy/consent, cookie/analytics policy, refund rules, tax/invoice policy and data retention as owner/legal decisions recorded in the decision register.

## 6. Version and change management

At each major implementation or release:

1. Verify the current version/status of referenced standards from the official publisher.
2. Check framework/library support and known security advisories.
3. Update this register and the verification matrix.
4. Record any deviation and compensating control in an ADR.
5. Re-run security and compatibility testing after meaningful upgrades.

## 7. Source verification notes

Official sources checked for this package:

- OpenAPI 3.2.1 specification and publication history: https://spec.openapis.org/oas/v3.2.1.html
- OWASP ASVS project page and stable release: https://asvs.owasp.org/
- W3C WCAG 2.2 Recommendation: https://www.w3.org/TR/WCAG22/
- W3C EPUB 3.3 Recommendation: https://www.w3.org/TR/epub-33/
- NIST SP 800-63B-4 final publication: https://csrc.nist.gov/pubs/sp/800/63/B/4/final
