# KARMA Payment Approval System - Workflow Diagrams

## 1. Overall System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           KARMA System                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐         │
│  │   Next.js   │     │  PostgreSQL │     │   Make.com  │         │
│  │  Frontend   │────▶│   Database  │     │  Webhooks   │         │
│  │     +       │     └─────────────┘     └──────┬──────┘         │
│  │  API Routes │                                 │                 │
│  └─────────────┘                                 │                 │
│         │                                         │                 │
│         └─────────────────┬──────────────────────┘                 │
│                           │                                         │
│  ┌─────────────┐    ┌────▼─────┐     ┌─────────────┐            │
│  │  DocuSign   │────│  Email   │────▶│  Gmail API  │            │
│  │     API     │    │  Queue   │     │             │            │
│  └─────────────┘    └──────────┘     └─────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 2. User Authentication Flow

```
┌──────┐                    ┌──────────┐                    ┌──────────┐
│ User │                    │ Frontend │                    │   API    │
└──┬───┘                    └────┬─────┘                    └────┬─────┘
   │                              │                                │
   │   1. Enter credentials       │                                │
   ├─────────────────────────────▶│                                │
   │                              │                                │
   │                              │  2. POST /api/auth/login       │
   │                              ├───────────────────────────────▶│
   │                              │                                │
   │                              │                                ├─┐
   │                              │                                │ │ 3. Validate
   │                              │                                │ │    credentials
   │                              │                                │◀┘
   │                              │                                │
   │                              │  4. Return JWT token           │
   │                              │◀───────────────────────────────┤
   │                              │                                │
   │  5. Redirect to dashboard    │                                │
   │◀─────────────────────────────┤                                │
   │                              │                                │
```

## 3. Payment Request Creation Flow

```
┌───────┐          ┌──────────┐          ┌─────────┐          ┌──────────┐
│ Kevin │          │ Frontend │          │   API   │          │ Database │
└───┬───┘          └────┬─────┘          └────┬────┘          └────┬─────┘
    │                    │                      │                    │
    │ 1. Fill form       │                      │                    │
    ├───────────────────▶│                      │                    │
    │                    │                      │                    │
    │                    │ 2. Validate inputs   │                    │
    │                    ├─┐                    │                    │
    │                    │◀┘                    │                    │
    │                    │                      │                    │
    │ 3. Select workflow │                      │                    │
    ├───────────────────▶│                      │                    │
    │                    │                      │                    │
    │ 4. Upload docs     │                      │                    │
    ├───────────────────▶│                      │                    │
    │                    │                      │                    │
    │ 5. Submit          │                      │                    │
    ├───────────────────▶│                      │                    │
    │                    │                      │                    │
    │                    │ 6. POST /api/payment-requests            │
    │                    ├─────────────────────▶│                    │
    │                    │                      │                    │
    │                    │                      │ 7. Save to DB      │
    │                    │                      ├───────────────────▶│
    │                    │                      │                    │
    │                    │                      │ 8. Generate ID     │
    │                    │                      │◀───────────────────┤
    │                    │                      │                    │
    │                    │                      ├─┐                  │
    │                    │                      │ │ 9. Trigger       │
    │                    │                      │ │    webhook       │
    │                    │                      │◀┘                  │
    │                    │                      │                    │
    │                    │ 10. Return success   │                    │
    │                    │◀─────────────────────┤                    │
    │                    │                      │                    │
    │ 11. Show confirm   │                      │                    │
    │◀───────────────────┤                      │                    │
    │                    │                      │                    │
```

## 4. Approval Workflow Process

