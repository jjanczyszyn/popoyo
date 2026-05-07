# Moto Rental Admin Panel Performance Spec

## Product

Admin panel for a small motorcycle rental business with a small fleet.

The panel helps the manager understand:

1. How the business performs across months
2. Which motorcycles get rented most
3. When demand is strongest and weakest
4. How much money the business makes each month
5. Which payment methods customers use
6. How much money is routed through each payment method
7. How much settlement balance exists between JJ and Karen

This spec intentionally uses one money number: **revenue**.

Do not use separate concepts of gross and net revenue in this version.

---

## Business Rules

### Ownership Split

All received customer payments are split:

| Person | Share |
|---|---:|
| JJ | 70% |
| Karen | 30% |

### Collection Rules

By default:

| Payment Method | Collected By |
|---|---|
| Cash | Karen |
| Zelle | JJ |
| Bank transfer | JJ |
| Stripe / Card | JJ |
| PayPal | JJ |
| Wise | JJ |
| Other | Manual selection |

Important rule:

Each payment must store `collected_by`.

The payment method should only auto-fill the collector. It should not permanently determine the collector.

Example:

If a customer pays cash but JJ physically receives the cash, the payment should be recorded as:

```txt
payment_method: cash
collected_by: JJ
```

This prevents settlement errors.

---

## Core Product Principle

The admin panel is not just a booking tracker.

It is a business cockpit.

The key question is:

```txt
Which motorcycle made money, during which month, through which payment method, and who currently owes whom between JJ and Karen?
```

---

## Main Navigation

1. Dashboard
2. Bookings
3. Motorcycles
4. Revenue
5. Payments
6. Seasonality
7. Partner Settlement
8. Reports
9. Settings

---

# 1. Dashboard

## Purpose

The dashboard should answer the business health question in under 30 seconds.

## Top Cards

Show these cards at the top:

| Card | Description |
|---|---|
| Revenue this month | Total customer payments received in selected month |
| Number of rentals | Total confirmed bookings in selected month |
| Rental days sold | Total number of motorcycle rental days sold |
| Occupancy rate | Percent of available motorcycle days rented |
| Average daily rate | Revenue divided by rental days sold |
| Best performing motorcycle | Motorcycle with highest revenue or occupancy |
| Most used payment method | Payment method with highest received amount |
| Partner settlement balance | Who owes whom between JJ and Karen |

Example:

```txt
May 2026

Revenue: $2,840
Rental days sold: 71
Occupancy: 76%
Average daily rate: $40
Top motorcycle: Yamaha XT 125 Blue
Top payment method: Cash
Settlement: Karen owes JJ $298
```

## Filters

Every dashboard card should respond to:

```txt
Date range
Month
Year
Motorcycle
Booking status
Payment method
Customer source
```

Default view:

```txt
Current month
Confirmed, active, and completed bookings
Received payments only
```

---

# 2. Revenue by Month

## Purpose

Show how much money the business makes month by month.

## Chart

Use a simple bar chart:

```txt
Jan   $1,200
Feb   $1,850
Mar   $2,600
Apr   $3,100
May   $2,840
Jun   $1,950
```

## Monthly Revenue Table

| Month | Revenue | Rentals | Rental Days | Occupancy | Average Daily Rate |
|---|---:|---:|---:|---:|---:|
| January | $1,200 | 9 | 31 | 34% | $38.71 |
| February | $1,850 | 14 | 45 | 54% | $41.11 |
| March | $2,600 | 19 | 66 | 71% | $39.39 |

## Requirements

Revenue should be calculated from received customer payments.

Do not calculate revenue from booking totals unless payment has actually been received.

This avoids fake revenue from unpaid or partially paid bookings.

---

# 3. Motorcycle Performance

## Purpose

Answer:

```txt
Which motorcycles are making money?
Which motorcycles are rented most?
Which motorcycles are underused?
Which motorcycles are blocked by maintenance?
```

## Motorcycle Ranking Table

