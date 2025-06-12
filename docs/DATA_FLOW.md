# Data Flow Architecture

## 🌊 Data Pipeline Overview

### Primary Data Sources
```
EXTERNAL SYSTEMS
├── Shopify → Orders, Customers, Products, Inventory
├── Gmail → Customer inquiries, Supplier communications
├── Meta Business Suite → Ads data, Social interactions
├── AusPost API → Shipping rates, Tracking updates
├── Xero → Financial transactions, Reconciliation
├── Later.com → Social media scheduling, Analytics
└── ChatGPT API → AI-generated content, Analysis

INTERNAL SYSTEMS
├── Airtable → Central database, Business logic
├── Make.com → Automation engine, Data routing
├── Notion → Knowledge base, Executive dashboard
├── Slack → Notifications, Human approvals
└── Google Drive → File storage, Document management
```

## 🔄 Real-Time Data Flows

### Flow #1: Customer Order Journey
```
DATA SOURCE: Shopify Order Created Webhook
TRIGGER: New order placed
FREQUENCY: Real-time
VOLUME: ~50 orders/day

Data Pipeline:
Shopify Order Created
│
├── Make.com Webhook Receiver
│   ├── Order ID: "MC-2024-001234"
│   ├── Customer Email: "customer@email.com"
│   ├── Line Items: [{product_id, quantity, price}]
│   ├── Shipping Address: {address_object}
│   ├── Payment Status: "paid"
│   └── Order Total: $89.95
│
├── Customer Lookup/Creation
│   ├── Search Airtable CUSTOMERS by email
│   ├── Update existing OR create new record
│   ├── Calculate: Total_Orders, Total_Revenue, AOV
│   └── Determine: VIP_Status, Lifecycle_Stage
│
├── Order Record Creation
│   ├── ORDERS table → Master order record
│   ├── ORDER_LINE_ITEMS → Individual product lines
│   ├── Link to CUSTOMERS record
│   └── Calculate: Profit margins, Costs
│
├── Inventory Impact
│   ├── PRODUCTS table → Reduce Current_Stock
│   ├── Calculate: Available_Stock, Days_of_Stock
│   ├── Check: Reorder_Point threshold
│   └── Trigger: Reorder alert if needed
│
├── Financial Metrics Update
│   ├── FINANCIAL_METRICS → Update daily totals
│   ├── Revenue: Add order value
│   ├── Orders_Count: Increment by 1
│   ├── Items_Sold: Add line item quantities
│   └── New_Customers: Increment if new customer
│
├── Fulfillment Initiation
│   ├── AusPost API → Generate shipping label
│   ├── Warehouse notification → Email pick list
│   ├── Customer notification → Order confirmation
│   └── Tracking setup → Monitor shipment
│
└── Analytics & Reporting
    ├── Real-time dashboard update
    ├── Performance metrics calculation
    ├── Trend analysis data points
    └── Executive summary refresh

DATA TRANSFORMATIONS:
├── Currency: Shopify CAD → Airtable AUD
├── Date Format: ISO 8601 → Airtable Date
├── Address: Object → Formatted string
├── Product IDs: Shopify SKU → Internal SKU
└── Customer: Shopify ID → Airtable Record ID

ERROR HANDLING:
├── Duplicate orders → Check Order_ID uniqueness
├── Invalid customer data → Validation rules
├── Stock unavailable → Backorder notification
├── API failures → Retry with exponential backoff
└── Data corruption → Rollback and alert
```

