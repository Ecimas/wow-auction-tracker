# WoW Auction Tracker - Data Models

## Overview

This document defines all data models, interfaces, and type definitions used throughout the WoW Auction Tracker application. These models provide type safety and ensure consistent data structures across the application.

## Core Data Types

### Basic Types

```typescript
// Unique identifiers
type AuctionId = string;
type ItemId = number;
type RealmId = string;
type UserId = string;

// Time representations
type Timestamp = Date;
type UnixTimestamp = number;
type ISODateString = string;

// Price representations
type GoldAmount = number;
type CopperAmount = number;

// Enumerations
type AuctionStatus = 'ACTIVE' | 'SOLD' | 'EXPIRED' | 'CANCELLED';
type TimeLeft = 'SHORT' | 'MEDIUM' | 'LONG' | 'VERY_LONG';
type TrendDirection = 'UP' | 'DOWN' | 'STABLE';
type AlertType = 'BELOW' | 'ABOVE' | 'CHANGE';
type SortOrder = 'ASC' | 'DESC';
```

## Auction Data Models

### AuctionItem

Core auction data structure representing a single auction listing.

```typescript
interface AuctionItem {
  // Primary identifiers
  id: AuctionId;
  itemId: ItemId;

  // Item details
  name: string;
  icon: string;
  quality: ItemQuality;
  level: number;
  requiredLevel: number;

  // Auction details
  quantity: number;
  buyoutPrice: GoldAmount;
  bidPrice: GoldAmount;
  currentBid?: GoldAmount;

  // Timing
  timeLeft: TimeLeft;
  postedAt: Timestamp;
  expiresAt: Timestamp;

  // Seller information
  seller: string;
  sellerId?: UserId;

  // Location
  realm: string;
  realmId: RealmId;

  // Additional metadata
  petSpeciesId?: number;
  petBreedId?: number;
  bonusIds?: number[];
  modifiers?: ItemModifier[];
}
```

### ItemQuality

Enumeration of item quality levels.

```typescript
enum ItemQuality {
  POOR = 0,
  COMMON = 1,
  UNCOMMON = 2,
  RARE = 3,
  EPIC = 4,
  LEGENDARY = 5,
  ARTIFACT = 6,
  HEIRLOOM = 7
}
```

### ItemModifier

Represents item bonuses and modifiers.

```typescript
interface ItemModifier {
  type: 'BONUS' | 'SOCKET' | 'ENCHANT' | 'GEM';
  value: number;
  description: string;
}
```

## Market Analysis Models

### MarketAnalysis

Comprehensive market analysis for an item.

```typescript
interface MarketAnalysis {
  itemId: ItemId;

  // Price statistics
  averagePrice: GoldAmount;
  medianPrice: GoldAmount;
  modePrice: GoldAmount; // Most common price
  weightedAverage: GoldAmount;

  // Price range
  minPrice: GoldAmount;
  maxPrice: GoldAmount;
  priceRange: GoldAmount;

  // Distribution metrics
  standardDeviation: number;
  variance: number;
  skewness: number;
  kurtosis: number;

  // Volume metrics
  totalQuantity: number;
  uniqueListings: number;
  totalSellers: number;

  // Trend analysis
  trend: TrendDirection;
  trendStrength: number; // 0-1
  trendDuration: number; // days

  // Volatility metrics
  volatility: number;
  beta: number; // Market correlation

  // Confidence metrics
  confidence: number; // 0-1
  sampleSize: number;
  lastUpdated: Timestamp;

  // Percentiles
  percentiles: {
    p25: GoldAmount; // 25th percentile
    p75: GoldAmount; // 75th percentile
    p90: GoldAmount; // 90th percentile
    p95: GoldAmount; // 95th percentile
  };
}
```

### PricePoint

Individual price data point for historical analysis.

```typescript
interface PricePoint {
  timestamp: Timestamp;
  price: GoldAmount;
  quantity: number;
  volume: GoldAmount; // price * quantity
  source: 'AUCTION' | 'VENDOR' | 'CRAFTING';
}
```

