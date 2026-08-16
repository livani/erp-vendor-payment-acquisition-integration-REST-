## Purpose

This document describes the HTTP requests used in the ERP Vendor Payment Integration process.

The integration uses REST APIs over HTTPS to:

1. Submit the Payment Acquisition request to the Finance Portal.
2. Retrieve the approved payment document after the Finance approval process has been completed.

The request data is exchanged using JSON for the Payment Acquisition submission.

---

## Request Flow

The integration consists of two main HTTP request operations.

```text
Vendor Portal
     |
     | POST Payment Acquisition Request
     | HTTPS / REST API
     ▼
Integration Layer
     |
     | Validate / Process Request
     ▼
Finance Portal
     |
     | Payment Approval Process
     |
     ▼
Approval Document Generated
     |
     | GET Document Request
     ▼
Integration Layer
     |
     ▼
Vendor Portal
```

The `POST` request initiates the Payment Acquisition process.

After the Finance Portal completes the approval process and generates the approval document, a `GET` request is used to retrieve the resulting document.

---

## 1. Payment Acquisition Request

### HTTP Method

| Property      | Value              |
| ------------- | ------------------ |
| HTTP Method   | `POST`             |
| Protocol      | `HTTPS`            |
| Content Type  | `application/json` |
| Source System | Vendor Portal      |
| Target System | Finance Portal     |

The `POST` request is used to submit a new Payment Acquisition request for Finance processing.

---

## Endpoint

The production endpoint is environment-dependent.

For documentation purposes, the endpoint is represented as:

```http
POST /api/payment-acquisition
```

Example:

```text
https://<finance-portal-host>/api/payment-acquisition
```

Production hostnames and environment-specific configuration are intentionally omitted.

---

## HTTP Headers

The request contains the following HTTP headers:

| Header          | Purpose                              |
| --------------- | ------------------------------------ |
| `Authorization` | Access token / bearer token          |
| `Content-Type`  | Defines the request body format      |
| `Accept`        | Defines the expected response format |

Example:

```http
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
```

Actual access tokens and authentication credentials must not be stored in the repository.

---

## Request Body

The Payment Acquisition request contains the vendor, contract, and invoice information required by the Finance Portal to process the payment approval.

The request uses a flat JSON structure because the API receives the required integration fields as a single payment transaction payload.

### Example JSON Request

```json
{
  "vendor_id": "V000123",
  "vendor_name": "ABC Engineering Ltd.",
  "tax_number": "TX123456789",
  "contract_number": "CON-2026-001",
  "work_description": "Maintenance and technical support services",
  "invoice_id": "INV-2026-00045",
  "invoice_amount": 50000000.00,
  "invoice_date": "2026-08-09",
  "currency": "IDR"
}
```

The request does not contain the approval document because the document is generated after the Finance approval process.

---

## Request Data Mapping

| Request Field      | Source        | Data Type | Required | Validation                   |
| ------------------ | ------------- | --------- | -------- | ---------------------------- |
| `vendor_id`        | Vendor Portal | String    | Yes      | Vendor must exist            |
| `vendor_name`      | Vendor Portal | String    | Yes      | Must not be empty            |
| `tax_number`       | Vendor Portal | String    | Yes      | Valid vendor tax number      |
| `contract_number`  | Vendor Portal | String    | Yes      | Contract must exist          |
| `work_description` | Vendor Portal | String    | Yes      | Must not be empty            |
| `invoice_id`       | Vendor Portal | String    | Yes      | Invoice must exist           |
| `invoice_amount`   | Vendor Portal | Decimal   | Yes      | Must be greater than 0       |
| `invoice_date`     | Vendor Portal | Date      | Yes      | Valid date                   |
| `currency`         | Vendor Portal | String    | Yes      | Valid ISO 4217 currency code |

The fields correspond to the data defined in `04_Data_Mapping.md`.

---

## Request Validation

Before the request is processed by the Finance Portal, the Integration Layer validates the incoming request.

### Mandatory Field Validation

The following fields must be present:

* Vendor ID
* Vendor Name
* Tax Number
* Contract Number
* Work Description
* Invoice ID
* Invoice Amount
* Invoice Date
* Currency

### Business Validation

The Integration Layer validates that:

* Vendor ID is present and valid.
* Contract Number exists and is associated with the vendor.
* Invoice ID is present and valid.
* Invoice Amount is greater than zero.
* Invoice Date follows the required date format.
* Currency uses a valid three-character ISO 4217 code.
* The request conforms to the expected JSON structure.

If validation fails, the request is rejected and the error is recorded according to the error-handling process documented in `09_Error_Handling.md`.

---

## Example HTTP Request

```http
POST /api/payment-acquisition HTTP/1.1
Host: <finance-portal-host>
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json

{
  "vendor_id": "V000123",
  "vendor_name": "ABC Engineering Ltd.",
  "tax_number": "TX123456789",
  "contract_number": "CON-2026-001",
  "work_description": "Maintenance and technical support services",
  "invoice_id": "INV-2026-00045",
  "invoice_amount": 50000000.00,
  "invoice_date": "2026-08-09",
  "currency": "IDR"
}
```

---

## 2. Approval Document Retrieval Request

After the Finance Portal completes the payment approval process, the approved payment document can be retrieved through a separate HTTP `GET` request.

### HTTP Method

| Property        | Value                                  |
| --------------- | -------------------------------------- |
| HTTP Method     | `GET`                                  |
| Protocol        | `HTTPS`                                |
| Response Format | Document / binary data                 |
| Purpose         | Retrieve the approved payment document |

The request uses query parameters to identify the approval document and vendor.

---

## Endpoint

The document retrieval endpoint follows the structure:

```http
GET /invoice?approval_document_id={approval_document_id}&vendor_id={vendor_id}
```

Example:

```http
GET /invoice?approval_document_id=1&vendor_id=1
```

The actual production host and endpoint configuration are intentionally omitted.

---

## Query Parameters

| Parameter              | Example | Required | Description                                        |
| ---------------------- | ------- | -------- | -------------------------------------------------- |
| `approval_document_id` | `1`     | Yes      | Identifies the approved payment document           |
| `vendor_id`            | `1`     | Yes      | Identifies the vendor associated with the document |

The `approval_document_id` is used during the document retrieval stage and is not part of the initial Payment Acquisition `POST` request.

---

## Example GET Request

```http
GET /invoice?approval_document_id=1&vendor_id=1 HTTP/1.1
Host: <finance-portal-host>
Authorization: Bearer <access_token>
Accept: application/pdf
```

The returned document is subsequently processed and stored according to the database design documented in `05_Database.md`.

---

