# Appendix 96 — Medhavy LTI 1.3 Setup Guide

*Developer-facing. This is not prose for the author — it is a minimal-viable sequence for the developer standing up the Medhavy LTI 1.3 integration and the institution approving it. Read Chapter 19 for context and rationale.*

---

## How to Use This Appendix

Each step below is labeled with its owner: **[Developer]**, **[Institution]**, or **[Both]**. Steps labeled **[Institution]** cannot be completed by the developer alone; they require the institution's Canvas administrator, privacy office, or IT security team. Steps labeled **[Both]** require coordination between the developer and institution before proceeding.

This appendix covers the standard path: LTI 1.3 + LTI Advantage (NRPS, Deep Linking, AGS). It does not cover custom Canvas REST API integration. Do not substitute the custom API path without explicit approval from the institution's LMS administrator.

Reference specifications:
- LTI 1.3 Core: IMS Global / 1EdTech, Version 1.0 Final
- LTI Advantage NRPS: 1EdTech Version 2.0
- LTI Advantage AGS: 1EdTech Version 2.0
- LTI Advantage Deep Linking: 1EdTech Version 2.0
- OIDC: OpenID Connect Core 1.0

---

## Step 1 — Institutional Review Gate (Complete Before Any Configuration)

**[Institution]** — This step must complete before any LTI registration or student-data flow begins.

| Sub-step | Owner | What is required |
|---|---|---|
| 1.1 FERPA review | Institution (Privacy Office) | Confirm Medhavy operates as a FERPA-covered School Official under a signed data processing agreement (DPA). Confirm no student PII flows to Medhavy before DPA is signed and in effect. |
| 1.2 Security review | Institution (IT Security) | Review Medhavy's infrastructure: hosting environment, data residency, encryption at rest and in transit, OIDC/JWT handling, data deletion and retention policy, incident response procedure. Produce a written approval or conditional approval with remediation steps. |
| 1.3 Canvas admin approval | Institution (Canvas Admin) | Confirm the institution will register Medhavy as a Developer Key. Obtain the institution's Canvas environment URL (e.g., `https://northeastern.instructure.com`) and the name of the Canvas admin who will complete Step 2. |

**Gate condition:** Do not proceed to Step 2 until Steps 1.1, 1.2, and 1.3 are complete and documented. If any sub-step is blocked, surface the blocker explicitly — do not work around it.

---

## Step 2 — Register Medhavy as an LTI 1.3 Developer Key in Canvas

**[Institution]** — Canvas Administrator performs this step in Canvas Admin → Developer Keys.

| Field | Value (Developer provides) |
|---|---|
| Key type | LTI Key |
| Key name | `Medhavy AI Tutor` (or as agreed with the institution) |
| OIDC initiation URL | Medhavy's OIDC login endpoint (e.g., `https://app.medhavy.com/lti/login`) |
| Redirect URI(s) | Medhavy's redirect URI(s) after OIDC completion (e.g., `https://app.medhavy.com/lti/callback`) |
| JWKS URL | Medhavy's public JSON Web Key Set (e.g., `https://app.medhavy.com/lti/jwks`) |
| Target link URI (default) | Medhavy's default launch target (e.g., `https://app.medhavy.com/lti/launch`) |
| Method | Public JWK URL (preferred) or Manual JWK entry |
| LTI Advantage services | Enable: Names and Role Provisioning Services, Assignment and Grade Services, Deep Linking |
| Placements | Course Navigation (optional), Assignment Selection, Link Selection (for module items) |

**[Institution]** After saving the Developer Key:
- Note the **Client ID** Canvas assigns to this key — the Developer will need it to complete Medhavy-side registration.
- Set the key to **ON** (enabled) for the relevant sub-account(s) or root account.
- Provide the following to the Developer:
  - Canvas environment URL
  - Client ID
  - Canvas Authorization Server URL (typically `https://<canvas-domain>/api/lti/authorize_redirect`)
  - Canvas JWKS URL (typically `https://<canvas-domain>/api/lti/security/jwks`)
  - Canvas OAuth2 Token endpoint (typically `https://<canvas-domain>/login/oauth2/token`)

