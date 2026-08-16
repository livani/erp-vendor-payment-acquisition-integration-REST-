# Use Case

## Purpose

This document describes the functional interactions between users and the ERP Vendor Payment Integration solution. It identifies the primary actors, their responsibilities, and the business functions supported by the system.

---

# Actors

| Actor | Description |
|--------|-------------|
| Vendor | External user who submits Payment Acquisition requests and downloads approved documents. |
| Finance Officer | Reviews and approves payment acquisition requests within the Finance Approval System. |
| ERP System | Maintains Vendor Master Data and Payment Acquisition records. |
| Integration Layer | Coordinates communication between the Vendor Portal, ERP System, and Finance Approval System. |

---

# Use Case Diagram

> Insert **use-case-diagram.png**

---

# Use Case Summary

| Use Case ID | Use Case | Primary Actor |
|--------------|-------------------------------|----------------|
| UC-01 | Submit Payment Acquisition Request | Vendor |
| UC-02 | Validate Vendor Information | Integration Layer |
| UC-03 | Retrieve Vendor Master Data | ERP System |
| UC-04 | Submit Payment Approval Request | Integration Layer |
| UC-05 | Approve Payment Request | Finance Officer |
| UC-06 | Generate Signed Payment Document | Finance Approval System |
| UC-07 | Update Payment Acquisition Record | Integration Layer |
| UC-08 | Download Approved Payment Document | Vendor |

---

# Use Case Details

---

## UC-01 – Submit Payment Acquisition Request

### Description

The Vendor submits a new Payment Acquisition request through the Vendor Portal.

### Primary Actor

Vendor

### Preconditions

- Vendor is authenticated.
- Vendor account exists in the ERP System.

### Main Flow

1. Vendor opens the Payment Acquisition page.
2. Vendor completes the request form.
3. Vendor submits the request.
4. Vendor Portal sends the request to the Integration Layer.

### Postconditions

- Request is received for validation.

---

## UC-02 – Validate Vendor Information

### Description

The Integration Layer validates the request payload and verifies that the Vendor exists in the ERP System.

### Primary Actor

Integration Layer

### Preconditions

- Payment request has been received.

### Main Flow

1. Validate request payload.
2. Authenticate request.
3. Retrieve Vendor information from ERP.
4. Confirm Vendor exists.

### Exception

- Vendor not found.
- Invalid payload.
- Authentication failed.

### Postconditions

- Valid request proceeds to Finance Approval.

---

## UC-03 – Retrieve Vendor Master Data

### Description

Retrieve Vendor Master Data from the ERP System.

### Primary Actor

ERP System

### Preconditions

- Vendor validation request received.

### Main Flow

1. Search Vendor Master Data.
2. Return Vendor details.

### Postconditions

- Vendor information returned to Integration Layer.

---

## UC-04 – Submit Payment Approval Request

### Description

The Integration Layer submits the validated Payment Acquisition request to the Finance Approval System.

### Primary Actor

Integration Layer

### Preconditions

- Vendor validation completed successfully.

### Main Flow

1. Transform request payload.
2. Send REST API request.
3. Receive acknowledgement.

### Postconditions

- Payment approval process begins.

---

## UC-05 – Approve Payment Request

### Description

Finance Officer reviews and approves or rejects the payment request.

### Primary Actor

Finance Officer

### Preconditions

- Payment request exists.

### Main Flow

1. Open request.
2. Review payment details.
3. Approve or reject request.

### Postconditions

- Approval status recorded.

---

## UC-06 – Generate Signed Payment Document

### Description

Generate the official signed Payment Acquisition document after approval.

### Primary Actor

Finance Approval System

### Preconditions

- Request approved.

### Main Flow

1. Generate PDF.
2. Apply digital signature.
3. Store document.
4. Return document information.

### Postconditions

- Signed document available.

---

## UC-07 – Update Payment Acquisition Record

### Description

The Integration Layer updates the ERP System with the approval status and document reference.

### Primary Actor

Integration Layer

### Preconditions

- Signed document received.

### Main Flow

1. Update approval status.
2. Save document reference.
3. Save approval timestamp.

### Postconditions

- ERP reflects latest payment status.

---

## UC-08 – Download Approved Payment Document

### Description

Vendor downloads the approved Payment Acquisition document.

### Primary Actor

Vendor

### Preconditions

- Payment request approved.

### Main Flow

1. Vendor opens request history.
2. Vendor selects approved request.
3. Vendor downloads signed PDF.

### Postconditions

- Vendor receives approved document.
