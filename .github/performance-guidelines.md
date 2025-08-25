# Performance Optimization Guidelines for React Applications

## Overview

Performance optimization in React applications involves multiple strategies: minimizing re-renders, optimizing bundle size, improving loading times, and ensuring smooth user interactions. This guide provides comprehensive performance optimization techniques.

## Core Performance Principles

### 1. Measure First, Optimize Second
**Always identify performance bottlenecks before optimizing.**
- Use React DevTools Profiler to identify slow components
- Monitor Core Web Vitals (LCP, FID, CLS) in production
- Set performance budgets and track metrics over time
- Profile with realistic data volumes and user scenarios

### 2. Optimize for User Experience
**Focus on metrics that impact user perception.**
- **Loading Performance**: Time to first meaningful content
- **Interactivity**: Response time to user inputs
- **Visual Stability**: Prevent layout shifts
- **Progressive Enhancement**: Show content as it becomes available

### 3. Progressive Enhancement
**Ensure basic functionality works, then enhance with optimizations.**
- Start with working code, then optimize critical paths
- Use performance budgets to prevent regressions
- Implement monitoring to track real-world performance

---

## Memoization Strategies

### When to Use Memoization

**Use memoization for:**
- Expensive calculations or data transformations
- Components with complex rendering logic
- Event handlers passed to multiple children
- Objects/arrays passed as props to memoized components

**Avoid memoization for:**
- Simple calculations (string concatenation, basic math)
- Components that always re-render anyway
- Functions with frequently changing dependencies
- Primitive values (already optimized by React)

### React.memo for Component Optimization

```typescript
// ✅ Effective use of React.memo
interface IUserCardProps {
  user: IUser;
  onEdit: (userId: string) => void;
  onDelete: (userId: string) => void;
  theme: 'light' | 'dark';
}

const UserCard = React.memo<IUserCardProps>(({ user, onEdit, onDelete, theme }) => {
  return (
    <div className={`user-card user-card--${theme}`}>
      <UserAvatar src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="user-actions">
        <button onClick={() => onEdit(user.id)}>Edit</button>
        <button onClick={() => onDelete(user.id)}>Delete</button>
      </div>
    </div>
  );
});

// ✅ Custom comparison for complex props
const ComplexUserCard = React.memo<IUserCardProps>(
  ({ user, onEdit, onDelete, theme }) => {
    // Component implementation
  },
  (prevProps, nextProps) => {
    // Only re-render if these specific properties change
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.updatedAt === nextProps.user.updatedAt &&
      prevProps.theme === nextProps.theme
    );
  }
);

// ❌ Don't memo simple components unnecessarily
const SimpleButton = React.memo(({ text, onClick }: { text: string; onClick: () => void }) => (
  <button onClick={onClick}>{text}</button>
)); // Unnecessary overhead for simple components
```

### useMemo for Expensive Calculations

