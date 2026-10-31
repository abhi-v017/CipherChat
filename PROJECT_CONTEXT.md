# CipherChat --- Master Project Context

**Last updated:** 31 October 2026\
**Purpose:** Shared source of truth for Claude Web, Antigravity Desktop,
and any coding assistant.

> Important: This document records the intended design and decisions
> made so far. It does **not** prove that any code, feature, test,
> deployment, or security review has already been completed. Always
> inspect the actual repository before claiming implementation status.

## 1. Project summary

**Working name:** CipherChat (placeholder; rename later if desired)\
**Type:** Open-source, self-hostable, responsive web messaging
application\
**Primary goal:** Build a manageable but technically credible
solo-developer project that demonstrates full-stack engineering,
real-time communication, database design, testing, Docker,
documentation, and responsible security engineering.

CipherChat aims to support private one-to-one messaging with end-to-end
encryption (E2EE). Encryption/decryption must happen on clients using a
maintained, established protocol/library---not custom cryptographic
algorithms. The server should route and persist ciphertext, not message
plaintext.

**Target clients:** Modern desktop browsers and Android mobile browsers.
This is a responsive web application, not a separate native Android app
in v1.

**Audience:** Developers and open-source contributors; the project
should be understandable, self-hostable, and honest about its security
limitations.

## 2. Product principles

1.  Build a small, complete, tested product before adding advanced
    features.
2.  Prefer boring, maintainable technology over unnecessary complexity.
3.  Security claims must match what is actually implemented and tested.
4.  Never invent cryptography or label placeholder encryption as E2EE.
5.  Keep secrets, private keys, plaintext messages, passwords, and
    session tokens out of logs and source control.
6.  Document architecture, setup, threat model, limitations, and
    contribution workflow.
7.  Make the UI accessible and usable on narrow Android screens as well
    as desktop.
8.  Make changes incrementally; run checks and report their real
    outcomes.
9.  Do not silently replace agreed technology or architecture. Explain
    trade-offs and request approval for significant changes.
10. Keep this document and the repository's architecture/security
    decisions aligned.

## 3. Intended v1 scope

### Core features

-   Account registration and login.
-   Secure session handling and logout/revocation.
-   One-to-one conversations.
-   Real-time message transport with Socket.IO.
-   Client-side encryption using a vetted protocol/library, once the
    library choice and browser support have been verified.
-   Ciphertext persistence in PostgreSQL.
-   Offline message retrieval/delivery and acknowledgements.
-   Conversation membership and authorization checks.
-   Identity/key verification and clear key-change warnings, as
    supported by the selected protocol.
-   Responsive layout for desktop and Android mobile browsers.
-   Docker Compose development environment.
-   Automated unit/integration tests and CI.
-   Clear README, threat model, security policy, license, and
    contribution guide.

### Explicitly out of scope for v1

-   Group chats.
-   Voice/video calls.
-   File and image sharing.
-   AI features.
-   Microservices.
-   Kubernetes.
-   Redis and multi-instance scaling.
-   Separate native Android application.
-   Complex multi-device synchronization.
-   Custom cryptographic algorithms.

Only add a stretch feature after the core experience works end-to-end
and its tests pass.

## 4. Technology choices

### Planned stack

-   **Frontend:** React, TypeScript, Vite, Tailwind CSS.
-   **Backend:** Node.js, Express, TypeScript.
-   **Realtime:** Socket.IO.
-   **Database:** PostgreSQL.
-   **ORM/migrations:** Prisma.
-   **Encryption:** A maintained, established protocol/library selected
    after a documented evaluation.
-   **Testing:** Vitest; Supertest for HTTP integration tests; add
    browser end-to-end testing when practical.
-   **Local infrastructure:** Docker and Docker Compose.
-   **CI:** GitHub Actions.
-   **Repository:** One repository, not a monorepo.
-   **Package manager:** Choose one (npm is the default unless the
    existing repository already uses another) and commit the lockfile.

### Do not add yet

Redis, Kubernetes, microservices, message queues, vector databases, or a
separate worker service. Reconsider Redis only if a demonstrated scaling
requirement needs shared Socket.IO state or presence across multiple
backend instances.

### Encryption-library decision is still open

Do not assume a library is selected. Evaluate candidates for: -
Browser/WebCrypto compatibility and supported platforms. - Protocol
completeness and correct handling of session setup, ratchets, prekeys,
and identity changes where applicable. - Maintenance activity, release
history, security advisories, documentation, and known limitations. -
License compatibility. - API stability and whether third-party use is
supported. - Test vectors, interoperability tests, and ease of safe
integration. - Multi-device implications and recovery limitations.

