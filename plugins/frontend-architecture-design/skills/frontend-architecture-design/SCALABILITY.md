# Scalability Architecture Guide

Comprehensive guide for building scalable frontend architectures that grow with your team and product.

## Scalability Dimensions

### Code Scalability
- Can the codebase grow without becoming unmanageable?
- Can multiple teams work independently?
- Can new features be added without extensive refactoring?

### Team Scalability
- Can new developers onboard quickly?
- Can teams work in parallel without conflicts?
- Is there clear ownership and boundaries?

### Performance Scalability
- Does the application perform well with increasing data?
- Can it handle more users without degradation?
- Are there caching and optimization strategies?

## Plugin Architecture

### Why Plugin Architecture?

Benefits:
- Extend functionality without modifying core
- Enable third-party extensions
- Support feature toggles and experiments
- Reduce core bundle size

### Plugin System Design

**Core Plugin Interface**:
```typescript
// plugin/types.ts
interface Plugin {
  name: string
  version: string
  initialize: (context: PluginContext) => void | Promise<void>
  destroy?: () => void
}

interface PluginContext {
  app: Application
  store: Store
  router: Router
  api: ApiClient
  events: EventEmitter
}

interface PluginManifest {
  name: string
  version: string
  description: string
  dependencies?: string[]
  permissions?: string[]
}
```

**Plugin Manager**:
```typescript
// plugin/PluginManager.ts
class PluginManager {
  private plugins = new Map<string, Plugin>()
  private context: PluginContext

  constructor(context: PluginContext) {
    this.context = context
  }

  async register(plugin: Plugin): Promise<void> {
    if (this.plugins.has(plugin.name)) {
      throw new Error(`Plugin ${plugin.name} already registered`)
    }

    await plugin.initialize(this.context)
    this.plugins.set(plugin.name, plugin)
  }

  async unregister(name: string): Promise<void> {
    const plugin = this.plugins.get(name)
    if (!plugin) return

    await plugin.destroy?.()
    this.plugins.delete(name)
  }

  getPlugin(name: string): Plugin | undefined {
    return this.plugins.get(name)
  }

  list(): Plugin[] {
    return Array.from(this.plugins.values())
  }
}
```

**Extension Points**:
```typescript
// plugin/ExtensionPoint.ts
class ExtensionPoint<T> {
  private extensions: T[] = []

  register(extension: T): void {
    this.extensions.push(extension)
  }

  unregister(extension: T): void {
    const index = this.extensions.indexOf(extension)
    if (index > -1) {
      this.extensions.splice(index, 1)
    }
  }

  getAll(): T[] {
    return [...this.extensions]
  }
}

// Usage
interface MenuExtension {
  label: string
  icon: string
  onClick: () => void
  order?: number
}

const menuExtensions = new ExtensionPoint<MenuExtension>()

// Plugin registers extension
menuExtensions.register({
  label: 'Settings',
  icon: 'settings',
  onClick: () => openSettings(),
  order: 100,
})

// Core renders extensions
const Menu = () => {
  const extensions = menuExtensions.getAll().sort((a, b) =>
    (a.order ?? 0) - (b.order ?? 0)
  )

  return (
    <nav>
      {extensions.map(ext => (
        <MenuItem key={ext.label} {...ext} />
      ))}
    </nav>
  )
}
```

### Plugin Example

```typescript
// plugins/analytics/index.ts
import type { Plugin, PluginContext } from '../types'

const analyticsPlugin: Plugin = {
  name: 'analytics',
  version: '1.0.0',

  initialize(context: PluginContext) {
    // Initialize analytics SDK
    analytics.init(context.app.config.analyticsId)

    // Track route changes
    context.router.afterEach((to) => {
      analytics.page(to.path)
    })

    // Track custom events
    context.events.on('user:action', (action) => {
      analytics.track(action.name, action.properties)
    })

    // Add global method
    context.app.$analytics = analytics
  },

  destroy() {
    analytics.shutdown()
  },
}

export default analyticsPlugin
```

