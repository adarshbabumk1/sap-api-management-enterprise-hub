# 🛡️ SAP API Management: Policy Governance & Implementation Best Practices

**Role:** Tech Lead - SAP Integration  
**Focus:** Enterprise API Security, Rate Limiting & Decoupled Mediation  
**Target Platform:** SAP BTP API Management

---

## 1. Enterprise Policy Execution Pipeline

In SAP API Management, performance and security rely entirely on the ordering of policies attached to proxy Request and Response flows:

```
[ Inbound Consumer Request ]
       │
       ▼
 1. Spike Arrest (SpikeArrest.xml)
    • In-memory sliding window check.
    • Protects backend from Denial of Service (DoS) and traffic spikes.
       │
       ▼
 2. CORS Preflight & API Key Validation (Verify-API-Key.xml)
    • Validates consumer origin and application registration credentials.
       │
       ▼
 3. OAuth 2.0 / JWT Verification (OAuth-v20-Verify-Token.xml)
    • Verifies Bearer token digital signature against SAP BTP XSUAA / Cloud Identity.
    • Enforces required role scopes (e.g. s4.businesspartner.read).
       │
       ▼
 4. Quota Enforcement (Quota-Policy.xml)
    • Enforces commercial rate limits (e.g. 10,000 calls/month).
       │
       ▼
 5. Payload Sanitization & Threat Protection
    • JSON/XML Threat Protection validates nesting depth and regex patterns.
       │
       ▼
 6. Mediation & Header Enrichment (AssignMessage / JavaScript)
    • Strips internal gateway tokens, injects correlation ID (SAP-Passport).
       │
       ▼
[ Forward to Target Backend: https://<apim-host>/v1/s4 ]
```

---

## 2. Dynamic Target Connections (Zero Hardcoded Hosts)

Never hardcode physical hostnames in API Proxy target endpoints. Instead, configure **Target Servers** in the API Portal:

```xml
<TargetEndpoint name="default">
    <HTTPTargetConnection>
        <LoadBalancer>
            <Server name="S4HANA_PRODUCTION_BACKEND"/>
        </LoadBalancer>
        <Path>/sap/opu/odata/sap/API_BUSINESS_PARTNER</Path>
    </HTTPTargetConnection>
</TargetEndpoint>
```

* **Benefit:** When switching between DEV, QA, and PRD environments, only the Target Server alias configuration in the BTP environment changes — zero proxy redeployments required.

---

## 3. Standardized Fault Handling with FaultRules

Ensure internal stack traces or backend system details are never exposed to external API consumers. Use `<FaultRules>` to map gateway errors into clean JSON payloads:

```xml
<FaultRules>
    <FaultRule name="RateLimitViolation">
        <Condition>(fault.name Matches "SpikeArrestViolation") or (fault.name Matches "QuotaViolation")</Condition>
        <Step>
            <Name>Assign-RateLimit-Error-Payload</Name>
        </Step>
    </FaultRule>
</FaultRules>
```
