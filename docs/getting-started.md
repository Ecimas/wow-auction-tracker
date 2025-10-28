# Getting Started

This guide will help you set up and run the WoW Auction Tracker application on your local machine.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** (v18.0.0 or higher)
- **npm** (v8.0.0 or higher) or **yarn** (v1.22.0 or higher)
- **PostgreSQL** (v13.0 or higher)
- **Git**

### System Requirements

- **Operating System**: Windows 10+, macOS 10.15+, or Linux (Ubuntu 18.04+)
- **RAM**: Minimum 4GB, recommended 8GB+
- **Storage**: At least 2GB free space
- **Network**: Stable internet connection for data fetching

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/wow-auction-tracker.git
cd wow-auction-tracker
```

### 2. Install Dependencies

```bash
# Using npm
npm install

# Or using yarn
yarn install
```

### 3. Environment Configuration

Create a `.env` file in the root directory:

```bash
cp .env.example .env
```

Edit the `.env` file with your configuration:

```env
# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/wow_auction_tracker
DB_HOST=localhost
DB_PORT=5432
DB_NAME=wow_auction_tracker
DB_USER=your_username
DB_PASSWORD=your_password

# Application Configuration
NODE_ENV=development
PORT=3000
API_BASE_URL=http://localhost:3000/api

# WoW API Configuration
WOW_API_KEY=your_blizzard_api_key
WOW_API_REGION=us
WOW_API_LOCALE=en_US

# Redis Configuration (for caching)
REDIS_URL=redis://localhost:6379

# JWT Configuration
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=7d

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

### 4. Database Setup

#### Install PostgreSQL

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

**macOS (using Homebrew):**
```bash
brew install postgresql
brew services start postgresql
```

**Windows:**
Download and install from [PostgreSQL official website](https://www.postgresql.org/download/windows/)

#### Create Database

```bash
# Connect to PostgreSQL
sudo -u postgres psql

# Create database and user
CREATE DATABASE wow_auction_tracker;
CREATE USER wow_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE wow_auction_tracker TO wow_user;
\q
```

#### Run Migrations

```bash
npm run db:migrate
```

### 5. Start the Application

#### Development Mode

```bash
# Start both frontend and backend
npm run dev

# Or start them separately
npm run dev:backend
npm run dev:frontend
```

#### Production Mode

```bash
# Build the application
npm run build

# Start production server
npm start
```

## Verification

Once the application is running, you can verify the installation:

1. **Frontend**: Open `http://localhost:3000` in your browser
2. **Backend API**: Test `http://localhost:3000/api/health`
3. **Database**: Check database connection in the admin panel

### Health Check Endpoints

- `GET /api/health` - Application health status
- `GET /api/health/database` - Database connection status
- `GET /api/health/redis` - Redis connection status

## Configuration Options

### Application Settings

| Setting | Description | Default | Environment Variable |
|---------|-------------|---------|---------------------|
| Port | Server port | 3000 | PORT |
| Environment | Application environment | development | NODE_ENV |
| Log Level | Logging verbosity | info | LOG_LEVEL |
| CORS Origins | Allowed CORS origins | * | CORS_ORIGINS |

### Database Settings

| Setting | Description | Default | Environment Variable |
|---------|-------------|---------|---------------------|
| Host | Database host | localhost | DB_HOST |
| Port | Database port | 5432 | DB_PORT |
| Name | Database name | wow_auction_tracker | DB_NAME |
| User | Database user | - | DB_USER |
| Password | Database password | - | DB_PASSWORD |

### WoW API Settings

| Setting | Description | Default | Environment Variable |
|---------|-------------|---------|---------------------|
| API Key | Blizzard API key | - | WOW_API_KEY |
| Region | WoW region | us | WOW_API_REGION |
| Locale | Language locale | en_US | WOW_API_LOCALE |

## Troubleshooting

### Common Issues

#### Port Already in Use

```bash
# Find process using port 3000
lsof -i :3000

# Kill the process
kill -9 <PID>
```

#### Database Connection Failed

1. Verify PostgreSQL is running: `sudo systemctl status postgresql`
2. Check database credentials in `.env`
3. Ensure database exists: `psql -U wow_user -d wow_auction_tracker`

#### Missing Dependencies

```bash
# Clear npm cache and reinstall
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

#### Permission Issues

```bash
# Fix npm permissions (Linux/macOS)
sudo chown -R $(whoami) ~/.npm
```

### Getting Help

If you encounter issues not covered here:

1. Check the [Troubleshooting Guide](./troubleshooting.md)
2. Search existing [GitHub Issues](https://github.com/your-username/wow-auction-tracker/issues)
3. Create a new issue with detailed information
4. Join our Discord community for real-time support

## Next Steps

Now that you have the application running:

1. **Explore the API**: Check out the [API Reference](./api-reference.md)
2. **Learn Components**: Read the [Component Library](./components.md)
3. **See Examples**: Browse [Usage Examples](./examples.md)
4. **Configure Settings**: Review [Configuration Options](./configuration.md)

## Development Workflow

### Recommended Development Setup

1. **IDE**: VS Code with recommended extensions
2. **Version Control**: Git with conventional commits
3. **Testing**: Jest for unit tests, Cypress for E2E tests
4. **Code Quality**: ESLint, Prettier, and Husky pre-commit hooks

### Available Scripts

```bash
# Development
npm run dev              # Start development server
npm run dev:backend      # Start backend only
npm run dev:frontend     # Start frontend only

# Building
npm run build            # Build for production
npm run build:backend    # Build backend only
npm run build:frontend   # Build frontend only

# Testing
npm test                 # Run all tests
npm run test:unit        # Run unit tests
npm run test:e2e         # Run E2E tests
npm run test:coverage    # Run tests with coverage

# Database
npm run db:migrate       # Run database migrations
npm run db:seed          # Seed database with sample data
npm run db:reset         # Reset database

# Code Quality
npm run lint             # Run ESLint
npm run lint:fix         # Fix ESLint issues
npm run format           # Format code with Prettier
```