# WoW Auction Tracker - Components

## Overview

This document describes the key components and services that make up the WoW Auction Tracker application. Components are organized into frontend UI components, data processing components, and backend services.

## Frontend Components

### Core UI Components

#### AuctionList

Displays a paginated list of auction items with filtering and sorting capabilities.

```typescript
interface AuctionListProps {
  auctions: AuctionItem[];
  loading?: boolean;
  error?: string;
  onAuctionSelect?: (auction: AuctionItem) => void;
  onSort?: (field: string, direction: SortOrder) => void;
  filters?: AuctionFilters;
}

function AuctionList({
  auctions,
  loading = false,
  error,
  onAuctionSelect,
  onSort,
  filters
}: AuctionListProps)
```

**Features:**
- Virtual scrolling for large lists
- Real-time price updates
- Advanced filtering (price range, seller, time left)
- Sorting by price, time, quantity, seller
- Bulk selection for watchlist
- Export functionality

**Usage:**
```jsx
<AuctionList
  auctions={auctions}
  onAuctionSelect={(auction) => setSelectedAuction(auction)}
  onSort={(field, direction) => handleSort(field, direction)}
  filters={{
    minPrice: 100000,
    maxPrice: 500000,
    timeLeft: ['MEDIUM', 'LONG']
  }}
/>
```

#### AuctionCard

Individual auction item display card.

```typescript
interface AuctionCardProps {
  auction: AuctionItem;
  showDetails?: boolean;
  highlight?: boolean;
  onClick?: () => void;
  actions?: React.ReactNode;
}

function AuctionCard({
  auction,
  showDetails = false,
  highlight = false,
  onClick,
  actions
}: AuctionCardProps)
```

**Features:**
- Item icon and quality border
- Price display with formatting
- Time remaining indicator
- Seller information
- Quick action buttons
- Tooltip with full item details

#### MarketAnalysisChart

Interactive chart component for price trends and market analysis.

```typescript
interface MarketAnalysisChartProps {
  data: PriceHistory;
  type: 'line' | 'bar' | 'candlestick';
  timeframe: '1D' | '7D' | '30D' | '90D';
  indicators?: ChartIndicator[];
  onPointClick?: (point: PricePoint) => void;
}

function MarketAnalysisChart({
  data,
  type = 'line',
  timeframe,
  indicators = [],
  onPointClick
}: MarketAnalysisChartProps)
```

**Features:**
- Multiple chart types (line, bar, candlestick)
- Technical indicators (SMA, EMA, RSI, MACD)
- Interactive tooltips
- Zoom and pan functionality
- Export chart as image
- Responsive design

#### PriceAlertForm

Form component for creating and managing price alerts.

```typescript
interface PriceAlertFormProps {
  itemId?: ItemId;
  realm?: string;
  onSubmit: (alert: PriceAlertConfig) => void;
  onCancel?: () => void;
  existingAlert?: PriceAlert;
}

function PriceAlertForm({
  itemId,
  realm,
  onSubmit,
  onCancel,
  existingAlert
}: PriceAlertFormProps)
```

**Features:**
- Alert type selection (above/below price)
- Price input with validation
- Notification method configuration
- Cooldown period settings
- Preview of alert conditions

### Dashboard Components

#### Dashboard

Main application dashboard with overview widgets.

```typescript
interface DashboardProps {
  user: User;
  widgets?: DashboardWidget[];
  layout?: 'grid' | 'list';
  onWidgetAdd?: (widget: DashboardWidget) => void;
  onWidgetRemove?: (widgetId: string) => void;
}

function Dashboard({
  user,
  widgets = [],
  layout = 'grid',
  onWidgetAdd,
  onWidgetRemove
}: DashboardProps)
```

**Features:**
- Customizable widget layout
- Real-time data updates
- Widget drag and drop
- Export dashboard data
- Responsive grid system

#### WatchlistWidget

Displays user's watched items with price alerts.