The Signal `libsignal` codebase is one possible research lead, not an
automatic choice: verify its current license, browser support,
maintenance, API stability, and external-use support before committing.
Never implement a homemade substitute if a library proves difficult.
Record the decision and rationale in `docs/decisions/`.

## 5. High-level architecture

``` text
React + TypeScript browser client (desktop / Android)
    ├── UI, local state, session UX
    ├── vetted protocol integration and client-side encryption
    ├── REST API client
    └── Socket.IO client
             │ HTTPS / WSS
             ▼
Node.js + Express + Socket.IO server
    ├── authentication and session middleware
    ├── REST routes and authorization
    ├── socket event validation and authorization
    ├── message persistence/delivery services
    └── Prisma ORM
             │
             ▼
         PostgreSQL
    users, sessions, conversations, membership,
    ciphertext, delivery/acknowledgement state,
    and protocol-specific public key material
```

The browser encrypts outgoing message content before sending it to the
server. The recipient client decrypts it. The server must not need
plaintext to deliver or store messages. Some metadata---such as account
identifiers, conversation membership, timing, IP information available
to infrastructure, message sizes, and delivery state---may remain
visible. Do not promise metadata secrecy unless it is specifically
designed and verified.

## 6. Initial repository structure

Keep the layout straightforward and adjust only when real complexity
requires it.

``` text
cipherchat/
├── client/
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── hooks/
│       ├── lib/
│       ├── crypto/             # Protocol adapter; never homemade crypto
│       ├── socket.ts
│       ├── App.tsx
│       └── main.tsx
├── server/
│   └── src/
│       ├── routes/
│       ├── middleware/
│       ├── services/
│       ├── socket.ts
│       └── index.ts
├── prisma/
│   └── schema.prisma
├── tests/
├── docs/
│   ├── threat-model.md
│   ├── architecture.md
│   ├── decisions/
│   └── security.md
├── docker-compose.yml
├── .env.example
├── .gitignore
├── README.md
├── CONTRIBUTING.md
├── LICENSE
└── package.json
```

Use the actual repository structure as the source of truth once files
exist. Do not create duplicate configs or move folders without
inspecting what is already present.

## 7. Initial data model (conceptual, not final schema)

-   **User:** `id`, `username`, `email`, `passwordHash`, `createdAt`.
-   **Session:** `id`, `userId`, `tokenHash`, `expiresAt`, `revokedAt`,
    timestamps as needed.
-   **Conversation:** `id`, `createdAt`.
-   **ConversationMember:** `conversationId`, `userId`, `joinedAt`;
    unique constraint on `(conversationId, userId)`.
-   **Message:** `id`, `conversationId`, `senderId`, `clientMessageId`,
    `ciphertext`, `createdAt`; enforce an appropriate
    uniqueness/idempotency constraint.
-   **Device / protocol key records:** design these only after selecting
    the protocol. Store only the public/protocol material the library
    requires on the server.

This is a starting model. Review normalization, indexes, deletion
behavior, foreign keys, retention, and the protocol's actual storage
needs before finalizing migrations.

**Never store server-side:** message plaintext, client private identity
keys, decryption secrets, raw passwords, raw session tokens, or
cryptographic secrets in logs. Store password hashes using a suitable
password-hashing library and store only session-token hashes if using
opaque bearer sessions.

## 8. Planned API and realtime behavior

Possible REST endpoints (names may change based on implementation): -
`POST /api/v1/auth/register` - `POST /api/v1/auth/login` -
`POST /api/v1/auth/logout` - `GET /api/v1/conversations` -
`POST /api/v1/conversations` -
`GET /api/v1/conversations/:id/messages` - `POST /api/v1/messages` -
`POST /api/v1/messages/:id/ack` - Device and public-key/prekey endpoints
only as required by the chosen protocol.

Possible Socket.IO events: - `connection:ready` - `message:send` -
`message:received` - `message:ack` - `message:read` -
`presence:update` - `error`

Treat these as proposed names, not a completed contract. Define event
payloads, validation schemas, error shapes, acknowledgement behavior,
and authorization before relying on them.

Every HTTP route and socket event must validate input and verify that
the authenticated user is authorized for the requested conversation.
Handle reconnects, duplicate sends, message ordering, offline delivery,
rate limits, payload size limits, origin policy, and errors
deliberately. Never trust a conversation ID merely because a client sent
it.

## 9. Security and privacy requirements

-   Use HTTPS/WSS in deployed environments.
-   Use secure password hashing through a maintained library.
-   Use safe session/cookie settings appropriate to the chosen auth
    design; mitigate CSRF if cookies are used.