## Micro-Frontend Architecture

### When to Use Micro-Frontends

**Good Fit**:
- Large organization with multiple teams
- Different tech stacks needed
- Independent deployment requirements
- Legacy system integration

**Poor Fit**:
- Small team (< 5 developers)
- Simple application
- Highly interconnected features
- No deployment autonomy needed

### Micro-Frontend Approaches

**1. Build-Time Integration**:
```typescript
// Package-based approach
// Each micro-frontend is an npm package

// package.json
{
  "dependencies": {
    "@org/header": "^1.0.0",
    "@org/footer": "^1.0.0",
    "@org/products": "^1.0.0",
    "@org/cart": "^1.0.0"
  }
}

// App.tsx
import { Header } from '@org/header'
import { Footer } from '@org/footer'
import { Products } from '@org/products'
import { Cart } from '@org/cart'

const App = () => (
  <>
    <Header />
    <main>
      <Products />
      <Cart />
    </main>
    <Footer />
  </>
)
```

**2. Runtime Integration (Module Federation)**:
```typescript
// webpack.config.js (host)
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin')

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        products: 'products@http://localhost:3001/remoteEntry.js',
        cart: 'cart@http://localhost:3002/remoteEntry.js',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
}

// App.tsx
const Products = React.lazy(() => import('products/Products'))
const Cart = React.lazy(() => import('cart/Cart'))

const App = () => (
  <Suspense fallback={<Loading />}>
    <Products />
    <Cart />
  </Suspense>
)
```

**3. Server-Side Composition**:
```typescript
// Using ESI (Edge Side Includes)
// Layout template
`
<html>
  <body>
    <esi:include src="/header" />
    <main>
      <esi:include src="/products" />
    </main>
    <esi:include src="/footer" />
  </body>
</html>
`
```

### Shared State in Micro-Frontends

**Event-Based Communication**:
```typescript
// shared/events.ts
const eventBus = new EventTarget()

// Cart micro-frontend
export const addToCart = (item: CartItem) => {
  eventBus.dispatchEvent(new CustomEvent('cart:add', { detail: item }))
}

// Header micro-frontend (listens for cart updates)
eventBus.addEventListener('cart:add', (event) => {
  updateCartCount(event.detail)
})
```

**Shared State Library**:
```typescript
// shared/store.ts
import { createStore } from 'zustand/vanilla'

export const sharedStore = createStore((set) => ({
  user: null,
  cartCount: 0,
  setUser: (user) => set({ user }),
  setCartCount: (count) => set({ cartCount: count }),
}))

// Any micro-frontend can access
const { user } = sharedStore.getState()
sharedStore.subscribe((state) => {
  console.log('State changed:', state)
})
```

## Component Library Design

### Library Architecture

**Monorepo Structure**:
```
packages/
├── components/        # React/Vue components
│   ├── src/
│   │   ├── Button/
│   │   ├── Input/
│   │   └── Modal/
│   ├── package.json
│   └── tsconfig.json
├── tokens/           # Design tokens
│   ├── colors.ts
│   ├── spacing.ts
│   └── typography.ts
├── icons/            # Icon library
├── hooks/            # Shared hooks
└── utils/            # Shared utilities
```

### Scalable Component API

