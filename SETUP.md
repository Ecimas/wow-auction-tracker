# WoW Auction Tracker - Setup Guide

## Overview

This guide provides step-by-step instructions for setting up and configuring the WoW Auction Tracker API and components in your development environment.

## Prerequisites

### System Requirements

- **Node.js**: Version 16.0 or higher
- **npm**: Version 8.0 or higher (or yarn/pnpm)
- **Git**: Version 2.0 or higher
- **Blizzard API Access**: Valid Blizzard Developer API credentials

### Hardware Requirements

- **Minimum**: 2GB RAM, 1GB storage
- **Recommended**: 4GB RAM, 2GB storage
- **Production**: 8GB+ RAM, SSD storage recommended

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/wow-auction-tracker.git
cd wow-auction-tracker
```

### 2. Install Dependencies

```bash
npm install
# or
yarn install
# or
pnpm install
```

### 3. Configure Environment Variables

Create a `.env` file in the root directory:

```bash
# API Configuration
PORT=3000
NODE_ENV=development

# Blizzard API Credentials
BLIZZARD_CLIENT_ID=your_blizzard_client_id
BLIZZARD_CLIENT_SECRET=your_blizzard_client_secret
BLIZZARD_REGION=us

# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/wow_auctions
REDIS_URL=redis://localhost:6379

# API Keys (for production)
JWT_SECRET=your_jwt_secret_key
API_ENCRYPTION_KEY=your_32_character_encryption_key

# External Services (optional)
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...

# Logging
LOG_LEVEL=info
LOG_FILE=logs/app.log

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Cache Configuration
CACHE_TTL_AUCTIONS=300000
CACHE_TTL_ANALYSIS=600000
```

### 4. Set Up Database

```bash
# Using PostgreSQL
createdb wow_auctions

# Run migrations
npm run db:migrate
# or
yarn db:migrate

# Seed with sample data (optional)
npm run db:seed
```

### 5. Start the Application

```bash
# Development mode
npm run dev
# or
yarn dev

# Production mode
npm run build
npm start
```

The API will be available at `http://localhost:3000`

## Development Setup

### Project Structure

```
wow-auction-tracker/
├── src/
│   ├── api/           # API routes and handlers
│   ├── components/    # React components
│   ├── services/      # Business logic services
│   ├── models/        # Data models and types
│   ├── utils/         # Utility functions
│   ├── middleware/    # Express middleware
│   └── config/        # Configuration files
├── tests/             # Test files
├── docs/              # Documentation
├── public/            # Static assets
├── package.json
├── tsconfig.json
└── README.md
```

### Available Scripts

```json
{
  "scripts": {
    "dev": "nodemon src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "db:migrate": "prisma migrate deploy",
    "db:seed": "prisma db seed",
    "docs:generate": "typedoc src --out docs/api"
  }
}
```

## Configuration

### Environment Variables Reference

#### API Configuration
- `PORT`: Server port (default: 3000)
- `NODE_ENV`: Environment mode (`development`, `production`, `test`)

#### Blizzard API Configuration
- `BLIZZARD_CLIENT_ID`: Your Blizzard API client ID
- `BLIZZARD_CLIENT_SECRET`: Your Blizzard API client secret
- `BLIZZARD_REGION`: API region (`us`, `eu`, `kr`, `tw`, `cn`)

#### Database Configuration
- `DATABASE_URL`: PostgreSQL connection string
- `REDIS_URL`: Redis connection string (optional, for caching)

#### Security
- `JWT_SECRET`: Secret key for JWT token generation
- `API_ENCRYPTION_KEY`: 32-character key for sensitive data encryption

#### External Integrations
- `DISCORD_WEBHOOK_URL`: Discord webhook for notifications
- `SLACK_WEBHOOK_URL`: Slack webhook for notifications
- `EMAIL_SMTP_*`: SMTP configuration for email notifications