```typescript
// ✅ Optimize expensive computations
interface IDataAnalyticsProps {
  data: IDataPoint[];
  filters: IFilters;
  sortBy: string;
  groupBy?: string;
}

const DataAnalytics: React.FC<IDataAnalyticsProps> = ({ data, filters, sortBy, groupBy }) => {
  // ✅ Memoize expensive data processing
  const processedData = useMemo(() => {
    console.log('Processing data...'); // Should only run when dependencies change
    
    let result = data
      // Apply filters
      .filter(item => {
        if (filters.category && item.category !== filters.category) return false;
        if (filters.dateRange && !isInDateRange(item.date, filters.dateRange)) return false;
        if (filters.minValue && item.value < filters.minValue) return false;
        return true;
      })
      // Apply expensive transformations
      .map(item => ({
        ...item,
        normalizedValue: normalizeValue(item.value),
        calculatedMetrics: calculateComplexMetrics(item),
        enrichedData: enrichWithExternalData(item)
      }));
    
    // Apply sorting
    result = sortData(result, sortBy);
    
    // Apply grouping if specified
    if (groupBy) {
      result = groupDataBy(result, groupBy);
    }
    
    return result;
  }, [data, filters.category, filters.dateRange, filters.minValue, sortBy, groupBy]);

  // ✅ Memoize derived statistics
  const statistics = useMemo(() => {
    const values = processedData.map(item => item.normalizedValue);
    
    return {
      total: values.reduce((sum, val) => sum + val, 0),
      average: values.length > 0 ? values.reduce((sum, val) => sum + val, 0) / values.length : 0,
      median: calculateMedian(values),
      standardDeviation: calculateStandardDeviation(values),
      percentiles: calculatePercentiles(values, [25, 50, 75, 90, 95])
    };
  }, [processedData]);

  // ✅ Memoize chart configuration
  const chartConfig = useMemo(() => ({
    type: 'line',
    data: processedData,
    options: {
      responsive: true,
      scales: {
        x: { type: 'time' },
        y: { beginAtZero: true }
      },
      plugins: {
        legend: { display: true },
        tooltip: { 
          callbacks: {
            label: (context: any) => `${context.label}: ${context.parsed.y}`
          }
        }
      }
    }
  }), [processedData]);

  return (
    <div className="data-analytics">
      <StatisticsSummary stats={statistics} />
      <DataChart config={chartConfig} />
      <DataTable data={processedData} />
    </div>
  );
};
```

### useCallback for Event Handlers

```typescript
// ✅ Optimize event handlers to prevent child re-renders
interface ITodoListProps {
  todos: ITodo[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
  onEdit: (id: string, text: string) => void;
  onReorder: (fromIndex: number, toIndex: number) => void;
}

const TodoList: React.FC<ITodoListProps> = ({ 
  todos, 
  onToggle, 
  onDelete, 
  onEdit, 
  onReorder 
}) => {
  // ✅ Memoize event handlers
  const handleToggle = useCallback((id: string) => {
    onToggle(id);
  }, [onToggle]);

  const handleDelete = useCallback((id: string) => {
    onDelete(id);
  }, [onDelete]);

  const handleEdit = useCallback((id: string, newText: string) => {
    onEdit(id, newText);
  }, [onEdit]);

  // ✅ Memoize complex event handlers
  const handleDragEnd = useCallback((result: DropResult) => {
    if (!result.destination) return;
    
    const fromIndex = result.source.index;
    const toIndex = result.destination.index;
    
    if (fromIndex !== toIndex) {
      onReorder(fromIndex, toIndex);
    }
  }, [onReorder]);

  // ✅ Memoize filtered todos
  const filteredTodos = useMemo(() => 
    todos.filter(todo => !todo.isDeleted), 
    [todos]
  );

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      <Droppable droppableId="todo-list">
        {(provided) => (
          <ul {...provided.droppableProps} ref={provided.innerRef}>
            {filteredTodos.map((todo, index) => (
              <TodoItem
                key={todo.id}
                todo={todo}
                index={index}
                onToggle={handleToggle}
                onDelete={handleDelete}
                onEdit={handleEdit}
              />
            ))}
            {provided.placeholder}
          </ul>
        )}
      </Droppable>
    </DragDropContext>
  );
};
```

## Code Splitting and Lazy Loading

### What is Code Splitting?
Code splitting breaks your application into smaller chunks that load on-demand, reducing initial bundle size and improving loading performance.

### Route-based Code Splitting

**Split by pages/routes** - Most effective approach for performance gains.


```typescript
// ✅ Basic lazy loading for routes
const HomePage = lazy(() => import('../pages/HomePage'));
const Dashboard = lazy(() => import('../pages/Dashboard'));
const UserProfile = lazy(() => import('../pages/UserProfile'));

// ✅ App with suspense fallback
const App: React.FC = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<UserProfile />} />
      </Routes>
    </Suspense>
  </Router>
);
```

### Component-based Code Splitting

