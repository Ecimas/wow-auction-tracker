# WoW Auction Tracker

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen)](https://nodejs.org)
[![Python Version](https://img.shields.io/badge/python-%3E%3D3.9-blue)](https://www.python.org)
[![API Status](https://img.shields.io/badge/API-Online-success)](https://api.wow-auction-tracker.com)

A comprehensive auction house tracking and analytics platform for World of Warcraft. Monitor prices, track trends, receive alerts, and gain market insights across all realms and regions.

## 🌟 Features

### Core Features

- **Real-time Auction Tracking** - Monitor auctions as they're posted and sold
- **Price History & Analytics** - View historical price trends and market statistics
- **Smart Alerts** - Get notified when items reach your target prices
- **Multi-Realm Support** - Track auctions across all WoW realms
- **Market Insights** - Identify profitable trading opportunities
- **REST & WebSocket APIs** - Full programmatic access to all features

### Advanced Features

- **Price Predictions** - ML-powered price forecasting
- **Arbitrage Detection** - Find cross-realm trading opportunities
- **Portfolio Tracking** - Monitor your auction house investments
- **Custom Dashboards** - Build personalized trading dashboards
- **Webhook Integrations** - Connect to external services
- **Historical Data Export** - Download data for analysis

## 📚 Documentation

- **[API Documentation](./API.md)** - Complete API reference with examples
- **[Architecture Guide](./ARCHITECTURE.md)** - System design and components
- **[Usage Guide](./USAGE.md)** - Comprehensive usage examples
- **[Contributing Guide](./CONTRIBUTING.md)** - How to contribute

## 🚀 Quick Start

### Installation

#### NPM Package

```bash
npm install wow-auction-tracker
```

#### Python Package

```bash
pip install wow-auction-tracker
```

#### CLI Tool

```bash
npm install -g wow-auction-tracker-cli
```

### Getting an API Key

1. Sign up at [https://wow-auction-tracker.com](https://wow-auction-tracker.com)
2. Navigate to your account settings
3. Generate a new API key

### Basic Usage

#### JavaScript/Node.js

```javascript
const WoWAuctionTracker = require('wow-auction-tracker');

const client = new WoWAuctionTracker('YOUR_API_KEY');

// Get current auctions
const auctions = await client.auctions.list({
  realm: 'stormrage',
  item_id: 19019,
  limit: 10
});

console.log(`Found ${auctions.data.auctions.length} auctions`);

// Get current price
const price = await client.prices.current({
  item_id: 19019,
  realm: 'stormrage'
});

console.log(`Market value: ${price.data.market_value} copper`);

// Create price alert
const alert = await client.alerts.create({
  item_id: 19019,
  realm: 'stormrage',
  trigger_type: 'below',
  target_price: 45000000,
  notification_method: 'email'
});

console.log(`Alert created: ${alert.data.alert_id}`);
```

#### Python

```python
from wow_auction_tracker import Client

client = Client(api_key='YOUR_API_KEY')

# Get current auctions
auctions = client.auctions.list(
    realm='stormrage',
    item_id=19019,
    limit=10
)

print(f"Found {len(auctions['data']['auctions'])} auctions")

# Get current price
price = client.prices.current(
    item_id=19019,
    realm='stormrage'
)

print(f"Market value: {price['data']['market_value']} copper")

# Create price alert
alert = client.alerts.create(
    item_id=19019,
    realm='stormrage',
    trigger_type='below',
    target_price=45000000,
    notification_method='email'
)

print(f"Alert created: {alert['data']['alert_id']}")
```

#### CLI

```bash
# Set your API key
export WOW_AUCTION_API_KEY="YOUR_API_KEY"

# List auctions
wow-auction auctions list --realm stormrage --item-id 19019

# Get item price
wow-auction prices current --realm stormrage --item-id 19019

# Create alert
wow-auction alerts create --realm stormrage --item-id 19019 \
  --trigger-type below --target-price 45000000
```

#### cURL

```bash
# Get auctions
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/auctions?realm=stormrage&item_id=19019"

# Get price
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/prices/current?realm=stormrage&item_id=19019"

# Create alert
curl -X POST -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"item_id":19019,"realm":"stormrage","trigger_type":"below","target_price":45000000,"notification_method":"email"}' \
  https://api.wow-auction-tracker.com/v1/alerts
```

## 💡 Use Cases

### Price Tracking

Monitor item prices across realms to find the best deals:

```javascript
async function trackPrices(itemId, realms) {
  const prices = [];
  
  for (const realm of realms) {
    const price = await client.prices.current({
      item_id: itemId,
      realm: realm
    });
    
    prices.push({
      realm: realm,
      price: price.data.market_value
    });
  }
  
  // Sort by price
  prices.sort((a, b) => a.price - b.price);
  
  console.log('Cheapest realms:');
  prices.slice(0, 3).forEach(p => {
    console.log(`${p.realm}: ${p.price} copper`);
  });
}

trackPrices(19019, ['stormrage', 'area-52', 'illidan']);
```

### Market Analysis

Analyze historical data to identify trends:

```python
import pandas as pd
import matplotlib.pyplot as plt

# Get 30 days of price history
history = client.prices.history(
    item_id=19019,
    realm='stormrage',
    start_date=(pd.Timestamp.now() - pd.Timedelta(days=30)).isoformat(),
    end_date=pd.Timestamp.now().isoformat(),
    interval='day'
)

# Convert to DataFrame
df = pd.DataFrame(history['data']['prices'])
df['timestamp'] = pd.to_datetime(df['timestamp'])

# Plot price trend
plt.figure(figsize=(12, 6))
plt.plot(df['timestamp'], df['avg_buyout'])
plt.title('Price Trend - Last 30 Days')
plt.xlabel('Date')
plt.ylabel('Price (copper)')
plt.show()
```

### Automated Sniping

Get real-time alerts for good deals:

```javascript
// Connect to WebSocket for real-time updates
const ws = await client.websocket.connect();

// Subscribe to item updates
ws.subscribe('auctions', {
  realm: 'stormrage',
  item_id: 19019
});

// Handle new auctions
ws.on('auction_update', (data) => {
  if (data.action === 'created' && data.data.buyout < 45000000) {
    console.log('🔔 DEAL ALERT!');
    console.log(`Price: ${data.data.buyout} copper`);
    console.log(`Seller: ${data.data.seller}`);
    // Send notification, play sound, etc.
  }
});
```

## 🏗️ Architecture

### Technology Stack

**Backend:**
- Node.js + Express (REST API)
- Socket.io (WebSocket)
- Python + Celery (Background workers)
- PostgreSQL (Main database)
- TimescaleDB (Time-series data)
- Redis (Caching)
- RabbitMQ (Message queue)

**Frontend:**
- React 18
- TypeScript
- Tailwind CSS
- Chart.js

**Infrastructure:**
- Docker + Kubernetes
- NGINX (Load balancer)
- Kong (API Gateway)
- Prometheus + Grafana (Monitoring)

### System Overview

```
Client → API Gateway → REST/WebSocket APIs → Services → Database
                                           ↓
                                   Message Queue → Workers → External APIs
```

For detailed architecture information, see [ARCHITECTURE.md](./ARCHITECTURE.md).

## 🔧 Development

### Prerequisites

- Node.js 18+
- Python 3.9+
- Docker & Docker Compose
- PostgreSQL 14+
- Redis 7+

### Setup

```bash
# Clone the repository
git clone https://github.com/wow-auction-tracker/wow-auction-tracker.git
cd wow-auction-tracker

# Install dependencies
npm install
cd workers && pip install -r requirements.txt && cd ..

# Set up environment
cp .env.example .env
# Edit .env with your configuration

# Start services
docker-compose up -d

# Run migrations
npm run migrate

# Start development servers
npm run dev          # API server
npm run dev:ws       # WebSocket server
cd workers && celery -A tasks worker --loglevel=info  # Workers
```

### Testing

```bash
# Run all tests
npm test

# Run tests with coverage
npm test -- --coverage

# Run specific test
npm test -- auction.service.test.ts
```

### Linting

```bash
# Run ESLint
npm run lint

# Fix linting issues
npm run lint:fix

# Format with Prettier
npm run format
```

## 📊 API Overview

### Base URL

```
https://api.wow-auction-tracker.com/v1
```

### Authentication

Include your API key in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

### Core Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auctions` | GET | List current auctions |
| `/auctions/:id` | GET | Get specific auction |
| `/items/search` | GET | Search for items |
| `/items/:id` | GET | Get item details |
| `/prices/current` | GET | Get current market price |
| `/prices/history` | GET | Get price history |
| `/alerts` | GET, POST | Manage price alerts |
| `/alerts/:id` | DELETE | Delete alert |
| `/realms` | GET | List all realms |

### WebSocket

```javascript
const ws = new WebSocket('wss://ws.wow-auction-tracker.com');

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'YOUR_API_KEY'
  }));
  
  ws.send(JSON.stringify({
    type: 'subscribe',
    channel: 'auctions',
    realm: 'stormrage'
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Update:', data);
};
```

For complete API documentation, see [API.md](./API.md).

## 💰 Pricing

| Tier | Requests/Min | Requests/Day | Price |
|------|--------------|--------------|-------|
| Free | 60 | 1,000 | $0 |
| Basic | 300 | 10,000 | $9/mo |
| Pro | 1,000 | 100,000 | $29/mo |
| Enterprise | Custom | Custom | Contact us |

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](./CONTRIBUTING.md) for details.

### Quick Contribution Steps

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🌐 Links

- **Website**: https://wow-auction-tracker.com
- **API Status**: https://status.wow-auction-tracker.com
- **Documentation**: https://docs.wow-auction-tracker.com
- **Community Forum**: https://community.wow-auction-tracker.com
- **Discord**: https://discord.gg/wow-auction-tracker
- **Twitter**: [@WoWAuctionTrack](https://twitter.com/WoWAuctionTrack)

## 📧 Support

- **Email**: support@wow-auction-tracker.com
- **Discord**: https://discord.gg/wow-auction-tracker
- **GitHub Issues**: https://github.com/wow-auction-tracker/wow-auction-tracker/issues

## ⭐ Acknowledgments

- Blizzard Entertainment for World of Warcraft and the Battle.net API
- All contributors who have helped improve this project
- The WoW community for feedback and suggestions

## 📈 Roadmap

### Q4 2025
- [ ] Mobile app (iOS/Android)
- [ ] Advanced analytics dashboard
- [ ] Machine learning price predictions
- [ ] Multi-game support (TBC, WotLK, etc.)

### Q1 2026
- [ ] Portfolio management
- [ ] Social features (sharing, following)
- [ ] Custom alerts with complex conditions
- [ ] API v2 with GraphQL

## 🔐 Security

If you discover a security vulnerability, please email security@wow-auction-tracker.com. Do not create a public issue.

---

**Made with ❤️ by the WoW Auction Tracker team**

**Last Updated:** October 4, 2025