```
┌─────────┐     ┌──────────┐     ┌─────────┐     ┌──────────┐     ┌─────────┐
│  Kevin  │     │Treasurer │     │Secretary│     │  Chair   │     │ Finance │
│         │     │          │     │ General │     │  Person  │     │         │
└────┬────┘     └────┬─────┘     └────┬────┘     └────┬─────┘     └────┬────┘
     │                │                 │                │                 │
     │ Create request │                 │                │                 │
     ├───────────────▶│                 │                │                 │
     │                │                 │                │                 │
     │                │ Email sent      │                │                 │
     │                │◀─ ─ ─ ─ ─ ─ ─ ─ │                │                 │
     │                │                 │                │                 │
     │                │ Review & Sign   │                │                 │
     │                ├─┐               │                │                 │
     │                │◀┘               │                │                 │
     │                │                 │                │                 │
     │ CC: Approved   │                 │                │                 │
     │◀─ ─ ─ ─ ─ ─ ─ ─│                 │                │                 │
     │                │                 │                │                 │
     │                │ Forward to SG   │                │                 │
     │                ├────────────────▶│                │                 │
     │                │                 │                │                 │
     │                │                 │ Email sent     │                 │
     │                │                 │◀─ ─ ─ ─ ─ ─ ─ ─│                 │
     │                │                 │                │                 │
     │                │                 │ Review & Sign  │                 │
     │                │                 ├─┐              │                 │
     │                │                 │◀┘              │                 │
     │                │                 │                │                 │
     │ CC: Approved   │                 │                │                 │
     │◀─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│                │                 │
     │                │                 │                │                 │
     │                │                 │ Forward (opt)  │                 │
     │                │                 ├───────────────▶│                 │
     │                │                 │                │                 │
     │                │                 │                │ Review & Sign  │
     │                │                 │                ├─┐              │
     │                │                 │                │◀┘              │
     │                │                 │                │                 │
     │ CC: Approved   │                 │                │                 │
     │◀─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│                 │
     │                │                 │                │                 │
     │                │                 │                │ Send to Finance │
     │                │                 │                ├────────────────▶│
     │                │                 │                │                 │
     │ Final notify   │                 │                │                 │
     │◀─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
     │                │                 │                │                 │
```

## 5. Rejection Flow

```
┌─────────┐          ┌──────────┐          ┌──────────┐          ┌─────────┐
│  Kevin  │          │Treasurer │          │Secretary │          │   API   │
│         │          │          │          │ General  │          │         │
└────┬────┘          └────┬─────┘          └────┬─────┘          └────┬────┘
     │                     │                      │                      │
     │ 1. Create request   │                      │                      │
     ├─────────────────────┼──────────────────────┼─────────────────────▶│
     │                     │                      │                      │
     │                     │ 2. Notification      │                      │
     │                     │◀─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
     │                     │                      │                      │
     │                     │ 3. Approve           │                      │
     │                     ├──────────────────────┼─────────────────────▶│
     │                     │                      │                      │
     │                     │                      │ 4. Notification      │
     │                     │                      │◀─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
     │                     │                      │                      │
     │                     │                      │ 5. Reject + Comments │
     │                     │                      ├─────────────────────▶│
     │                     │                      │                      │
     │                     │                      │                      ├─┐
     │                     │                      │                      │ │ 6. Update
     │                     │                      │                      │ │    status
     │                     │                      │                      │◀┘
     │                     │                      │                      │
     │ 7. Rejection email  │                      │                      │
     │◀─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
     │                     │                      │                      │
     ├─┐                   │                      │                      │
     │ │ 8. Review &       │                      │                      │
     │ │    resubmit       │                      │                      │
     │◀┘                   │                      │                      │
     │                     │                      │                      │
```

## 6. Make.com Webhook Integration

```
┌──────────┐          ┌─────────┐          ┌──────────┐          ┌─────────┐
│   API    │          │ Make.com│          │  Gmail   │          │Approver │
└────┬─────┘          └────┬────┘          └────┬─────┘          └────┬────┘
     │                      │                     │                     │
     │ 1. Trigger webhook   │                     │                     │
     ├─────────────────────▶│                     │                     │
     │   (POST with data)   │                     │                     │
     │                      │                     │                     │
     │                      ├─┐                   │                     │
     │                      │ │ 2. Process        │                     │
     │                      │ │    scenario       │                     │
     │                      │◀┘                   │                     │
     │                      │                     │                     │
     │                      │ 3. Send email      │                     │
     │                      ├───────────────────▶│                     │
     │                      │   (Gmail API)      │                     │
     │                      │                     │                     │
     │                      │                     │ 4. Deliver email   │
     │                      │                     ├───────────────────▶│
     │                      │                     │                     │
     │ 5. Webhook response  │                     │                     │
     │◀─────────────────────┤                     │                     │
     │   (success/failure)  │                     │                     │
     │                      │                     │                     │
```

## 7. DocuSign Integration Flow