```typescript
interface WatchlistWidgetProps {
  items: WatchlistItem[];
  alerts: PriceAlert[];
  onItemRemove?: (itemId: ItemId) => void;
  onAlertCreate?: (itemId: ItemId) => void;
  compact?: boolean;
}

function WatchlistWidget({
  items,
  alerts,
  onItemRemove,
  onAlertCreate,
  compact = false
}: WatchlistWidgetProps)
```

**Features:**
- Real-time price monitoring
- Alert status indicators
- Quick price trend view
- Bulk operations
- Compact and expanded modes

#### MarketOverviewWidget

High-level market statistics and trends.

```typescript
interface MarketOverviewWidgetProps {
  realm: string;
  timeframe: number;
  metrics?: MarketMetrics[];
  refreshInterval?: number;
}

function MarketOverviewWidget({
  realm,
  timeframe = 24,
  metrics = ['volume', 'average_price', 'trends'],
  refreshInterval = 300000
}: MarketOverviewWidgetProps)
```

**Features:**
- Key market indicators
- Trend direction indicators
- Volume and liquidity metrics
- Realm comparison
- Auto-refresh capability

### Search and Filter Components

#### SearchBar

Advanced search component with autocomplete.

```typescript
interface SearchBarProps {
  onSearch: (query: SearchQuery) => void;
  suggestions?: SearchSuggestion[];
  placeholder?: string;
  showFilters?: boolean;
  recentSearches?: string[];
}

function SearchBar({
  onSearch,
  suggestions = [],
  placeholder = "Search items...",
  showFilters = true,
  recentSearches = []
}: SearchBarProps)
```

**Features:**
- Real-time search suggestions
- Item name and ID search
- Advanced filter panel
- Search history
- Keyboard shortcuts

#### FilterPanel

Advanced filtering sidebar component.

```typescript
interface FilterPanelProps {
  filters: AuctionFilters;
  availableFilters: FilterOptions;
  onFiltersChange: (filters: AuctionFilters) => void;
  onReset: () => void;
  collapsed?: boolean;
}

function FilterPanel({
  filters,
  availableFilters,
  onFiltersChange,
  onReset,
  collapsed = false
}: FilterPanelProps)
```

**Features:**
- Price range sliders
- Multi-select dropdowns
- Time left filters
- Seller filters
- Quality filters
- Collapsible sections

## Data Processing Components

### AuctionProcessor

Service component for processing auction data.

```typescript
class AuctionProcessor {
  async processAuctionBatch(auctions: AuctionItem[]): Promise<ProcessedAuctionData>
  async calculateMarketMetrics(auctions: AuctionItem[]): Promise<MarketAnalysis>
  async detectPriceAnomalies(auctions: AuctionItem[]): Promise<AnomalyReport>
  async generatePricePredictions(history: PriceHistory[]): Promise<PricePrediction[]>
}
```

**Methods:**
- `processAuctionBatch()`: Cleans and normalizes auction data
- `calculateMarketMetrics()`: Computes statistical metrics
- `detectPriceAnomalies()`: Identifies unusual price patterns
- `generatePricePredictions()`: Uses ML models for price forecasting

### PriceAnalyzer

Advanced price analysis service.

```typescript
class PriceAnalyzer {
  async analyzeTrend(prices: PricePoint[]): Promise<TrendAnalysis>
  async calculateVolatility(prices: PricePoint[]): Promise<VolatilityMetrics>
  async identifySupportResistance(prices: PricePoint[]): Promise<SupportResistanceLevels>
  async computeCorrelation(items: ItemId[]): Promise<CorrelationMatrix>
}
```

**Features:**
- Technical analysis indicators
- Statistical analysis
- Pattern recognition
- Correlation analysis

### AlertManager

Manages price alerts and notifications.

```typescript
class AlertManager {
  async createAlert(config: PriceAlertConfig): Promise<PriceAlert>
  async checkAlerts(auctions: AuctionItem[]): Promise<TriggeredAlert[]>
  async sendNotifications(alerts: TriggeredAlert[]): Promise<NotificationResult[]>
  async updateAlertStatus(alertId: string, status: AlertStatus): Promise<void>
}
```

**Features:**
- Alert creation and management
- Real-time alert checking
- Multi-channel notifications
- Alert performance tracking

## Backend Services

