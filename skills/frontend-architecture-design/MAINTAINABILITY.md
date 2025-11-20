# Maintainability Design Guide

Comprehensive guide for analyzing and improving frontend code maintainability.

## Code Organization Principles

### Directory Structure Patterns

**Feature-Based Structure** (Recommended for medium to large projects):
```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   ├── types/
│   │   ├── utils/
│   │   └── index.ts
│   ├── user/
│   └── products/
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── types/
├── layouts/
├── pages/ or routes/
└── app/ or main.ts
```

**Benefits**:
- Clear feature boundaries
- Easy to locate related code
- Facilitates code ownership
- Scales well with team growth

**Layer-Based Structure** (For smaller projects):
```
src/
├── components/
│   ├── common/
│   ├── forms/
│   └── layouts/
├── hooks/
├── services/
├── stores/
├── utils/
├── types/
└── pages/
```

**Benefits**:
- Simple and intuitive
- Easy to understand for beginners
- Works well for small teams

### Evaluating Directory Structure

**Red Flags**:
```bash
# Search for overly large directories
# If a single directory has > 15 files, consider splitting

# Check for deep nesting (> 5 levels)
# Use Glob: src/**/**/**/**/**/*.tsx

# Look for unclear naming
# Avoid: misc/, other/, temp/, stuff/
```

**Questions to Ask**:
1. Can new developers quickly locate code?
2. Is the structure consistent across features?
3. Are there clear boundaries between modules?
4. Is shared code properly identified?
5. Does the structure support the team's workflow?

## Module Design

### Single Responsibility Principle

**Component Responsibility**:

Good - Single responsibility:
```typescript
// UserAvatar.tsx - Only renders avatar
export const UserAvatar = ({ user, size }: Props) => (
  <img src={user.avatarUrl} alt={user.name} width={size} height={size} />
)

// UserProfile.tsx - Composes multiple single-purpose components
export const UserProfile = ({ userId }: Props) => {
  const user = useUser(userId)
  return (
    <div>
      <UserAvatar user={user} size={64} />
      <UserInfo user={user} />
      <UserActions user={user} />
    </div>
  )
}
```

Bad - Multiple responsibilities:
```typescript
// UserComponent.tsx - Does too much
export const UserComponent = ({ userId }: Props) => {
  const [user, setUser] = useState()
  const [posts, setPosts] = useState()
  const [settings, setSettings] = useState()

  useEffect(() => { /* fetch user */ }, [])
  useEffect(() => { /* fetch posts */ }, [])
  useEffect(() => { /* fetch settings */ }, [])

  // 300+ lines of mixed logic...
}
```

**How to Review**:
```bash
# Find large components (> 250 lines)
# Use Read on components and count lines

# Search for multiple useState/useEffect in one component
# May indicate too many responsibilities
```

### Separation of Concerns

**Container/Presenter Pattern**:

Container (logic):
```typescript
// UserProfileContainer.tsx
export const UserProfileContainer = ({ userId }: Props) => {
  const user = useUser(userId)
  const { updateUser, deleteUser } = useUserActions()
  const navigate = useNavigate()

  const handleUpdate = async (data: UserData) => {
    await updateUser(userId, data)
  }

  const handleDelete = async () => {
    await deleteUser(userId)
    navigate('/users')
  }

  return (
    <UserProfileView
      user={user}
      onUpdate={handleUpdate}
      onDelete={handleDelete}
    />
  )
}
```

Presenter (UI):
```typescript
// UserProfileView.tsx
export const UserProfileView = ({
  user,
  onUpdate,
  onDelete,
}: Props) => (
  <div>
    <h1>{user.name}</h1>
    <UserForm user={user} onSubmit={onUpdate} />
    <Button onClick={onDelete}>Delete</Button>
  </div>
)
```

**Custom Hooks for Logic Extraction**:

```typescript
// useUserProfile.ts
export const useUserProfile = (userId: string) => {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [userId])

  const updateUser = async (data: Partial<User>) => {
    // update logic
  }

  return { user, loading, error, updateUser }
}

// Component uses the hook
const UserProfile = ({ userId }: Props) => {
  const { user, loading, error, updateUser } = useUserProfile(userId)

  if (loading) return <Spinner />
  if (error) return <Error error={error} />

  return <UserForm user={user} onSubmit={updateUser} />
}
```