### PriceHistory

Historical price data for trend analysis.

```typescript
interface PriceHistory {
  itemId: ItemId;
  realm: string;

  // Time series data
  daily: PricePoint[];
  weekly: PricePoint[];
  monthly: PricePoint[];

  // Aggregated statistics
  overall: {
    minPrice: GoldAmount;
    maxPrice: GoldAmount;
    averagePrice: GoldAmount;
    totalVolume: GoldAmount;
  };

  // Trend data
  trends: {
    shortTerm: TrendInfo; // 7 days
    mediumTerm: TrendInfo; // 30 days
    longTerm: TrendInfo; // 90 days
  };

  lastUpdated: Timestamp;
}
```

### TrendInfo

Trend analysis information.

```typescript
interface TrendInfo {
  direction: TrendDirection;
  strength: number; // 0-1
  slope: number; // Price change per day
  rSquared: number; // Goodness of fit
  periods: number; // Number of periods analyzed
}
```

## Opportunity Models

### MarketOpportunity

Represents a potential profit opportunity.

```typescript
interface MarketOpportunity {
  id: string;

  // Item information
  itemId: ItemId;
  itemName: string;
  itemIcon: string;
  itemQuality: ItemQuality;

  // Price information
  buyPrice: GoldAmount;
  sellPrice: GoldAmount;
  profit: GoldAmount;
  margin: number; // Percentage

  // Investment metrics
  investment: GoldAmount;
  roi: number; // Return on investment percentage
  paybackPeriod: number; // Days to recover investment

  // Risk metrics
  risk: 'LOW' | 'MEDIUM' | 'HIGH';
  confidence: number; // 0-1
  volatility: number;

  // Market conditions
  competition: number; // Number of competing listings
  demand: 'LOW' | 'MEDIUM' | 'HIGH';
  supply: 'LOW' | 'MEDIUM' | 'HIGH';

  // Timing
  timeToSell: number; // Estimated days
  bestTimeToPost: string; // Optimal posting time

  // Location
  realm: string;
  region: string;

  // Metadata
  discoveredAt: Timestamp;
  expiresAt?: Timestamp;
  source: 'ANALYSIS' | 'HISTORICAL' | 'USER_TIP';
}
```

### FlipOpportunity

Specific type of opportunity for item flipping.

```typescript
interface FlipOpportunity extends MarketOpportunity {
  type: 'FLIP';

  // Flip-specific data
  priceDiscrepancy: number; // Price difference across realms
  sourceRealm: string;
  targetRealm: string;

  // Transfer costs
  transferFee: GoldAmount;
  netProfit: GoldAmount;

  // Transfer logistics
  transferTime: number; // Hours
  transferMethod: 'MAIL' | 'TRADE' | 'AH_CUTOFF';
}
```

## Alert Models

### PriceAlert

User-configured price monitoring alert.

```typescript
interface PriceAlert {
  id: string;
  userId: UserId;

  // Alert configuration
  itemId: ItemId;
  realm: string;
  alertType: AlertType;
  triggerPrice: GoldAmount;

  // Notification settings
  notificationMethods: NotificationMethod[];
  cooldownPeriod: number; // Minutes between notifications

  // Status
  status: 'ACTIVE' | 'PAUSED' | 'TRIGGERED' | 'EXPIRED';
  isActive: boolean;

  // Timing
  createdAt: Timestamp;
  lastTriggered?: Timestamp;
  expiresAt?: Timestamp;

  // Trigger history
  triggerHistory: AlertTrigger[];

  // Additional settings
  filters?: AlertFilters;
}
```

### NotificationMethod

How alerts are delivered to users.

```typescript
interface NotificationMethod {
  type: 'WEBHOOK' | 'EMAIL' | 'DISCORD' | 'PUSH' | 'SMS';
  destination: string; // URL, email, webhook URL, etc.
  template?: string; // Custom notification template
}
```

### AlertTrigger

Record of when an alert was triggered.

