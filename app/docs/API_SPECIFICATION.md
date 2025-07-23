# KARMA Payment Approval System - API Specification

## Base URL
```
Development: http://localhost:3000/api
Production: https://karma-approval.vercel.app/api
```

## Authentication
All authenticated endpoints require JWT token in Authorization header:
```
Authorization: Bearer <jwt_token>
```

## API Endpoints

### 1. Authentication Endpoints

#### POST /api/auth/login
Login user and receive JWT token.

**Request Body:**
```json
{
  "email": "kevin@karma.org",
  "password": "securepassword"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "user_123",
      "email": "kevin@karma.org",
      "name": "Kevin",
      "role": "manager"
    }
  }
}
```

**Error Response (401):**
```json
{
  "success": false,
  "error": "Invalid credentials"
}
```

#### POST /api/auth/logout
Logout user and invalidate session.

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

#### POST /api/auth/forgot-password
Send password reset email.

**Request Body:**
```json
{
  "email": "kevin@karma.org"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password reset email sent"
}
```

#### POST /api/auth/reset-password
Reset password with token.

**Request Body:**
```json
{
  "token": "reset_token_here",
  "newPassword": "newSecurePassword"
}
```

### 2. User Management

#### GET /api/users
Get list of all users (Admin only).

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "user_123",
      "email": "kevin@karma.org",
      "name": "Kevin",
      "role": "manager",
      "createdAt": "2025-07-23T10:00:00Z"
    }
  ]
}
```

#### GET /api/users/profile
Get current user profile.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "email": "kevin@karma.org",
    "name": "Kevin",
    "role": "manager",
    "permissions": ["create_request", "view_all_requests"]
  }
}
```

#### PUT /api/users/:id
Update user information.

**Request Body:**
```json
{
  "email": "kevin.new@karma.org",
  "name": "Kevin Manager"
}
```

### 3. Payment Request Endpoints

#### POST /api/payment-requests
Create new payment request.

**Request Body:**
```json
{
  "supplierName": "ABC Suppliers Ltd",
  "supplierEmail": "accounts@abcsuppliers.com",
  "supplierPhone": "+254700000000",
  "bankName": "KCB Bank",
  "accountNumber": "1234567890",
  "amount": 50000,
  "currency": "KES",
  "description": "Payment for office supplies delivered on July 20, 2025",
  "approvalWorkflow": ["treasurer", "secretary_general", "finance"],
  "documents": ["doc_123", "doc_124"]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "req_20250723_001",
    "status": "pending",
    "createdBy": "user_123",
    "createdAt": "2025-07-23T10:30:00Z",
    "currentApprover": "treasurer",
    "approvalStep": 1
  }
}
```

#### GET /api/payment-requests
Get all payment requests with filters.

**Query Parameters:**
- `status`: pending | approved | rejected | completed
- `createdBy`: user_id
- `dateFrom`: ISO date string
- `dateTo`: ISO date string
- `page`: number (default: 1)
- `limit`: number (default: 20)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "requests": [
      {
        "id": "req_20250723_001",
        "supplierName": "ABC Suppliers Ltd",
        "amount": 50000,
        "status": "pending",
        "createdAt": "2025-07-23T10:30:00Z",
        "currentApprover": "treasurer"
      }
    ],
    "pagination": {
      "total": 45,
      "page": 1,
      "limit": 20,
      "pages": 3
    }
  }
}
```

#### GET /api/payment-requests/:id
Get specific payment request details.

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "req_20250723_001",
    "supplierName": "ABC Suppliers Ltd",
    "supplierDetails": {
      "email": "accounts@abcsuppliers.com",
      "phone": "+254700000000",
      "bankName": "KCB Bank",
      "accountNumber": "1234567890"
    },
    "amount": 50000,
    "currency": "KES",
    "description": "Payment for office supplies...",
    "status": "pending",
    "approvalWorkflow": ["treasurer", "secretary_general", "finance"],
    "currentApprover": "treasurer",
    "approvalHistory": [],
    "documents": [
      {
        "id": "doc_123",
        "name": "invoice.pdf",
        "url": "/api/documents/doc_123",
        "uploadedAt": "2025-07-23T10:25:00Z"
      }
    ],
    "createdBy": {
      "id": "user_123",
      "name": "Kevin"
    },
    "createdAt": "2025-07-23T10:30:00Z"
  }
}
```

#### PUT /api/payment-requests/:id
Update payment request (only if pending and by creator).

**Request Body:**
```json
{
  "amount": 55000,
  "description": "Updated payment description"
}
```

