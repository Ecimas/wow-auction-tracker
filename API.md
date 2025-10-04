# API Documentation

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [REST API Endpoints](#rest-api-endpoints)
4. [WebSocket API](#websocket-api)
5. [Data Models](#data-models)
6. [Error Handling](#error-handling)
7. [Rate Limiting](#rate-limiting)
8. [Code Examples](#code-examples)

---

## Overview

The WoW Auction Tracker API provides programmatic access to World of Warcraft auction house data, including real-time price tracking, historical data analysis, and market trends.

**Base URL:** `https://api.wow-auction-tracker.com/v1`

**API Version:** 1.0.0

---

## Authentication

All API requests require authentication using an API key.

### Getting an API Key

1. Register for an account at https://wow-auction-tracker.com/register
2. Navigate to your account settings
3. Generate a new API key

### Using Your API Key

Include your API key in the request header:

```http
Authorization: Bearer YOUR_API_KEY
```

### Example Request

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/auctions
```

---

## REST API Endpoints

### Auctions

#### Get All Auctions

Retrieve a list of current auctions for a specific realm.

**Endpoint:** `GET /auctions`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `realm` | string | Yes | Realm name (e.g., "stormrage") |
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 100, max: 500) |
| `item_id` | integer | No | Filter by specific item ID |
| `min_price` | integer | No | Minimum buyout price (in copper) |
| `max_price` | integer | No | Maximum buyout price (in copper) |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/auctions?realm=stormrage&item_id=19019&limit=50"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "auctions": [
      {
        "id": "1234567890",
        "item": {
          "id": 19019,
          "name": "Thunderfury, Blessed Blade of the Windseeker",
          "quality": "legendary",
          "item_level": 80
        },
        "buyout": 50000000,
        "bid": 45000000,
        "quantity": 1,
        "time_left": "LONG",
        "seller": "SellerName",
        "realm": "Stormrage"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 156,
      "total_pages": 4
    }
  }
}
```

#### Get Auction by ID

Retrieve details for a specific auction.

**Endpoint:** `GET /auctions/:id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Auction ID |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/auctions/1234567890
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "id": "1234567890",
    "item": {
      "id": 19019,
      "name": "Thunderfury, Blessed Blade of the Windseeker",
      "quality": "legendary",
      "item_level": 80,
      "icon": "inv_sword_39.jpg"
    },
    "buyout": 50000000,
    "bid": 45000000,
    "quantity": 1,
    "time_left": "LONG",
    "seller": "SellerName",
    "realm": "Stormrage",
    "created_at": "2025-10-04T12:00:00Z"
  }
}
```

---

### Items

#### Get Item Information

Retrieve detailed information about a specific item.

**Endpoint:** `GET /items/:id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | Yes | Item ID |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/items/19019
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "id": 19019,
    "name": "Thunderfury, Blessed Blade of the Windseeker",
    "quality": "legendary",
    "item_level": 80,
    "required_level": 60,
    "item_class": "weapon",
    "item_subclass": "sword",
    "icon": "inv_sword_39.jpg",
    "description": "Chance on hit: Blasts your enemy with lightning...",
    "sell_price": 0,
    "is_auctionable": true
  }
}
```

#### Search Items

Search for items by name or other criteria.

**Endpoint:** `GET /items/search`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | Yes | Search query |
| `quality` | string | No | Filter by quality (poor, common, uncommon, rare, epic, legendary) |
| `item_class` | string | No | Filter by item class |
| `min_level` | integer | No | Minimum required level |
| `max_level` | integer | No | Maximum required level |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/items/search?q=thunderfury&quality=legendary"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 19019,
        "name": "Thunderfury, Blessed Blade of the Windseeker",
        "quality": "legendary",
        "item_level": 80,
        "icon": "inv_sword_39.jpg"
      }
    ],
    "total": 1
  }
}
```

---

### Price History

#### Get Price History

Retrieve historical pricing data for an item on a specific realm.

**Endpoint:** `GET /prices/history`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `item_id` | integer | Yes | Item ID |
| `realm` | string | Yes | Realm name |
| `start_date` | string | No | Start date (ISO 8601 format) |
| `end_date` | string | No | End date (ISO 8601 format) |
| `interval` | string | No | Data interval (hour, day, week) (default: day) |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/prices/history?item_id=19019&realm=stormrage&interval=day"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "item_id": 19019,
    "realm": "Stormrage",
    "interval": "day",
    "prices": [
      {
        "timestamp": "2025-10-01T00:00:00Z",
        "min_buyout": 45000000,
        "avg_buyout": 50000000,
        "max_buyout": 55000000,
        "quantity": 3
      },
      {
        "timestamp": "2025-10-02T00:00:00Z",
        "min_buyout": 46000000,
        "avg_buyout": 51000000,
        "max_buyout": 56000000,
        "quantity": 2
      }
    ]
  }
}
```

#### Get Current Market Price

Get the current market price statistics for an item.

**Endpoint:** `GET /prices/current`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `item_id` | integer | Yes | Item ID |
| `realm` | string | Yes | Realm name |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/prices/current?item_id=19019&realm=stormrage"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "item_id": 19019,
    "realm": "Stormrage",
    "market_value": 50000000,
    "min_buyout": 45000000,
    "avg_buyout": 50000000,
    "max_buyout": 55000000,
    "quantity_available": 3,
    "daily_sold": 1,
    "sale_rate": 0.33,
    "last_updated": "2025-10-04T12:00:00Z"
  }
}
```

---

### Alerts

#### Create Price Alert

Create an alert for when an item reaches a specific price.

**Endpoint:** `POST /alerts`

**Request Body:**

```json
{
  "item_id": 19019,
  "realm": "Stormrage",
  "trigger_type": "below",
  "target_price": 45000000,
  "notification_method": "email"
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `item_id` | integer | Yes | Item ID to track |
| `realm` | string | Yes | Realm name |
| `trigger_type` | string | Yes | Alert trigger (below, above) |
| `target_price` | integer | Yes | Target price in copper |
| `notification_method` | string | Yes | Notification method (email, webhook, push) |
| `webhook_url` | string | Conditional | Required if notification_method is webhook |

**Example Request:**

```bash
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"item_id":19019,"realm":"Stormrage","trigger_type":"below","target_price":45000000,"notification_method":"email"}' \
  https://api.wow-auction-tracker.com/v1/alerts
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "alert_id": "alert_123456",
    "item_id": 19019,
    "realm": "Stormrage",
    "trigger_type": "below",
    "target_price": 45000000,
    "notification_method": "email",
    "status": "active",
    "created_at": "2025-10-04T12:00:00Z"
  }
}
```

#### Get User Alerts

Retrieve all alerts for the authenticated user.

**Endpoint:** `GET /alerts`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter by status (active, triggered, disabled) |
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 50) |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/alerts?status=active"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "alerts": [
      {
        "alert_id": "alert_123456",
        "item_id": 19019,
        "item_name": "Thunderfury, Blessed Blade of the Windseeker",
        "realm": "Stormrage",
        "trigger_type": "below",
        "target_price": 45000000,
        "current_price": 50000000,
        "status": "active",
        "created_at": "2025-10-04T12:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 1,
      "total_pages": 1
    }
  }
}
```

#### Delete Alert

Delete a specific alert.

**Endpoint:** `DELETE /alerts/:id`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Alert ID |

**Example Request:**

```bash
curl -X DELETE -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/alerts/alert_123456
```

**Example Response:**

```json
{
  "success": true,
  "message": "Alert deleted successfully"
}
```

---

### Realms

#### Get All Realms

Retrieve a list of all available realms.

**Endpoint:** `GET /realms`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `region` | string | No | Filter by region (us, eu, kr, tw, cn) |
| `locale` | string | No | Filter by locale |

**Example Request:**

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/realms?region=us"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "realms": [
      {
        "id": 1,
        "name": "Stormrage",
        "slug": "stormrage",
        "region": "us",
        "locale": "en_US",
        "timezone": "America/New_York",
        "connected_realm_id": 1,
        "population": "high",
        "type": "pve"
      }
    ],
    "total": 250
  }
}
```

