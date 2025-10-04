# WoW Auction Tracker - Usage Examples

## Overview

This document provides practical examples and code samples for using the WoW Auction Tracker API and components. Examples range from basic usage to advanced scenarios.

## Basic API Usage

### Fetching Auction Data

```javascript
// Basic auction fetch
const response = await fetch('/api/auctions?realm=Stormrage&limit=100', {
  headers: {
    'X-API-Key': 'your-api-key-here',
    'Content-Type': 'application/json'
  }
});

const data = await response.json();
console.log(`Found ${data.auctions.length} auctions on Stormrage`);

// Filter by price range
const affordableAuctions = await fetch('/api/auctions?realm=Stormrage&minPrice=100000&maxPrice=500000');
const data = await affordableAuctions.json();

// Search for specific item
const swordAuctions = await fetch('/api/auctions?itemId=12345&realm=Stormrage');
const swords = await swordAuctions.json();
```

### Working with Real-time Data

```javascript
// WebSocket connection for real-time updates
const ws = new WebSocket('wss://api.wowauctiontracker.com/ws');

ws.onopen = () => {
  console.log('Connected to auction feed');

  // Subscribe to specific realm
  ws.send(JSON.stringify({
    type: 'subscribe',
    realm: 'Stormrage'
  }));
};

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);

  switch(update.type) {
    case 'NEW_AUCTION':
      console.log('New auction:', update.data.itemName, 'for', update.data.buyoutPrice, 'gold');
      break;
    case 'AUCTION_SOLD':
      console.log('Auction sold:', update.data.itemName);
      break;
    case 'PRICE_CHANGE':
      console.log('Price updated:', update.data.itemName, 'new price:', update.data.buyoutPrice);
      break;
  }
};
```

## Price Analysis Examples

### Market Analysis

```javascript
// Analyze market trends for an item
async function analyzeItemMarket(itemId, realm = 'Stormrage') {
  try {
    const response = await fetch(`/api/auctions/${itemId}?realm=${realm}&timeframe=7d`);
    const data = await response.json();

    console.log(`Market analysis for item ${data.itemName}:`);
    console.log(`Average price: ${data.analysis.averagePrice} gold`);
    console.log(`Price trend: ${data.analysis.trend} (${data.analysis.confidence * 100}% confidence)`);
    console.log(`Volume: ${data.analysis.volume} items sold`);
    console.log(`Price range: ${data.analysis.minPrice} - ${data.analysis.maxPrice} gold`);

    return data.analysis;
  } catch (error) {
    console.error('Failed to analyze market:', error);
  }
}

// Find profitable opportunities
async function findOpportunities(minProfit = 50000) {
  const response = await fetch(`/api/market/opportunities?minProfit=${minProfit}&realm=Stormrage`);
  const opportunities = await response.json();

  opportunities.forEach(opp => {
    console.log(`${opp.itemName}:`);
    console.log(`  Buy: ${opp.buyPrice}g | Sell: ${opp.sellPrice}g | Profit: ${opp.profit}g (${opp.roi}% ROI)`);
    console.log(`  Confidence: ${opp.confidence * 100}% | Risk: ${opp.risk}`);
  });

  return opportunities;
}
```

### Price Alert Setup

```javascript
// Create a price alert
async function createPriceAlert(itemId, targetPrice, alertType = 'BELOW') {
  const alertConfig = {
    itemId: itemId,
    realm: 'Stormrage',
    alertType: alertType, // 'BELOW' or 'ABOVE'
    price: targetPrice,
    webhook: 'https://your-server.com/webhook/alerts'
  };

  const response = await fetch('/api/alerts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': 'your-api-key'
    },
    body: JSON.stringify(alertConfig)
  });

  const alert = await response.json();
  console.log(`Alert created: ${alert.alertId}`);
  return alert;
}

// Monitor multiple items
async function setupWatchlist(watchlist) {
  const alerts = [];

  for (const item of watchlist) {
    // Create below-market alert for buying
    const buyAlert = await createPriceAlert(item.id, item.targetBuyPrice, 'BELOW');

    // Create above-market alert for selling
    const sellAlert = await createPriceAlert(item.id, item.targetSellPrice, 'ABOVE');

    alerts.push({ item: item.name, buyAlert: buyAlert.alertId, sellAlert: sellAlert.alertId });
  }

  console.log('Watchlist alerts created:', alerts);
  return alerts;
}
```