**Split heavy components** that aren't always needed.

```typescript
// ✅ Lazy load heavy components conditionally
const ChartComponent = lazy(() => import('../components/Chart'));
const ReportGenerator = lazy(() => import('../components/ReportGenerator'));

const Dashboard: React.FC = ({ showCharts, showReports }) => {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {showCharts && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <ChartComponent />
        </Suspense>
      )}
      
      {showReports && (
        <Suspense fallback={<div>Loading reports...</div>}>
          <ReportGenerator />
        </Suspense>
      )}
    </div>
  );
};
```

### Library-based Code Splitting

**Split large third-party libraries** to reduce bundle size.

```typescript
// ✅ Dynamic library imports
const useChartLibrary = () => {
  const [chartLib, setChartLib] = useState(null);
  const [loading, setLoading] = useState(false);

  const loadChart = useCallback(async () => {
    if (chartLib) return chartLib;
    
    setLoading(true);
    try {
      const lib = await import('chart.js/auto');
      setChartLib(lib);
      return lib;
    } finally {
      setLoading(false);
    }
  }, [chartLib]);

  return { chartLib, loading, loadChart };
};

// Usage
const ChartWidget: React.FC = () => {
  const { chartLib, loading, loadChart } = useChartLibrary();

  useEffect(() => {
    loadChart();
  }, [loadChart]);

  if (loading) return <div>Loading chart library...</div>;
  if (!chartLib) return <div>Chart unavailable</div>;

  return <div>Chart rendered with {chartLib.Chart.version}</div>;
};
```

### Loading States and Error Handling

**Provide good user experience** during loading and errors.

```typescript
// ✅ Enhanced suspense with error boundary
const LoadingFallback: React.FC = () => (
  <div className="loading-container">
    <div className="spinner" />
    <p>Loading...</p>
  </div>
);

const ErrorFallback: React.FC<{ error: Error; retry: () => void }> = ({ error, retry }) => (
  <div className="error-container">
    <p>Something went wrong: {error.message}</p>
    <button onClick={retry}>Try again</button>
  </div>
);

const AppWithErrorHandling: React.FC = () => (
  <Router>
  <ErrorBoundary fallback={ErrorFallback}>
      <Suspense fallback={<LoadingFallback />}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </ErrorBoundary>
  </Router>
);
```

### Best Practices

**When to use code splitting:**
- ✅ Split by routes (pages) - highest impact
- ✅ Split large components (>50KB) that aren't always visible
- ✅ Split heavy third-party libraries (Chart.js, Editor, etc.)
- ✅ Split admin/power-user features from regular features

**When NOT to split:**
- ❌ Small components (<10KB)
- ❌ Components always used together
- ❌ Critical above-the-fold content


**Loading strategy:**
- Use skeleton screens for better perceived performance
- Preload routes user is likely to visit
- Show meaningful error states with retry options
- Keep fallback components lightweight

## Virtual Scrolling for Large Lists

Rendering thousands of items in a list or table can slow down your app and hurt user experience. **Virtual scrolling** (also called windowing) only renders the visible items, making large lists fast and responsive.

**When to use:**  
- Lists or tables with hundreds or thousands of items  
- Infinite scrolling feeds  
- Any UI where only a small portion of a large dataset is visible at once

### Example: Virtual List with `react-window`

```typescript
import { FixedSizeList as List } from 'react-window';

const MyVirtualList = ({ items }) => (
  <List
    height={400}           // Height of the list viewport (px)
    itemCount={items.length}
    itemSize={50}          // Height of each row (px)
    width="100%"
  >
    {({ index, style }) => (
      <div style={style}>
        {items[index].label}
      </div>
    )}
  </List>
);
```

**Key points:**
- Only the visible rows are rendered to the DOM.
- Dramatically improves performance for large lists.
- `itemSize` can be fixed or variable (see `VariableSizeList` for variable heights).

### Infinite Loading Example