```
┌──────────┐          ┌─────────┐          ┌──────────┐          ┌──────────┐
│ Approver │          │   API   │          │ DocuSign │          │ Database │
└────┬─────┘          └────┬────┘          └────┬─────┘          └────┬─────┘
     │                      │                     │                     │
     │ 1. Click approve     │                     │                     │
     ├─────────────────────▶│                     │                     │
     │                      │                     │                     │
     │                      │ 2. Create envelope  │                     │
     │                      ├───────────────────▶│                     │
     │                      │                     │                     │
     │                      │ 3. Return URL      │                     │
     │                      │◀───────────────────┤                     │
     │                      │                     │                     │
     │ 4. Redirect to sign  │                     │                     │
     │◀─────────────────────┤                     │                     │
     │                      │                     │                     │
     │ 5. Sign document     │                     │                     │
     ├──────────────────────┼───────────────────▶│                     │
     │                      │                     │                     │
     │                      │ 6. Webhook callback│                     │
     │                      │◀───────────────────┤                     │
     │                      │                     │                     │
     │                      │ 7. Store signature │                     │
     │                      ├─────────────────────┼───────────────────▶│
     │                      │                     │                     │
     │ 8. Return to app     │                     │                     │
     │◀─────────────────────┼─────────────────────┤                     │
     │                      │                     │                     │
```

## 8. Database Transaction Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Payment Request Lifecycle                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────┐        ┌───────────┐        ┌───────────┐              │
│  │  DRAFT   │───────▶│  PENDING  │───────▶│ APPROVED  │              │
│  └──────────┘        └─────┬─────┘        └─────┬─────┘              │
│                            │                      │                     │
│                            │                      │                     │
│                            ▼                      ▼                     │
│                      ┌───────────┐        ┌───────────┐              │
│                      │ REJECTED  │        │ COMPLETED │              │
│                      └───────────┘        └───────────┘              │
│                                                                         │
│  Database Updates:                                                      │
│  1. payment_requests.status                                            │
│  2. payment_requests.current_approver_role                             │
│  3. approval_workflows.is_completed                                    │
│  4. approvals (new record)                                             │
│  5. audit_logs (automatic)                                             │
│  6. email_queue (new record)                                           │
│  7. notifications (new record)                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 9. Error Handling Flow

```
┌──────────┐          ┌─────────┐          ┌──────────┐          ┌─────────┐
│   User   │          │   API   │          │ Service  │          │  Error  │
│          │          │         │          │(Email/DB)│          │ Handler │
└────┬─────┘          └────┬────┘          └────┬─────┘          └────┬────┘
     │                      │                     │                     │
     │ 1. Make request      │                     │                     │
     ├─────────────────────▶│                     │                     │
     │                      │                     │                     │
     │                      │ 2. Process request  │                     │
     │                      ├───────────────────▶│                     │
     │                      │                     │                     │
     │                      │                     X Service fails       │
     │                      │                     │                     │
     │                      │ 3. Error response   │                     │
     │                      │◀───────────────────┤                     │
     │                      │                     │                     │
     │                      │ 4. Log error        │                     │
     │                      ├─────────────────────┼───────────────────▶│
     │                      │                     │                     │
     │                      ├─┐                   │                     │
     │                      │ │ 5. Retry logic    │                     │
     │                      │◀┘                   │                     │
     │                      │                     │                     │
     │ 6. User-friendly msg │                     │                     │
     │◀─────────────────────┤                     │                     │
     │                      │                     │                     │
```

## 10. Real-time Notification Flow

```
┌──────────┐          ┌─────────┐          ┌──────────┐          ┌─────────┐
│  User A  │          │   API   │          │WebSocket │          │ User B  │
│ (Kevin)  │          │         │          │ Server   │          │(Approver)
└────┬─────┘          └────┬────┘          └────┬─────┘          └────┬────┘
     │                      │                     │                     │
     │ 1. Submit request    │                     │                     │
     ├─────────────────────▶│                     │                     │
     │                      │                     │                     │
     │                      │ 2. Broadcast event  │                     │
     │                      ├───────────────────▶│                     │
     │                      │                     │                     │
     │                      │                     │ 3. Push to clients │
     │                      │                     ├───────────────────▶│
     │                      │                     │                     │
     │                      │                     │                     ├─┐
     │                      │                     │                     │ │ 4. Update
     │                      │                     │                     │ │    UI
     │                      │                     │                     │◀┘
     │                      │                     │                     │
```

## Key Workflow Rules

1. **Minimum Approvers**: At least 2 approvals required before Finance
2. **Sequential Processing**: Approvals must follow the selected order
3. **Rejection Handling**: Any rejection returns to Kevin for review
4. **CC Requirements**: Kevin is CC'd on all approval emails
5. **Audit Trail**: Every action is logged with timestamp and user info
6. **Digital Signatures**: Required for all approvals via DocuSign
7. **Real-time Updates**: Dashboard reflects current status immediately
8. **Email Notifications**: Sent at each workflow step via Make.com