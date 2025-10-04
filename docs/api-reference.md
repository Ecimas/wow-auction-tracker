# API Reference

This document provides comprehensive documentation for all public APIs in the WoW Auction Tracker application.

## Base URL

All API endpoints are prefixed with `/api/v1`:

```
https://your-domain.com/api/v1
```

## Authentication

Most endpoints require authentication. Include the JWT token in the Authorization header:

```http
Authorization: Bearer <your-jwt-token>
```

### Getting an API Token

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "your-password"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 1,
      "email": "user@example.com",
      "name": "John Doe"
    }
  }
}
```

## Error Handling

All API responses follow a consistent format:

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation completed successfully"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": { ... }
  }
}
```

### HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Validation Error |
| 429 | Rate Limited |
| 500 | Internal Server Error |

## Endpoints

### Authentication

#### POST /auth/register

Register a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "secure-password",
  "name": "John Doe"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "email": "user@example.com",
      "name": "John Doe",
      "createdAt": "2024-01-01T00:00:00Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### POST /auth/login

Authenticate user and return JWT token.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "secure-password"
}
```

#### POST /auth/refresh

Refresh JWT token.

**Headers:**
```http
Authorization: Bearer <current-token>
```

#### POST /auth/logout

Logout user and invalidate token.

**Headers:**
```http
Authorization: Bearer <token>
```

### Realms

#### GET /realms

Get list of available WoW realms.

**Query Parameters:**
- `region` (optional): Filter by region (us, eu, kr, tw)
- `status` (optional): Filter by realm status (active, inactive)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Aegwynn",
      "slug": "aegwynn",
      "region": "us",
      "type": "pvp",
      "population": "medium",
      "status": "active"
    }
  ]
}
```

#### GET /realms/{id}

Get detailed information about a specific realm.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Aegwynn",
    "slug": "aegwynn",
    "region": "us",
    "type": "pvp",
    "population": "medium",
    "status": "active",
    "timezone": "America/New_York",
    "connectedRealms": [2, 3]
  }
}
```

### Items

#### GET /items

Search for items in the WoW database.

**Query Parameters:**
- `search` (optional): Search term for item name
- `category` (optional): Item category filter
- `quality` (optional): Item quality filter (poor, common, uncommon, rare, epic, legendary)
- `level` (optional): Minimum item level
- `limit` (optional): Number of results (default: 50, max: 100)
- `offset` (optional): Pagination offset

**Example:**
```http
GET /api/v1/items?search=sword&quality=epic&limit=20
```

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 12345,
        "name": "Thunderfury, Blessed Blade of the Windseeker",
        "icon": "inv_sword_39.jpg",
        "quality": "legendary",
        "level": 60,
        "category": "weapon",
        "subcategory": "sword",
        "description": "A legendary sword of immense power..."
      }
    ],
    "pagination": {
      "total": 150,
      "limit": 20,
      "offset": 0,
      "hasMore": true
    }
  }
}
```

#### GET /items/{id}