**Variant System**:
```typescript
// components/Button/Button.tsx
type ButtonVariant = 'primary' | 'secondary' | 'ghost' | 'danger'
type ButtonSize = 'xs' | 'sm' | 'md' | 'lg' | 'xl'

interface ButtonProps {
  variant?: ButtonVariant
  size?: ButtonSize
  fullWidth?: boolean
  loading?: boolean
  disabled?: boolean
  leftIcon?: React.ReactNode
  rightIcon?: React.ReactNode
  children: React.ReactNode
  onClick?: () => void
}

export const Button = ({
  variant = 'primary',
  size = 'md',
  fullWidth = false,
  loading = false,
  disabled = false,
  leftIcon,
  rightIcon,
  children,
  onClick,
  ...props
}: ButtonProps) => {
  return (
    <button
      className={cn(
        'btn',
        `btn-${variant}`,
        `btn-${size}`,
        fullWidth && 'btn-full',
        loading && 'btn-loading',
      )}
      disabled={disabled || loading}
      onClick={onClick}
      {...props}
    >
      {loading ? (
        <Spinner size={size} />
      ) : (
        <>
          {leftIcon && <span className="btn-icon-left">{leftIcon}</span>}
          {children}
          {rightIcon && <span className="btn-icon-right">{rightIcon}</span>}
        </>
      )}
    </button>
  )
}
```

**Compound Components**:
```typescript
// components/Menu/Menu.tsx
interface MenuContextValue {
  activeItem: string | null
  setActiveItem: (item: string) => void
}

const MenuContext = createContext<MenuContextValue>(null!)

const Menu = ({ children, defaultActive }: MenuProps) => {
  const [activeItem, setActiveItem] = useState(defaultActive)

  return (
    <MenuContext.Provider value={{ activeItem, setActiveItem }}>
      <nav className="menu">{children}</nav>
    </MenuContext.Provider>
  )
}

const MenuItem = ({ id, children, onClick }: MenuItemProps) => {
  const { activeItem, setActiveItem } = useContext(MenuContext)

  return (
    <button
      className={cn('menu-item', activeItem === id && 'active')}
      onClick={() => {
        setActiveItem(id)
        onClick?.()
      }}
    >
      {children}
    </button>
  )
}

const MenuDivider = () => <div className="menu-divider" />

// Export compound component
Menu.Item = MenuItem
Menu.Divider = MenuDivider

// Usage
<Menu defaultActive="home">
  <Menu.Item id="home">Home</Menu.Item>
  <Menu.Item id="products">Products</Menu.Item>
  <Menu.Divider />
  <Menu.Item id="settings">Settings</Menu.Item>
</Menu>
```

### Design Tokens

**Token Definition**:
```typescript
// tokens/index.ts
export const tokens = {
  colors: {
    primary: {
      50: '#eff6ff',
      100: '#dbeafe',
      500: '#3b82f6',
      600: '#2563eb',
      700: '#1d4ed8',
    },
    gray: {
      50: '#f9fafb',
      100: '#f3f4f6',
      // ...
    },
  },
  spacing: {
    0: '0',
    1: '0.25rem',
    2: '0.5rem',
    4: '1rem',
    8: '2rem',
    // ...
  },
  typography: {
    fontFamily: {
      sans: 'Inter, system-ui, sans-serif',
      mono: 'Fira Code, monospace',
    },
    fontSize: {
      xs: '0.75rem',
      sm: '0.875rem',
      base: '1rem',
      lg: '1.125rem',
      xl: '1.25rem',
    },
  },
  // ...
}
```

**CSS Custom Properties**:
```css
:root {
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --spacing-4: 1rem;
  --font-sans: Inter, system-ui, sans-serif;
}

.btn-primary {
  background: var(--color-primary-500);
}
```

## API Abstraction Layer

### Repository Pattern

**Interface Definition**:
```typescript
// repositories/types.ts
interface Repository<T, ID = string> {
  findById(id: ID): Promise<T | null>
  findAll(params?: QueryParams): Promise<T[]>
  create(data: CreateDTO<T>): Promise<T>
  update(id: ID, data: UpdateDTO<T>): Promise<T>
  delete(id: ID): Promise<void>
}

interface QueryParams {
  page?: number
  limit?: number
  sort?: string
  filter?: Record<string, any>
}
```

