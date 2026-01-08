# Property Management System - Domain Model Analysis

## Bounded Context: Tenant Management

### Entities
- **Tenant**
  - Tenant ID
  - Name
  - Phone Number
  - Email
  - Account Number
  - Status (Active, Former)
  - Documents Collection
  - Payment History

- **Guarantor**
  - Guarantor ID
  - Name
  - Contact Information
  - Relationship to Tenant

### Value Objects
- **TenantContact**
  - Phone Number
  - Email Address
  - Preferred Communication Channel

- **TenantStatus**
  - Status Type (Active, Former, Pending)
  - Effective Date

### Domain Events
- **TenantAccountActivated**
  - Tenant ID
  - Activation Date
  - Unit Assigned

- **TenantMovedOut**
  - Tenant ID
  - Move-Out Date
  - Final Balance

- **TenantDocumentUploaded**
  - Tenant ID
  - Document Type
  - Upload Timestamp

### Domain Logic
- Tenant account can only be activated after lease is signed
- Former tenants retain access to historical documents but not portal
- Tenant account number must be unique within property
- Tenant must provide valid identification documents

---

## Bounded Context: Lease Management

### Entities
- **Lease**
  - Lease ID
  - Tenant Reference
  - Unit Reference
  - Start Date
  - End Date
  - Monthly Rent Amount
  - Deposit Amount
  - Rent Due Day
  - Status (Draft, Pending Signature, Partially Signed, Active, Expired)
  - Signatures Collection
  - Terms and Conditions

- **LeaseRenewal**
  - Renewal ID
  - Original Lease Reference
  - Proposed Start Date
  - Proposed End Date
  - New Rent Amount
  - Status

- **Signature**
  - Signature ID
  - Signer Type (Tenant, Guarantor, Landlord)
  - Signature Data
  - Signed Date
  - IP Address

### Value Objects
- **LeaseTerm**
  - Duration in Months
  - Start Date
  - End Date

- **RentSchedule**
  - Initial Rent Amount
  - Annual Increase Percentage
  - Rent Adjustments by Year

- **SigningLink**
  - URL
  - Expiration Date
  - Token
  - Used Status

- **UtilityTerms**
  - Water Billing Method
  - Water Rate
  - Garbage Collection Fee
  - Electricity Arrangement

### Domain Events
- **LeaseCreated**
  - Lease ID
  - Tenant ID
  - Unit ID
  - Creation Timestamp

- **LeaseSigningInvitationSent**
  - Lease ID
  - Recipient Email
  - Signing Link
  - Expiration Date

- **LeaseSigned**
  - Lease ID
  - Signer ID
  - Signed Date

- **LeaseActivated**
  - Lease ID
  - Activation Date
  - All Signatories

- **LeaseExpirationApproaching**
  - Lease ID
  - Days Until Expiration
  - Notification Date

- **LeaseExpired**
  - Lease ID
  - Expiration Date

- **LeaseRenewalInitiated**
  - Original Lease ID
  - Renewal ID

### Domain Logic
- Lease cannot be activated until all required signatures collected
- Signing link expires after 7 days
- Lease with annual increase automatically adjusts rent after 12 months
- Renewal notifications triggered 60 days before expiration
- Unit status changes to Occupied only when lease is Active
- When lease expires, tenant becomes Former Tenant

---

## Bounded Context: Property & Unit Management

### Entities
- **Property**
  - Property ID
  - Property Name
  - Address
  - Owner/Landlord Reference
  - Total Units Count
  - Units Collection
  - Policy Settings

- **Unit**
  - Unit ID
  - Unit Number
  - Property Reference
  - Unit Type
  - Size (Square Meters)
  - Bedrooms Count
  - Bathrooms Count
  - Status (Vacant, Occupied)
  - Current Tenant Reference
  - Amenities List
  - Monthly Rent

- **UnitType**
  - Type ID
  - Type Name (Studio, 1-Bedroom, etc.)
  - Default Rent Amount

### Value Objects
- **Address**
  - Street
  - Building
  - City
  - County
  - Postal Code