## React Component Examples

### Using AuctionList Component

```jsx
import React, { useState, useEffect } from 'react';
import { AuctionList } from './components/AuctionList';

function AuctionDashboard() {
  const [auctions, setAuctions] = useState([]);
  const [loading, setLoading] = useState(true);
  const [filters, setFilters] = useState({
    minPrice: 100000,
    maxPrice: 1000000,
    timeLeft: ['MEDIUM', 'LONG']
  });

  useEffect(() => {
    fetchAuctions();
  }, [filters]);

  const fetchAuctions = async () => {
    setLoading(true);
    try {
      const queryParams = new URLSearchParams({
        realm: 'Stormrage',
        minPrice: filters.minPrice.toString(),
        maxPrice: filters.maxPrice.toString(),
        timeLeft: filters.timeLeft.join(','),
        limit: '100'
      });

      const response = await fetch(`/api/auctions?${queryParams}`, {
        headers: { 'X-API-Key': 'your-api-key' }
      });

      const data = await response.json();
      setAuctions(data.auctions);
    } catch (error) {
      console.error('Failed to fetch auctions:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleAuctionSelect = (auction) => {
    console.log('Selected auction:', auction);
    // Navigate to detail view or add to watchlist
  };

  const handleSort = (field, direction) => {
    const sorted = [...auctions].sort((a, b) => {
      const aVal = a[field];
      const bVal = b[field];

      if (direction === 'ASC') {
        return aVal > bVal ? 1 : -1;
      } else {
        return aVal < bVal ? 1 : -1;
      }
    });

    setAuctions(sorted);
  };

  return (
    <div className="auction-dashboard">
      <h1>Stormrage Auction House</h1>

      <AuctionList
        auctions={auctions}
        loading={loading}
        onAuctionSelect={handleAuctionSelect}
        onSort={handleSort}
        filters={filters}
      />
    </div>
  );
}

export default AuctionDashboard;
```

### Market Analysis Dashboard

```jsx
import React, { useState, useEffect } from 'react';
import { MarketAnalysisChart } from './components/MarketAnalysisChart';
import { MarketOverviewWidget } from './components/MarketOverviewWidget';

function MarketAnalysisPage() {
  const [selectedItem, setSelectedItem] = useState(null);
  const [priceHistory, setPriceHistory] = useState(null);
  const [marketOverview, setMarketOverview] = useState(null);

  useEffect(() => {
    if (selectedItem) {
      fetchPriceHistory(selectedItem.id);
    }
    fetchMarketOverview();
  }, [selectedItem]);

  const fetchPriceHistory = async (itemId) => {
    try {
      const response = await fetch(`/api/auctions/${itemId}/history?days=30`, {
        headers: { 'X-API-Key': 'your-api-key' }
      });

      const history = await response.json();
      setPriceHistory(history);
    } catch (error) {
      console.error('Failed to fetch price history:', error);
    }
  };

  const fetchMarketOverview = async () => {
    try {
      const response = await fetch('/api/market/trends?realm=Stormrage&timeframe=24h', {
        headers: { 'X-API-Key': 'your-api-key' }
      });

      const overview = await response.json();
      setMarketOverview(overview);
    } catch (error) {
      console.error('Failed to fetch market overview:', error);
    }
  };

  return (
    <div className="market-analysis">
      <div className="market-header">
        <h1>Market Analysis</h1>
        <MarketOverviewWidget
          realm="Stormrage"
          timeframe={24}
          refreshInterval={300000} // 5 minutes
        />
      </div>

      <div className="analysis-content">
        <div className="item-selector">
          <select onChange={(e) => setSelectedItem({ id: e.target.value })}>
            <option value="">Select an item...</option>
            <option value="12345">Epic Sword of Awesomeness</option>
            <option value="67890">Legendary Staff of Power</option>
          </select>
        </div>

        {priceHistory && (
          <MarketAnalysisChart
            data={priceHistory}
            type="line"
            timeframe="30D"
            indicators={['SMA', 'EMA']}
            onPointClick={(point) => console.log('Clicked point:', point)}
          />
        )}
      </div>
    </div>
  );
}

export default MarketAnalysisPage;
```