| Motorcycle | Revenue | Rental Days | Occupancy | Average Daily Rate | Bookings | Maintenance Days | Revenue per Available Day |
|---|---:|---:|---:|---:|---:|---:|---:|
| Yamaha XT 125 Blue | $1,420 | 36 | 80% | $39.44 | 11 | 2 | $31.55 |
| Yamaha XT 125 Red | $980 | 25 | 56% | $39.20 | 8 | 0 | $21.77 |
| Honda Navi | $440 | 14 | 31% | $31.42 | 6 | 3 | $9.77 |

## Key Metrics

```txt
Rental days sold = sum of booked days across confirmed, active, and completed bookings
```

```txt
Available motorcycle days = days in selected period minus maintenance days minus blocked days
```

```txt
Occupancy rate = rental days sold / available motorcycle days
```

```txt
Average daily rate = revenue / rental days sold
```

```txt
Revenue per available day = revenue / available motorcycle days
```

## Product Note

Do not rank motorcycles only by number of bookings.

A motorcycle rented once for 14 days is more valuable than a motorcycle rented five times for one day each.

Primary ranking should support:

1. Revenue
2. Rental days
3. Occupancy
4. Revenue per available day

---

# 4. Seasonality

## Purpose

Answer:

```txt
What are the best months?
What are the weakest months?
When should we raise prices?
When should we offer discounts?
When should we schedule maintenance?
```

## Monthly Demand Heatmap

Rows are motorcycles.

Columns are months.

Each cell shows occupancy percentage.

| Motorcycle | Jan | Feb | Mar | Apr | May | Jun | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Yamaha Blue | 42% | 68% | 90% | 82% | 75% | 50% | 31% | 28% | 35% | 45% | 61% | 80% |
| Yamaha Red | 35% | 55% | 78% | 70% | 62% | 40% | 22% | 20% | 29% | 39% | 52% | 69% |
| Honda Navi | 18% | 29% | 40% | 38% | 31% | 20% | 12% | 10% | 15% | 21% | 28% | 35% |

## Best Months Table

| Rank | Month | Revenue | Occupancy | Rental Days | Notes |
|---|---|---:|---:|---:|---|
| 1 | March | $2,600 | 78% | 66 | Strong tourist and surf demand |
| 2 | April | $2,430 | 74% | 61 | Strong travel demand |
| 3 | December | $2,200 | 70% | 58 | Holiday demand |

## Lowest Demand Months Table

| Rank | Month | Revenue | Occupancy | Rental Days | Suggested Action |
|---|---|---:|---:|---:|---|
| 1 | August | $620 | 19% | 16 | Offer longer rental discounts |
| 2 | September | $740 | 23% | 19 | Schedule maintenance |
| 3 | July | $890 | 28% | 24 | Partner with hotels and hostels |

## Seasonality Insight Cards

The app should automatically surface simple insights:

```txt
March is your strongest month with 78% occupancy and $2,600 revenue.
```

```txt
August is your weakest month with 19% occupancy and $620 revenue.
```

```txt
Yamaha Blue is rented 2.6x more often than Honda Navi.
```

```txt
Low-demand months are the best time for maintenance, repairs, content creation, and partner outreach.
```

---

# 5. Calendar View

## Purpose

Give a visual overview of motorcycle availability.

## Layout

Rows are motorcycles.

Columns are days.

```txt
             May 1   May 2   May 3   May 4   May 5
Yamaha Blue  Rented  Rented  Free    Free    Rented
Yamaha Red   Free    Rented  Rented  Free    Free
Honda Navi   Free    Free    Free    Maint.  Maint.
```

## Statuses

| Status | Meaning |
|---|---|
| Available | Motorcycle can be booked |
| Reserved | Booking request pending |
| Rented | Confirmed rental |
| Maintenance | Not available due to maintenance |
| Blocked | Manually unavailable |

## Requirements

The calendar should prevent double-booking.

