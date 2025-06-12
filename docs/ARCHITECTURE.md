# Core System Architecture

## Executive Dashboard Overview

**Operational Efficiency Target:** 95% automation of routine tasks | **Founder Time Reduction:** 80% | **Response Time SLA:** <2 hours

## 1. CENTRAL DATA HUB (Airtable)

### Database Schema

#### CUSTOMERS TABLE
```
├── Customer_ID (Primary Key - Auto Number)
├── Email (Email - Unique)
├── First_Name (Single Line Text)
├── Last_Name (Single Line Text)
├── Phone (Phone Number)
├── Instagram_Handle (Single Line Text)
├── Pet_Name (Single Line Text)
├── Pet_Breed (Single Select: Cavoodle, Other)
├── Pet_Birthday (Date)
├── Customer_Since (Created Time)
├── VIP_Status (Formula: IF(Total_Revenue>500,'VIP','Standard'))
├── Customer_Tags (Multiple Select)
├── Total_Orders (Rollup from ORDERS)
├── Total_Revenue (Rollup from ORDERS)
├── AOV (Formula: Total_Revenue/Total_Orders)
├── Last_Order_Date (Lookup from ORDERS)
├── Days_Since_Last_Order (Formula)
├── Customer_Lifecycle_Stage (Formula)
└── Marketing_Preferences (Multiple Select)
```

#### ORDERS TABLE
```
├── Order_ID (Single Line Text - Shopify Sync)
├── Order_Number (Single Line Text - Shopify Sync)
├── Customer (Link to CUSTOMERS)
├── Order_Date (Date)
├── Order_Status (Single Select: Pending, Processing, Shipped, Delivered, Cancelled)
├── Payment_Status (Single Select: Pending, Paid, Refunded, Failed)
├── Shipping_Method (Single Select: Standard, Express, Premium)
├── Tracking_Number (Single Line Text)
├── Subtotal (Currency AUD)
├── Shipping_Revenue (Currency AUD)
├── Discount_Amount (Currency AUD)
├── Total_Revenue (Currency AUD)
├── Shipping_Cost (Currency AUD)
├── Total_Product_Cost (Rollup from ORDER_LINE_ITEMS)
├── Gross_Profit (Formula: Total_Revenue - Total_Product_Cost - Shipping_Cost)
├── Profit_Margin_% (Formula: Gross_Profit/Total_Revenue*100)
├── Days_Since_Order (Formula)
├── Order_Week (Formula)
├── Order_Month (Formula)
├── Review_Request_Sent (Checkbox)
├── Customer_Notified (Checkbox)
└── Customer_Satisfaction_Score (Rating 1-5)
```

#### CUSTOMER_INQUIRIES TABLE
```
├── Inquiry_ID (Auto Number)
├── Date_Created (Created Time)
├── Platform (Single Select: Gmail, Instagram, Facebook, Shopify, SMS)
├── Customer_Email (Email)
├── Customer (Link to CUSTOMERS)
├── Subject (Single Line Text)
├── Message (Long Text)
├── Category (Single Select: Product Info, Shipping, Returns, Complaints, General)
├── Urgency (Single Select: Low, Medium, High, Urgent)
├── Sentiment_Score (Number: -5 to +5)
├── Status (Single Select: New, In Progress, Resolved, Escalated, Closed)
├── Assigned_To (Single Select: AI Bot, VA, Bella)
├── Response_Sent (Checkbox)
├── Response_Time_Minutes (Number)
├── Customer_Satisfaction (Rating 1-5)
├── SLA_Status (Formula)
├── Days_Open (Formula)
└── Resolution_Notes (Long Text)
```

#### PRODUCTS TABLE
```
├── SKU (Single Line Text - Primary Key)
├── Product_Name (Single Line Text)
├── Category (Single Select: Shampoo, Conditioner, Accessories, Bundles)
├── Active_Status (Checkbox)
├── Cost_Price (Currency AUD)
├── Retail_Price (Currency AUD)
├── Compare_Price (Currency AUD)
├── Current_Stock (Number)
├── Stock_Committed (Number)
├── Available_Stock (Formula: Current_Stock - Stock_Committed)
├── Reorder_Point (Number)
├── Reorder_Quantity (Number)
├── Lead_Time_Days (Number)
├── Primary_Supplier (Link to SUPPLIERS)
├── Gross_Margin_% (Formula)
├── 7_Day_Avg_Sales (Formula)
├── 30_Day_Avg_Sales (Formula)
├── 90_Day_Avg_Sales (Formula)
├── Days_of_Stock (Formula: Available_Stock / 7_Day_Avg_Sales)
├── Stock_Status (Formula)
└── Reorder_Alert (Formula: IF(Days_of_Stock <= Lead_Time_Days, '🚨 REORDER NOW', ''))
```

