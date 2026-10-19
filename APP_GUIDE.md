# Manjunadha Disposable Hub — App Guide & Feature Reference

> Last updated: October 2026
> Use this document when adding new features — understand what exists first.

---

## Overview

A single-file web app (index.html) for managing a paper plate business.
Hosted free on GitHub Pages. Data stored on Firebase Firestore (Google).

- Live Link: https://manikanagarla.github.io/plate-manager/
- GitHub Repo: https://github.com/ManiKanagarla/plate-manager
- Firebase Project: disposable-business-559f2
- App File: index.html (all HTML + CSS + JS in one file)

---

## Tech Stack

| Layer | Tool |
|---|---|
| Frontend | Plain HTML + CSS + JavaScript (no framework) |
| Database | Firebase Firestore (free Spark plan) |
| Hosting | GitHub Pages (free) |
| Image generation | html2canvas (CDN) |
| Fonts | Google Fonts — Inter |

---

## Authentication

- Password-protected login screen
- Password stored in JS constant: APP_PASSWORD (around line 1913)
- Current password: mdh@1210
- Session-based — stays logged in until browser/tab closed
- Logout button in header

---

## Plate Types (6 types)

```javascript
const PLATE_TYPES = [
    { key: '8_paper',         label: '8 inch Paper'              },
    { key: '10_paper',        label: '10 inch Paper'             },
    { key: '10_buffet',       label: '10 inch Buffet'            },
    { key: '12_buffet_green', label: '12 inch Buffet Green'      },
    { key: '12_buffet_multi', label: '12 inch Buffet Multicolor' },
    { key: '12_leaf',         label: '12 inch Leaf'              },
];
```

---

## Firebase Collections (Database Structure)

### 1. customers
```
id              auto by Firebase
name            string
phone           string
location        string
address         string (optional)
mapLink         string (optional)
notes           string (optional)
rates           object  { '8_paper': 800, '10_buffet': 1000, ... }
createdAt       ISO string
updatedAt       ISO string
```
Note: Old customers (before plate type update) may have a single rate field
instead of rates object. Both are handled in the UI.

---

### 2. sales
```
id              auto by Firebase
date            string  YYYY-MM-DD
customerId      string  (ref to customers)
customerName    string  (denormalized for display)
location        string  (denormalized)
stockNumber     number  (which stock this came from)
items           array   [ { plateKey, plateLabel, size, type, bundles, rate, subtotal } ]
bundles         number  (total bundles across all items — for dashboard/stock tracking)
total           number  (sum of all item subtotals)
status          string  pending | paid | partial
paymentMode     string  cash | phonepay | upi | bank
paymentDate     string  YYYY-MM-DD
amountPaid      number  (for partial payments)
notes           string
createdAt       ISO string
updatedAt       ISO string
```
Note: Old sales (before plate type update) may not have items array.
UI handles both — falls back to showing bundles count for legacy sales.

---

### 3. stocks
```
id              auto by Firebase
stockNumber     number  (1, 2, 3 ... sequential)
billingMonth    string  YYYY-MM (which month this counts under in reports)
orders          array   (multi-order format — see below)
bundles         number  (total bundles across all orders — auto calculated)
purchaseCost    number  (total purchase cost — auto calculated)
transportCost   number  (total transport cost — auto calculated)
totalCost       number  (purchaseCost + transportCost — auto calculated)
source          string  (comma-joined unique sources from orders)
paymentMode     string  cash | phonepay | upi | bank
paymentDate     string  YYYY-MM-DD
returnBundles   number  (bundles returned to supplier, optional)
notes           string
createdAt       ISO string
updatedAt       ISO string
```

Each order inside `orders` array:
```
date            string  YYYY-MM-DD (actual purchase date of this order)
source          string  (e.g. Hyderabad)
supplier        string  (optional)
transportCost   number
items           array   [ { plateKey, plateLabel, bundles, purchaseCost } ]
```

Note: Old stocks (before multi-order update) have flat fields (date, source, supplier, bundles, purchaseCost).
UI handles both formats — old stocks auto-migrated to single-order display when editing.

---

### 4. expenses
```
id              auto by Firebase
month           string  YYYY-MM  (one entry per month)
rent            number
petrol          number
electricity     number
other           number
otherNotes      string
savings         number
notes           string
createdAt       ISO string
updatedAt       ISO string
```

---

## In-Memory State (JS Variables)

```javascript
let customers = [];   // live from Firestore onSnapshot
let sales = [];       // ordered by date desc
let stocks = [];      // ordered by stockNumber desc
let expenses = [];    // ordered by month desc
let chitfund = [];    // ordered by month desc
```

All 5 collections use onSnapshot — real-time sync across devices.

---

