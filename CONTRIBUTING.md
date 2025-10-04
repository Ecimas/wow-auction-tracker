# Contributing Guide

Thank you for your interest in contributing to WoW Auction Tracker! This guide will help you get started with contributing to the project.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Setup](#development-setup)
4. [Project Structure](#project-structure)
5. [Development Workflow](#development-workflow)
6. [Coding Standards](#coding-standards)
7. [Testing Guidelines](#testing-guidelines)
8. [Documentation](#documentation)
9. [Submitting Changes](#submitting-changes)
10. [Review Process](#review-process)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment for all contributors. We expect everyone to:

- Be respectful and considerate in communication
- Accept constructive criticism gracefully
- Focus on what is best for the community
- Show empathy towards other community members

### Unacceptable Behavior

- Harassment, intimidation, or discrimination
- Trolling, insulting/derogatory comments
- Publishing others' private information
- Any conduct which could reasonably be considered inappropriate

### Enforcement

Instances of unacceptable behavior may be reported to the project team at conduct@wow-auction-tracker.com. All complaints will be reviewed and investigated promptly and fairly.

---

## Getting Started

### Prerequisites

Before you begin, ensure you have:

- **Node.js** 18+ installed
- **Python** 3.9+ installed (for workers)
- **Docker** and **Docker Compose** (for local development)
- **Git** for version control
- A **GitHub account**
- Basic knowledge of TypeScript/JavaScript and Python

### Finding Issues to Work On

1. Check the [Issues](https://github.com/wow-auction-tracker/wow-auction-tracker/issues) page
2. Look for issues labeled `good-first-issue` or `help-wanted`
3. Read the issue description and comments
4. Comment on the issue to express interest before starting work

### Types of Contributions

We welcome various types of contributions:

- **Bug fixes**: Fix issues reported in the issue tracker
- **New features**: Implement new functionality
- **Documentation**: Improve or add documentation
- **Tests**: Add or improve test coverage
- **Performance**: Optimize existing code
- **Refactoring**: Improve code quality and structure

---

## Development Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/wow-auction-tracker.git
cd wow-auction-tracker

# Add upstream remote
git remote add upstream https://github.com/wow-auction-tracker/wow-auction-tracker.git
```

### 2. Install Dependencies

```bash
# Install Node.js dependencies
npm install

# Install Python dependencies
cd workers
pip install -r requirements.txt
pip install -r requirements-dev.txt
cd ..
```

### 3. Set Up Environment

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your local configuration
# You'll need:
# - Database credentials
# - Redis connection string
# - Blizzard API credentials (for testing)
```

### 4. Start Development Services

```bash
# Start all services with Docker Compose
docker-compose up -d

# Services started:
# - PostgreSQL (port 5432)
# - Redis (port 6379)
# - RabbitMQ (port 5672, management UI: 15672)
# - TimescaleDB (port 5433)
```

### 5. Initialize Database

```bash
# Run database migrations
npm run migrate

# Seed with test data (optional)
npm run seed
```

### 6. Start Development Servers

```bash
# Terminal 1: Start API server
npm run dev

# Terminal 2: Start WebSocket server
npm run dev:ws

# Terminal 3: Start Python workers
cd workers
celery -A tasks worker --loglevel=info
```

### 7. Verify Setup

```bash
# Test API endpoint
curl http://localhost:3000/health

# Expected response:
# {"status": "ok", "timestamp": "2025-10-04T12:00:00Z"}
```

---

## Project Structure

```
wow-auction-tracker/
├── src/                      # Source code
│   ├── api/                  # API layer
│   │   ├── routes/           # API routes
│   │   ├── controllers/      # Request handlers
│   │   └── middleware/       # Express middleware
│   ├── services/             # Business logic
│   │   ├── auction.service.ts
│   │   ├── price.service.ts
│   │   ├── alert.service.ts
│   │   └── cache.service.ts
│   ├── models/               # Data models
│   ├── websocket/            # WebSocket server
│   ├── utils/                # Utility functions
│   └── config/               # Configuration
├── workers/                  # Python workers
│   ├── ingestion/            # Data ingestion
│   ├── analytics/            # Analytics processing
│   └── notifications/        # Notification handling
├── tests/                    # Test files
│   ├── unit/                 # Unit tests
│   ├── integration/          # Integration tests
│   └── e2e/                  # End-to-end tests
├── docs/                     # Documentation
├── migrations/               # Database migrations
├── scripts/                  # Utility scripts
└── k8s/                      # Kubernetes configs

Key Files:
├── package.json              # Node.js dependencies
├── tsconfig.json             # TypeScript configuration
├── jest.config.js            # Jest test configuration
├── .eslintrc.js              # ESLint configuration
├── .prettierrc               # Prettier configuration
└── docker-compose.yml        # Docker services
```

---

## Development Workflow

### 1. Create a Branch

```bash
# Always create a new branch for your work
git checkout -b feature/your-feature-name

# Branch naming conventions:
# - feature/feature-name  (new features)
# - fix/bug-description   (bug fixes)
# - docs/doc-description  (documentation)
# - refactor/description  (refactoring)
# - test/test-description (tests)
```

### 2. Make Changes

Write your code following our [coding standards](#coding-standards).

### 3. Write Tests

All new features and bug fixes should include tests:

```typescript
// tests/unit/services/auction.service.test.ts
import { AuctionService } from '../../../src/services/auction.service';

describe('AuctionService', () => {
  let service: AuctionService;

  beforeEach(() => {
    service = new AuctionService();
  });

  describe('findAuctions', () => {
    it('should return auctions for a valid realm', async () => {
      const result = await service.findAuctions({
        realm: 'stormrage',
        page: 1,
        limit: 10
      });

      expect(result.auctions).toBeDefined();
      expect(Array.isArray(result.auctions)).toBe(true);
    });

    it('should filter by item_id when provided', async () => {
      const result = await service.findAuctions({
        realm: 'stormrage',
        itemId: 19019,
        page: 1,
        limit: 10
      });

      result.auctions.forEach(auction => {
        expect(auction.item_id).toBe(19019);
      });
    });
  });
});
```

### 4. Run Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run specific test file
npm test -- auction.service.test.ts

# Run tests with coverage
npm test -- --coverage
```

### 5. Lint Your Code

```bash
# Run ESLint
npm run lint

# Auto-fix linting issues
npm run lint:fix

# Format code with Prettier
npm run format
```

### 6. Commit Changes

We follow [Conventional Commits](https://www.conventionalcommits.org/) specification:

```bash
# Format: <type>(<scope>): <description>

# Examples:
git commit -m "feat(api): add price alert endpoint"
git commit -m "fix(websocket): resolve connection timeout issue"
git commit -m "docs(api): update authentication examples"
git commit -m "test(services): add tests for price service"
git commit -m "refactor(database): optimize auction queries"
```

**Commit Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `perf`: Performance improvements

### 7. Keep Your Branch Updated

```bash
# Fetch latest changes from upstream
git fetch upstream

# Rebase your branch on upstream/main
git rebase upstream/main

# If conflicts occur, resolve them and continue
git rebase --continue

# Force push to your fork (after rebase)
git push origin feature/your-feature-name --force
```

---

## Coding Standards

### TypeScript/JavaScript

#### Style Guide

We use ESLint and Prettier for code formatting. Key conventions:

```typescript
// Use PascalCase for classes
class AuctionService {
  // Use camelCase for methods and variables
  async findAuctions(filters: AuctionFilters): Promise<AuctionResponse> {
    // Use const/let, not var
    const results = await this.query(filters);
    
    // Use template literals
    const message = `Found ${results.length} auctions`;
    
    // Use arrow functions for callbacks
    const mapped = results.map(r => this.transform(r));
    
    return { auctions: mapped };
  }
}

// Use UPPER_CASE for constants
const MAX_RESULTS_PER_PAGE = 500;

// Use descriptive names
const getUserById = (id: string) => { /* ... */ };  // ✓
const get = (id: string) => { /* ... */ };          // ✗
```

#### Type Safety

```typescript
// Always use types/interfaces
interface AuctionFilters {
  realm: string;
  itemId?: number;
  page: number;
  limit: number;
}

// Avoid 'any' type
const processData = (data: any) => { /* ... */ };     // ✗
const processData = (data: Auction[]) => { /* ... */ }; // ✓

// Use generics when appropriate
function paginate<T>(items: T[], page: number, limit: number): T[] {
  const start = (page - 1) * limit;
  return items.slice(start, start + limit);
}
```

#### Async/Await

```typescript
// Use async/await over promises
// Good:
async function fetchAuctions() {
  try {
    const result = await api.getAuctions();
    return result;
  } catch (error) {
    logger.error('Failed to fetch auctions', error);
    throw error;
  }
}

// Avoid:
function fetchAuctions() {
  return api.getAuctions()
    .then(result => result)
    .catch(error => {
      logger.error('Failed to fetch auctions', error);
      throw error;
    });
}
```

#### Error Handling

```typescript
// Create custom error classes
export class APIError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = 'APIError';
  }
}

// Use error classes
throw new APIError('REALM_NOT_FOUND', 'Realm does not exist', 404);

// Handle errors appropriately
try {
  await service.processAuctions();
} catch (error) {
  if (error instanceof APIError) {
    logger.error(`API Error: ${error.code}`, error);
  } else {
    logger.error('Unexpected error', error);
  }
  throw error;
}
```

### Python

#### Style Guide

Follow [PEP 8](https://www.python.org/dev/peps/pep-0008/) style guide:

```python
# Use snake_case for functions and variables
def calculate_market_value(auctions):
    total_value = sum(a.buyout * a.quantity for a in auctions)
    total_quantity = sum(a.quantity for a in auctions)
    return total_value / total_quantity

# Use PascalCase for classes
class PriceAnalyzer:
    def __init__(self, client):
        self.client = client
    
    def analyze_trends(self, item_id, realm):
        """Analyze price trends for an item"""
        history = self.client.get_price_history(item_id, realm)
        return self._calculate_statistics(history)

# Use ALL_CAPS for constants
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30

# Use type hints
def process_auction(auction: dict) -> dict:
    """Process auction data"""
    return {
        'id': auction['id'],
        'price': auction['buyout']
    }
```

#### Documentation

```python
def calculate_profit_margin(buy_price: int, sell_price: int, 
                           ah_cut: float = 0.05) -> dict:
    """
    Calculate profit margin after auction house cut.
    
    Args:
        buy_price: Purchase price in copper
        sell_price: Selling price in copper
        ah_cut: Auction house commission (default: 5%)
    
    Returns:
        Dictionary containing gross_profit, net_profit, and roi
    
    Raises:
        ValueError: If buy_price or sell_price is negative
    
    Example:
        >>> calculate_profit_margin(40000000, 50000000)
        {'gross_profit': 10000000, 'net_profit': 9500000, 'roi': 23.75}
    """
    if buy_price < 0 or sell_price < 0:
        raise ValueError("Prices cannot be negative")
    
    gross_profit = sell_price - buy_price
    net_profit = gross_profit * (1 - ah_cut)
    roi = (net_profit / buy_price) * 100
    
    return {
        'gross_profit': gross_profit,
        'net_profit': net_profit,
        'roi': roi
    }
```

---

## Testing Guidelines

### Test Structure

```typescript
// Arrange - Act - Assert pattern
describe('calculateProfit', () => {
  it('should calculate profit correctly', () => {
    // Arrange
    const buyPrice = 1000;
    const sellPrice = 1500;
    
    // Act
    const profit = calculateProfit(buyPrice, sellPrice);
    
    // Assert
    expect(profit).toBe(500);
  });
});
```

### Unit Tests

Test individual functions/methods in isolation:

```typescript
// Mock dependencies
jest.mock('../../../src/services/database.service');

describe('AuctionService', () => {
  let service: AuctionService;
  let mockDB: jest.Mocked<DatabaseService>;

  beforeEach(() => {
    mockDB = {
      query: jest.fn()
    } as any;
    service = new AuctionService(mockDB);
  });

  it('should query database with correct parameters', async () => {
    mockDB.query.mockResolvedValue([]);
    
    await service.findAuctions({ realm: 'stormrage', page: 1, limit: 10 });
    
    expect(mockDB.query).toHaveBeenCalledWith(
      expect.stringContaining('SELECT'),
      expect.arrayContaining(['stormrage'])
    );
  });
});
```

### Integration Tests

Test interaction between components:

```typescript
describe('Auction API Integration', () => {
  let app: Express;
  let db: TestDatabase;

  beforeAll(async () => {
    db = await TestDatabase.create();
    app = createApp(db);
  });

  afterAll(async () => {
    await db.close();
  });

  it('should create and retrieve auction', async () => {
    // Create auction
    const createResponse = await request(app)
      .post('/v1/auctions')
      .send({
        item_id: 19019,
        realm: 'stormrage',
        buyout: 50000000,
        quantity: 1
      })
      .expect(201);

    const auctionId = createResponse.body.data.id;

    // Retrieve auction
    const getResponse = await request(app)
      .get(`/v1/auctions/${auctionId}`)
      .expect(200);

    expect(getResponse.body.data.item_id).toBe(19019);
  });
});
```

### Test Coverage

Maintain minimum 80% code coverage:

```bash
# Run tests with coverage
npm test -- --coverage

# View coverage report
open coverage/lcov-report/index.html
```

---

## Documentation

### Code Comments

```typescript
/**
 * Calculate the market value for an item based on recent auctions
 * 
 * @param auctions - Array of recent auctions for the item
 * @returns Market value in copper
 * 
 * @example
 * ```typescript
 * const auctions = await getRecentAuctions(19019, 'stormrage');
 * const marketValue = calculateMarketValue(auctions);
 * console.log(`Market value: ${marketValue} copper`);
 * ```
 */
function calculateMarketValue(auctions: Auction[]): number {
  // Use weighted average based on quantity
  const totalValue = auctions.reduce(
    (sum, a) => sum + (a.buyout * a.quantity),
    0
  );
  const totalQuantity = auctions.reduce((sum, a) => sum + a.quantity, 0);
  
  return Math.floor(totalValue / totalQuantity);
}
```

### API Documentation

When adding new endpoints, update API.md:

```markdown
#### Get Auction Statistics

Get statistical analysis for auctions on a realm.

**Endpoint:** `GET /auctions/stats`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `realm` | string | Yes | Realm name |
| `item_id` | integer | No | Filter by item |

**Example Request:**

\`\`\`bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.wow-auction-tracker.com/v1/auctions/stats?realm=stormrage"
\`\`\`

**Example Response:**

\`\`\`json
{
  "success": true,
  "data": {
    "total_auctions": 15000,
    "total_value": 5000000000,
    "unique_items": 3500
  }
}
\`\`\`
```

---

## Submitting Changes

### 1. Push Your Branch

```bash
git push origin feature/your-feature-name
```

### 2. Create Pull Request

1. Go to your fork on GitHub
2. Click "New Pull Request"
3. Select your branch
4. Fill in the PR template:

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] All tests passing

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings generated

## Related Issues
Closes #123
```

### 3. Wait for Review

Maintainers will review your PR and may request changes.

---

## Review Process

### What Reviewers Look For

1. **Correctness**: Does the code work as intended?
2. **Tests**: Are there adequate tests?
3. **Code Quality**: Is the code clean and maintainable?
4. **Performance**: Are there any performance concerns?
5. **Security**: Are there any security issues?
6. **Documentation**: Is the code well-documented?

### Responding to Feedback

```bash
# Make requested changes
git add .
git commit -m "fix: address review feedback"
git push origin feature/your-feature-name

# If asked to squash commits:
git rebase -i HEAD~3  # Interactive rebase last 3 commits
# Mark commits to squash, save and exit
git push origin feature/your-feature-name --force
```

### After Approval

Once approved:
1. A maintainer will merge your PR
2. Your branch will be deleted
3. Changes will be included in the next release

### Cleaning Up

```bash
# Delete local branch
git checkout main
git branch -D feature/your-feature-name

# Delete remote branch (if not auto-deleted)
git push origin --delete feature/your-feature-name

# Update your main branch
git pull upstream main
```

---

## Additional Resources

### Documentation

- [API Documentation](./API.md)
- [Architecture Guide](./ARCHITECTURE.md)
- [Usage Examples](./USAGE.md)

### Communication

- **Discord**: https://discord.gg/wow-auction-tracker
- **Forum**: https://community.wow-auction-tracker.com
- **Email**: dev@wow-auction-tracker.com

### Tools

- [GitHub Issues](https://github.com/wow-auction-tracker/wow-auction-tracker/issues)
- [Project Board](https://github.com/wow-auction-tracker/wow-auction-tracker/projects)
- [Wiki](https://github.com/wow-auction-tracker/wow-auction-tracker/wiki)

---

## Questions?

If you have questions:

1. Check existing documentation
2. Search [closed issues](https://github.com/wow-auction-tracker/wow-auction-tracker/issues?q=is%3Aissue+is%3Aclosed)
3. Ask in [Discord](https://discord.gg/wow-auction-tracker)
4. Open a [new issue](https://github.com/wow-auction-tracker/wow-auction-tracker/issues/new)

Thank you for contributing to WoW Auction Tracker! 🎉

---

**Last Updated:** October 4, 2025
