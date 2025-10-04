# Usage Guide and Examples

## Table of Contents

1. [Getting Started](#getting-started)
2. [Quick Start Examples](#quick-start-examples)
3. [Common Use Cases](#common-use-cases)
4. [Advanced Examples](#advanced-examples)
5. [Best Practices](#best-practices)
6. [Troubleshooting](#troubleshooting)
7. [Code Snippets Library](#code-snippets-library)

---

## Getting Started

### Prerequisites

- An active World of Warcraft account
- API key from WoW Auction Tracker (sign up at https://wow-auction-tracker.com)
- Basic knowledge of REST APIs or your chosen programming language

### Installation

#### JavaScript/Node.js

```bash
npm install wow-auction-tracker
```

#### Python

```bash
pip install wow-auction-tracker
```

#### CLI Tool

```bash
# Install globally
npm install -g wow-auction-tracker-cli

# Or use with npx
npx wow-auction-tracker-cli
```

### Authentication Setup

```javascript
// JavaScript
const WoWAuctionTracker = require('wow-auction-tracker');
const client = new WoWAuctionTracker('YOUR_API_KEY');
```

```python
# Python
from wow_auction_tracker import Client
client = Client(api_key='YOUR_API_KEY')
```

```bash
# CLI - Set API key as environment variable
export WOW_AUCTION_API_KEY="YOUR_API_KEY"

# Or use config file
wow-auction config set api_key YOUR_API_KEY
```

---

## Quick Start Examples

### Example 1: Get Current Auctions

**JavaScript:**

```javascript
const WoWAuctionTracker = require('wow-auction-tracker');
const client = new WoWAuctionTracker('YOUR_API_KEY');

async function getCurrentAuctions() {
  try {
    const auctions = await client.auctions.list({
      realm: 'stormrage',
      limit: 10
    });
    
    console.log(`Found ${auctions.data.auctions.length} auctions`);
    auctions.data.auctions.forEach(auction => {
      console.log(`${auction.item.name}: ${auction.buyout} copper`);
    });
  } catch (error) {
    console.error('Error:', error.message);
  }
}

getCurrentAuctions();
```

**Python:**

```python
from wow_auction_tracker import Client

client = Client(api_key='YOUR_API_KEY')

def get_current_auctions():
    try:
        auctions = client.auctions.list(realm='stormrage', limit=10)
        
        print(f"Found {len(auctions['data']['auctions'])} auctions")
        for auction in auctions['data']['auctions']:
            print(f"{auction['item']['name']}: {auction['buyout']} copper")
    except Exception as e:
        print(f"Error: {e}")

get_current_auctions()
```

**CLI:**

```bash
# List auctions
wow-auction auctions list --realm stormrage --limit 10

# Filter by item
wow-auction auctions list --realm stormrage --item-id 19019
```

**Output:**

```
Found 10 auctions
Thunderfury, Blessed Blade of the Windseeker: 50000000 copper
Sulfuras, Hand of Ragnaros: 35000000 copper
Atiesh, Greatstaff of the Guardian: 40000000 copper
...
```

---

### Example 2: Search for Specific Items

**JavaScript:**

```javascript
async function searchItems(query) {
  const results = await client.items.search({
    q: query,
    quality: 'legendary'
  });
  
  console.log(`Found ${results.data.total} items matching "${query}"`);
  results.data.items.forEach(item => {
    console.log(`[${item.id}] ${item.name} (iLvl ${item.item_level})`);
  });
}

searchItems('thunderfury');
```

**Python:**

```python
def search_items(query):
    results = client.items.search(q=query, quality='legendary')
    
    print(f"Found {results['data']['total']} items matching '{query}'")
    for item in results['data']['items']:
        print(f"[{item['id']}] {item['name']} (iLvl {item['item_level']})")

search_items('thunderfury')
```

**Output:**

```
Found 1 items matching "thunderfury"
[19019] Thunderfury, Blessed Blade of the Windseeker (iLvl 80)
```

---

### Example 3: Check Item Price

**JavaScript:**

```javascript
async function checkPrice(itemId, realm) {
  const price = await client.prices.current({
    item_id: itemId,
    realm: realm
  });
  
  const gold = Math.floor(price.data.market_value / 10000);
  const silver = Math.floor((price.data.market_value % 10000) / 100);
  const copper = price.data.market_value % 100;
  
  console.log(`Market Value: ${gold}g ${silver}s ${copper}c`);
  console.log(`Min Buyout: ${formatPrice(price.data.min_buyout)}`);
  console.log(`Max Buyout: ${formatPrice(price.data.max_buyout)}`);
  console.log(`Available: ${price.data.quantity_available}`);
  console.log(`Daily Sales: ${price.data.daily_sold}`);
}

function formatPrice(copper) {
  const g = Math.floor(copper / 10000);
  const s = Math.floor((copper % 10000) / 100);
  const c = copper % 100;
  return `${g}g ${s}s ${c}c`;
}

checkPrice(19019, 'stormrage');
```

**Output:**

```
Market Value: 5000g 0s 0c
Min Buyout: 4500g 0s 0c
Max Buyout: 5500g 0s 0c
Available: 3
Daily Sales: 1
```

---

### Example 4: Set Price Alert

**JavaScript:**

```javascript
async function createPriceAlert(itemId, realm, targetPrice) {
  const alert = await client.alerts.create({
    item_id: itemId,
    realm: realm,
    trigger_type: 'below',
    target_price: targetPrice,
    notification_method: 'email'
  });
  
  console.log('Alert created successfully!');
  console.log(`Alert ID: ${alert.data.alert_id}`);
  console.log(`You'll be notified when price drops below ${formatPrice(targetPrice)}`);
}

createPriceAlert(19019, 'stormrage', 45000000); // 4500 gold
```

**Python:**

```python
def create_price_alert(item_id, realm, target_price):
    alert = client.alerts.create(
        item_id=item_id,
        realm=realm,
        trigger_type='below',
        target_price=target_price,
        notification_method='email'
    )
    
    print("Alert created successfully!")
    print(f"Alert ID: {alert['data']['alert_id']}")
    print(f"You'll be notified when price drops below {format_price(target_price)}")

create_price_alert(19019, 'stormrage', 45000000)
```

---

## Common Use Cases

### Use Case 1: Track Best Deals

Find items selling below market value for potential flipping opportunities.

```javascript
async function findBestDeals(realm, minDiscount = 0.20) {
  // Get popular items
  const popularItems = await client.items.popular({ realm });
  
  const deals = [];
  
  for (const item of popularItems.data.items) {
    const price = await client.prices.current({
      item_id: item.id,
      realm: realm
    });
    
    const auctions = await client.auctions.list({
      realm: realm,
      item_id: item.id,
      limit: 5
    });
    
    for (const auction of auctions.data.auctions) {
      const discount = (price.data.market_value - auction.buyout) / price.data.market_value;
      
      if (discount >= minDiscount) {
        deals.push({
          item: item.name,
          market_value: price.data.market_value,
          buyout: auction.buyout,
          discount: (discount * 100).toFixed(1),
          profit: price.data.market_value - auction.buyout
        });
      }
    }
  }
  
  // Sort by profit
  deals.sort((a, b) => b.profit - a.profit);
  
  console.log('Top Deals:');
  deals.slice(0, 10).forEach((deal, i) => {
    console.log(`${i + 1}. ${deal.item}`);
    console.log(`   Market: ${formatPrice(deal.market_value)}`);
    console.log(`   Buyout: ${formatPrice(deal.buyout)}`);
    console.log(`   Discount: ${deal.discount}%`);
    console.log(`   Profit: ${formatPrice(deal.profit)}\n`);
  });
}

findBestDeals('stormrage', 0.25); // 25% discount or more
```

---

### Use Case 2: Price History Analysis

Analyze price trends to determine best buying/selling times.

```javascript
async function analyzePriceTrends(itemId, realm, days = 30) {
  const history = await client.prices.history({
    item_id: itemId,
    realm: realm,
    start_date: new Date(Date.now() - days * 24 * 60 * 60 * 1000).toISOString(),
    end_date: new Date().toISOString(),
    interval: 'day'
  });
  
  const prices = history.data.prices.map(p => p.avg_buyout);
  
  // Calculate statistics
  const avg = prices.reduce((sum, p) => sum + p, 0) / prices.length;
  const min = Math.min(...prices);
  const max = Math.max(...prices);
  const current = prices[prices.length - 1];
  
  const volatility = ((max - min) / avg * 100).toFixed(2);
  const trend = current > avg ? 'INCREASING' : 'DECREASING';
  const recommendation = current < avg * 0.9 ? 'BUY' : current > avg * 1.1 ? 'SELL' : 'HOLD';
  
  console.log(`Price Analysis for Item ${itemId} on ${realm}`);
  console.log(`Period: ${days} days`);
  console.log(`\nStatistics:`);
  console.log(`  Current: ${formatPrice(current)}`);
  console.log(`  Average: ${formatPrice(avg)}`);
  console.log(`  Min: ${formatPrice(min)}`);
  console.log(`  Max: ${formatPrice(max)}`);
  console.log(`  Volatility: ${volatility}%`);
  console.log(`\nTrend: ${trend}`);
  console.log(`Recommendation: ${recommendation}`);
  
  // Plot simple ASCII chart
  console.log('\nPrice Chart (last 7 days):');
  const recentPrices = prices.slice(-7);
  plotSimpleChart(recentPrices);
}

function plotSimpleChart(prices) {
  const max = Math.max(...prices);
  const min = Math.min(...prices);
  const range = max - min;
  const height = 10;
  
  for (let i = 0; i < prices.length; i++) {
    const normalized = ((prices[i] - min) / range) * height;
    const bars = '█'.repeat(Math.round(normalized));
    console.log(`Day ${i + 1}: ${bars} ${formatPrice(prices[i])}`);
  }
}

analyzePriceTrends(19019, 'stormrage', 30);
```

**Output:**

```
Price Analysis for Item 19019 on stormrage
Period: 30 days

Statistics:
  Current: 5000g 0s 0c
  Average: 5100g 0s 0c
  Min: 4500g 0s 0c
  Max: 5600g 0s 0c
  Volatility: 21.57%

Trend: DECREASING
Recommendation: BUY

Price Chart (last 7 days):
Day 1: ████████ 5200g 0s 0c
Day 2: █████████ 5300g 0s 0c
Day 3: ████████ 5100g 0s 0c
Day 4: ███████ 5000g 0s 0c
Day 5: ██████ 4900g 0s 0c
Day 6: ██████ 4800g 0s 0c
Day 7: ██████ 5000g 0s 0c
```

---

### Use Case 3: Automated Sniping Bot

Monitor auctions in real-time and alert when good deals appear.

```javascript
class AuctionSniper {
  constructor(client, realm, targetItems) {
    this.client = client;
    this.realm = realm;
    this.targetItems = targetItems; // Array of {item_id, max_price}
    this.ws = null;
  }
  
  async start() {
    // Connect to WebSocket for real-time updates
    this.ws = await this.client.websocket.connect();
    
    // Subscribe to auction updates
    for (const target of this.targetItems) {
      this.ws.subscribe('auctions', {
        realm: this.realm,
        item_id: target.item_id
      });
    }
    
    // Handle auction updates
    this.ws.on('auction_update', (data) => {
      if (data.action === 'created') {
        this.checkAuction(data.data);
      }
    });
    
    console.log(`Sniping bot started for ${this.realm}`);
    console.log(`Monitoring ${this.targetItems.length} items`);
  }
  
  checkAuction(auction) {
    const target = this.targetItems.find(t => t.item_id === auction.item_id);
    
    if (target && auction.buyout <= target.max_price) {
      this.alert(auction, target);
    }
  }
  
  alert(auction, target) {
    const price = formatPrice(auction.buyout);
    const maxPrice = formatPrice(target.max_price);
    
    console.log('\n🔔 SNIPE OPPORTUNITY!');
    console.log(`Item: ${auction.item_id}`);
    console.log(`Price: ${price} (Target: ${maxPrice})`);
    console.log(`Quantity: ${auction.quantity}`);
    console.log(`Seller: ${auction.seller}`);
    console.log(`Time Left: ${auction.time_left}`);
    
    // Could also send push notification, play sound, etc.
    this.sendNotification(auction, target);
  }
  
  async sendNotification(auction, target) {
    // Send push notification or webhook
    // Implementation depends on your notification service
  }
  
  stop() {
    if (this.ws) {
      this.ws.disconnect();
      console.log('Sniping bot stopped');
    }
  }
}

// Usage
const sniper = new AuctionSniper(client, 'stormrage', [
  { item_id: 19019, max_price: 45000000 },  // Thunderfury < 4500g
  { item_id: 17182, max_price: 30000000 },  // Sulfuras < 3000g
  { item_id: 22589, max_price: 100000 }     // Atiesh < 10g
]);

sniper.start();

// Stop after 1 hour
setTimeout(() => sniper.stop(), 60 * 60 * 1000);
```

---

### Use Case 4: Multi-Realm Price Comparison

Compare prices across multiple realms to find arbitrage opportunities.

```python
import pandas as pd
from wow_auction_tracker import Client

class MultiRealmAnalyzer:
    def __init__(self, api_key):
        self.client = Client(api_key=api_key)
    
    def compare_prices(self, item_id, realms):
        """Compare prices for an item across multiple realms"""
        results = []
        
        for realm in realms:
            try:
                price = self.client.prices.current(
                    item_id=item_id,
                    realm=realm
                )
                
                results.append({
                    'realm': realm,
                    'market_value': price['data']['market_value'],
                    'min_buyout': price['data']['min_buyout'],
                    'quantity': price['data']['quantity_available'],
                    'sale_rate': price['data']['sale_rate']
                })
            except Exception as e:
                print(f"Error fetching data for {realm}: {e}")
        
        # Create DataFrame for analysis
        df = pd.DataFrame(results)
        df = df.sort_values('market_value')
        
        return df
    
    def find_arbitrage_opportunities(self, item_ids, realms):
        """Find items with significant price differences across realms"""
        opportunities = []
        
        for item_id in item_ids:
            df = self.compare_prices(item_id, realms)
            
            if len(df) < 2:
                continue
            
            cheapest = df.iloc[0]
            most_expensive = df.iloc[-1]
            
            profit_margin = (most_expensive['market_value'] - cheapest['market_value']) / cheapest['market_value']
            
            if profit_margin > 0.3:  # 30% profit margin
                opportunities.append({
                    'item_id': item_id,
                    'buy_realm': cheapest['realm'],
                    'buy_price': cheapest['market_value'],
                    'sell_realm': most_expensive['realm'],
                    'sell_price': most_expensive['market_value'],
                    'profit_margin': profit_margin * 100,
                    'profit_per_unit': most_expensive['market_value'] - cheapest['market_value']
                })
        
        return pd.DataFrame(opportunities).sort_values('profit_per_unit', ascending=False)

# Usage
analyzer = MultiRealmAnalyzer('YOUR_API_KEY')

realms = ['stormrage', 'area-52', 'illidan', 'tichondrius', 'malganis']
item_ids = [19019, 17182, 22589]  # Legendary items

print("Comparing prices across realms...\n")

# Compare single item
print("Thunderfury prices:")
prices = analyzer.compare_prices(19019, realms)
print(prices.to_string(index=False))

print("\n" + "="*70 + "\n")

# Find arbitrage opportunities
print("Arbitrage opportunities:")
opportunities = analyzer.find_arbitrage_opportunities(item_ids, realms)
if not opportunities.empty:
    for idx, opp in opportunities.iterrows():
        print(f"\nItem {opp['item_id']}:")
        print(f"  Buy on {opp['buy_realm']} for {opp['buy_price']:,} copper")
        print(f"  Sell on {opp['sell_realm']} for {opp['sell_price']:,} copper")
        print(f"  Profit: {opp['profit_per_unit']:,} copper ({opp['profit_margin']:.1f}%)")
else:
    print("No profitable opportunities found")
```

---

## Advanced Examples

### Advanced Example 1: Building a Trading Dashboard

Create a real-time trading dashboard with multiple data sources.

```javascript
const blessed = require('blessed');
const contrib = require('blessed-contrib');

class TradingDashboard {
  constructor(client, realm) {
    this.client = client;
    this.realm = realm;
    this.screen = blessed.screen();
    this.setupLayout();
    this.watchList = [19019, 17182, 22589];
  }
  
  setupLayout() {
    const grid = new contrib.grid({ rows: 12, cols: 12, screen: this.screen });
    
    // Price chart
    this.priceChart = grid.set(0, 0, 6, 8, contrib.line, {
      label: 'Price History',
      showLegend: true,
      style: { line: 'yellow' }
    });
    
    // Watch list
    this.watchListTable = grid.set(0, 8, 6, 4, contrib.table, {
      label: 'Watch List',
      columnWidth: [20, 15, 15]
    });
    
    // Recent auctions
    this.auctionLog = grid.set(6, 0, 6, 8, contrib.log, {
      label: 'Recent Auctions',
      bufferLength: 100
    });
    
    // Stats
    this.statsBox = grid.set(6, 8, 6, 4, blessed.box, {
      label: 'Statistics',
      content: ''
    });
    
    // Quit on Escape or Ctrl-C
    this.screen.key(['escape', 'q', 'C-c'], () => process.exit(0));
  }
  
  async start() {
    // Initial data load
    await this.updateWatchList();
    await this.updatePriceChart(this.watchList[0]);
    
    // Set up real-time updates
    this.ws = await this.client.websocket.connect();
    
    for (const itemId of this.watchList) {
      this.ws.subscribe('auctions', {
        realm: this.realm,
        item_id: itemId
      });
    }
    
    this.ws.on('auction_update', (data) => {
      this.handleAuctionUpdate(data);
    });
    
    // Periodic updates
    setInterval(() => this.updateWatchList(), 60000); // Every minute
    setInterval(() => this.updateStats(), 30000); // Every 30 seconds
    
    this.screen.render();
  }
  
  async updateWatchList() {
    const data = [];
    
    for (const itemId of this.watchList) {
      const price = await this.client.prices.current({
        item_id: itemId,
        realm: this.realm
      });
      
      const item = await this.client.items.get(itemId);
      
      data.push([
        item.data.name,
        formatPrice(price.data.market_value),
        price.data.quantity_available.toString()
      ]);
    }
    
    this.watchListTable.setData({
      headers: ['Item', 'Price', 'Available'],
      data: data
    });
    
    this.screen.render();
  }
  
  async updatePriceChart(itemId) {
    const history = await this.client.prices.history({
      item_id: itemId,
      realm: this.realm,
      start_date: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString(),
      end_date: new Date().toISOString(),
      interval: 'hour'
    });
    
    const x = history.data.prices.map((_, i) => i.toString());
    const y = history.data.prices.map(p => p.avg_buyout / 10000); // Convert to gold
    
    this.priceChart.setData([{
      title: 'Price',
      x: x,
      y: y,
      style: { line: 'yellow' }
    }]);
    
    this.screen.render();
  }
  
  handleAuctionUpdate(data) {
    if (data.action === 'created') {
      const auction = data.data;
      const msg = `NEW: ${auction.item_id} - ${formatPrice(auction.buyout)} x${auction.quantity}`;
      this.auctionLog.log(msg);
      this.screen.render();
    }
  }
  
  async updateStats() {
    // Calculate and display stats
    const stats = await this.calculateStats();
    
    this.statsBox.setContent(
      `Total Items: ${stats.totalItems}\n` +
      `Total Value: ${formatPrice(stats.totalValue)}\n` +
      `Avg Price: ${formatPrice(stats.avgPrice)}\n` +
      `Updated: ${new Date().toLocaleTimeString()}`
    );
    
    this.screen.render();
  }
  
  async calculateStats() {
    // Implement stats calculation
    return {
      totalItems: 100,
      totalValue: 5000000,
      avgPrice: 50000
    };
  }
}

// Usage
const dashboard = new TradingDashboard(client, 'stormrage');
dashboard.start();
```

---

### Advanced Example 2: Machine Learning Price Prediction

Use historical data to predict future prices with machine learning.

```python
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error
from wow_auction_tracker import Client

class PricePredictor:
    def __init__(self, api_key):
        self.client = Client(api_key=api_key)
        self.model = RandomForestRegressor(n_estimators=100, random_state=42)
    
    def fetch_training_data(self, item_id, realm, days=90):
        """Fetch historical price data for training"""
        start_date = pd.Timestamp.now() - pd.Timedelta(days=days)
        
        history = self.client.prices.history(
            item_id=item_id,
            realm=realm,
            start_date=start_date.isoformat(),
            end_date=pd.Timestamp.now().isoformat(),
            interval='hour'
        )
        
        df = pd.DataFrame(history['data']['prices'])
        df['timestamp'] = pd.to_datetime(df['timestamp'])
        df = df.sort_values('timestamp')
        
        return df
    
    def engineer_features(self, df):
        """Create features for machine learning"""
        # Time-based features
        df['hour'] = df['timestamp'].dt.hour
        df['day_of_week'] = df['timestamp'].dt.dayofweek
        df['day_of_month'] = df['timestamp'].dt.day
        
        # Price-based features
        df['price_change'] = df['avg_buyout'].diff()
        df['price_change_pct'] = df['avg_buyout'].pct_change()
        
        # Rolling statistics
        df['ma_24h'] = df['avg_buyout'].rolling(window=24).mean()
        df['ma_72h'] = df['avg_buyout'].rolling(window=72).mean()
        df['volatility_24h'] = df['avg_buyout'].rolling(window=24).std()
        
        # Lag features
        for lag in [1, 6, 12, 24]:
            df[f'lag_{lag}'] = df['avg_buyout'].shift(lag)
        
        return df.dropna()
    
    def train(self, item_id, realm):
        """Train the price prediction model"""
        print(f"Fetching training data for item {item_id} on {realm}...")
        df = self.fetch_training_data(item_id, realm)
        
        print("Engineering features...")
        df = self.engineer_features(df)
        
        # Prepare features and target
        feature_cols = [
            'hour', 'day_of_week', 'day_of_month',
            'min_buyout', 'max_buyout', 'quantity',
            'ma_24h', 'ma_72h', 'volatility_24h',
            'lag_1', 'lag_6', 'lag_12', 'lag_24'
        ]
        
        X = df[feature_cols]
        y = df['avg_buyout']
        
        # Train/test split
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=0.2, shuffle=False
        )
        
        print("Training model...")
        self.model.fit(X_train, y_train)
        
        # Evaluate
        y_pred = self.model.predict(X_test)
        mae = mean_absolute_error(y_test, y_pred)
        mape = np.mean(np.abs((y_test - y_pred) / y_test)) * 100
        
        print(f"\nModel Performance:")
        print(f"  MAE: {mae:,.0f} copper")
        print(f"  MAPE: {mape:.2f}%")
        
        # Feature importance
        feature_importance = pd.DataFrame({
            'feature': feature_cols,
            'importance': self.model.feature_importances_
        }).sort_values('importance', ascending=False)
        
        print(f"\nTop 5 Important Features:")
        print(feature_importance.head().to_string(index=False))
        
        return self.model
    
    def predict_next_24h(self, item_id, realm):
        """Predict prices for the next 24 hours"""
        # Get recent data
        df = self.fetch_training_data(item_id, realm, days=7)
        df = self.engineer_features(df)
        
        predictions = []
        last_row = df.iloc[-1]
        
        for hour in range(24):
            # Prepare features
            features = pd.DataFrame([{
                'hour': (last_row['hour'] + hour) % 24,
                'day_of_week': last_row['day_of_week'],
                'day_of_month': last_row['day_of_month'],
                'min_buyout': last_row['min_buyout'],
                'max_buyout': last_row['max_buyout'],
                'quantity': last_row['quantity'],
                'ma_24h': last_row['ma_24h'],
                'ma_72h': last_row['ma_72h'],
                'volatility_24h': last_row['volatility_24h'],
                'lag_1': last_row['avg_buyout'],
                'lag_6': last_row['lag_1'],
                'lag_12': last_row['lag_6'],
                'lag_24': last_row['lag_12']
            }])
            
            pred_price = self.model.predict(features)[0]
            predictions.append({
                'hour_ahead': hour + 1,
                'predicted_price': pred_price
            })
        
        return pd.DataFrame(predictions)

# Usage
predictor = PricePredictor('YOUR_API_KEY')

# Train model
predictor.train(19019, 'stormrage')

# Make predictions
predictions = predictor.predict_next_24h(19019, 'stormrage')
print("\nPrice Predictions for Next 24 Hours:")
print(predictions.to_string(index=False))

# Find best time to buy/sell
best_buy_time = predictions.loc[predictions['predicted_price'].idxmin()]
best_sell_time = predictions.loc[predictions['predicted_price'].idxmax()]

print(f"\nBest time to BUY: {best_buy_time['hour_ahead']} hours from now")
print(f"Predicted price: {best_buy_time['predicted_price']:,.0f} copper")
print(f"\nBest time to SELL: {best_sell_time['hour_ahead']} hours from now")
print(f"Predicted price: {best_sell_time['predicted_price']:,.0f} copper")
```

---

## Best Practices

### 1. Rate Limiting

Respect API rate limits to avoid being throttled.

```javascript
class RateLimitedClient {
  constructor(client, requestsPerMinute = 60) {
    this.client = client;
    this.requestsPerMinute = requestsPerMinute;
    this.queue = [];
    this.processing = false;
  }
  
  async request(fn) {
    return new Promise((resolve, reject) => {
      this.queue.push({ fn, resolve, reject });
      this.processQueue();
    });
  }
  
  async processQueue() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const interval = 60000 / this.requestsPerMinute;
    
    while (this.queue.length > 0) {
      const { fn, resolve, reject } = this.queue.shift();
      
      try {
        const result = await fn();
        resolve(result);
      } catch (error) {
        reject(error);
      }
      
      if (this.queue.length > 0) {
        await new Promise(r => setTimeout(r, interval));
      }
    }
    
    this.processing = false;
  }
}

// Usage
const rateLimitedClient = new RateLimitedClient(client, 60);

// Make requests through the rate limiter
const result = await rateLimitedClient.request(() => 
  client.auctions.list({ realm: 'stormrage' })
);
```

---

### 2. Error Handling

Implement robust error handling for API failures.

```javascript
async function robustAPICall(fn, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      console.error(`Attempt ${attempt} failed:`, error.message);
      
      if (error.code === 'RATE_LIMIT_EXCEEDED') {
        const waitTime = error.details.retry_after * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await new Promise(r => setTimeout(r, waitTime));
        continue;
      }
      
      if (attempt === maxRetries) {
        throw error;
      }
      
      // Exponential backoff
      const backoff = Math.pow(2, attempt) * 1000;
      console.log(`Retrying in ${backoff}ms...`);
      await new Promise(r => setTimeout(r, backoff));
    }
  }
}

// Usage
const auctions = await robustAPICall(() => 
  client.auctions.list({ realm: 'stormrage' })
);
```

---

### 3. Caching

Cache frequently accessed data to reduce API calls.

```python
from functools import lru_cache
from datetime import datetime, timedelta

class CachedClient:
    def __init__(self, client):
        self.client = client
        self.cache = {}
        self.cache_ttl = 300  # 5 minutes
    
    def get_cached(self, key, fetch_fn):
        """Get data from cache or fetch if expired"""
        now = datetime.now()
        
        if key in self.cache:
            data, timestamp = self.cache[key]
            if now - timestamp < timedelta(seconds=self.cache_ttl):
                return data
        
        # Cache miss or expired
        data = fetch_fn()
        self.cache[key] = (data, now)
        return data
    
    def get_item(self, item_id):
        """Get item data with caching"""
        key = f"item_{item_id}"
        return self.get_cached(key, lambda: self.client.items.get(item_id))
    
    def get_current_price(self, item_id, realm):
        """Get current price with caching"""
        key = f"price_{realm}_{item_id}"
        return self.get_cached(key, lambda: self.client.prices.current(
            item_id=item_id,
            realm=realm
        ))

# Usage
cached_client = CachedClient(client)

# These calls will use cache if available
item1 = cached_client.get_item(19019)
item2 = cached_client.get_item(19019)  # From cache
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Authentication Errors

**Problem:** `INVALID_API_KEY` error

**Solution:**
```javascript
// Verify API key is set correctly
console.log('API Key:', process.env.WOW_AUCTION_API_KEY?.substring(0, 10) + '...');

// Test authentication
try {
  const test = await client.auctions.list({ realm: 'stormrage', limit: 1 });
  console.log('Authentication successful!');
} catch (error) {
  if (error.code === 'INVALID_API_KEY') {
    console.error('API key is invalid. Please check your credentials.');
  }
}
```

#### Issue 2: Rate Limiting

**Problem:** `RATE_LIMIT_EXCEEDED` error

**Solution:**
```python
import time

def handle_rate_limit(client, fn, *args, **kwargs):
    """Automatically handle rate limiting"""
    while True:
        try:
            return fn(*args, **kwargs)
        except Exception as e:
            if e.code == 'RATE_LIMIT_EXCEEDED':
                wait_time = e.details.get('retry_after', 60)
                print(f"Rate limited. Waiting {wait_time} seconds...")
                time.sleep(wait_time)
            else:
                raise

# Usage
auctions = handle_rate_limit(
    client,
    client.auctions.list,
    realm='stormrage'
)
```

#### Issue 3: Missing Data

**Problem:** No results returned for a realm

**Solution:**
```javascript
// Verify realm name is correct
const realms = await client.realms.list({ region: 'us' });
const realmNames = realms.data.realms.map(r => r.slug);
console.log('Available realms:', realmNames);

// Use correct slug format
const correct = 'area-52';  // ✓
const wrong = 'Area 52';     // ✗
```

---

## Code Snippets Library

### Convert Copper to Gold/Silver/Copper

```javascript
function formatPrice(copper) {
  const gold = Math.floor(copper / 10000);
  const silver = Math.floor((copper % 10000) / 100);
  const copperRemain = copper % 100;
  return `${gold}g ${silver}s ${copperRemain}c`;
}
```

### Calculate Profit After Auction House Cut

```javascript
function calculateProfit(buyPrice, sellPrice, ahCut = 0.05) {
  const profit = sellPrice - buyPrice;
  const afterCut = profit * (1 - ahCut);
  return {
    grossProfit: profit,
    netProfit: afterCut,
    roi: (afterCut / buyPrice) * 100
  };
}

// Example
const result = calculateProfit(40000000, 50000000);
console.log(`Net profit: ${formatPrice(result.netProfit)}`);
console.log(`ROI: ${result.roi.toFixed(2)}%`);
```

### Batch Process Multiple Items

```javascript
async function batchProcess(items, processor, batchSize = 10) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => processor(item))
    );
    results.push(...batchResults);
    
    // Small delay between batches
    if (i + batchSize < items.length) {
      await new Promise(r => setTimeout(r, 1000));
    }
  }
  
  return results;
}

// Usage
const itemIds = [19019, 17182, 22589, /* ... */];
const prices = await batchProcess(itemIds, async (itemId) => {
  return client.prices.current({
    item_id: itemId,
    realm: 'stormrage'
  });
});
```

### Export Data to CSV

```python
import csv

def export_to_csv(data, filename):
    """Export auction data to CSV file"""
    if not data:
        print("No data to export")
        return
    
    keys = data[0].keys()
    
    with open(filename, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=keys)
        writer.writeheader()
        writer.writerows(data)
    
    print(f"Exported {len(data)} rows to {filename}")

# Usage
auctions = client.auctions.list(realm='stormrage', limit=1000)
export_to_csv(auctions['data']['auctions'], 'auctions.csv')
```

---

For more examples and use cases, visit:
- GitHub Examples Repository: https://github.com/wow-auction-tracker/examples
- Community Forums: https://community.wow-auction-tracker.com
- Video Tutorials: https://youtube.com/wow-auction-tracker

**Last Updated:** October 4, 2025
