# AEP-NNNN: <Extension interface name>

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Stability** | Experimental |
| **Authors** | <handles> |
| **Extension class** | Planner \| Runtime \| Compiler backend \| Package format \| Channel \| Policy provider \| Knowledge provider \| Secret provider \| Controller SDK |
| **Created** | YYYY-MM-DD |
| **Interface version** | v0.1.0 |
| **Conformance suite** | <link, once it exists> |

> An AEP is a promise to third-party authors. It carries the standard 15-section design shape (see the RFC template) **plus** the extension-specific sections below. Fill both.

## Standard design sections
*(Sections 1–15 exactly as in the RFC template: Executive Summary … Future Improvements.)*

## E1. Interface definition
*The exact contract an implementer codes against: types, methods, wire format, serialization, and the complete error model. This is normative and precise.*

## E2. Lifecycle & negotiation
*How the host discovers, loads, initializes, health-checks, and shuts down the extension. How host and extension negotiate a compatible interface version.*

## E3. Capabilities & isolation
*Exactly which capabilities the extension is granted and denied (ties to Book 11). Its trust boundary, resource limits, and failure containment.*

## E4. Data contract
*What the extension receives and returns in terms of ARM Resources / AOS IR / Events. Show concrete examples.*

## E5. Conformance tests
*The suite an implementation MUST pass to claim conformance at this stability level. What each test asserts.*

## E6. Stability & compatibility
*Current stability level and the criteria to promote to the next (Experimental → Beta → Stable). Deprecation and removal policy for this interface.*

## E7. Reference implementation
*Pointer to the canonical implementation (Roadmap phase), and at least one worked example extension.*

## E8. Ecosystem impact
*Effect on the Package format, the Marketplace, and existing extensions.*