**Vue Composables**:

```typescript
// useUserProfile.ts
export const useUserProfile = (userId: Ref<string>) => {
  const user = ref<User | null>(null)
  const loading = ref(true)
  const error = ref<Error | null>(null)

  const fetchUser = async () => {
    loading.value = true
    try {
      user.value = await api.getUser(userId.value)
    } catch (e) {
      error.value = e
    } finally {
      loading.value = false
    }
  }

  watchEffect(fetchUser)

  return { user, loading, error }
}
```

## Design Patterns

### Common Patterns for Frontend

**1. Factory Pattern**

When to use:
- Creating different types of objects based on conditions
- Complex object creation logic

```typescript
// ComponentFactory.ts
type ComponentType = 'button' | 'link' | 'submit'

interface ComponentFactory {
  createComponent(type: ComponentType): React.ComponentType
}

export const componentFactory: ComponentFactory = {
  createComponent(type) {
    switch (type) {
      case 'button': return Button
      case 'link': return LinkButton
      case 'submit': return SubmitButton
      default: throw new Error(`Unknown type: ${type}`)
    }
  }
}
```

**2. Strategy Pattern**

When to use:
- Multiple algorithms for same task
- Conditional logic that varies

```typescript
// SortStrategy.ts
interface SortStrategy<T> {
  sort(items: T[]): T[]
}

class DateSortStrategy implements SortStrategy<Item> {
  sort(items: Item[]) {
    return [...items].sort((a, b) => a.date.getTime() - b.date.getTime())
  }
}

class NameSortStrategy implements SortStrategy<Item> {
  sort(items: Item[]) {
    return [...items].sort((a, b) => a.name.localeCompare(b.name))
  }
}

// Usage
const useSorting = (items: Item[], strategy: SortStrategy<Item>) => {
  return useMemo(() => strategy.sort(items), [items, strategy])
}
```

**3. Observer Pattern**

When to use:
- Event handling
- State change notifications

```typescript
// EventEmitter.ts
type Listener<T = any> = (data: T) => void

class EventEmitter {
  private listeners = new Map<string, Set<Listener>>()

  on<T>(event: string, listener: Listener<T>) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set())
    }
    this.listeners.get(event)!.add(listener)

    return () => this.off(event, listener)
  }

  off(event: string, listener: Listener) {
    this.listeners.get(event)?.delete(listener)
  }

  emit<T>(event: string, data: T) {
    this.listeners.get(event)?.forEach(listener => listener(data))
  }
}

// Usage
const events = new EventEmitter()

const useUserEvents = () => {
  useEffect(() => {
    const unsubscribe = events.on('user:updated', (user) => {
      console.log('User updated:', user)
    })
    return unsubscribe
  }, [])
}
```

**4. Adapter Pattern**

When to use:
- Integrating third-party libraries
- Creating abstraction over external APIs

```typescript
// StorageAdapter.ts
interface StorageAdapter {
  get<T>(key: string): T | null
  set<T>(key: string, value: T): void
  remove(key: string): void
}

class LocalStorageAdapter implements StorageAdapter {
  get<T>(key: string): T | null {
    const item = localStorage.getItem(key)
    return item ? JSON.parse(item) : null
  }

  set<T>(key: string, value: T): void {
    localStorage.setItem(key, JSON.stringify(value))
  }

  remove(key: string): void {
    localStorage.removeItem(key)
  }
}

class MemoryStorageAdapter implements StorageAdapter {
  private storage = new Map<string, any>()

  get<T>(key: string): T | null {
    return this.storage.get(key) ?? null
  }

  set<T>(key: string, value: T): void {
    this.storage.set(key, value)
  }

  remove(key: string): void {
    this.storage.delete(key)
  }
}

// Usage - easy to swap implementations
const storage: StorageAdapter = new LocalStorageAdapter()
```

**5. Facade Pattern**

When to use:
- Simplifying complex subsystems
- Providing unified interface