## Advanced Usage Examples

### Automated Trading Bot

```javascript
class AuctionBot {
  constructor(apiKey, realm = 'Stormrage') {
    this.apiKey = apiKey;
    this.realm = realm;
    this.watchlist = new Set();
    this.activeAlerts = new Map();
  }

  async start() {
    console.log(`Starting auction bot for ${this.realm}`);

    // Load watchlist from configuration
    await this.loadWatchlist();

    // Set up alerts for watchlist items
    await this.setupAlerts();

    // Start monitoring
    this.startMonitoring();
  }

  async loadWatchlist() {
    // Load from file or database
    this.watchlist = new Set([12345, 67890, 11111]);
  }

  async setupAlerts() {
    for (const itemId of this.watchlist) {
      // Get current market price
      const marketData = await this.getMarketData(itemId);
      const currentPrice = marketData.analysis.averagePrice;

      // Set up buy alert 10% below market
      const buyPrice = Math.floor(currentPrice * 0.9);
      const buyAlert = await this.createAlert(itemId, buyPrice, 'BELOW');

      // Set up sell alert 20% above market
      const sellPrice = Math.floor(currentPrice * 1.2);
      const sellAlert = await this.createAlert(itemId, sellPrice, 'ABOVE');

      this.activeAlerts.set(itemId, { buyAlert, sellAlert, buyPrice, sellPrice });
    }
  }

  async createAlert(itemId, price, type) {
    const response = await fetch('/api/alerts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': this.apiKey
      },
      body: JSON.stringify({
        itemId,
        realm: this.realm,
        alertType: type,
        price,
        webhook: process.env.WEBHOOK_URL
      })
    });

    return await response.json();
  }

  async getMarketData(itemId) {
    const response = await fetch(`/api/auctions/${itemId}?realm=${this.realm}`, {
      headers: { 'X-API-Key': this.apiKey }
    });

    return await response.json();
  }

  startMonitoring() {
    // Set up webhook endpoint to receive alerts
    this.setupWebhookHandler();
  }

  setupWebhookHandler() {
    // Express.js webhook handler
    app.post('/webhook/alerts', (req, res) => {
      const alert = req.body;

      console.log('Alert triggered:', alert);

      switch(alert.alertType) {
        case 'BELOW':
          this.handleBuyOpportunity(alert);
          break;
        case 'ABOVE':
          this.handleSellOpportunity(alert);
          break;
      }

      res.json({ received: true });
    });
  }

  async handleBuyOpportunity(alert) {
    console.log(`Buy opportunity for item ${alert.itemId} at ${alert.price} gold`);

    // Check if we have enough gold
    if (await this.canAfford(alert.price)) {
      // Execute buy order
      await this.buyItem(alert.itemId, alert.price);
    }
  }

  async handleSellOpportunity(alert) {
    console.log(`Sell opportunity for item ${alert.itemId} at ${alert.price} gold`);

    // Check if we have the item in inventory
    if (await this.hasItem(alert.itemId)) {
      // Execute sell order
      await this.sellItem(alert.itemId, alert.price);
    }
  }
}

// Usage
const bot = new AuctionBot('your-api-key');
bot.start();
```

### Data Analysis Pipeline

