# WoW Auction Tracker - Component Documentation

## Table of Contents

1. [Frontend Components](#frontend-components)
2. [Backend Components](#backend-components)
3. [Data Models](#data-models)
4. [Utility Functions](#utility-functions)
5. [Configuration](#configuration)

## Frontend Components

### AuctionList Component

A React component for displaying auction listings with filtering and sorting capabilities.

#### Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `realm` | string | Yes | - | The realm to fetch auctions from |
| `itemId` | number | No | - | Filter by specific item ID |
| `pageSize` | number | No | 50 | Number of auctions per page |
| `sortBy` | string | No | 'price' | Sort field (price, time_left, item_name) |
| `sortOrder` | string | No | 'asc' | Sort order (asc, desc) |
| `onAuctionClick` | function | No | - | Callback when auction is clicked |

#### Usage Example

```jsx
import { AuctionList } from './components/AuctionList';

function App() {
  const handleAuctionClick = (auction) => {
    console.log('Selected auction:', auction);
  };

  return (
    <AuctionList
      realm="stormrage"
      itemId={19019}
      pageSize={25}
      sortBy="price"
      sortOrder="asc"
      onAuctionClick={handleAuctionClick}
    />
  );
}
```

#### Component Structure

```jsx
const AuctionList = ({ realm, itemId, pageSize, sortBy, sortOrder, onAuctionClick }) => {
  const [auctions, setAuctions] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [currentPage, setCurrentPage] = useState(1);

  // Fetch auctions effect
  useEffect(() => {
    fetchAuctions();
  }, [realm, itemId, currentPage, sortBy, sortOrder]);

  const fetchAuctions = async () => {
    setLoading(true);
    try {
      const response = await api.getAuctions(realm, {
        item_id: itemId,
        limit: pageSize,
        offset: (currentPage - 1) * pageSize,
        sort_by: sortBy,
        sort_order: sortOrder
      });
      setAuctions(response.data.auctions);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;

  return (
    <div className="auction-list">
      <AuctionFilters onFilterChange={handleFilterChange} />
      <AuctionTable 
        auctions={auctions} 
        onAuctionClick={onAuctionClick}
        sortBy={sortBy}
        sortOrder={sortOrder}
        onSort={handleSort}
      />
      <Pagination 
        currentPage={currentPage}
        totalPages={Math.ceil(auctions.length / pageSize)}
        onPageChange={setCurrentPage}
      />
    </div>
  );
};
```

### PriceChart Component

A React component for displaying price history charts using Chart.js.

#### Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `realm` | string | Yes | - | The realm name |
| `itemId` | number | Yes | - | The item ID |
| `period` | string | No | '7d' | Time period (1d, 7d, 30d, 90d) |
| `chartType` | string | No | 'line' | Chart type (line, candlestick) |
| `height` | number | No | 400 | Chart height in pixels |

#### Usage Example

```jsx
import { PriceChart } from './components/PriceChart';

function ItemDetails({ itemId }) {
  return (
    <div>
      <h2>Price History</h2>
      <PriceChart
        realm="stormrage"
        itemId={itemId}
        period="30d"
        chartType="line"
        height={300}
      />
    </div>
  );
}
```

### ItemSearch Component

A React component for searching and filtering items.

#### Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `onItemSelect` | function | Yes | - | Callback when item is selected |
| `placeholder` | string | No | 'Search items...' | Search input placeholder |
| `filters` | object | No | {} | Additional filters |
| `maxResults` | number | No | 10 | Maximum search results |

#### Usage Example

```jsx
import { ItemSearch } from './components/ItemSearch';

function SearchPage() {
  const handleItemSelect = (item) => {
    navigate(`/item/${item.id}`);
  };

  return (
    <ItemSearch
      onItemSelect={handleItemSelect}
      placeholder="Find your item..."
      filters={{ quality: 'epic', min_level: 60 }}
      maxResults={20}
    />
  );
}
```

### WatchlistManager Component

A React component for managing user watchlists.

#### Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `userId` | number | Yes | - | User ID |
| `onWatchlistChange` | function | No | - | Callback when watchlist changes |

#### Usage Example

```jsx
import { WatchlistManager } from './components/WatchlistManager';

function UserDashboard({ user }) {
  const handleWatchlistChange = (watchlist) => {
    console.log('Watchlist updated:', watchlist);
  };

  return (
    <div>
      <h2>My Watchlist</h2>
      <WatchlistManager
        userId={user.id}
        onWatchlistChange={handleWatchlistChange}
      />
    </div>
  );
}
```

### NotificationCenter Component

A React component for displaying and managing notifications.

#### Props

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `userId` | number | Yes | - | User ID |
| `autoRefresh` | boolean | No | true | Auto-refresh notifications |
| `refreshInterval` | number | No | 30000 | Refresh interval in ms |

#### Usage Example

```jsx
import { NotificationCenter } from './components/NotificationCenter';

function Header({ user }) {
  return (
    <header>
      <nav>...</nav>
      <NotificationCenter
        userId={user.id}
        autoRefresh={true}
        refreshInterval={60000}
      />
    </header>
  );
}
```

## Backend Components

### AuctionService Class

Handles auction data fetching and processing.

#### Methods

##### `fetchAuctions(realm, options)`

Fetches auction data from Blizzard API.

**Parameters:**
- `realm` (string): Realm name or ID
- `options` (object): Query options

**Returns:** Promise<AuctionData[]>

**Example:**
```javascript
const auctionService = new AuctionService();

const auctions = await auctionService.fetchAuctions('stormrage', {
  item_id: 19019,
  limit: 100
});
```

##### `processAuctionData(rawData)`

Processes raw auction data from Blizzard API.

**Parameters:**
- `rawData` (object): Raw auction data from Blizzard API

**Returns:** ProcessedAuctionData[]

**Example:**
```javascript
const processedData = auctionService.processAuctionData(rawBlizzardData);
```

##### `calculateStatistics(auctions)`

Calculates statistical data for auctions.

**Parameters:**
- `auctions` (AuctionData[]): Array of auction data

**Returns:** AuctionStatistics

**Example:**
```javascript
const stats = auctionService.calculateStatistics(auctions);
console.log(`Average price: ${stats.averagePrice}`);
```

### PriceHistoryService Class

Manages price history data and calculations.

#### Methods

##### `recordPrices(realm, auctions)`

Records current auction prices to history.

**Parameters:**
- `realm` (string): Realm name
- `auctions` (AuctionData[]): Current auction data

**Returns:** Promise<void>

##### `getPriceHistory(realm, itemId, period)`

Retrieves price history for an item.

**Parameters:**
- `realm` (string): Realm name
- `itemId` (number): Item ID
- `period` (string): Time period

**Returns:** Promise<PriceHistoryData[]>

**Example:**
```javascript
const priceService = new PriceHistoryService();

const history = await priceService.getPriceHistory('stormrage', 19019, '7d');
```

##### `calculateTrends(priceHistory)`

Calculates price trends and volatility.

**Parameters:**
- `priceHistory` (PriceHistoryData[]): Historical price data

**Returns:** TrendData

### NotificationService Class

Handles user notifications and alerts.

#### Methods

##### `createNotification(userId, type, data)`

Creates a new notification for a user.

**Parameters:**
- `userId` (number): User ID
- `type` (string): Notification type
- `data` (object): Notification data

**Returns:** Promise<Notification>

##### `sendPriceAlert(userId, item, currentPrice, targetPrice)`

Sends a price alert notification.

**Parameters:**
- `userId` (number): User ID
- `item` (ItemData): Item information
- `currentPrice` (number): Current price
- `targetPrice` (number): Target price

**Returns:** Promise<void>

**Example:**
```javascript
const notificationService = new NotificationService();

await notificationService.sendPriceAlert(
  123,
  { id: 19019, name: 'Thunderfury' },
  900000,
  950000
);
```

### CacheManager Class

Manages caching for improved performance.

#### Methods

##### `get(key)`

Retrieves data from cache.

**Parameters:**
- `key` (string): Cache key

**Returns:** Promise<any>

##### `set(key, data, ttl)`

Stores data in cache.

**Parameters:**
- `key` (string): Cache key
- `data` (any): Data to cache
- `ttl` (number): Time to live in seconds

**Returns:** Promise<void>

##### `invalidate(pattern)`

Invalidates cache entries matching pattern.

**Parameters:**
- `pattern` (string): Cache key pattern

**Returns:** Promise<void>

**Example:**
```javascript
const cache = new CacheManager();

// Cache auction data for 5 minutes
await cache.set('auctions:stormrage', auctionData, 300);

// Retrieve cached data
const cachedAuctions = await cache.get('auctions:stormrage');

// Invalidate all auction cache
await cache.invalidate('auctions:*');
```

## Data Models

### Auction Model

Represents an auction listing.

```javascript
class Auction {
  constructor(data) {
    this.id = data.id;
    this.itemId = data.item_id;
    this.itemName = data.item_name;
    this.quality = data.quality;
    this.quantity = data.quantity;
    this.unitPrice = data.unit_price;
    this.totalPrice = data.total_price;
    this.seller = data.seller;
    this.realm = data.realm;
    this.timeLeft = data.time_left;
    this.buyout = data.buyout;
    this.bid = data.bid;
    this.createdAt = new Date(data.created_at);
  }

  // Calculate price per unit
  getPricePerUnit() {
    return this.totalPrice / this.quantity;
  }

  // Check if auction is ending soon
  isEndingSoon() {
    return this.timeLeft === 'SHORT';
  }

  // Format price for display
  formatPrice() {
    return formatCurrency(this.totalPrice);
  }
}
```

### Item Model

Represents a WoW item.

```javascript
class Item {
  constructor(data) {
    this.id = data.id;
    this.name = data.name;
    this.quality = data.quality;
    this.itemLevel = data.item_level;
    this.requiredLevel = data.required_level;
    this.itemClass = data.item_class;
    this.itemSubclass = data.item_subclass;
    this.icon = data.icon;
    this.description = data.description;
    this.bindType = data.bind_type;
    this.unique = data.unique;
    this.maxStack = data.max_stack;
    this.sellPrice = data.sell_price;
    this.stats = data.stats;
  }

  // Get item icon URL
  getIconUrl(size = 'medium') {
    const sizes = { small: 32, medium: 64, large: 128 };
    return `https://render-us.worldofwarcraft.com/icons/${sizes[size]}/${this.icon}.jpg`;
  }

  // Check if item is tradeable
  isTradeable() {
    return this.bindType !== 'bind_on_pickup';
  }

  // Get quality color
  getQualityColor() {
    const colors = {
      poor: '#9d9d9d',
      common: '#ffffff',
      uncommon: '#1eff00',
      rare: '#0070dd',
      epic: '#a335ee',
      legendary: '#ff8000'
    };
    return colors[this.quality] || colors.common;
  }
}
```

### User Model

Represents a user account.

```javascript
class User {
  constructor(data) {
    this.id = data.id;
    this.username = data.username;
    this.email = data.email;
    this.realm = data.realm;
    this.characterName = data.character_name;
    this.createdAt = new Date(data.created_at);
    this.apiKey = data.api_key;
    this.watchlist = data.watchlist || [];
    this.notifications = data.notifications || [];
  }

  // Add item to watchlist
  addToWatchlist(itemId, targetPrice) {
    const watchlistItem = {
      itemId,
      targetPrice,
      createdAt: new Date()
    };
    this.watchlist.push(watchlistItem);
  }

  // Remove item from watchlist
  removeFromWatchlist(itemId) {
    this.watchlist = this.watchlist.filter(item => item.itemId !== itemId);
  }

  // Check if item is in watchlist
  isWatching(itemId) {
    return this.watchlist.some(item => item.itemId === itemId);
  }
}
```

## Utility Functions

### Currency Formatting

```javascript
/**
 * Formats copper amount to gold/silver/copper display
 * @param {number} copper - Amount in copper
 * @returns {string} Formatted currency string
 */
function formatCurrency(copper) {
  const gold = Math.floor(copper / 10000);
  const silver = Math.floor((copper % 10000) / 100);
  const remainingCopper = copper % 100;

  let result = '';
  if (gold > 0) result += `${gold}g `;
  if (silver > 0) result += `${silver}s `;
  if (remainingCopper > 0 || result === '') result += `${remainingCopper}c`;

  return result.trim();
}

// Usage
console.log(formatCurrency(123456)); // "12g 34s 56c"
console.log(formatCurrency(5678)); // "56s 78c"
console.log(formatCurrency(42)); // "42c"
```

### Time Formatting

```javascript
/**
 * Formats time left string to human readable format
 * @param {string} timeLeft - Time left from Blizzard API
 * @returns {string} Human readable time
 */
function formatTimeLeft(timeLeft) {
  const timeMap = {
    'VERY_LONG': 'More than 12 hours',
    'LONG': '2-12 hours',
    'MEDIUM': '30 minutes - 2 hours',
    'SHORT': 'Less than 30 minutes'
  };
  return timeMap[timeLeft] || 'Unknown';
}

/**
 * Formats timestamp to relative time
 * @param {Date} date - Date to format
 * @returns {string} Relative time string
 */
function formatRelativeTime(date) {
  const now = new Date();
  const diffMs = now - date;
  const diffMins = Math.floor(diffMs / 60000);
  const diffHours = Math.floor(diffMins / 60);
  const diffDays = Math.floor(diffHours / 24);

  if (diffMins < 1) return 'Just now';
  if (diffMins < 60) return `${diffMins} minutes ago`;
  if (diffHours < 24) return `${diffHours} hours ago`;
  return `${diffDays} days ago`;
}
```

### Data Validation

```javascript
/**
 * Validates realm name
 * @param {string} realm - Realm name to validate
 * @returns {boolean} True if valid
 */
function isValidRealm(realm) {
  return typeof realm === 'string' && 
         realm.length > 0 && 
         /^[a-zA-Z0-9-']+$/.test(realm);
}

/**
 * Validates item ID
 * @param {number} itemId - Item ID to validate
 * @returns {boolean} True if valid
 */
function isValidItemId(itemId) {
  return Number.isInteger(itemId) && itemId > 0;
}

/**
 * Validates price amount
 * @param {number} price - Price to validate
 * @returns {boolean} True if valid
 */
function isValidPrice(price) {
  return Number.isInteger(price) && price >= 0;
}

/**
 * Sanitizes user input
 * @param {string} input - User input to sanitize
 * @returns {string} Sanitized input
 */
function sanitizeInput(input) {
  return input.toString()
              .trim()
              .replace(/[<>]/g, '')
              .substring(0, 255);
}
```

### API Helpers

```javascript
/**
 * Makes authenticated API request
 * @param {string} endpoint - API endpoint
 * @param {object} options - Request options
 * @returns {Promise<object>} API response
 */
async function apiRequest(endpoint, options = {}) {
  const config = {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${getApiKey()}`,
      'Content-Type': 'application/json',
      ...options.headers
    },
    ...options
  };

  const response = await fetch(`${API_BASE_URL}${endpoint}`, config);
  
  if (!response.ok) {
    throw new Error(`API Error: ${response.status} ${response.statusText}`);
  }

  return response.json();
}

/**
 * Handles API errors consistently
 * @param {Error} error - Error to handle
 * @returns {object} Formatted error response
 */
function handleApiError(error) {
  console.error('API Error:', error);
  
  return {
    success: false,
    error: {
      message: error.message || 'An unexpected error occurred',
      code: error.code || 'UNKNOWN_ERROR'
    }
  };
}

/**
 * Builds query string from parameters
 * @param {object} params - Query parameters
 * @returns {string} Query string
 */
function buildQueryString(params) {
  const searchParams = new URLSearchParams();
  
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null) {
      searchParams.append(key, value.toString());
    }
  });
  
  return searchParams.toString();
}
```

## Configuration

### Environment Configuration

```javascript
// config/environment.js
const config = {
  development: {
    API_BASE_URL: 'http://localhost:3000/api/v1',
    BLIZZARD_API_URL: 'https://us.api.blizzard.com',
    CACHE_TTL: 300, // 5 minutes
    RATE_LIMIT: 100,
    LOG_LEVEL: 'debug'
  },
  
  production: {
    API_BASE_URL: 'https://api.wow-auction-tracker.com/v1',
    BLIZZARD_API_URL: 'https://us.api.blizzard.com',
    CACHE_TTL: 600, // 10 minutes
    RATE_LIMIT: 1000,
    LOG_LEVEL: 'error'
  },
  
  test: {
    API_BASE_URL: 'http://localhost:3001/api/v1',
    BLIZZARD_API_URL: 'https://us.api.blizzard.com',
    CACHE_TTL: 60, // 1 minute
    RATE_LIMIT: 1000,
    LOG_LEVEL: 'silent'
  }
};

module.exports = config[process.env.NODE_ENV || 'development'];
```

### Database Configuration

```javascript
// config/database.js
module.exports = {
  development: {
    host: 'localhost',
    port: 5432,
    database: 'wow_auction_tracker_dev',
    username: 'postgres',
    password: 'password',
    dialect: 'postgres',
    logging: console.log
  },
  
  production: {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT || 5432,
    database: process.env.DB_NAME,
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    dialect: 'postgres',
    logging: false,
    pool: {
      max: 20,
      min: 5,
      acquire: 30000,
      idle: 10000
    }
  }
};
```

### Redis Configuration

```javascript
// config/redis.js
module.exports = {
  development: {
    host: 'localhost',
    port: 6379,
    db: 0
  },
  
  production: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD,
    db: 0,
    retryDelayOnFailover: 100,
    maxRetriesPerRequest: 3
  }
};
```

This comprehensive component documentation covers all the major components, models, utilities, and configuration needed for a WoW auction tracker application. Each component includes detailed props, usage examples, and implementation details to help developers understand and use them effectively.