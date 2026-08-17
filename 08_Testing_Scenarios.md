# Test Scenarios

## Purpose

This document defines the test scenarios used to validate the ERP Vendor Payment Integration process.

The scenarios cover request validation, authentication, payment approval, approval document retrieval, document validation, integration failures, logging, and the complete end-to-end integration flow.

---

## Test Data

The following sample data is used as the baseline for the integration test scenarios.

| Field | Test Value |
|---|---|
| Vendor ID | `V000123` |
| Vendor Name | `ABC Engineering Ltd.` |
| Tax Number | `TX123456789` |
| Contract Number | `CON-2026-001` |
| Work Description | `Maintenance and technical support services` |
| Invoice ID | `INV-2026-00045` |
| Invoice Amount | `50000000.00` |
| Invoice Date | `2026-08-09` |
| Currency | `IDR` |

---

# Testing Scenarios

The following test matrix covers the primary functional, technical, and end-to-end scenarios for the integration.

| Test ID | Scenario | Type | Test Data | Status | Result | Log |
|---|---|---|---|---|---|---|
| **TS-001** | Successful Payment Acquisition Request | Positive | `vendor_id=V000123`; `contract_number=CON-2026-001`; `invoice_id=INV-2026-00045`; `invoice_amount=50000000.00`; `currency=IDR` | **200 OK** | Payment request accepted and forwarded for Finance processing | `SUCCESS` |
| **TS-002** | Missing Mandatory Field | Negative | `invoice_id` omitted from request body | **400 Bad Request** | Request rejected because a mandatory field is missing | Validation error |
| **TS-003** | Invalid Invoice Amount | Negative | `invoice_amount=0` | **400 Bad Request** | Request rejected because invoice amount must be greater than zero | Validation error |
| **TS-004** | Invalid Currency | Negative | `currency=XXXX` | **400 Bad Request** | Request rejected because currency code is invalid | Validation error |
| **TS-005** | Invalid Authentication Token | Negative | Valid request body with invalid or expired `Authorization` token | **401 Unauthorized** | Request rejected because authentication failed | Authentication error |
| **TS-006** | Finance Portal Rejects Payment | Business Negative | Valid payment request with complete vendor, contract, and invoice data | **200 OK** | Finance Portal processes the request but returns `REJECTED`; no approval document is retrieved | Rejection recorded |
| **TS-007** | Successful Approval Document Retrieval | Positive | `GET /invoice?approval_document_id=1&vendor_id=1` | **200 OK** | Signed PDF successfully retrieved and processed | `SUCCESS` |
| **TS-008** | Approval Document Not Found | Negative | `GET /invoice?approval_document_id=999&vendor_id=1` | **404 Not Found** | Requested approval document cannot be retrieved | Retrieval error |
| **TS-009** | Invalid Approval Document Format | Negative | Valid GET request returns a non-PDF document | **200 OK** | HTTP request succeeds, but document fails format validation and is not stored | Document validation error |
| **TS-010** | Approval Document Exceeds Size Limit | Negative | Valid GET request returns PDF larger than `65,535 bytes` | **200 OK** | HTTP request succeeds, but document fails size validation and is not stored | Document size error |
| **TS-011** | Finance Portal System Error | Technical Negative | Valid Payment Acquisition request | **500 Internal Server Error** | Finance Portal fails to process the request | System error |
| **TS-012** | Integration Logging | Technical | Successful or failed HTTP integration operation | **200 OK / 4xx / 5xx** | Integration execution is recorded with operation, source, target, status, date, and time | Log created |
| **TS-013** | Complete End-to-End Integration | End-to-End | Valid POST request → approved payment → `GET /invoice?approval_document_id=1&vendor_id=1` | **200 OK** | Payment request approved, signed PDF retrieved, validated, and made available through Vendor Portal | Successful integration logs |

---


## Test Coverage

The test scenarios cover:

- HTTP request validation
- Authentication
- Business validation
- Payment approval
- Approval rejection
- Approval document retrieval
- PDF validation
- Document size validation
- HTTP and system errors
- Integration logging
- End-to-end processing

Detailed error classifications and handling procedures are documented in `09_Error_Handling.md`.
