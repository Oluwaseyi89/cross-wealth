## CrossWealth - Implementable Issues (categorized)

This file contains realistic, contributor-ready issues grouped by app / service. Each issue includes a concise description and short acceptance criteria so contributors can start implementing.

**Frontend (Next.js web app)**
- Wallet Integration: Stellar Account Connect & Transaction Signing
   - Implement `WalletConnect`/Freighter/Albedo integration, secure key handling in-browser, transaction builder, and UX for signing transfers and contract calls.
   - Acceptance criteria: working connect/disconnect flows; successfully build, sign, and submit a Stellar payment and Soroban transaction on testnet; error states surfaced to user.

- Soroban Client: Contract Interaction Library
   - Build `soroban-client.ts` helpers to call Soroban contracts (credit-check, loan issuance, savings vault) with retries, gas-estimates, and typed responses.
   - Acceptance criteria: helper functions for read/write contract calls; sample page invoking credit-scoring contract; unit tests mocking contract responses.

- Save Flow: Round-up & Scheduled Savings UI
   - Implement UI and background scheduling for rounding-up transactions and creating scheduled deposits into a savings vault on Stellar.
   - Acceptance criteria: user can enable round-ups, preview saved amounts, schedule recurring deposits, and see on-chain pending/confirmed deposits in UI.

- Borrow Flow: Credit Application UI + Score Request
   - Implement loan application UX that requests a privacy-preserved credit score from the credit oracle, shows estimated terms, and submits loan requests.
   - Acceptance criteria: flow displays score fetch progress, shows loan terms based on returned score, and builds a Soroban loan request transaction.

- Send Flow: Corridor Selection & Estimate Engine
   - Implement corridor-based remittance UX that calls the bridge `exchange` API to show fees, FX, delivery methods and estimated arrival times.
   - Acceptance criteria: corridor dropdown, real-time rate fetch, fee breakdown, and ability to initiate a send that triggers bridge workflows.

- KYC & Non-Custodial Verification (UI + Webhook Handling)
   - Add UI for starting non-custodial KYC, uploading documents locally, and registering verification callbacks; implement a local webhook handler in `src/app/api/kyc`.
   - Acceptance criteria: KYC flow triggers partner callback, UI shows verification states, and no sensitive docs are uploaded to our servers unless required by partner.

- Offline / PWA & Low-Bandwidth Optimizations
   - Make the app a PWA with caching strategies for critical pages, offline-safe state, and small bundle sizes for feature-phone users.
   - Acceptance criteria: installable PWA, cached dashboard view, and page load metrics under 2s on 3G throttling in Lighthouse.

- Internationalization, Accessibility & Usability for Low-Literacy Users
   - Expand `i18n.ts`, add voice-friendly copy and large-icon layouts, ensure WCAG AA accessibility, and provide SMS/USSD fallback hooks.
   - Acceptance criteria: translations present for `en`, `es`, `fr`, `sw`; accessibility audit passes AA; USSD/SMS call-to-action implemented.

- End-to-End Tests & CI for Frontend
   - Add Playwright/Cypress flows for key user journeys (save, send, borrow, KYC) and integrate tests into CI.
   - Acceptance criteria: CI runs E2E tests on pull requests and reports failures; minimal stable test set covering 80% of flows.

**Soroban Contracts (stellar-layer/soroban-contracts)**
- Contract: Credit-Scoring Oracle Adapter
   - Create a Soroban contract that receives oracle-signed, privacy-preserved credit scores and maps them to reputation tokens or flags used by lending logic.
   - Acceptance criteria: contract verifies oracle signatures, stores score metadata per account, and exposes read-only APIs for frontends and other contracts.

- Contract: Multi-Sig Savings Vault
   - Implement a savings vault supporting scheduled deposits, multi-signature withdrawals, emergency pause, and deposit forwarding to asset issuer.
   - Acceptance criteria: deposit/withdraw tests, scheduled deposit scheduler (simulated), and multi-sig approval flow tested in contract tests.