- **UnitStatus**
  - Status (Vacant, Occupied)
  - Status Change Date
  - Reason

- **OccupancyRate**
  - Total Units
  - Occupied Units
  - Vacant Units
  - Percentage

- **Amenity**
  - Amenity Type
  - Description

### Domain Events
- **UnitMarkedVacant**
  - Unit ID
  - Vacancy Date
  - Previous Tenant ID

- **UnitMarkedOccupied**
  - Unit ID
  - Occupancy Date
  - New Tenant ID

- **UnitListingPublished**
  - Unit ID
  - Listing ID
  - Publication Date

- **UnitListingUnpublished**
  - Unit ID
  - Unpublish Reason

- **OccupancyRateChanged**
  - Property ID
  - Old Rate
  - New Rate
  - Calculation Date

### Domain Logic
- Unit cannot be marked as Occupied without active lease
- When unit marked Vacant, listing should be auto-published
- When unit marked Occupied, listing should be auto-unpublished
- Occupancy rate recalculated whenever unit status changes
- Unit inherits default rent from Unit Type but can be overridden

---

## Bounded Context: Payments & Billing

### Entities
- **Payment**
  - Payment ID
  - Tenant Reference
  - Amount
  - Payment Method
  - Transaction ID
  - Payment Date
  - Status (Pending, Completed, Failed)
  - Receipt Number
  - Allocation Details

- **Invoice**
  - Invoice ID
  - Invoice Number
  - Tenant Reference
  - Unit Reference
  - Billing Period
  - Line Items Collection
  - Total Amount
  - Due Date
  - Status (Draft, Sent, Paid, Overdue)
  - Previous Balance

- **InvoiceLineItem**
  - Description
  - Amount
  - Category (Rent, Water, Garbage, Late Fee)
  - Period Reference

- **Receipt**
  - Receipt Number
  - Payment Reference
  - Issue Date
  - Amount
  - Recipient Information

- **UnallocatedPayment**
  - Payment ID
  - Amount
  - Received Date
  - Reference Provided
  - Allocation Status

### Value Objects
- **Money**
  - Amount
  - Currency (KES)

- **PaymentMethod**
  - Method Type (M-Pesa, Bank Transfer, Cash, Paybill)
  - Account Details

- **PaymentAllocation**
  - Invoice Reference
  - Amount Allocated
  - Allocation Date

- **TransactionReference**
  - Transaction ID
  - External System
  - Timestamp

- **BillingPeriod**
  - Month
  - Year
  - Start Date
  - End Date

### Domain Events
- **PaymentInitiated**
  - Payment ID
  - Tenant ID
  - Amount
  - Method

- **PaymentReceived**
  - Payment ID
  - Transaction ID
  - Amount
  - Timestamp

- **PaymentConfirmed**
  - Payment ID
  - Confirmation Timestamp

- **PaymentFailed**
  - Payment ID
  - Failure Reason
  - Timestamp

- **PaymentAllocated**
  - Payment ID
  - Invoice ID
  - Amount Allocated

- **ReceiptGenerated**
  - Receipt Number
  - Payment ID
  - Generation Timestamp

- **InvoiceGenerated**
  - Invoice ID
  - Tenant ID
  - Total Amount
  - Due Date

- **InvoiceSent**
  - Invoice ID
  - Delivery Channels
  - Sent Timestamp

- **InvoicePaid**
  - Invoice ID
  - Payment ID
  - Paid Date

- **UnallocatedPaymentReceived**
  - Payment ID
  - Amount
  - Invalid Reference

- **UnallocatedPaymentAllocated**
  - Payment ID
  - Tenant ID
  - Allocated By

### Domain Logic
- Payments allocated to oldest outstanding invoice first (FIFO)
- Partial payments reduce balance but don't fully settle invoice
- Payment must be confirmed before balance is updated
- Receipt generated automatically upon payment confirmation
- Unallocated payments require manual intervention
- Invoice includes previous balance carried forward
- Invoice due date calculated from rent due day setting
- Monthly invoices generated automatically for occupied units

