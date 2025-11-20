# Performance Optimization Guide

Comprehensive guide for analyzing and improving frontend application performance.

## Performance Analysis Checklist

### Bundle Size Analysis

**What to Check**:
```bash
# Find build output directory
# Look for: dist/, build/, .next/, .nuxt/
```

**Key Metrics**:
- Total bundle size (target: < 200KB initial load for mobile)
- Number of chunks
- Largest modules/dependencies
- Code splitting effectiveness

**Common Issues**:
1. **Large Dependencies**
   - Location: Check `package.json` and build stats
   - Use Grep to search: `import.*from ['"]lodash['"]`
   - Recommendation: Use tree-shakeable alternatives (lodash-es, date-fns)

2. **Missing Code Splitting**
   - Check routing files for dynamic imports
   - Search for: `const Component = () => import('...')`
   - Recommendation: Implement route-based code splitting

3. **Duplicate Dependencies**
   - Multiple versions of same library
   - Recommendation: Use package manager's deduplication

### Lazy Loading Strategy

**Route-Based Lazy Loading**:

Check if routes are lazy loaded:
```
# React Router
const Home = lazy(() => import('./Home'))

# Vue Router
component: () => import('./Home.vue')

# Next.js (automatic)
# pages/about.tsx
```

**Component-Based Lazy Loading**:

For heavy components (charts, editors, modals):
```
# React
const Chart = lazy(() => import('./Chart'))

# Vue
defineAsyncComponent(() => import('./Chart.vue'))
```

**When to Use**:
- Components > 50KB
- Below-the-fold content
- Modal/drawer content
- Admin/dashboard features
- Rich text editors, data visualization

### Asset Optimization

**Image Optimization**:
- Use next-gen formats (WebP, AVIF)
- Implement lazy loading for images
- Use responsive images with srcset
- Compress images appropriately

**Font Optimization**:
- Use font-display: swap
- Preload critical fonts
- Subset fonts to required characters
- Use variable fonts when possible

**CSS Optimization**:
- Remove unused CSS (PurgeCSS)
- Critical CSS inlining
- CSS modules or scoped styles
- Minimize runtime CSS-in-JS

## Caching Strategy

### Browser Caching

**Static Assets**:
```
# Check build output file naming
# Look for: [name].[contenthash].js
```

**Cache Headers**:
- Immutable assets: `Cache-Control: public, max-age=31536000, immutable`
- HTML: `Cache-Control: no-cache`
- API responses: Appropriate cache headers

### Application-Level Caching

**Data Fetching**:

React:
```typescript
// React Query
const { data } = useQuery({
  queryKey: ['user', id],
  queryFn: fetchUser,
  staleTime: 5 * 60 * 1000, // 5 minutes
})

// SWR
const { data } = useSWR('/api/user', fetcher, {
  revalidateOnFocus: false,
})
```

Vue:
```typescript
// Pinia with cache
export const useUserStore = defineStore('user', {
  state: () => ({ cache: new Map() }),
  actions: {
    async fetchUser(id: string) {
      if (this.cache.has(id)) return this.cache.get(id)
      const user = await api.getUser(id)
      this.cache.set(id, user)
      return user
    }
  }
})
```

**Memoization**:

React:
```typescript
// Expensive calculations
const sortedData = useMemo(() =>
  data.sort((a, b) => a.value - b.value),
  [data]
)

// Callback stability
const handleClick = useCallback(() => {
  console.log(count)
}, [count])

// Component memoization
const MemoizedChild = memo(Child)
```

Vue:
```typescript
// Computed properties (automatically memoized)
const sortedData = computed(() =>
  data.value.sort((a, b) => a.value - b.value)
)
```

## Runtime Performance

### Rendering Optimization

**React Rendering Issues**:

Check for:
```typescript
# Search for potential issues
grep -r "useEffect.*\[\]" src/
grep -r "setState.*render" src/
```

Common problems:
1. **Unnecessary re-renders**
   - Missing dependencies in useEffect
   - Creating new objects/arrays in render
   - Inline function definitions in props

2. **Large component trees**
   - Components with > 100 children
   - Deeply nested components (> 10 levels)

**Vue Rendering Issues**:

1. **Reactive overhead**
   - Large arrays/objects in reactive state
   - Unnecessary reactive conversions
   - Missing `shallowRef` for large data

2. **Template inefficiency**
   - Missing `v-memo` for expensive lists
   - Inefficient `v-for` without keys
   - Computed properties recalculating unnecessarily

### List Rendering

**Virtual Scrolling**:

When to use:
- Lists with > 100 items
- Complex list items
- Infinite scroll scenarios

Libraries:
- `react-window` / `react-virtual` (React)
- `vue-virtual-scroller` (Vue)

**Efficient List Updates**:

React:
```typescript
// Good: Stable keys
{items.map(item => <Item key={item.id} {...item} />)}

// Bad: Index as key
{items.map((item, i) => <Item key={i} {...item} />)}
```

Vue:
```vue
<!-- Good: v-memo for expensive items -->
<div v-for="item in items" :key="item.id" v-memo="[item.id, item.status]">
  <ExpensiveComponent :item="item" />
</div>
```

### JavaScript Execution

**Heavy Computations**:

Strategies:
1. **Web Workers**
   - Data processing
   - Image manipulation
   - Complex calculations

2. **Debouncing/Throttling**
   - Search inputs
   - Scroll handlers
   - Resize handlers

3. **Time Slicing**
   - Break work into chunks
   - Use `requestIdleCallback`
   - React Concurrent Mode