A motorcycle cannot have overlapping confirmed or active bookings.

---

# 6. Payments

## Purpose

Answer:

```txt
How much money came in?
Which payment methods are customers using?
How much money was routed through each method?
Who collected each payment?
```

## Payment Method Summary

| Payment Method | Amount Received | Percent of Revenue | Number of Payments | Default Collector |
|---|---:|---:|---:|---|
| Cash | $1,250 | 44% | 13 | Karen |
| Zelle | $680 | 24% | 6 | JJ |
| Stripe / Card | $540 | 19% | 8 | JJ |
| PayPal | $250 | 9% | 3 | JJ |
| Wise | $120 | 4% | 1 | JJ |

## Payment Method Chart

Use a donut chart or bar chart:

```txt
Cash: 44%
Zelle: 24%
Stripe/Card: 19%
PayPal: 9%
Wise: 4%
```

## Payment List

| Date | Booking | Customer | Motorcycle | Method | Amount | Collected By |
|---|---|---|---|---|---:|---|
| May 1 | #1042 | Anna K. | Yamaha Blue | Cash | $200 | Karen |
| May 3 | #1043 | Mark T. | Honda Navi | Zelle | $160 | JJ |
| May 4 | #1044 | Sofia M. | Yamaha Red | Cash | $80 | Karen |

## Payment Rules

Only received payments count as revenue.

Pending, failed, or cancelled payments do not count as revenue.

Refunds should reduce revenue.

Each payment must support:

```txt
amount
payment_method
collected_by
payment_status
received_at
booking_id
notes
```

---

# 7. Partner Settlement

## Purpose

Answer:

```txt
Based on how customers paid, who currently owes whom between JJ and Karen?
```

This is the most important financial reconciliation screen.

## Business Logic

All customer payments are split:

```txt
JJ gets 70%
Karen gets 30%
```

But the actual collector depends on payment method:

```txt
Cash usually goes to Karen.
All other payment methods usually go to JJ.
```

This creates a settlement balance.

## Mental Model

Every payment has two meanings:

```txt
1. Who collected the money?
2. Who is entitled to what share?
```

Examples:

```txt
Customer pays $100 in cash.
Karen collects $100.
JJ should receive $70.
Karen should keep $30.
Settlement impact: Karen owes JJ $70.
```

```txt
Customer pays $100 by Zelle.
JJ collects $100.
JJ should keep $70.
Karen should receive $30.
Settlement impact: JJ owes Karen $30.
```

## Settlement Page Layout

```txt
[Month selector]

Total received: $2,840

Expected split:
JJ 70%: $1,988
Karen 30%: $852

Actually collected:
JJ: $1,690
Karen: $1,150

Settlement:
Karen owes JJ $298

[Record settlement transfer]
```

## Settlement Formula

For any date range:

```txt
total_revenue = sum(received customer payments)
```

```txt
jj_expected = total_revenue * 0.70
karen_expected = total_revenue * 0.30
```

```txt
jj_collected = sum(payments where collected_by = JJ)
karen_collected = sum(payments where collected_by = Karen)
```

```txt
jj_balance_before_settlements = jj_expected - jj_collected
```

Then account for settlement transfers:

```txt
jj_received_settlements = sum(settlement transfers where to = JJ)
jj_sent_settlements = sum(settlement transfers where from = JJ)
```

```txt
jj_final_balance = jj_expected - jj_collected - jj_received_settlements + jj_sent_settlements
```

Interpretation:

| JJ Final Balance | Meaning |
|---:|---|
| Positive | Karen owes JJ this amount |
| Negative | JJ owes Karen this amount |
| Zero | Settled |

Example:

```txt
Total revenue: $2,840
JJ expected: $1,988
JJ collected: $1,690
JJ final balance before settlements: $298

Karen owes JJ $298.
```

## Partner Settlement Table