### AuctionDataService

Handles auction data retrieval and caching.

```typescript
class AuctionDataService {
  async fetchAuctions(filters: AuctionFilters): Promise<AuctionItem[]>
  async getAuctionHistory(itemId: ItemId, timeframe: number): Promise<PriceHistory>
  async getRealmData(realm: string): Promise<RealmInfo>
  async searchItems(query: string): Promise<ItemSearchResult[]>
}
```

**Features:**
- Blizzard API integration
- Data caching and optimization
- Rate limiting compliance
- Error handling and retries

### MarketAnalysisService

Performs market analysis and opportunity detection.

```typescript
class MarketAnalysisService {
  async analyzeMarket(realm: string): Promise<MarketAnalysisReport>
  async findOpportunities(criteria: OpportunityCriteria): Promise<MarketOpportunity[]>
  async calculateProfitability(opportunity: MarketOpportunity): Promise<ProfitabilityReport>
  async generateMarketReport(timeframe: number): Promise<MarketReport>
}
```

**Features:**
- Market trend analysis
- Opportunity identification
- Profitability calculations
- Report generation

### NotificationService

Handles alert notifications across multiple channels.

```typescript
class NotificationService {
  async sendWebhook(url: string, payload: any): Promise<void>
  async sendEmail(to: string, subject: string, content: string): Promise<void>
  async sendDiscordMessage(webhookUrl: string, embed: DiscordEmbed): Promise<void>
  async sendPushNotification(deviceToken: string, message: string): Promise<void>
}
```

**Features:**
- Multi-channel notification support
- Template rendering
- Delivery tracking
- Rate limiting

## Utility Components

### LoadingSpinner

Reusable loading indicator component.

```typescript
interface LoadingSpinnerProps {
  size?: 'small' | 'medium' | 'large';
  color?: string;
  text?: string;
}

function LoadingSpinner({
  size = 'medium',
  color = '#007bff',
  text = 'Loading...'
}: LoadingSpinnerProps)
```

### ErrorBoundary

React error boundary component.

```typescript
interface ErrorBoundaryProps {
  fallback?: React.ComponentType<ErrorFallbackProps>;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

function ErrorBoundary({
  fallback: Fallback,
  onError,
  children
}: React.PropsWithChildren<ErrorBoundaryProps>)
```

### Tooltip

Contextual tooltip component.

```typescript
interface TooltipProps {
  content: React.ReactNode;
  placement?: 'top' | 'right' | 'bottom' | 'left';
  trigger?: 'hover' | 'click' | 'focus';
  delay?: number;
}

function Tooltip({
  content,
  placement = 'top',
  trigger = 'hover',
  delay = 300,
  children
}: React.PropsWithChildren<TooltipProps>)
```

## Data Visualization Components

### PriceChart

Interactive price history chart.

```typescript
interface PriceChartProps {
  data: PricePoint[];
  height?: number;
  showVolume?: boolean;
  indicators?: TechnicalIndicator[];
  onRangeSelect?: (start: Date, end: Date) => void;
}

function PriceChart({
  data,
  height = 400,
  showVolume = true,
  indicators = [],
  onRangeSelect
}: PriceChartProps)
```

**Features:**
- Interactive price line/bar chart
- Volume visualization
- Technical indicators overlay
- Date range selection
- Export functionality

### Heatmap

Market heatmap showing price movements.

```typescript
interface HeatmapProps {
  data: MarketHeatmapData;
  metric: 'price_change' | 'volume' | 'volatility';
  timeframe: number;
  onCellClick?: (itemId: ItemId, realm: string) => void;
}

function Heatmap({
  data,
  metric = 'price_change',
  timeframe = 24,
  onCellClick
}: HeatmapProps)
```

**Features:**
- Color-coded market data
- Interactive cells
- Legend and scale
- Filtering options

### ComparisonChart

Side-by-side comparison of items or realms.

```typescript
interface ComparisonChartProps {
  datasets: ComparisonDataset[];
  type: 'price' | 'volume' | 'trend';
  normalize?: boolean;
  showLegend?: boolean;
}

function ComparisonChart({
  datasets,
  type = 'price',
  normalize = false,
  showLegend = true
}: ComparisonChartProps)
```