#### DELETE /api/payment-requests/:id
Delete payment request (only if pending and by creator).

### 4. Approval Endpoints

#### POST /api/approvals
Submit approval or rejection.

**Request Body:**
```json
{
  "requestId": "req_20250723_001",
  "action": "approve", // or "reject"
  "comments": "Verified and approved",
  "signatureId": "docusign_sig_123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "approval_123",
    "requestId": "req_20250723_001",
    "approver": "treasurer",
    "action": "approve",
    "comments": "Verified and approved",
    "signatureId": "docusign_sig_123",
    "timestamp": "2025-07-23T11:00:00Z",
    "nextApprover": "secretary_general"
  }
}
```

#### GET /api/approvals/:requestId
Get approval history for a request.

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "approval_123",
      "approver": {
        "id": "user_456",
        "name": "Stephen Magwilu",
        "role": "treasurer"
      },
      "action": "approve",
      "comments": "Verified and approved",
      "signatureUrl": "/api/signatures/docusign_sig_123",
      "timestamp": "2025-07-23T11:00:00Z"
    }
  ]
}
```

### 5. Document Management

#### POST /api/documents/upload
Upload supporting documents.

**Request:** Multipart form data
- `file`: File binary
- `requestId`: Associated payment request ID

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "doc_125",
    "name": "receipt.pdf",
    "size": 125000,
    "mimeType": "application/pdf",
    "url": "/api/documents/doc_125"
  }
}
```

#### GET /api/documents/:id
Download document.

**Response:** File download with appropriate headers

### 6. Webhook Endpoints

#### POST /api/webhooks/make
Receive webhooks from Make.com.

**Headers:**
```
X-Webhook-Secret: configured_secret
```

**Request Body:**
```json
{
  "event": "approval_submitted",
  "requestId": "req_20250723_001",
  "approver": "treasurer",
  "action": "approve",
  "timestamp": "2025-07-23T11:00:00Z"
}
```

#### POST /api/webhooks/docusign
Receive DocuSign webhook events.

**Request Body:** DocuSign event payload

### 7. Audit & Reporting

#### GET /api/audit-logs
Get audit logs with filters.

**Query Parameters:**
- `userId`: Filter by user
- `action`: Filter by action type
- `entityType`: payment_request | approval
- `dateFrom`: ISO date string
- `dateTo`: ISO date string

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "log_123",
      "userId": "user_123",
      "userName": "Kevin",
      "action": "create_payment_request",
      "entityType": "payment_request",
      "entityId": "req_20250723_001",
      "details": {
        "amount": 50000,
        "supplier": "ABC Suppliers Ltd"
      },
      "ipAddress": "192.168.1.100",
      "timestamp": "2025-07-23T10:30:00Z"
    }
  ]
}
```

#### GET /api/reports/summary
Get summary statistics.

**Query Parameters:**
- `period`: daily | weekly | monthly | yearly
- `date`: ISO date string

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "period": "monthly",
    "date": "2025-07-01",
    "statistics": {
      "totalRequests": 45,
      "pendingRequests": 5,
      "approvedRequests": 35,
      "rejectedRequests": 5,
      "totalAmount": 2500000,
      "averageApprovalTime": "2.5 days",
      "approvalRate": "87.5%"
    }
  }
}
```

## Error Handling

All errors follow this format:
```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {} // Optional additional information
}
```

### Common Error Codes:
- `UNAUTHORIZED`: Missing or invalid authentication
- `FORBIDDEN`: Insufficient permissions
- `NOT_FOUND`: Resource not found
- `VALIDATION_ERROR`: Input validation failed
- `DUPLICATE_ENTRY`: Resource already exists
- `INTERNAL_ERROR`: Server error

## Rate Limiting

- 100 requests per minute per IP
- 1000 requests per hour per authenticated user
- Headers included in response:
  - `X-RateLimit-Limit`
  - `X-RateLimit-Remaining`
  - `X-RateLimit-Reset`

## Webhooks

### Outgoing Webhooks (to Make.com)

Events sent:
1. `payment_request.created`
2. `payment_request.approved`
3. `payment_request.rejected`
4. `payment_request.completed`

Payload format:
```json
{
  "event": "payment_request.approved",
  "timestamp": "2025-07-23T11:00:00Z",
  "data": {
    "requestId": "req_20250723_001",
    "approver": "treasurer",
    "nextApprover": "secretary_general",
    "requestDetails": {
      // Full request object
    }
  }
}
```