| Month | Revenue | JJ Share 70% | Karen Share 30% | JJ Collected | Karen Collected | Settlement Balance |
|---|---:|---:|---:|---:|---:|---|
| Jan | $1,200 | $840 | $360 | $900 | $300 | JJ owes Karen $60 |
| Feb | $1,850 | $1,295 | $555 | $1,100 | $750 | Karen owes JJ $195 |
| Mar | $2,600 | $1,820 | $780 | $1,700 | $900 | Karen owes JJ $120 |
| Apr | $3,100 | $2,170 | $930 | $2,500 | $600 | JJ owes Karen $330 |

## Settlement by Payment Method

| Payment Method | Amount | Collected By | JJ Share | Karen Share | Settlement Impact |
|---|---:|---|---:|---:|---|
| Cash | $1,150 | Karen | $805 | $345 | Karen owes JJ $805 |
| Zelle | $820 | JJ | $574 | $246 | JJ owes Karen $246 |
| Stripe / Card | $540 | JJ | $378 | $162 | JJ owes Karen $162 |
| PayPal | $230 | JJ | $161 | $69 | JJ owes Karen $69 |
| Wise | $100 | JJ | $70 | $30 | JJ owes Karen $30 |

Summary:

```txt
Cash created balance in favor of JJ: $805
Non-cash created balance in favor of Karen: $507

Net settlement:
Karen owes JJ $298
```

## Settlement Status

Each month or date range should have one of these statuses:

| Status | Meaning |
|---|---|
| Open | Month still active |
| Needs settlement | A balance exists and has not been fully settled |
| Partially settled | Some settlement transfers have been recorded |
| Settled | Balance is zero |
| Locked | Month finalized and edits are restricted |

## Settlement Transfers

Settlement transfers are payments between JJ and Karen.

They are not customer payments.

They should not count as business revenue.

### Settlement Transfer Fields

```txt
id
from: JJ / Karen
to: JJ / Karen
amount
date
method
settlement_month
notes
created_at
```

Example:

```txt
Karen paid JJ $298 by bank transfer for May settlement.
```

After this is recorded, May becomes settled.

## Settlement Transfer History

| Date | From | To | Amount | Method | Month Applied To | Notes |
|---|---|---|---:|---|---|---|
| May 8 | Karen | JJ | $100 | Bank transfer | May 2026 | Partial settlement |
| May 16 | Karen | JJ | $198 | Bank transfer | May 2026 | Final settlement |

## UX Requirement

The final settlement result should be visually prominent.

Examples:

```txt
Karen owes JJ $298
```

```txt
JJ owes Karen $120
```

```txt
Settled
```

Do not label this as debt.

Use the neutral language:

```txt
Partner Settlement Balance
```

---

# 8. Bookings

## Purpose

Manage operational bookings and connect them to payments.

## Booking List

| Booking | Customer | Motorcycle | Dates | Rental Days | Revenue Expected | Paid | Balance | Status |
|---|---|---|---|---:|---:|---:|---:|---|
| #1042 | Anna K. | Yamaha Blue | May 1 to May 5 | 5 | $200 | $200 | $0 | Paid |
| #1043 | Mark T. | Honda Navi | May 3 to May 4 | 2 | $60 | $30 | $30 | Partial |
| #1044 | Sofia M. | Yamaha Red | May 6 to May 9 | 4 | $160 | $160 | $0 | Paid |

## Booking Statuses

| Status | Meaning |
|---|---|
| Pending | Request received but not confirmed |
| Confirmed | Booking accepted |
| Active | Motorcycle currently rented |
| Completed | Rental finished |
| Cancelled | Booking cancelled |
| No-show | Customer did not arrive |

## Payment Statuses

| Status | Meaning |
|---|---|
| Unpaid | No received payments |
| Partial | Some money received |
| Paid | Full booking amount received |
| Overpaid | Received amount exceeds booking amount |
| Refunded | Booking was refunded |

## Booking Fields

