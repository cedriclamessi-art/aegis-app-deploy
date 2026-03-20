# 🤖 Twin Integration Guide

## Overview

Twin.so is integrated into AEGIS SUPREMATIE as the **execution layer** for external operations. This document explains how Twin works with AEGIS and how to use it.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────┐
│      AEGIS (Brain)                  │
│  - Scoring & Decisions              │
│  - Risk Management                  │
│  - Scaling Logic                    │
│  - Data Validation                  │
└────────────────┬────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────┐
│   Execution Router                  │
│  IF API → API                       │
│  ELSE IF Internal → AEGIS           │
│  ELSE → Twin (fallback)             │
└────────────────┬────────────────────┘
                 │
        ┌────────┴────────┬────────┐
        ↓                 ↓        ↓
     API              AEGIS    Twin
   (Direct)          (Core)   (Ops)
```

---

## ✅ What Twin DOES

Twin is used for **execution only**:

- ✅ Navigate websites (without API)
- ✅ Collect competitor data
- ✅ Interact with Ads dashboards
- ✅ Publish content to platforms
- ✅ Automate complex workflows
- ✅ Extract data from HTML
- ✅ Fill forms
- ✅ Take screenshots

---

## ❌ What Twin DOES NOT DO

Twin **never** handles:

- ❌ Strategy decisions
- ❌ Budget allocation
- ❌ Scaling decisions
- ❌ Product scoring
- ❌ Business logic
- ❌ Critical data
- ❌ Financial calculations
- ❌ Risk management

**→ All of this stays in AEGIS**

---

## 🔄 Execution Flow

### Example: Product Analysis

```
1. User submits product URL
   ↓
2. AEGIS receives request
   ├─ Validates URL
   ├─ Checks if API available
   └─ Decides execution method
   ↓
3. Execution Router decides:
   - Amazon API available? → Use API
   - No API? → Use Twin
   ↓
4. Twin executes (if needed):
   ├─ Navigate to product page
   ├─ Extract data (price, reviews, images)
   └─ Return raw data
   ↓
5. AEGIS processes data:
   ├─ Validate data quality
   ├─ Calculate scores
   ├─ Assess risks
   └─ Return analysis
   ↓
6. User receives result
```

---

## 🔌 API Endpoints

### Execute Twin Action

**Endpoint**: `POST /api/twin/execute`

**Request**:
```json
{
  "action": "scrape_product",
  "params": {
    "url": "https://amazon.com/dp/B123456789",
    "fields": ["price", "rating", "reviews"]
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "price": 29.99,
    "rating": 4.5,
    "reviews": 1234
  },
  "executedBy": "twin",
  "duration": 3200
}
```

### Publish Content

**Endpoint**: `POST /api/twin/publish`

**Request**:
```json
{
  "platform": "facebook",
  "content": {
    "image": "https://cdn.../image.jpg",
    "copy": "Limited time offer...",
    "cta": "Buy Now"
  },
  "targeting": {
    "age": [18, 65],
    "interests": ["shopping", "deals"]
  }
}
```

**Response**:
```json
{
  "success": true,
  "adId": "123456789",
  "platform": "facebook",
  "status": "published",
  "executedBy": "twin"
}
```

### Collect Data

**Endpoint**: `POST /api/twin/collect`

**Request**:
```json
{
  "sources": [
    {
      "type": "website",
      "url": "https://competitor.com"
    },
    {
      "type": "google_trends",
      "query": "wireless earbuds"
    }
  ]
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "website": {...},
    "trends": {...}
  },
  "executedBy": "twin"
}
```

---

## 🎯 Twin Capabilities

### Data Collection

```typescript
// Scrape product data
twin.scrape({
  url: "https://amazon.com/...",
  selectors: {
    price: ".a-price",
    rating: ".a-star-small",
    reviews: ".a-count"
  }
})

// Extract competitor info
twin.extract({
  url: "https://competitor.com",
  fields: ["products", "pricing", "reviews"]
})

// Collect trends
twin.trends({
  source: "google_trends",
  query: "product name",
  region: "US"
})
```

### Content Publishing

```typescript
// Publish to Facebook
twin.publish({
  platform: "facebook",
  content: {
    image: "...",
    copy: "...",
    cta: "..."
  }
})

// Post to Instagram
twin.post({
  platform: "instagram",
  content: {
    image: "...",
    caption: "..."
  }
})

// Send email
twin.email({
  to: "customer@example.com",
  subject: "...",
  body: "..."
})
```

### Workflow Automation

```typescript
// Fill form
twin.fillForm({
  url: "https://form.example.com",
  fields: {
    name: "John Doe",
    email: "john@example.com"
  }
})

// Navigate and interact
twin.navigate({
  url: "https://ads.facebook.com",
  actions: [
    { type: "click", selector: ".create-campaign" },
    { type: "fill", selector: "input[name=name]", value: "Campaign 1" },
    { type: "click", selector: ".submit" }
  ]
})