## App Sections

### 1. Home (Dashboard)

Stat cards (top):
- Today's bundles sold + amount
- This month's bundles + collected amount
- Total pending amount (pending + partial sales)
- Total customers count (active / inactive breakdown)

Monthly Target card:
- 1000 Bundles Monthly Target — resets every month
- Progress bar: current month bundles sold vs 1000
- Shows current month name, days left until end of month, bundles/day needed
- Turns green with "Target achieved this month!" when 1000 is crossed

Current Stock card:
- Total / Sold / Remaining bundles
- Per plate type breakdown: purchased vs sold vs remaining
  - Green = healthy, Amber = ≤10 left, Red = 0 or negative

Overdue Payments alert:
- Auto-appears when any pending/partial sale is older than 15 days
- Groups by customer, shows total due and oldest sale age
- Auto-hides when no overdue sales

Recent 5 sales

---

### 2. Customers

List view:
- Search by name or phone
- Filter: All | Pending Only
- Filter by location/area (dynamic chips)
- Shows pending amount per customer in red

Customer detail modal:
- Name, phone, location
- Rates per plate type (e.g. 8 inch Paper -> Rs.800/bundle)
- Stats: Sales count, Bundles sold, Collected, Pending
- Actions: Share Payment Card, Edit, Delete

Share Payment Card:
- Generates PNG image using html2canvas
- Card shows: Customer details, Last payment (date + amount),
  Pending amount, Pending bills table (date + plate types + amount)
- Mobile: Web Share API -> WhatsApp native share
- Desktop: Downloads PNG

---

### 3. Sales

List view:
- Search by customer name
- Filter chips: All | Pending | Paid | Partial | Today | This Month
- Filter dropdowns: Customer, Stock, Payment Mode, Plate Type
- Date range filter (from -> to)
- Summary bar: count, bundles, total, pending

Sale card shows:
- Customer name + date + location + stock
- Plate type items (e.g. 5 x 8 inch Paper, 3 x 10 inch Buffet)
- Total bill amount
- Balance Due in red (for pending and partial sales)
- Status badge (Paid / Pending / Partial)
- Overdue badge + red left border if sale is pending/partial and older than 15 days (e.g. "⚠ 20d overdue")

Sale detail modal:
- Full breakdown: each plate type -> bundles x rate = subtotal
- Total amount
- Payment details (mode, date, amount paid, balance due)
- Mark as Paid (Cash) quick button for pending sales
- Edit / Delete

Add/Edit Sale form:
- Date, Customer, Stock Number
- Dynamic plate type items — + Add Plate Type button
  - Each row: select plate type -> rate auto-fills from customer rates
  - Enter bundles -> subtotal auto-calculates
  - Multiple rows supported (one per plate type in same delivery)
- Total auto-calculates across all items
- Payment status: Pending / Paid / Partial
- Payment mode, date, amount paid (for paid/partial)

---

### 4. Stock Purchase

Stock list:
- Filter by billing month and source
- Shows: stock number, source, date, billing month,
  total/sold/remaining bundles, costs
- Edit / Delete per stock

Add/Edit Stock form:
- Stock number, billing month
- Multi-order support — add multiple orders under one stock number
- Per order: date, source, supplier, transport cost
- Per order items: plate type + bundles + purchase cost (one row per plate type)
- Grand total auto-calculates across all orders and items
- Payment mode, payment date
- Return bundles (optional)

Monthly Expenses (separate section on same page):
- One entry per month
- Shop rent, petrol/travel, electricity, other expenses (with notes), savings
- Edit / Delete per month

---

### 5. Reports

Top Customers card (by bundles sold, top 5)

Monthly report cards (one per month, latest first):

Sales section:
- Transactions count
- Total bundles sold
- Plate type breakdown (e.g. 8 inch Paper: 120 bundles, 10 inch Buffet: 80 bundles)
- Total revenue, Collected amount, Pending amount

Stocks section (stocks with billingMonth = this month):
- Each stock: cost, sold/total bundles, remaining
- Total stock cost

Monthly Expenses section:
- Rent, petrol, electricity, other breakdown
- Total expenses

Savings section

Profit / Loss section (auto-calculated, shows only if stock cost or expenses exist):
```
Collected - Stock Cost - Expenses - Chit Fund (paid) = Net Profit
```
- Net Profit shown in green (profit) or red (loss)
- Chit fund deducted only if marked as paid for that month

### 6. Chit Fund
- Track monthly chit fund installment payments
- Per month: amount, status (paid/pending), payment date, notes
- Summary: total paid, total pending, months count
- Edit / Delete per month entry

---

## Key Business Logic