### Flow #2: Customer Inquiry Processing
```
DATA SOURCE: Gmail Push Notification
TRIGGER: New email received
FREQUENCY: Real-time
VOLUME: ~30 inquiries/day

Data Pipeline:
Gmail Push Notification
│
├── Make.com Gmail Module
│   ├── Message ID: "msg_abc123"
│   ├── From: "customer@email.com"
│   ├── Subject: "Question about shampoo"
│   ├── Body: "Hi, I was wondering..."
│   ├── Thread ID: "thread_xyz789"
│   └── Received Time: "2024-06-12T10:30:00Z"
│
├── Customer Identification
│   ├── Search CUSTOMERS table by email
│   ├── Retrieve: Customer history, VIP status
│   ├── Order history: Recent purchases
│   └── Previous inquiries: Past interactions
│
├── AI Content Analysis
│   ├── ChatGPT API → Extract key information
│   │   ├── Intent: "Product information request"
│   │   ├── Product mentioned: "Gentle Shampoo"
│   │   ├── Urgency: "Low"
│   │   └── Sentiment: "+2 (neutral-positive)"
│   │
│   ├── Category Classification
│   │   ├── Product Info: 85% confidence
│   │   ├── Shipping: 10% confidence
│   │   ├── Complaint: 5% confidence
│   │   └── Final: "Product Info"
│   │
│   └── Response Recommendations
│       ├── FAQ match: "Shampoo ingredients"
│       ├── Template: "Product information"
│       ├── Confidence: 92%
│       └── Auto-response: Recommended
│
├── Inquiry Record Creation
│   ├── CUSTOMER_INQUIRIES table → New record
│   ├── Link to CUSTOMERS record
│   ├── Store all AI analysis results
│   ├── Start SLA timer
│   └── Set initial status: "New"
│
├── Response Routing Decision
│   ├── IF Confidence > 90% AND Sentiment > -2
│   │   ├── Route: Automated response
│   │   ├── Generate personalised reply
│   │   ├── Send via Gmail API
│   │   └── Update status: "Resolved"
│   │
│   ├── ELSE IF Urgency = "Medium"
│   │   ├── Route: VA assignment
│   │   ├── Slack notification with context
│   │   ├── 2-hour SLA timer
│   │   └── Update assigned_to: "VA"
│   │
│   └── ELSE (High urgency or negative sentiment)
│       ├── Route: Bella escalation
│       ├── Slack + SMS notification
│       ├── 30-minute SLA timer
│       └── Update assigned_to: "Bella"
│
└── Performance Tracking
    ├── Response time measurement
    ├── Resolution success tracking
    ├── Customer satisfaction survey
    └── Analytics dashboard update

DATA TRANSFORMATIONS:
├── Email Body: HTML → Plain text extraction
├── Sentiment Score: Text → Numerical (-5 to +5)
├── Timestamp: GMT → AEST conversion
├── Category: AI output → Dropdown selection
└── Priority: Algorithm → Human-readable urgency
```

### Flow #3: Inventory Management Sync
```
DATA SOURCE: Multiple sources
TRIGGER: Scheduled + Event-driven
FREQUENCY: Real-time + Daily analysis
VOLUME: ~500 stock movements/day

Data Pipeline:
Inventory Triggers
├── Shopify Order → Stock reduction
├── Manual Adjustment → Stock correction
├── Supplier Delivery → Stock increase
├── Daily Analysis → Reorder calculations
└── Weekly Review → Optimisation analysis
│
├── Stock Level Calculation
│   ├── Current_Stock: Physical inventory
│   ├── Stock_Committed: Pending orders
│   ├── Available_Stock: Current - Committed
│   ├── Stock_Incoming: Pending deliveries
│   └── Net_Available: Available + Incoming
│
├── Sales Velocity Analysis
│   ├── 7-Day Sales: SUM(last 7 days)
│   ├── 30-Day Sales: SUM(last 30 days)
│   ├── 90-Day Sales: SUM(last 90 days)
│   ├── Seasonal Adjustment: Apply multipliers
│   └── Trend Analysis: Growth/decline patterns
│
├── Reorder Point Calculation
│   ├── Lead Time Demand: Avg_Daily_Sales × Lead_Time
│   ├── Safety Stock: 1.5 × Lead_Time_Demand
│   ├── Reorder Point: Lead_Time_Demand + Safety_Stock
│   ├── Economic Order Quantity: √(2DS/H)
│   └── Days of Stock: Available_Stock / Avg_Daily_Sales
│
├── Automated Decision Logic
│   ├── IF Days_of_Stock ≤ Lead_Time_Days
│   │   ├── Generate purchase order
│   │   ├── Calculate optimal quantity
│   │   ├── Select supplier (primary/backup)
│   │   └── Request approval if >$500
│   │
│   ├── IF Available_Stock = 0
│   │   ├── Mark product as out of stock
│   │   ├── Disable Shopify product
│   │   ├── Notify customers on waitlist
│   │   └── Update ETA based on supplier lead time
│   │
│   └── IF Stock > 90_Day_Sales
│       ├── Flag as overstock
│       ├── Suggest promotion campaign
│       ├── Calculate carrying costs
│       └── Recommend clearance strategy
│
├── Supplier Integration
│   ├── PO Generation → PDF creation
│   ├── Email Notification → Supplier contact
│   ├── Delivery Tracking → Expected dates
│   ├── Quality Metrics → Supplier scorecard
│   └── Payment Terms → Financial tracking
│
└── Reporting & Alerts
    ├── Daily Stock Report → Management summary
    ├── Reorder Alerts → Immediate notifications
    ├── Overstock Warnings → Weekly review
    ├── Supplier Performance → Monthly analysis
    └── Inventory Valuation → Financial reports

DATA TRANSFORMATIONS:
├── Stock Levels: Integer → Calculated availability
├── Sales Data: Order history → Velocity metrics
├── Lead Times: Days → Reorder calculations
├── Costs: Multiple currencies → AUD standard
└── Dates: Various formats → Standardised timestamps
```

