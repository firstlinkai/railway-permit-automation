# Airtable Structure for Railway Permit System

## Base Setup

**Base Name:** Railway Permit Management
**Description:** Digital permit-to-work system for railway operations

## Tables

### 1. Permits (Main Table)

| Field Name | Field Type | Description | Example |
|------------|------------|-------------|---------|
| Permit ID | Formula | Auto-generated ID | `20260201083045` |
| Worker Name | Single line text | Name of worker requesting permit | John Smith |
| Worker Phone | Phone | Contact number for SMS notifications | +639171234567 |
| Work Type | Single select | Type of work being performed | Maintenance, Construction, Inspection, Electrical |
| Location | Single line text | Specific work location | Track 3, Signal Box A, Maintenance Depot |
| Duration (hours) | Number | Estimated work duration | 4, 8, 12 |
| Contractor | Single line text | Contractor company name | ABC Contractors Inc. |
| Hazards Identified | Long text | Identified hazards and controls | Working at height, PPE required |
| JSA Document | Attachment | Job Safety Analysis document | jsa_20260201.pdf |
| Status | Single select | Current permit status | pending_supervisor, pending_safety, approved, rejected, completed |
| Supervisor | Collaborator | Supervisor responsible for approval | supervisor@railway.com |
| Safety Officer | Collaborator | Safety officer for review | safety@railway.com |
| Permit Issuer | Collaborator | Final permit issuer | issuer@railway.com |
| Submitted At | Date & time | When permit was submitted | 2026-02-01 08:30 |
| Supervisor Approved At | Date & time | Supervisor approval timestamp | 2026-02-01 08:45 |
| Safety Approved At | Date & time | Safety approval timestamp | 2026-02-01 09:15 |
| Issued At | Date & time | When permit was issued | 2026-02-01 09:30 |
| Completed At | Date & time | When work was completed | 2026-02-01 13:30 |
| Digital Permit URL | URL | Link to PDF permit | https://permit.railway.com/20260201083045 |
| QR Code | Barcode | QR code for site verification | [QR image] |
| Notes | Long text | Additional notes or comments | Additional lighting required |

### 2. Workers

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Name | Single line text | Full name |
| Employee ID | Single line text | Company ID |
| Department | Single select | Maintenance, Electrical, Signals, etc. |
| Phone | Phone | Contact number |
| Email | Email | Company email |
| Training Expiry | Date | Safety training expiry date |
| Active | Checkbox | Is worker active? |

### 3. Contractors

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Company Name | Single line text | Contractor name |
| Contact Person | Single line text | Primary contact |
| Phone | Phone | Contact number |
| Email | Email | Contact email |
| Insurance Expiry | Date | Insurance validity |
| Safety Rating | Rating | 1-5 safety rating |
| Status | Single select | Active, Suspended, Inactive |

### 4. Locations

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Location Code | Single line text | Unique location code | TRK-03, SIG-A, DEP-01 |
| Description | Long text | Location description |
| Zone | Single select | Red, Amber, Green zones |
| Required Approvals | Multiple select | Supervisor, Safety, Manager, etc. |
| Special Requirements | Long text | Any special requirements |

## Views

### 1. Active Permits Dashboard
- **Filter:** `Status` is not "completed" and not "rejected"
- **Group by:** `Status`
- **Sort:** `Submitted At` descending
- **Fields:** Permit ID, Worker Name, Work Type, Location, Duration, Status

### 2. Supervisor Approval Queue
- **Filter:** `Status` = "pending_supervisor"
- **Sort:** `Submitted At` ascending
- **Fields:** Permit ID, Worker Name, Work Type, Location, Hazards, Submitted At

### 3. Safety Review Queue
- **Filter:** `Status` = "pending_safety"
- **Sort:** `Submitted At` ascending
- **Fields:** Permit ID, Worker Name, Work Type, Location, Hazards, Supervisor Approved At

### 4. Today's Permits
- **Filter:** `Submitted At` is today
- **Sort:** `Submitted At` descending
- **Fields:** Permit ID, Worker Name, Work Type, Location, Status, Issued At

### 5. Compliance Report
- **Filter:** `Completed At` is in the last 30 days
- **Group by:** `Work Type`
- **Fields:** Permit ID, Worker Name, Location, Duration, Supervisor, Safety Officer

## Formulas

### 1. Permit ID (Auto-generated)
```
DATETIME_FORMAT(NOW(), 'YYYYMMDDHHmmss')
```

### 2. Processing Time
```
IF(
  AND({Issued At}, {Submitted At}),
  DATETIME_DIFF({Issued At}, {Submitted At}, 'minutes'),
  BLANK()
)
```

### 3. Status Color
```
SWITCH(
  {Status},
  "pending_supervisor", "yellow",
  "pending_safety", "orange", 
  "approved", "green",
  "rejected", "red",
  "completed", "blue",
  "gray"
)
```

## Automation Rules

### 1. Auto-assign Supervisor
**Trigger:** Record created
**Condition:** `{Status}` = "pending_supervisor"
**Action:** Assign to supervisor based on location zone

### 2. Status Change Notifications
**Trigger:** `{Status}` changes
**Action:** Send email/Slack notification based on new status

### 3. Training Expiry Alert
**Trigger:** 30 days before `{Training Expiry}`
**Action:** Send reminder to HR and worker

## Integration Points

### 1. n8n Webhook
- **Endpoint:** `https://hooks.airtable.com/workflows/...`
- **Trigger:** New record in Permits table
- **Payload:** Full record data

### 2. Form Submission
- **Tool:** Airtable Forms or external form (Typeform, Jotform)
- **Mapping:** Form fields → Airtable fields
- **Validation:** Required fields, phone format, etc.

### 3. PDF Generation
- **Trigger:** `{Status}` changes to "approved"
- **Action:** Generate PDF with QR code
- **Storage:** Airtable attachments or cloud storage

## Security Considerations

1. **Field-level permissions:** HR data visible only to HR department
2. **Location restrictions:** Maintenance staff see only their zone
3. **Audit logging:** All changes tracked in activity log
4. **Export controls:** Limited export permissions

## Best Practices

1. **Data validation:** Use single select fields where possible
2. **Naming conventions:** Consistent naming across all tables
3. **Backup:** Weekly base backups
4. **Training:** Document all processes for new users

---

*This structure supports 100-500 monthly permits with room for scaling.*
*Total setup time: 2-3 hours for experienced Airtable user.*