```typescript
import { FixedSizeList as List } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';

const InfiniteVirtualList = ({ items, loadMore, hasNextPage }) => {
  const itemCount = hasNextPage ? items.length + 1 : items.length;

  const isItemLoaded = index => !hasNextPage || index < items.length;

  return (
    <InfiniteLoader
      isItemLoaded={isItemLoaded}
      itemCount={itemCount}
      loadMoreItems={loadMore}
    >
      {({ onItemsRendered, ref }) => (
        <List
          height={400}
          itemCount={itemCount}
          itemSize={50}
          width="100%"
          onItemsRendered={onItemsRendered}
          ref={ref}
        >
          {({ index, style }) =>
            isItemLoaded(index) ? (
              <div style={style}>{items[index].label}</div>
            ) : (
              <div style={style}>Loading...</div>
            )
          }
        </List>
      )}
    </InfiniteLoader>
  );
};
```

**Best practices:**
- Use virtual scrolling for any list with 100+ items.
- Keep each row’s rendering logic simple for best performance.
- Use a fixed height (`FixedSizeList`) if possible; use `VariableSizeList` only if row heights vary.

---

**Summary:**  
Virtual scrolling is essential for fast, smooth large lists in React. Use `react-window` for simple, efficient implementation.

## Bundle Optimization

### Tree Shaking and Dead Code Elimination

```typescript
// ✅ Prefer named imports for better tree shaking
import { debounce, throttle, pick, omit } from 'lodash';

// ❌ Avoid default imports from large libraries
import _ from 'lodash'; // Imports entire library

// ✅ Use specific imports from libraries
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

// ✅ Use tree-shakable alternatives
import { debounce } from 'lodash-es'; // ES modules version
```

### Dynamic Imports for Utilities

```typescript
// ✅ Dynamic imports for heavy utilities
const loadChartLibrary = async () => {
  const { Chart } = await import('chart.js/auto');
  return Chart;
};

const loadDateLibrary = async () => {
  const dateFns = await import('date-fns');
  return dateFns;
};

// ✅ Conditional loading based on features
const ChartComponent: React.FC<IChartProps> = ({ data, type }) => {
  const [ChartLib, setChartLib] = useState<any>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadChart = async () => {
      try {
        const Chart = await loadChartLibrary();
        setChartLib(() => Chart);
      } finally {
        setLoading(false);
      }
    };

    loadChart();
  }, []);

  if (loading) return <ChartSkeleton />;
  if (!ChartLib) return <ChartError />;

  return <ChartLib data={data} type={type} />;
};
```

### Webpack Bundle Analysis

```typescript
// webpack.config.js optimization
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          priority: 5,
          reuseExistingChunk: true
        },
        // Separate bundle for large libraries
        charts: {
          test: /[\\/]node_modules[\\/](chart\.js|d3|plotly\.js)[\\/]/,
          name: 'charts',
          chunks: 'all',
          priority: 15
        }
      }
    }
  }
};
```

## Image and Asset Optimization

Optimizing images and assets is crucial for fast load times and a smooth user experience.

### Responsive Images

**Use responsive images with lazy loading to reduce bandwidth and speed up page loads.**

```typescript
// ✅ Responsive image with lazy loading
const OptimizedImage = ({ src, alt, width, height, className }) => (
  <img
    src={src}
    srcSet={`
      ${src}?w=400 400w,
      ${src}?w=800 800w,
      ${src}?w=1200 1200w
    `}
    sizes="(max-width: 600px) 100vw, 50vw"
    alt={alt}
    width={width}
    height={height}
    loading="lazy"
    className={className}
  />
);
```

- Use the `loading="lazy"` attribute for images below the fold.
- Provide multiple image sizes with `srcSet` and `sizes` for responsive loading.
- Always set `alt` text for accessibility.

### Asset Preloading

**Preload critical images and fonts to improve perceived performance.**

