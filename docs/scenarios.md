Feature: Tenant Rent Balance and Payment History

  Scenario: Tenant views current rent balance with no outstanding payments
    Given I am logged in as tenant "John Doe" for unit "A101"
    And my rent is 25000 KES per month
    And my rent is due on the 5th of each month
    And I have paid all rent up to the current month
    When I navigate to my rent balance page
    Then I should see my current balance as 0 KES
    And I should see my next payment due date as "5th January 2026"
    And I should see my next payment amount as 25000 KES

  Scenario: Tenant views balance with one month overdue
    Given I am logged in as tenant "Jane Smith" for unit "B203"
    And my monthly rent is 30000 KES
    And my rent is due on the 1st of each month
    And today is "15th January 2026"
    And I have not paid December 2025 rent
    And I have not paid January 2026 rent
    When I navigate to my rent balance page
    Then I should see my current balance as 60000 KES
    And I should see "2 months overdue"
    And I should see a breakdown showing:
      | Period         | Amount    | Status  |
      | December 2025  | 30000 KES | Overdue |
      | January 2026   | 30000 KES | Due     |

  Scenario: Tenant views payment history for the last 6 months
    Given I am logged in as tenant "Peter Kamau"
    And I have the following payment history:
      | Date       | Amount    | Method  | Receipt Number |
      | 2025-12-03 | 25000 KES | M-Pesa  | RCT-2025-1203  |
      | 2025-11-05 | 25000 KES | M-Pesa  | RCT-2025-1105  |
      | 2025-10-04 | 25000 KES | M-Pesa  | RCT-2025-1004  |
      | 2025-09-02 | 25000 KES | Bank    | RCT-2025-0902  |
      | 2025-08-06 | 25000 KES | M-Pesa  | RCT-2025-0806  |
      | 2025-07-03 | 25000 KES | Cash    | RCT-2025-0703  |
    When I view my payment history
    Then I should see all 6 payments listed
    And each payment should show the date, amount, payment method, and receipt number

---

Feature: Mobile Money Rent Payment

  Scenario: Tenant initiates M-Pesa rent payment
    Given I am logged in as tenant "Mary Wanjiku"
    And my current rent balance is 28000 KES
    And my M-Pesa number is "0722123456"
    When I select "Pay Rent" option
    And I choose "M-Pesa" as payment method
    And I enter payment amount of 28000 KES
    And I confirm the payment
    Then I should receive an M-Pesa STK push to my phone
    And I should see "Waiting for payment confirmation" message

  Scenario: Successful M-Pesa payment updates balance immediately
    Given I am tenant "James Omondi" with balance 35000 KES
    And I have initiated an M-Pesa payment of 35000 KES
    When the M-Pesa payment is confirmed with transaction ID "QA12BC3456"
    Then my rent balance should be updated to 0 KES
    And a payment record should be created with:
      | Transaction ID | QA12BC3456   |
      | Amount         | 35000 KES    |
      | Method         | M-Pesa       |
      | Status         | Completed    |
    And I should receive a payment confirmation SMS
    And I should receive a payment receipt via email

  Scenario: Partial rent payment
    Given I am tenant "Sarah Njeri" with monthly rent 40000 KES
    And my current balance is 80000 KES for 2 months
    When I pay 40000 KES via M-Pesa
    Then my balance should be reduced to 40000 KES
    And the payment should be allocated to the oldest outstanding month first
    And I should see 1 month still outstanding

  Scenario: M-Pesa payment fails due to insufficient funds
    Given I am tenant "David Mutua"
    And I initiate a payment of 30000 KES
    When the M-Pesa transaction fails with error "Insufficient funds"
    Then my rent balance should remain unchanged
    And I should see error message "Payment failed: Insufficient funds in M-Pesa account"
    And no payment record should be created

---

