# Product Requirements Document (PRD): CCP Ingestion & AI Delivery

## 1. Problem Statement & User Personas
Internal engineering groups face a bottleneck when attempting to map and publish localized user actions into the core customer database. Concurrently, AI teams lack a predictable, real-time, privacy-compliant stream of structured customer attributes to ground their Large Language Models, risking hallucinations and compliance violations.

* **Data Producers (Internal Developers):** Need to safely expose their data to the enterprise with minimal administrative overhead.
* **Data Consumers (AI & Analytics Teams):** Require absolute data freshness, structural reliability, and standardized API keys.
* **Trust & Compliance Teams:** Mandate zero accidental leaks of Personally Identifiable Information (PII).

## 2. Core Functional Requirements & User Stories

### Feature Group A: Schema Enforcement & Self-Service Ingestion
* **User Story:** As an internal developer, I want to upload a declarative contract file so that my tracking pipeline is spun up automatically without scheduling cross-functional engineering reviews.
* **Acceptance Criteria:**
  * Ingestion gateway must instantly reject payloads missing mandatory global keys (`account_id`, `event_timestamp`).
  * System must automatically spin up isolated topic buffers upon successful schema validation.

### Feature Group B: Multi-Consumer SLA Partitioning
* **Requirement:** The platform must split its delivery architecture to prevent long-running analytical queries from degrading real-time AI context delivery.
  * **Real-Time API Layer:** Must serve under a 100ms P99 latency threshold to support interactive AI agents.
  * **Analytical Warehouse Layer:** Must maintain consistency with real-time states within a maximum 15-minute reconciliation boundary.

### Feature Group C: Programmatic Privacy Guardrails
* **User Story:** As a corporate risk officer, I want user data to automatically mask its own sensitive attributes based on customer consent state, so we prevent regulatory fines at scale.
* **Acceptance Criteria:**
  * When `legal_consent_status == FALSE`, fields classified as PII must be programmatically hashed (SHA256) before reaching downstream AI or analytical environments.

## 3. Success Metrics & Key Performance Indicators (KPIs)
* **Velocity:** Time-to-onboard new system data pipelines reduced from weeks to < 1 hour.
* **Adoption Rate:** Percentage of target internal systems migrated to the new gateway (Target: 85% by end of Q2).
* **Reliability:** Data parity between real-time caching layers and data warehouse snapshots maintained at a 99.99% match rate.

## 🌐 4. Multi-Persona UI & API Egress Framework

The Core Customer Profile Platform (CCP) exposes its unified data through tailored internal user interfaces and API endpoints, matching the exact operational needs of our three core company personas.

### Persona 1: The AI Engineer / Product Developer UI
* **User Goal:** Needs instant, low-latency access to token-optimized customer context to ground live LLM chat agents.
* **UI Delivery Format:** Sub-50ms Real-Time REST/gRPC API Endpoint (`GET /v2/profiles/{account_id}/ai-context`)

```json
// UI Response Payload optimized for downstream RAG Injection
{
  "status": "SUCCESS",
  "latency_ms": 12,
  "token_count": 142,
  "payload": {
    "account_id": "usr_ccp_99812",
    "customer_tier": "Enterprise_Premium",
    "masked_email": "e3b0c44298fc1c149afbf4c8996fb92427ae...",
    "live_context": {
      "session_mins": 42,
      "active_error": "ERR_LICENSE_EXPIRED_403",
      "intent_prediction": "BILLING_TROUBLESHOOTING"
    }
  }
}
```

---

### Persona 2: The Data Analyst / Fraud Operations UI
* **User Goal:** Needs to query millions of historical rows to spot massive fraud trends, calculate churn metrics, and build company dashboards.
* **UI Delivery Format:** Columnar Data Warehouse View (`ccp_analytics.unified_profiles_fact`) in Snowflake/DuckDB accessed via a BI Tool (e.g., Tableau or Looker).

| ACCOUNT_ID | CUSTOMER_TIER | COMPLIANCE_STATUS | REGION_CODE | RECORD_CREATION_TIME | SIGN_UP_SOURCE |
| :--- | :--- | :--- | :--- | :--- | :--- |
| usr_ccp_99812 | Enterprise_Premium | MASKED_PII | AMER | 2026-10-03 13:00:00 | Web_Portal_Direct |
| usr_ccp_99813 | Free_Tier | FULL_ACCESS | EMEA | 2026-10-03 13:05:22 | Mobile_App_OAuth |
| usr_ccp_99814 | Professional_Suite | MASKED_PII | APAC | 2026-10-03 13:12:05 | Third_Party_Partner |

---

### Persona 3: The Data Provider / Internal Platform Engineer UI
* **User Goal:** Needs a self-service console to register their system's data stream into the platform in minutes, upload their Data Contract, and monitor pipeline health.
* **UI Delivery Format:** Text-Based Internal Developer Portal Mockup

```text
================================================================================
[ CCP DEVELOPER SELF-SERVICE PORTAL ]                   (User Role: Ingestion_Admin)
================================================================================

[+] REGISTER NEW DATA PRODUCER STREAM
--------------------------------------------------------------------------------
1. Upload Schema Specification:     [ Choose File: contracts/user_profile_v2.yaml ]
2. System Ingestion Gateway Scan:   [ RUNNING AUTOMATED COMPLIANCE AUDIT... ]
                                    🟢 SUCCESS: Data Contract Validated.

3. Select Target Downstream Channels:
   [X] Real-Time AI Cache Layer (SLA: <50ms P99 latency)
   [X] Analytical Data Warehouse Layer (SLA: <15min Sync Wall)

[ CANCEL REGISTRATION ]                                [ LAUNCH AUTOMATED PIPELINE ]

--------------------------------------------------------------------------------
LIVE PLATFORM PIPELINE HEALTH MONITOR:
--------------------------------------------------------------------------------
-> Ingestion_Stream_A  | Status: 🟢 HEALTHY  | Latency: 14ms  | Data Parity: 100%
-> Ingestion_Stream_B  | Status: 🟡 WARNING  | Latency: 48ms  | Data Parity: 99.98%
================================================================================
```