```txt
id
customer_name
customer_phone
customer_email
motorcycle_id
start_date
end_date
rental_days
daily_rate
discount
total_price
status
source
notes
jj_share_percentage
karen_share_percentage
created_at
updated_at
```

## Important Rule

Store the split percentage on each booking.

Default:

```txt
jj_share_percentage = 70
karen_share_percentage = 30
```

Why:

If the business split changes later, historical reports should not change.

---

# 9. Booking Sources

## Purpose

Answer:

```txt
Where do good customers come from?
Which source brings longer rentals?
Which source brings the most money?
```

## Source Performance Table

| Source | Bookings | Revenue | Rental Days | Average Booking Value |
|---|---:|---:|---:|---:|
| WhatsApp | 14 | $1,480 | 37 | $105 |
| Website | 8 | $920 | 23 | $115 |
| Referral | 5 | $700 | 17 | $140 |
| Hotel Partner | 3 | $480 | 12 | $160 |
| Walk-in | 2 | $160 | 4 | $80 |

## Source Values

```txt
website
WhatsApp
referral
hotel_partner
walk_in
Instagram
Facebook
other
```

## Product Note

The source with the most bookings may not be the best source.

The best source is usually the one with:

```txt
high revenue
long rentals
low hassle
low cancellation rate
```

---

# 10. Reports

## Required Reports

### Monthly Performance Report

Shows:

```txt
Revenue
Bookings
Rental days
Occupancy
Average daily rate
Best motorcycle
Worst motorcycle
Top payment method
Partner settlement balance
```

### Motorcycle Performance Report

Shows:

```txt
Revenue by motorcycle
Rental days by motorcycle
Occupancy by motorcycle
Maintenance days by motorcycle
Revenue per available day
```

### Payment Method Report

Shows:

```txt
Revenue by payment method
Number of payments by method
Collected by JJ
Collected by Karen
Settlement impact by method
```

### Partner Settlement Report

Shows:

```txt
Expected JJ share
Expected Karen share
Actually collected by JJ
Actually collected by Karen
Settlement transfers
Final settlement balance
Settlement status
```

### Seasonality Report

Shows:

```txt
Best months
Lowest demand months
Month-by-month occupancy
Month-by-month revenue
Suggested actions
```

## Export Requirements

Each report should support:

```txt
Export CSV
Export PDF, optional later
```

MVP only requires CSV.

---

# 11. Settings

## Business Settings

```txt
business_name
currency
timezone
default_daily_rate
jj_share_percentage
karen_share_percentage
```

## Payment Method Settings

Admin can configure default collector.

| Payment Method | Default Collector |
|---|---|
| Cash | Karen |
| Zelle | JJ |
| Bank transfer | JJ |
| Stripe / Card | JJ |
| PayPal | JJ |
| Wise | JJ |
| Other | Manual |

## Motorcycle Settings

Each motorcycle should have:

```txt
name
model
license_plate
status
daily_rate
photo_url
notes
```

## Status Options

Motorcycle status:

```txt
active
inactive
maintenance
sold
```

---

# 12. Data Model

## motorcycles

```txt
id
name
model
license_plate
status
daily_rate
photo_url
notes
created_at
updated_at
```

## bookings

```txt
id
customer_name
customer_phone
customer_email
motorcycle_id
start_date
end_date
rental_days
daily_rate
discount
total_price
status
source
notes
jj_share_percentage
karen_share_percentage
created_at
updated_at
```

## payments

```txt
id
booking_id
amount
currency
payment_method
collected_by: JJ / Karen
payment_type: deposit / balance / full_payment / refund
payment_status: received / pending / failed / cancelled / refunded
received_at
notes
created_at
updated_at
```

## motorcycle_availability

```txt
id
motorcycle_id
date
status: available / reserved / rented / maintenance / blocked
booking_id
notes
created_at
updated_at
```

## settlement_transfers

```txt
id
from_partner: JJ / Karen
to_partner: JJ / Karen
amount
currency
date
method
settlement_month
notes
created_at
updated_at
```