Feature: Automatic Payment Recording and Receipting

  Scenario: Paybill payment automatically recorded
    Given property manager "Alice Mwangi" has set up M-Pesa paybill integration
    And tenant "Tom Otieno" with account number "A101" owes 25000 KES
    When a payment of 25000 KES is received on the paybill
    And the account number "A101" is provided
    Then the system should automatically:
      | Action                        | Detail                           |
      | Create payment record         | Amount: 25000 KES, Method: Paybill |
      | Update tenant balance         | New balance: 0 KES               |
      | Generate receipt              | Receipt number: auto-generated   |
      | Send SMS to tenant            | With receipt number and amount   |
      | Send email to tenant          | With PDF receipt attached        |
      | Notify property manager       | New payment received             |

  Scenario: Bank transfer automatically matched to tenant
    Given property manager has integrated bank account "0123456789"
    And tenant "Lucy Akinyi" account number is "B205"
    And tenant reference format is configured as account number
    When a bank transfer of 32000 KES is received
    And the transfer reference is "B205"
    Then the system should match the payment to Lucy Akinyi
    And create a payment record automatically
    And send confirmation to the tenant

  Scenario: Payment received without valid account reference
    Given a payment of 20000 KES is received via paybill
    And the account number provided is "X999" which doesn't exist
    Then the payment should be marked as "Unallocated"
    And property manager should be notified of unallocated payment
    And the payment should appear in "Pending Allocation" queue
    And property manager should be able to manually assign it to correct tenant

---

Feature: Automatic Late Fee Calculation

  Scenario: Late fee applied after grace period expires
    Given property "Sunset Apartments" has late fee policy:
      | Grace Period  | 5 days       |
      | Late Fee Type | Flat         |
      | Late Fee      | 500 KES      |
    And tenant "Michael Kariuki" in unit "C301" has rent due on "1st January 2026"
    And monthly rent is 28000 KES
    And today is "7th January 2026"
    And the rent has not been paid
    Then a late fee of 500 KES should be automatically added
    And tenant should be notified of the late fee charge
    And new balance should be 28500 KES

  Scenario: Percentage-based late fee calculation
    Given property has late fee policy:
      | Grace Period  | 3 days       |
      | Late Fee Type | Percentage   |
      | Late Fee Rate | 5%           |
    And tenant "Grace Njoki" has rent 30000 KES due on "5th January 2026"
    And today is "9th January 2026"
    And rent remains unpaid
    Then late fee of 1500 KES (5% of 30000) should be charged
    And total balance should be 31500 KES

  Scenario: No late fee within grace period
    Given property has 5-day grace period
    And tenant "Paul Maina" has rent due on "1st January 2026"
    And today is "4th January 2026"
    When the system checks for late fees
    Then no late fee should be applied
    And tenant balance should remain at 25000 KES

---

Feature: Landlord Payment Notifications

  Scenario: Landlord receives notification for tenant payment
    Given landlord "Robert Kibet" owns property "Green Valley Apartments"
    And property has tenant "Ann Wambui" in unit "A102"
    And landlord has notifications enabled
    When tenant Ann Wambui pays 27000 KES
    Then landlord Robert Kibet should receive notification:
      | Channel | Details                                    |
      | Email   | Subject: Payment Received - Unit A102      |
      | SMS     | Ann Wambui paid 27000 KES for unit A102    |
    And notification should include:
      | Tenant Name    | Ann Wambui           |
      | Unit           | A102                 |
      | Amount         | 27000 KES            |
      | Payment Date   | Current date         |
      | Payment Method | M-Pesa               |

  Scenario: Landlord receives daily payment summary
    Given landlord "Susan Chebet" owns multiple units
    And has enabled daily summary notifications
    And today three tenants made payments:
      | Tenant          | Unit | Amount    |
      | Joseph Kimani   | B101 | 30000 KES |
      | Faith Auma      | B102 | 28000 KES |
      | Daniel Odhiambo | B105 | 32000 KES |
    When the daily summary is generated at 6:00 PM
    Then landlord should receive one summary notification
    And it should show total received: 90000 KES
    And it should list all three payments

---

Feature: Digital Lease Agreement Creation

  Scenario: Property manager creates new lease for tenant
    Given I am logged in as property manager "Patrick Ngugi"
    And unit "D204" is vacant
    And I am on the lease creation page
    When I enter the following lease details:
      | Tenant Name       | Alice Moraa          |
      | Tenant Phone      | 0733445566           |
      | Tenant Email      | alice@email.com      |
      | Unit              | D204                 |
      | Monthly Rent      | 35000 KES            |
      | Deposit           | 70000 KES            |
      | Lease Start Date  | 1st February 2026    |
      | Lease End Date    | 31st January 2027    |
      | Rent Due Day      | 1st of each month    |
    And I upload tenant ID document
    And I click "Create Lease"
    Then a new lease agreement should be created
    And lease status should be "Pending Signature"
    And tenant Alice Moraa should receive email invitation to sign
    And unit D204 status should remain "Vacant" until lease is signed

  Scenario: Lease includes utility billing terms
    Given I am creating a lease for unit "E105"
    When I configure lease terms:
      | Base Rent              | 40000 KES          |
      | Water Billing          | Per meter reading  |
      | Water Rate             | 50 KES per unit    |
      | Garbage Collection Fee | 500 KES per month  |
      | Electricity            | Direct to KPLC     |
    Then the lease document should include all utility terms
    And tenant invoices should automatically include configured charges

  Scenario: Multi-year lease with annual rent increase
    Given I am creating a 2-year lease
    When I set:
      | Initial Rent        | 45000 KES |
      | Annual Increase     | 7%        |
      | Lease Duration      | 24 months |
    Then the lease should show:
      | Year 1 Monthly Rent | 45000 KES |
      | Year 2 Monthly Rent | 48150 KES |
    And system should automatically adjust rent after 12 months