**Implementation**:
```typescript
// repositories/UserRepository.ts
class UserRepository implements Repository<User> {
  constructor(private api: ApiClient) {}

  async findById(id: string): Promise<User | null> {
    try {
      return await this.api.get<User>(`/users/${id}`)
    } catch (error) {
      if (error.status === 404) return null
      throw error
    }
  }

  async findAll(params?: QueryParams): Promise<User[]> {
    return this.api.get<User[]>('/users', { params })
  }

  async create(data: CreateUserDTO): Promise<User> {
    return this.api.post<User>('/users', data)
  }

  async update(id: string, data: UpdateUserDTO): Promise<User> {
    return this.api.patch<User>(`/users/${id}`, data)
  }

  async delete(id: string): Promise<void> {
    await this.api.delete(`/users/${id}`)
  }

  // Domain-specific methods
  async findByEmail(email: string): Promise<User | null> {
    const [user] = await this.findAll({ filter: { email } })
    return user ?? null
  }

  async updatePassword(id: string, password: string): Promise<void> {
    await this.api.post(`/users/${id}/password`, { password })
  }
}
```

### Service Layer

```typescript
// services/UserService.ts
class UserService {
  constructor(
    private userRepo: UserRepository,
    private authService: AuthService,
    private notificationService: NotificationService
  ) {}

  async registerUser(data: RegisterUserDTO): Promise<User> {
    // Business logic
    const existingUser = await this.userRepo.findByEmail(data.email)
    if (existingUser) {
      throw new ConflictError('Email already registered')
    }

    // Create user
    const user = await this.userRepo.create({
      ...data,
      password: await hashPassword(data.password),
    })

    // Side effects
    await this.authService.createSession(user)
    await this.notificationService.sendWelcomeEmail(user)

    return user
  }

  async updateProfile(id: string, data: UpdateProfileDTO): Promise<User> {
    const user = await this.userRepo.findById(id)
    if (!user) throw new NotFoundError('User not found')

    return this.userRepo.update(id, data)
  }
}
```

### Dependency Injection

```typescript
// di/container.ts
class Container {
  private instances = new Map<string, any>()
  private factories = new Map<string, () => any>()

  register<T>(token: string, factory: () => T): void {
    this.factories.set(token, factory)
  }

  resolve<T>(token: string): T {
    if (!this.instances.has(token)) {
      const factory = this.factories.get(token)
      if (!factory) throw new Error(`No factory for ${token}`)
      this.instances.set(token, factory())
    }
    return this.instances.get(token)
  }
}

// Setup
const container = new Container()

container.register('ApiClient', () => new ApiClient())
container.register('UserRepository', () =>
  new UserRepository(container.resolve('ApiClient'))
)
container.register('UserService', () =>
  new UserService(
    container.resolve('UserRepository'),
    container.resolve('AuthService'),
    container.resolve('NotificationService')
  )
)

// Usage
const userService = container.resolve<UserService>('UserService')
```

## Module Federation

### Federated Module Design

**Remote Configuration**:
```typescript
// products/webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'products',
      filename: 'remoteEntry.js',
      exposes: {
        './Products': './src/Products',
        './ProductDetail': './src/ProductDetail',
        './useProducts': './src/hooks/useProducts',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        '@company/design-system': { singleton: true },
      },
    }),
  ],
}
```

**Host Configuration**:
```typescript
// host/webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        products: `products@${process.env.PRODUCTS_URL}/remoteEntry.js`,
        cart: `cart@${process.env.CART_URL}/remoteEntry.js`,
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
        '@company/design-system': { singleton: true },
      },
    }),
  ],
}
```

### Error Boundaries for Federated Modules

```typescript
// ErrorBoundary.tsx
class FederatedModuleErrorBoundary extends React.Component<Props, State> {
  state = { hasError: false, error: null }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="module-error">
          <h3>Failed to load module</h3>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Retry
          </button>
        </div>
      )
    }

    return this.props.children
  }
}

// Usage
const Products = React.lazy(() => import('products/Products'))

<FederatedModuleErrorBoundary>
  <Suspense fallback={<Loading />}>
    <Products />
  </Suspense>
</FederatedModuleErrorBoundary>
```