-   Validate all request bodies, query parameters, and socket payloads
    with schemas.
-   Apply rate limits to login, registration, and abuse-prone endpoints.
-   Restrict CORS/origins intentionally; do not use permissive
    production defaults.
-   Enforce conversation membership on every read, send,
    acknowledgement, and state change.
-   Never log message bodies, credentials, session tokens, private keys,
    or decrypted content.
-   Avoid secrets in client bundles, Git, screenshots, issue reports,
    and CI logs.
-   Keep dependencies updated and review advisories.
-   Define account deletion, data retention, backup, and key-loss
    behavior.
-   Provide key/identity verification and warn users when identities
    change, as supported by the protocol.
-   Explain what the server can see and what it cannot.
-   Create a threat model covering malicious server operators, network
    attackers, compromised endpoints, account takeover, metadata
    exposure, lost devices, and dependency compromise.
-   Do not call the product "unhackable," "military-grade," or
    production-secure merely because a crypto library is used.
-   A serious security review is still needed before making strong
    production security claims.

## 10. UI and UX direction

Build a clean, modern, calm messaging interface---not a feature-packed
social network.

### Desktop

-   Conversation/sidebar navigation.
-   Main chat panel.
-   Clear connection and message delivery states.
-   Account/settings and identity verification access.

### Android/mobile browser

-   Mobile-first responsive layout.
-   Conversation list and chat screen should work at narrow widths.
-   Touch-friendly controls, readable typography, safe-area-aware
    spacing, and no horizontal overflow.
-   Keyboard should not cover the composer.
-   Loading, empty, offline, error, and reconnect states should be
    designed.
-   Respect reduced-motion preferences and maintain accessible contrast.

Do not spend weeks polishing animations before the core message flow
works. Build a usable, accessible baseline first.

## 11. Development roadmap

This is an estimate, not a deadline promise. The encryption-library
decision may change the schedule.

### Phase 0 --- Inspect and plan

-   Inspect the current repo and tools.
-   Confirm Node/npm versions and package manager.
-   Create a small implementation plan and initial issues.
-   Record the initial architecture decision and crypto evaluation
    criteria.

### Phase 1 --- Foundation

-   Create client/server setup.
-   Add TypeScript, lint/format scripts, environment validation.
-   Add Docker Compose for local PostgreSQL and app services as
    appropriate.
-   Confirm a clean clone can run from documented instructions.

### Phase 2 --- Database and authentication

-   Create Prisma schema and migrations.
-   Implement registration/login/logout and session
    expiration/revocation.
-   Add validation and authorization tests.
-   Never use real credentials in test fixtures or commits.

### Phase 3 --- Basic conversation flow

-   Implement one-to-one conversations and membership.
-   Implement real-time transport and persistence using clearly labelled
    development/test fixtures only.
-   Build the responsive UI, reconnect handling, acknowledgements, and
    idempotency.
-   Do not present plaintext demo mode as encrypted or ship it as the
    secure product.

### Phase 4 --- Protocol integration

-   Finish the protocol/library evaluation first.
-   Integrate the established protocol according to its documented
    usage.
-   Handle device identity, session setup, public-key material,
    verification, key changes, and error states.
-   Test using library test vectors and multiple client instances where
    possible.
-   If the protocol cannot be integrated safely in the browser, revisit
    the architecture rather than faking encryption.

### Phase 5 --- Hardening and quality

-   Add unit/integration tests for auth, authorization, messaging,
    reconnects, duplicates, and error cases.
-   Add end-to-end tests for two clients exchanging encrypted messages.
-   Confirm ciphertext persistence and no plaintext leakage in logs.
-   Add dependency checks and CI.
-   Review rate limits, input limits, CORS, session controls, and threat
    model.

### Phase 6 --- Open-source readiness

-   Complete README, architecture diagram, threat model, limitations,
    setup instructions, license, contribution guide, and security
    reporting policy.
-   Verify a clean checkout and Docker setup from scratch.
-   Add a small, honest demo only if it can be deployed safely.
-   Capture real measured performance metrics only after measuring them.

A focused MVP may take roughly 6--8 weeks, but do not sacrifice protocol
correctness to hit a date.

## 12. Definition of done

A feature is not complete merely because its UI renders.