```typescript
interface AlertTrigger {
  timestamp: Timestamp;
  triggerPrice: GoldAmount;
  marketPrice: GoldAmount;
  quantity: number;
  notificationSent: boolean;
}
```

### AlertFilters

Additional filtering options for alerts.

```typescript
interface AlertFilters {
  minQuantity?: number;
  maxQuantity?: number;
  sellerBlacklist?: string[];
  timeLeft?: TimeLeft[];
  quality?: ItemQuality[];
}
```

## User and Account Models

### User

User account information.

```typescript
interface User {
  id: UserId;
  username: string;
  email: string;

  // Account settings
  preferences: UserPreferences;
  subscriptions: SubscriptionInfo[];

  // API access
  apiKey: string;
  rateLimit: RateLimitInfo;

  // Account status
  status: 'ACTIVE' | 'SUSPENDED' | 'EXPIRED';
  createdAt: Timestamp;
  lastLogin?: Timestamp;
}
```

### UserPreferences

User-configurable preferences.

```typescript
interface UserPreferences {
  // Notification preferences
  notifications: {
    email: boolean;
    push: boolean;
    webhook: boolean;
  };

  // Display preferences
  display: {
    currency: 'GOLD' | 'COPPER' | 'SILVER';
    timezone: string;
    dateFormat: string;
    numberFormat: 'COMMA' | 'DOT';
  };

  // Analysis preferences
  analysis: {
    defaultTimeframe: number; // days
    confidenceThreshold: number; // 0-1
    riskTolerance: 'LOW' | 'MEDIUM' | 'HIGH';
  };

  // Privacy settings
  privacy: {
    showInLeaderboards: boolean;
    shareDataAnonymously: boolean;
    allowDataCollection: boolean;
  };
}
```

### SubscriptionInfo

User subscription details.

```typescript
interface SubscriptionInfo {
  id: string;
  plan: 'FREE' | 'BASIC' | 'PREMIUM' | 'ENTERPRISE';
  status: 'ACTIVE' | 'CANCELLED' | 'EXPIRED';

  // Limits and quotas
  limits: {
    apiCallsPerHour: number;
    alerts: number;
    watchlistItems: number;
    dataRetentionDays: number;
  };

  // Billing
  billing: {
    interval: 'MONTHLY' | 'YEARLY';
    amount: number;
    currency: string;
    nextBillingDate: Timestamp;
  };

  // Features
  features: string[]; // List of enabled features
}
```

### RateLimitInfo

API rate limiting information.

```typescript
interface RateLimitInfo {
  requests: {
    used: number;
    limit: number;
    resetTime: Timestamp;
  };

  concurrency: {
    current: number;
    max: number;
  };
}
```

## Realm and Region Models

### Realm

World of Warcraft realm information.

```typescript
interface Realm {
  id: RealmId;
  name: string;
  slug: string; // URL-friendly name

  // Location
  region: 'US' | 'EU' | 'KR' | 'TW' | 'CN';
  timezone: string;
  locale: string;

  // Realm characteristics
  type: 'NORMAL' | 'PVP' | 'RP' | 'PVP_RP';
  population: 'LOW' | 'MEDIUM' | 'HIGH' | 'FULL';

  // Connected realms
  connectedRealms: RealmId[];

  // Status
  status: 'UP' | 'DOWN' | 'MAINTENANCE';
  lastUpdated: Timestamp;

  // Auction house info
  auctionHouse: {
    lastScan: Timestamp;
    totalListings: number;
    uniqueItems: number;
  };
}
```

### Region

Regional grouping of realms.

```typescript
interface Region {
  id: string;
  name: string;
  code: 'US' | 'EU' | 'KR' | 'TW' | 'CN';

  // Region statistics
  totalRealms: number;
  totalPopulation: number;

  // Economic indicators
  economy: {
    averagePrices: Map<ItemId, GoldAmount>;
    tradeVolume: GoldAmount;
    inflationRate: number;
  };

  // Connectivity
  connectedRegions: string[]; // For cross-region trading
}
```

## API Response Models

### APIResponse

Standard API response wrapper.

```typescript
interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: APIError;
  meta: ResponseMeta;
}
```