### Stock Tracking
- bundles sold from stock X = sum of sale.bundles where sale.stockNumber === X
- remaining = stock.bundles - soldBundles
- Per plate type remaining = purchased (from stock.orders.items) - sold (from sales.items)
- Stock numbers are sequential (1, 2, 3...) — up to 2 stocks per month
- billingMonth controls which month a stock appears in reports

### Overdue Detection
- A sale is overdue if: status is pending or partial AND daysSince(sale.date) > 15
- daysSince(dateStr) = Math.floor((now - saleDate) / 86400000)
- Overdue sales show: red left border + "⚠ Xd overdue" badge on sale cards
- Dashboard shows overdue alert section grouped by customer (auto-hides when none)

### Profit/Loss Calculation (per month in Reports)
```
Net Profit = (collected + partialCollected)
           - monthStocks.totalCost        (purchaseCost + transportCost)
           - expenses (rent+petrol+electricity+other)
           - chitfund.amount (only if status === 'paid' for that month)
```

### 1000 Bundles Monthly Target
- Target: 1000 bundles every month (resets each month)
- monthSoldBundles = sum of sales.bundles where sale.date starts with current YYYY-MM
- remaining = 1000 - monthSoldBundles
- daysLeft = days until last day of current month
- perDayNeeded = remaining / daysLeft
- Turns green and shows "Target achieved this month!" when monthSoldBundles >= 1000

### Pending Amount Calculation
```
if status === pending  -> due = total
if status === partial  -> due = total - amountPaid
if status === paid     -> due = 0

Customer total pending = sum of due across all their pending/partial sales
```

### Sale Total Calculation
```
Each item:  subtotal = bundles x rate
Sale total  = sum of all item subtotals
Total bundles = sum of all item bundles (used for stock tracking)
```

---

## UI Patterns

| Pattern | How it works |
|---|---|
| Modals | Bottom sheet slide-up (.modal-overlay + .modal) |
| Toast | Small pill at bottom — showToast('message') |
| Confirm delete | Centre overlay dialog |
| Real-time | All collections use Firestore onSnapshot |
| XSS safety | esc() function used for all user data in HTML |
| INR formatting | fmtINR(n) -> Rs.1,23,456 |
| Date formatting | fmtDate('YYYY-MM-DD') -> DD/MM/YYYY |
| Days since | daysSince(dateStr) -> number of days from date to today |
| Overdue threshold | 15 days (change the `> 15` checks in renderDashboard + buildSaleCard) |

---

## What Does NOT Exist Yet (Potential Future Features)

- No customer-wise full sales history page
- No invoice / bill PDF generation
- No export to Excel
- No multi-user login (everyone uses same password)
- No delivery route / map view
- No GST tracking
- No supplier management page (supplier info only inside stock entries)

---

## Completed Features (Added Oct 2026)

- Profit/Loss auto-calculation in Reports (collected - stock cost - expenses - chit fund paid)
- Overdue payment detection (>15 days) — red border + badge on sale cards + dashboard alert grouped by customer
- Chit fund deducted in monthly profit/loss (only when marked paid)
- Monthly 1000 Bundles Target tracker on dashboard — resets each month, progress bar, days left in month, daily rate needed
- Current stock plate type breakdown on dashboard — per type: purchased / sold / remaining with colour coding
- Stock multi-order format — multiple orders with plate type items per stock number

---

## Planned — Fast2SMS Integration (Next Priority)

**Goal:** Auto-send SMS payment reminders to customers with pending/overdue dues.

**Platform:** Fast2SMS (fast2sms.com)
- Cost: ~Rs.50/month for typical usage
- Channel: Regular SMS (works without WhatsApp)
- India only

**What it will do:**
- "Send Reminders" button on dashboard
- Loops over all pending/overdue customers
- Sends SMS via Fast2SMS API automatically
- Two message types: normal reminder and urgent overdue (>15 days)
- Language per customer: English or Telugu

**Setup needed before building:**
1. Sign up at fast2sms.com
2. Get API key from Dev API section
3. Complete DLT registration (Indian telecom requirement for bulk SMS)
4. Share API key to add to app

**Reference:** See AUTOMATION_MESSAGING_GUIDE.md for full API docs, code examples, and message templates.

---

## How to Update the App

```bash
cd /mnt/d/personal-projects/business-project
# make changes to index.html
git add index.html
git commit -m "describe what changed"
git push origin main
# wait 1-2 minutes -> live link auto-updates
```

---

## Important Credentials

| Item | Value |
|---|---|
| GitHub Username | ManiKanagarla |
| Repo | plate-manager |
| Firebase Project ID | disposable-business-559f2 |
| App Password | mdh@1210 |
| Live Link | https://manikanagarla.github.io/plate-manager/ |

Keep password and Firebase API key private.
