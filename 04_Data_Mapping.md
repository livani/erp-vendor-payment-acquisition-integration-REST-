# Data Mapping

## Purpose

This document defines the mapping of business data exchanged between the Vendor Portal, Finance Approval Portal, and the underlying database during the Payment Acquisition process.

The Vendor Portal is the source of the payment-related business information, including vendor, contract, work, and invoice details. The Finance Approval Portal receives the validated information for approval processing and generates the digitally signed payment approval document.

The Integration Layer is responsible for transforming and routing data between the systems where required.

---

## Data Mapping Table

| Data Name | Vendor Portal | Finance Portal | Database | Data Type | Data Length | Validation |
|-----------|---------------|----------------|----------|-----------|-------------|------------|
| Vendor ID | `vendor_id` | `vendor_id` | `vendor_id` | VARCHAR | 20 | Required; must exist |
| Vendor Name | `vendor_name` | `vendor_name` | `vendor_name` | VARCHAR | 100 | Required; cannot be empty |
| Contract Number | `contract_number` | `contract_number` | `contract_number` | VARCHAR | 50 | Required; must exist |
| Work Description | `work_description` | `work_description` | `work_description` | VARCHAR | 500 | Required; cannot be empty |
| Invoice Number | `invoice_number` | `invoice_number` | `invoice_number` | VARCHAR | 50 | Required; must be unique |
| Invoice Amount | `invoice_amount` | `invoice_amount` | `invoice_amount` | DECIMAL | 18,2 | Required; must be greater than 0 |
| Invoice Date | `invoice_date` | `invoice_date` | `invoice_date` | DATE | — | Required; valid date |
| Currency Code | `currency_code` | `currency_code` | `currency_code` | CHAR | 3 | Required; valid ISO 4217 code |
| Approval Document | `approval_document` | `signed_document` | `approval_document` | PDF | — | Required after approval; digitally signed |
| Created Date | `created_date` | `created_date` | `created_date` | DATETIME | — | System generated |


## Mapping Notes

- The Vendor Portal is the source of vendor, contract, work, and invoice information used by the Payment Acquisition process.
- The Integration Layer validates and routes the data between the Vendor Portal and Finance Portal.
- The Finance Portal uses the received information to perform the payment approval workflow.
- The Finance Portal generates the digitally signed approval document after successful approval.
- The signed approval document is returned through the Integration Layer and made available through the Vendor Portal.
- The database stores the relevant transaction data and the resulting approval document.
- `—` indicates that data length is not applicable to the field.