- Contract: Micro-Loan Issuance & Repayment
   - Build loan lifecycle contract: application, approval (by protocol or partner), disbursement, repayment schedule, interest accrual, and default handling.
   - Acceptance criteria: unit tests covering loan issuance, partial/full repayment, delinquency flagging, and on-chain events emitted for off-chain jobs.

- Contract: Asset Issuer & Reputation Token
   - Implement stablecoin issuance controls and a non-transferable reputation/credit token representing on-chain credit standings.
   - Acceptance criteria: mint/burn ACLs, trustline onboarding, reputation minting logic, and integration tests with sample wallets.

- Security & Gas Optimizations
   - Audit and optimize contract storage and call patterns, add property-based tests, and prepare for an external security review.
   - Acceptance criteria: documented gas costs for operations, test coverage > 80% for core logic, and a checklist for external audit readiness.

**Credit Oracle (off-chain-services/credit-oracle, Go)**
- gRPC API: Score Request/Response + Auth
   - Implement a gRPC server that accepts signed requests from Soroban contracts (via relayers) or partner gateways, returning signed, privacy-preserved scores.
   - Acceptance criteria: secure mutual TLS or token-based auth, proto definitions, and example client fetch that verifies oracle signatures.

- Federated Learning Orchestration & Device SDK
   - Build server-side orchestration for federated learning rounds, plus a lightweight client library that runs on-device or in partner apps to contribute model updates.
   - Acceptance criteria: a retrain simulation run, model aggregation pipeline, and docs for integrating the client SDK.

- Model Training Pipeline & Retrain Jobs
   - Implement `model_training.go` and a retrain HTTP job that pulls aggregated data, trains a model, validates fairness, and publishes artifacts to `ml_models`.
   - Acceptance criteria: reproducible training job, checkpointing, evaluation metrics, and an HTTP-triggered retrain endpoint with access controls.

- Fairness & Bias Detection Module
   - Implement `fairness.go` to compute demographic parity, equalized odds, and other metrics with alerts for drift or biases.
   - Acceptance criteria: automated fairness report generation, alert hooks, and unit tests for metric calculations.

- Data Ingestion & Anonymizer Connectors
   - Build connectors to mobile-money/bank APIs and an `anonymizer.go` that tokenizes PII, computes aggregates, and writes to the anonymized data lake.
   - Acceptance criteria: connector interface, mock connector tests, and anonymized records written to the data lake format.

- Oracle Security & Rate Limiting
   - Add request signing, replay protection, ACLs, and per-client rate limiting to protect scoring endpoints.
   - Acceptance criteria: authentication enforced, rate limit behavior tested, and replay attack mitigation implemented.

**Bridge Service (off-chain-services/bridge-service, Python)**
- Mobile Money Adapters with Idempotency
   - Implement reliable adapters for M-Pesa, Orange Money, Wave including idempotency keys, retry policies, and reconciliation endpoints.
   - Acceptance criteria: adapter interface, simulated integration tests, and reconciliation job that matches bridge ledger with provider callbacks.

- Exchange & Corridor Pricing Engine
   - Build an `exchange` service that aggregates liquidity providers, computes corridor-specific rates and fees, and provides quoted estimates for the frontend.
   - Acceptance criteria: rate caching, failover between providers, and API endpoints returning corridor quotes with TTL.

- Non-Custodial KYC Orchestration
   - Orchestrate partner-driven KYC flows where PII remains with the user or partner, while our system receives only verification tokens and statuses.
   - Acceptance criteria: KYC state machine, webhook endpoints for partner callbacks, and front-end integration examples.

- AML Screening & Monitoring
   - Add AML screening logic for inbound/outbound remittances with threshold alerts and a case-management export for compliance teams.
   - Acceptance criteria: screening rules engine, flagged-case export, and scheduled monitoring jobs.

