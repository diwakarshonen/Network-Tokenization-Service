## Overview

A **Java 21 / Spring Boot** reference architecture for integrating payment orchestration systems with a network tokenization provider. The design focuses on secure token provisioning, token lifecycle management, transaction-specific cryptogram generation, and controlled fallback to PAN-based processing.

> **Confidentiality note:** This README is intentionally anonymized. It does not contain company names, internal URLs, credentials, proprietary identifiers, customer data, PANs, BIN lists, production metrics, or implementation-specific secrets.

## Key Capabilities

- Secure card enrollment and token provisioning through an external network tokenization provider.
- Eligibility checks based on network support and configurable internal policy.
- Asynchronous provisioning and callback/webhook processing.
- Persistent storage of token references, instrument identifiers, token state, lifecycle metadata, and audit references.
- Fresh, single-use cryptogram generation for each payment authorization.
- Token lifecycle handling for events such as reissue, expiry, suspension, resumption, and replacement.
- Idempotent processing for repeated provisioning requests and duplicate webhook events.
- Policy-driven fallback to PAN rails when token processing is unavailable or not permitted.
- Operational monitoring for provisioning, cryptogram, authorization, fallback, and webhook outcomes.

## High-Level Flow

### Provisioning

1. An upstream payment system submits a secure enrollment request.
2. The service validates the request and correlation metadata.
3. Eligibility and policy checks are performed.
4. Eligible requests are submitted asynchronously to the network tokenization provider.
5. A callback/event confirms success or failure.
6. Successful token metadata is stored and linked to the payment instrument.
7. Failed provisioning remains non-tokenized and can be retried or handled according to policy.

### Transaction Usage

1. The payment orchestration layer requests payment credentials.
2. The service looks up the active network token for the payment instrument.
3. A new transaction-specific cryptogram is requested for the authorization.
4. Token + cryptogram + required metadata are returned downstream.
5. If token usage fails, PAN fallback is considered only when policy allows.
6. The usage outcome is recorded for operational analysis.

### Lifecycle Events

External lifecycle events are authenticated, validated, and deduplicated using a source event identifier. Token state is then updated and downstream systems can be notified when payment readiness is affected.