---

## Bounded Context: Late Fees & Penalties

### Entities
- **LateFeePolicy**
  - Policy ID
  - Property Reference
  - Grace Period Days
  - Fee Type (Flat, Percentage)
  - Fee Amount or Percentage
  - Active Status

- **LateFeeCharge**
  - Charge ID
  - Tenant Reference
  - Invoice Reference
  - Amount
  - Applied Date
  - Reason

### Value Objects
- **GracePeriod**
  - Days
  - Start Date (Due Date)
  - End Date (Due Date + Days)

- **FeeCalculation**
  - Base Amount
  - Fee Type
  - Fee Rate/Amount
  - Calculated Fee

### Domain Events
- **GracePeriodExpired**
  - Tenant ID
  - Invoice ID
  - Expiry Date

- **LateFeeApplied**
  - Charge ID
  - Tenant ID
  - Amount
  - Application Date

- **LateFeeNotificationSent**
  - Tenant ID
  - Charge Amount
  - Notification Timestamp

### Domain Logic
- Grace period starts from rent due date
- Late fee only applied after grace period expires
- Percentage-based fee calculated on outstanding rent amount
- Flat fee is fixed amount regardless of rent
- No late fee if payment received within grace period
- Tenant notified when late fee is applied

---

## Bounded Context: Utilities Management

### Entities
- **WaterMeterReading**
  - Reading ID
  - Unit Reference
  - Meter Number
  - Previous Reading
  - Current Reading
  - Reading Date
  - Recorded By
  - Consumption Calculated
  - Verification Status

- **WaterBillingConfiguration**
  - Config ID
  - Property Reference
  - Rate per Unit
  - Billing Frequency

- **ConsumptionAlert**
  - Alert ID
  - Unit Reference
  - Reading Reference
  - Unusual Consumption Amount
  - Alert Date
  - Notes

- **GarbageCollectionConfiguration**
  - Config ID
  - Property Reference
  - Monthly Fee
  - Applied To (All Units, Occupied Only)

### Value Objects
- **WaterConsumption**
  - Units Consumed
  - Calculation Date
  - Amount Charged

- **MeterReading**
  - Reading Value
  - Reading Date
  - Reader Identity

- **ConsumptionPattern**
  - Average Monthly Usage
  - Historical Readings
  - Deviation Threshold

### Domain Events
- **MeterReadingRecorded**
  - Reading ID
  - Unit ID
  - Current Reading
  - Timestamp

- **BulkReadingsRecorded**
  - Property ID
  - Number of Readings
  - Recording Date

- **UnusualConsumptionDetected**
  - Unit ID
  - Reading ID
  - Consumption Amount
  - Threshold Exceeded

- **WaterChargeCalculated**
  - Unit ID
  - Consumption
  - Amount
  - Billing Period

- **GarbageFeeConfigure**
  - Property ID
  - Fee Amount
  - Effective Date

### Domain Logic
- Water charge = (Current Reading - Previous Reading) × Rate
- Consumption flagged as unusual if exceeds 150% of 3-month average
- Manager must verify unusual consumption readings
- Water charges added to monthly invoice automatically
- Garbage fee only applied to occupied units
- Previous reading becomes next month's starting point

---

## Bounded Context: Communications & Notifications

### Entities
- **Announcement**
  - Announcement ID
  - Property Reference
  - Subject
  - Message Body
  - Urgency Level
  - Created By
  - Created Date
  - Scheduled Delivery Date
  - Target Recipients
  - Delivery Status

- **Notification**
  - Notification ID
  - Recipient Reference
  - Notification Type
  - Subject
  - Message
  - Channels (SMS, Email, In-App)
  - Scheduled Time
  - Delivery Status
  - Sent Timestamp

- **NotificationTemplate**
  - Template ID
  - Template Type
  - Subject Template
  - Body Template
  - Variables

### Value Objects
- **NotificationChannel**
  - Channel Type (SMS, Email, In-App, WhatsApp)
  - Delivery Status
  - Delivery Timestamp