```javascript
class MarketAnalyzer {
  async analyzeRealmPerformance(realm) {
    console.log(`Analyzing performance for ${realm}`);

    // Fetch recent auction data
    const auctions = await this.fetchAuctionData(realm, 1000);

    // Group by item category
    const categories = this.categorizeItems(auctions);

    // Analyze each category
    const analysis = {};

    for (const [category, items] of Object.entries(categories)) {
      analysis[category] = await this.analyzeCategory(items);
    }

    return analysis;
  }

  async analyzeCategory(auctions) {
    const prices = auctions.map(a => a.buyoutPrice);
    const quantities = auctions.map(a => a.quantity);

    return {
      totalListings: auctions.length,
      averagePrice: this.calculateAverage(prices),
      medianPrice: this.calculateMedian(prices),
      totalVolume: prices.reduce((sum, price, i) => sum + (price * quantities[i]), 0),
      priceVolatility: this.calculateVolatility(prices),
      topSellers: this.findTopSellers(auctions)
    };
  }

  categorizeItems(auctions) {
    const categories = {};

    auctions.forEach(auction => {
      const category = this.getItemCategory(auction.itemId);

      if (!categories[category]) {
        categories[category] = [];
      }

      categories[category].push(auction);
    });

    return categories;
  }

  getItemCategory(itemId) {
    // Simplified categorization logic
    if (itemId < 10000) return 'Consumables';
    if (itemId < 20000) return 'Weapons';
    if (itemId < 30000) return 'Armor';
    return 'Miscellaneous';
  }

  calculateAverage(values) {
    return values.reduce((sum, val) => sum + val, 0) / values.length;
  }

  calculateMedian(values) {
    const sorted = [...values].sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 === 0
      ? (sorted[mid - 1] + sorted[mid]) / 2
      : sorted[mid];
  }

  calculateVolatility(prices) {
    const mean = this.calculateAverage(prices);
    const variance = prices.reduce((sum, price) => sum + Math.pow(price - mean, 2), 0) / prices.length;
    return Math.sqrt(variance);
  }

  findTopSellers(auctions) {
    const sellers = {};

    auctions.forEach(auction => {
      sellers[auction.seller] = (sellers[auction.seller] || 0) + auction.quantity;
    });

    return Object.entries(sellers)
      .sort(([,a], [,b]) => b - a)
      .slice(0, 10)
      .map(([seller, quantity]) => ({ seller, quantity }));
  }
}

// Usage
const analyzer = new MarketAnalyzer();
const stormrageAnalysis = await analyzer.analyzeRealmPerformance('Stormrage');
console.log('Stormrage analysis:', JSON.stringify(stormrageAnalysis, null, 2));
```

### Custom Dashboard Widget

```jsx
import React, { useState, useEffect } from 'react';

function ProfitCalculatorWidget({ watchlist }) {
  const [profits, setProfits] = useState({});
  const [totalProfit, setTotalProfit] = useState(0);

  useEffect(() => {
    calculateProfits();
  }, [watchlist]);

  const calculateProfits = async () => {
    const profitData = {};

    for (const item of watchlist) {
      try {
        const marketData = await fetch(`/api/auctions/${item.id}?realm=Stormrage`, {
          headers: { 'X-API-Key': 'your-api-key' }
        }).then(res => res.json());

        const avgPrice = marketData.analysis.averagePrice;
        const potentialProfit = (item.sellPrice - avgPrice) * item.quantity;

        profitData[item.id] = {
          name: item.name,
          currentPrice: avgPrice,
          sellPrice: item.sellPrice,
          quantity: item.quantity,
          profit: potentialProfit,
          roi: (potentialProfit / (avgPrice * item.quantity)) * 100
        };
      } catch (error) {
        console.error(`Failed to calculate profit for ${item.name}:`, error);
      }
    }

    setProfits(profitData);

    const total = Object.values(profitData).reduce((sum, item) => sum + item.profit, 0);
    setTotalProfit(total);
  };

  const formatGold = (amount) => {
    return `${(amount / 10000).toFixed(1)}k gold`;
  };

  return (
    <div className="profit-calculator-widget">
      <h3>Potential Profits</h3>

      <div className="total-profit">
        <strong>Total: {formatGold(totalProfit)}</strong>
      </div>

      <div className="profit-breakdown">
        {Object.entries(profits).map(([itemId, data]) => (
          <div key={itemId} className="profit-item">
            <div className="item-name">{data.name}</div>
            <div className="profit-details">
              <span>Current: {formatGold(data.currentPrice)}</span>
              <span>Sell: {formatGold(data.sellPrice)}</span>
              <span>Profit: {formatGold(data.profit)}</span>
              <span>ROI: {data.roi.toFixed(1)}%</span>
            </div>
          </div>
        ))}
      </div>

      <button onClick={calculateProfits} className="refresh-btn">
        Refresh
      </button>
    </div>
  );
}

export default ProfitCalculatorWidget;
```

