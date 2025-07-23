# KARMA Payment Approval System - Requirements Document

## 1. Project Overview

### 1.1 Purpose
Automate KARMA's manual supplier payment approval process by digitizing forms, implementing electronic signatures, and creating an intelligent workflow system with email notifications.

### 1.2 Stakeholders
- **Kevin** - Manager (Form initiator)
- **Dr. Cleophas Ambira** - Chair Person (Approver)
- **Collins Mutimba** - Secretary General (Approver)
- **Stephen Magwilu** - Treasurer (Approver)
- **Finance Office** - Payment processor
- **Auditors** - System users for compliance checking

## 2. Functional Requirements

### 2.1 User Authentication
- **FR-001**: System shall allow users to log in using email and password
- **FR-002**: System shall maintain user sessions with JWT tokens
- **FR-003**: System shall support password reset functionality
- **FR-004**: System shall timeout inactive sessions after 30 minutes

### 2.2 Payment Request Management
- **FR-005**: Kevin shall be able to create new payment requests with:
  - Supplier name and details
  - Payment amount
  - Payment description/reason
  - Supporting documents (PDF, images)
  - Approval workflow selection
- **FR-006**: System shall generate unique request IDs for tracking
- **FR-007**: Users shall view all their payment requests with status
- **FR-008**: Users shall filter requests by status, date, amount

### 2.3 Approval Workflow
- **FR-009**: Kevin shall select approval order from predefined options:
  - Treasurer → Secretary General → Finance
  - Treasurer → Chair Person → Finance
  - Secretary General → Treasurer → Finance
  - Secretary General → Chair Person → Finance
  - Treasurer → Secretary General → Chair Person → Finance
  - Other valid combinations
- **FR-010**: System shall enforce minimum 2 approvals before finance
- **FR-011**: Approvers shall:
  - View complete request details
  - Approve with digital signature
  - Reject with mandatory comments
  - Add optional comments on approval
- **FR-012**: System shall return request to Kevin if any approver rejects

### 2.4 Digital Signatures
- **FR-013**: System shall integrate with DocuSign for legal signatures
- **FR-014**: Each approval shall embed approver's digital signature
- **FR-015**: Signatures shall include timestamp and IP address

### 2.5 Email Notifications
- **FR-016**: System shall send emails via Make.com webhooks for:
  - New request submitted (to first approver)
  - Request approved (to next approver)
  - Request rejected (to Kevin)
  - All approvals complete (to Finance + Kevin)
  - Request requires review (to Kevin)
- **FR-017**: Kevin shall be CC'd on all approval emails
- **FR-018**: Emails shall include request summary and action links

### 2.6 Audit Trail & Compliance
- **FR-019**: System shall log all actions with:
  - User identification
  - Action type
  - Timestamp
  - IP address
  - Previous and new values (for updates)
- **FR-020**: System shall store all documents permanently
- **FR-021**: Auditors shall search and export audit logs
- **FR-022**: System shall generate compliance reports

### 2.7 Real-time Updates
- **FR-023**: Dashboard shall show real-time status updates
- **FR-024**: Users shall receive in-app notifications
- **FR-025**: System shall use WebSockets for live updates

## 3. Non-Functional Requirements

### 3.1 Performance
- **NFR-001**: Page load time < 3 seconds
- **NFR-002**: API response time < 1 second
- **NFR-003**: Support 50 concurrent users
- **NFR-004**: Handle 100 requests per day

### 3.2 Security
- **NFR-005**: All data transmission via HTTPS
- **NFR-006**: Passwords hashed using bcrypt
- **NFR-007**: API authentication via JWT
- **NFR-008**: Input validation against injections
- **NFR-009**: CORS properly configured

### 3.3 Usability
- **NFR-010**: Mobile-responsive design
- **NFR-011**: Support Chrome, Firefox, Safari, Edge
- **NFR-012**: Intuitive UI requiring minimal training
- **NFR-013**: Clear error messages

### 3.4 Reliability
- **NFR-014**: 99% uptime
- **NFR-015**: Daily automated backups
- **NFR-016**: Graceful error handling
- **NFR-017**: Transaction integrity

### 3.5 Compliance
- **NFR-018**: Comply with Kenya Data Protection Act
- **NFR-019**: Maintain audit trail for 7 years
- **NFR-020**: Support data export for audits

## 4. System Constraints

### 4.1 Technical Constraints
- Frontend: Next.js with JavaScript
- Backend: Next.js API routes
- Database: PostgreSQL (cloud-hosted)
- Email: Gmail API via Make.com
- E-signatures: DocuSign API
- Hosting: Vercel (frontend), cloud (database)

### 4.2 Business Constraints
- Development timeline: 15 days
- Must integrate with existing KARMA processes
- Zero downtime migration from manual process

## 5. Assumptions

1. All approvers have email addresses
2. All users have internet access
3. DocuSign is legally valid in Kenya
4. Make.com webhooks are reliable
5. Users have modern web browsers

## 6. Dependencies

1. DocuSign developer account approval
2. Gmail API credentials
3. Make.com webhook availability
4. Cloud database provisioning
5. Vercel hosting setup