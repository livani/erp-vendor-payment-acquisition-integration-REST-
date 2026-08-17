# Error Handling

## Purpose

This document defines the error-handling approach for the ERP Vendor Payment Integration process.

The integration handles errors across request validation, authentication, HTTP communication, business processing, approval document retrieval, and document validation.

The objective is to prevent invalid or incomplete transactions from progressing through the integration while providing sufficient technical information for troubleshooting and monitoring.

---

## Error Handling Flow

Errors are handled by the Integration Layer according to the stage at which the failure occurs.

```text
Vendor Portal
      |
      | HTTP Request
      ▼
Integration Layer
      |
      ├── Authentication
      |       |
      |       └── Error → Reject Request
      |
      ├── Request Validation
      |       |
      |       └── Error → Reject Request
      |
      ├── Finance Portal
      |       |
      |       └── HTTP / System Error
      |
      ├── Approval Processing
      |       |
      |       └── Business Rejection
      |
      ├── Document Retrieval
      |       |
      |       └── Retrieval Error
      |
      └── Document Validation
              |
              └── Validation Error
```

Each error is recorded in the Error Log for technical monitoring and troubleshooting.

---

# Error Handling Scenarios

| Error ID | Error Scenario | Error Type | HTTP Status | Handling | Result | Log |
|---|---|---|---|---|---|---|
| **ERR-001** | Missing mandatory request field | Validation | **400 Bad Request** | Reject request before forwarding to Finance Portal | Invalid request is not processed | Error recorded |
| **ERR-002** | Invalid invoice amount | Validation | **400 Bad Request** | Validate invoice amount and reject values less than or equal to zero | Request rejected | Error recorded |
| **ERR-003** | Invalid currency code | Validation | **400 Bad Request** | Validate currency against the required three-character format | Request rejected | Error recorded |
| **ERR-004** | Invalid or expired access token | Authentication | **401 Unauthorized** | Reject request and do not continue processing | Authentication failure returned | Error recorded |
| **ERR-005** | Access not permitted | Authorization | **403 Forbidden** | Reject request because the authenticated client does not have permission | Request rejected | Error recorded |
| **ERR-006** | Requested approval document not found | Retrieval | **404 Not Found** | Stop document processing and return retrieval failure | Document is not stored | Error recorded |
| **ERR-007** | Finance Portal system failure | System | **500 Internal Server Error** | Stop processing and record the failure | Payment request not completed | Error recorded |
| **ERR-008** | Finance Portal unavailable | Communication | **5xx / Connection Error** | Record communication failure and prevent successful completion | Transaction remains unsuccessful | Error recorded |
| **ERR-009** | Approval request rejected by Finance | Business | **200 OK** | Process the business rejection without retrieving an approval document | Payment remains rejected | Rejection recorded |
| **ERR-010** | Invalid approval document format | Document Validation | **200 OK** | Validate returned document and reject if it is not a PDF | Document is not stored | Error recorded |
| **ERR-011** | Approval document exceeds size limit | Document Validation | **200 OK** | Validate document size before storage | Document is not stored | Error recorded |
| **ERR-012** | Missing approval document ID | Response Validation | **200 OK** | Reject response processing because the required document identifier is missing | Document retrieval is not initiated | Error recorded |

---

# HTTP Error Handling

The Integration Layer uses HTTP status codes to distinguish different classes of technical failures.

| HTTP Status | Meaning | Integration Handling |
|---|---|---|
| `200 OK` | Request successfully processed at HTTP level | Process response and validate business result |
| `400 Bad Request` | Request contains invalid data | Reject request |
| `401 Unauthorized` | Authentication failed | Reject request |
| `403 Forbidden` | Client is not authorized | Reject request |
| `404 Not Found` | Requested resource does not exist | Stop processing and record error |
| `500 Internal Server Error` | Receiving system encountered an internal failure | Record error and prevent successful completion |
| `5xx / Connection Error` | Remote system or communication failure | Record communication failure |

HTTP success does not necessarily mean that the business transaction was successful.

For example:

```text
HTTP 200 OK
      |
      ▼
Business Response
      |
      ├── APPROVED
      |
      └── REJECTED
```

Similarly, a successful document retrieval at HTTP level can still fail application-level validation:

```text
HTTP 200 OK
      |
      ▼
PDF Retrieved
      |
      ├── Valid PDF
      |
      └── Invalid PDF / Size Exceeded
```

---

# Business Error Handling

Business errors are different from technical HTTP failures.

For example, the Finance Portal may successfully receive and process a Payment Acquisition request but determine that the payment should not be approved.

```text
Payment Request
      |
      ▼
Finance Portal
      |
      | HTTP 200 OK
      ▼
Business Result
      |
      | REJECTED
      ▼
Integration Layer
      |
      ├── Record rejection
      └── Do not retrieve approval document
```

The integration therefore treats the HTTP communication as successful while handling the business rejection separately.

---

# Error Logging

Errors are recorded in the independent `Error Log` table described in `05_Database.md`.

The Error Log contains:

| Column | Description |
|---|---|
| `log_id` | Unique error log identifier |
| `error_type` | Category of the integration error |
| `reason` | Description of the error |
| `error_log_time` | Date and time the error occurred |

### Example

| log_id | error_type | reason | error_log_time |
|---:|---|---|---|
| `500001` | `VALIDATION_ERROR` | Invoice ID is missing | `2026-08-09 10:16:02` |
| `500002` | `AUTHENTICATION_ERROR` | Access token is invalid or expired | `2026-08-09 10:16:15` |
| `500003` | `DOCUMENT_VALIDATION` | Returned document is not a valid PDF | `2026-08-09 10:17:14` |
| `500004` | `SYSTEM_ERROR` | Finance Portal returned HTTP 500 | `2026-08-09 10:18:31` |

The Error Log is intentionally independent from the business entities and Integration Log.

---

# Error Handling Principles

### 1. Validate Before Processing

The Integration Layer validates authentication and request data before forwarding the request to the Finance Portal.

### 2. Stop Invalid Transactions

Requests that fail mandatory validation or authentication are not forwarded for further processing.

### 3. Validate Responses

Successful HTTP responses are not automatically treated as successful business transactions.

The Integration Layer validates the returned business status and required response data.

### 4. Validate Documents Before Storage

Retrieved approval documents are validated for format and size before being stored as binary data.

### 5. Record Technical Failures

Errors are recorded in the Error Log to support troubleshooting and operational monitoring.

### 6. Separate Business and Technical Errors

HTTP communication errors and business approval decisions are handled separately.

For example:

- `500 Internal Server Error` → technical failure
- `200 OK + REJECTED` → business rejection

---

# Error Handling Summary

The overall error-handling approach is:

```text
HTTP Request
     |
     ▼
Authenticate
     |
     ├── Failure → 401/403 → Error Log
     |
     ▼
Validate Request
     |
     ├── Failure → 400 → Error Log
     |
     ▼
Finance Portal
     |
     ├── Technical Failure → 5xx → Error Log
     |
     ▼
Process Business Response
     |
     ├── REJECTED → Record Result
     |
     └── APPROVED
             |
             ▼
      Retrieve Document
             |
             ├── Failure → Error Log
             |
             ▼
      Validate Document
             |
             ├── Invalid → Error Log
             |
             ▼
       Store / Deliver Document
```