## Error Handling Examples

### Robust API Client

```javascript
class RobustAuctionClient {
  constructor(apiKey, baseUrl = 'https://api.wowauctiontracker.com') {
    this.apiKey = apiKey;
    this.baseUrl = baseUrl;
    this.retryCount = 3;
    this.retryDelay = 1000;
  }

  async makeRequest(endpoint, options = {}) {
    const url = `${this.baseUrl}${endpoint}`;

    for (let attempt = 1; attempt <= this.retryCount; attempt++) {
      try {
        const response = await fetch(url, {
          ...options,
          headers: {
            'X-API-Key': this.apiKey,
            'Content-Type': 'application/json',
            ...options.headers
          }
        });

        if (response.ok) {
          return await response.json();
        }

        // Handle rate limiting
        if (response.status === 429) {
          const retryAfter = response.headers.get('Retry-After');
          const delay = retryAfter ? parseInt(retryAfter) * 1000 : this.retryDelay * attempt;
          await this.sleep(delay);
          continue;
        }

        // Handle server errors
        if (response.status >= 500) {
          await this.sleep(this.retryDelay * attempt);
          continue;
        }

        // Handle client errors
        const error = await response.json().catch(() => ({ message: 'Unknown error' }));
        throw new Error(`API Error ${response.status}: ${error.message}`);

      } catch (error) {
        if (attempt === this.retryCount) {
          throw new Error(`Failed after ${this.retryCount} attempts: ${error.message}`);
        }

        console.warn(`Attempt ${attempt} failed: ${error.message}`);
        await this.sleep(this.retryDelay * attempt);
      }
    }
  }

  async getAuctions(filters = {}) {
    const params = new URLSearchParams(filters);
    return this.makeRequest(`/api/auctions?${params}`);
  }

  async createAlert(alertConfig) {
    return this.makeRequest('/api/alerts', {
      method: 'POST',
      body: JSON.stringify(alertConfig)
    });
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const client = new RobustAuctionClient('your-api-key');

try {
  const auctions = await client.getAuctions({
    realm: 'Stormrage',
    minPrice: 100000,
    limit: 100
  });

  console.log(`Successfully fetched ${auctions.auctions.length} auctions`);
} catch (error) {
  console.error('Failed to fetch auctions:', error.message);
}
```

### Batch Processing with Error Recovery

```javascript
async function processAuctionBatch(auctions, batchSize = 100) {
  const results = [];
  const errors = [];

  // Process in batches
  for (let i = 0; i < auctions.length; i += batchSize) {
    const batch = auctions.slice(i, i + batchSize);

    try {
      const batchResults = await processBatch(batch);
      results.push(...batchResults);
    } catch (error) {
      console.error(`Failed to process batch ${Math.floor(i / batchSize) + 1}:`, error);

      // Try to process items individually
      for (const auction of batch) {
        try {
          const result = await processAuction(auction);
          results.push(result);
        } catch (itemError) {
          console.error(`Failed to process auction ${auction.id}:`, itemError);
          errors.push({ auction: auction.id, error: itemError.message });
        }
      }
    }
  }

  return { results, errors };
}

async function processAuction(auction) {
  // Validate auction data
  if (!auction.itemId || !auction.buyoutPrice) {
    throw new Error('Invalid auction data');
  }

  // Calculate market value
  const marketValue = await calculateMarketValue(auction.itemId);

  // Determine if it's a good deal
  const discount = ((marketValue - auction.buyoutPrice) / marketValue) * 100;

  return {
    auctionId: auction.id,
    marketValue,
    listPrice: auction.buyoutPrice,
    discount,
    isGoodDeal: discount > 15 // 15%+ discount
  };
}
```

