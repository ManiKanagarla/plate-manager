# Automation Messaging Guide
# WhatsApp, SMS & Notification Platforms — How They Work

> Reference doc for learning how to integrate messaging automation into any future project.
> Written based on business-project needs (Oct 2026).

---

## The Big Picture — How Messaging Automation Works

```
Your App  →  Messaging Platform API  →  Customer's Phone
(index.html)   (Fast2SMS / Interakt)     (SMS / WhatsApp)
```

Your app sends an API request (HTTP POST) with:
- Phone number
- Message text
- Your API key (for authentication)

The platform delivers the message to the customer. That's it.

---

## Platform 1 — Fast2SMS (SMS)

### What it is
Indian bulk SMS platform. Cheapest option for sending text messages to any mobile number in India.
No WhatsApp needed — goes as a regular SMS.

### Pricing
- Free signup with some credits to test
- ~Rs.10 for 50 SMS (Rs.0.20 per SMS)
- ~Rs.50/month for typical small business usage

### How to Set Up
1. Sign up at fast2sms.com
2. Go to Dev API section → copy your API Key
3. Add the key to your app (store it safely)

### How the API Works

**Endpoint:**
```
POST https://www.fast2sms.com/dev/bulkV2
```

**Headers:**
```
authorization: YOUR_API_KEY
Content-Type: application/json
```

**Request Body:**
```json
{
  "route": "q",
  "message": "Hi Ramu, your payment of Rs.2000 is pending. Please pay soon. - Manjunadha Disposable Hub",
  "language": "english",
  "flash": 0,
  "numbers": "9876543210"
}
```

**Multiple numbers** (comma separated):
```json
{
  "numbers": "9876543210,9123456789,9000001111"
}
```

**Response (success):**
```json
{
  "return": true,
  "request_id": "abc123",
  "message": ["SMS sent successfully."]
}
```

**JavaScript fetch example:**
```javascript
async function sendSMS(phone, message) {
    const response = await fetch('https://www.fast2sms.com/dev/bulkV2', {
        method: 'POST',
        headers: {
            'authorization': 'YOUR_API_KEY_HERE',
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            route: 'q',
            message: message,
            language: 'english',
            flash: 0,
            numbers: phone
        })
    });
    const result = await response.json();
    return result;
}

// Usage
sendSMS('9876543210', 'Hi Ramu, Rs.2000 pending. Please pay. - MDH');
```

### Limitations
- India only (Indian phone numbers)
- SMS character limit: 160 chars per SMS (multiple SMS auto-joined)
- DLT registration required for sending promotional SMS (Indian regulation since 2021)
  - DLT = Distributed Ledger Technology — government portal for SMS senders
  - Register at: trai.gov.in or through Fast2SMS itself
  - Transactional SMS (OTP, alerts) may not need full DLT
- No images — plain text only

### DLT Registration (Important for India)
Indian telecom rules require all businesses sending bulk SMS to register:
1. Register on DLT portal (Jio, Airtel, or Vi portal — pick one)
2. Register your Sender ID (e.g., MNDHDH)
3. Register your message templates
4. Link DLT to Fast2SMS account
This is a one-time setup. Takes 2–7 days approval.

---

## Platform 2 — Interakt (WhatsApp Business API)

### What it is
Official WhatsApp Business API provider approved by Meta. Sends real WhatsApp messages — same as someone texting you on WhatsApp, but automated.

### Pricing
- Rs.999/month flat plan (as of 2026)
- Includes limited free conversations per month
- Per-message charges after limit (Meta charges ~Rs.0.40–0.80 per conversation)

### How to Set Up
1. Sign up at interakt.shop
2. Connect your WhatsApp Business number (must be a number not already on personal WhatsApp)
3. Create message templates in Interakt dashboard — Meta must approve them (1–3 days)
4. Get your API Key from Interakt

### How Templates Work
WhatsApp Business API does NOT allow free-form messages to customers who haven't messaged you first.
You must use pre-approved templates:

**Example template:**
```
Hi {{1}}, your payment of Rs.{{2}} is pending with Manjunadha Disposable Hub.
Please clear at your earliest. Thank you.
```
`{{1}}` and `{{2}}` are variables filled at send time.

### How the API Works

**Endpoint:**
```
POST https://api.interakt.ai/v1/public/message/
```

**Headers:**
```
Authorization: Basic YOUR_BASE64_API_KEY
Content-Type: application/json
```

**Request Body:**
```json
{
  "countryCode": "+91",
  "phoneNumber": "9876543210",
  "callbackData": "some_id",
  "type": "Template",
  "template": {
    "name": "payment_reminder",
    "languageCode": "en",
    "bodyValues": ["Ramu", "2000"]
  }
}
```

**JavaScript fetch example:**
```javascript
async function sendWhatsApp(phone, customerName, amount) {
    const response = await fetch('https://api.interakt.ai/v1/public/message/', {
        method: 'POST',
        headers: {
            'Authorization': 'Basic ' + btoa('YOUR_API_KEY:'),
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            countryCode: '+91',
            phoneNumber: phone,
            callbackData: 'reminder_' + phone,
            type: 'Template',
            template: {
                name: 'payment_reminder',
                languageCode: 'en',
                bodyValues: [customerName, String(amount)]
            }
        })
    });
    const result = await response.json();
    return result;
}
```

### Limitations
- Template messages only (must be pre-approved by Meta)
- Cannot send to someone who blocked your number
- WhatsApp number used for API cannot be used on personal WhatsApp app simultaneously
- More complex setup than SMS

---

## Platform 3 — MSG91 (SMS + WhatsApp)

### What it is
Indian platform that does both SMS and WhatsApp Business API. One dashboard for both channels.