#### FINANCIAL_METRICS TABLE
```
├── Date (Date - Primary Key)
├── Revenue (Currency AUD)
├── Orders_Count (Number)
├── New_Customers (Number)
├── Items_Sold (Number)
├── Store_Sessions (Number)
├── Meta_Ad_Spend (Currency AUD)
├── Fixed_Costs (Currency AUD)
├── Variable_Costs (Currency AUD)
├── MER_Ratio (Formula: Revenue / Meta_Ad_Spend)
├── AOV (Formula: Revenue / Orders_Count)
├── Conversion_Rate_% (Formula: Orders_Count / Store_Sessions * 100)
├── Gross_Profit (Currency AUD)
├── Profit_Margin_% (Formula: Gross_Profit / Revenue * 100)
├── CPA (Formula: Meta_Ad_Spend / New_Customers)
├── Customer_Acquisition_Cost (Currency AUD)
└── ROI_% (Formula)
```

## 2. INTEGRATION ARCHITECTURE

### API Connections

#### Shopify Integration
```
Webhook Endpoints:
├── /orders/create → New order processing
├── /orders/updated → Order status updates
├── /orders/paid → Payment confirmations
├── /customers/create → New customer onboarding
├── /inventory_levels/update → Stock level changes
└── /products/update → Product information sync

API Calls:
├── GET /admin/api/2023-10/orders.json
├── POST /admin/api/2023-10/orders.json
├── PUT /admin/api/2023-10/orders/{order_id}.json
├── GET /admin/api/2023-10/customers.json
├── GET /admin/api/2023-10/products.json
└── PUT /admin/api/2023-10/inventory_levels.json
```

#### Gmail Integration
```
Watch Configuration:
├── Topic: projects/mycavoodle-automation/topics/gmail-push
├── Label Filters: INBOX, UNREAD
├── History Types: messageAdded, labelAdded
└── Webhook URL: https://hook.make.com/gmail-webhook

API Scopes:
├── https://www.googleapis.com/auth/gmail.readonly
├── https://www.googleapis.com/auth/gmail.send
├── https://www.googleapis.com/auth/gmail.modify
└── https://www.googleapis.com/auth/gmail.compose
```

#### Meta Business Suite Integration
```
Webhook Events:
├── messages → Direct message notifications
├── messaging_postbacks → Quick reply responses
├── messaging_optins → Subscription events
└── feed → Post comments and mentions

API Endpoints:
├── /{page-id}/messages
├── /{page-id}/conversations
├── /{page-id}/posts
└── /{ad-account-id}/insights
```

## 3. SECURITY & COMPLIANCE

### Data Protection
```
Encryption:
├── API Keys: Encrypted in Make.com vault
├── Customer Data: AES-256 encryption at rest
├── Webhooks: HTTPS with signature verification
└── Database: Airtable enterprise security

Access Control:
├── Role-based permissions (Bella, VA, Josh)
├── API rate limiting
├── Webhook signature validation
└── Regular access audits

Compliance:
├── GDPR: Data retention policies
├── Privacy Policy: Customer consent
├── Security Monitoring: Real-time alerts
└── Backup Strategy: Daily automated backups
```

### Error Handling
```
Failsafe Mechanisms:
├── Retry Logic: 3 attempts with exponential backoff
├── Dead Letter Queue: Failed webhook storage
├── Manual Fallback: Critical path alternatives
├── Alert System: Immediate failure notifications
└── Health Checks: System status monitoring
```

## 4. PERFORMANCE SPECIFICATIONS

### System Requirements
```
Latency Targets:
├── Customer Inquiry Response: <5 minutes
├── Order Processing: <30 seconds
├── Inventory Updates: Real-time
├── Report Generation: <60 seconds
└── Dashboard Refresh: <10 seconds

Throughput Capacity:
├── Orders per Hour: 100+
├── Inquiries per Hour: 200+
├── API Calls per Minute: 1000+
├── Webhook Processing: Real-time
└── Concurrent Users: 50+

Availability:
├── System Uptime: >99.9%
├── Response Success Rate: >99.5%
├── Data Accuracy: >99.9%
├── Automation Success: >95%
└── SLA Compliance: >98%
```

This architecture provides the foundation for a highly automated, scalable e-commerce operation that significantly reduces founder dependency while maintaining exceptional customer experience.