# WoW Auction Tracker API Documentation

## Overview

The WoW Auction Tracker API provides comprehensive functionality for tracking, analyzing, and managing World of Warcraft auction house data. This API allows developers to fetch auction data, analyze market trends, track item prices, and build auction-related applications.

## Core Concepts

### Auction Data Structure

```typescript
interface AuctionItem {
  id: number;
  itemId: number;
  name: string;
  quantity: number;
  buyoutPrice: number;
  bidPrice: number;
  timeLeft: 'SHORT' | 'MEDIUM' | 'LONG' | 'VERY_LONG';
  seller: string;
  realm: string;
  timestamp: Date;
}

interface MarketAnalysis {
  itemId: number;
  averagePrice: number;
  medianPrice: number;
  minPrice: number;
  maxPrice: number;
  volume: number;
  trend: 'UP' | 'DOWN' | 'STABLE';
  confidence: number;
}
```

## Main API Endpoints

### Auction Data API

#### `GET /api/auctions`

Fetches current auction house data with filtering options.

**Parameters:**
- `realm` (optional): Filter by specific realm
- `itemId` (optional): Filter by item ID
- `minPrice` (optional): Minimum price filter
- `maxPrice` (optional): Maximum price filter
- `limit` (optional): Number of results (default: 100)
- `offset` (optional): Pagination offset

**Response:**
```json
{
  "auctions": [AuctionItem],
  "total": 1500,
  "realm": "Stormrage",
  "lastUpdated": "2024-01-15T10:30:00Z"
}
```

#### `GET /api/auctions/{itemId}`

Fetches auction data for a specific item across all realms.

**Parameters:**
- `realm` (optional): Specific realm filter
- `timeframe` (optional): Analysis timeframe (1h, 24h, 7d, 30d)

**Response:**
```json
{
  "itemId": 12345,
  "itemName": "Epic Sword of Awesomeness",
  "auctions": [AuctionItem],
  "analysis": MarketAnalysis
}
```

#### `GET /api/auctions/{itemId}/history`

Retrieves historical price data for trend analysis.

**Parameters:**
- `days` (optional): Number of days of history (default: 30)

**Response:**
```json
{
  "itemId": 12345,
  "history": [
    {
      "date": "2024-01-15",
      "averagePrice": 150000,
      "volume": 25,
      "minPrice": 120000,
      "maxPrice": 200000
    }
  ]
}
```

### Market Analysis API

#### `GET /api/market/trends`

Provides market trend analysis for multiple items.

**Parameters:**
- `itemIds`: Comma-separated list of item IDs
- `realm`: Target realm
- `timeframe`: Analysis period

**Response:**
```json
{
  "trends": [
    {
      "itemId": 12345,
      "trend": "UP",
      "change": 15.5,
      "confidence": 0.85
    }
  ]
}
```

#### `GET /api/market/opportunities`

Identifies potential profitable opportunities based on market analysis.

**Parameters:**
- `realm`: Target realm
- `minProfit`: Minimum profit threshold
- `maxInvestment`: Maximum investment amount

**Response:**
```json
{
  "opportunities": [
    {
      "itemId": 12345,
      "itemName": "Epic Sword",
      "buyPrice": 150000,
      "sellPrice": 200000,
      "profit": 50000,
      "roi": 33.3,
      "confidence": 0.8
    }
  ]
}
```

### Price Alert API

#### `POST /api/alerts`

Creates a price alert for item monitoring.

**Request Body:**
```json
{
  "itemId": 12345,
  "realm": "Stormrage",
  "alertType": "BELOW" | "ABOVE",
  "price": 150000,
  "webhook": "https://your-webhook-url.com"
}
```

**Response:**
```json
{
  "alertId": "alert_123",
  "status": "ACTIVE",
  "created": "2024-01-15T10:30:00Z"
}
```

#### `GET /api/alerts`

Retrieves all active price alerts.

**Response:**
```json
{
  "alerts": [
    {
      "alertId": "alert_123",
      "itemId": 12345,
      "realm": "Stormrage",
      "alertType": "BELOW",
      "price": 150000,
      "status": "ACTIVE"
    }
  ]
}
```