### Pricing
- SMS: ~Rs.0.15–0.25 per SMS (slightly cheaper than Fast2SMS for bulk)
- WhatsApp: pay per conversation (~Rs.0.50–1.00)
- Monthly plans available

### How the API Works (SMS)

**Endpoint:**
```
POST https://api.msg91.com/api/v5/flow/
```

**Request Body:**
```json
{
  "template_id": "YOUR_TEMPLATE_ID",
  "short_url": "0",
  "realTimeResponse": "1",
  "recipients": [
    {
      "mobiles": "919876543210",
      "name": "Ramu",
      "amount": "2000"
    }
  ]
}
```

### Difference from Fast2SMS
- MSG91 uses template-based SMS (similar to WhatsApp templates)
- Requires more upfront template setup
- Better delivery rates for high volume
- Better suited for larger businesses

---

## Platform 4 — Wati (WhatsApp Business API)

### What it is
Another WhatsApp Business API provider. More feature-rich than Interakt — includes chatbot, bulk broadcast, CRM-like features.

### Pricing
- Starts ~Rs.2,500/month
- Better for businesses needing 2-way conversations (customer replies handled)

### When to use over Interakt
- You want customers to reply and have automated responses
- You need a team inbox (multiple staff handling WhatsApp)
- Higher message volumes

### API Structure
Similar to Interakt — template-based, REST API, JSON.

---

## Platform 5 — Twilio (SMS + WhatsApp — International)

### What it is
Global messaging platform. Works worldwide. More expensive for India but very reliable.

### Pricing
- SMS India: ~$0.0075 per message (~Rs.0.63)
- WhatsApp: ~$0.005 per message (~Rs.0.42) + Meta charges
- Pay as you go, no monthly fee

### When to use
- If your business expands internationally
- If you need very high reliability with SLA guarantees
- For global projects outside India

### API Example (SMS)
```javascript
const twilio = require('twilio');
const client = twilio('ACCOUNT_SID', 'AUTH_TOKEN');

client.messages.create({
    body: 'Hi Ramu, Rs.2000 pending at MDH.',
    from: '+1234567890',  // your Twilio number
    to: '+919876543210'
});
```

---

## Platform 6 — Firebase Cloud Messaging (FCM) — Free

### What it is
Google's push notification service. Sends browser/app notifications — the small popups you see from websites.

### Pricing
Completely free.

### How it works
1. User visits your app
2. Browser asks "Allow notifications?" — user clicks Allow
3. FCM gives a unique token for that browser
4. You save that token in Firestore
5. When you want to notify, send to that token via FCM

### Limitation for this business
Customers would need to open your app in their browser and allow notifications.
Not practical for a B2B paper plate delivery business — customers won't do this.
Better suited for apps customers use daily (e-commerce, food delivery).

---

## Comparison Table

| Platform | Channel | Cost/month | Setup Difficulty | India-specific | Best For |
|---|---|---|---|---|---|
| Fast2SMS | SMS | ~Rs.50 | Easy | Yes | Small Indian businesses, quick start |
| Interakt | WhatsApp | Rs.999 | Medium | Yes | WhatsApp-first Indian businesses |
| MSG91 | SMS + WhatsApp | Rs.300+ | Medium | Yes | Medium businesses, both channels |
| Wati | WhatsApp | Rs.2,500 | Medium-Hard | Partial | Teams needing 2-way chat |
| Twilio | SMS + WhatsApp | Variable | Medium | No | International or enterprise |
| FCM | Push notification | Free | Hard | No | Consumer apps with daily users |

---

## How to Store API Keys Safely

Never put API keys directly in frontend JavaScript code — anyone can view source and steal them.

### Option A — Firebase environment (for this app)
Store in Firestore in a private admin document only you can read:
```
Collection: config
Document: secrets
Field: fast2sms_key = "your_key_here"
```
Then fetch at runtime (still not fully secure but better than hardcoding).

### Option B — Backend function (most secure)
Use Firebase Cloud Functions (free tier available):
```javascript
// Cloud Function (runs on Google servers, key never exposed to browser)
exports.sendSMS = functions.https.onCall(async (data, context) => {
    const apiKey = process.env.FAST2SMS_KEY; // stored in Firebase env
    // call Fast2SMS API here
    return result;
});
```
Your app calls the function, function calls Fast2SMS. Key never leaves server.

### Option C — For this app (simplest)
Since only you use this app (password protected, one user), storing the key in the JS is acceptable risk. Just don't share the source code publicly on GitHub.

---

## For the Business Project — Recommended Next Steps

1. **Sign up at fast2sms.com** — free account, test with your own number first
2. **Get API key** from their Dev API section
3. **Test one SMS** using the fetch example above in browser console
4. **Add DLT registration** (Fast2SMS guides you through this)
5. Once working, integrate "Send Reminders" button in index.html
6. Button triggers: loop over pending customers → call Fast2SMS API per customer → done

---

## Message Templates to Use (Ready to Copy)

### Normal Pending Reminder (English)
```
Hi [Name], your payment of Rs.[Amount] is pending with Manjunadha Disposable Hub.
Please pay at your earliest convenience. Thank you.
```

### Overdue Reminder (English — >30 days)
```
Hi [Name], your payment of Rs.[Amount] with Manjunadha Disposable Hub is overdue.
Kindly clear it immediately. Contact: [Your Number].
```

### Normal Pending Reminder (Telugu)
```
Namaskaram [Name] garu, mee payment Rs.[Amount] pending ga undi.
Veelunaipudu chellinchavalani vinnapam. - Manjunadha Disposable Hub
```

### Overdue Reminder (Telugu — >30 days)
```
[Name] garu, mee Rs.[Amount] payment chala rojulga pending ga undi.
Dayachesi venta clear cheyandi. - Manjunadha Disposable Hub
```

---

Last updated: October 2026
