# KARMA Payment Approval System - Database Schema

## Database: PostgreSQL

## Tables

### 1. users
Stores system users (Kevin, Approvers, Finance)

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL CHECK (role IN ('manager', 'treasurer', 'secretary_general', 'chair_person', 'finance', 'auditor')),
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

### 2. payment_requests
Main table for payment requests

```sql
CREATE TABLE payment_requests (
    id VARCHAR(50) PRIMARY KEY, -- Format: req_YYYYMMDD_XXX
    created_by UUID NOT NULL REFERENCES users(id),
    supplier_name VARCHAR(255) NOT NULL,
    supplier_email VARCHAR(255),
    supplier_phone VARCHAR(50),
    bank_name VARCHAR(255),
    account_number VARCHAR(100),
    amount DECIMAL(15,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'KES',
    description TEXT NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'pending' CHECK (status IN ('draft', 'pending', 'approved', 'rejected', 'completed', 'cancelled')),
    current_approver_role VARCHAR(50),
    approval_step INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    CONSTRAINT fk_created_by FOREIGN KEY (created_by) REFERENCES users(id)
);

-- Indexes
CREATE INDEX idx_payment_requests_status ON payment_requests(status);
CREATE INDEX idx_payment_requests_created_by ON payment_requests(created_by);
CREATE INDEX idx_payment_requests_created_at ON payment_requests(created_at);
```

### 3. approval_workflows
Defines the approval order for each request

```sql
CREATE TABLE approval_workflows (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id VARCHAR(50) NOT NULL REFERENCES payment_requests(id) ON DELETE CASCADE,
    step_order INTEGER NOT NULL,
    approver_role VARCHAR(50) NOT NULL,
    is_completed BOOLEAN DEFAULT false,
    completed_at TIMESTAMP,
    CONSTRAINT uk_workflow_step UNIQUE (request_id, step_order)
);

-- Indexes
CREATE INDEX idx_approval_workflows_request ON approval_workflows(request_id);
```

### 4. approvals
Records all approval actions

```sql
CREATE TABLE approvals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id VARCHAR(50) NOT NULL REFERENCES payment_requests(id) ON DELETE CASCADE,
    approver_id UUID NOT NULL REFERENCES users(id),
    action VARCHAR(20) NOT NULL CHECK (action IN ('approve', 'reject')),
    comments TEXT,
    signature_id VARCHAR(255), -- DocuSign envelope ID
    signature_url TEXT,
    ip_address INET,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_approvals_request ON approvals(request_id);
CREATE INDEX idx_approvals_approver ON approvals(approver_id);
CREATE INDEX idx_approvals_created_at ON approvals(created_at);
```

### 5. documents
Stores uploaded supporting documents

```sql
CREATE TABLE documents (
    id VARCHAR(50) PRIMARY KEY, -- Format: doc_XXX
    request_id VARCHAR(50) NOT NULL REFERENCES payment_requests(id) ON DELETE CASCADE,
    uploaded_by UUID NOT NULL REFERENCES users(id),
    file_name VARCHAR(255) NOT NULL,
    file_size INTEGER NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    storage_path TEXT NOT NULL,
    is_deleted BOOLEAN DEFAULT false,
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_documents_request ON documents(request_id);
```

### 6. audit_logs
Comprehensive audit trail

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    user_email VARCHAR(255), -- Store email in case user is deleted
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(50) NOT NULL,
    entity_id VARCHAR(255),
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
```

### 7. email_queue
Tracks email notifications

```sql
CREATE TABLE email_queue (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    request_id VARCHAR(50) REFERENCES payment_requests(id),
    recipient_email VARCHAR(255) NOT NULL,
    cc_emails TEXT[], -- Array of CC emails
    subject VARCHAR(500) NOT NULL,
    template_name VARCHAR(100) NOT NULL,
    template_data JSONB NOT NULL,
    status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'sent', 'failed')),
    attempts INTEGER DEFAULT 0,
    sent_at TIMESTAMP,
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_email_queue_status ON email_queue(status);
CREATE INDEX idx_email_queue_request ON email_queue(request_id);
```

### 8. webhook_logs
Logs all webhook activities

```sql
CREATE TABLE webhook_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    webhook_type VARCHAR(50) NOT NULL, -- 'incoming' or 'outgoing'
    service VARCHAR(50) NOT NULL, -- 'make', 'docusign'
    event_type VARCHAR(100),
    url TEXT,
    headers JSONB,
    payload JSONB,
    response_status INTEGER,
    response_body JSONB,
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_webhook_logs_type ON webhook_logs(webhook_type, service);
CREATE INDEX idx_webhook_logs_created_at ON webhook_logs(created_at);
```

### 9. sessions
User session management

```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) UNIQUE NOT NULL,
    ip_address INET,
    user_agent TEXT,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_sessions_user ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token_hash);