- **DeliveryReport**
  - Total Recipients
  - Successful Deliveries
  - Failed Deliveries
  - Pending Deliveries

- **RecipientList**
  - Tenant IDs
  - Selection Criteria
  - Total Count

### Domain Events
- **AnnouncementCreated**
  - Announcement ID
  - Creator ID
  - Creation Date

- **AnnouncementScheduled**
  - Announcement ID
  - Delivery Date
  - Recipient Count

- **AnnouncementSent**
  - Announcement ID
  - Sent Date
  - Delivery Report

- **NotificationSent**
  - Notification ID
  - Recipient ID
  - Channel
  - Sent Timestamp

- **NotificationDelivered**
  - Notification ID
  - Delivery Timestamp

- **NotificationFailed**
  - Notification ID
  - Failure Reason

- **ReminderSent**
  - Tenant ID
  - Reminder Type
  - Amount Due
  - Sent Timestamp

- **PaymentConfirmationSent**
  - Payment ID
  - Tenant ID
  - Channels Used

### Domain Logic
- Announcement can be scheduled for future delivery
- Targeted announcements only sent to specified units/buildings
- Scheduled announcements can be edited/cancelled before delivery time
- Multi-channel notifications sent simultaneously
- Payment confirmations sent within 30 seconds of confirmation
- Reminders not sent if payment already received
- Escalating reminder tone based on days overdue

---

## Bounded Context: Reminder & Alert Management

### Entities
- **ReminderPolicy**
  - Policy ID
  - Property Reference
  - Reminder Type (Rent Due, Late Payment, Lease Expiry)
  - Days Before/After Trigger
  - Channels
  - Message Template
  - Active Status

- **ReminderSchedule**
  - Schedule ID
  - Tenant Reference
  - Reminder Type
  - Scheduled Date
  - Status (Pending, Sent, Cancelled)
  - Cancellation Reason

- **EscalationPolicy**
  - Policy ID
  - Property Reference
  - Escalation Stages
  - Days Between Stages

- **EscalationStage**
  - Stage Number
  - Days Overdue
  - Reminder Type (Friendly, Formal, Demand)
  - Message Template

### Value Objects
- **ReminderTrigger**
  - Trigger Type
  - Calculation Base (Due Date, Payment Date)
  - Days Offset

- **ReminderStatus**
  - Status Type
  - Status Date
  - Reason

### Domain Events
- **ReminderScheduled**
  - Schedule ID
  - Tenant ID
  - Trigger Date

- **ReminderSent**
  - Schedule ID
  - Sent Date
  - Channels Used

- **ReminderCancelled**
  - Schedule ID
  - Cancellation Reason
  - Cancelled Date

- **EscalationTriggered**
  - Tenant ID
  - Stage Number
  - Trigger Date

### Domain Logic
- Reminders automatically scheduled based on policy settings
- Reminder cancelled if payment received before send time
- Escalation stages triggered at configured day intervals
- Reminder tone escalates with each stage
- Lease expiry reminders sent 60 days before expiration
- Rent due reminders sent 5 days before due date

---

## Bounded Context: Document Management

### Entities
- **Document**
  - Document ID
  - Document Type
  - File Name
  - File Size
  - Upload Date
  - Uploaded By
  - Associated Entity Type (Tenant, Unit, Lease)
  - Associated Entity ID
  - Storage Location
  - Encryption Status
  - Access Level

- **DocumentType**
  - Type ID
  - Type Name (ID, Lease, Inspection, Notice)
  - Required For
  - Retention Period

### Value Objects
- **DocumentMetadata**
  - File Name
  - File Type
  - File Size
  - Upload Timestamp

- **DocumentAccessControl**
  - Access Level (Public, Private, Manager Only)
  - Allowed Roles
  - Expiry Date

- **DocumentTag**
  - Tag Name
  - Tag Category

### Domain Events
- **DocumentUploaded**
  - Document ID
  - Uploaded By
  - Upload Timestamp

- **DocumentAccessed**
  - Document ID
  - Accessed By
  - Access Timestamp