Get detailed information about a specific item.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 12345,
    "name": "Thunderfury, Blessed Blade of the Windseeker",
    "icon": "inv_sword_39.jpg",
    "quality": "legendary",
    "level": 60,
    "category": "weapon",
    "subcategory": "sword",
    "description": "A legendary sword of immense power...",
    "stats": {
      "damage": "89-134",
      "speed": 1.9,
      "dps": 58.7
    },
    "requirements": {
      "level": 60,
      "class": ["warrior", "paladin", "death-knight"]
    },
    "sources": [
      {
        "type": "quest",
        "name": "The Eye of the Storm"
      }
    ]
  }
}
```

### Auctions

#### GET /auctions

Get auction house data for a specific realm.

**Query Parameters:**
- `realmId` (required): Realm ID
- `itemId` (optional): Filter by specific item
- `minPrice` (optional): Minimum price filter
- `maxPrice` (optional): Maximum price filter
- `sortBy` (optional): Sort field (price, timeLeft, quantity)
- `sortOrder` (optional): Sort order (asc, desc)
- `limit` (optional): Number of results (default: 50, max: 100)

**Example:**
```http
GET /api/v1/auctions?realmId=1&itemId=12345&sortBy=price&sortOrder=asc
```

**Response:**
```json
{
  "success": true,
  "data": {
    "auctions": [
      {
        "id": 67890,
        "itemId": 12345,
        "itemName": "Thunderfury, Blessed Blade of the Windseeker",
        "quantity": 1,
        "unitPrice": 50000,
        "totalPrice": 50000,
        "timeLeft": "2 days",
        "seller": "PlayerName",
        "realmId": 1,
        "realmName": "Aegwynn",
        "createdAt": "2024-01-01T12:00:00Z"
      }
    ],
    "realm": {
      "id": 1,
      "name": "Aegwynn"
    },
    "lastUpdated": "2024-01-01T12:00:00Z"
  }
}
```

#### GET /auctions/history/{itemId}

Get price history for a specific item.

**Query Parameters:**
- `realmId` (optional): Filter by realm
- `period` (optional): Time period (1d, 7d, 30d, 90d, 1y)
- `interval` (optional): Data interval (1h, 6h, 1d)

**Example:**
```http
GET /api/v1/auctions/history/12345?realmId=1&period=30d&interval=1d
```

**Response:**
```json
{
  "success": true,
  "data": {
    "itemId": 12345,
    "itemName": "Thunderfury, Blessed Blade of the Windseeker",
    "realmId": 1,
    "realmName": "Aegwynn",
    "period": "30d",
    "interval": "1d",
    "history": [
      {
        "timestamp": "2024-01-01T00:00:00Z",
        "avgPrice": 45000,
        "minPrice": 40000,
        "maxPrice": 50000,
        "volume": 5
      }
    ],
    "statistics": {
      "avgPrice": 45000,
      "minPrice": 40000,
      "maxPrice": 50000,
      "totalVolume": 150,
      "priceChange": 0.05
    }
  }
}
```

### Watchlists

#### GET /watchlists

Get user's watchlists.

**Headers:**
```http
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Legendary Weapons",
      "description": "Tracking legendary weapon prices",
      "itemCount": 15,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T12:00:00Z"
    }
  ]
}
```

#### POST /watchlists

Create a new watchlist.

**Headers:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "My Watchlist",
  "description": "Items I'm interested in",
  "itemIds": [12345, 67890]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 2,
    "name": "My Watchlist",
    "description": "Items I'm interested in",
    "itemCount": 2,
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### GET /watchlists/{id}

Get detailed watchlist information.

**Headers:**
```http
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Legendary Weapons",
    "description": "Tracking legendary weapon prices",
    "items": [
      {
        "itemId": 12345,
        "itemName": "Thunderfury, Blessed Blade of the Windseeker",
        "currentPrice": 50000,
        "priceChange": 0.05,
        "lastUpdated": "2024-01-01T12:00:00Z"
      }
    ],
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T12:00:00Z"
  }
}
```

#### PUT /watchlists/{id}

Update watchlist.

**Headers:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Updated Watchlist Name",
  "description": "Updated description",
  "itemIds": [12345, 67890, 11111]
}
```

#### DELETE /watchlists/{id}

Delete watchlist.

**Headers:**
```http
Authorization: Bearer <token>
```

### Alerts

#### GET /alerts

Get user's alerts.

**Headers:**
```http
Authorization: Bearer <token>
```

**Query Parameters:**
- `status` (optional): Filter by status (active, triggered, disabled)
- `type` (optional): Filter by alert type (price_drop, price_rise, availability)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Thunderfury Price Alert",
      "type": "price_drop",
      "itemId": 12345,
      "itemName": "Thunderfury, Blessed Blade of the Windseeker",
      "threshold": 40000,
      "condition": "below",
      "status": "active",
      "triggeredAt": null,
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ]
}
```

#### POST /alerts

Create a new alert.

**Headers:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Item Price Alert",
  "type": "price_drop",
  "itemId": 12345,
  "threshold": 40000,
  "condition": "below",
  "realmId": 1,
  "notificationMethods": ["email", "push"]
}
```

#### PUT /alerts/{id}

Update alert.

