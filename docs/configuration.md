# Configuration Guide

This guide covers all configuration options available in the WoW Auction Tracker application.

## Table of Contents

- [Environment Variables](#environment-variables)
- [Application Configuration](#application-configuration)
- [Database Configuration](#database-configuration)
- [API Configuration](#api-configuration)
- [Security Configuration](#security-configuration)
- [Performance Configuration](#performance-configuration)
- [Logging Configuration](#logging-configuration)
- [Feature Flags](#feature-flags)

## Environment Variables

### Core Application Settings

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `NODE_ENV` | Application environment | `development` | No |
| `PORT` | Server port | `3000` | No |
| `API_BASE_URL` | Base URL for API endpoints | `http://localhost:3000/api` | No |
| `FRONTEND_URL` | Frontend application URL | `http://localhost:3000` | No |

### Database Configuration

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DATABASE_URL` | Complete database connection string | - | Yes |
| `DB_HOST` | Database host | `localhost` | No |
| `DB_PORT` | Database port | `5432` | No |
| `DB_NAME` | Database name | `wow_auction_tracker` | No |
| `DB_USER` | Database username | - | Yes |
| `DB_PASSWORD` | Database password | - | Yes |
| `DB_SSL` | Enable SSL connection | `false` | No |
| `DB_POOL_MIN` | Minimum connection pool size | `2` | No |
| `DB_POOL_MAX` | Maximum connection pool size | `10` | No |

### WoW API Configuration

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `WOW_API_KEY` | Blizzard API key | - | Yes |
| `WOW_API_SECRET` | Blizzard API secret | - | Yes |
| `WOW_API_REGION` | WoW region (us, eu, kr, tw) | `us` | No |
| `WOW_API_LOCALE` | API locale | `en_US` | No |
| `WOW_API_BASE_URL` | Blizzard API base URL | `https://us.api.blizzard.com` | No |

### Redis Configuration

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `REDIS_URL` | Redis connection string | `redis://localhost:6379` | No |
| `REDIS_HOST` | Redis host | `localhost` | No |
| `REDIS_PORT` | Redis port | `6379` | No |
| `REDIS_PASSWORD` | Redis password | - | No |
| `REDIS_DB` | Redis database number | `0` | No |

### Authentication & Security

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `JWT_SECRET` | JWT signing secret | - | Yes |
| `JWT_EXPIRES_IN` | JWT expiration time | `7d` | No |
| `JWT_REFRESH_EXPIRES_IN` | Refresh token expiration | `30d` | No |
| `BCRYPT_ROUNDS` | Bcrypt salt rounds | `12` | No |
| `CORS_ORIGINS` | Allowed CORS origins | `*` | No |
| `RATE_LIMIT_WINDOW_MS` | Rate limit window | `900000` | No |
| `RATE_LIMIT_MAX_REQUESTS` | Max requests per window | `100` | No |

### Email Configuration

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `SMTP_HOST` | SMTP server host | - | No |
| `SMTP_PORT` | SMTP server port | `587` | No |
| `SMTP_USER` | SMTP username | - | No |
| `SMTP_PASSWORD` | SMTP password | - | No |
| `SMTP_FROM` | From email address | - | No |
| `EMAIL_VERIFICATION_REQUIRED` | Require email verification | `false` | No |

### File Storage

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `STORAGE_TYPE` | Storage type (local, s3, gcs) | `local` | No |
| `STORAGE_PATH` | Local storage path | `./uploads` | No |
| `AWS_ACCESS_KEY_ID` | AWS access key | - | No |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | - | No |
| `AWS_REGION` | AWS region | `us-east-1` | No |
| `AWS_S3_BUCKET` | S3 bucket name | - | No |

## Application Configuration

### config/app.js

```javascript
module.exports = {
  // Application settings
  name: process.env.APP_NAME || 'WoW Auction Tracker',
  version: process.env.APP_VERSION || '1.0.0',
  description: 'World of Warcraft Auction House Tracker',
  
  // Server configuration
  server: {
    port: parseInt(process.env.PORT) || 3000,
    host: process.env.HOST || '0.0.0.0',
    timeout: parseInt(process.env.SERVER_TIMEOUT) || 30000
  },
  
  // API configuration
  api: {
    version: 'v1',
    prefix: '/api',
    timeout: parseInt(process.env.API_TIMEOUT) || 10000,
    retries: parseInt(process.env.API_RETRIES) || 3
  },
  
  // Pagination defaults
  pagination: {
    defaultLimit: parseInt(process.env.PAGINATION_LIMIT) || 50,
    maxLimit: parseInt(process.env.PAGINATION_MAX_LIMIT) || 100
  },
  
  // Cache settings
  cache: {
    ttl: parseInt(process.env.CACHE_TTL) || 300, // 5 minutes
    maxSize: parseInt(process.env.CACHE_MAX_SIZE) || 1000
  }
};
```

### config/database.js

```javascript
module.exports = {
  development: {
    client: 'postgresql',
    connection: {
      host: process.env.DB_HOST || 'localhost',
      port: parseInt(process.env.DB_PORT) || 5432,
      database: process.env.DB_NAME || 'wow_auction_tracker_dev',
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      ssl: process.env.DB_SSL === 'true' ? { rejectUnauthorized: false } : false
    },
    pool: {
      min: parseInt(process.env.DB_POOL_MIN) || 2,
      max: parseInt(process.env.DB_POOL_MAX) || 10
    },
    migrations: {
      directory: './database/migrations'
    },
    seeds: {
      directory: './database/seeds'
    }
  },
  
  production: {
    client: 'postgresql',
    connection: process.env.DATABASE_URL,
    pool: {
      min: parseInt(process.env.DB_POOL_MIN) || 5,
      max: parseInt(process.env.DB_POOL_MAX) || 20
    },
    migrations: {
      directory: './database/migrations'
    }
  }
};
```

## Database Configuration

### Connection Pool Settings

```javascript
// config/database-pool.js
module.exports = {
  // Connection pool configuration
  pool: {
    // Minimum number of connections
    min: parseInt(process.env.DB_POOL_MIN) || 2,
    
    // Maximum number of connections
    max: parseInt(process.env.DB_POOL_MAX) || 10,
    
    // Connection timeout in milliseconds
    acquireTimeoutMillis: parseInt(process.env.DB_ACQUIRE_TIMEOUT) || 30000,
    
    // Idle timeout in milliseconds
    idleTimeoutMillis: parseInt(process.env.DB_IDLE_TIMEOUT) || 30000,
    
    // Create timeout in milliseconds
    createTimeoutMillis: parseInt(process.env.DB_CREATE_TIMEOUT) || 30000,
    
    // Destroy timeout in milliseconds
    destroyTimeoutMillis: parseInt(process.env.DB_DESTROY_TIMEOUT) || 5000,
    
    // Reap interval in milliseconds
    reapIntervalMillis: parseInt(process.env.DB_REAP_INTERVAL) || 1000,
    
    // Create retry interval in milliseconds
    createRetryIntervalMillis: parseInt(process.env.DB_CREATE_RETRY_INTERVAL) || 200
  },
  
  // Query configuration
  query: {
    // Query timeout in milliseconds
    timeout: parseInt(process.env.DB_QUERY_TIMEOUT) || 30000,
    
    // Enable query logging
    logging: process.env.DB_QUERY_LOGGING === 'true'
  }
};
```

### Migration Configuration

```javascript
// config/migrations.js
module.exports = {
  // Migration settings
  migrations: {
    // Directory containing migration files
    directory: './database/migrations',
    
    // Migration table name
    tableName: 'knex_migrations',
    
    // Disable foreign key checks during migrations
    disableForeignKeys: false,
    
    // Rollback configuration
    rollback: {
      // Maximum number of migrations to rollback
      maxRollbacks: 10,
      
      // Confirm rollbacks in production
      confirmProduction: true
    }
  },
  
  // Seed settings
  seeds: {
    // Directory containing seed files
    directory: './database/seeds',
    
    // Seed table name
    tableName: 'knex_seeds'
  }
};
```

## API Configuration

### WoW API Settings

```javascript
// config/wow-api.js
module.exports = {
  // Blizzard API configuration
  blizzard: {
    clientId: process.env.WOW_API_KEY,
    clientSecret: process.env.WOW_API_SECRET,
    region: process.env.WOW_API_REGION || 'us',
    locale: process.env.WOW_API_LOCALE || 'en_US',
    
    // API endpoints
    endpoints: {
      token: 'https://oauth.battle.net/oauth/token',
      api: 'https://us.api.blizzard.com',
      data: 'https://us.api.blizzard.com/data/wow'
    },
    
    // Rate limiting
    rateLimit: {
      requestsPerSecond: parseInt(process.env.WOW_API_RPS) || 10,
      burstLimit: parseInt(process.env.WOW_API_BURST) || 100
    },
    
    // Retry configuration
    retry: {
      attempts: parseInt(process.env.WOW_API_RETRIES) || 3,
      delay: parseInt(process.env.WOW_API_RETRY_DELAY) || 1000,
      backoff: 'exponential'
    },
    
    // Cache settings
    cache: {
      enabled: process.env.WOW_API_CACHE_ENABLED !== 'false',
      ttl: parseInt(process.env.WOW_API_CACHE_TTL) || 3600, // 1 hour
      maxSize: parseInt(process.env.WOW_API_CACHE_SIZE) || 10000
    }
  },
  
  // Data collection settings
  collection: {
    // Auction house scan interval (minutes)
    scanInterval: parseInt(process.env.AUCTION_SCAN_INTERVAL) || 15,
    
    // Maximum concurrent scans
    maxConcurrentScans: parseInt(process.env.MAX_CONCURRENT_SCANS) || 5,
    
    // Realm priority (higher priority realms scanned first)
    realmPriority: {
      'aegwynn': 10,
      'aerie-peak': 9,
      'agamaggan': 8
    },
    
    // Item filters
    itemFilters: {
      // Minimum item level to track
      minItemLevel: parseInt(process.env.MIN_ITEM_LEVEL) || 1,
      
      // Maximum item level to track
      maxItemLevel: parseInt(process.env.MAX_ITEM_LEVEL) || 1000,
      
      // Item qualities to track
      qualities: (process.env.TRACKED_QUALITIES || 'common,uncommon,rare,epic,legendary').split(','),
      
      // Categories to track
      categories: (process.env.TRACKED_CATEGORIES || 'weapon,armor,misc').split(',')
    }
  }
};
```

### Rate Limiting Configuration

```javascript
// config/rate-limiting.js
module.exports = {
  // Global rate limiting
  global: {
    windowMs: parseInt(process.env.RATE_LIMIT_WINDOW_MS) || 900000, // 15 minutes
    max: parseInt(process.env.RATE_LIMIT_MAX_REQUESTS) || 100,
    message: 'Too many requests from this IP, please try again later.',
    standardHeaders: true,
    legacyHeaders: false
  },
  
  // API-specific rate limiting
  api: {
    windowMs: parseInt(process.env.API_RATE_LIMIT_WINDOW) || 60000, // 1 minute
    max: parseInt(process.env.API_RATE_LIMIT_MAX) || 60,
    skipSuccessfulRequests: true
  },
  
  // Authentication rate limiting
  auth: {
    windowMs: parseInt(process.env.AUTH_RATE_LIMIT_WINDOW) || 900000, // 15 minutes
    max: parseInt(process.env.AUTH_RATE_LIMIT_MAX) || 5,
    skipSuccessfulRequests: false
  },
  
  // Search rate limiting
  search: {
    windowMs: parseInt(process.env.SEARCH_RATE_LIMIT_WINDOW) || 60000, // 1 minute
    max: parseInt(process.env.SEARCH_RATE_LIMIT_MAX) || 30,
    skipSuccessfulRequests: true
  }
};
```

## Security Configuration

### Authentication Settings

```javascript
// config/auth.js
module.exports = {
  // JWT configuration
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '30d',
    issuer: process.env.JWT_ISSUER || 'wow-auction-tracker',
    audience: process.env.JWT_AUDIENCE || 'wow-auction-tracker-users'
  },
  
  // Password requirements
  password: {
    minLength: parseInt(process.env.PASSWORD_MIN_LENGTH) || 8,
    requireUppercase: process.env.PASSWORD_REQUIRE_UPPERCASE !== 'false',
    requireLowercase: process.env.PASSWORD_REQUIRE_LOWERCASE !== 'false',
    requireNumbers: process.env.PASSWORD_REQUIRE_NUMBERS !== 'false',
    requireSpecialChars: process.env.PASSWORD_REQUIRE_SPECIAL !== 'false',
    maxAge: parseInt(process.env.PASSWORD_MAX_AGE) || 90 // days
  },
  
  // Session configuration
  session: {
    secret: process.env.SESSION_SECRET || process.env.JWT_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: process.env.NODE_ENV === 'production',
      httpOnly: true,
      maxAge: parseInt(process.env.SESSION_MAX_AGE) || 86400000 // 24 hours
    }
  },
  
  // Account lockout
  lockout: {
    maxAttempts: parseInt(process.env.MAX_LOGIN_ATTEMPTS) || 5,
    lockoutDuration: parseInt(process.env.LOCKOUT_DURATION) || 900000, // 15 minutes
    resetAttemptsAfter: parseInt(process.env.RESET_ATTEMPTS_AFTER) || 3600000 // 1 hour
  }
};
```

### CORS Configuration

```javascript
// config/cors.js
module.exports = {
  // CORS settings
  cors: {
    origin: function(origin, callback) {
      const allowedOrigins = process.env.CORS_ORIGINS 
        ? process.env.CORS_ORIGINS.split(',')
        : ['http://localhost:3000', 'http://localhost:3001'];
      
      // Allow requests with no origin (mobile apps, etc.)
      if (!origin) return callback(null, true);
      
      if (allowedOrigins.indexOf(origin) !== -1) {
        callback(null, true);
      } else {
        callback(new Error('Not allowed by CORS'));
      }
    },
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
    exposedHeaders: ['X-Total-Count', 'X-Page-Count'],
    maxAge: parseInt(process.env.CORS_MAX_AGE) || 86400 // 24 hours
  }
};
```

## Performance Configuration

### Caching Configuration

```javascript
// config/cache.js
module.exports = {
  // Redis cache configuration
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT) || 6379,
    password: process.env.REDIS_PASSWORD,
    db: parseInt(process.env.REDIS_DB) || 0,
    
    // Connection options
    retryDelayOnFailover: 100,
    enableReadyCheck: false,
    maxRetriesPerRequest: null,
    
    // Pool configuration
    family: 4,
    keepAlive: true,
    lazyConnect: true
  },
  
  // Cache TTL settings (in seconds)
  ttl: {
    // API responses
    api: parseInt(process.env.CACHE_API_TTL) || 300, // 5 minutes
    
    // Item data
    items: parseInt(process.env.CACHE_ITEMS_TTL) || 3600, // 1 hour
    
    // Auction data
    auctions: parseInt(process.env.CACHE_AUCTIONS_TTL) || 900, // 15 minutes
    
    // Realm data
    realms: parseInt(process.env.CACHE_REALMS_TTL) || 86400, // 24 hours
    
    // User data
    users: parseInt(process.env.CACHE_USERS_TTL) || 1800, // 30 minutes
    
    // Search results
    search: parseInt(process.env.CACHE_SEARCH_TTL) || 600 // 10 minutes
  },
  
  // Cache invalidation
  invalidation: {
    // Patterns for cache invalidation
    patterns: {
      user: 'user:*',
      item: 'item:*',
      auction: 'auction:*',
      search: 'search:*'
    },
    
    // Automatic invalidation
    autoInvalidate: process.env.CACHE_AUTO_INVALIDATE !== 'false'
  }
};
```

### Database Optimization

```javascript
// config/database-optimization.js
module.exports = {
  // Query optimization
  queries: {
    // Enable query logging in development
    logging: process.env.NODE_ENV === 'development',
    
    // Query timeout
    timeout: parseInt(process.env.DB_QUERY_TIMEOUT) || 30000,
    
    // Enable query caching
    cache: process.env.DB_QUERY_CACHE !== 'false',
    
    // Query result limits
    limits: {
      maxRows: parseInt(process.env.DB_MAX_ROWS) || 10000,
      maxJoins: parseInt(process.env.DB_MAX_JOINS) || 10
    }
  },
  
  // Index optimization
  indexes: {
    // Automatic index creation
    autoCreate: process.env.DB_AUTO_INDEX !== 'false',
    
    // Index monitoring
    monitor: process.env.DB_MONITOR_INDEXES !== 'false',
    
    // Index maintenance
    maintenance: {
      enabled: process.env.DB_INDEX_MAINTENANCE !== 'false',
      interval: parseInt(process.env.DB_INDEX_INTERVAL) || 86400000 // 24 hours
    }
  },
  
  // Connection optimization
  connections: {
    // Connection pooling
    pooling: {
      enabled: true,
      min: parseInt(process.env.DB_POOL_MIN) || 2,
      max: parseInt(process.env.DB_POOL_MAX) || 10,
      idle: parseInt(process.env.DB_POOL_IDLE) || 30000
    },
    
    // Connection monitoring
    monitoring: {
      enabled: process.env.DB_MONITOR_CONNECTIONS !== 'false',
      interval: parseInt(process.env.DB_MONITOR_INTERVAL) || 60000 // 1 minute
    }
  }
};
```

## Logging Configuration

### Logging Settings

```javascript
// config/logging.js
module.exports = {
  // Log level configuration
  level: process.env.LOG_LEVEL || 'info',
  
  // Log format
  format: process.env.LOG_FORMAT || 'combined',
  
  // Log destinations
  destinations: {
    // Console logging
    console: {
      enabled: process.env.LOG_CONSOLE !== 'false',
      level: process.env.LOG_CONSOLE_LEVEL || 'info'
    },
    
    // File logging
    file: {
      enabled: process.env.LOG_FILE !== 'false',
      level: process.env.LOG_FILE_LEVEL || 'warn',
      filename: process.env.LOG_FILE_PATH || './logs/app.log',
      maxSize: process.env.LOG_FILE_MAX_SIZE || '10m',
      maxFiles: parseInt(process.env.LOG_FILE_MAX_FILES) || 5
    },
    
    // Database logging
    database: {
      enabled: process.env.LOG_DATABASE === 'true',
      level: process.env.LOG_DATABASE_LEVEL || 'error',
      table: process.env.LOG_DATABASE_TABLE || 'logs'
    }
  },
  
  // Log categories
  categories: {
    // Application logs
    app: {
      level: process.env.LOG_APP_LEVEL || 'info',
      includeStackTrace: process.env.LOG_APP_STACK !== 'false'
    },
    
    // API logs
    api: {
      level: process.env.LOG_API_LEVEL || 'info',
      includeRequestBody: process.env.LOG_API_REQUEST === 'true',
      includeResponseBody: process.env.LOG_API_RESPONSE === 'true'
    },
    
    // Database logs
    database: {
      level: process.env.LOG_DB_LEVEL || 'warn',
      includeQueries: process.env.LOG_DB_QUERIES === 'true',
      includeSlowQueries: process.env.LOG_DB_SLOW === 'true',
      slowQueryThreshold: parseInt(process.env.LOG_DB_SLOW_THRESHOLD) || 1000
    },
    
    // Security logs
    security: {
      level: process.env.LOG_SECURITY_LEVEL || 'warn',
      includeFailedLogins: true,
      includeSuspiciousActivity: true
    }
  },
  
  // Log rotation
  rotation: {
    enabled: process.env.LOG_ROTATION !== 'false',
    schedule: process.env.LOG_ROTATION_SCHEDULE || '0 0 * * *', // Daily at midnight
    maxAge: parseInt(process.env.LOG_MAX_AGE) || 30, // days
    maxSize: process.env.LOG_MAX_SIZE || '100m'
  }
};
```

## Feature Flags

### Feature Configuration

```javascript
// config/features.js
module.exports = {
  // Feature flags
  features: {
    // User features
    userRegistration: process.env.FEATURE_USER_REGISTRATION !== 'false',
    userProfiles: process.env.FEATURE_USER_PROFILES !== 'false',
    userPreferences: process.env.FEATURE_USER_PREFERENCES !== 'false',
    
    // Auction features
    auctionTracking: process.env.FEATURE_AUCTION_TRACKING !== 'false',
    auctionAlerts: process.env.FEATURE_AUCTION_ALERTS !== 'false',
    auctionHistory: process.env.FEATURE_AUCTION_HISTORY !== 'false',
    
    // Item features
    itemSearch: process.env.FEATURE_ITEM_SEARCH !== 'false',
    itemDetails: process.env.FEATURE_ITEM_DETAILS !== 'false',
    itemFavorites: process.env.FEATURE_ITEM_FAVORITES !== 'false',
    
    // Analytics features
    marketAnalytics: process.env.FEATURE_MARKET_ANALYTICS !== 'false',
    pricePredictions: process.env.FEATURE_PRICE_PREDICTIONS !== 'false',
    trendAnalysis: process.env.FEATURE_TREND_ANALYSIS !== 'false',
    
    // Social features
    watchlists: process.env.FEATURE_WATCHLISTS !== 'false',
    sharing: process.env.FEATURE_SHARING !== 'false',
    comments: process.env.FEATURE_COMMENTS !== 'false',
    
    // Premium features
    premiumFeatures: process.env.FEATURE_PREMIUM !== 'false',
    advancedAnalytics: process.env.FEATURE_ADVANCED_ANALYTICS !== 'false',
    customAlerts: process.env.FEATURE_CUSTOM_ALERTS !== 'false',
    
    // Experimental features
    experimentalFeatures: process.env.FEATURE_EXPERIMENTAL === 'true',
    betaFeatures: process.env.FEATURE_BETA === 'true',
    alphaFeatures: process.env.FEATURE_ALPHA === 'true'
  },
  
  // Feature limits
  limits: {
    // User limits
    maxWatchlists: parseInt(process.env.MAX_WATCHLISTS) || 10,
    maxAlerts: parseInt(process.env.MAX_ALERTS) || 50,
    maxFavorites: parseInt(process.env.MAX_FAVORITES) || 100,
    
    // Data limits
    maxSearchResults: parseInt(process.env.MAX_SEARCH_RESULTS) || 1000,
    maxHistoryDays: parseInt(process.env.MAX_HISTORY_DAYS) || 365,
    maxExportRows: parseInt(process.env.MAX_EXPORT_ROWS) || 10000,
    
    // API limits
    maxApiRequests: parseInt(process.env.MAX_API_REQUESTS) || 1000,
    maxConcurrentRequests: parseInt(process.env.MAX_CONCURRENT_REQUESTS) || 10
  },
  
  // Feature dependencies
  dependencies: {
    auctionAlerts: ['auctionTracking', 'userProfiles'],
    pricePredictions: ['marketAnalytics', 'auctionHistory'],
    customAlerts: ['premiumFeatures', 'auctionAlerts']
  }
};
```

## Configuration Validation

### Validation Schema

```javascript
// config/validation.js
const Joi = require('joi');

const configSchema = Joi.object({
  // Application
  NODE_ENV: Joi.string().valid('development', 'production', 'test').default('development'),
  PORT: Joi.number().port().default(3000),
  
  // Database
  DATABASE_URL: Joi.string().uri().required(),
  DB_HOST: Joi.string().hostname().default('localhost'),
  DB_PORT: Joi.number().port().default(5432),
  DB_NAME: Joi.string().required(),
  DB_USER: Joi.string().required(),
  DB_PASSWORD: Joi.string().required(),
  
  // WoW API
  WOW_API_KEY: Joi.string().required(),
  WOW_API_SECRET: Joi.string().required(),
  WOW_API_REGION: Joi.string().valid('us', 'eu', 'kr', 'tw').default('us'),
  
  // Security
  JWT_SECRET: Joi.string().min(32).required(),
  CORS_ORIGINS: Joi.string().default('*'),
  
  // Redis
  REDIS_URL: Joi.string().uri().default('redis://localhost:6379'),
  
  // Email
  SMTP_HOST: Joi.string().hostname().optional(),
  SMTP_PORT: Joi.number().port().default(587),
  SMTP_USER: Joi.string().optional(),
  SMTP_PASSWORD: Joi.string().optional(),
  
  // Storage
  STORAGE_TYPE: Joi.string().valid('local', 's3', 'gcs').default('local'),
  AWS_ACCESS_KEY_ID: Joi.string().when('STORAGE_TYPE', {
    is: 's3',
    then: Joi.required(),
    otherwise: Joi.optional()
  }),
  AWS_SECRET_ACCESS_KEY: Joi.string().when('STORAGE_TYPE', {
    is: 's3',
    then: Joi.required(),
    otherwise: Joi.optional()
  }),
  AWS_S3_BUCKET: Joi.string().when('STORAGE_TYPE', {
    is: 's3',
    then: Joi.required(),
    otherwise: Joi.optional()
  })
});

module.exports = {
  validate: (config) => {
    const { error, value } = configSchema.validate(config, {
      allowUnknown: true,
      stripUnknown: true
    });
    
    if (error) {
      throw new Error(`Configuration validation error: ${error.message}`);
    }
    
    return value;
  }
};
```

## Environment-Specific Configuration

### Development Configuration

```javascript
// config/development.js
module.exports = {
  // Development-specific settings
  development: {
    // Enable debug mode
    debug: true,
    
    // Detailed error messages
    detailedErrors: true,
    
    // Hot reloading
    hotReload: true,
    
    // Mock data
    useMockData: process.env.USE_MOCK_DATA === 'true',
    
    // Development tools
    devTools: {
      enabled: true,
      profiler: true,
      inspector: true
    },
    
    // Database
    database: {
      logging: true,
      migrations: {
        autoRun: true
      }
    },
    
    // API
    api: {
      mockResponses: process.env.MOCK_API_RESPONSES === 'true',
      slowDown: parseInt(process.env.API_SLOW_DOWN) || 0
    }
  }
};
```

### Production Configuration

```javascript
// config/production.js
module.exports = {
  // Production-specific settings
  production: {
    // Security
    security: {
      helmet: true,
      cors: {
        origin: process.env.CORS_ORIGINS?.split(',') || false,
        credentials: true
      }
    },
    
    // Performance
    performance: {
      compression: true,
      caching: true,
      minification: true
    },
    
    // Monitoring
    monitoring: {
      enabled: true,
      metrics: true,
      healthChecks: true
    },
    
    // Error handling
    errorHandling: {
      detailedErrors: false,
      logErrors: true,
      notifyOnError: process.env.ERROR_NOTIFICATION_EMAIL ? true : false
    }
  }
};
```

## Configuration Management

### Configuration Loader

```javascript
// config/loader.js
const path = require('path');
const fs = require('fs');

class ConfigLoader {
  constructor() {
    this.config = {};
    this.loadConfig();
  }
  
  loadConfig() {
    // Load environment variables
    this.loadEnvironment();
    
    // Load configuration files
    this.loadConfigFiles();
    
    // Validate configuration
    this.validateConfig();
  }
  
  loadEnvironment() {
    // Load .env file if it exists
    const envPath = path.join(process.cwd(), '.env');
    if (fs.existsSync(envPath)) {
      require('dotenv').config({ path: envPath });
    }
    
    // Load environment-specific .env file
    const envSpecificPath = path.join(process.cwd(), `.env.${process.env.NODE_ENV}`);
    if (fs.existsSync(envSpecificPath)) {
      require('dotenv').config({ path: envSpecificPath });
    }
  }
  
  loadConfigFiles() {
    const configDir = path.join(__dirname);
    const files = fs.readdirSync(configDir).filter(file => file.endsWith('.js'));
    
    files.forEach(file => {
      const configName = path.basename(file, '.js');
      const configPath = path.join(configDir, file);
      
      try {
        this.config[configName] = require(configPath);
      } catch (error) {
        console.warn(`Failed to load config file: ${file}`, error.message);
      }
    });
  }
  
  validateConfig() {
    const { validate } = require('./validation');
    this.config = validate(this.config);
  }
  
  get(path) {
    return path.split('.').reduce((obj, key) => obj?.[key], this.config);
  }
  
  set(path, value) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    const target = keys.reduce((obj, key) => {
      if (!obj[key]) obj[key] = {};
      return obj[key];
    }, this.config);
    target[lastKey] = value;
  }
}

module.exports = new ConfigLoader();
```

This comprehensive configuration guide covers all aspects of the WoW Auction Tracker application configuration, from basic environment variables to advanced performance and security settings. The configuration is designed to be flexible, secure, and easy to manage across different environments.