- **DocumentDeleted**
  - Document ID
  - Deleted By
  - Deletion Timestamp

- **SignedLeaseStored**
  - Lease ID
  - Document ID
  - Storage Timestamp

### Domain Logic
- Documents encrypted at rest
- Tenants can only access their own documents
- Historical documents retained after tenant moves out
- Documents tagged with type for filtering
- Signed leases automatically stored when finalized
- Move-in inspection reports linked to both tenant and unit

---

## Bounded Context: Listing & Marketing

### Entities
- **Listing**
  - Listing ID
  - Unit Reference
  - Title
  - Description
  - Monthly Rent
  - Bedrooms
  - Bathrooms
  - Size
  - Amenities
  - Available From Date
  - Photos Collection
  - Status (Draft, Published, Unpublished)
  - Publication Date
  - Views Count

- **ListingPhoto**
  - Photo ID
  - Listing Reference
  - Image URL
  - Display Order
  - Upload Date

### Value Objects
- **ListingStatus**
  - Status Type
  - Status Change Date

- **PropertyFeatures**
  - Bedrooms
  - Bathrooms
  - Size in SqM
  - Parking Spaces
  - Amenities List

### Domain Events
- **ListingCreated**
  - Listing ID
  - Unit ID
  - Creation Date

- **ListingPublished**
  - Listing ID
  - Publication Date

- **ListingUnpublished**
  - Listing ID
  - Unpublish Reason
  - Unpublish Date

- **ListingViewed**
  - Listing ID
  - View Timestamp
  - Viewer Location

### Domain Logic
- Listing auto-published when unit marked vacant
- Listing auto-unpublished when unit marked occupied
- Listing requires minimum information before publishing
- Photos ordered by display order preference
- Listing appears on public website within 5 minutes of publishing

---

## Bounded Context: Financial Reporting & Analytics

### Entities
- **FinancialStatement**
  - Statement ID
  - Landlord Reference
  - Property Reference
  - Period
  - Rent Collected
  - Expenses
  - Management Commission
  - Net Income
  - Generated Date

- **Expense**
  - Expense ID
  - Property Reference
  - Category
  - Amount
  - Description
  - Date
  - Receipt/Invoice Reference

- **ManagementCommission**
  - Commission ID
  - Property Reference
  - Period
  - Rent Collected Amount
  - Commission Rate
  - Commission Amount

### Value Objects
- **FinancialPeriod**
  - Month
  - Year
  - Start Date
  - End Date

- **ExpenseCategory**
  - Category Name
  - Category Code

- **CommissionCalculation**
  - Base Amount
  - Commission Rate
  - Calculated Amount

### Domain Events
- **MonthlyStatementGenerated**
  - Statement ID
  - Landlord ID
  - Period
  - Generation Date

- **StatementSent**
  - Statement ID
  - Sent Date
  - Delivery Channel

- **ExpenseRecorded**
  - Expense ID
  - Property ID
  - Amount
  - Recording Date

- **CommissionCalculated**
  - Commission ID
  - Amount
  - Calculation Date

### Domain Logic
- Management commission calculated as percentage of rent collected
- Expenses reduce net income in landlord statement
- Statements generated automatically at month end
- Landlord receives statement via email automatically
- Expenses categorized for reporting purposes

---

## Bounded Context: Maintenance Management

### Entities
- **MaintenanceRequest**
  - Request ID
  - Tenant Reference
  - Unit Reference
  - Issue Description
  - Photos Collection
  - Priority
  - Status (Pending, In Progress, Completed)
  - Reported Date
  - Assigned To
  - Resolution Notes
  - Completed Date

- **MaintenancePhoto**
  - Photo ID
  - Request Reference
  - Image URL
  - Upload Timestamp

### Value Objects
- **RequestStatus**
  - Status Type
  - Status Change Date
  - Changed By

- **Priority**
  - Priority Level (Low, Medium, High, Emergency)

### Domain Events
- **MaintenanceRequestCreated**
  - Request ID
  - Tenant ID
  - Unit ID
  - Creation Timestamp