**Headers:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Updated Alert Name",
  "threshold": 35000,
  "status": "active"
}
```

#### DELETE /alerts/{id}

Delete alert.

**Headers:**
```http
Authorization: Bearer <token>
```

### Analytics

#### GET /analytics/market-trends

Get market trend analysis.

**Query Parameters:**
- `realmId` (optional): Filter by realm
- `category` (optional): Filter by item category
- `period` (optional): Time period (7d, 30d, 90d)

**Response:**
```json
{
  "success": true,
  "data": {
    "period": "30d",
    "realmId": 1,
    "realmName": "Aegwynn",
    "trends": [
      {
        "category": "weapon",
        "avgPriceChange": 0.15,
        "volumeChange": 0.25,
        "topItems": [
          {
            "itemId": 12345,
            "itemName": "Thunderfury, Blessed Blade of the Windseeker",
            "priceChange": 0.20,
            "volume": 50
          }
        ]
      }
    ],
    "summary": {
      "totalItems": 1500,
      "avgPriceChange": 0.08,
      "totalVolume": 10000
    }
  }
}
```

#### GET /analytics/profit-calculator

Calculate potential profit for item flipping.

**Query Parameters:**
- `itemId` (required): Item ID
- `realmId` (required): Realm ID
- `quantity` (optional): Quantity to analyze (default: 1)

**Response:**
```json
{
  "success": true,
  "data": {
    "itemId": 12345,
    "itemName": "Thunderfury, Blessed Blade of the Windseeker",
    "realmId": 1,
    "realmName": "Aegwynn",
    "analysis": {
      "currentPrice": 50000,
      "avgPrice": 45000,
      "potentialProfit": 5000,
      "profitMargin": 0.11,
      "riskLevel": "medium",
      "recommendation": "buy"
    },
    "priceHistory": {
      "minPrice": 40000,
      "maxPrice": 55000,
      "avgPrice": 45000,
      "volatility": 0.15
    }
  }
}
```

## Rate Limiting

API requests are rate limited to prevent abuse:

- **Authenticated users**: 1000 requests per hour
- **Unauthenticated users**: 100 requests per hour
- **Burst limit**: 10 requests per second

Rate limit headers are included in responses:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

## WebSocket Events

Real-time updates are available via WebSocket connection:

### Connection

```javascript
const ws = new WebSocket('wss://your-domain.com/ws');
```

### Events

#### auction.updated
```json
{
  "event": "auction.updated",
  "data": {
    "auctionId": 67890,
    "itemId": 12345,
    "newPrice": 45000,
    "realmId": 1
  }
}
```

#### alert.triggered
```json
{
  "event": "alert.triggered",
  "data": {
    "alertId": 1,
    "itemId": 12345,
    "itemName": "Thunderfury, Blessed Blade of the Windseeker",
    "threshold": 40000,
    "currentPrice": 38000
  }
}
```

## SDKs and Libraries

### JavaScript/TypeScript

```bash
npm install wow-auction-tracker-sdk
```

```javascript
import { WoWAuctionTracker } from 'wow-auction-tracker-sdk';

const client = new WoWAuctionTracker({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.your-domain.com/v1'
});

// Get auctions
const auctions = await client.auctions.list({
  realmId: 1,
  itemId: 12345
});

// Create alert
const alert = await client.alerts.create({
  name: 'Price Alert',
  type: 'price_drop',
  itemId: 12345,
  threshold: 40000
});
```

### Python

```bash
pip install wow-auction-tracker
```

```python
from wow_auction_tracker import WoWAuctionTracker

client = WoWAuctionTracker(api_key='your-api-key')

# Get auctions
auctions = client.auctions.list(realm_id=1, item_id=12345)

# Create alert
alert = client.alerts.create(
    name='Price Alert',
    type='price_drop',
    item_id=12345,
    threshold=40000
)
```

## Examples

### Complete Workflow Example

```javascript
// 1. Authenticate
const authResponse = await fetch('/api/v1/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password'
  })
});

const { data: { token } } = await authResponse.json();

// 2. Get realms
const realmsResponse = await fetch('/api/v1/realms', {
  headers: { 'Authorization': `Bearer ${token}` }
});

// 3. Search for items
const itemsResponse = await fetch('/api/v1/items?search=sword&quality=epic', {
  headers: { 'Authorization': `Bearer ${token}` }
});

// 4. Get auction data
const auctionsResponse = await fetch('/api/v1/auctions?realmId=1&itemId=12345', {
  headers: { 'Authorization': `Bearer ${token}` }
});

// 5. Create watchlist
const watchlistResponse = await fetch('/api/v1/watchlists', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'My Items',
    itemIds: [12345, 67890]
  })
});

// 6. Set up alert
const alertResponse = await fetch('/api/v1/alerts', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Price Drop Alert',
    type: 'price_drop',
    itemId: 12345,
    threshold: 40000,
    realmId: 1
  })
});
```

## Support

For API support and questions:

- **Documentation**: This comprehensive guide
- **Issues**: GitHub Issues for bug reports
- **Discussions**: GitHub Discussions for questions
- **Email**: api-support@your-domain.com