#### Performance & Monitoring
- `LOG_LEVEL`: Logging level (`debug`, `info`, `warn`, `error`)
- `RATE_LIMIT_*`: Rate limiting configuration
- `CACHE_TTL_*`: Cache time-to-live settings

### Blizzard API Setup

1. **Create Blizzard Developer Account**
   - Go to [Blizzard Developer Portal](https://develop.battle.net)
   - Sign up for a developer account

2. **Register Your Application**
   - Create a new application in the developer portal
   - Select "World of Warcraft" as the game
   - Choose appropriate redirect URIs

3. **Get API Credentials**
   - Note your Client ID and Client Secret
   - These will be used in your `.env` file

4. **Configure API Access**
   - Ensure your application has access to WoW auction data APIs
   - Test your credentials with a simple API call

## Database Setup

### PostgreSQL Setup

```bash
# Install PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib

# Create database user
sudo -u postgres createuser --interactive --pwprompt wow_tracker

# Create database
sudo -u postgres createdb -O wow_tracker wow_auctions

# Grant permissions
sudo -u postgres psql
GRANT ALL PRIVILEGES ON DATABASE wow_auctions TO wow_tracker;
\q
```

### Redis Setup (Optional)

```bash
# Install Redis
sudo apt install redis-server

# Configure Redis (optional)
sudo nano /etc/redis/redis.conf
# Set maxmemory and maxmemory-policy if needed

# Start Redis service
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

### Database Migrations

The application uses Prisma ORM for database management:

```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate deploy

# View database schema
npx prisma studio

# Reset database (development only)
npx prisma migrate reset
```

## API Key Management

### Generating API Keys

```bash
# Generate a new API key for a user
npm run scripts:generate-api-key user@example.com

# Or via API
curl -X POST http://localhost:3000/api/auth/api-key \
  -H "Authorization: Bearer your-admin-token" \
  -d '{"email": "user@example.com"}'
```

### API Key Security

- Store API keys securely (environment variables, secret management)
- Rotate keys regularly
- Use different keys for different environments
- Monitor API key usage

## Development Workflow

### Code Organization

- **API Routes**: `src/api/routes/`
- **Business Logic**: `src/services/`
- **Data Models**: `src/models/`
- **Utilities**: `src/utils/`
- **Components**: `src/components/` (for frontend)

### Adding New Features

1. **Create Feature Branch**
   ```bash
   git checkout -b feature/new-analysis-tool
   ```

2. **Implement Feature**
   - Add API routes in `src/api/routes/`
   - Implement business logic in `src/services/`
   - Add data models in `src/models/`
   - Update tests

3. **Test Feature**
   ```bash
   npm test
   npm run test:coverage
   ```

4. **Update Documentation**
   - Update API docs
   - Add usage examples
   - Update README

5. **Submit Pull Request**
   ```bash
   git add .
   git commit -m "Add new analysis tool"
   git push origin feature/new-analysis-tool
   ```

## Testing

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run specific test file
npx jest src/services/auctionService.test.ts
```

### Test Categories

- **Unit Tests**: Individual function testing
- **Integration Tests**: API endpoint testing
- **E2E Tests**: Full workflow testing
- **Performance Tests**: Load and stress testing

### Writing Tests

```typescript
import { AuctionService } from '../services/AuctionService';

describe('AuctionService', () => {
  let service: AuctionService;

  beforeEach(() => {
    service = new AuctionService();
  });

  it('should fetch auctions successfully', async () => {
    const auctions = await service.getAuctions({ realm: 'Stormrage' });
    expect(auctions).toBeDefined();
    expect(Array.isArray(auctions)).toBe(true);
  });
});
```

## Deployment

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist
COPY prisma ./prisma

RUN npx prisma generate

EXPOSE 3000
CMD ["npm", "start"]
```

```bash
# Build and run
docker build -t wow-auction-tracker .
docker run -p 3000:3000 \
  -e DATABASE_URL=$DATABASE_URL \
  -e BLIZZARD_CLIENT_ID=$BLIZZARD_CLIENT_ID \
  wow-auction-tracker
```

### Production Deployment

#### Using PM2

```bash
# Install PM2
npm install -g pm2

# Start application
pm2 start dist/index.js --name "wow-auction-tracker"

# Enable auto-restart
pm2 startup
pm2 save

# Monitor application
pm2 monit
```

#### Using Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - BLIZZARD_CLIENT_ID=${BLIZZARD_CLIENT_ID}
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=wow_auctions
      - POSTGRES_USER=wow_tracker
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Environment Setup

#### Development Environment

```bash
# Install development tools
npm install -g nodemon ts-node typescript

# Set up pre-commit hooks
npx husky install
npm run prepare

# Install API testing tools
npm install -g @types/jest
```

#### Production Environment

```bash
# Install production tools
npm install -g pm2

# Set up log rotation
sudo apt install logrotate
sudo nano /etc/logrotate.d/wow-auction-tracker

# Configure firewall
sudo ufw allow 3000
sudo ufw enable
```

## Monitoring and Maintenance

### Health Checks

The API includes built-in health check endpoints:

```bash
# Health check
curl http://localhost:3000/health

# Database health
curl http://localhost:3000/health/db

# External services health
curl http://localhost:3000/health/external
```

### Logging

Configure logging levels in your `.env` file:

```bash
# Development
LOG_LEVEL=debug

# Production
LOG_LEVEL=info
```

Log files are written to `logs/` directory:

```
logs/
├── app.log          # Application logs
├── error.log        # Error logs
├── access.log       # Access logs
└── performance.log  # Performance metrics
```

### Performance Monitoring

```bash
# Enable performance monitoring
npm install clinic

# Run performance analysis
clinic doctor -- node dist/index.js

# Memory usage analysis
clinic heapprofiler -- node dist/index.js
```

## Troubleshooting

### Common Issues

#### Database Connection Issues

```bash
# Check database connectivity
psql $DATABASE_URL -c "SELECT 1;"

# View database logs
sudo tail -f /var/log/postgresql/postgresql-*.log
```

#### API Rate Limiting

```bash
# Check rate limit status
curl -H "X-API-Key: your-key" http://localhost:3000/api/rate-limit

# Monitor rate limit headers
curl -v -H "X-API-Key: your-key" http://localhost:3000/api/auctions
```

#### Memory Issues

```bash
# Check memory usage
pm2 monit

# Restart application
pm2 restart wow-auction-tracker

# Clear cache
curl -X POST http://localhost:3000/api/admin/cache/clear
```

### Debug Mode

Enable debug logging for troubleshooting:

```bash
# Set debug level
export DEBUG=wow-auction-tracker:*

# Or in .env
LOG_LEVEL=debug
```

### Getting Help

1. **Check the Logs**: `tail -f logs/app.log`
2. **Review Documentation**: Check `/docs` folder
3. **Community Support**: GitHub Issues
4. **Professional Support**: Contact the development team

## Security Considerations

### API Key Security

- Never commit API keys to version control
- Use environment variables or secret management systems
- Rotate keys regularly
- Monitor for unauthorized usage

### Data Protection

- Encrypt sensitive data at rest
- Use HTTPS in production
- Implement proper authentication
- Regular security audits

### Network Security

- Use firewalls appropriately
- Monitor for suspicious activity
- Keep dependencies updated
- Regular vulnerability scanning

## Performance Optimization

### Caching Strategy

```bash
# Configure Redis caching
export REDIS_URL=redis://localhost:6379

# Set appropriate TTL values
CACHE_TTL_AUCTIONS=300000    # 5 minutes
CACHE_TTL_ANALYSIS=600000    # 10 minutes
```

### Database Optimization

```sql
-- Create indexes for better performance
CREATE INDEX idx_auctions_item_id ON auctions(item_id);
CREATE INDEX idx_auctions_realm ON auctions(realm);
CREATE INDEX idx_auctions_price ON auctions(buyout_price);
CREATE INDEX idx_auctions_timestamp ON auctions(created_at);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM auctions WHERE realm = 'Stormrage' AND buyout_price < 100000;
```

### API Optimization

- Implement pagination for large responses
- Use compression for API responses
- Cache frequently accessed data
- Implement request batching where appropriate

## Backup and Recovery

### Database Backups

```bash
# Create backup
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore from backup
psql $DATABASE_URL < backup_20240115_120000.sql

# Automated backups
# Set up cron job for regular backups
echo "0 2 * * * pg_dump $DATABASE_URL > /backups/backup_\$(date +\%Y\%m\%d_\%H\%M\%S).sql" | crontab -
```

### Application Backups

```bash
# Backup application code and configuration
tar -czf app_backup_$(date +%Y%m%d_%H%M%S).tar.gz \
  --exclude=node_modules \
  --exclude=.git \
  --exclude=logs \
  .

# Restore application
tar -xzf app_backup_20240115_120000.tar.gz
```

## Updates and Maintenance

### Updating Dependencies

```bash
# Check for outdated packages
npm outdated

# Update packages
npm update

# Update major versions carefully
npm install package@latest
npm test  # Ensure tests pass after updates
```

### Application Updates

```bash
# Pull latest changes
git pull origin main

# Install new dependencies
npm install

# Run database migrations
npm run db:migrate

# Build and restart
npm run build
pm2 restart wow-auction-tracker
```

### Rollback Procedures

```bash
# Quick rollback to previous version
pm2 stop wow-auction-tracker
git reset --hard HEAD~1
npm install
npm run build
pm2 start wow-auction-tracker

# Database rollback (if needed)
psql $DATABASE_URL < rollback_script.sql
```

## API Testing

### Using cURL

```bash
# Get auctions
curl -H "X-API-Key: your-key" \
     "http://localhost:3000/api/auctions?realm=Stormrage&limit=10"

# Create price alert
curl -X POST http://localhost:3000/api/alerts \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{
    "itemId": 12345,
    "realm": "Stormrage",
    "alertType": "BELOW",
    "price": 150000
  }'