## Scalability Patterns

### Feature Flags

```typescript
// features/flags.ts
interface FeatureFlags {
  newDashboard: boolean
  darkMode: boolean
  experimentalSearch: boolean
}

class FeatureFlagService {
  private flags: FeatureFlags

  constructor(config: FeatureFlags) {
    this.flags = config
  }

  isEnabled(flag: keyof FeatureFlags): boolean {
    return this.flags[flag] ?? false
  }

  // React hook
  useFeatureFlag = (flag: keyof FeatureFlags): boolean => {
    return useMemo(() => this.isEnabled(flag), [flag])
  }
}

// Usage
const featureFlags = new FeatureFlagService({
  newDashboard: true,
  darkMode: false,
  experimentalSearch: true,
})

// In component
const Dashboard = () => {
  const showNewDashboard = featureFlags.useFeatureFlag('newDashboard')

  return showNewDashboard ? <NewDashboard /> : <OldDashboard />
}
```

### A/B Testing Infrastructure

```typescript
// experiments/types.ts
interface Experiment {
  id: string
  name: string
  variants: Variant[]
  allocation: number // Percentage of traffic
}

interface Variant {
  id: string
  weight: number
}

// experiments/ExperimentService.ts
class ExperimentService {
  private assignments = new Map<string, string>()

  getVariant(experimentId: string, userId: string): string {
    const key = `${experimentId}:${userId}`

    if (!this.assignments.has(key)) {
      const experiment = this.getExperiment(experimentId)
      const variant = this.assignVariant(experiment, userId)
      this.assignments.set(key, variant.id)
      this.trackAssignment(experimentId, variant.id, userId)
    }

    return this.assignments.get(key)!
  }

  private assignVariant(experiment: Experiment, userId: string): Variant {
    // Deterministic assignment based on hash
    const hash = this.hash(`${experiment.id}:${userId}`)
    let cumulative = 0

    for (const variant of experiment.variants) {
      cumulative += variant.weight
      if (hash < cumulative) return variant
    }

    return experiment.variants[0]
  }
}
```

### Internationalization (i18n)

```typescript
// i18n/types.ts
interface I18nConfig {
  defaultLocale: string
  supportedLocales: string[]
  fallbackLocale: string
}

// i18n/I18nService.ts
class I18nService {
  private locale: string
  private messages: Record<string, string> = {}

  async loadMessages(locale: string): Promise<void> {
    this.messages = await import(`./locales/${locale}.json`)
    this.locale = locale
  }

  t(key: string, params?: Record<string, any>): string {
    let message = this.messages[key] ?? key

    if (params) {
      Object.entries(params).forEach(([k, v]) => {
        message = message.replace(`{${k}}`, String(v))
      })
    }

    return message
  }
}

// React hook
const useTranslation = () => {
  const i18n = useContext(I18nContext)
  return {
    t: i18n.t,
    locale: i18n.locale,
    changeLocale: i18n.changeLocale,
  }
}

// Usage
const Welcome = ({ name }: Props) => {
  const { t } = useTranslation()
  return <h1>{t('welcome.message', { name })}</h1>
}
```

## Scalability Review Checklist

When reviewing scalability:

- [ ] Can features be developed independently?
- [ ] Are there clear module boundaries?
- [ ] Is there a plugin/extension system if needed?
- [ ] Can the application support micro-frontends if needed?
- [ ] Is the component library well-designed for reuse?
- [ ] Are design tokens centralized?
- [ ] Is there a proper API abstraction layer?
- [ ] Is dependency injection implemented?
- [ ] Are feature flags supported?
- [ ] Is i18n considered?
- [ ] Can teams work in parallel without conflicts?
- [ ] Is there clear code ownership?
- [ ] Can new developers onboard easily?
- [ ] Does performance scale with data/users?
- [ ] Are there monitoring and observability tools?