## Network Performance

### API Request Optimization

**Request Batching**:
```typescript
// Batch multiple requests
const [user, posts, comments] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id),
])
```

**GraphQL Optimization**:
- Request only needed fields
- Use fragments for reusability
- Implement data loader pattern
- Cache query results

**REST API Optimization**:
- Use pagination
- Implement field filtering
- Compress responses (gzip/brotli)
- Use HTTP/2 multiplexing

### Resource Hints

**Preload Critical Resources**:
```html
<link rel="preload" href="/font.woff2" as="font" crossorigin>
<link rel="preload" href="/api/critical" as="fetch" crossorigin>
```

**Prefetch Future Resources**:
```html
<link rel="prefetch" href="/next-page-bundle.js">
<link rel="dns-prefetch" href="https://api.example.com">
```

**Preconnect to Required Origins**:
```html
<link rel="preconnect" href="https://cdn.example.com">
```

## Build Optimization

### Webpack/Vite Configuration

**Code Splitting**:

Webpack:
```javascript
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        priority: 10,
      },
      common: {
        minChunks: 2,
        priority: 5,
        reuseExistingChunk: true,
      },
    },
  },
}
```

Vite:
```javascript
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        'react-vendor': ['react', 'react-dom'],
        'ui-library': ['@mui/material'],
      },
    },
  },
}
```

**Tree Shaking**:
- Ensure `sideEffects` in package.json
- Use ES modules
- Avoid barrel exports for large libraries

**Compression**:
```javascript
// Enable gzip/brotli compression
compression: {
  algorithm: 'brotliCompress',
  threshold: 10240,
}
```

## Performance Monitoring

### Metrics to Track

**Core Web Vitals**:
1. **LCP (Largest Contentful Paint)** - Target: < 2.5s
   - Optimize server response time
   - Preload hero images
   - Remove render-blocking resources

2. **FID (First Input Delay)** - Target: < 100ms
   - Reduce JavaScript execution time
   - Code split and defer non-critical JS
   - Use web workers

3. **CLS (Cumulative Layout Shift)** - Target: < 0.1
   - Set image/video dimensions
   - Reserve space for dynamic content
   - Avoid inserting content above existing content

**Custom Metrics**:
```typescript
// Measure custom timings
performance.mark('feature-start')
// ... feature code
performance.mark('feature-end')
performance.measure('feature', 'feature-start', 'feature-end')
```

### Performance Testing

**Tools**:
- Lighthouse (automated audits)
- WebPageTest (real-world conditions)
- Chrome DevTools Performance panel
- Bundle analyzers (webpack-bundle-analyzer)

**Automated Testing**:
```javascript
// Lighthouse CI
{
  "ci": {
    "collect": {
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "first-contentful-paint": ["error", {"maxNumericValue": 2000}],
        "interactive": ["error", {"maxNumericValue": 3500}]
      }
    }
  }
}
```

## Performance Review Checklist

When reviewing a project's performance:

- [ ] Check bundle size and analyze largest modules
- [ ] Verify route-based code splitting is implemented
- [ ] Confirm lazy loading for heavy components
- [ ] Review image optimization strategy
- [ ] Check font loading performance
- [ ] Verify cache headers on static assets
- [ ] Review data fetching and caching strategy
- [ ] Check for unnecessary re-renders (React) or reactive overhead (Vue)
- [ ] Verify efficient list rendering (keys, virtual scrolling)
- [ ] Review API request patterns for optimization
- [ ] Check resource hints usage (preload, prefetch)
- [ ] Verify build configuration for optimization
- [ ] Check Core Web Vitals scores
- [ ] Review monitoring and performance tracking setup

## Framework-Specific Best Practices

### React Performance

**Common Optimizations**:
- Use `React.memo()` for expensive pure components
- Implement `useMemo()` for expensive calculations
- Use `useCallback()` for stable callback references
- Consider React Compiler for automatic memoization
- Use Suspense for data fetching
- Implement error boundaries

**React Server Components** (Next.js 13+):
- Move data fetching to server
- Reduce client bundle size
- Stream HTML progressively

### Vue Performance

**Common Optimizations**:
- Use `shallowRef` for large immutable data
- Implement `v-once` for static content
- Use `v-memo` for expensive list items
- Leverage computed properties for derived state
- Use `defineAsyncComponent` for code splitting
- Implement `KeepAlive` for component caching

**Nuxt Performance**:
- Use server-side rendering (SSR)
- Enable automatic code splitting
- Implement payload extraction
- Use Nuxt Image for optimization

## Performance Budget

Establish performance budgets:

```javascript
// Example performance budget
{
  "budgets": [
    {
      "resourceType": "initial",
      "maximumSizeInBytes": 200000
    },
    {
      "resourceType": "script",
      "maximumSizeInBytes": 150000
    },
    {
      "resourceType": "image",
      "maximumSizeInBytes": 100000
    }
  ]
}
```

**Monitoring Budget**:
- Fail CI builds if budget exceeded
- Track budget over time
- Set alerts for budget violations

## Quick Performance Wins

Priority optimizations with high impact:

1. **Enable Compression** - 60-80% size reduction
2. **Add Cache Headers** - Instant load for repeat visits
3. **Implement Code Splitting** - 30-50% initial bundle reduction
4. **Optimize Images** - 40-70% image size reduction
5. **Lazy Load Below Fold** - Faster initial render
6. **Preload Critical Resources** - Improved LCP
7. **Remove Unused Code** - Smaller bundle size
8. **Use Production Builds** - Optimized and minified code