# Get market analysis
curl -H "X-API-Key: your-key" \
     "http://localhost:3000/api/market/opportunities?minProfit=50000"
```

### Using Postman

1. Import the provided Postman collection
2. Set environment variables:
   - `baseUrl`: `http://localhost:3000`
   - `apiKey`: `your-api-key`
3. Test endpoints with different parameters

### Automated API Testing

```bash
# Install testing tools
npm install -g newman artillery

# Run API tests
newman run WowAuctionTracker.postman_collection.json \
  --environment WowAuctionTracker.postman_environment.json

# Load testing
artillery run load-test.yml
```

## Support and Resources

### Documentation

- **API Documentation**: `/docs/API.md`
- **Component Guide**: `/docs/COMPONENTS.md`
- **Examples**: `/docs/EXAMPLES.md`
- **Models**: `/docs/MODELS.md`

### Community

- **GitHub Issues**: Report bugs and feature requests
- **Discussions**: Ask questions and share ideas
- **Wiki**: Community-contributed guides

### Professional Support

For enterprise customers:
- Email: support@wowauctiontracker.com
- Phone: +1 (555) 123-4567
- Response time: 24-48 hours

## License and Terms

This software is licensed under the MIT License. See LICENSE file for details.

By using this software, you agree to:
- Comply with Blizzard API terms of service
- Respect rate limits and usage guidelines
- Not use for malicious purposes
- Keep API credentials secure

---

**Need Help?** Check our [troubleshooting guide](#troubleshooting) or [contact support](mailto:support@wowauctiontracker.com). Happy auction tracking! 🎯