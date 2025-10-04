# WoW Auction Tracker - Core Utilities

## Overview

This document details the core utility functions and data processing capabilities of the WoW Auction Tracker system. These utilities provide essential functionality for data manipulation, price calculations, and market analysis.

## Price Calculation Utilities

### Market Price Calculations

#### `calculateAveragePrice(auctions)`

Calculates the arithmetic mean price from auction data.

```typescript
function calculateAveragePrice(auctions: AuctionItem[]): number
```

**Parameters:**
- `auctions`: Array of auction items

**Returns:** Average price as number

**Example:**
```typescript
const auctions = [
  { buyoutPrice: 100000 },
  { buyoutPrice: 150000 },
  { buyoutPrice: 200000 }
];

const average = calculateAveragePrice(auctions);
// Returns: 150000
```

#### `calculateMedianPrice(auctions)`

Calculates the median price from auction data.

```typescript
function calculateMedianPrice(auctions: AuctionItem[]): number
```

**Parameters:**
- `auctions`: Array of auction items

**Returns:** Median price as number

**Example:**
```typescript
const auctions = [
  { buyoutPrice: 100000 },
  { buyoutPrice: 150000 },
  { buyoutPrice: 200000 },
  { buyoutPrice: 300000 }
];

const median = calculateMedianPrice(auctions);
// Returns: 200000 (middle value when sorted)
```

#### `calculateWeightedPrice(auctions, weightBy)`

Calculates weighted average price based on quantity.

```typescript
function calculateWeightedPrice(
  auctions: AuctionItem[],
  weightBy: 'quantity' | 'time' = 'quantity'
): number
```

**Parameters:**
- `auctions`: Array of auction items
- `weightBy`: Weighting method

**Example:**
```typescript
const auctions = [
  { buyoutPrice: 100000, quantity: 1 },
  { buyoutPrice: 150000, quantity: 5 },
  { buyoutPrice: 200000, quantity: 2 }
];

const weighted = calculateWeightedPrice(auctions, 'quantity');
// Returns: ~155,000 (weighted by quantity)
```

#### `calculateTrimmedMean(auctions, trimPercentage)`

Calculates trimmed mean by removing outliers.

```typescript
function calculateTrimmedMean(
  auctions: AuctionItem[],
  trimPercentage: number = 0.1
): number
```

**Parameters:**
- `auctions`: Array of auction items
- `trimPercentage`: Percentage to trim from each end (0.1 = 10%)

**Example:**
```typescript
const auctions = [
  { buyoutPrice: 50000 },   // Outlier low
  { buyoutPrice: 150000 },
  { buyoutPrice: 155000 },
  { buyoutPrice: 160000 },
  { buyoutPrice: 165000 },
  { buyoutPrice: 300000 }   // Outlier high
];

const trimmed = calculateTrimmedMean(auctions, 0.15);
// Removes 15% lowest and highest, calculates mean of remainder
```

### Price Range Calculations

#### `calculatePriceRange(auctions)`

Calculates min, max, and standard deviation of prices.

```typescript
function calculatePriceRange(auctions: AuctionItem[]): {
  min: number;
  max: number;
  range: number;
  stdDev: number;
}
```

**Returns:** Object with price statistics

**Example:**
```typescript
const stats = calculatePriceRange(auctions);
console.log(`Price range: ${stats.min} - ${stats.max}`);
console.log(`Standard deviation: ${stats.stdDev}`);
```

#### `detectOutliers(auctions, threshold)`

Identifies price outliers using statistical methods.

```typescript
function detectOutliers(
  auctions: AuctionItem[],
  threshold: number = 2
): { outliers: AuctionItem[], clean: AuctionItem[] }
```

**Parameters:**
- `threshold`: Standard deviation threshold (default: 2)

**Example:**
```typescript
const result = detectOutliers(auctions, 2.5);
console.log(`${result.outliers.length} outliers detected`);
```

## Data Processing Utilities

### Auction Data Filters

#### `filterByPriceRange(auctions, minPrice, maxPrice)`

Filters auctions by price range.

```typescript
function filterByPriceRange(
  auctions: AuctionItem[],
  minPrice?: number,
  maxPrice?: number
): AuctionItem[]
```

**Example:**
```typescript
const affordable = filterByPriceRange(auctions, 100000, 500000);
const expensive = filterByPriceRange(auctions, undefined, 1000000);
```

#### `filterBySeller(auctions, sellerName)`

Filters auctions by seller name.

```typescript
function filterBySeller(
  auctions: AuctionItem[],
  sellerName: string
): AuctionItem[]
```