```typescript
// ApiClient.ts (Facade)
class ApiClient {
  constructor(
    private http: HttpClient,
    private auth: AuthService,
    private cache: CacheService
  ) {}

  async get<T>(url: string, options?: RequestOptions): Promise<T> {
    const cacheKey = `${url}-${JSON.stringify(options)}`

    // Check cache
    const cached = this.cache.get<T>(cacheKey)
    if (cached) return cached

    // Get auth token
    const token = await this.auth.getToken()

    // Make request
    const data = await this.http.get<T>(url, {
      ...options,
      headers: { ...options?.headers, Authorization: `Bearer ${token}` }
    })

    // Cache result
    this.cache.set(cacheKey, data)

    return data
  }
}

// Simple usage
const api = new ApiClient(http, auth, cache)
const user = await api.get<User>('/users/me')
```

### Pattern Anti-patterns

**Over-abstraction**:
```typescript
// Bad - Unnecessary abstraction
interface IButtonProps { }
abstract class AbstractButton { }
class ButtonFactory { }
class ButtonBuilder { }
// Just for a simple button!

// Good - Simple and direct
const Button = ({ children, onClick }: ButtonProps) => (
  <button onClick={onClick}>{children}</button>
)
```

**Pattern Misuse**:
- Using Singleton for everything
- Factory for simple object creation
- Observer when simple callbacks suffice

## Code Reuse Strategies

### Composition over Inheritance

**Good - Composition**:
```typescript
// Composable behaviors
const withLoading = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P & { loading?: boolean }) => {
    if (props.loading) return <Spinner />
    return <Component {...props} />
  }
}

const withError = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P & { error?: Error }) => {
    if (props.error) return <ErrorMessage error={props.error} />
    return <Component {...props} />
  }
}

// Compose behaviors
const UserList = withLoading(withError(UserListBase))
```

**Bad - Deep Inheritance**:
```typescript
// Avoid deep inheritance chains
class BaseComponent extends React.Component { }
class DataComponent extends BaseComponent { }
class UserComponent extends DataComponent { }
class AdminUserComponent extends UserComponent { }
// Hard to understand and maintain
```

### Utility Functions

**Good Organization**:
```typescript
// utils/date.ts - Focused utilities
export const formatDate = (date: Date): string => { }
export const parseDate = (str: string): Date => { }
export const addDays = (date: Date, days: number): Date => { }

// utils/string.ts
export const capitalize = (str: string): string => { }
export const truncate = (str: string, length: number): string => { }

// utils/array.ts
export const groupBy = <T>(arr: T[], key: keyof T): Record<string, T[]> => { }
export const unique = <T>(arr: T[]): T[] => { }
```

**Bad Organization**:
```typescript
// utils/helpers.ts - Everything in one file
export const formatDate = () => { }
export const validateEmail = () => { }
export const debounce = () => { }
export const sortArray = () => { }
// 1000+ lines of unrelated utilities
```

### Shared Components

**Component Library Structure**:
```
shared/components/
├── Button/
│   ├── Button.tsx
│   ├── Button.test.tsx
│   ├── Button.stories.tsx
│   ├── types.ts
│   └── index.ts
├── Input/
├── Modal/
└── index.ts (barrel export)
```

**Prop Design for Reusability**:
```typescript
// Good - Flexible and composable
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  children: React.ReactNode
  onClick?: () => void
  disabled?: boolean
  className?: string
  // Allow all HTML button attributes
  [key: string]: any
}

// Bad - Too specific
interface SubmitUserFormButtonProps {
  userId: string
  formData: UserFormData
  // Hard to reuse in other contexts
}
```

## State Management

### State Organization

**Local vs Global State**:

Local state (use component state):
- UI state (open/closed, active tab)
- Form inputs
- Temporary data

Global state (use context/store):
- User authentication
- App configuration
- Shared data across routes

**State Management Patterns**:

React:
```typescript
// Context + Reducer for complex state
const UserContext = createContext<UserContextValue>(null!)

const userReducer = (state: UserState, action: UserAction): UserState => {
  switch (action.type) {
    case 'SET_USER': return { ...state, user: action.payload }
    case 'UPDATE_USER': return { ...state, user: { ...state.user, ...action.payload } }
    case 'LOGOUT': return { ...state, user: null }
    default: return state
  }
}

export const UserProvider = ({ children }: Props) => {
  const [state, dispatch] = useReducer(userReducer, initialState)

  const value = useMemo(
    () => ({ ...state, dispatch }),
    [state]
  )

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>
}
```