#### `DELETE /api/alerts/{alertId}`

Deletes a price alert.

**Response:** 204 No Content

## Utility Functions

### Price Calculation Functions

#### `calculateMarketPrice(auctions, method)`

Calculates market price using different methods.

```typescript
function calculateMarketPrice(
  auctions: AuctionItem[],
  method: 'average' | 'median' | 'weighted' | 'trimmed'
): number
```

**Example:**
```typescript
const auctions = [/* auction data */];
const averagePrice = calculateMarketPrice(auctions, 'average');
const medianPrice = calculateMarketPrice(auctions, 'median');
```

#### `analyzeTrend(prices, timeframe)`

Analyzes price trends over time.

```typescript
function analyzeTrend(
  prices: number[],
  timeframe: number
): { trend: 'UP' | 'DOWN' | 'STABLE', strength: number }
```

**Example:**
```typescript
const prices = [100, 105, 102, 110, 115];
const trend = analyzeTrend(prices, 7); // 7-day analysis
console.log(trend); // { trend: 'UP', strength: 0.75 }
```

### Data Processing Functions

#### `filterAuctions(auctions, criteria)`

Filters auction data based on criteria.

```typescript
function filterAuctions(
  auctions: AuctionItem[],
  criteria: {
    minPrice?: number;
    maxPrice?: number;
    seller?: string;
    timeLeft?: string;
  }
): AuctionItem[]
```

#### `aggregateAuctionData(auctions)`

Aggregates auction data by item.

```typescript
function aggregateAuctionData(auctions: AuctionItem[]): Map<number, MarketAnalysis>
```

## Real-time Data Functions

### WebSocket Connection

#### `connectWebSocket(realm, callback)`

Establishes WebSocket connection for real-time auction updates.

```typescript
function connectWebSocket(
  realm: string,
  callback: (data: AuctionUpdate) => void
): WebSocket
```

**Example:**
```typescript
const ws = connectWebSocket('Stormrage', (update) => {
  console.log('New auction:', update);
});
```

### Event Types

```typescript
interface AuctionUpdate {
  type: 'NEW_AUCTION' | 'AUCTION_SOLD' | 'PRICE_CHANGE';
  itemId: number;
  data: AuctionItem;
  timestamp: Date;
}
```

## Error Handling

All API endpoints follow consistent error response format:

```json
{
  "error": {
    "code": "INVALID_ITEM_ID",
    "message": "The provided item ID is not valid",
    "details": {
      "itemId": 999999
    }
  }
}
```

Common HTTP status codes:
- `200`: Success
- `400`: Bad Request
- `401`: Unauthorized
- `404`: Not Found
- `429`: Rate Limited
- `500`: Internal Server Error

## Rate Limits

- Standard API: 1000 requests per hour
- Premium API: 10000 requests per hour
- WebSocket connections: 5 per client

## Authentication

API key authentication via header:
```
X-API-Key: your-api-key-here
```

## Examples

### Basic Auction Fetching

```javascript
const response = await fetch('/api/auctions?realm=Stormrage&limit=50', {
  headers: {
    'X-API-Key': 'your-api-key'
  }
});

const data = await response.json();
console.log(`Found ${data.auctions.length} auctions`);
```

### Price Alert Setup

```javascript
const alert = await fetch('/api/alerts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-API-Key': 'your-api-key'
  },
  body: JSON.stringify({
    itemId: 12345,
    realm: 'Stormrage',
    alertType: 'BELOW',
    price: 150000,
    webhook: 'https://your-server.com/webhook'
  })
});

console.log('Alert created:', alert.alertId);
```

### Market Analysis

```javascript
const analysis = await fetch('/api/market/opportunities?realm=Stormrage&minProfit=10000');
const opportunities = await analysis.json();

opportunities.forEach(opp => {
  console.log(`${opp.itemName}: ${opp.profit} gold profit (${opp.roi}% ROI)`);
});
```