For the v1 release: - A clean clone can be installed and run from
documented steps. - Database migrations work from a clean database. -
Registration, login, logout, expiration, and revocation behave
correctly. - Unauthorized users cannot read or send to conversations
they do not belong to. - Two separate clients can exchange messages in
real time. - Offline/reconnect behavior and duplicate submissions are
tested. - Messages persist as ciphertext, and clients decrypt through
the chosen protocol. - The protocol's identity/key-change behavior is
handled and documented. - Tests and CI pass; failures are not hidden or
described as successes. - No plaintext, passwords, session tokens, or
private keys appear in server storage/logs. - The UI works on Android
mobile browsers and desktop browsers. - README, threat model, security
limitations, and license are present. - Security claims are
proportionate to the tests and review actually performed.

## 13. Git, workflow, and collaboration rules for AI tools

Claude Web and Antigravity Desktop should work from the **same Git
repository**. A chat transcript is not the source of truth; committed
code and docs are.

-   Before coding, inspect `README.md`, `package.json`, the file tree,
    git status, recent commits, and relevant files.
-   Do not assume the repository is empty.
-   Do not overwrite user changes or reset/delete files without asking.
-   Work in small, reviewable changes.
-   Explain the intended change before a broad refactor.
-   Prefer a focused branch per feature if practical.
-   Do not create commits unless asked.
-   After changes, run the relevant lint, typecheck, tests, and build
    commands that actually exist.
-   Report commands run and their real results. Clearly state what was
    not run.
-   Never claim tests pass without executing them.
-   Update docs and `.env.example` when behavior/configuration changes.
-   Never commit `.env`, credentials, tokens, private keys, real user
    data, or generated secrets.
-   Keep `package-lock.json` (or the selected manager's lockfile)
    consistent with dependency changes.
-   If another agent has changed the same files, inspect and reconcile;
    never blindly overwrite.
-   When handing off between tools, summarize changed files, decisions,
    tests, blockers, and the next exact task.

### Recommended task handoff template

``` text
Task:
Goal and acceptance criteria:
Files inspected:
Files changed:
Design decisions:
Commands run and results:
Tests added/updated:
Known limitations or risks:
Open questions:
Suggested next task:
```

## 14. How to keep Claude and Antigravity synchronized

1.  Keep this document at repository root as `PROJECT_CONTEXT.md`.
2.  Add concise agent-specific instructions in `CLAUDE.md` and
    `AGENTS.md` (or the instruction file Antigravity currently
    supports). Those files should point back to `PROJECT_CONTEXT.md`,
    not duplicate a giant, drifting specification.
3.  Keep `README.md` focused on what the project is, how to run it, and
    current status.
4.  Record durable architectural decisions in `docs/decisions/` (for
    example, the crypto library choice).
5.  Update a `docs/STATUS.md` after meaningful sessions: current phase,
    completed work, failing checks, and next task.
6.  Use Git commits as shared checkpoints. Both tools must open the same
    local repo/remote and sync changes before editing.
7.  At the start of each session, ask the agent to read the project
    context and status, inspect the repo, and report what exists before
    proposing changes.
8.  If an AI suggestion conflicts with the actual code, the actual code
    plus documented decisions wins until a deliberate change is agreed.

## 15. First instruction to give either AI

``` text
You are helping me build CipherChat, a solo-developer open-source messaging project.
Read PROJECT_CONTEXT.md, README.md, docs/STATUS.md if present, and the relevant
repository instructions before making changes.

First inspect the actual repository, git status, package scripts, and existing tests.
Do not assume any feature is implemented just because it appears in the context.
Report:
1. what already exists,
2. what is incomplete or broken,
3. the smallest next implementation task,
4. acceptance criteria and tests for that task.

Do not start a broad rewrite. Do not overwrite existing work. Do not invent cryptography,
claim unimplemented security, add out-of-scope infrastructure, or say tests pass without
running them. Make one small, reviewable change at a time and report exact files changed
and commands/results.
```

## 16. Current status snapshot

As of 31 October 2026, this is the project plan and architecture context
carried over from planning. The current repository state has **not**
been inspected as part of this document generation.

-   [ ] Repository inspected and current status documented
-   [ ] Project scaffold confirmed
-   [ ] Stack and package manager confirmed
-   [ ] PostgreSQL/Docker development environment working
-   [ ] Initial Prisma schema/migrations
-   [ ] Authentication and sessions
-   [ ] One-to-one conversations and authorization
-   [ ] Real-time delivery and offline/reconnect behavior
-   [ ] Encryption protocol evaluated and selected
-   [ ] Protocol integration and identity verification
-   [ ] Automated tests and CI
-   [ ] Threat model, README, license, contributor/security docs
-   [ ] Android + desktop responsiveness verified

**Next action:** Open the actual repository in Antigravity, give it the
instruction in section 15, and have it inspect the current state before
writing code. Then use the resulting status report to update
`docs/STATUS.md` and this context if any decisions change.