**Example:**
```typescript
const competitorAuctions = filterBySeller(auctions, 'UndercutMaster');
```

#### `filterByTimeLeft(auctions, timeLeft)`

Filters auctions by time remaining.

```typescript
function filterByTimeLeft(
  auctions: AuctionItem[],
  timeLeft: 'SHORT' | 'MEDIUM' | 'LONG' | 'VERY_LONG'
): AuctionItem[]
```

**Example:**
```typescript
const endingSoon = filterByTimeLeft(auctions, 'SHORT');
const longTerm = filterByTimeLeft(auctions, 'VERY_LONG');
```

#### `filterByRealm(auctions, realm)`

Filters auctions by realm.

```typescript
function filterByRealm(
  auctions: AuctionItem[],
  realm: string
): AuctionItem[]
```

**Example:**
```typescript
const stormrageAuctions = filterByRealm(auctions, 'Stormrage');
```

### Data Aggregation

#### `groupByItem(auctions)`

Groups auctions by item ID.

```typescript
function groupByItem(auctions: AuctionItem[]): Map<number, AuctionItem[]>
```

**Returns:** Map with item ID as key and auctions array as value

**Example:**
```typescript
const grouped = groupByItem(auctions);
for (const [itemId, itemAuctions] of grouped) {
  console.log(`Item ${itemId}: ${itemAuctions.length} auctions`);
}
```

#### `aggregateItemStats(auctions)`

Aggregates statistics for each item.

```typescript
function aggregateItemStats(auctions: AuctionItem[]): Map<number, ItemStats>

interface ItemStats {
  itemId: number;
  totalQuantity: number;
  uniqueSellers: number;
  priceRange: { min: number; max: number };
  avgPrice: number;
  lastSeen: Date;
}
```

**Example:**
```typescript
const stats = aggregateItemStats(auctions);
for (const [itemId, itemStats] of stats) {
  console.log(`Item ${itemId} stats:`, itemStats);
}
```

## Market Analysis Utilities

### Trend Analysis

#### `calculateTrend(prices, periods)`

Calculates price trend over time periods.

```typescript
function calculateTrend(
  prices: number[],
  periods: number = 7
): { direction: 'UP' | 'DOWN' | 'STABLE', strength: number, slope: number }
```

**Parameters:**
- `prices`: Array of historical prices
- `periods`: Number of periods to analyze

**Example:**
```typescript
const prices = [100, 102, 105, 108, 110, 112, 115];
const trend = calculateTrend(prices, 7);
// Returns: { direction: 'UP', strength: 0.85, slope: 2.5 }
```

#### `detectSeasonalPatterns(prices, cycleLength)`

Detects seasonal or cyclical patterns in price data.

```typescript
function detectSeasonalPatterns(
  prices: number[],
  cycleLength: number = 7
): { hasPattern: boolean, strength: number, peaks: number[] }
```

**Example:**
```typescript
const dailyPrices = [/* 30 days of data */];
const patterns = detectSeasonalPatterns(dailyPrices, 7);
// Returns seasonal analysis
```

### Volatility Analysis

#### `calculateVolatility(prices, periods)`

Calculates price volatility metrics.

```typescript
function calculateVolatility(
  prices: number[],
  periods: number = 30
): { volatility: number, beta: number, sharpe: number }
```

**Example:**
```typescript
const prices = [/* historical prices */];
const vol = calculateVolatility(prices, 30);
// Returns volatility metrics
```

#### `isVolatileItem(itemId, threshold)`

Determines if an item shows volatile price behavior.

```typescript
function isVolatileItem(
  itemId: number,
  threshold: number = 0.3
): boolean
```

**Example:**
```typescript
if (isVolatileItem(12345, 0.25)) {
  console.log('This item has volatile pricing');
}
```

## Data Validation Utilities

### Input Validation

#### `validateAuctionData(auction)`

Validates auction data structure and values.

```typescript
function validateAuctionData(auction: any): { valid: boolean, errors: string[] }
```

**Example:**
```typescript
const auction = { /* auction data */ };
const validation = validateAuctionData(auction);

if (!validation.valid) {
  console.log('Validation errors:', validation.errors);
}
```

#### `sanitizePrice(price)`

Sanitizes and validates price values.

```typescript
function sanitizePrice(price: any): number | null
```

**Example:**
```typescript
const cleanPrice = sanitizePrice('150,000g'); // Returns: 150000
const invalidPrice = sanitizePrice('invalid'); // Returns: null
```

### Data Cleaning

#### `removeDuplicates(auctions)`

Removes duplicate auction entries.

```typescript
function removeDuplicates(auctions: AuctionItem[]): AuctionItem[]
```