---

## WebSocket API

Real-time auction updates via WebSocket connection.

**WebSocket URL:** `wss://ws.wow-auction-tracker.com`

### Connection

```javascript
const ws = new WebSocket('wss://ws.wow-auction-tracker.com');

ws.onopen = () => {
  // Authenticate
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'YOUR_API_KEY'
  }));
};
```

### Subscribe to Auction Updates

```javascript
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'auctions',
  realm: 'stormrage',
  item_id: 19019 // Optional: filter by item
}));
```

### Message Format

**Incoming Messages:**

```json
{
  "type": "auction_update",
  "action": "created",
  "data": {
    "id": "1234567890",
    "item_id": 19019,
    "buyout": 50000000,
    "quantity": 1,
    "time_left": "LONG",
    "realm": "Stormrage"
  },
  "timestamp": "2025-10-04T12:00:00Z"
}
```

**Action Types:**
- `created` - New auction listed
- `updated` - Auction bid updated
- `sold` - Auction completed
- `expired` - Auction expired

### Unsubscribe

```javascript
ws.send(JSON.stringify({
  type: 'unsubscribe',
  channel: 'auctions',
  realm: 'stormrage'
}));
```

---

## Data Models

### Auction

```typescript
interface Auction {
  id: string;
  item: Item;
  buyout: number;        // Price in copper
  bid: number;           // Current bid in copper
  quantity: number;
  time_left: TimeLeft;
  seller: string;
  realm: string;
  created_at: string;    // ISO 8601 timestamp
}

type TimeLeft = "SHORT" | "MEDIUM" | "LONG" | "VERY_LONG";
```

