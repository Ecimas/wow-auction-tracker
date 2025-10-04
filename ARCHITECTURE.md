# Architecture Documentation

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [Technology Stack](#technology-stack)
6. [Database Schema](#database-schema)
7. [Microservices](#microservices)
8. [Caching Strategy](#caching-strategy)
9. [Security Architecture](#security-architecture)
10. [Deployment Architecture](#deployment-architecture)
11. [Scalability Considerations](#scalability-considerations)

---

## System Overview

The WoW Auction Tracker is a distributed system designed to track, analyze, and provide real-time insights into World of Warcraft auction house data across multiple realms and regions.

### Key Features

- **Real-time Auction Tracking**: Monitor auctions as they're posted and sold
- **Historical Data Analysis**: Analyze price trends over time
- **Price Alerts**: Notify users when items reach target prices
- **Market Analytics**: Provide insights into market trends and opportunities
- **Multi-realm Support**: Track auctions across all WoW realms

### Design Principles

1. **Scalability**: Handle millions of auctions across hundreds of realms
2. **Real-time Processing**: Process and distribute updates with minimal latency
3. **High Availability**: Ensure 99.9% uptime with redundancy
4. **Data Integrity**: Maintain accurate and consistent data
5. **Performance**: Fast query responses even with large datasets

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
├─────────────┬─────────────────┬─────────────────┬───────────────┤
│  Web App    │   Mobile App    │   CLI Tool      │  Third-party  │
│  (React)    │   (React Native)│   (Python)      │  Integrations │
└─────────────┴─────────────────┴─────────────────┴───────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                         API Gateway                              │
│                    (Rate Limiting, Auth)                         │
└─────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
┌───────────────────┐ ┌──────────────┐ ┌──────────────────┐
│   REST API        │ │  GraphQL API │ │  WebSocket API   │
│   (Express.js)    │ │  (Apollo)    │ │  (Socket.io)     │
└───────────────────┘ └──────────────┘ └──────────────────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
┌───────────────────┐ ┌──────────────┐ ┌──────────────────┐
│  Auction Service  │ │ Price Service│ │  Alert Service   │
└───────────────────┘ └──────────────┘ └──────────────────┘
        │                     │                 │
        └─────────────────────┼─────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Message Queue (RabbitMQ)                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
┌────────────────────┐ ┌─────────────┐ ┌──────────────────┐
│ Data Ingestion     │ │  Analytics  │ │  Notification    │
│ Worker             │ │  Worker     │ │  Worker          │
└────────────────────┘ └─────────────┘ └──────────────────┘
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
┌────────────────────┐ ┌─────────────┐ ┌──────────────────┐
│   PostgreSQL       │ │   Redis     │ │  TimescaleDB     │
│   (Main DB)        │ │   (Cache)   │ │  (Time Series)   │
└────────────────────┘ └─────────────┘ └──────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              External Services (Blizzard API)                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. API Gateway

**Purpose**: Single entry point for all client requests

**Responsibilities**:
- Authentication and authorization
- Rate limiting
- Request routing
- Load balancing
- SSL termination
- API versioning

**Technology**: NGINX / Kong

**Configuration Example**:

```yaml
# api-gateway-config.yaml
services:
  - name: auction-api
    url: http://auction-service:3000
    routes:
      - paths: ["/v1/auctions"]
        methods: ["GET", "POST"]
    plugins:
      - name: rate-limiting
        config:
          minute: 60
          hour: 1000
      - name: jwt
        config:
          secret_is_base64: false
```

---

### 2. REST API Service

**Purpose**: Handle synchronous HTTP requests

**Responsibilities**:
- Process GET/POST/PUT/DELETE requests
- Validate input data
- Query databases and caches
- Return formatted responses
- Handle pagination

**Technology**: Node.js + Express.js + TypeScript

**Key Modules**:

```typescript
// src/api/routes/auctions.ts
import { Router } from 'express';
import { AuctionController } from '../controllers/auction.controller';
import { authenticate } from '../middleware/auth.middleware';
import { validateQuery } from '../middleware/validation.middleware';

const router = Router();
const auctionController = new AuctionController();

router.get(
  '/auctions',
  authenticate,
  validateQuery,
  auctionController.getAuctions
);

router.get(
  '/auctions/:id',
  authenticate,
  auctionController.getAuctionById
);

export default router;
```

**Controller Example**:

```typescript
// src/api/controllers/auction.controller.ts
export class AuctionController {
  private auctionService: AuctionService;
  private cacheService: CacheService;

  constructor() {
    this.auctionService = new AuctionService();
    this.cacheService = new CacheService();
  }

  async getAuctions(req: Request, res: Response) {
    const { realm, item_id, page, limit } = req.query;
    
    // Check cache first
    const cacheKey = `auctions:${realm}:${item_id}:${page}`;
    const cached = await this.cacheService.get(cacheKey);
    
    if (cached) {
      return res.json(cached);
    }

    // Query database
    const auctions = await this.auctionService.findAuctions({
      realm,
      itemId: item_id,
      page: Number(page) || 1,
      limit: Number(limit) || 100
    });

    // Cache result
    await this.cacheService.set(cacheKey, auctions, 300); // 5 min TTL

    return res.json({
      success: true,
      data: auctions
    });
  }
}
```

---

### 3. WebSocket Service

**Purpose**: Provide real-time auction updates

**Responsibilities**:
- Maintain persistent connections
- Subscribe/unsubscribe to channels
- Broadcast real-time updates
- Handle connection management

**Technology**: Socket.io / ws

**Implementation Example**:

```typescript
// src/websocket/server.ts
import { Server } from 'socket.io';
import { authenticateSocket } from './middleware/auth';
import { RedisAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

export class WebSocketServer {
  private io: Server;
  private redis: RedisClient;

  constructor(httpServer: any) {
    this.io = new Server(httpServer, {
      cors: { origin: '*' }
    });

    this.setupRedisAdapter();
    this.setupMiddleware();
    this.setupEventHandlers();
  }

  private async setupRedisAdapter() {
    const pubClient = createClient({ url: 'redis://localhost:6379' });
    const subClient = pubClient.duplicate();
    
    await Promise.all([pubClient.connect(), subClient.connect()]);
    
    this.io.adapter(RedisAdapter(pubClient, subClient));
  }

  private setupMiddleware() {
    this.io.use(authenticateSocket);
  }

  private setupEventHandlers() {
    this.io.on('connection', (socket) => {
      console.log(`Client connected: ${socket.id}`);

      socket.on('subscribe', (data) => {
        const { channel, realm, item_id } = data;
        const room = `${channel}:${realm}:${item_id || '*'}`;
        socket.join(room);
        console.log(`Client ${socket.id} subscribed to ${room}`);
      });

      socket.on('unsubscribe', (data) => {
        const { channel, realm, item_id } = data;
        const room = `${channel}:${realm}:${item_id || '*'}`;
        socket.leave(room);
      });

      socket.on('disconnect', () => {
        console.log(`Client disconnected: ${socket.id}`);
      });
    });
  }

  public broadcastAuctionUpdate(realm: string, auction: any) {
    const room = `auctions:${realm}:${auction.item_id}`;
    this.io.to(room).emit('auction_update', {
      type: 'auction_update',
      action: 'created',
      data: auction,
      timestamp: new Date().toISOString()
    });
  }
}
```

---

### 4. Auction Service

**Purpose**: Core business logic for auction management

**Responsibilities**:
- Query and filter auctions
- Track auction lifecycle
- Calculate market statistics
- Integrate with Blizzard API

**Technology**: Node.js + TypeScript

**Service Example**:

```typescript
// src/services/auction.service.ts
export class AuctionService {
  private db: Database;
  private blizzardAPI: BlizzardAPIClient;

  constructor() {
    this.db = Database.getInstance();
    this.blizzardAPI = new BlizzardAPIClient();
  }

  async findAuctions(filters: AuctionFilters): Promise<AuctionResponse> {
    const { realm, itemId, page, limit, minPrice, maxPrice } = filters;

    const query = this.db
      .select('*')
      .from('auctions')
      .where('realm', realm)
      .andWhere('status', 'active');

    if (itemId) {
      query.andWhere('item_id', itemId);
    }

    if (minPrice) {
      query.andWhere('buyout', '>=', minPrice);
    }

    if (maxPrice) {
      query.andWhere('buyout', '<=', maxPrice);
    }

    const offset = (page - 1) * limit;
    const auctions = await query.limit(limit).offset(offset);
    const total = await this.getAuctionCount(filters);

    return {
      auctions,
      pagination: {
        page,
        limit,
        total,
        total_pages: Math.ceil(total / limit)
      }
    };
  }

  async syncAuctionsFromBlizzard(realm: string): Promise<void> {
    const auctionData = await this.blizzardAPI.getAuctions(realm);
    
    // Process in batches
    const batchSize = 1000;
    for (let i = 0; i < auctionData.auctions.length; i += batchSize) {
      const batch = auctionData.auctions.slice(i, i + batchSize);
      await this.processBatch(batch, realm);
    }
  }

  private async processBatch(auctions: any[], realm: string): Promise<void> {
    const operations = auctions.map(auction => ({
      updateOne: {
        filter: { auction_id: auction.id, realm },
        update: {
          $set: {
            item_id: auction.item.id,
            buyout: auction.buyout,
            bid: auction.bid,
            quantity: auction.quantity,
            time_left: auction.time_left,
            seller: auction.seller,
            updated_at: new Date()
          }
        },
        upsert: true
      }
    }));

    await this.db.collection('auctions').bulkWrite(operations);
  }
}
```

---

### 5. Price Service

**Purpose**: Calculate and track price history

**Responsibilities**:
- Calculate market prices
- Store historical data
- Generate price trends
- Provide statistical analysis

**Technology**: Node.js + TypeScript + TimescaleDB

**Service Example**:

```typescript
// src/services/price.service.ts
export class PriceService {
  private timeseriesDB: TimescaleDB;
  private cache: RedisClient;

  async getCurrentPrice(itemId: number, realm: string): Promise<MarketPrice> {
    // Check cache
    const cacheKey = `price:${realm}:${itemId}`;
    const cached = await this.cache.get(cacheKey);
    if (cached) return JSON.parse(cached);

    // Calculate from recent auctions
    const recentAuctions = await this.getRecentAuctions(itemId, realm);
    
    const marketPrice: MarketPrice = {
      item_id: itemId,
      realm,
      market_value: this.calculateMarketValue(recentAuctions),
      min_buyout: Math.min(...recentAuctions.map(a => a.buyout)),
      avg_buyout: this.calculateAverage(recentAuctions),
      max_buyout: Math.max(...recentAuctions.map(a => a.buyout)),
      quantity_available: recentAuctions.reduce((sum, a) => sum + a.quantity, 0),
      daily_sold: await this.getDailySold(itemId, realm),
      sale_rate: await this.calculateSaleRate(itemId, realm),
      last_updated: new Date()
    };

    // Cache for 5 minutes
    await this.cache.setex(cacheKey, 300, JSON.stringify(marketPrice));

    return marketPrice;
  }

  async getPriceHistory(
    itemId: number,
    realm: string,
    startDate: Date,
    endDate: Date,
    interval: string
  ): Promise<PriceHistory[]> {
    const query = `
      SELECT 
        time_bucket($1, timestamp) AS timestamp,
        MIN(buyout) as min_buyout,
        AVG(buyout) as avg_buyout,
        MAX(buyout) as max_buyout,
        SUM(quantity) as quantity
      FROM auction_snapshots
      WHERE item_id = $2 
        AND realm = $3
        AND timestamp >= $4
        AND timestamp <= $5
      GROUP BY time_bucket($1, timestamp)
      ORDER BY timestamp ASC
    `;

    const result = await this.timeseriesDB.query(query, [
      interval,
      itemId,
      realm,
      startDate,
      endDate
    ]);

    return result.rows;
  }

  private calculateMarketValue(auctions: Auction[]): number {
    // Use weighted average based on quantity
    const totalValue = auctions.reduce(
      (sum, a) => sum + (a.buyout * a.quantity),
      0
    );
    const totalQuantity = auctions.reduce((sum, a) => sum + a.quantity, 0);
    return totalValue / totalQuantity;
  }
}
```

---

### 6. Alert Service

**Purpose**: Manage price alerts and notifications

**Responsibilities**:
- Create and manage alerts
- Monitor price conditions
- Trigger notifications
- Handle multiple notification channels

**Technology**: Node.js + TypeScript

**Service Example**:

```typescript
// src/services/alert.service.ts
export class AlertService {
  private db: Database;
  private notificationService: NotificationService;
  private priceService: PriceService;

  async createAlert(userId: string, alertData: CreateAlertDTO): Promise<Alert> {
    const alert = await this.db.insert('alerts', {
      user_id: userId,
      item_id: alertData.item_id,
      realm: alertData.realm,
      trigger_type: alertData.trigger_type,
      target_price: alertData.target_price,
      notification_method: alertData.notification_method,
      webhook_url: alertData.webhook_url,
      status: 'active',
      created_at: new Date()
    });

    // Subscribe to price updates for this item
    await this.subscribeToPriceUpdates(alert);

    return alert;
  }

  async checkAlerts(itemId: number, realm: string, currentPrice: number): Promise<void> {
    const activeAlerts = await this.db
      .select('*')
      .from('alerts')
      .where('item_id', itemId)
      .andWhere('realm', realm)
      .andWhere('status', 'active');

    for (const alert of activeAlerts) {
      const triggered = this.checkTriggerCondition(
        alert.trigger_type,
        currentPrice,
        alert.target_price
      );

      if (triggered) {
        await this.triggerAlert(alert, currentPrice);
      }
    }
  }

  private checkTriggerCondition(
    triggerType: string,
    currentPrice: number,
    targetPrice: number
  ): boolean {
    if (triggerType === 'below') {
      return currentPrice <= targetPrice;
    } else if (triggerType === 'above') {
      return currentPrice >= targetPrice;
    }
    return false;
  }

  private async triggerAlert(alert: Alert, currentPrice: number): Promise<void> {
    // Send notification
    await this.notificationService.send({
      userId: alert.user_id,
      method: alert.notification_method,
      webhookUrl: alert.webhook_url,
      message: {
        title: 'Price Alert Triggered',
        body: `Item ${alert.item_id} on ${alert.realm} is now ${currentPrice} copper`,
        data: { alert, currentPrice }
      }
    });

    // Update alert status
    await this.db
      .update('alerts')
      .set({
        status: 'triggered',
        triggered_at: new Date(),
        triggered_price: currentPrice
      })
      .where('alert_id', alert.alert_id);
  }
}
```

---

### 7. Data Ingestion Worker

**Purpose**: Fetch and process auction data from Blizzard API

**Responsibilities**:
- Poll Blizzard API for updates
- Process and normalize data
- Detect changes and updates
- Publish events to message queue

**Technology**: Python + Celery

**Worker Example**:

```python
# workers/data_ingestion.py
from celery import Celery
import requests
from datetime import datetime

app = Celery('data_ingestion', broker='amqp://localhost')

class BlizzardAPIClient:
    def __init__(self, client_id, client_secret):
        self.client_id = client_id
        self.client_secret = client_secret
        self.access_token = None
        self.authenticate()
    
    def authenticate(self):
        """Get OAuth access token"""
        response = requests.post(
            'https://oauth.battle.net/token',
            data={'grant_type': 'client_credentials'},
            auth=(self.client_id, self.client_secret)
        )
        self.access_token = response.json()['access_token']
    
    def get_auctions(self, realm_id, region='us'):
        """Fetch auction data for a realm"""
        url = f'https://{region}.api.blizzard.com/data/wow/connected-realm/{realm_id}/auctions'
        headers = {'Authorization': f'Bearer {self.access_token}'}
        
        response = requests.get(url, headers=headers)
        return response.json()

@app.task
def ingest_realm_auctions(realm_id, region='us'):
    """Celery task to ingest auctions for a realm"""
    client = BlizzardAPIClient(
        client_id=os.getenv('BLIZZARD_CLIENT_ID'),
        client_secret=os.getenv('BLIZZARD_CLIENT_SECRET')
    )
    
    # Fetch auctions
    auction_data = client.get_auctions(realm_id, region)
    
    # Process auctions
    processed = process_auction_data(auction_data, realm_id)
    
    # Store in database
    store_auctions(processed)
    
    # Publish events
    publish_auction_updates(processed)
    
    return {
        'realm_id': realm_id,
        'auctions_processed': len(processed),
        'timestamp': datetime.now().isoformat()
    }

@app.task
def schedule_realm_ingestion():
    """Schedule ingestion for all realms"""
    realms = get_all_realms()
    
    for realm in realms:
        ingest_realm_auctions.delay(realm['id'], realm['region'])
```

---

### 8. Analytics Worker

**Purpose**: Process and aggregate data for analytics

**Responsibilities**:
- Calculate statistics
- Generate reports
- Identify trends
- Create data summaries

**Technology**: Python + Pandas

**Worker Example**:

```python
# workers/analytics.py
import pandas as pd
from sqlalchemy import create_engine

class AnalyticsWorker:
    def __init__(self, db_url):
        self.engine = create_engine(db_url)
    
    def calculate_daily_statistics(self, date):
        """Calculate daily market statistics"""
        query = f"""
        SELECT 
            item_id,
            realm,
            MIN(buyout) as min_price,
            AVG(buyout) as avg_price,
            MAX(buyout) as max_price,
            COUNT(*) as auction_count,
            SUM(quantity) as total_quantity
        FROM auctions
        WHERE DATE(created_at) = '{date}'
        GROUP BY item_id, realm
        """
        
        df = pd.read_sql(query, self.engine)
        
        # Calculate additional metrics
        df['price_volatility'] = (df['max_price'] - df['min_price']) / df['avg_price']
        df['date'] = date
        
        # Store results
        df.to_sql('daily_statistics', self.engine, if_exists='append', index=False)
        
        return df
    
    def identify_trending_items(self, region='us'):
        """Identify items with unusual price movements"""
        query = """
        WITH price_changes AS (
            SELECT 
                item_id,
                realm,
                AVG(buyout) as current_avg,
                LAG(AVG(buyout)) OVER (PARTITION BY item_id, realm ORDER BY date) as prev_avg
            FROM daily_statistics
            WHERE region = %s
            GROUP BY item_id, realm, date
        )
        SELECT 
            item_id,
            realm,
            current_avg,
            prev_avg,
            ((current_avg - prev_avg) / prev_avg * 100) as price_change_pct
        FROM price_changes
        WHERE ABS((current_avg - prev_avg) / prev_avg) > 0.20
        ORDER BY ABS(price_change_pct) DESC
        LIMIT 100
        """
        
        return pd.read_sql(query, self.engine, params=[region])
```

---

## Data Flow

### 1. Auction Data Ingestion Flow

```
Blizzard API → Data Ingestion Worker → Message Queue → Database
                                              ↓
                                     WebSocket Service → Clients
                                              ↓
                                        Alert Service
```

**Steps**:
1. Scheduled task triggers data ingestion worker
2. Worker fetches auction data from Blizzard API
3. Data is normalized and validated
4. Changes are detected by comparing with existing data
5. Updates published to message queue
6. Database updated with new/changed auctions
7. WebSocket service broadcasts updates to subscribed clients
8. Alert service checks if any alerts should be triggered

---

### 2. Price Query Flow

```
Client → API Gateway → REST API → Cache (Redis)
                                      ↓ (cache miss)
                                 Database → Cache → Client
```

**Steps**:
1. Client requests price data
2. Request passes through API gateway (auth + rate limiting)
3. REST API checks Redis cache
4. If cache hit, return immediately
5. If cache miss, query database
6. Store result in cache with TTL
7. Return formatted response to client

---

### 3. Alert Creation and Trigger Flow

```
Client → API → Alert Service → Database
                      ↓
              Price Monitor → Notification Service → User
```

**Steps**:
1. User creates alert via API
2. Alert stored in database
3. Price monitor subscribes to price updates for the item
4. When price update received, check alert conditions
5. If triggered, notification service sends notification
6. Alert status updated to "triggered"

---

## Technology Stack

### Backend

| Component | Technology | Purpose |
|-----------|------------|---------|
| API Server | Node.js + Express | REST API endpoints |
| WebSocket Server | Socket.io | Real-time updates |
| Workers | Python + Celery | Background processing |
| Message Queue | RabbitMQ | Async communication |
| Task Scheduler | Celery Beat | Scheduled tasks |

### Database

| Component | Technology | Purpose |
|-----------|------------|---------|
| Primary Database | PostgreSQL 14+ | Relational data storage |
| Time Series DB | TimescaleDB | Price history data |
| Cache | Redis 7+ | Fast data access |
| Search Engine | Elasticsearch | Full-text search |

### Infrastructure

| Component | Technology | Purpose |
|-----------|------------|---------|
| Container Platform | Docker | Containerization |
| Orchestration | Kubernetes | Container orchestration |
| Load Balancer | NGINX | Traffic distribution |
| API Gateway | Kong | API management |
| Monitoring | Prometheus + Grafana | Metrics and dashboards |
| Logging | ELK Stack | Centralized logging |
| CDN | CloudFlare | Static asset delivery |

### Frontend

| Component | Technology | Purpose |
|-----------|------------|---------|
| Web Framework | React 18 | User interface |
| State Management | Redux Toolkit | Application state |
| Styling | Tailwind CSS | Styling framework |
| Charts | Chart.js | Data visualization |
| HTTP Client | Axios | API requests |

---

## Database Schema

### PostgreSQL Schema

```sql
-- Users table
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    api_key VARCHAR(255) UNIQUE,
    tier VARCHAR(50) DEFAULT 'free',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Realms table
CREATE TABLE realms (
    realm_id INTEGER PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) NOT NULL,
    region VARCHAR(10) NOT NULL,
    locale VARCHAR(10),
    timezone VARCHAR(50),
    connected_realm_id INTEGER,
    population VARCHAR(20),
    type VARCHAR(20),
    UNIQUE(slug, region)
);

-- Items table
CREATE TABLE items (
    item_id INTEGER PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    quality VARCHAR(20),
    item_level INTEGER,
    required_level INTEGER,
    item_class VARCHAR(50),
    item_subclass VARCHAR(50),
    icon VARCHAR(255),
    description TEXT,
    sell_price INTEGER,
    is_auctionable BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Auctions table
CREATE TABLE auctions (
    auction_id BIGINT PRIMARY KEY,
    item_id INTEGER REFERENCES items(item_id),
    realm_id INTEGER REFERENCES realms(realm_id),
    buyout BIGINT,
    bid BIGINT,
    quantity INTEGER,
    time_left VARCHAR(20),
    seller VARCHAR(255),
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_auctions_item_realm (item_id, realm_id, status),
    INDEX idx_auctions_status (status),
    INDEX idx_auctions_created (created_at)
);

-- Alerts table
CREATE TABLE alerts (
    alert_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    item_id INTEGER REFERENCES items(item_id),
    realm_id INTEGER REFERENCES realms(realm_id),
    trigger_type VARCHAR(20) NOT NULL,
    target_price BIGINT NOT NULL,
    notification_method VARCHAR(50) NOT NULL,
    webhook_url VARCHAR(500),
    status VARCHAR(20) DEFAULT 'active',
    triggered_at TIMESTAMP,
    triggered_price BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_alerts_user (user_id, status),
    INDEX idx_alerts_item_realm (item_id, realm_id, status)
);

-- API usage tracking
CREATE TABLE api_usage (
    usage_id BIGSERIAL PRIMARY KEY,
    user_id UUID REFERENCES users(user_id),
    endpoint VARCHAR(255),
    method VARCHAR(10),
    status_code INTEGER,
    response_time INTEGER,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_usage_user_timestamp (user_id, timestamp),
    INDEX idx_usage_timestamp (timestamp)
);
```

### TimescaleDB Schema

```sql
-- Price snapshots (hypertable for time-series data)
CREATE TABLE auction_snapshots (
    time TIMESTAMPTZ NOT NULL,
    item_id INTEGER NOT NULL,
    realm_id INTEGER NOT NULL,
    buyout BIGINT,
    quantity INTEGER,
    auction_count INTEGER
);

-- Convert to hypertable
SELECT create_hypertable('auction_snapshots', 'time');

-- Create indexes
CREATE INDEX idx_snapshots_item_realm_time 
ON auction_snapshots (item_id, realm_id, time DESC);

-- Continuous aggregate for daily stats
CREATE MATERIALIZED VIEW daily_price_stats
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 day', time) AS day,
    item_id,
    realm_id,
    MIN(buyout) as min_price,
    AVG(buyout) as avg_price,
    MAX(buyout) as max_price,
    SUM(quantity) as total_quantity
FROM auction_snapshots
GROUP BY day, item_id, realm_id;
```

---

## Caching Strategy

### Cache Layers

1. **Application Cache (Redis)**
   - Current prices (5 min TTL)
   - Auction listings (2 min TTL)
   - Item data (1 hour TTL)
   - User sessions (24 hour TTL)

2. **CDN Cache (CloudFlare)**
   - Static assets (7 days)
   - API responses for public endpoints (1 min)
   - Image assets (30 days)

### Cache Invalidation

```typescript
// src/services/cache.service.ts
export class CacheService {
  private redis: RedisClient;

  async invalidateAuctionCache(realm: string, itemId?: number) {
    const pattern = itemId 
      ? `auctions:${realm}:${itemId}:*`
      : `auctions:${realm}:*`;
    
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }

  async invalidatePriceCache(realm: string, itemId: number) {
    await this.redis.del(`price:${realm}:${itemId}`);
  }
}
```

---

## Security Architecture

### Authentication

- JWT tokens for API authentication
- OAuth 2.0 for third-party integrations
- API keys for programmatic access
- Session tokens for web application

### Authorization

- Role-based access control (RBAC)
- Resource-level permissions
- Rate limiting per tier
- IP whitelisting for enterprise

### Data Security

- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- Database connection encryption
- Secure password hashing (bcrypt)

### API Security

```typescript
// src/middleware/auth.middleware.ts
export async function authenticate(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({
      success: false,
      error: { code: 'MISSING_TOKEN', message: 'Authentication required' }
    });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = await getUserById(decoded.userId);
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      error: { code: 'INVALID_TOKEN', message: 'Invalid or expired token' }
    });
  }
}
```

---

## Deployment Architecture

### Kubernetes Deployment

```yaml
# k8s/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auction-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auction-api
  template:
    metadata:
      labels:
        app: auction-api
    spec:
      containers:
      - name: api
        image: wow-auction-tracker/api:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: REDIS_URL
          value: redis://redis-service:6379
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## Scalability Considerations

### Horizontal Scaling

- API servers: Auto-scale based on CPU/memory
- Workers: Scale based on queue depth
- Database: Read replicas for query distribution
- Cache: Redis cluster with sharding

### Performance Optimization

- Database indexing strategy
- Query optimization and caching
- Connection pooling
- Batch processing for bulk operations
- Async processing for heavy operations

### Monitoring and Alerting

```yaml
# prometheus/alerts.yaml
groups:
  - name: auction-tracker
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        annotations:
          summary: "High error rate detected"
      
      - alert: SlowResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 1
        annotations:
          summary: "95th percentile response time > 1s"
      
      - alert: HighMemoryUsage
        expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
        annotations:
          summary: "Container memory usage > 90%"
```

---

**Last Updated:** October 4, 2025
