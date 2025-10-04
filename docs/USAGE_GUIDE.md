# WoW Auction Tracker - Usage Guide

## Table of Contents

1. [Getting Started](#getting-started)
2. [Quick Start Examples](#quick-start-examples)
3. [Common Use Cases](#common-use-cases)
4. [Integration Examples](#integration-examples)
5. [Best Practices](#best-practices)
6. [Troubleshooting](#troubleshooting)

## Getting Started

### Prerequisites

- Node.js 16+ or Python 3.8+
- Valid Blizzard API credentials
- Database (PostgreSQL recommended)
- Redis for caching (optional but recommended)

### Installation

#### JavaScript/Node.js

```bash
npm install wow-auction-tracker-sdk
```

#### Python

```bash
pip install wow-auction-tracker
```

### Initial Setup

1. **Get API Credentials**
   ```javascript
   // Register at https://wow-auction-tracker.com/api
   const apiKey = 'your-api-key-here';
   ```

2. **Initialize Client**
   ```javascript
   const WoWAuctionTracker = require('wow-auction-tracker-sdk');
   
   const client = new WoWAuctionTracker({
     apiKey: apiKey,
     baseURL: 'https://api.wow-auction-tracker.com/v1'
   });
   ```

3. **Test Connection**
   ```javascript
   async function testConnection() {
     try {
       const realms = await client.realms.getAll();
       console.log('Connected successfully!');
       console.log(`Found ${realms.data.realms.length} realms`);
     } catch (error) {
       console.error('Connection failed:', error.message);
     }
   }
   
   testConnection();
   ```

## Quick Start Examples

### 1. Get Current Auctions

```javascript
// Get all auctions for a realm
async function getCurrentAuctions() {
  try {
    const auctions = await client.auctions.get('stormrage', {
      limit: 100,
      sort_by: 'price',
      sort_order: 'asc'
    });
    
    console.log(`Found ${auctions.data.auctions.length} auctions`);
    
    auctions.data.auctions.forEach(auction => {
      console.log(`${auction.item_name}: ${formatPrice(auction.unit_price)}`);
    });
  } catch (error) {
    console.error('Error fetching auctions:', error);
  }
}

function formatPrice(copper) {
  const gold = Math.floor(copper / 10000);
  const silver = Math.floor((copper % 10000) / 100);
  const remainingCopper = copper % 100;
  return `${gold}g ${silver}s ${remainingCopper}c`;
}

getCurrentAuctions();
```

### 2. Search for Specific Items

```javascript
// Find all Thunderfury auctions
async function findThunderfury() {
  try {
    const auctions = await client.auctions.get('stormrage', {
      item_id: 19019, // Thunderfury item ID
      limit: 10
    });
    
    if (auctions.data.auctions.length === 0) {
      console.log('No Thunderfury auctions found');
      return;
    }
    
    console.log('Thunderfury Auctions:');
    auctions.data.auctions.forEach(auction => {
      console.log(`Price: ${formatPrice(auction.buyout)} | Time Left: ${auction.time_left}`);
    });
  } catch (error) {
    console.error('Error searching for item:', error);
  }
}

findThunderfury();
```

### 3. Get Price History

```javascript
// Get 30-day price history for an item
async function getPriceHistory() {
  try {
    const history = await client.prices.getHistory('stormrage', 19019, {
      period: '30d',
      granularity: '1d'
    });
    
    console.log('Price History for Thunderfury:');
    console.log(`Current Price: ${formatPrice(history.data.summary.current_price)}`);
    console.log(`24h Change: ${history.data.summary.price_change_24h > 0 ? '+' : ''}${formatPrice(history.data.summary.price_change_24h)}`);
    
    // Show last 7 days
    const recentHistory = history.data.price_history.slice(-7);
    recentHistory.forEach(day => {
      const date = new Date(day.timestamp).toLocaleDateString();
      console.log(`${date}: Avg ${formatPrice(day.avg_price)} | Vol: ${day.volume}`);
    });
  } catch (error) {
    console.error('Error fetching price history:', error);
  }
}

getPriceHistory();
```

### 4. Set Up Price Alerts

```javascript
// Create a watchlist and set up alerts
async function setupPriceAlerts(userId) {
  try {
    // Add item to watchlist
    await client.users.addToWatchlist(userId, {
      item_id: 19019,
      target_price: 900000, // 90 gold
      notify_on_drop: true
    });
    
    console.log('Added Thunderfury to watchlist with 90g target price');
    
    // Set up notification subscription
    await client.users.createSubscription(userId, {
      type: 'price_alert',
      item_id: 19019,
      condition: 'below',
      threshold: 900000,
      channels: ['email', 'webhook']
    });
    
    console.log('Price alert subscription created');
  } catch (error) {
    console.error('Error setting up alerts:', error);
  }
}

// setupPriceAlerts(123);
```

## Common Use Cases

### Use Case 1: Market Analysis Dashboard

Build a dashboard to analyze market trends across multiple realms.

```javascript
class MarketAnalyzer {
  constructor(client) {
    this.client = client;
  }
  
  async analyzeMarket(realms, itemIds) {
    const analysis = {};
    
    for (const realm of realms) {
      analysis[realm] = {};
      
      for (const itemId of itemIds) {
        try {
          // Get current auctions
          const auctions = await this.client.auctions.get(realm, {
            item_id: itemId,
            limit: 100
          });
          
          // Get price history
          const history = await this.client.prices.getHistory(realm, itemId, {
            period: '7d'
          });
          
          // Calculate metrics
          const prices = auctions.data.auctions.map(a => a.unit_price);
          const avgPrice = prices.reduce((a, b) => a + b, 0) / prices.length;
          const minPrice = Math.min(...prices);
          const maxPrice = Math.max(...prices);
          
          analysis[realm][itemId] = {
            current_avg: avgPrice,
            current_min: minPrice,
            current_max: maxPrice,
            volume: auctions.data.auctions.length,
            trend: history.data.summary.price_change_7d,
            volatility: history.data.summary.volatility
          };
          
        } catch (error) {
          console.error(`Error analyzing ${realm}:${itemId}:`, error);
          analysis[realm][itemId] = null;
        }
      }
    }
    
    return analysis;
  }
  
  findArbitrageOpportunities(analysis) {
    const opportunities = [];
    const itemIds = Object.keys(analysis[Object.keys(analysis)[0]]);
    
    itemIds.forEach(itemId => {
      const realmPrices = {};
      
      Object.keys(analysis).forEach(realm => {
        if (analysis[realm][itemId]) {
          realmPrices[realm] = analysis[realm][itemId].current_min;
        }
      });
      
      const realms = Object.keys(realmPrices);
      if (realms.length < 2) return;
      
      const minRealm = realms.reduce((a, b) => 
        realmPrices[a] < realmPrices[b] ? a : b
      );
      const maxRealm = realms.reduce((a, b) => 
        realmPrices[a] > realmPrices[b] ? a : b
      );
      
      const profit = realmPrices[maxRealm] - realmPrices[minRealm];
      const profitPercent = (profit / realmPrices[minRealm]) * 100;
      
      if (profitPercent > 20) { // 20% profit threshold
        opportunities.push({
          item_id: itemId,
          buy_realm: minRealm,
          sell_realm: maxRealm,
          buy_price: realmPrices[minRealm],
          sell_price: realmPrices[maxRealm],
          profit: profit,
          profit_percent: profitPercent
        });
      }
    });
    
    return opportunities.sort((a, b) => b.profit_percent - a.profit_percent);
  }
}

// Usage
async function runMarketAnalysis() {
  const analyzer = new MarketAnalyzer(client);
  
  const realms = ['stormrage', 'tichondrius', 'area-52'];
  const itemIds = [19019, 17182, 18803]; // Legendary items
  
  console.log('Analyzing market...');
  const analysis = await analyzer.analyzeMarket(realms, itemIds);
  
  console.log('Finding arbitrage opportunities...');
  const opportunities = analyzer.findArbitrageOpportunities(analysis);
  
  console.log('Top Arbitrage Opportunities:');
  opportunities.slice(0, 5).forEach(opp => {
    console.log(`Item ${opp.item_id}: Buy on ${opp.buy_realm} for ${formatPrice(opp.buy_price)}, sell on ${opp.sell_realm} for ${formatPrice(opp.sell_price)} (${opp.profit_percent.toFixed(1)}% profit)`);
  });
}

// runMarketAnalysis();
```

### Use Case 2: Automated Bidding Bot

Create a bot that automatically bids on underpriced items.

```javascript
class BiddingBot {
  constructor(client, userId) {
    this.client = client;
    this.userId = userId;
    this.isRunning = false;
    this.bidHistory = [];
  }
  
  async start(config) {
    this.isRunning = true;
    this.config = config;
    
    console.log('Starting bidding bot...');
    
    while (this.isRunning) {
      try {
        await this.scanForOpportunities();
        await this.sleep(config.scanInterval || 30000); // 30 seconds
      } catch (error) {
        console.error('Bot error:', error);
        await this.sleep(5000); // Wait 5 seconds on error
      }
    }
  }
  
  async scanForOpportunities() {
    for (const target of this.config.targets) {
      try {
        const auctions = await this.client.auctions.get(target.realm, {
          item_id: target.item_id,
          max_price: target.max_price,
          limit: 10
        });
        
        for (const auction of auctions.data.auctions) {
          if (this.shouldBid(auction, target)) {
            await this.placeBid(auction, target);
          }
        }
      } catch (error) {
        console.error(`Error scanning ${target.realm}:${target.item_id}:`, error);
      }
    }
  }
  
  shouldBid(auction, target) {
    // Don't bid if we already bid on this auction
    if (this.bidHistory.some(bid => bid.auction_id === auction.id)) {
      return false;
    }
    
    // Don't bid if price is too high
    if (auction.bid > target.max_price) {
      return false;
    }
    
    // Don't bid if time left is too short
    if (auction.time_left === 'SHORT' && !target.allow_short_auctions) {
      return false;
    }
    
    return true;
  }
  
  async placeBid(auction, target) {
    const bidAmount = Math.min(
      auction.bid + target.bid_increment,
      target.max_price
    );
    
    console.log(`Placing bid of ${formatPrice(bidAmount)} on ${auction.item_name}`);
    
    // Note: This would require additional API endpoints for bidding
    // This is a conceptual example
    try {
      // await this.client.auctions.placeBid(auction.id, bidAmount);
      
      this.bidHistory.push({
        auction_id: auction.id,
        item_id: auction.item_id,
        bid_amount: bidAmount,
        timestamp: new Date()
      });
      
      console.log('Bid placed successfully');
    } catch (error) {
      console.error('Failed to place bid:', error);
    }
  }
  
  stop() {
    this.isRunning = false;
    console.log('Stopping bidding bot...');
  }
  
  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const biddingConfig = {
  scanInterval: 60000, // 1 minute
  targets: [
    {
      realm: 'stormrage',
      item_id: 19019,
      max_price: 1000000, // 100 gold max
      bid_increment: 10000, // 1 gold increment
      allow_short_auctions: false
    }
  ]
};

// const bot = new BiddingBot(client, userId);
// bot.start(biddingConfig);
```

### Use Case 3: Portfolio Tracker

Track the value of your in-game items and investments.

```javascript
class PortfolioTracker {
  constructor(client, userId) {
    this.client = client;
    this.userId = userId;
  }
  
  async addItem(realm, itemId, quantity, purchasePrice, purchaseDate) {
    const portfolio = await this.getPortfolio();
    
    const item = {
      id: Date.now(), // Simple ID generation
      realm,
      item_id: itemId,
      quantity,
      purchase_price: purchasePrice,
      purchase_date: purchaseDate || new Date(),
      current_value: null,
      profit_loss: null
    };
    
    portfolio.items.push(item);
    await this.savePortfolio(portfolio);
    
    return item;
  }
  
  async updatePortfolioValues() {
    const portfolio = await this.getPortfolio();
    let totalValue = 0;
    let totalCost = 0;
    
    for (const item of portfolio.items) {
      try {
        // Get current market price
        const auctions = await this.client.auctions.get(item.realm, {
          item_id: item.item_id,
          limit: 20
        });
        
        if (auctions.data.auctions.length > 0) {
          const prices = auctions.data.auctions.map(a => a.unit_price);
          const avgPrice = prices.reduce((a, b) => a + b, 0) / prices.length;
          
          item.current_value = avgPrice * item.quantity;
          item.profit_loss = item.current_value - (item.purchase_price * item.quantity);
          
          totalValue += item.current_value;
          totalCost += (item.purchase_price * item.quantity);
        }
      } catch (error) {
        console.error(`Error updating ${item.item_id}:`, error);
      }
    }
    
    portfolio.total_value = totalValue;
    portfolio.total_cost = totalCost;
    portfolio.total_profit_loss = totalValue - totalCost;
    portfolio.last_updated = new Date();
    
    await this.savePortfolio(portfolio);
    return portfolio;
  }
  
  async getPerformanceReport() {
    const portfolio = await this.updatePortfolioValues();
    
    const report = {
      summary: {
        total_items: portfolio.items.length,
        total_value: portfolio.total_value,
        total_cost: portfolio.total_cost,
        total_profit_loss: portfolio.total_profit_loss,
        profit_loss_percent: ((portfolio.total_profit_loss / portfolio.total_cost) * 100).toFixed(2)
      },
      top_performers: portfolio.items
        .filter(item => item.profit_loss !== null)
        .sort((a, b) => b.profit_loss - a.profit_loss)
        .slice(0, 5),
      worst_performers: portfolio.items
        .filter(item => item.profit_loss !== null)
        .sort((a, b) => a.profit_loss - b.profit_loss)
        .slice(0, 5)
    };
    
    return report;
  }
  
  async getPortfolio() {
    // In a real implementation, this would load from database
    return {
      user_id: this.userId,
      items: [],
      total_value: 0,
      total_cost: 0,
      total_profit_loss: 0,
      last_updated: null
    };
  }
  
  async savePortfolio(portfolio) {
    // In a real implementation, this would save to database
    console.log('Portfolio saved:', portfolio);
  }
}

// Usage
async function trackPortfolio() {
  const tracker = new PortfolioTracker(client, userId);
  
  // Add some items to portfolio
  await tracker.addItem('stormrage', 19019, 1, 950000, new Date('2023-10-01'));
  await tracker.addItem('stormrage', 17182, 2, 500000, new Date('2023-09-15'));
  
  // Get performance report
  const report = await tracker.getPerformanceReport();
  
  console.log('Portfolio Performance Report:');
  console.log(`Total Value: ${formatPrice(report.summary.total_value)}`);
  console.log(`Total Cost: ${formatPrice(report.summary.total_cost)}`);
  console.log(`P&L: ${formatPrice(report.summary.total_profit_loss)} (${report.summary.profit_loss_percent}%)`);
  
  console.log('\nTop Performers:');
  report.top_performers.forEach(item => {
    console.log(`Item ${item.item_id}: ${formatPrice(item.profit_loss)} profit`);
  });
}

// trackPortfolio();
```

## Integration Examples

### React Integration

```jsx
// hooks/useAuctions.js
import { useState, useEffect } from 'react';
import { WoWAuctionTracker } from 'wow-auction-tracker-sdk';

const client = new WoWAuctionTracker({
  apiKey: process.env.REACT_APP_API_KEY
});

export function useAuctions(realm, filters = {}) {
  const [auctions, setAuctions] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    const fetchAuctions = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await client.auctions.get(realm, filters);
        if (!isCancelled) {
          setAuctions(response.data.auctions);
        }
      } catch (err) {
        if (!isCancelled) {
          setError(err.message);
        }
      } finally {
        if (!isCancelled) {
          setLoading(false);
        }
      }
    };

    if (realm) {
      fetchAuctions();
    }

    return () => {
      isCancelled = true;
    };
  }, [realm, JSON.stringify(filters)]);

  return { auctions, loading, error };
}

// components/AuctionDashboard.jsx
import React, { useState } from 'react';
import { useAuctions } from '../hooks/useAuctions';

function AuctionDashboard() {
  const [realm, setRealm] = useState('stormrage');
  const [itemId, setItemId] = useState('');
  
  const { auctions, loading, error } = useAuctions(realm, {
    item_id: itemId || undefined,
    limit: 50
  });

  if (loading) return <div>Loading auctions...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="auction-dashboard">
      <div className="filters">
        <select value={realm} onChange={(e) => setRealm(e.target.value)}>
          <option value="stormrage">Stormrage</option>
          <option value="tichondrius">Tichondrius</option>
          <option value="area-52">Area-52</option>
        </select>
        
        <input
          type="number"
          placeholder="Item ID (optional)"
          value={itemId}
          onChange={(e) => setItemId(e.target.value)}
        />
      </div>

      <div className="auction-list">
        {auctions.map(auction => (
          <div key={auction.id} className="auction-item">
            <h3>{auction.item_name}</h3>
            <p>Price: {formatPrice(auction.unit_price)}</p>
            <p>Quantity: {auction.quantity}</p>
            <p>Time Left: {auction.time_left}</p>
          </div>
        ))}
      </div>
    </div>
  );
}

function formatPrice(copper) {
  const gold = Math.floor(copper / 10000);
  const silver = Math.floor((copper % 10000) / 100);
  const remainingCopper = copper % 100;
  return `${gold}g ${silver}s ${remainingCopper}c`;
}

export default AuctionDashboard;
```

### Express.js Backend Integration

```javascript
// routes/auctions.js
const express = require('express');
const { WoWAuctionTracker } = require('wow-auction-tracker-sdk');
const router = express.Router();

const client = new WoWAuctionTracker({
  apiKey: process.env.API_KEY
});

// Get auctions with caching
router.get('/:realm', async (req, res) => {
  try {
    const { realm } = req.params;
    const { item_id, limit = 50, sort_by = 'price' } = req.query;
    
    // Check cache first
    const cacheKey = `auctions:${realm}:${item_id || 'all'}:${limit}:${sort_by}`;
    let auctions = await redis.get(cacheKey);
    
    if (!auctions) {
      const response = await client.auctions.get(realm, {
        item_id: item_id ? parseInt(item_id) : undefined,
        limit: parseInt(limit),
        sort_by
      });
      
      auctions = response.data;
      
      // Cache for 5 minutes
      await redis.setex(cacheKey, 300, JSON.stringify(auctions));
    } else {
      auctions = JSON.parse(auctions);
    }
    
    res.json({
      success: true,
      data: auctions,
      cached: !!auctions
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

// Get price history
router.get('/:realm/:itemId/history', async (req, res) => {
  try {
    const { realm, itemId } = req.params;
    const { period = '7d' } = req.query;
    
    const response = await client.prices.getHistory(realm, parseInt(itemId), {
      period
    });
    
    res.json({
      success: true,
      data: response.data
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

module.exports = router;

// app.js
const express = require('express');
const auctionRoutes = require('./routes/auctions');

const app = express();

app.use('/api/auctions', auctionRoutes);

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Python Flask Integration

```python
# app.py
from flask import Flask, request, jsonify
from wow_auction_tracker import WoWAuctionTracker
import redis
import json
import os

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

client = WoWAuctionTracker(
    api_key=os.getenv('API_KEY'),
    base_url='https://api.wow-auction-tracker.com/v1'
)

@app.route('/api/auctions/<realm>')
def get_auctions(realm):
    try:
        item_id = request.args.get('item_id', type=int)
        limit = request.args.get('limit', default=50, type=int)
        sort_by = request.args.get('sort_by', default='price')
        
        # Check cache
        cache_key = f"auctions:{realm}:{item_id or 'all'}:{limit}:{sort_by}"
        cached_data = redis_client.get(cache_key)
        
        if cached_data:
            auctions = json.loads(cached_data)
            return jsonify({
                'success': True,
                'data': auctions,
                'cached': True
            })
        
        # Fetch from API
        auctions = client.auctions.get(realm, {
            'item_id': item_id,
            'limit': limit,
            'sort_by': sort_by
        })
        
        # Cache for 5 minutes
        redis_client.setex(cache_key, 300, json.dumps(auctions['data']))
        
        return jsonify({
            'success': True,
            'data': auctions['data'],
            'cached': False
        })
        
    except Exception as e:
        return jsonify({
            'success': False,
            'error': str(e)
        }), 500

@app.route('/api/prices/<realm>/<int:item_id>/history')
def get_price_history(realm, item_id):
    try:
        period = request.args.get('period', default='7d')
        
        history = client.prices.get_history(realm, item_id, {
            'period': period
        })
        
        return jsonify({
            'success': True,
            'data': history['data']
        })
        
    except Exception as e:
        return jsonify({
            'success': False,
            'error': str(e)
        }), 500

if __name__ == '__main__':
    app.run(debug=True)
```

## Best Practices

### 1. Error Handling

```javascript
// Comprehensive error handling
async function robustApiCall(apiFunction, ...args) {
  const maxRetries = 3;
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await apiFunction(...args);
    } catch (error) {
      lastError = error;
      
      // Don't retry on client errors (4xx)
      if (error.status >= 400 && error.status < 500) {
        throw error;
      }
      
      // Exponential backoff
      if (attempt < maxRetries) {
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw lastError;
}

// Usage
try {
  const auctions = await robustApiCall(
    client.auctions.get.bind(client.auctions),
    'stormrage',
    { limit: 100 }
  );
} catch (error) {
  console.error('All retry attempts failed:', error);
}
```

### 2. Rate Limiting

```javascript
class RateLimiter {
  constructor(requestsPerSecond = 10) {
    this.requestsPerSecond = requestsPerSecond;
    this.requests = [];
  }
  
  async throttle() {
    const now = Date.now();
    
    // Remove requests older than 1 second
    this.requests = this.requests.filter(time => now - time < 1000);
    
    if (this.requests.length >= this.requestsPerSecond) {
      const oldestRequest = Math.min(...this.requests);
      const waitTime = 1000 - (now - oldestRequest);
      
      if (waitTime > 0) {
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
    
    this.requests.push(now);
  }
}

// Usage
const rateLimiter = new RateLimiter(5); // 5 requests per second

async function makeApiCall() {
  await rateLimiter.throttle();
  return client.auctions.get('stormrage');
}
```

### 3. Caching Strategy

```javascript
class SmartCache {
  constructor(redis, defaultTTL = 300) {
    this.redis = redis;
    this.defaultTTL = defaultTTL;
  }
  
  generateKey(endpoint, params) {
    const sortedParams = Object.keys(params)
      .sort()
      .map(key => `${key}:${params[key]}`)
      .join('|');
    return `api:${endpoint}:${sortedParams}`;
  }
  
  async get(endpoint, params) {
    const key = this.generateKey(endpoint, params);
    const cached = await this.redis.get(key);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    return null;
  }
  
  async set(endpoint, params, data, ttl = this.defaultTTL) {
    const key = this.generateKey(endpoint, params);
    await this.redis.setex(key, ttl, JSON.stringify(data));
  }
  
  async invalidatePattern(pattern) {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}

// Usage with different TTL for different data types
const cache = new SmartCache(redis);

// Cache auctions for 5 minutes
await cache.set('auctions', { realm: 'stormrage' }, auctionData, 300);

// Cache item data for 1 hour (items don't change often)
await cache.set('items', { id: 19019 }, itemData, 3600);

// Cache price history for 10 minutes
await cache.set('price_history', { realm: 'stormrage', item_id: 19019 }, historyData, 600);
```

### 4. Data Validation

```javascript
const Joi = require('joi');

const auctionQuerySchema = Joi.object({
  realm: Joi.string().alphanum().min(3).max(50).required(),
  item_id: Joi.number().integer().positive().optional(),
  min_price: Joi.number().integer().min(0).optional(),
  max_price: Joi.number().integer().min(0).optional(),
  limit: Joi.number().integer().min(1).max(1000).default(50),
  offset: Joi.number().integer().min(0).default(0)
});

function validateAuctionQuery(query) {
  const { error, value } = auctionQuerySchema.validate(query);
  
  if (error) {
    throw new Error(`Invalid query parameters: ${error.details[0].message}`);
  }
  
  return value;
}

// Usage
try {
  const validatedQuery = validateAuctionQuery(req.query);
  const auctions = await client.auctions.get(validatedQuery.realm, validatedQuery);
} catch (error) {
  res.status(400).json({ error: error.message });
}
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Authentication Errors

**Problem:** Getting 401 Unauthorized errors

**Solutions:**
```javascript
// Check API key format
if (!apiKey || apiKey.length < 32) {
  throw new Error('Invalid API key format');
}

// Verify API key is active
async function verifyApiKey() {
  try {
    await client.realms.getAll();
    console.log('API key is valid');
  } catch (error) {
    if (error.status === 401) {
      console.error('API key is invalid or expired');
    }
    throw error;
  }
}
```

#### 2. Rate Limiting Issues

**Problem:** Getting 429 Too Many Requests errors

**Solutions:**
```javascript
// Implement exponential backoff
async function handleRateLimit(apiCall) {
  let delay = 1000; // Start with 1 second
  
  while (true) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.status === 429) {
        console.log(`Rate limited, waiting ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= 2; // Double the delay
        continue;
      }
      throw error;
    }
  }
}
```

#### 3. Data Inconsistency

**Problem:** Auction data seems outdated or inconsistent

**Solutions:**
```javascript
// Check data freshness
function isDataFresh(timestamp, maxAgeMinutes = 10) {
  const now = new Date();
  const dataTime = new Date(timestamp);
  const ageMinutes = (now - dataTime) / (1000 * 60);
  
  return ageMinutes <= maxAgeMinutes;
}

// Validate auction data
function validateAuctionData(auction) {
  const required = ['id', 'item_id', 'unit_price', 'quantity'];
  
  for (const field of required) {
    if (auction[field] === undefined || auction[field] === null) {
      throw new Error(`Missing required field: ${field}`);
    }
  }
  
  if (auction.unit_price <= 0) {
    throw new Error('Invalid price: must be positive');
  }
  
  if (auction.quantity <= 0) {
    throw new Error('Invalid quantity: must be positive');
  }
}
```

#### 4. Performance Issues

**Problem:** Slow API responses or timeouts

**Solutions:**
```javascript
// Implement request timeout
const client = new WoWAuctionTracker({
  apiKey: apiKey,
  timeout: 10000, // 10 seconds
  retries: 3
});

// Use pagination for large datasets
async function getAllAuctions(realm, batchSize = 100) {
  const allAuctions = [];
  let offset = 0;
  let hasMore = true;
  
  while (hasMore) {
    const batch = await client.auctions.get(realm, {
      limit: batchSize,
      offset: offset
    });
    
    allAuctions.push(...batch.data.auctions);
    
    hasMore = batch.data.auctions.length === batchSize;
    offset += batchSize;
    
    // Add small delay between requests
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return allAuctions;
}
```

#### 5. Memory Issues with Large Datasets

**Problem:** Application running out of memory when processing large auction datasets

**Solutions:**
```javascript
// Stream processing for large datasets
async function* auctionStream(realm, batchSize = 1000) {
  let offset = 0;
  let hasMore = true;
  
  while (hasMore) {
    const batch = await client.auctions.get(realm, {
      limit: batchSize,
      offset: offset
    });
    
    for (const auction of batch.data.auctions) {
      yield auction;
    }
    
    hasMore = batch.data.auctions.length === batchSize;
    offset += batchSize;
  }
}

// Usage
async function processAllAuctions(realm) {
  let processedCount = 0;
  
  for await (const auction of auctionStream(realm)) {
    // Process one auction at a time
    await processAuction(auction);
    processedCount++;
    
    if (processedCount % 1000 === 0) {
      console.log(`Processed ${processedCount} auctions`);
    }
  }
}
```

This comprehensive usage guide provides practical examples and solutions for common scenarios when working with the WoW Auction Tracker API. The examples progress from simple use cases to more complex integrations and include best practices for production environments.