---

Feature: Electronic Lease Signing

  Scenario: Tenant signs lease via mobile phone
    Given I am tenant "Brian Wekesa"
    And I have received a lease signing invitation via email
    And the lease is for unit "F301" at 32000 KES per month
    When I click the signing link in the email
    And I review the lease terms
    And I check the "I agree to terms" checkbox
    And I draw my signature on the signature pad
    And I click "Submit Signature"
    Then my signature should be captured and attached to the lease
    And lease status should change to "Active"
    And unit F301 should be marked as "Occupied"
    And my tenant account should be activated
    And I should receive a copy of the signed lease via email

  Scenario: Lease requires both tenant and guarantor signatures
    Given lease for "George Kiprop" requires guarantor
    And guarantor is "Samuel Korir"
    When George signs the lease
    Then lease status should be "Partially Signed"
    And guarantor Samuel should receive signing invitation
    When Samuel signs the lease
    Then lease status should change to "Active"
    And both signatures should be visible on the document

  Scenario: Lease signing link expires after 7 days
    Given tenant "Christine Adhiambo" received signing invitation on "1st January 2026"
    And today is "9th January 2026"
    When Christine clicks the signing link
    Then she should see "Signing link expired" message
    And property manager should be notified
    And property manager should be able to resend signing invitation

---

Feature: Automatic Lease Renewal Notifications

  Scenario: Manager notified 60 days before lease expiration
    Given property manager "Emily Wangari" manages "Maple Heights"
    And tenant "Kevin Omondi" has lease ending on "31st March 2026"
    And today is "30th January 2026" (60 days before expiration)
    When the daily renewal check runs
    Then manager should receive notification:
      | Title   | Lease Expiring Soon - Unit G102           |
      | Message | Kevin Omondi's lease expires in 60 days   |
      | Action  | Start renewal process                     |
    And tenant should appear in "Leases Expiring Soon" dashboard section

  Scenario: Multiple leases expiring in same period
    Given property manager has 5 leases expiring in next 60 days:
      | Tenant         | Unit | Expiry Date  | Days Until |
      | John Kibet     | A201 | 15-Feb-2026  | 39         |
      | Mary Njeri     | A305 | 28-Feb-2026  | 52         |
      | Peter Kamau    | B102 | 10-Mar-2026  | 62         |
      | Grace Akinyi   | B204 | 15-Mar-2026  | 67         |
      | James Otieno   | C101 | 20-Mar-2026  | 72         |
    When viewing the lease renewal dashboard
    Then all 5 leases should be listed
    And they should be sorted by expiry date (earliest first)
    And each should show days remaining

  Scenario: Tenant receives renewal reminder
    Given my lease expires on "30th April 2026"
    And today is "1st March 2026" (60 days before)
    When the renewal reminder is sent
    Then I should receive email with subject "Your Lease is Expiring Soon"
    And SMS saying "Your lease for unit H103 expires in 60 days. Contact office to renew"
    And I should see renewal reminder in my tenant portal

---