## Performance Optimization Examples

### Caching Strategy

```javascript
class AuctionCache {
  constructor(ttl = 300000) { // 5 minutes default
    this.cache = new Map();
    this.ttl = ttl;
  }

  async get(key, fetcher) {
    const cached = this.cache.get(key);

    if (cached && Date.now() < cached.expiry) {
      console.log(`Cache hit for ${key}`);
      return cached.data;
    }

    console.log(`Cache miss for ${key}, fetching...`);
    const data = await fetcher();

    this.cache.set(key, {
      data,
      expiry: Date.now() + this.ttl
    });

    return data;
  }

  invalidate(key) {
    this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  // Clean up expired entries
  cleanup() {
    const now = Date.now();
    for (const [key, value] of this.cache.entries()) {
      if (now >= value.expiry) {
        this.cache.delete(key);
      }
    }
  }
}

// Usage
const cache = new AuctionCache(600000); // 10 minutes

async function getAuctionsWithCache(realm, filters) {
  const cacheKey = `auctions:${realm}:${JSON.stringify(filters)}`;

  return cache.get(cacheKey, async () => {
    const response = await fetch(`/api/auctions?realm=${realm}&${new URLSearchParams(filters)}`, {
      headers: { 'X-API-Key': 'your-api-key' }
    });

    return response.json();
  });
}
```

### Rate Limiting Implementation

```javascript
class RateLimiter {
  constructor(requestsPerSecond = 10) {
    this.requestsPerSecond = requestsPerSecond;
    this.interval = 1000 / requestsPerSecond; // ms between requests
    this.lastRequest = 0;
    this.queue = [];
  }

  async throttle() {
    const now = Date.now();
    const timeSinceLastRequest = now - this.lastRequest;

    if (timeSinceLastRequest < this.interval) {
      const waitTime = this.interval - timeSinceLastRequest;
      await this.sleep(waitTime);
    }

    this.lastRequest = Date.now();
  }

  async makeRequest(url, options) {
    await this.throttle();

    return fetch(url, {
      ...options,
      headers: {
        'X-API-Key': 'your-api-key',
        ...options.headers
      }
    });
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const limiter = new RateLimiter(5); // 5 requests per second

async function fetchMultipleAuctions(itemIds) {
  const promises = itemIds.map(async (itemId) => {
    try {
      const response = await limiter.makeRequest(`/api/auctions/${itemId}`);
      return await response.json();
    } catch (error) {
      console.error(`Failed to fetch auctions for item ${itemId}:`, error);
      return null;
    }
  });

  const results = await Promise.allSettled(promises);
  return results
    .filter(result => result.status === 'fulfilled' && result.value)
    .map(result => result.value);
}
```

These examples demonstrate various ways to use the WoW Auction Tracker API and components, from basic data fetching to advanced automated trading systems. The code samples show best practices for error handling, performance optimization, and robust API usage.```

I've completed a comprehensive set of usage examples and code samples that demonstrate:

1. **Basic API Usage** - Simple fetch operations and WebSocket connections
2. **Price Analysis Examples** - Market analysis, trend detection, and opportunity finding
3. **Price Alert Setup** - Creating and managing price monitoring alerts
4. **React Component Examples** - Using UI components in real applications
5. **Advanced Usage Examples** - Automated trading bot and data analysis pipeline
6. **Error Handling Examples** - Robust error handling and retry logic
7. **Performance Optimization Examples** - Caching strategies and rate limiting

The examples progress from basic to advanced usage, showing practical implementations that developers can adapt for their own applications. Each example includes proper error handling, type safety, and follows best practices for production code.