```typescript
// ✅ Preload important assets in <head> (e.g., in index.html or with React Helmet)
<link rel="preload" as="image" href="/hero-image.webp" />
<link rel="preload" as="font" href="/fonts/inter-regular.woff2" type="font/woff2" crossorigin="anonymous" />
```

- Preload your logo, hero images, and key fonts.
- Use `rel="preload"` for assets needed immediately on page load.

### Best Practices

- **Compress images** (use WebP, AVIF, or optimized JPEG/PNG).
- **Use SVGs** for icons and simple graphics.
- **Serve scaled images** (don’t send 2000px images for 200px displays).
- **Leverage browser caching** for static assets.
- **Test with Lighthouse** to find unoptimized images.

---

**Summary:**  
Optimized images and assets make your app feel faster and more professional. Use responsive images, lazy loading, and preload only what’s needed for the best results.

## Performance Monitoring

Monitoring your app’s performance helps you catch slowdowns and regressions early. Focus on **Core Web Vitals** and component render times for the best user experience.

### Core Web Vitals Monitoring

Track these key metrics:
- **LCP** (Largest Contentful Paint): Loading speed
- **FID** (First Input Delay): Interactivity
- **CLS** (Cumulative Layout Shift): Visual stability
- **FCP** (First Contentful Paint): First visible content

**How to monitor:**

```typescript
// ✅ Simple Core Web Vitals logging with web-vitals library
import { getCLS, getFID, getLCP, getFCP } from 'web-vitals';

getCLS(console.log);
getFID(console.log);
getLCP(console.log);
getFCP(console.log);
```

- Send these metrics to your analytics or monitoring service.

### Best Practices

- **Monitor in production**: Always track real user metrics, not just local dev.
- **Set performance budgets**: Warn if metrics exceed targets.
- **Automate reporting**: Send metrics to analytics for ongoing tracking.
- **Use browser tools**: Chrome DevTools and Lighthouse help spot issues.

---

**Summary:**  
Regularly monitor Core Web Vitals and component render times. Use simple hooks and libraries to catch performance issues early and keep your app fast for users.

## Performance Optimization Checklist

### Rendering Optimization
- [ ] Components use React.memo where appropriate
- [ ] Expensive calculations are memoized with useMemo
- [ ] Event handlers are memoized with useCallback
- [ ] List items have stable keys
- [ ] Avoid creating objects/arrays in render
- [ ] Use production builds for deployment

### Bundle Optimization
- [ ] Code splitting implemented for routes
- [ ] Heavy components are lazy loaded
- [ ] Tree shaking is working correctly
- [ ] Bundle analyzer used to identify large dependencies
- [ ] Dead code is eliminated
- [ ] Polyfills are loaded conditionally

### Loading Performance
- [ ] Critical resources are preloaded
- [ ] Images are optimized and responsive
- [ ] Fonts are optimized and preloaded
- [ ] Service worker caches static assets
- [ ] Gzip/Brotli compression enabled
- [ ] CDN used for static assets

### Runtime Performance
- [ ] Virtual scrolling for large lists
- [ ] Debounced/throttled input handlers
- [ ] Efficient state updates
- [ ] Proper cleanup in useEffect
- [ ] Memory leaks prevented
- [ ] Long tasks are broken up or moved to workers

### Monitoring
- [ ] Core Web Vitals tracked
- [ ] Performance monitoring implemented
- [ ] Error tracking configured
- [ ] Performance budgets defined
- [ ] Regular performance audits scheduled

## Summary

Effective performance optimization requires:

1. **Measurement-driven approach**: Always measure before optimizing
2. **User-focused metrics**: Optimize for user perception, not just technical metrics
3. **Incremental improvements**: Small, consistent optimizations compound
4. **Monitoring and alerting**: Continuous performance monitoring in production
5. **Team education**: Ensure team understands performance best practices

Remember: **Premature optimization is the root of all evil**. Focus on user experience and measure the impact of optimizations.