Feature: Document Storage

  Scenario: Manager uploads tenant identification documents
    Given I am managing lease for tenant "Patricia Wanjiru"
    When I upload the following documents:
      | Document Type | File Name        | Size    |
      | National ID   | id_front.jpg     | 1.2 MB  |
      | National ID   | id_back.jpg      | 1.1 MB  |
    Then documents should be saved to Patricia's tenant profile
    And I should be able to view them anytime
    And documents should be encrypted at rest

  Scenario: Manager stores signed lease agreement
    Given tenant "Samuel Mutiso" has signed the lease electronically
    When the lease is finalized
    Then the signed PDF should be automatically stored
    And it should be linked to both tenant profile and unit record
    And both manager and tenant should be able to download it

  Scenario: Manager uploads property inspection report
    Given unit "J205" requires move-in inspection
    When I upload inspection report:
      | File Name             | inspection_j205.pdf |
      | Document Type         | Move-in Inspection  |
      | Associated Tenant     | Rebecca Anyango     |
      | Associated Unit       | J205                |
      | Upload Date           | Current date        |
    Then document should be searchable by tenant name or unit number
    And it should be tagged with document type for filtering

  Scenario: Document retention and access control
    Given tenant "David Njoroge" moved out on "31st December 2025"
    And his lease and documents are stored in the system
    When I search for David's documents
    Then I should still be able to access all historical documents
    And documents should be marked as "Historical" or "Archived"
    But David should no longer have access to the tenant portal

---

Feature: Tenant Lease Document Access

  Scenario: Tenant views current lease agreement
    Given I am logged in as tenant "Monica Achieng"
    And I have an active lease for unit "K108"
    When I navigate to "My Lease" section
    Then I should see my current lease details:
      | Lease Start    | 1st September 2025   |
      | Lease End      | 31st August 2026     |
      | Monthly Rent   | 29000 KES            |
      | Deposit Paid   | 58000 KES            |
      | Rent Due Date  | 5th of each month    |
    And I should be able to download the signed lease PDF
    And I should see my landlord's contact information

  Scenario: Tenant cannot access other tenants' documents
    Given I am tenant "Eric Mwangi" in unit "K108"
    When I attempt to access documents for unit "K109"
    Then I should receive "Access Denied" error
    And I should only see my own unit's documents

  Scenario: Tenant views move-in inspection report
    Given I moved into unit "L102" on "1st January 2026"
    And move-in inspection was completed
    When I access "Documents" section
    Then I should see the move-in inspection report
    And I should be able to download it for my records

---

Feature: Unit Vacancy and Occupancy Tracking

  Scenario: Manager marks unit as vacant
    Given unit "M201" has tenant "Andrew Kimutai"
    And tenant's lease ended on "31st December 2025"
    When I navigate to unit M201
    And I click "Mark as Vacant"
    And I enter move-out date as "31st December 2025"
    And I confirm the action
    Then unit status should change to "Vacant"
    And tenant Andrew Kimutai should be marked as "Former Tenant"
    And unit should appear in "Available Units" list
    And unit should be published on listings website

  Scenario: Manager tracks occupancy rate across property
    Given property "Riverside Apartments" has 20 units
    And current occupancy is:
      | Status    | Count |
      | Occupied  | 17    |
      | Vacant    | 3     |
    When I view property dashboard
    Then I should see occupancy rate calculated as 85%
    And I should see 3 vacant units listed
    And I should see year-over-year occupancy trend

  Scenario: Unit becomes occupied after lease signing
    Given unit "N305" is currently vacant
    And tenant "Lydia Wafula" signs lease for unit N305
    When the lease is activated
    Then unit status should automatically change to "Occupied"
    And unit should be removed from "Available Units" list
    And unit should be unpublished from listings website
    And occupancy rate should be recalculated

---

Feature: Vacant Unit Listings

  Scenario: Manager creates listing for vacant unit
    Given unit "P102" is vacant
    When I create a listing with:
      | Monthly Rent    | 38000 KES                    |
      | Bedrooms        | 2                            |
      | Bathrooms       | 2                            |
      | Size            | 95 sqm                       |
      | Amenities       | Parking, Security, Balcony   |
      | Available From  | 1st February 2026            |
    And I upload 5 photos of the unit
    Then listing should be saved
    And status should be "Draft"

  Scenario: Manager publishes listing
    Given I have created a listing for unit "P102"
    And listing has all required information and photos
    When I click "Publish Listing"
    Then listing status should change to "Published"
    And unit should appear on public website within 5 minutes
    And listing should show as active in my listings dashboard

  Scenario: Manager sets different rent for different unit types
    Given property "Skyline Towers" has mixed unit types
    When I set pricing:
      | Unit Type       | Monthly Rent |
      | Studio          | 22000 KES    |
      | 1-Bedroom       | 32000 KES    |
      | 2-Bedroom       | 45000 KES    |
      | 3-Bedroom       | 65000 KES    |
      | Penthouse       | 120000 KES   |
    Then each vacant unit should inherit pricing based on its type
    And I should be able to override pricing for specific units

---