[verify — Canvas Developer Key field names and paths are subject to change with Canvas releases; confirm against current Canvas Admin documentation at canvas.instructure.com]

---

## Step 3 — First LTI Launch (OIDC Flow Verification)

**[Developer]** — Confirm the complete OIDC launch sequence works end to end from a Canvas test course before configuring any course-specific settings.

**Launch sequence:**

1. Instructor or developer adds a Medhavy External Tool module item to a Canvas test course module (using the Canvas Module editor → Add Item → External Tool → Medhavy).
2. Instructor opens the item as a student (Student View) or a test student account clicks the link.
3. Canvas sends an OIDC initiation request to Medhavy's OIDC login URL, including `iss` (Canvas domain), `login_hint`, `target_link_uri`, and `lti_message_hint`.
4. Medhavy responds with an OIDC authentication request redirect to Canvas's Authorization Server URL.
5. Canvas validates the `client_id` and redirect URI, then POSTs a signed `id_token` (JWT) to Medhavy's redirect URI.
6. Medhavy validates the JWT: verify signature against Canvas's JWKS, validate `iss`, `aud`, `nonce`, `exp`, `iat`, and `https://purl.imsglobal.org/spec/lti/claim/version` = `1.3.0`.
7. Medhavy parses the launch claims (see Step 4) and completes the launch — user sees Medhavy interface.

**[Developer]** Verification checklist:
- [ ] OIDC flow completes without errors
- [ ] JWT signature validates against Canvas JWKS
- [ ] `sub` claim (stable user identifier) is present and consistent across launches
- [ ] `https://purl.imsglobal.org/spec/lti/claim/context` is present (course ID, title)
- [ ] `https://purl.imsglobal.org/spec/lti/claim/roles` is present
- [ ] Launch completes in < 3 seconds for a typical network
- [ ] Failure mode tested: expired JWT, invalid nonce, mismatched `client_id` — all return an appropriate error, not a crash

---

## Step 4 — Account Auto-Create / Lookup from Launch Claims

**[Developer]** — On each launch, Medhavy must look up or create a user account using the Canvas-provided identity claims. Do not rely on email alone as the primary key — Canvas email is not guaranteed unique across contexts.

| Claim | Use |
|---|---|
| `sub` | Stable, opaque user identifier from Canvas — use as the primary Medhavy user key for this institution |
| `https://purl.imsglobal.org/spec/lti/claim/lis` → `person_sourcedid` | Institution's SIS ID for the user (optional, may not be present) |
| `email` (if present) | Secondary display / communication field — not the primary key |
| `name`, `given_name`, `family_name` | Display name — update on each launch to catch name changes |
| `https://purl.imsglobal.org/spec/lti/claim/roles` | Determine user role: instructor, student, TA |

**Account creation logic (recommended):**

```
On launch:
  user = lookup by (institution_id, sub_claim)
  if not found:
    create user with (institution_id, sub_claim, email, display_name, role)
  else:
    update display_name and role if changed
  load or create course session for (user, context_id, resource_link_id)
```

**[Developer]** — Never store the Canvas JWT beyond the launch handshake. Parse claims, store what you need, discard the token.

---

## Step 5 — NRPS Roster Sync

**[Developer]** — Use NRPS to pull the current course roster for analytics, gradebook alignment, and proactive student identification.

**Endpoint pattern:** Canvas provides the NRPS endpoint URL in the launch JWT at `https://purl.imsglobal.org/spec/lti-nrps/claim/namesroleservice` → `context_memberships_url`.

**Authentication:** Medhavy must obtain an OAuth 2.0 access token from Canvas's token endpoint using the Client Credentials grant with `scope=https://purl.imsglobal.org/spec/lti-nrps/scope/contextmembership.readonly`.

**Sync behavior:**
- Pull roster on first instructor launch for a course, and on a scheduled basis (e.g., daily during active course periods).
- Store: `user_id` (Canvas `sub`), role, status (active/inactive).
- Use for: analytics coverage (identify enrolled students with zero tutor interactions), gradebook consistency checks.