- **MaintenanceRequestAssigned**
  - Request ID
  - Assigned To
  - Assignment Date

- **MaintenanceRequestStatusChanged**
  - Request ID
  - Old Status
  - New Status
  - Change Date

- **MaintenanceRequestCompleted**
  - Request ID
  - Completion Date
  - Resolution Notes

### Domain Logic
- Tenants can report issues through app or WhatsApp
- Issues can include photos for clarity
- Property manager assigns requests to contractors
- Tenants can track status of their requests
- High priority requests flagged for immediate attention

---

## Bounded Context: User & Access Management

### Entities
- **User**
  - User ID
  - User Type (Tenant, Property Manager, Landlord, Staff)
  - Name
  - Email
  - Phone
  - Roles Collection
  - Permissions
  - Status (Active, Inactive)
  - Last Login

- **Role**
  - Role ID
  - Role Name
  - Permissions Collection

- **Permission**
  - Permission ID
  - Resource
  - Action
  - Scope

### Value Objects
- **UserCredentials**
  - Email
  - Password Hash
  - Two-Factor Enabled

- **AccessLevel**
  - Level Type (Read-Only, Full Access, Admin)

### Domain Events
- **UserCreated**
  - User ID
  - User Type
  - Creation Date

- **UserLoggedIn**
  - User ID
  - Login Timestamp
  - IP Address

- **RoleAssigned**
  - User ID
  - Role ID
  - Assignment Date

- **AccessGranted**
  - User ID
  - Resource
  - Granted Date

- **AccessRevoked**
  - User ID
  - Resource
  - Revoked Date

### Domain Logic
- Landlords have read-only access to their properties
- Property managers can create staff accounts with specific permissions
- Tenants can only access their own data
- Former tenants lose portal access but historical documents retained
- Staff permissions scoped to specific properties

---

## Bounded Context: Integration & External Systems

### Entities
- **PaymentIntegration**
  - Integration ID
  - Integration Type (M-Pesa, Bank, Paybill)
  - Configuration Details
  - Active Status

- **AccountingIntegration**
  - Integration ID
  - System Type (QuickBooks, Xero)
  - Sync Configuration
  - Last Sync Date

- **TaxComplianceIntegration**
  - Integration ID
  - System Type (KRA ETIMS)
  - Configuration
  - Compliance Status

### Value Objects
- **IntegrationConfiguration**
  - API Credentials
  - Endpoint URLs
  - Sync Frequency

- **SyncStatus**
  - Last Sync Date
  - Records Synced
  - Errors

### Domain Events
- **PaymentIntegrationConfigured**
  - Integration ID
  - Configuration Date

- **PaymentReceivedViaIntegration**
  - Payment ID
  - Integration Type
  - Transaction Details

- **TransactionSyncedToAccounting**
  - Transaction ID
  - Accounting System
  - Sync Timestamp

- **ETIMSInvoiceSubmitted**
  - Invoice ID
  - Submission Timestamp
  - ETIMS Response

### Domain Logic
- Paybill payments automatically matched using account number
- Bank transfers matched using reference
- Unmatched payments queued for manual allocation
- QuickBooks sync happens automatically for all transactions
- ETIMS invoices submitted automatically when payment received
- Integration failures logged and retry attempted

---

## Cross-Cutting Concerns

### Value Objects (Shared)
- **DateRange**
  - Start Date
  - End Date

- **PhoneNumber**
  - Country Code
  - Number
  - Formatted Display

- **EmailAddress**
  - Email String
  - Validated Status

- **Currency**
  - Code (KES)
  - Symbol
  - Decimal Places

### Domain Events (Shared)
- **AuditLogCreated**
  - User ID
  - Action
  - Entity Type
  - Entity ID
  - Timestamp
  - IP Address

### Common Domain Logic
- All monetary amounts in KES
- All dates in Africa/Nairobi timezone
- Audit trail for all critical operations
- Data encrypted at rest and in transit
- Multi-tenant data isolation enforced