### APIError

Error response structure.

```typescript
interface APIError {
  code: string;
  message: string;
  details?: any;
  stack?: string; // In development mode
}
```

### ResponseMeta

Response metadata.

```typescript
interface ResponseMeta {
  timestamp: Timestamp;
  requestId: string;
  version: string;
  processingTime: number; // milliseconds
  rateLimit?: RateLimitInfo;
}
```

### PaginatedResponse

Paginated API response.

```typescript
interface PaginatedResponse<T> extends APIResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}
```

## WebSocket Models

### WebSocketMessage

Base WebSocket message structure.

```typescript
interface WebSocketMessage {
  type: string;
  timestamp: Timestamp;
  data: any;
}
```

### AuctionUpdateMessage

Real-time auction update message.

```typescript
interface AuctionUpdateMessage extends WebSocketMessage {
  type: 'AUCTION_UPDATE';
  data: {
    eventType: 'NEW' | 'UPDATE' | 'DELETE' | 'SOLD';
    auction: AuctionItem;
    changes?: Partial<AuctionItem>;
  };
}
```

### MarketAlertMessage

Real-time market alert message.

```typescript
interface MarketAlertMessage extends WebSocketMessage {
  type: 'MARKET_ALERT';
  data: {
    alertType: 'OPPORTUNITY' | 'TREND_CHANGE' | 'VOLATILITY';
    itemId: ItemId;
    message: string;
    severity: 'INFO' | 'WARNING' | 'CRITICAL';
  };
}
```

## Configuration Models

### AppConfig

Application configuration.

```typescript
interface AppConfig {
  // API settings
  api: {
    port: number;
    host: string;
    cors: {
      origins: string[];
      credentials: boolean;
    };
  };

  // Database settings
  database: {
    url: string;
    name: string;
    options: any;
  };

  // Cache settings
  cache: {
    redis: {
      host: string;
      port: number;
      password?: string;
    };
    ttl: {
      prices: number;
      analysis: number;
      trends: number;
    };
  };

  // External services
  services: {
    blizzard: {
      clientId: string;
      clientSecret: string;
      region: string;
    };
    discord?: {
      webhookUrl: string;
      botToken?: string;
    };
  };

  // Feature flags
  features: {
    realTimeUpdates: boolean;
    advancedAnalytics: boolean;
    crossRealmTrading: boolean;
    priceAlerts: boolean;
  };

  // Rate limiting
  rateLimit: {
    windowMs: number;
    maxRequests: number;
    skipSuccessfulRequests: boolean;
  };
}
```

## Analytics Models

### AnalyticsEvent

User interaction and system event tracking.

```typescript
interface AnalyticsEvent {
  id: string;
  userId?: UserId;
  sessionId: string;

  // Event details
  type: string;
  category: 'USER_ACTION' | 'SYSTEM_EVENT' | 'ERROR' | 'PERFORMANCE';
  action: string;

  // Context
  timestamp: Timestamp;
  userAgent?: string;
  ip?: string;
  referrer?: string;

  // Event data
  data: Record<string, any>;

  // Performance metrics
  performance?: {
    loadTime: number;
    renderTime: number;
    apiCalls: number;
  };
}
```

### SystemMetrics

System performance and health metrics.

```typescript
interface SystemMetrics {
  timestamp: Timestamp;

  // Performance
  responseTime: {
    average: number;
    p50: number;
    p95: number;
    p99: number;
  };

  // Throughput
  throughput: {
    requestsPerSecond: number;
    auctionsProcessed: number;
    dataPointsAnalyzed: number;
  };

  // Resource usage
  resources: {
    cpu: number; // percentage
    memory: number; // MB
    disk: number; // percentage
    network: {
      inbound: number; // bytes/sec
      outbound: number; // bytes/sec
    };
  };

  // Error rates
  errors: {
    total: number;
    rate: number; // per minute
    byType: Record<string, number>;
  };

  // Cache performance
  cache: {
    hitRate: number;
    size: number;
    evictions: number;
  };
}
```