CREATE INDEX idx_sessions_expires ON sessions(expires_at);
```

### 10. notifications
In-app notifications

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    data JSONB,
    is_read BOOLEAN DEFAULT false,
    read_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_notifications_user ON notifications(user_id, is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

## Views

### 1. payment_request_summary
Simplified view for dashboards

```sql
CREATE VIEW payment_request_summary AS
SELECT 
    pr.id,
    pr.supplier_name,
    pr.amount,
    pr.currency,
    pr.status,
    pr.created_at,
    pr.current_approver_role,
    u.name as created_by_name,
    u.email as created_by_email,
    (
        SELECT COUNT(*) 
        FROM approvals a 
        WHERE a.request_id = pr.id AND a.action = 'approve'
    ) as approval_count,
    (
        SELECT COUNT(*) 
        FROM approval_workflows aw 
        WHERE aw.request_id = pr.id
    ) as total_approvers_required
FROM payment_requests pr
JOIN users u ON pr.created_by = u.id;
```

### 2. user_pending_approvals
Shows pending approvals for each user

```sql
CREATE VIEW user_pending_approvals AS
SELECT 
    u.id as user_id,
    u.name as user_name,
    u.role as user_role,
    pr.id as request_id,
    pr.supplier_name,
    pr.amount,
    pr.created_at as request_date,
    pr.current_approver_role
FROM users u
JOIN payment_requests pr ON pr.current_approver_role = u.role
WHERE pr.status = 'pending'
    AND NOT EXISTS (
        SELECT 1 FROM approvals a 
        WHERE a.request_id = pr.id 
        AND a.approver_id = u.id
    );
```

## Functions & Triggers

### 1. Update timestamp trigger

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Apply to tables
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_payment_requests_updated_at BEFORE UPDATE ON payment_requests
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_email_queue_updated_at BEFORE UPDATE ON email_queue
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 2. Audit log trigger

```sql
CREATE OR REPLACE FUNCTION create_audit_log()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_logs (
        user_id,
        user_email,
        action,
        entity_type,
        entity_id,
        old_values,
        new_values,
        ip_address
    ) VALUES (
        current_setting('app.current_user_id')::UUID,
        current_setting('app.current_user_email'),
        TG_OP,
        TG_TABLE_NAME,
        CASE 
            WHEN TG_OP = 'DELETE' THEN OLD.id::VARCHAR
            ELSE NEW.id::VARCHAR
        END,
        CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN to_jsonb(OLD) ELSE NULL END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) ELSE NULL END,
        inet(current_setting('app.current_ip_address'))
    );
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Apply to critical tables
CREATE TRIGGER audit_payment_requests AFTER INSERT OR UPDATE OR DELETE ON payment_requests
    FOR EACH ROW EXECUTE FUNCTION create_audit_log();

CREATE TRIGGER audit_approvals AFTER INSERT OR UPDATE OR DELETE ON approvals
    FOR EACH ROW EXECUTE FUNCTION create_audit_log();
```

## Initial Data Seeds

```sql
-- Insert default users
INSERT INTO users (email, password_hash, name, role) VALUES
('kevin@karma.org', '$2b$10$...', 'Kevin', 'manager'),
('treasurer@karma.org', '$2b$10$...', 'Stephen Magwilu', 'treasurer'),
('sg@karma.org', '$2b$10$...', 'Collins Mutimba', 'secretary_general'),
('chair@karma.org', '$2b$10$...', 'Dr. Cleophas Ambira', 'chair_person'),
('finance@karma.org', '$2b$10$...', 'Finance Office', 'finance');
```

## Backup Strategy

1. **Automated Daily Backups**
   - Full database backup at 2 AM EAT
   - Retain for 30 days
   - Store in separate cloud location

2. **Point-in-Time Recovery**
   - Enable WAL archiving
   - 7-day recovery window

3. **Audit Log Archival**
   - Archive logs older than 1 year
   - Compress and store for 7 years

## Performance Considerations

1. **Indexes**: All foreign keys and commonly queried fields
2. **Partitioning**: Consider partitioning audit_logs by month
3. **Connection Pooling**: Use PgBouncer or similar
4. **Query Optimization**: Monitor slow queries
5. **Vacuum**: Regular autovacuum for performance