### Item

```typescript
interface Item {
  id: number;
  name: string;
  quality: Quality;
  item_level: number;
  required_level?: number;
  item_class: string;
  item_subclass: string;
  icon: string;
  description?: string;
  sell_price: number;
  is_auctionable: boolean;
}

type Quality = "poor" | "common" | "uncommon" | "rare" | "epic" | "legendary" | "artifact";
```

### PriceHistory

```typescript
interface PriceHistory {
  timestamp: string;     // ISO 8601 timestamp
  min_buyout: number;
  avg_buyout: number;
  max_buyout: number;
  quantity: number;
}
```

### Alert

```typescript
interface Alert {
  alert_id: string;
  item_id: number;
  realm: string;
  trigger_type: "below" | "above";
  target_price: number;
  notification_method: "email" | "webhook" | "push";
  webhook_url?: string;
  status: "active" | "triggered" | "disabled";
  created_at: string;
}
```

---

## Error Handling

### Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "INVALID_API_KEY",
    "message": "The provided API key is invalid or has been revoked",
    "details": {}
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_API_KEY` | 401 | Invalid or missing API key |
| `UNAUTHORIZED` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `VALIDATION_ERROR` | 400 | Invalid request parameters |
| `REALM_NOT_FOUND` | 404 | Specified realm does not exist |
| `ITEM_NOT_FOUND` | 404 | Specified item does not exist |
| `INTERNAL_ERROR` | 500 | Internal server error |

---

## Rate Limiting

API requests are rate-limited to ensure fair usage.

### Limits

| Tier | Requests per Minute | Requests per Day |
|------|---------------------|------------------|
| Free | 60 | 1,000 |
| Basic | 300 | 10,000 |
| Pro | 1,000 | 100,000 |
| Enterprise | Custom | Custom |

### Rate Limit Headers

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1633363200
```

### Handling Rate Limits

When you exceed the rate limit, you'll receive a 429 status code:

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please wait before making more requests.",
    "details": {
      "retry_after": 30
    }
  }
}
```

---

## Code Examples

### Python

```python
import requests
from datetime import datetime, timedelta

class WoWAuctionTracker:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.wow-auction-tracker.com/v1"
        self.headers = {"Authorization": f"Bearer {api_key}"}
    
    def get_auctions(self, realm, item_id=None, limit=100):
        """Get current auctions for a realm"""
        params = {"realm": realm, "limit": limit}
        if item_id:
            params["item_id"] = item_id
        
        response = requests.get(
            f"{self.base_url}/auctions",
            headers=self.headers,
            params=params
        )
        response.raise_for_status()
        return response.json()
    
    def get_price_history(self, item_id, realm, days=30):
        """Get price history for an item"""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days)
        
        params = {
            "item_id": item_id,
            "realm": realm,
            "start_date": start_date.isoformat(),
            "end_date": end_date.isoformat(),
            "interval": "day"
        }
        
        response = requests.get(
            f"{self.base_url}/prices/history",
            headers=self.headers,
            params=params
        )
        response.raise_for_status()
        return response.json()
    
    def create_alert(self, item_id, realm, trigger_type, target_price):
        """Create a price alert"""
        data = {
            "item_id": item_id,
            "realm": realm,
            "trigger_type": trigger_type,
            "target_price": target_price,
            "notification_method": "email"
        }
        
        response = requests.post(
            f"{self.base_url}/alerts",
            headers=self.headers,
            json=data
        )
        response.raise_for_status()
        return response.json()