## Integration Components

### BlizzardAPIClient

Client for Blizzard's WoW API.

```typescript
class BlizzardAPIClient {
  constructor(config: BlizzardConfig)

  async getAuctions(realm: string): Promise<AuctionData>
  async getItem(itemId: ItemId): Promise<ItemInfo>
  async getRealmStatus(realm: string): Promise<RealmStatus>
  async getTokenPrices(): Promise<TokenPrice[]>
}
```

**Features:**
- OAuth2 authentication
- Automatic token refresh
- Rate limiting
- Error handling

### DatabaseService

Data persistence and retrieval service.

```typescript
class DatabaseService {
  async saveAuctions(auctions: AuctionItem[]): Promise<void>
  async getAuctions(filters: QueryFilters): Promise<AuctionItem[]>
  async savePriceHistory(history: PriceHistory): Promise<void>
  async getPriceHistory(itemId: ItemId, timeframe: number): Promise<PriceHistory>
  async saveUserData(userId: UserId, data: any): Promise<void>
}
```

**Features:**
- Connection pooling
- Query optimization
- Data validation
- Backup and recovery

## Real-time Components

### WebSocketManager

Manages WebSocket connections for real-time updates.

```typescript
class WebSocketManager {
  connect(realm: string, handlers: WebSocketHandlers): Promise<void>
  disconnect(realm: string): void
  subscribe(eventType: string, handler: (data: any) => void): void
  unsubscribe(eventType: string): void
}
```

**Features:**
- Auto-reconnection
- Event subscription management
- Connection health monitoring
- Message queuing

### RealTimeAuctionFeed

Component for displaying real-time auction updates.

```typescript
interface RealTimeAuctionFeedProps {
  realm: string;
  filters?: AuctionFilters;
  maxItems?: number;
  onAuctionUpdate?: (auction: AuctionItem) => void;
}

function RealTimeAuctionFeed({
  realm,
  filters,
  maxItems = 100,
  onAuctionUpdate
}: RealTimeAuctionFeedProps)
```

**Features:**
- Live auction stream
- Filtering and highlighting
- Scroll management
- Performance optimization

## Mobile Components

### MobileAuctionCard

Mobile-optimized auction display.

```typescript
interface MobileAuctionCardProps {
  auction: AuctionItem;
  compact?: boolean;
  swipeActions?: SwipeAction[];
}

function MobileAuctionCard({
  auction,
  compact = false,
  swipeActions = []
}: MobileAuctionCardProps)
```

**Features:**
- Touch-friendly interface
- Swipe gestures
- Collapsible details
- Quick actions

### MobileDashboard

Mobile-optimized dashboard layout.

```typescript
interface MobileDashboardProps {
  widgets: MobileWidget[];
  layout: 'tabs' | 'drawer' | 'bottom_nav';
  onNavigate?: (route: string) => void;
}

function MobileDashboard({
  widgets,
  layout = 'tabs',
  onNavigate
}: MobileDashboardProps)
```

**Features:**
- Responsive design
- Touch navigation
- Gesture support
- Performance optimization

## Testing Components

### MockAuctionProvider

Mock data provider for testing.

```typescript
class MockAuctionProvider {
  generateMockAuctions(count: number): AuctionItem[]
  generateMockPriceHistory(itemId: ItemId, days: number): PriceHistory
  simulateAuctionUpdates(interval: number): void
}
```

**Features:**
- Realistic test data generation
- Configurable scenarios
- Deterministic randomness
- Performance testing support

### ComponentTestUtils

Utilities for component testing.

```typescript
const ComponentTestUtils = {
  renderWithProviders: (component: React.ReactElement, providers?: any[]) => RenderResult,
  createMockAuction: (overrides?: Partial<AuctionItem>) => AuctionItem,
  waitForAuctionUpdate: () => Promise<void>,
  simulateUserInteraction: (element: HTMLElement, action: string) => void
}
```

**Features:**
- Test provider setup
- Mock data generation
- Interaction simulation
- Async operation waiting
```