Feature: Water Meter Reading and Billing

  Scenario: Manager records monthly water meter reading
    Given it is "30th January 2026"
    And unit "Q203" has tenant "Francis Onyango"
    And previous meter reading was 1250 units on "31st December 2025"
    When I navigate to water billing section
    And I enter new meter reading of 1320 units
    And water rate is 50 KES per unit
    Then consumption should be calculated as 70 units
    And water charge should be calculated as 3500 KES
    And reading should be saved with date "30th January 2026"

  Scenario: Manager records readings for entire property
    Given property "Acacia Apartments" has 15 units
    And I am on bulk water reading entry page
    When I enter readings for all units:
      | Unit | Previous | Current | Consumption |
      | A101 | 1200     | 1265    | 65         |
      | A102 | 1450     | 1520    | 70         |
      | A103 | 1100     | 1155    | 55         |
      # ... continuing for all 15 units
    And I click "Save All Readings"
    Then all 15 readings should be saved
    And water charges should be calculated for all units
    And readings should be available for next month's invoice

  Scenario: System flags unusual water consumption
    Given unit "R104" typically uses 50-70 units per month
    And previous 3 months average is 60 units
    When I enter current reading showing consumption of 250 units
    Then system should display warning: "Unusually high consumption detected"
    And I should be prompted to verify the reading
    And I should be able to add notes about the high usage (e.g., "Leak reported")

---

Feature: Tenant Water Bill Viewing

  Scenario: Tenant views current month water consumption
    Given I am tenant "Caroline Njambi" in unit "S105"
    And my January 2026 water meter readings are:
      | Previous Reading | 1350 units      |
      | Current Reading  | 1410 units      |
      | Consumption      | 60 units        |
      | Rate             | 50 KES per unit |
      | Charge           | 3000 KES        |
    When I view my utility bills
    Then I should see January water charge of 3000 KES
    And I should see the consumption breakdown
    And I should see both meter readings with dates

  Scenario: Tenant views water consumption history
    Given I have lived in my unit for 6 months
    When I view water consumption history
    Then I should see a chart showing monthly consumption:
      | Month      | Units | Charge    |
      | January    | 60    | 3000 KES  |
      | December   | 55    | 2750 KES  |
      | November   | 58    | 2900 KES  |
      | October    | 62    | 3100 KES  |
      | September  | 57    | 2850 KES  |
      | August     | 59    | 2950 KES  |
    And I should see average monthly consumption
    And I should be able to compare with property average

---

Feature: Garbage Collection Fee Management

  Scenario: Property manager sets up garbage collection charges
    Given property "Westlands Residences" has garbage collection service
    When I configure garbage collection:
      | Charge Type     | Fixed Monthly        |
      | Amount          | 500 KES              |
      | Applied To      | All occupied units   |
      | Billing Cycle   | Monthly              |
    Then the configuration should be saved
    And all occupied units should be flagged for garbage fee

  Scenario: Garbage fee automatically added to tenant invoice
    Given tenant "Isaac Ouma" in unit "T207"
    And property has 500 KES monthly garbage fee
    And it is time to generate February 2026 invoice
    When invoice is generated
    Then invoice should include:
      | Rent              | 35000 KES |
      | Garbage Collection| 500 KES   |
      | Total             | 35500 KES |

  Scenario: Garbage fee not charged for vacant units
    Given property has 500 KES monthly garbage fee
    And unit "T208" is vacant
    When monthly invoices are generated
    Then unit T208 should not receive garbage collection charge
    And no invoice should be generated for vacant unit

---

Feature: Consolidated Monthly Invoice Generation

  Scenario: System generates combined invoice for tenant
    Given it is "30th January 2026"
    And tenant "Helen Mutuku" in unit "U101" has:
      | Monthly Rent       | 32000 KES |
      | Water Consumption  | 65 units  |
      | Water Rate         | 50 KES    |
      | Garbage Collection | 500 KES   |
    When February 2026 invoice is generated
    Then invoice should show:
      | Description          | Amount    |
      | Rent - February 2026 | 32000 KES |
      | Water - January 2026 | 3250 KES  |
      | Garbage Collection   | 500 KES   |
      | Total Amount Due     | 35750 KES |
    And invoice should show due date as "5th February 2026"
    And invoice number should be auto-generated

  Scenario: Invoice includes previous balance carried forward
    Given tenant "Joseph Barasa" has unpaid balance of 15000 KES from previous month
    And current month charges are 33000 KES
    When February invoice is generated
    Then invoice should show:
      | Previous Balance     | 15000 KES |
      | Current Charges      | 33000 KES |
      | Total Amount Due     | 48000 KES |
    And previous balance should be clearly highlighted

  Scenario: Invoice sent automatically via multiple channels
    Given tenant "Nancy Wangui" has:
      | Email | nancy@email.com |
      | Phone | 0744556677      |
    And invoice is generated
    When invoice is finalized
    Then invoice should be automatically sent via:
      | Channel | Content                           |
      | Email   | PDF attachment with full details  |
      | SMS     | "Your Feb rent invoice: 35000 KES"|
    And tenant should see invoice in portal immediately

