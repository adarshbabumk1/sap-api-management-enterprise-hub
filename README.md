# 🛡️ SAP API Management: Enterprise Security & Governance Hub

[![SAP API Management](https://img.shields.io/badge/SAP-API_Management-009688?style=for-the-badge&logo=sap&logoColor=white)](https://community.sap.com/topics/api-management)
[![OpenAPI 3.0](https://img.shields.io/badge/OpenAPI-3.0.3-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)](https://swagger.io/specification/)
[![Security Hardened](https://img.shields.io/badge/Security-OAuth2%20%7C%20mTLS%20%7C%20JWT-red.svg?style=for-the-badge)](policies/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)

An architectural reference repository providing production-ready **Policy Bundles**, **OpenAPI 3.0 specifications**, and **API Governance patterns** for **SAP API Management** on **SAP BTP Integration Suite**.

This project demonstrates how to protect, decouple, monitor, and monetize SAP backend systems (S/4HANA, ECC, Cloud Integration, SAP Ariba) using enterprise gateway policies.

---

## 📑 Table of Contents

1. [Architectural Overview](#-architectural-overview)
2. [API-Led Connectivity Architecture](#-api-led-connectivity-architecture)
3. [Production Policy Repository](#-production-policy-repository)
4. [Governed OpenAPI Specifications](#-governed-openapi-specifications)
5. [Security & Traffic Flow Lifecycle](#-security--traffic-flow-lifecycle)

---

## 🏛️ Architectural Overview

SAP API Management acts as the **front door** to enterprise digital assets. It isolates core SAP systems from direct internet exposure, mitigating security vulnerabilities, managing sudden traffic spikes, and offering a unified Developer Portal.

```mermaid
sequenceDiagram
    autonumber
    actor Client as Client App / Mobile / Third-Party
    participant APIGW as SAP API Management Gateway
    participant IDP as Corporate Identity Provider (XSUAA / IAS)
    participant CPI as SAP Cloud Integration (CPI)
    participant S4 as SAP S/4HANA Backend

    Client->>APIGW: 1. Request with Bearer JWT / API Key
    Note over APIGW: SpikeArrest & Quota Check
    APIGW->>IDP: 2. Verify OAuth 2.0 Token & Scopes
    IDP-->>APIGW: 3. Token Validated (Claims Returned)
    Note over APIGW: Extract JWT Claims & Enrich Headers
    APIGW->>CPI: 4. Governed Request (ProcessDirect / HTTPS)
    CPI->>S4: 5. OData / RFC Query via Cloud Connector
    S4-->>CPI: 6. Business Data Response
    CPI-->>APIGW: 7. Internal Payload
    Note over APIGW: JSON Threat Sanitization & CORS Headers
    APIGW-->>Client: 8. Sanitized API Response (200 OK)
```

---

## 🌐 API-Led Connectivity Architecture

We structure enterprise APIs into three decoupled tiers:

```
┌────────────────────────────────────────────────────────┐
│ 1. Experience APIs (Mobile, Partner Portal, B2B Web)   │
│    Governed by SAP APIM (Rate Limits, CORS, Monetize)  │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ 2. Process APIs (Orchestration & Business Logic)       │
│    Orchestrated by SAP Cloud Integration (CPI iFlows)  │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ 3. System APIs (Core Records: S/4HANA, Salesforce)     │
│    Protected by Cloud Connector & Principal Propagation│
└────────────────────────────────────────────────────────┘
```

---

## 📦 Production Policy Repository

Explore production-grade policies in [`policies/`](policies/):

### 1. Traffic Management
* **[`Spike-Arrest.xml`](policies/traffic/Spike-Arrest.xml)**: Prevents traffic spikes and Distributed Denial of Service (DDoS) by smoothing requests using the leaky bucket algorithm (e.g. `100pm` or `20ps`).
* **[`Quota-Policy.xml`](policies/traffic/Quota-Policy.xml)**: Enforces business tier monetization limits (e.g., Free Tier: 1,000 calls/month; Gold Tier: 100,000 calls/month).

### 2. Security & Authentication
* **[`Verify-API-Key.xml`](policies/security/Verify-API-Key.xml)**: Validates client API Key passed via header `APIKey` or query parameter.
* **[`OAuth-v20-Verify-Token.xml`](policies/security/OAuth-v20-Verify-Token.xml)**: Validates OAuth 2.0 access tokens against SAP BTP XSUAA or SAP Cloud Identity Services.
* **[`CORS-Preflight.xml`](policies/security/CORS-Preflight.xml)**: Handles browser `OPTIONS` preflight requests for Single Page Apps (Angular, React, SAP Fiori).

### 3. Mediation & Transformation
* **[`Extract-JWT-Claims.xml`](policies/mediation/Extract-JWT-Claims.xml)**: Extracts user email, tenant ID, and custom roles from JSON Web Tokens without backend overhead.
* **[`JSON-to-XML.xml`](policies/mediation/JSON-to-XML.xml)**: Converts modern REST payloads to legacy SOAP/XML for backend compatibility.

---

## 📜 Governed OpenAPI Specifications

* **[`s4hana-business-partner-v1.yaml`](openapi/s4hana-business-partner-v1.yaml)**: Complete OpenAPI 3.0.3 specification exposing S/4HANA Business Partner entities with strict response schemas, error codes, and security definitions.

---

## 🔒 Security Best Practices

1. **Never expose raw internal URLs**: All backend S/4HANA endpoints must be masked behind APIM target endpoints.
2. **Apply Spike Arrest before Quota**: Reject abusive burst traffic before performing expensive database or cache lookups.
3. **Principal Propagation**: Use SAP Cloud Connector and JWT identity propagation to ensure the end-user's enterprise authorization is honored in ABAP.
