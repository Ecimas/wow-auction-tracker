# WoW Auction Tracker

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/badge/node-%3E%3D16.0.0-brightgreen)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue)](https://www.typescriptlang.org/)

A comprehensive World of Warcraft auction house tracking and analysis platform that provides real-time auction data, market analysis, price alerts, and trading opportunities.

## 🌟 Features

- **Real-time Auction Data**: Live auction house data from multiple realms
- **Advanced Market Analysis**: Trend analysis, volatility metrics, and price predictions
- **Price Alerts**: Automated notifications for price changes and opportunities
- **Trading Tools**: Profit calculators, opportunity detection, and market insights
- **API-First Design**: RESTful API with comprehensive documentation
- **React Components**: Ready-to-use UI components for building auction tools
- **WebSocket Support**: Real-time updates via WebSocket connections
- **Multi-realm Support**: Track auctions across different WoW realms

## 🚀 Quick Start

### Installation

```bash
git clone https://github.com/your-org/wow-auction-tracker.git
cd wow-auction-tracker
npm install
```

### Configuration

1. **Get Blizzard API Credentials**
   - Visit [Blizzard Developer Portal](https://develop.battle.net)
   - Create an application and get your Client ID and Secret

2. **Set up Environment Variables**

```bash
cp .env.example .env
# Edit .env with your credentials
```

3. **Start the Application**

```bash
npm run dev
```

The API will be available at `http://localhost:3000`

## 📚 Documentation

### 📖 Complete Documentation

- **[API Documentation](./API.md)** - Complete API reference with endpoints, parameters, and examples
- **[Setup Guide](./SETUP.md)** - Detailed installation and configuration instructions
- **[Usage Examples](./EXAMPLES.md)** - Practical code examples and use cases
- **[Component Guide](./COMPONENTS.md)** - Frontend and backend component documentation
- **[Data Models](./MODELS.md)** - TypeScript interfaces and data structures

### 🔗 Key API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auctions` | GET | Fetch auction data with filtering |
| `/api/auctions/{itemId}` | GET | Get auctions for specific item |
| `/api/auctions/{itemId}/history` | GET | Historical price data |
| `/api/market/trends` | GET | Market trend analysis |
| `/api/market/opportunities` | GET | Profit opportunities |
| `/api/alerts` | GET/POST | Price alert management |
| `/health` | GET | Health check |

### 🛠️ Core Utilities

- **Price Calculations**: Average, median, weighted, and trimmed mean calculations
- **Data Processing**: Filtering, aggregation, and normalization utilities
- **Market Analysis**: Trend detection, volatility analysis, and pattern recognition
- **Alert Management**: Price monitoring and notification system
- **Caching**: Performance optimization with Redis caching
- **Rate Limiting**: API rate limiting and throttling

## 💡 Usage Examples

### Basic API Usage

```javascript
// Fetch auctions for a specific realm
const response = await fetch('/api/auctions?realm=Stormrage&limit=100', {
  headers: {
    'X-API-Key': 'your-api-key-here',
    'Content-Type': 'application/json'
  }
});

const data = await response.json();
console.log(`Found ${data.auctions.length} auctions`);
```

### Real-time Updates

```javascript
// WebSocket connection for live updates
const ws = new WebSocket('wss://api.wowauctiontracker.com/ws');

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  console.log('New auction:', update.data.itemName);
};
```

### Price Alert Setup

```javascript
// Create a price alert
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
```

## 🏗️ Architecture

### Technology Stack

- **Backend**: Node.js, TypeScript, Express.js
- **Database**: PostgreSQL with Prisma ORM
- **Caching**: Redis
- **Real-time**: WebSocket (Socket.io)
- **Frontend**: React, TypeScript
- **Authentication**: JWT tokens
- **API Documentation**: OpenAPI/Swagger

### System Components

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   React Client  │◄──►│  Express Server  │◄──►│  Blizzard API   │
│                 │    │                  │    │                 │
│ - Components    │    │ - REST API       │    │ - Auction Data  │
│ - Real-time UI  │    │ - WebSocket      │    │ - Item Info     │
│ - Charts        │    │ - Authentication │    │ - Realm Status  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   PostgreSQL    │    │      Redis       │    │   File System   │
│                 │    │                  │    │                 │
│ - Auctions      │    │ - Price Cache    │    │ - Logs          │
│ - Users         │    │ - Session Store  │    │ - Backups       │
│ - Alerts        │    │ - Rate Limiting  │    │ - Config        │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 🔧 Configuration

### Environment Variables

```bash
# API Configuration
PORT=3000
NODE_ENV=development

# Blizzard API
BLIZZARD_CLIENT_ID=your_client_id
BLIZZARD_CLIENT_SECRET=your_client_secret
BLIZZARD_REGION=us

# Database
DATABASE_URL=postgresql://username:password@localhost:5432/wow_auctions

# Security
JWT_SECRET=your_jwt_secret
API_ENCRYPTION_KEY=your_32_char_key

# External Services (Optional)
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
```

### Rate Limits

- **Free Tier**: 1,000 requests/hour
- **Basic Tier**: 10,000 requests/hour
- **Premium Tier**: 100,000 requests/hour
- **Enterprise**: Custom limits

## 🎯 Use Cases

### For Traders
- **Price Monitoring**: Track item prices across realms
- **Opportunity Detection**: Find profitable flips and deals
- **Market Analysis**: Understand market trends and patterns
- **Automated Alerts**: Get notified of price changes

### For Developers
- **Market Data API**: Build auction-related applications
- **Real-time Updates**: WebSocket integration for live data
- **Analytics Tools**: Market analysis and reporting
- **Trading Bots**: Automated trading systems

### For Guilds
- **Resource Tracking**: Monitor guild-needed materials
- **Price Intelligence**: Make informed purchasing decisions
- **Market Research**: Analyze market conditions
- **Bulk Operations**: Manage large auction operations

## 📊 Data Coverage

### Supported Regions
- **Americas**: US realms (Stormrage, Illidan, etc.)
- **Europe**: EU realms (Ravencrest, Sylvanas, etc.)
- **Asia-Pacific**: KR/TW realms

### Data Types
- **Auction Listings**: Current and historical auction data
- **Item Information**: Complete WoW item database
- **Price History**: Time-series price data for analysis
- **Market Metrics**: Volume, volatility, and trend data
- **Realm Information**: Server status and population data

## 🔒 Security & Privacy

- **API Key Authentication**: Secure API access
- **Rate Limiting**: Protection against abuse
- **Data Encryption**: Sensitive data protection
- **Privacy Compliance**: GDPR and CCPA compliant
- **Audit Logging**: Complete activity tracking

## 🚨 Error Handling

All API endpoints include comprehensive error handling:

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

## 🧪 Testing

```bash
# Run test suite
npm test

# Run with coverage
npm run test:coverage

# Integration tests
npm run test:integration

# Load testing
npm run test:load
```

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](./CONTRIBUTING.md) for details.

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Blizzard Entertainment** for the World of Warcraft API
- **Community Contributors** for their valuable feedback and contributions
- **Open Source Projects** that made this possible

## 📞 Support

### Getting Help

- **📖 Documentation**: Check our comprehensive docs above
- 🐛 **Issues**: [GitHub Issues](https://github.com/your-org/wow-auction-tracker/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/your-org/wow-auction-tracker/discussions)
- 📧 **Email**: support@wowauctiontracker.com

### Professional Support

For enterprise customers, we offer:
- Priority support
- Custom development
- Training and consulting
- SLA guarantees

## 🔄 Changelog

See our [Changelog](./CHANGELOG.md) for version history and updates.

## 🗺️ Roadmap

### Upcoming Features
- [ ] Mobile app (React Native)
- [ ] Advanced machine learning predictions
- [ ] Multi-region arbitrage detection
- [ ] Guild management tools
- [ ] Auction sniper tools
- [ ] Market sentiment analysis

### Recently Released
- ✅ WebSocket real-time updates
- ✅ Advanced filtering and search
- ✅ Price alert webhooks
- ✅ Market opportunity detection
- ✅ Comprehensive API documentation

---

**Made with ❤️ for the WoW community**

*Happy auction tracking! May your gold reserves grow ever larger! 🏆*