---

Feature: Rent Payment Reminders

  Scenario: Automatic reminder sent 5 days before due date
    Given today is "27th January 2026"
    And tenant "Bernard Kipchirchir" has rent due on "1st February 2026"
    And amount due is 40000 KES
    And property has reminder settings:
      | Days Before Due Date | 5    |
      | Channels             | SMS, Email, In-App |
    When the daily reminder job runs
    Then Bernard should receive reminder via:
      | SMS   | Your rent of 40000 KES is due on 1st Feb |
      | Email | Detailed invoice and payment options     |
    And reminder should show in tenant portal with "Pay Now" button

  Scenario: No reminder sent if rent already paid
    Given tenant "Esther Muthoni" has rent due on "1st February 2026"
    And reminder is scheduled for "27th January 2026"
    But Esther paid her rent on "25th January 2026"
    When reminder job runs on "27th January 2026"
    Then no reminder should be sent to Esther
    And her reminder should be marked as "Cancelled - Already Paid"

  Scenario: Escalating reminders for overdue rent
    Given tenant "Martin Odongo" has rent overdue since "5th January 2026"
    And property has escalation policy:
      | Days Overdue | Reminder Type    |
      | 5            | Friendly SMS     |
      | 10           | Formal Email     |
      | 15           | Demand Letter    |
    When each milestone is reached
    Then appropriate reminder should be sent
    And tone should escalate with each reminder

---

Feature: Rent Due Date Notifications

  Scenario: Tenant receives notification when rent is due
    Given I am tenant "Ruth Jepkosgei"
    And my rent is due on "1st February 2026"
    And today is "1st February 2026"
    When the due date notification is sent
    Then I should receive SMS: "Your rent of 38000 KES is due today. Pay now to avoid late fees."
    And I should receive email with payment instructions
    And I should see prominent notification in tenant portal

  Scenario: Tenant receives notification on custom due date
    Given tenant "Victor Mwenda" has custom due date of "10th" each month
    And today is "10th February 2026"
    When due date arrives
    Then notification should be sent on the 10th
    And reminder should reference the custom due date

---

Feature: Mass Tenant Announcements

  Scenario: Manager sends announcement to all tenants
    Given I am property manager for "Palm Gardens" with 30 occupied units
    When I compose an announcement:
      | Subject | Water Interruption Notice           |
      | Message | Water will be off tomorrow 9am-5pm  |
      | Urgent  | Yes                                 |
    And I select "Send to All Tenants"
    And I click "Send Now"
    Then all 30 tenants should receive the announcement via:
      | SMS   | Immediate delivery            |
      | Email | Within 5 minutes              |
    And announcement should appear in each tenant's portal
    And delivery report should show successful sends

  Scenario: Manager sends targeted announcement to specific units
    Given property has multiple buildings
    When I create announcement about "Building A elevator maintenance"
    And I select only Building A units:
      | A101, A102, A103, A201, A202, A203 |
    And I send the announcement
    Then only tenants in Building A should receive notification
    And other tenants should not be notified

  Scenario: Manager schedules announcement for future delivery
    Given I want to announce gate access code change
    And change takes effect on "15th February 2026"
    When I create announcement and set delivery date to "14th February 2026 at 8:00 AM"
    Then announcement should be queued
    And it should be sent automatically on "14th February 2026 at 8:00 AM"
    And I should be able to edit or cancel before delivery time

---

Feature: Instant Payment Confirmation

  Scenario: Tenant receives SMS confirmation immediately after payment
    Given I am tenant "Angela Kemunto"
    And I have just completed M-Pesa payment of 28000 KES
    When payment is confirmed by system
    Then within 30 seconds I should receive SMS:
      """
      Payment received! Amount: 28000 KES
      Receipt No: RCT-2026-0107-001
      New Balance: 0 KES
      Thank you for your payment.