# Usage example
tracker = WoWAuctionTracker("YOUR_API_KEY")

# Get auctions for Thunderfury on Stormrage
auctions = tracker.get_auctions("stormrage", item_id=19019)
print(f"Found {len(auctions['data']['auctions'])} auctions")

# Get 30-day price history
history = tracker.get_price_history(19019, "stormrage", days=30)
prices = history['data']['prices']
print(f"Average price: {sum(p['avg_buyout'] for p in prices) / len(prices)}")

# Create alert when price drops below 45M gold
alert = tracker.create_alert(19019, "stormrage", "below", 45000000)
print(f"Alert created: {alert['data']['alert_id']}")
```

### JavaScript/Node.js

```javascript
const axios = require('axios');

class WoWAuctionTracker {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.wow-auction-tracker.com/v1';
    this.headers = { Authorization: `Bearer ${apiKey}` };
  }

  async getAuctions(realm, itemId = null, limit = 100) {
    const params = { realm, limit };
    if (itemId) params.item_id = itemId;

    const response = await axios.get(`${this.baseUrl}/auctions`, {
      headers: this.headers,
      params
    });
    return response.data;
  }

  async getPriceHistory(itemId, realm, days = 30) {
    const endDate = new Date();
    const startDate = new Date(endDate.getTime() - days * 24 * 60 * 60 * 1000);

    const params = {
      item_id: itemId,
      realm,
      start_date: startDate.toISOString(),
      end_date: endDate.toISOString(),
      interval: 'day'
    };

    const response = await axios.get(`${this.baseUrl}/prices/history`, {
      headers: this.headers,
      params
    });
    return response.data;
  }

  async createAlert(itemId, realm, triggerType, targetPrice) {
    const data = {
      item_id: itemId,
      realm,
      trigger_type: triggerType,
      target_price: targetPrice,
      notification_method: 'email'
    };

    const response = await axios.post(`${this.baseUrl}/alerts`, data, {
      headers: this.headers
    });
    return response.data;
  }
}

// Usage example
(async () => {
  const tracker = new WoWAuctionTracker('YOUR_API_KEY');

  // Get auctions
  const auctions = await tracker.getAuctions('stormrage', 19019);
  console.log(`Found ${auctions.data.auctions.length} auctions`);

  // Get price history
  const history = await tracker.getPriceHistory(19019, 'stormrage', 30);
  const avgPrice = history.data.prices.reduce((sum, p) => sum + p.avg_buyout, 0) / history.data.prices.length;
  console.log(`Average price: ${avgPrice}`);

  // Create alert
  const alert = await tracker.createAlert(19019, 'stormrage', 'below', 45000000);
  console.log(`Alert created: ${alert.data.alert_id}`);
})();
```

### cURL Examples

```bash
# Get auctions for a specific item
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/auctions?realm=stormrage&item_id=19019&limit=50"

# Get item information
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/items/19019

# Get price history
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/prices/history?item_id=19019&realm=stormrage&interval=day"

# Create price alert
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"item_id":19019,"realm":"Stormrage","trigger_type":"below","target_price":45000000,"notification_method":"email"}' \
  https://api.wow-auction-tracker.com/v1/alerts

# Get all alerts
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/alerts

# Delete an alert
curl -X DELETE -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.wow-auction-tracker.com/v1/alerts/alert_123456
```

---

## Support

For additional support or questions:

- Documentation: https://docs.wow-auction-tracker.com
- Email: support@wow-auction-tracker.com
- Discord: https://discord.gg/wow-auction-tracker
- GitHub Issues: https://github.com/wow-auction-tracker/issues

---

**Last Updated:** October 4, 2025