## business_settings

```txt
id
business_name
currency
timezone
jj_share_percentage
karen_share_percentage
created_at
updated_at
```

## payment_method_settings

```txt
id
payment_method
default_collector: JJ / Karen / manual
enabled
created_at
updated_at
```

## expenses, optional V2

```txt
id
motorcycle_id
category
amount
currency
date
notes
created_at
updated_at
```

Expenses are optional for V1.

Without expenses, the app tracks revenue and settlement, not profit.

---

# 13. Calculation Details

## Revenue

```txt
revenue = sum(payments where payment_status = received and payment_type is not refund)
          - sum(payments where payment_type = refund)
```

## Rental Days

```txt
rental_days = difference between end_date and start_date
```

Decide whether end date is inclusive or exclusive.

Recommended:

```txt
start_date is inclusive
end_date is exclusive
```

Example:

```txt
May 1 to May 5 = 4 rental days
```

This matches hotel-style booking logic and avoids off-by-one errors.

## Available Motorcycle Days

```txt
available_motorcycle_days = number of days in selected period
                            - maintenance days
                            - blocked days
```

Per motorcycle.

For fleet-wide availability:

```txt
fleet_available_days = sum(available_motorcycle_days for each active motorcycle)
```

## Occupancy

```txt
occupancy_rate = rental_days_sold / available_motorcycle_days
```

## Average Daily Rate

```txt
average_daily_rate = revenue / rental_days_sold
```

If rental days sold is zero, show:

```txt
N/A
```

## Revenue per Available Day

```txt
revenue_per_available_day = revenue / available_motorcycle_days
```

## Booking Balance

```txt
booking_balance = total_price - sum(received payments for booking)
```

Refunds should reduce the received amount.

## Settlement

Use booking-level split when available.

For each payment:

```txt
jj_expected_from_payment = payment.amount * booking.jj_share_percentage / 100
karen_expected_from_payment = payment.amount * booking.karen_share_percentage / 100
```

For a refund:

```txt
jj_expected_from_refund = refund.amount * booking.jj_share_percentage / 100
karen_expected_from_refund = refund.amount * booking.karen_share_percentage / 100
```

The refund reduces both expected shares.

For V1, settlement can be calculated from received customer payments only.

Pending payments do not count.

---

# 14. Example Settlement Calculation

Customer payments in May:

| Method | Amount | Collected By |
|---|---:|---|
| Cash | $1,000 | Karen |
| Zelle | $800 | JJ |
| Stripe / Card | $400 | JJ |

Total revenue:

```txt
$2,200
```

Expected split:

```txt
JJ: $1,540
Karen: $660
```

Actually collected:

```txt
JJ: $1,200
Karen: $1,000
```

Settlement:

```txt
JJ should have $1,540 but collected $1,200.

Karen owes JJ $340.
```

---

# 15. MVP Scope

Build these first:

1. Admin authentication
2. Motorcycle list
3. Booking list
4. Payment recording
5. Monthly revenue chart
6. Revenue by motorcycle table
7. Rental days and occupancy by motorcycle
8. Payment method breakdown
9. Partner settlement page
10. Settlement transfer recording
11. Month comparison table
12. CSV export

## MVP Exclusions

Do not build these in V1:

```txt
Profit tracking
Expense tracking
Advanced dynamic pricing
Automated tax reporting
PDF reports
Customer accounts
Customer login
Complex accounting features
Multi-partner splits
Inventory beyond motorcycles
```

---

# 16. V2 Scope

Add later:

```txt
Expense tracking
Profit by motorcycle
Maintenance cost per motorcycle
Dynamic pricing suggestions
Customer repeat rate
Cancellation rate
Deposit tracking
Low-season promotion suggestions
Year-over-year comparison
PDF reports
Automated WhatsApp reminders
```

---

# 17. Suggested Admin UX

## Dashboard Page

