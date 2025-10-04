# Component Library

This document provides comprehensive documentation for all React components in the WoW Auction Tracker application.

## Table of Contents

- [Layout Components](#layout-components)
- [Data Display Components](#data-display-components)
- [Form Components](#form-components)
- [Navigation Components](#navigation-components)
- [Utility Components](#utility-components)
- [Chart Components](#chart-components)
- [Modal Components](#modal-components)

## Layout Components

### AppLayout

Main application layout wrapper that provides consistent structure across all pages.

```tsx
import { AppLayout } from '@/components/layout/AppLayout';

<AppLayout
  sidebar={<Sidebar />}
  header={<Header />}
  footer={<Footer />}
>
  <main>Your page content</main>
</AppLayout>
```

**Props:**
- `sidebar?: React.ReactNode` - Sidebar component
- `header?: React.ReactNode` - Header component
- `footer?: React.ReactNode` - Footer component
- `children: React.ReactNode` - Main content
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function DashboardPage() {
  return (
    <AppLayout
      sidebar={<NavigationSidebar />}
      header={<AppHeader />}
    >
      <div className="p-6">
        <h1>Dashboard</h1>
        <DashboardContent />
      </div>
    </AppLayout>
  );
}
```

### PageContainer

Standardized page container with consistent spacing and responsive behavior.

```tsx
import { PageContainer } from '@/components/layout/PageContainer';

<PageContainer
  title="Page Title"
  subtitle="Page description"
  actions={<ActionButtons />}
>
  <PageContent />
</PageContainer>
```

**Props:**
- `title?: string` - Page title
- `subtitle?: string` - Page subtitle/description
- `actions?: React.ReactNode` - Action buttons in header
- `children: React.ReactNode` - Page content
- `loading?: boolean` - Show loading state
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function ItemsPage() {
  const [loading, setLoading] = useState(false);

  return (
    <PageContainer
      title="Items"
      subtitle="Browse and search WoW items"
      actions={
        <div className="flex gap-2">
          <Button variant="outline">Export</Button>
          <Button>Add Item</Button>
        </div>
      }
      loading={loading}
    >
      <ItemsGrid />
    </PageContainer>
  );
}
```

### Card

Flexible card component for displaying content in containers.

```tsx
import { Card } from '@/components/ui/Card';

<Card
  title="Card Title"
  subtitle="Card description"
  actions={<CardActions />}
  className="w-full"
>
  <CardContent />
</Card>
```

**Props:**
- `title?: string` - Card title
- `subtitle?: string` - Card subtitle
- `actions?: React.ReactNode` - Action buttons
- `children: React.ReactNode` - Card content
- `className?: string` - Additional CSS classes
- `hover?: boolean` - Enable hover effects
- `clickable?: boolean` - Make card clickable

**Example:**
```tsx
function ItemCard({ item }) {
  return (
    <Card
      title={item.name}
      subtitle={`Level ${item.level} ${item.quality}`}
      actions={
        <Button size="sm" variant="outline">
          View Details
        </Button>
      }
      hover
      clickable
    >
      <div className="flex items-center gap-4">
        <img src={item.icon} alt={item.name} className="w-12 h-12" />
        <div>
          <p className="text-sm text-gray-600">{item.category}</p>
          <p className="font-medium">{item.description}</p>
        </div>
      </div>
    </Card>
  );
}
```

## Data Display Components

### DataTable

Advanced data table with sorting, filtering, and pagination.

```tsx
import { DataTable } from '@/components/data/DataTable';

<DataTable
  columns={columns}
  data={data}
  loading={loading}
  pagination={{
    page: 1,
    limit: 50,
    total: 1000,
    onPageChange: handlePageChange
  }}
  sorting={{
    field: 'name',
    direction: 'asc',
    onSortChange: handleSortChange
  }}
  filtering={{
    filters: activeFilters,
    onFilterChange: handleFilterChange
  }}
/>
```

**Props:**
- `columns: ColumnDef[]` - Column definitions
- `data: any[]` - Table data
- `loading?: boolean` - Loading state
- `pagination?: PaginationConfig` - Pagination configuration
- `sorting?: SortingConfig` - Sorting configuration
- `filtering?: FilteringConfig` - Filtering configuration
- `className?: string` - Additional CSS classes

**Example:**
```tsx
const columns: ColumnDef[] = [
  {
    key: 'name',
    title: 'Item Name',
    sortable: true,
    render: (item) => (
      <div className="flex items-center gap-2">
        <img src={item.icon} alt="" className="w-6 h-6" />
        <span>{item.name}</span>
      </div>
    )
  },
  {
    key: 'price',
    title: 'Price',
    sortable: true,
    render: (item) => formatGold(item.price)
  },
  {
    key: 'timeLeft',
    title: 'Time Left',
    render: (item) => formatTimeLeft(item.timeLeft)
  }
];

function AuctionsTable({ auctions, loading }) {
  return (
    <DataTable
      columns={columns}
      data={auctions}
      loading={loading}
      pagination={{
        page: 1,
        limit: 50,
        total: auctions.length,
        onPageChange: (page) => console.log('Page changed:', page)
      }}
    />
  );
}
```

### ItemCard

Specialized card component for displaying WoW items.

```tsx
import { ItemCard } from '@/components/items/ItemCard';

<ItemCard
  item={item}
  showPrice={true}
  showStats={true}
  onClick={handleItemClick}
  className="w-64"
/>
```

**Props:**
- `item: Item` - Item data object
- `showPrice?: boolean` - Show current price
- `showStats?: boolean` - Show item statistics
- `onClick?: (item: Item) => void` - Click handler
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function ItemGrid({ items }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {items.map(item => (
        <ItemCard
          key={item.id}
          item={item}
          showPrice
          showStats
          onClick={(item) => router.push(`/items/${item.id}`)}
        />
      ))}
    </div>
  );
}
```

### PriceDisplay

Component for displaying gold prices with proper formatting.

```tsx
import { PriceDisplay } from '@/components/ui/PriceDisplay';

<PriceDisplay
  amount={50000}
  showChange={true}
  change={0.05}
  size="lg"
  className="font-bold"
/>
```

**Props:**
- `amount: number` - Price amount in copper
- `showChange?: boolean` - Show price change indicator
- `change?: number` - Price change percentage
- `size?: 'sm' | 'md' | 'lg'` - Display size
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function AuctionRow({ auction }) {
  return (
    <div className="flex items-center justify-between">
      <span>{auction.itemName}</span>
      <PriceDisplay
        amount={auction.price}
        showChange
        change={auction.priceChange}
        size="md"
      />
    </div>
  );
}
```

## Form Components

### SearchInput

Advanced search input with suggestions and filters.

```tsx
import { SearchInput } from '@/components/forms/SearchInput';

<SearchInput
  value={searchTerm}
  onChange={setSearchTerm}
  placeholder="Search items..."
  suggestions={suggestions}
  filters={filters}
  onFilterChange={handleFilterChange}
  loading={loading}
/>
```

**Props:**
- `value: string` - Current search value
- `onChange: (value: string) => void` - Value change handler
- `placeholder?: string` - Placeholder text
- `suggestions?: string[]` - Search suggestions
- `filters?: FilterOption[]` - Available filters
- `onFilterChange?: (filters: FilterOption[]) => void` - Filter change handler
- `loading?: boolean` - Loading state
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function ItemSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [filters, setFilters] = useState([]);
  const [suggestions, setSuggestions] = useState([]);

  const filterOptions = [
    { key: 'quality', label: 'Quality', options: ['Epic', 'Legendary'] },
    { key: 'category', label: 'Category', options: ['Weapon', 'Armor'] },
    { key: 'level', label: 'Level', type: 'range', min: 1, max: 60 }
  ];

  return (
    <SearchInput
      value={searchTerm}
      onChange={setSearchTerm}
      placeholder="Search for items..."
      suggestions={suggestions}
      filters={filterOptions}
      onFilterChange={setFilters}
    />
  );
}
```

### FilterPanel

Collapsible panel for advanced filtering options.

```tsx
import { FilterPanel } from '@/components/forms/FilterPanel';

<FilterPanel
  filters={filters}
  onFiltersChange={setFilters}
  onReset={handleReset}
  className="w-64"
/>
```

**Props:**
- `filters: FilterConfig[]` - Filter configurations
- `onFiltersChange: (filters: FilterConfig[]) => void` - Filter change handler
- `onReset?: () => void` - Reset handler
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function AuctionFilters() {
  const [filters, setFilters] = useState([]);

  const filterConfigs = [
    {
      key: 'priceRange',
      type: 'range',
      label: 'Price Range',
      min: 0,
      max: 1000000
    },
    {
      key: 'quality',
      type: 'select',
      label: 'Quality',
      options: [
        { value: 'poor', label: 'Poor' },
        { value: 'common', label: 'Common' },
        { value: 'uncommon', label: 'Uncommon' },
        { value: 'rare', label: 'Rare' },
        { value: 'epic', label: 'Epic' },
        { value: 'legendary', label: 'Legendary' }
      ]
    }
  ];

  return (
    <FilterPanel
      filters={filterConfigs}
      onFiltersChange={setFilters}
      onReset={() => setFilters([])}
    />
  );
}
```

### FormField

Reusable form field component with validation and error handling.

```tsx
import { FormField } from '@/components/forms/FormField';

<FormField
  label="Item Name"
  name="itemName"
  type="text"
  value={value}
  onChange={setValue}
  error={error}
  required
  helperText="Enter the item name"
/>
```

**Props:**
- `label: string` - Field label
- `name: string` - Field name
- `type?: string` - Input type
- `value: any` - Field value
- `onChange: (value: any) => void` - Change handler
- `error?: string` - Error message
- `required?: boolean` - Required field
- `helperText?: string` - Helper text
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function CreateAlertForm() {
  const [formData, setFormData] = useState({
    name: '',
    threshold: 0,
    itemId: null
  });
  const [errors, setErrors] = useState({});

  return (
    <form className="space-y-4">
      <FormField
        label="Alert Name"
        name="name"
        value={formData.name}
        onChange={(value) => setFormData({ ...formData, name: value })}
        error={errors.name}
        required
      />
      
      <FormField
        label="Price Threshold"
        name="threshold"
        type="number"
        value={formData.threshold}
        onChange={(value) => setFormData({ ...formData, threshold: value })}
        error={errors.threshold}
        required
        helperText="Price in copper"
      />
    </form>
  );
}
```

## Navigation Components

### NavigationSidebar

Main navigation sidebar with collapsible sections.

```tsx
import { NavigationSidebar } from '@/components/navigation/NavigationSidebar';

<NavigationSidebar
  items={navigationItems}
  activeItem={activeItem}
  onItemClick={handleItemClick}
  collapsed={collapsed}
  onToggleCollapse={setCollapsed}
/>
```

**Props:**
- `items: NavItem[]` - Navigation items
- `activeItem?: string` - Currently active item
- `onItemClick: (item: NavItem) => void` - Item click handler
- `collapsed?: boolean` - Collapsed state
- `onToggleCollapse?: () => void` - Toggle collapse handler
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function AppSidebar() {
  const [collapsed, setCollapsed] = useState(false);
  const router = useRouter();

  const navigationItems = [
    {
      id: 'dashboard',
      label: 'Dashboard',
      icon: 'dashboard',
      path: '/dashboard'
    },
    {
      id: 'items',
      label: 'Items',
      icon: 'items',
      path: '/items',
      children: [
        { id: 'all-items', label: 'All Items', path: '/items' },
        { id: 'my-items', label: 'My Items', path: '/items/my' }
      ]
    },
    {
      id: 'auctions',
      label: 'Auctions',
      icon: 'auctions',
      path: '/auctions'
    }
  ];

  return (
    <NavigationSidebar
      items={navigationItems}
      activeItem={router.pathname}
      onItemClick={(item) => router.push(item.path)}
      collapsed={collapsed}
      onToggleCollapse={() => setCollapsed(!collapsed)}
    />
  );
}
```

### Breadcrumbs

Breadcrumb navigation component.

```tsx
import { Breadcrumbs } from '@/components/navigation/Breadcrumbs';

<Breadcrumbs
  items={breadcrumbItems}
  separator="/"
  className="mb-4"
/>
```

**Props:**
- `items: BreadcrumbItem[]` - Breadcrumb items
- `separator?: string` - Separator character
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function ItemDetailPage({ item }) {
  const breadcrumbItems = [
    { label: 'Home', path: '/' },
    { label: 'Items', path: '/items' },
    { label: item.name, path: `/items/${item.id}` }
  ];

  return (
    <div>
      <Breadcrumbs items={breadcrumbItems} />
      <ItemDetails item={item} />
    </div>
  );
}
```

### Pagination

Pagination component for data tables and lists.

```tsx
import { Pagination } from '@/components/navigation/Pagination';

<Pagination
  currentPage={currentPage}
  totalPages={totalPages}
  onPageChange={setCurrentPage}
  showSizeChanger={true}
  pageSize={pageSize}
  onPageSizeChange={setPageSize}
/>
```

**Props:**
- `currentPage: number` - Current page number
- `totalPages: number` - Total number of pages
- `onPageChange: (page: number) => void` - Page change handler
- `showSizeChanger?: boolean` - Show page size selector
- `pageSize?: number` - Current page size
- `onPageSizeChange?: (size: number) => void` - Page size change handler
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function AuctionsList({ auctions, pagination }) {
  return (
    <div>
      <AuctionsTable data={auctions} />
      <Pagination
        currentPage={pagination.page}
        totalPages={pagination.totalPages}
        onPageChange={(page) => fetchAuctions(page)}
        showSizeChanger
        pageSize={pagination.limit}
        onPageSizeChange={(size) => fetchAuctions(1, size)}
      />
    </div>
  );
}
```

## Utility Components

### LoadingSpinner

Loading spinner component with customizable size and color.

```tsx
import { LoadingSpinner } from '@/components/ui/LoadingSpinner';

<LoadingSpinner
  size="lg"
  color="primary"
  text="Loading items..."
/>
```

**Props:**
- `size?: 'sm' | 'md' | 'lg'` - Spinner size
- `color?: 'primary' | 'secondary' | 'white'` - Spinner color
- `text?: string` - Loading text
- `className?: string` - Additional CSS classes

### ErrorBoundary

Error boundary component for catching and displaying errors.

```tsx
import { ErrorBoundary } from '@/components/ui/ErrorBoundary';

<ErrorBoundary
  fallback={<ErrorFallback />}
  onError={handleError}
>
  <ComponentThatMightError />
</ErrorBoundary>
```

**Props:**
- `fallback?: React.ComponentType<ErrorFallbackProps>` - Error fallback component
- `onError?: (error: Error, errorInfo: ErrorInfo) => void` - Error handler
- `children: React.ReactNode` - Child components

### Tooltip

Tooltip component for displaying additional information.

```tsx
import { Tooltip } from '@/components/ui/Tooltip';

<Tooltip
  content="This is a tooltip"
  placement="top"
  trigger="hover"
>
  <Button>Hover me</Button>
</Tooltip>
```

**Props:**
- `content: React.ReactNode` - Tooltip content
- `placement?: 'top' | 'bottom' | 'left' | 'right'` - Tooltip placement
- `trigger?: 'hover' | 'click' | 'focus'` - Trigger event
- `children: React.ReactNode` - Trigger element
- `className?: string` - Additional CSS classes

## Chart Components

### PriceChart

Interactive price history chart component.

```tsx
import { PriceChart } from '@/components/charts/PriceChart';

<PriceChart
  data={priceData}
  height={400}
  showVolume={true}
  showMovingAverage={true}
  onDataPointClick={handleDataPointClick}
/>
```

**Props:**
- `data: PriceDataPoint[]` - Price data points
- `height?: number` - Chart height
- `showVolume?: boolean` - Show volume bars
- `showMovingAverage?: boolean` - Show moving average line
- `onDataPointClick?: (point: PriceDataPoint) => void` - Data point click handler
- `className?: string` - Additional CSS classes

**Example:**
```tsx
function ItemPriceHistory({ itemId }) {
  const [priceData, setPriceData] = useState([]);

  useEffect(() => {
    fetchPriceHistory(itemId).then(setPriceData);
  }, [itemId]);

  return (
    <PriceChart
      data={priceData}
      height={300}
      showVolume
      showMovingAverage
      onDataPointClick={(point) => {
        console.log('Clicked:', point);
      }}
    />
  );
}
```

### MarketTrendChart

Chart component for displaying market trends.

```tsx
import { MarketTrendChart } from '@/components/charts/MarketTrendChart';

<MarketTrendChart
  data={trendData}
  type="line"
  height={300}
  showLegend={true}
  colors={['#3B82F6', '#EF4444', '#10B981']}
/>
```

**Props:**
- `data: TrendData[]` - Trend data
- `type?: 'line' | 'bar' | 'area'` - Chart type
- `height?: number` - Chart height
- `showLegend?: boolean` - Show legend
- `colors?: string[]` - Chart colors
- `className?: string` - Additional CSS classes

## Modal Components

### Modal

Base modal component with customizable content and actions.

```tsx
import { Modal } from '@/components/ui/Modal';

<Modal
  isOpen={isOpen}
  onClose={handleClose}
  title="Modal Title"
  size="md"
>
  <ModalContent />
  <ModalActions>
    <Button variant="outline" onClick={handleClose}>Cancel</Button>
    <Button onClick={handleConfirm}>Confirm</Button>
  </ModalActions>
</Modal>
```

**Props:**
- `isOpen: boolean` - Modal open state
- `onClose: () => void` - Close handler
- `title?: string` - Modal title
- `size?: 'sm' | 'md' | 'lg' | 'xl'` - Modal size
- `children: React.ReactNode` - Modal content
- `className?: string` - Additional CSS classes

### ConfirmDialog

Confirmation dialog component.

```tsx
import { ConfirmDialog } from '@/components/ui/ConfirmDialog';

<ConfirmDialog
  isOpen={isOpen}
  onClose={handleClose}
  onConfirm={handleConfirm}
  title="Confirm Action"
  message="Are you sure you want to delete this item?"
  confirmText="Delete"
  cancelText="Cancel"
  variant="danger"
/>
```

**Props:**
- `isOpen: boolean` - Dialog open state
- `onClose: () => void` - Close handler
- `onConfirm: () => void` - Confirm handler
- `title: string` - Dialog title
- `message: string` - Confirmation message
- `confirmText?: string` - Confirm button text
- `cancelText?: string` - Cancel button text
- `variant?: 'default' | 'danger' | 'warning'` - Dialog variant

## Usage Examples

### Complete Item Search Page

```tsx
import React, { useState, useEffect } from 'react';
import {
  PageContainer,
  SearchInput,
  FilterPanel,
  DataTable,
  ItemCard,
  LoadingSpinner
} from '@/components';

function ItemsPage() {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  const [filters, setFilters] = useState([]);
  const [viewMode, setViewMode] = useState('table'); // 'table' or 'grid'

  const columns = [
    {
      key: 'name',
      title: 'Name',
      sortable: true,
      render: (item) => (
        <div className="flex items-center gap-2">
          <img src={item.icon} alt="" className="w-6 h-6" />
          <span>{item.name}</span>
        </div>
      )
    },
    {
      key: 'quality',
      title: 'Quality',
      render: (item) => (
        <span className={`quality-${item.quality}`}>
          {item.quality}
        </span>
      )
    },
    {
      key: 'level',
      title: 'Level',
      sortable: true
    }
  ];

  const filterConfigs = [
    {
      key: 'quality',
      type: 'select',
      label: 'Quality',
      options: [
        { value: 'poor', label: 'Poor' },
        { value: 'common', label: 'Common' },
        { value: 'uncommon', label: 'Uncommon' },
        { value: 'rare', label: 'Rare' },
        { value: 'epic', label: 'Epic' },
        { value: 'legendary', label: 'Legendary' }
      ]
    },
    {
      key: 'levelRange',
      type: 'range',
      label: 'Level Range',
      min: 1,
      max: 60
    }
  ];

  const handleSearch = async () => {
    setLoading(true);
    try {
      const results = await searchItems(searchTerm, filters);
      setItems(results);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (searchTerm || filters.length > 0) {
      handleSearch();
    }
  }, [searchTerm, filters]);

  return (
    <PageContainer
      title="Items"
      subtitle="Search and browse WoW items"
      actions={
        <div className="flex gap-2">
          <Button
            variant={viewMode === 'table' ? 'default' : 'outline'}
            onClick={() => setViewMode('table')}
          >
            Table
          </Button>
          <Button
            variant={viewMode === 'grid' ? 'default' : 'outline'}
            onClick={() => setViewMode('grid')}
          >
            Grid
          </Button>
        </div>
      }
    >
      <div className="flex gap-4">
        <div className="w-64">
          <FilterPanel
            filters={filterConfigs}
            onFiltersChange={setFilters}
            onReset={() => setFilters([])}
          />
        </div>
        
        <div className="flex-1">
          <SearchInput
            value={searchTerm}
            onChange={setSearchTerm}
            placeholder="Search items..."
            loading={loading}
          />
          
          {loading ? (
            <LoadingSpinner text="Searching items..." />
          ) : viewMode === 'table' ? (
            <DataTable
              columns={columns}
              data={items}
              loading={loading}
            />
          ) : (
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 mt-4">
              {items.map(item => (
                <ItemCard
                  key={item.id}
                  item={item}
                  showPrice
                  showStats
                />
              ))}
            </div>
          )}
        </div>
      </div>
    </PageContainer>
  );
}
```

### Alert Management Component

```tsx
import React, { useState } from 'react';
import {
  Card,
  Button,
  Modal,
  FormField,
  ConfirmDialog,
  DataTable
} from '@/components';

function AlertManagement() {
  const [alerts, setAlerts] = useState([]);
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);
  const [isDeleteDialogOpen, setIsDeleteDialogOpen] = useState(false);
  const [selectedAlert, setSelectedAlert] = useState(null);
  const [newAlert, setNewAlert] = useState({
    name: '',
    itemId: '',
    threshold: 0,
    condition: 'below'
  });

  const columns = [
    {
      key: 'name',
      title: 'Alert Name',
      sortable: true
    },
    {
      key: 'itemName',
      title: 'Item',
      render: (alert) => (
        <div className="flex items-center gap-2">
          <img src={alert.itemIcon} alt="" className="w-6 h-6" />
          <span>{alert.itemName}</span>
        </div>
      )
    },
    {
      key: 'threshold',
      title: 'Threshold',
      render: (alert) => formatGold(alert.threshold)
    },
    {
      key: 'status',
      title: 'Status',
      render: (alert) => (
        <span className={`status-${alert.status}`}>
          {alert.status}
        </span>
      )
    },
    {
      key: 'actions',
      title: 'Actions',
      render: (alert) => (
        <div className="flex gap-2">
          <Button
            size="sm"
            variant="outline"
            onClick={() => handleEdit(alert)}
          >
            Edit
          </Button>
          <Button
            size="sm"
            variant="danger"
            onClick={() => handleDelete(alert)}
          >
            Delete
          </Button>
        </div>
      )
    }
  ];

  const handleCreateAlert = async () => {
    try {
      const alert = await createAlert(newAlert);
      setAlerts([...alerts, alert]);
      setIsCreateModalOpen(false);
      setNewAlert({ name: '', itemId: '', threshold: 0, condition: 'below' });
    } catch (error) {
      console.error('Failed to create alert:', error);
    }
  };

  const handleDelete = (alert) => {
    setSelectedAlert(alert);
    setIsDeleteDialogOpen(true);
  };

  const confirmDelete = async () => {
    try {
      await deleteAlert(selectedAlert.id);
      setAlerts(alerts.filter(a => a.id !== selectedAlert.id));
      setIsDeleteDialogOpen(false);
      setSelectedAlert(null);
    } catch (error) {
      console.error('Failed to delete alert:', error);
    }
  };

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">Price Alerts</h2>
        <Button onClick={() => setIsCreateModalOpen(true)}>
          Create Alert
        </Button>
      </div>

      <DataTable
        columns={columns}
        data={alerts}
        loading={false}
      />

      <Modal
        isOpen={isCreateModalOpen}
        onClose={() => setIsCreateModalOpen(false)}
        title="Create Price Alert"
        size="md"
      >
        <div className="space-y-4">
          <FormField
            label="Alert Name"
            name="name"
            value={newAlert.name}
            onChange={(value) => setNewAlert({ ...newAlert, name: value })}
            required
          />
          
          <FormField
            label="Item ID"
            name="itemId"
            type="number"
            value={newAlert.itemId}
            onChange={(value) => setNewAlert({ ...newAlert, itemId: value })}
            required
          />
          
          <FormField
            label="Price Threshold"
            name="threshold"
            type="number"
            value={newAlert.threshold}
            onChange={(value) => setNewAlert({ ...newAlert, threshold: value })}
            required
            helperText="Price in copper"
          />
        </div>
        
        <div className="flex justify-end gap-2 mt-6">
          <Button
            variant="outline"
            onClick={() => setIsCreateModalOpen(false)}
          >
            Cancel
          </Button>
          <Button onClick={handleCreateAlert}>
            Create Alert
          </Button>
        </div>
      </Modal>

      <ConfirmDialog
        isOpen={isDeleteDialogOpen}
        onClose={() => setIsDeleteDialogOpen(false)}
        onConfirm={confirmDelete}
        title="Delete Alert"
        message={`Are you sure you want to delete the alert "${selectedAlert?.name}"?`}
        confirmText="Delete"
        cancelText="Cancel"
        variant="danger"
      />
    </div>
  );
}
```

## Best Practices

### Component Design

1. **Single Responsibility**: Each component should have a single, well-defined purpose
2. **Composition over Inheritance**: Use composition to build complex components
3. **Props Interface**: Define clear, typed interfaces for component props
4. **Default Props**: Provide sensible defaults for optional props
5. **Error Boundaries**: Wrap components that might error in error boundaries

### Performance

1. **Memoization**: Use `React.memo` for expensive components
2. **Lazy Loading**: Implement lazy loading for large component trees
3. **Virtual Scrolling**: Use virtual scrolling for large lists
4. **Debouncing**: Debounce search inputs and API calls

### Accessibility

1. **ARIA Labels**: Provide proper ARIA labels and descriptions
2. **Keyboard Navigation**: Ensure all interactive elements are keyboard accessible
3. **Focus Management**: Manage focus properly in modals and dynamic content
4. **Color Contrast**: Ensure sufficient color contrast for text and UI elements

### Testing

1. **Unit Tests**: Write unit tests for component logic
2. **Integration Tests**: Test component interactions
3. **Visual Regression**: Use visual regression testing for UI components
4. **Accessibility Tests**: Include accessibility testing in your test suite