- Notifications: Multilingual SMS + Delivery Guarantees
   - Implement `multilingual_sms.py` to send localized notifications, with fallback and retry for delivery failures.
   - Acceptance criteria: templating for `en/es/fr/sw`, delivery status tracking, and resend logic for failed sends.

**Partner Gateway (partner-gateway, NestJS)**
- Analytics Module & Regulatory Reporting
   - Implement the `analytics` module that ingests anonymized events and produces partner-facing dashboards and PDF compliance reports.
   - Acceptance criteria: endpoints for report generation, scheduled exports, and PDF output matching regulatory templates.

- Agent Portal & Transaction Management APIs
   - Build agent-facing APIs for onboarding customers, depositing/withdrawing on behalf of users, and operator dashboards for reconciliation.
   - Acceptance criteria: secure agent auth, transaction lifecycle endpoints, and audit logs for agent actions.

- Remittance API & Partner Onboarding Flow
   - Implement partner onboarding workflows, credentials provisioning, and idempotent remittance APIs with callback webhooks.
   - Acceptance criteria: partner API keys, sandbox mode, and robust webhook retry/backoff.

- Oracle Client & Caching Layer
   - Add `oracle.client.ts` to fetch raw (anonymized) credit data, with caching and fallback for degraded oracle availability.
   - Acceptance criteria: cached responses, TTL eviction, and graceful degradation behavior in the UI.

- RBAC & Audit Logging for Partner Operations
   - Implement role-based access control, organization scoping, and tamper-evident audit logs for partner operations.
   - Acceptance criteria: roles (admin/analyst/agent), policy enforcement, and searchable audit logs.

**Data & ML Infrastructure**
- Model Registry, CI & Artifact Storage
   - Create CI pipelines to validate models, run fairness tests, and push artifacts to a registry with versioning and provenance.
   - Acceptance criteria: automated tests for new models, signed model artifacts, and reproducible deployment steps.

- Anonymized Data Lake Schema & ETL
   - Define canonical schemas for `credit-events`, `savings-patterns`, and `corridor-analytics` and implement ETL jobs to populate them.
   - Acceptance criteria: schema docs, ETL jobs with retries, and sample dashboards using the aggregated tables.

- Reporting: SDG Impact Metrics & Partner Dashboards
   - Implement pipelines and dashboards that compute impact metrics aligned with the UN SDGs for partners and reporting.
   - Acceptance criteria: SDG metrics computed monthly, CSV/JSON exports, and partner dashboard endpoints.

**Stellar Assets & Horizon/Webhooks**
- Asset Definition & Trustline Onboarding Scripts
   - Produce `corridor-usdc.xml`, `local-stable.xml` and onboarding scripts that create issuers, set policies, and guide partners to trustlines.
   - Acceptance criteria: scripts that run on testnet/mainnet (dry-run), documentation for partners, and trustline UI hooks.

- Horizon Webhook Reliability & Event Delivery
   - Harden `webhook-config.yaml` and the delivery pipeline to guarantee event ordering, idempotency, and retries to off-chain consumers.
   - Acceptance criteria: deduplication key, ordered delivery tests, and dead-letter queue for failed deliveries.

**Integration & Partnerships**
- Sandbox Configs & Regulatory Templates
   - Produce per-country sandbox configs and regulatory JSON schemas to speed partner integration and compliance validation.
   - Acceptance criteria: sandbox manifests for top corridors, example compliance documents, and onboarding checklist.

- Agent Network Onboarding Toolkit
   - Create agent onboarding tooling (QR-code kits, docs, small web portal) to register agents, print QR codes, and manage cash-in/out limits.
   - Acceptance criteria: agent registration flow, printable QR pack generation, and transaction limits enforced in partner APIs.

---

If you'd like, I can now:
- open each issue as GitHub issues with labels and assignees, or
- expand any selected issue into a task breakdown with milestones and estimated effort.