```txt
[Month selector] [Date range] [Export CSV]

Revenue       Rentals       Rental Days       Occupancy
$2,840        23            71                76%

Top Motorcycle
Yamaha XT 125 Blue
$1,420 revenue
36 rental days
80% occupancy

Top Payment Method
Cash
$1,250 received
44% of revenue

Partner Settlement
Karen owes JJ $298
```

## Partner Settlement Page

```txt
[Month selector]

Total received: $2,840

Expected split
JJ: $1,988
Karen: $852

Actually collected
JJ: $1,690
Karen: $1,150

Settlement balance
Karen owes JJ $298

[Record settlement transfer]

Payment method breakdown
Cash: $1,150, collected by Karen
Zelle: $820, collected by JJ
Stripe/Card: $540, collected by JJ
PayPal: $230, collected by JJ
Wise: $100, collected by JJ

Settlement history
May 8: Karen paid JJ $100
May 16: Karen paid JJ $198

Status: Settled
```

---

# 18. Validation Rules

## Booking Validation

```txt
start_date is required
end_date is required
end_date must be after start_date
motorcycle_id is required
customer_name is required
total_price must be >= 0
booking cannot overlap another confirmed or active booking for the same motorcycle
```

## Payment Validation

```txt
booking_id is required
amount must be > 0
payment_method is required
collected_by is required
payment_status is required
received_at is required if payment_status = received
```

## Settlement Transfer Validation

```txt
from_partner is required
to_partner is required
from_partner cannot equal to_partner
amount must be > 0
date is required
settlement_month is required
```

---

# 19. Permissions

## Admin Only

All admin pages require manager authentication.

Admin can:

```txt
Create bookings
Edit bookings
Cancel bookings
Record payments
Record refunds
Edit payment collector
View reports
Record settlement transfers
Export CSV
Edit business settings
```

## No Customer Accounts

Customers do not need accounts.

The admin panel is manager-only.

---

# 20. Success Criteria

The admin panel is successful when the manager can answer these questions in under 30 seconds:

```txt
How much money did we receive this month?
Which motorcycle made the most money?
Which motorcycle was rented most?
What was our occupancy this month?
What are our strongest and weakest months?
Which payment methods are customers using?
How much cash did Karen collect?
How much non-cash did JJ collect?
Who owes whom between JJ and Karen?
Has this month been settled?
```

---

# 21. Product Manager Notes

## Most Important Implementation Choice

Settlement should be based on received payments, not bookings.

The real-world problem is:

```txt
Where did the money land?
```

The booking tells what was reserved.

The payment tells what money was actually received.

The settlement tells who needs to transfer money to whom.

## Biggest Accounting Risk

Do not calculate settlement based only on payment method.

Store `collected_by` on every payment.

Why:

```txt
Cash usually goes to Karen, but not always.
Non-cash usually goes to JJ, but not always.
```

The system should support real life, not wishful accounting.

## Future-Proofing

Store the JJ/Karen split on every booking.

Default it from business settings.

This protects historical reports if the business split changes later.

---

# 22. Terminology

Use these labels in the interface:

| Use This | Avoid This |
|---|---|
| Revenue | Gross revenue |
| Revenue | Net revenue |
| Partner Settlement Balance | Debt |
| Collected by | Owner |
| Received payment | Income event |
| Settlement transfer | Customer payment |
| Paid | Settled, unless referring to partner settlement |

---

# 23. Open Questions

Resolve before development if needed:

1. Should the app support multiple currencies?
2. Should revenue be tracked in USD only?
3. Should partial-day rentals exist?
4. Should delivery fees be included in revenue?
5. Should deposits be counted immediately when received?
6. Should security deposits be excluded from revenue if refundable?
7. Should refunds reduce the original month or the month when refund happens?

Recommended defaults:

```txt
Use one currency for V1.
Use full-day rentals only.
Include delivery fees in revenue if they are charged to the customer.
Count deposits when received.
Do not count refundable security deposits as revenue.
Apply refunds to the month when the refund happens.
```