## 📊 Data Quality & Governance

### Data Validation Rules
```
CUSTOMERS Table:
├── Email: Must be unique, valid format
├── Phone: Australian format validation
├── Names: No special characters, proper case
├── VIP_Status: Auto-calculated, no manual override
└── Total_Revenue: Must equal sum of order values

ORDERS Table:
├── Order_ID: Must be unique, Shopify format
├── Total_Revenue: Must be positive number
├── Order_Date: Cannot be future date
├── Customer: Must link to existing customer
└── Profit_Margin: Must be calculated field

PRODUCTS Table:
├── SKU: Must be unique, alphanumeric only
├── Prices: Must be positive, AUD currency
├── Stock: Cannot be negative
├── Lead_Time: Must be positive integer
└── Supplier: Must link to valid supplier
```

### Data Synchronisation Strategy
```
REAL-TIME SYNC (Immediate):
├── Order creation/updates
├── Customer inquiry receipt
├── Inventory movements
├── Payment confirmations
└── Shipping updates

NEAR REAL-TIME (Within 5 minutes):
├── Customer profile updates
├── Product information changes
├── Pricing modifications
├── Stock adjustments
└── Supplier communications

BATCH SYNC (Hourly):
├── Financial reconciliation
├── Analytics aggregation
├── Performance metrics
├── Report generation
└── Data backup

DAILY SYNC (End of day):
├── Complete data validation
├── Metric calculations
├── Trend analysis
├── System health checks
└── Audit trail generation
```

### Error Handling & Recovery
```
DATA INTEGRITY PROTECTION:
├── Transaction Rollback → Failed operations
├── Duplicate Detection → Prevent data duplication
├── Orphaned Records → Automatic cleanup
├── Consistency Checks → Cross-table validation
└── Backup Verification → Daily integrity tests

ERROR ESCALATION:
├── Level 1: Automatic retry (3 attempts)
├── Level 2: Alternative data source
├── Level 3: Manual intervention alert
├── Level 4: System administrator notification
└── Level 5: Business continuity plan activation

RECOVERY PROCEDURES:
├── Point-in-time recovery → Last known good state
├── Incremental backup → Recent changes only
├── Full system restore → Complete data recovery
├── Data reconstruction → Rebuild from sources
└── Manual data entry → Last resort option
```

## 🔒 Data Security & Privacy

### Security Measures
```
DATA ENCRYPTION:
├── At Rest: AES-256 encryption
├── In Transit: TLS 1.3 minimum
├── API Keys: Encrypted vault storage
├── Database: Enterprise security features
└── Backups: Encrypted archives

ACCESS CONTROL:
├── Role-Based Permissions:
│   ├── Bella: Full administrative access
│   ├── VA: Limited operational access
│   ├── Josh: System configuration access
│   └── Automated Systems: Minimal required access
│
├── Authentication:
│   ├── Multi-factor authentication required
│   ├── API token rotation (90 days)
│   ├── Session timeout (4 hours)
│   └── Failed login monitoring
│
└── Audit Logging:
    ├── All data access logged
    ├── Modification tracking
    ├── Failed access attempts
    └── Regular audit reviews

PRIVACY COMPLIANCE:
├── GDPR Compliance:
│   ├── Right to access data
│   ├── Right to rectification
│   ├── Right to erasure
│   ├── Data portability
│   └── Consent management
│
├── Data Retention:
│   ├── Customer data: 7 years
│   ├── Order history: 7 years
│   ├── Inquiry logs: 3 years
│   ├── Analytics data: 2 years
│   └── System logs: 1 year
│
└── Data Minimisation:
    ├── Collect only necessary data
    ├── Regular data purging
    ├── Anonymisation where possible
    └── Third-party data sharing restrictions
```

This comprehensive data flow architecture ensures reliable, secure, and efficient data management across all business operations while maintaining strict quality and governance standards.