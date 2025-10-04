# WoW Auction Tracker - API Documentation

## Table of Contents

1. [Authentication](#authentication)
2. [Auction House API](#auction-house-api)
3. [Server Management API](#server-management-api)
4. [Item Database API](#item-database-api)
5. [Price History API](#price-history-api)
6. [User Management API](#user-management-api)
7. [Notification API](#notification-api)
8. [Error Handling](#error-handling)

## Authentication

### Overview
All API endpoints require authentication using API keys or OAuth tokens.

### Base URL
```
https://api.wow-auction-tracker.com/v1
```

### Authentication Methods

#### API Key Authentication
```http
GET /api/v1/auctions
Authorization: Bearer YOUR_API_KEY
```

#### OAuth 2.0 Authentication
```http
GET /api/v1/auctions
Authorization: Bearer YOUR_OAUTH_TOKEN
```

### Example Authentication Request
```javascript
const response = await fetch('https://api.wow-auction-tracker.com/v1/auctions', {
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
  }
});
```

## Auction House API

### Get Current Auctions

Retrieves current auction data for a specific realm.

**Endpoint:** `GET /auctions/{realm}`

**Parameters:**
- `realm` (string, required): The realm name or ID
- `item_id` (integer, optional): Filter by specific item ID
- `quality` (string, optional): Filter by item quality (poor, common, uncommon, rare, epic, legendary)
- `min_price` (integer, optional): Minimum price filter in copper
- `max_price` (integer, optional): Maximum price filter in copper
- `limit` (integer, optional): Number of results to return (default: 100, max: 1000)
- `offset` (integer, optional): Pagination offset (default: 0)

**Response:**
```json
{
  "success": true,
  "data": {
    "auctions": [
      {
        "id": 12345,
        "item_id": 19019,
        "item_name": "Thunderfury, Blessed Blade of the Windseeker",
        "quality": "legendary",
        "quantity": 1,
        "unit_price": 1000000,
        "total_price": 1000000,
        "seller": "PlayerName",
        "realm": "stormrage",
        "time_left": "LONG",
        "buyout": 1000000,
        "bid": 800000,
        "created_at": "2023-10-04T15:30:00Z"
      }
    ],
    "total_count": 1,
    "page_info": {
      "has_next_page": false,
      "has_previous_page": false,
      "current_page": 1,
      "total_pages": 1
    }
  }
}
```

**Example Usage:**
```javascript
// Get all auctions for Stormrage realm
const auctions = await fetch('/api/v1/auctions/stormrage');

// Get expensive items only
const expensiveAuctions = await fetch('/api/v1/auctions/stormrage?min_price=100000');

// Get specific item auctions
const thunderfuryAuctions = await fetch('/api/v1/auctions/stormrage?item_id=19019');
```

### Get Auction Statistics

Retrieves statistical data for auctions on a realm.

**Endpoint:** `GET /auctions/{realm}/stats`

**Parameters:**
- `realm` (string, required): The realm name or ID
- `period` (string, optional): Time period for stats (1h, 6h, 24h, 7d, 30d) (default: 24h)

**Response:**
```json
{
  "success": true,
  "data": {
    "realm": "stormrage",
    "period": "24h",
    "total_auctions": 15420,
    "total_value": 125000000,
    "average_price": 8108,
    "most_expensive": {
      "item_id": 19019,
      "item_name": "Thunderfury, Blessed Blade of the Windseeker",
      "price": 1000000
    },
    "most_common": {
      "item_id": 2589,
      "item_name": "Linen Cloth",
      "count": 1250
    }
  }
}
```

## Server Management API

### Get Available Realms

Retrieves list of all available realms.

**Endpoint:** `GET /realms`

**Parameters:**
- `region` (string, optional): Filter by region (us, eu, kr, tw, cn)
- `status` (string, optional): Filter by status (online, offline, maintenance)

**Response:**
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
        "timezone": "America/New_York",
        "status": "online",
        "population": "high",
        "type": "pve",
        "last_updated": "2023-10-04T15:30:00Z"
      }
    ]
  }
}
```

### Get Realm Details

**Endpoint:** `GET /realms/{realm}`

**Response:**
```json
{
  "success": true,
  "data": {
    "realm": {
      "id": 1,
      "name": "Stormrage",
      "slug": "stormrage",
      "region": "us",
      "timezone": "America/New_York",
      "status": "online",
      "population": "high",
      "type": "pve",
      "connected_realms": ["Azgalor", "Azshara", "Destromath"],
      "auction_house_id": 1,
      "last_updated": "2023-10-04T15:30:00Z"
    }
  }
}
```

## Item Database API

### Get Item Information

**Endpoint:** `GET /items/{item_id}`

**Response:**
```json
{
  "success": true,
  "data": {
    "item": {
      "id": 19019,
      "name": "Thunderfury, Blessed Blade of the Windseeker",
      "quality": "legendary",
      "item_level": 80,
      "required_level": 60,
      "item_class": "Weapon",
      "item_subclass": "One-Handed Swords",
      "icon": "inv_sword_39",
      "description": "Blessed Blade of the Windseeker",
      "bind_type": "bind_on_pickup",
      "unique": true,
      "max_stack": 1,
      "sell_price": 51234,
      "stats": {
        "damage": "36 - 68",
        "speed": 1.9,
        "dps": 27.37,
        "attributes": [
          "+5 Agility",
          "+8 Stamina",
          "+5 Fire Resistance",
          "+5 Nature Resistance"
        ]
      }
    }
  }
}
```

### Search Items

**Endpoint:** `GET /items/search`

**Parameters:**
- `q` (string, required): Search query
- `quality` (string, optional): Filter by quality
- `item_class` (string, optional): Filter by item class
- `min_level` (integer, optional): Minimum required level
- `max_level` (integer, optional): Maximum required level
- `limit` (integer, optional): Number of results (default: 20, max: 100)

**Response:**
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
        "icon": "inv_sword_39"
      }
    ],
    "total_count": 1
  }
}
```

## Price History API

### Get Price History

**Endpoint:** `GET /prices/{realm}/{item_id}`

**Parameters:**
- `realm` (string, required): Realm name or ID
- `item_id` (integer, required): Item ID
- `period` (string, optional): Time period (1d, 7d, 30d, 90d) (default: 7d)
- `granularity` (string, optional): Data granularity (1h, 6h, 1d) (default: 1h)

**Response:**
```json
{
  "success": true,
  "data": {
    "item_id": 19019,
    "realm": "stormrage",
    "period": "7d",
    "granularity": "1h",
    "price_history": [
      {
        "timestamp": "2023-10-01T00:00:00Z",
        "min_price": 950000,
        "max_price": 1050000,
        "avg_price": 1000000,
        "median_price": 1000000,
        "volume": 2
      }
    ],
    "summary": {
      "current_price": 1000000,
      "price_change_24h": 50000,
      "price_change_7d": -25000,
      "volatility": 0.05
    }
  }
}
```

### Get Market Trends

**Endpoint:** `GET /trends/{realm}`

**Parameters:**
- `realm` (string, required): Realm name or ID
- `category` (string, optional): Item category filter
- `period` (string, optional): Time period (1d, 7d, 30d) (default: 7d)

**Response:**
```json
{
  "success": true,
  "data": {
    "realm": "stormrage",
    "period": "7d",
    "trending_up": [
      {
        "item_id": 19019,
        "item_name": "Thunderfury, Blessed Blade of the Windseeker",
        "price_change": 0.15,
        "current_price": 1000000
      }
    ],
    "trending_down": [
      {
        "item_id": 2589,
        "item_name": "Linen Cloth",
        "price_change": -0.08,
        "current_price": 50
      }
    ]
  }
}
```

## User Management API

### Create User Profile

**Endpoint:** `POST /users`

**Request Body:**
```json
{
  "username": "player123",
  "email": "player@example.com",
  "realm": "stormrage",
  "character_name": "MyCharacter"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 12345,
      "username": "player123",
      "email": "player@example.com",
      "realm": "stormrage",
      "character_name": "MyCharacter",
      "created_at": "2023-10-04T15:30:00Z",
      "api_key": "your-generated-api-key"
    }
  }
}
```

### Get User Watchlist

**Endpoint:** `GET /users/{user_id}/watchlist`

**Response:**
```json
{
  "success": true,
  "data": {
    "watchlist": [
      {
        "id": 1,
        "item_id": 19019,
        "item_name": "Thunderfury, Blessed Blade of the Windseeker",
        "target_price": 900000,
        "current_price": 1000000,
        "created_at": "2023-10-01T10:00:00Z"
      }
    ]
  }
}
```

### Add Item to Watchlist

**Endpoint:** `POST /users/{user_id}/watchlist`

**Request Body:**
```json
{
  "item_id": 19019,
  "target_price": 900000,
  "notify_on_drop": true
}
```

## Notification API

### Get Notifications

**Endpoint:** `GET /users/{user_id}/notifications`

**Parameters:**
- `status` (string, optional): Filter by status (read, unread, all) (default: all)
- `type` (string, optional): Filter by type (price_alert, auction_ending, new_listing)

**Response:**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": 1,
        "type": "price_alert",
        "title": "Price Drop Alert",
        "message": "Thunderfury price dropped to 900000 copper",
        "item_id": 19019,
        "status": "unread",
        "created_at": "2023-10-04T15:30:00Z"
      }
    ]
  }
}
```

### Create Notification Subscription

**Endpoint:** `POST /users/{user_id}/subscriptions`

**Request Body:**
```json
{
  "type": "price_alert",
  "item_id": 19019,
  "condition": "below",
  "threshold": 900000,
  "channels": ["email", "webhook"]
}
```

## Error Handling

### Error Response Format

All API errors follow a consistent format:

```json
{
  "success": false,
  "error": {
    "code": "INVALID_REALM",
    "message": "The specified realm does not exist",
    "details": {
      "realm": "invalid-realm-name"
    }
  }
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_API_KEY` | 401 | API key is invalid or expired |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INVALID_REALM` | 400 | Realm does not exist |
| `INVALID_ITEM_ID` | 400 | Item ID does not exist |
| `MISSING_PARAMETER` | 400 | Required parameter is missing |
| `INTERNAL_ERROR` | 500 | Internal server error |

### Rate Limiting

- **Free tier**: 100 requests per hour
- **Premium tier**: 1000 requests per hour
- **Enterprise tier**: 10000 requests per hour

Rate limit headers are included in all responses:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1696435800
```

## SDK Examples

### JavaScript/Node.js

```javascript
const WoWAuctionTracker = require('wow-auction-tracker-sdk');

const client = new WoWAuctionTracker({
  apiKey: 'your-api-key',
  baseURL: 'https://api.wow-auction-tracker.com/v1'
});

// Get current auctions
const auctions = await client.auctions.get('stormrage', {
  item_id: 19019,
  limit: 10
});

// Get price history
const priceHistory = await client.prices.getHistory('stormrage', 19019, {
  period: '7d'
});

// Add item to watchlist
await client.users.addToWatchlist(userId, {
  item_id: 19019,
  target_price: 900000
});
```

### Python

```python
from wow_auction_tracker import WoWAuctionTracker

client = WoWAuctionTracker(
    api_key='your-api-key',
    base_url='https://api.wow-auction-tracker.com/v1'
)

# Get current auctions
auctions = client.auctions.get('stormrage', item_id=19019, limit=10)

# Get price history
price_history = client.prices.get_history('stormrage', 19019, period='7d')

# Add item to watchlist
client.users.add_to_watchlist(user_id, {
    'item_id': 19019,
    'target_price': 900000
})
```