Vue (Pinia):
```typescript
// stores/user.ts
export const useUserStore = defineStore('user', {
  state: () => ({
    user: null as User | null,
    isAuthenticated: false,
  }),

  getters: {
    userName: (state) => state.user?.name ?? 'Guest',
  },

  actions: {
    setUser(user: User) {
      this.user = user
      this.isAuthenticated = true
    },

    logout() {
      this.user = null
      this.isAuthenticated = false
    },
  },
})
```

### State Management Review Checklist

- [ ] Is state placed at the appropriate level?
- [ ] Is global state truly global, or could it be local?
- [ ] Are actions/mutations well-defined?
- [ ] Is state normalized (no duplicate data)?
- [ ] Is derived state computed, not stored?
- [ ] Are side effects handled properly?
- [ ] Is state immutable (no direct mutations)?

## Naming Conventions

### Consistent Naming

**Components**:
```
PascalCase for components:
- UserProfile.tsx
- ProductCard.tsx
- AuthButton.tsx
```

**Hooks/Composables**:
```
camelCase with prefix:
- useUserData.ts
- useForm.ts (React)
- useProductList.ts (Vue)
```

**Utils/Helpers**:
```
camelCase:
- formatDate.ts
- validateEmail.ts
- parseUrl.ts
```

**Constants**:
```
UPPER_SNAKE_CASE:
- API_BASE_URL
- MAX_RETRY_COUNT
- DEFAULT_PAGE_SIZE
```

**Types/Interfaces**:
```
PascalCase:
- UserProfile (type)
- ApiResponse (interface)
- ProductStatus (enum)
```

### Meaningful Names

**Good Names**:
```typescript
// Descriptive and clear
const activeUsers = users.filter(u => u.isActive)
const formatCurrency = (amount: number) => `$${amount.toFixed(2)}`
const hasPermission = (user: User, permission: string) => { }
```

**Bad Names**:
```typescript
// Unclear abbreviations
const au = users.filter(u => u.isActive)
const fmt = (a: number) => `$${a.toFixed(2)}`
const chk = (u: User, p: string) => { }
```

## Documentation

### Code Comments

**When to Comment**:
- Complex algorithms
- Non-obvious business logic
- Workarounds and hacks
- Public API documentation

**When NOT to Comment**:
- Obvious code
- Redundant information
- Outdated comments

**Good Comments**:
```typescript
// Calculate discount based on user tier and purchase amount
// Tier 1: 5%, Tier 2: 10%, Tier 3: 15%
// Additional 5% for purchases > $100
const calculateDiscount = (user: User, amount: number): number => {
  let discount = user.tier * 0.05
  if (amount > 100) discount += 0.05
  return discount
}

// HACK: API returns null instead of empty array
// Remove this when API v2 is released
const items = response.items ?? []
```

**Bad Comments**:
```typescript
// Set user name
setUserName(name)

// Loop through items
for (const item of items) { }

// This function adds two numbers
const add = (a: number, b: number) => a + b
```

### JSDoc for Public APIs

```typescript
/**
 * Fetches user data from the API
 * @param userId - The unique identifier of the user
 * @param options - Optional request configuration
 * @returns Promise resolving to User object
 * @throws {ApiError} When user is not found or request fails
 * @example
 * ```ts
 * const user = await fetchUser('123')
 * console.log(user.name)
 * ```
 */
export async function fetchUser(
  userId: string,
  options?: RequestOptions
): Promise<User> {
  // implementation
}
```

## Maintainability Review Checklist

When reviewing code maintainability:

- [ ] Is the directory structure clear and consistent?
- [ ] Are components single-purpose and appropriately sized?
- [ ] Is there clear separation between logic and presentation?
- [ ] Are design patterns used appropriately?
- [ ] Is code properly abstracted without over-engineering?
- [ ] Are naming conventions consistent?
- [ ] Is state management organized logically?
- [ ] Is shared code properly identified and organized?
- [ ] Are utilities and helpers well-organized?
- [ ] Is the codebase easy to navigate?
- [ ] Would new developers understand the structure?
- [ ] Is technical debt documented and tracked?
- [ ] Are there clear patterns for common tasks?
- [ ] Is configuration centralized and manageable?
