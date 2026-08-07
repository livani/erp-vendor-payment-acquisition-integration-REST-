# System Architecture

## Purpose

This document describes the overall architecture of the ERP Vendor Payment Integration solution. It identifies the participating systems, their responsibilities, communication methods, and the end-to-end interaction during the Payment Acquisition process.

---

# Architecture Overview

The solution integrates the Vendor Portal, ERP System, and Finance Approval System through an Integration Layer.

The Integration Layer acts as the central middleware responsible for:

- Authenticating incoming requests
- Validating request payloads
- Retrieving Vendor Master Data
- Routing requests to the Finance Approval System
- Updating payment records in the ERP System
- Returning approved payment documents to the Vendor Portal

---

# High-Level Architecture

The following diagram illustrates the overall system architecture.

![High-Level Architecture](../assets/architecture.png)

**Figure 1. High-Level Architecture Diagram**

---

# System Components

| Component | Responsibility |
|-----------|----------------|
| Vendor Portal | Allows vendors to submit Payment Acquisition requests and download approved payment documents. |
| Integration Layer | Handles authentication, request validation, data transformation, routing, logging, and communication between systems. |
| ERP System | Stores Vendor Master Data and Payment Acquisition records. Serves as the system of record. |
| Finance Approval System | Reviews payment requests, performs approval workflow, generates digitally signed payment documents, and returns approval status. |

---

# Communication Matrix

| Source | Destination | Protocol | Purpose |
|---------|-------------|----------|---------|
| Vendor Portal | Integration Layer | HTTPS REST API | Submit Payment Acquisition Request |
| Integration Layer | ERP System | HTTPS REST API | Retrieve Vendor Master Data |
| Integration Layer | Finance Approval System | HTTPS REST API | Submit Payment Approval Request |
| Finance Approval System | Integration Layer | HTTPS REST API | Return Approval Status and Signed Document |
| Integration Layer | ERP System | HTTPS REST API | Update Payment Acquisition Record |
| Integration Layer | Vendor Portal | HTTPS REST API | Return Approved Payment Document |

---

# End-to-End Integration Flow

The following sequence diagram illustrates the interaction between all participating systems during a Payment Acquisition transaction.

![Sequence Diagram](../assets/sequence-diagram.png)

**Figure 2. End-to-End Integration Flow**

---

# Process Description

| Step | Activity |
|------|----------|
| 1 | Vendor submits a Payment Acquisition request through the Vendor Portal. |
| 2 | Integration Layer authenticates the request and validates the payload. |
| 3 | Integration Layer retrieves Vendor Master Data from the ERP System. |
| 4 | ERP System returns Vendor information. |
| 5 | Integration Layer submits the payment request to the Finance Approval System. |
| 6 | Finance Approval System reviews the request, approves it, and generates a digitally signed Payment Acquisition document. |
| 7 | Integration Layer updates the Payment Acquisition record in the ERP System with the approval status and document reference. |
| 8 | Integration Layer returns the approved document to the Vendor Portal for download. |

---

# Security Considerations

| Control | Description |
|----------|-------------|
| HTTPS | Encrypts all communication between systems. |
| JWT Authentication | Authenticates API requests before processing. |
| Payload Validation | Validates required fields and request format before forwarding requests. |
| Audit Logging | Records all requests, responses, and transaction activities. |
| Error Logging | Captures integration failures for monitoring and troubleshooting. |

---

# Assumptions

- The ERP System exposes REST APIs for Vendor Master Data and Payment Acquisition records.
- The Finance Approval System exposes REST APIs for approval processing.
- Vendors are authenticated before accessing the Vendor Portal.
- All system communication occurs over secure HTTPS connections.
- The Integration Layer is stateless and does not permanently store business data.
- All participating systems use UTF-8 encoded JSON payloads.