// Take screenshot
twin.screenshot({
  url: "https://example.com",
  format: "png"
})
```

---

## 🔐 Security & Limits

### Rate Limiting

```
- Radar tier: 100 Twin actions/day
- Brand tier: 500 Twin actions/day
- Executive tier: 2,000 Twin actions/day
- Suprématie tier: 10,000 Twin actions/day
```

### Timeout

```
- Default: 30 seconds
- Max: 5 minutes
- Retry: 3 times
```

### Error Handling

```typescript
try {
  const result = await twin.execute(action);
} catch (error) {
  if (error.code === 'TIMEOUT') {
    // Retry or fallback
  } else if (error.code === 'RATE_LIMIT') {
    // Queue for later
  } else if (error.code === 'INVALID_ACTION') {
    // Use AEGIS instead
  }
}
```

---

## 🧠 Decision Logic

### When to Use Twin

```typescript
if (action === 'scrape_data') {
  // Check if API available
  if (hasAPI(source)) {
    return executeViaAPI();  // Faster, more reliable
  } else {
    return executeTwin();    // Fallback
  }
}

if (action === 'publish_content') {
  // Check if API available
  if (hasAPI(platform)) {
    return executeViaAPI();  // Direct integration
  } else {
    return executeTwin();    // UI automation
  }
}

if (action === 'calculate_score') {
  return executeAEGIS();     // Never Twin
}

if (action === 'scale_campaign') {
  return executeAEGIS();     // Never Twin
}
```

---

## 📊 Execution Stats

### Typical Performance

```
Action Type          | Avg Duration | Success Rate | Preferred Method
─────────────────────┼──────────────┼──────────────┼─────────────────
Scrape product       | 3-5s         | 95%          | Twin
Collect trends       | 5-8s         | 90%          | Twin
Publish ad           | 2-3s         | 98%          | Twin
Extract reviews      | 4-6s         | 92%          | Twin
Calculate score      | <100ms       | 99.9%        | AEGIS
Scaling decision     | <50ms        | 99.9%        | AEGIS
Risk assessment      | <100ms       | 99.9%        | AEGIS
```

---

## 🚨 Error Handling

### Common Errors

```
Error: TWIN_TIMEOUT
→ Action took too long
→ Solution: Retry or use API

Error: TWIN_BLOCKED
→ Website blocked Twin
→ Solution: Use API or manual review

Error: TWIN_INVALID_SELECTOR
→ Website structure changed
→ Solution: Update selectors

Error: TWIN_RATE_LIMIT
→ Too many requests
→ Solution: Queue or upgrade tier
```

---

## 🔄 Integration Examples

### Example 1: Product Analysis

```typescript
// src/pages/api/products/analyze.ts

export default async function handler(req, res) {
  const { url } = req.body;
  const user = verifyToken(req.headers.authorization);
  
  // Step 1: AEGIS validates URL
  if (!isValidProductURL(url)) {
    return res.status(400).json({ error: 'Invalid URL' });
  }
  
  // Step 2: Execution Router decides
  let productData;
  if (hasAPI(url)) {
    productData = await executeViaAPI(url);
  } else {
    // Step 3: Twin collects data
    productData = await twin.execute('scrape_product', { url });
  }
  
  // Step 4: AEGIS processes
  const analysis = await aegis.analyzeProduct(productData);
  
  // Step 5: Return result
  res.status(200).json(analysis);
}
```

### Example 2: Campaign Publishing

```typescript
// src/pages/api/campaigns/publish.ts

export default async function handler(req, res) {
  const { campaignId } = req.body;
  const user = verifyToken(req.headers.authorization);
  
  // Step 1: AEGIS validates campaign
  const campaign = await db.campaign.findUnique({ where: { id: campaignId } });
  if (!campaign) {
    return res.status(404).json({ error: 'Campaign not found' });
  }
  
  // Step 2: AEGIS checks guardrails
  const validation = await aegis.validateCampaign(campaign);
  if (!validation.approved) {
    return res.status(400).json({ error: validation.reason });
  }
  
  // Step 3: Twin publishes
  const result = await twin.execute('publish_campaign', {
    campaign,
    platforms: ['facebook', 'instagram']
  });
  
  // Step 4: AEGIS records
  await db.campaign.update({
    where: { id: campaignId },
    data: { status: 'published', publishedAt: new Date() }
  });
  
  res.status(200).json(result);
}
```

---

## 📈 Monitoring

### Twin Health

```bash
# Check Twin status
curl http://localhost:3000/api/twin/health

# View Twin metrics
curl http://localhost:3000/api/twin/metrics

# Check execution logs
curl http://localhost:3000/api/twin/logs?limit=100
```

### Metrics

```json
{
  "status": "healthy",
  "uptime": 99.9,
  "actionsExecuted": 15234,
  "successRate": 94.2,
  "avgDuration": 3500,
  "errors": {
    "timeout": 234,
    "blocked": 45,
    "invalid": 12
  }
}
```

---

## 🎓 Best Practices

### ✅ DO

- ✅ Use Twin for data collection
- ✅ Use Twin for content publishing
- ✅ Use Twin as fallback when API unavailable
- ✅ Monitor Twin execution
- ✅ Handle Twin errors gracefully
- ✅ Use AEGIS for decisions

### ❌ DON'T

- ❌ Use Twin for scoring/decisions
- ❌ Trust Twin data without validation
- ❌ Rely on Twin for critical operations
- ❌ Skip AEGIS validation
- ❌ Expose Twin directly to users
- ❌ Use Twin for financial calculations

---

## 🔗 Related Documentation

- [AEGIS_PRESENTATION_CORRECTED.md](./AEGIS_PRESENTATION_CORRECTED.md)
- [DEVELOPMENT_GUIDE.md](./DEVELOPMENT_GUIDE.md)
- [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)

---

*Last Updated: 2026-03-19*