**[Developer]** Verification checklist:
- [ ] NRPS endpoint URL extracted from launch claims correctly
- [ ] OAuth 2.0 token request succeeds with correct scope
- [ ] Pagination handled (NRPS responses may be paginated via `Link` header)
- [ ] Inactive / dropped enrollments handled correctly (do not treat as active)

[verify — NRPS scope string and pagination behavior against current 1EdTech NRPS 2.0 specification]

---

## Step 6 — Deep Linking (Adding Medhavy Items to Canvas Modules)

**[Developer]** — Deep Linking lets instructors pick Medhavy content from inside Canvas's module editor without hard-coding URLs.

**Flow:**
1. Instructor opens Canvas module, clicks Add Item → External Tool → Medhavy.
2. Canvas sends a Deep Linking request launch (message type: `LtiDeepLinkingRequest`) to Medhavy's launch URL.
3. Medhavy displays a content picker: available chapter tutors, formative activities, analytics links.
4. Instructor selects one or more items.
5. Medhavy returns a `LtiDeepLinkingResponse` JWT containing `content_items` — an array of `LtiResourceLink` objects, each with a `url`, `title`, and optional `custom` parameters.
6. Canvas stores the resource links as module items.

**Content item types to support at minimum:**
- Chapter tutor (one per chapter, carrying `chapter_id` in `custom`)
- Formative activity (carrying `activity_id` and `course_id` in `custom`)
- Instructor analytics dashboard (role-gated at launch time)

**[Developer]** Verification checklist:
- [ ] Deep Linking launch recognized by message type claim
- [ ] Content picker renders cleanly inside Canvas module editor iframe
- [ ] `LtiDeepLinkingResponse` JWT signed with Medhavy's private key, verified by Canvas against Medhavy's JWKS
- [ ] Created module items launch correctly via the standard OIDC flow (Step 3)
- [ ] `custom` parameters round-trip correctly from Deep Linking response to subsequent launches

---

## Step 7 — AGS Grade and Completion Passback

**[Developer]** — Use AGS to write activity completion scores and grades from Medhavy back to the Canvas gradebook.

**Endpoint pattern:** Canvas provides the AGS endpoint URL in the launch JWT at `https://purl.imsglobal.org/spec/lti-ags/claim/endpoint` → `lineItems`.

**Authentication:** OAuth 2.0 access token from Canvas's token endpoint, scopes:
- `https://purl.imsglobal.org/spec/lti-ags/scope/lineitem` (read/write line items)
- `https://purl.imsglobal.org/spec/lti-ags/scope/result.readonly` (read results)
- `https://purl.imsglobal.org/spec/lti-ags/scope/score` (write scores)

**Line item creation:** For each gradable Medhavy activity, create a Canvas line item via AGS with a `label`, `scoreMaximum`, and optional `resourceId` (Medhavy activity ID). Do this on first instructor launch or when the instructor configures the activity.

**Score passback:** When a student completes a gradable Medhavy activity:
1. Obtain or confirm the line item URL for this activity.
2. POST a score object to `{lineItemUrl}/scores`:
   - `userId`: Canvas `sub` from the student's launch
   - `scoreGiven`, `scoreMaximum`
   - `activityProgress`: `Completed`
   - `gradingProgress`: `FullyGraded`
   - `timestamp`: ISO 8601

**[Developer]** Verification checklist:
- [ ] AGS endpoint URL extracted from launch claims correctly
- [ ] Line items created for all gradable activities before any student attempts
- [ ] Score passback posts successfully and appears in Canvas gradebook
- [ ] Passback is idempotent (resubmitting the same score does not create duplicate gradebook entries)
- [ ] Edge cases tested: student has no launch session (grade-only passback), score of zero, max score, late submission

[verify — AGS scope strings and score object schema against current 1EdTech AGS 2.0 specification]

---

## Step 8 — Instructor Analytics Dashboard

**[Developer]** — The analytics dashboard is accessible to instructors via an LTI launch (role-gated) or a standalone Medhavy login. Minimum viable dashboard for launch:

| Metric | Grain | Notes |
|---|---|---|
| Tutor sessions opened | Per student, per chapter | Identify non-engaging students |
| Session duration | Per student, per session | Proxy for engagement depth |
| Activity completions | Per student, per activity | Ties to AGS passback |
| Question frequency by concept | Course aggregate | Identifies high-confusion topics |
| Students with zero tutor interactions | Course aggregate | Actionable by instructor |

**[Developer]** The dashboard must be role-gated at launch time using the `roles` claim — instructors and TAs see course-level data; students see only their own data.

**[Institution]** Confirm with your privacy office: does displaying roster-level analytics in the Medhavy dashboard require an additional notice to students beyond the course syllabus disclosure? Some FERPA interpretations require explicit disclosure when AI systems log and analyze student learning behavior.

---

## Step 9 — Deployment Verification Checklist (Pre-Launch)

**[Both]** — Complete before enabling the integration for any live student enrollment.

| Check | Owner | Pass condition |
|---|---|---|
| LTI registration active | Institution | Developer Key shows ON in Canvas Admin for the target sub-account |
| OIDC launch works for student role | Developer | Test student account launches without error |
| OIDC launch works for instructor role | Developer | Instructor account launches and sees correct role-gated UI |
| NRPS roster pulls correctly | Developer | All enrolled test students appear; dropped students excluded |
| Deep Linking creates module items correctly | Developer + Institution | Module items created in test course launch correctly |
| AGS passback writes to gradebook | Developer | Test completion score appears in Canvas gradebook |
| FERPA DPA signed and on file | Institution | Privacy office confirms |
| Security review approval documented | Institution | IT Security approval on file |
| Student data disclosure in syllabus | Institution | Course syllabus includes disclosure that AI tutor interaction data is logged |
| Rollback plan documented | Both | Developer has documented procedure to disable LTI key if a critical issue is found post-launch |

---

## Notes and Constraints

**Do not build a parallel Canvas REST API integration.** Canvas REST API tokens are user-scoped, not institution-scoped. LTI 1.3 is the maintainable path. If the institution's IT refuses LTI registration, escalate before building a workaround.

**The Medhavy SDD (System Design Document)** covers the full Medhavy application architecture — database schema, session management, AI context scoping, analytics pipeline. This appendix covers only the LTI 1.3 integration boundary. Refer to the Medhavy SDD for application internals.

**Specification freshness.** LTI 1.3 and LTI Advantage are maintained by 1EdTech. Canvas's implementation follows the 1EdTech specifications but may lag or extend them. Before implementation, verify current Canvas LTI 1.3 documentation at canvas.instructure.com/doc/api/file.lti_dev_key_config.html. [verify — this path subject to Canvas documentation restructuring]

**Timeline estimate (typical institutional process):**
- FERPA + security review: 4–8 weeks
- Canvas admin registration: 1–3 business days after approval
- Developer integration and testing: 2–4 weeks (parallel with institutional review where possible)
- Total calendar time from initiation to live: 6–12 weeks

Plan accordingly. Initiate the institutional process at least one term before the intended course launch date.

---

## References

1. 1EdTech. *IMS LTI 1.3 Core Specification*, Version 1.0 Final. https://www.imsglobal.org/spec/lti/v1p3/
2. 1EdTech. *LTI Advantage: Names and Role Provisioning Services (NRPS)*, Version 2.0. https://www.imsglobal.org/spec/lti-nrps/v2p0
3. 1EdTech. *LTI Advantage: Assignment and Grade Services (AGS)*, Version 2.0. https://www.imsglobal.org/spec/lti-ags/v2p0
4. 1EdTech. *LTI Advantage: Deep Linking*, Version 2.0. https://www.imsglobal.org/spec/lti-dl/v2p0
5. Instructure. *Canvas LTI 1.3 Developer Key Configuration*. https://canvas.instructure.com/doc/api/file.lti_dev_key_config.html
6. OpenID Foundation. *OpenID Connect Core 1.0*. https://openid.net/specs/openid-connect-core-1_0.html
7. U.S. Department of Education. *FERPA: General Guidance for Students*. https://studentprivacy.ed.gov/ferpa
8. Chapter 19 — Medhavy: the AI-Tutor Layer via LTI. See `19-medhavy-lti.md`.