**Example:**
```typescript
const cleanAuctions = removeDuplicates(rawAuctions);
console.log(`Removed ${rawAuctions.length - cleanAuctions.length} duplicates`);
```

#### `normalizeAuctionData(auctions)`

Normalizes auction data to consistent format.

```typescript
function normalizeAuctionData(auctions: any[]): AuctionItem[]
```

**Example:**
```typescript
const normalized = normalizeAuctionData(rawData);
// Converts various input formats to standard AuctionItem[]
```

## Performance Utilities

### Caching Functions

#### `createPriceCache(ttl)`

Creates a time-based cache for price data.

```typescript
function createPriceCache(ttl: number = 300000): Map<string, { data: any, expiry: number }>
```

**Example:**
```typescript
const cache = createPriceCache(60000); // 1 minute TTL
cache.set('item_12345', { data: priceData, expiry: Date.now() + 60000 });
```

#### `getCachedPrice(cache, key)`

Retrieves cached price data if still valid.

```typescript
function getCachedPrice(
  cache: Map<string, any>,
  key: string
): any | null
```

**Example:**
```typescript
const cached = getCachedPrice(cache, 'item_12345');
if (cached) {
  console.log('Using cached data:', cached);
}
```

### Batch Processing

#### `processBatch(items, processor, batchSize)`

Processes large datasets in batches.

```typescript
function processBatch<T, R>(
  items: T[],
  processor: (batch: T[]) => R[],
  batchSize: number = 100
): R[]
```

**Example:**
```typescript
const auctions = [/* large array */];
const processed = processBatch(auctions, calculatePrices, 50);
```

## Configuration Utilities

### Settings Management

#### `loadConfig(path)`

Loads configuration from file.

```typescript
function loadConfig(path: string): AppConfig
```

#### `validateConfig(config)`

Validates configuration object.

```typescript
function validateConfig(config: any): { valid: boolean, errors: string[] }
```

#### `mergeConfig(defaults, overrides)`

Merges default and override configurations.

```typescript
function mergeConfig(defaults: any, overrides: any): any
```

## Export and Import Utilities

### Data Export

#### `exportToCSV(data, filename)`

Exports data to CSV format.

```typescript
function exportToCSV(data: any[], filename: string): void
```

**Example:**
```typescript
exportToCSV(auctions, 'auctions_export.csv');
```

#### `exportToJSON(data, filename, pretty)`

Exports data to JSON format.

```typescript
function exportToJSON(data: any, filename: string, pretty: boolean = true): void
```

**Example:**
```typescript
exportToJSON(analysis, 'market_analysis.json', true);
```

### Data Import

#### `importFromCSV(file)`

Imports auction data from CSV file.

```typescript
function importFromCSV(file: File): Promise<AuctionItem[]>
```

#### `importFromJSON(file)`

Imports data from JSON file.

```typescript
function importFromJSON(file: File): Promise<any>
```

## Error Handling Utilities

### Error Classification

#### `classifyError(error)`

Classifies errors by type and severity.

```typescript
function classifyError(error: Error): { type: string, severity: 'LOW' | 'MEDIUM' | 'HIGH' }
```

#### `createErrorReport(error, context)`

Creates detailed error reports.

```typescript
function createErrorReport(error: Error, context: any): ErrorReport
```

### Retry Logic

#### `withRetry(operation, options)`

Wraps operations with retry logic.

```typescript
function withRetry<T>(
  operation: () => Promise<T>,
  options: { maxRetries: number, delay: number }
): Promise<T>
```

**Example:**
```typescript
const result = await withRetry(
  () => apiCall(),
  { maxRetries: 3, delay: 1000 }
);
```

## Monitoring and Logging

### Performance Monitoring

#### `trackPerformance(operation)`

Tracks performance metrics for operations.

```typescript
function trackPerformance<T>(operation: () => T): { result: T, duration: number }
```

**Example:**
```typescript
const { result, duration } = trackPerformance(() => processAuctions());
console.log(`Processing took ${duration}ms`);
```

#### `createMetricsCollector()`

Creates a metrics collection system.

```typescript
function createMetricsCollector(): {
  record: (metric: string, value: number) => void;
  getStats: () => any;
}
```

### Logging Utilities

#### `createLogger(level)`

Creates a configurable logger.

```typescript
function createLogger(level: 'DEBUG' | 'INFO' | 'WARN' | 'ERROR' = 'INFO')
```

**Example:**
```typescript
const logger = createLogger('DEBUG');
logger.info('Application started');
logger.error('Failed to fetch auctions', error);
```