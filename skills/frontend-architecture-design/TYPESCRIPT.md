# TypeScript Architecture Practices

Comprehensive guide for TypeScript configuration, type system design, and type-safe architectural patterns.

## TypeScript Configuration

### Recommended tsconfig.json

**Strict Configuration**:
```json
{
  "compilerOptions": {
    // Language & Environment
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",

    // Strict Type Checking
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,

    // Additional Checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,

    // Emit
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": true,
    "importHelpers": true,

    // Interop
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,

    // Path Aliases
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@utils/*": ["src/utils/*"],
      "@types/*": ["src/types/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Configuration Levels

**Level 1 - Basic** (for legacy or quick projects):
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

**Level 2 - Recommended** (for most projects):
```json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

**Level 3 - Maximum** (for critical applications):
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noPropertyAccessFromIndexSignature": true
  }
}
```

## Type System Design

### Domain Types

**Entity Types**:
```typescript
// types/user.ts
export interface User {
  id: UserId
  email: Email
  name: string
  role: UserRole
  createdAt: Date
  updatedAt: Date
}

// Branded types for type safety
export type UserId = string & { readonly brand: unique symbol }
export type Email = string & { readonly brand: unique symbol }

// Enums vs Union types
export type UserRole = 'admin' | 'user' | 'guest'

// Factory functions for branded types
export const createUserId = (id: string): UserId => id as UserId
export const createEmail = (email: string): Email => {
  if (!isValidEmail(email)) {
    throw new Error(`Invalid email: ${email}`)
  }
  return email as Email
}
```

**DTO Types**:
```typescript
// types/dto.ts

// Create DTO - omit auto-generated fields
export type CreateUserDTO = Omit<User, 'id' | 'createdAt' | 'updatedAt'>

// Update DTO - all fields optional except id
export type UpdateUserDTO = Partial<Omit<User, 'id'>> & { id: UserId }

// Response DTOs
export interface UserResponse {
  user: User
  token?: string
}

export interface PaginatedResponse<T> {
  items: T[]
  total: number
  page: number
  pageSize: number
  hasMore: boolean
}
```

### Utility Types

**Built-in Utility Types**:
```typescript
// Partial - all properties optional
type PartialUser = Partial<User>

// Required - all properties required
type RequiredUser = Required<User>

// Pick - select specific properties
type UserCredentials = Pick<User, 'email' | 'name'>

// Omit - exclude specific properties
type UserWithoutId = Omit<User, 'id'>

// Record - object with specific key and value types
type UserPermissions = Record<string, boolean>

// Extract - extract union members
type AdminRole = Extract<UserRole, 'admin'>

// Exclude - remove union members
type NonAdminRole = Exclude<UserRole, 'admin'>

// NonNullable - remove null and undefined
type RequiredEmail = NonNullable<Email | null>

// ReturnType - get function return type
type FetchUserReturn = ReturnType<typeof fetchUser>

// Parameters - get function parameter types
type FetchUserParams = Parameters<typeof fetchUser>

// Awaited - unwrap Promise type
type UserData = Awaited<ReturnType<typeof fetchUser>>
```

**Custom Utility Types**:
```typescript
// types/utils.ts

// Deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

// Deep readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P]
}

// Make specific fields required
type RequireFields<T, K extends keyof T> = T & Required<Pick<T, K>>

// Make specific fields optional
type OptionalFields<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>

// Nullable type
type Nullable<T> = T | null

// Maybe type (for optional chaining)
type Maybe<T> = T | null | undefined

// Dictionary type
type Dictionary<T> = Record<string, T>

// Value of object type
type ValueOf<T> = T[keyof T]

// Tuple to union
type TupleToUnion<T extends readonly any[]> = T[number]

// Function type
type AnyFunction = (...args: any[]) => any

// Async function type
type AsyncFunction<T> = (...args: any[]) => Promise<T>
```

### Generic Constraints

```typescript
// Constrain to specific types
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b }
}

// Constrain to have specific property
function getId<T extends { id: string }>(item: T): string {
  return item.id
}

// Constrain to array
function first<T extends any[]>(arr: T): T[0] {
  return arr[0]
}

// Conditional types
type IsArray<T> = T extends any[] ? true : false
type UnwrapArray<T> = T extends (infer U)[] ? U : T

// Mapped types with constraints
type ReadonlyDeep<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? T[K] extends Function
      ? T[K]
      : ReadonlyDeep<T[K]>
    : T[K]
}
```

## Component Typing

### React Components

**Functional Component Props**:
```typescript
// Basic props
interface ButtonProps {
  children: React.ReactNode
  onClick?: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
}

export const Button = ({
  children,
  onClick,
  disabled = false,
  variant = 'primary',
  size = 'md',
}: ButtonProps) => {
  // implementation
}

// Props with HTML attributes
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string
  error?: string
}

export const Input = ({ label, error, ...props }: InputProps) => {
  return (
    <div>
      <label>{label}</label>
      <input {...props} />
      {error && <span className="error">{error}</span>}
    </div>
  )
}

// Polymorphic component
type PolymorphicProps<E extends React.ElementType> = {
  as?: E
  children: React.ReactNode
} & Omit<React.ComponentPropsWithoutRef<E>, 'as' | 'children'>

export const Box = <E extends React.ElementType = 'div'>({
  as,
  children,
  ...props
}: PolymorphicProps<E>) => {
  const Component = as || 'div'
  return <Component {...props}>{children}</Component>
}

// Usage
<Box>div by default</Box>
<Box as="section">section element</Box>
<Box as="a" href="#">link element</Box>
```

**Component with Generics**:
```typescript
interface ListProps<T> {
  items: T[]
  renderItem: (item: T, index: number) => React.ReactNode
  keyExtractor: (item: T) => string
  emptyMessage?: string
}

export const List = <T,>({
  items,
  renderItem,
  keyExtractor,
  emptyMessage = 'No items',
}: ListProps<T>) => {
  if (items.length === 0) {
    return <div>{emptyMessage}</div>
  }

  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
      ))}
    </ul>
  )
}

// Usage
<List
  items={users}
  renderItem={(user) => <UserCard user={user} />}
  keyExtractor={(user) => user.id}
/>
```

### Vue Components

**Script Setup with TypeScript**:
```vue
<script setup lang="ts">
interface Props {
  title: string
  items: Item[]
  loading?: boolean
}

interface Emits {
  (e: 'select', item: Item): void
  (e: 'delete', id: string): void
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
})

const emit = defineEmits<Emits>()

const handleSelect = (item: Item) => {
  emit('select', item)
}
</script>
```

**Composable with Types**:
```typescript
// composables/useUser.ts
interface UseUserReturn {
  user: Ref<User | null>
  loading: Ref<boolean>
  error: Ref<Error | null>
  fetchUser: (id: string) => Promise<void>
  updateUser: (data: Partial<User>) => Promise<void>
}

export function useUser(): UseUserReturn {
  const user = ref<User | null>(null)
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const fetchUser = async (id: string) => {
    loading.value = true
    try {
      user.value = await api.getUser(id)
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  const updateUser = async (data: Partial<User>) => {
    if (!user.value) return
    user.value = await api.updateUser(user.value.id, data)
  }

  return { user, loading, error, fetchUser, updateUser }
}
```

## API Type Safety

### API Client Types

```typescript
// api/types.ts

// HTTP methods
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE'

// Request configuration
interface RequestConfig<T = unknown> {
  method?: HttpMethod
  headers?: Record<string, string>
  params?: Record<string, string | number>
  data?: T
  timeout?: number
}

// Response types
interface ApiResponse<T> {
  data: T
  status: number
  headers: Record<string, string>
}

interface ApiError {
  message: string
  code: string
  status: number
  details?: Record<string, any>
}

// Typed API client
class ApiClient {
  async get<T>(url: string, config?: RequestConfig): Promise<T> {
    return this.request<T>({ ...config, method: 'GET', url })
  }

  async post<T, D = unknown>(
    url: string,
    data?: D,
    config?: RequestConfig<D>
  ): Promise<T> {
    return this.request<T>({ ...config, method: 'POST', url, data })
  }

  async put<T, D = unknown>(
    url: string,
    data?: D,
    config?: RequestConfig<D>
  ): Promise<T> {
    return this.request<T>({ ...config, method: 'PUT', url, data })
  }

  async delete<T>(url: string, config?: RequestConfig): Promise<T> {
    return this.request<T>({ ...config, method: 'DELETE', url })
  }

  private async request<T>(config: RequestConfig & { url: string }): Promise<T> {
    // implementation
  }
}
```

### Type-Safe API Endpoints

```typescript
// api/endpoints.ts

// Define API structure with types
interface ApiEndpoints {
  users: {
    list: {
      params: { page?: number; limit?: number }
      response: PaginatedResponse<User>
    }
    get: {
      params: { id: string }
      response: User
    }
    create: {
      body: CreateUserDTO
      response: User
    }
    update: {
      params: { id: string }
      body: UpdateUserDTO
      response: User
    }
    delete: {
      params: { id: string }
      response: void
    }
  }
  products: {
    // Similar structure
  }
}

// Type-safe API function
type EndpointConfig = ApiEndpoints['users']['create']

async function createUser(
  data: EndpointConfig['body']
): Promise<EndpointConfig['response']> {
  return api.post('/users', data)
}
```

### Zod Schema Validation

```typescript
// validation/schemas.ts
import { z } from 'zod'

// Define schema
const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(100),
  role: z.enum(['admin', 'user', 'guest']),
  createdAt: z.date(),
})

// Infer type from schema
type User = z.infer<typeof userSchema>

// Create DTO schema
const createUserSchema = userSchema.omit({
  id: true,
  createdAt: true,
})

type CreateUserDTO = z.infer<typeof createUserSchema>

// API validation
async function createUser(data: unknown): Promise<User> {
  // Validate input
  const validated = createUserSchema.parse(data)

  // API call
  const response = await api.post('/users', validated)

  // Validate response
  return userSchema.parse(response)
}
```

## State Management Types

### React Context

```typescript
// contexts/AuthContext.tsx

interface AuthState {
  user: User | null
  isAuthenticated: boolean
  isLoading: boolean
}

interface AuthActions {
  login: (credentials: LoginCredentials) => Promise<void>
  logout: () => Promise<void>
  refreshToken: () => Promise<void>
}

type AuthContextValue = AuthState & AuthActions

const AuthContext = createContext<AuthContextValue | null>(null)

export const useAuth = (): AuthContextValue => {
  const context = useContext(AuthContext)
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider')
  }
  return context
}

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [state, setState] = useState<AuthState>({
    user: null,
    isAuthenticated: false,
    isLoading: true,
  })

  const login = async (credentials: LoginCredentials) => {
    const { user, token } = await authApi.login(credentials)
    setToken(token)
    setState({ user, isAuthenticated: true, isLoading: false })
  }

  const logout = async () => {
    await authApi.logout()
    clearToken()
    setState({ user: null, isAuthenticated: false, isLoading: false })
  }

  const refreshToken = async () => {
    const { token } = await authApi.refresh()
    setToken(token)
  }

  return (
    <AuthContext.Provider value={{ ...state, login, logout, refreshToken }}>
      {children}
    </AuthContext.Provider>
  )
}
```

### Redux Toolkit

```typescript
// store/slices/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit'

interface UserState {
  user: User | null
  loading: boolean
  error: string | null
}

const initialState: UserState = {
  user: null,
  loading: false,
  error: null,
}

// Typed async thunk
export const fetchUser = createAsyncThunk<
  User,
  string,
  { rejectValue: string }
>('user/fetch', async (userId, { rejectWithValue }) => {
  try {
    return await userApi.getUser(userId)
  } catch (error) {
    return rejectWithValue(getErrorMessage(error))
  }
})

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload
    },
    clearUser: (state) => {
      state.user = null
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true
        state.error = null
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.user = action.payload
        state.loading = false
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.error = action.payload ?? 'Unknown error'
        state.loading = false
      })
  },
})

// Typed selectors
export const selectUser = (state: RootState) => state.user.user
export const selectUserLoading = (state: RootState) => state.user.loading

// Typed hooks
export const useAppDispatch = () => useDispatch<AppDispatch>()
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector
```

### Pinia Store

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

interface UserState {
  user: User | null
  loading: boolean
  error: string | null
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    user: null,
    loading: false,
    error: null,
  }),

  getters: {
    isAuthenticated: (state): boolean => !!state.user,
    userName: (state): string => state.user?.name ?? 'Guest',
    userInitials: (state): string => {
      if (!state.user) return ''
      return state.user.name
        .split(' ')
        .map((n) => n[0])
        .join('')
        .toUpperCase()
    },
  },

  actions: {
    async fetchUser(id: string): Promise<void> {
      this.loading = true
      this.error = null
      try {
        this.user = await userApi.getUser(id)
      } catch (e) {
        this.error = getErrorMessage(e)
        throw e
      } finally {
        this.loading = false
      }
    },

    async updateUser(data: Partial<User>): Promise<void> {
      if (!this.user) return
      this.user = await userApi.updateUser(this.user.id, data)
    },

    logout(): void {
      this.user = null
    },
  },
})
```

## Type Guards and Assertions

### Type Guards

```typescript
// Type predicate
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value &&
    'name' in value
  )
}

// Discriminated unions
interface SuccessResult<T> {
  success: true
  data: T
}

interface ErrorResult {
  success: false
  error: string
}

type Result<T> = SuccessResult<T> | ErrorResult

function handleResult<T>(result: Result<T>) {
  if (result.success) {
    // TypeScript knows result.data exists
    console.log(result.data)
  } else {
    // TypeScript knows result.error exists
    console.error(result.error)
  }
}

// Array type guards
function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) && value.every((item) => typeof item === 'string')
}

// Exhaustive checks
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`)
}

type Status = 'pending' | 'active' | 'completed'

function handleStatus(status: Status): string {
  switch (status) {
    case 'pending':
      return 'Waiting...'
    case 'active':
      return 'In progress'
    case 'completed':
      return 'Done!'
    default:
      return assertNever(status)
  }
}
```

### Assertion Functions

```typescript
// Assertion function
function assertIsDefined<T>(
  value: T,
  message?: string
): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error(message ?? 'Value is not defined')
  }
}

// Usage
function processUser(user: User | null) {
  assertIsDefined(user, 'User is required')
  // TypeScript now knows user is User
  console.log(user.name)
}

// Assertion with type guard
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error('Value is not a User')
  }
}
```

## Common Type Patterns

### Error Handling

```typescript
// Custom error types
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message)
    this.name = 'AppError'
  }
}

class ValidationError extends AppError {
  constructor(
    message: string,
    public fields: Record<string, string>
  ) {
    super(message, 'VALIDATION_ERROR', 400)
    this.name = 'ValidationError'
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 'NOT_FOUND', 404)
    this.name = 'NotFoundError'
  }
}

// Type-safe error handling
function isAppError(error: unknown): error is AppError {
  return error instanceof AppError
}

function handleError(error: unknown): string {
  if (isAppError(error)) {
    return error.message
  }
  if (error instanceof Error) {
    return error.message
  }
  return 'Unknown error'
}
```

### Event Handling

```typescript
// Typed event system
type EventMap = {
  'user:login': User
  'user:logout': void
  'cart:add': CartItem
  'cart:remove': string
}

class TypedEventEmitter {
  private listeners = new Map<string, Set<Function>>()

  on<K extends keyof EventMap>(
    event: K,
    handler: (data: EventMap[K]) => void
  ): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set())
    }
    this.listeners.get(event)!.add(handler)

    return () => this.off(event, handler)
  }

  off<K extends keyof EventMap>(
    event: K,
    handler: (data: EventMap[K]) => void
  ): void {
    this.listeners.get(event)?.delete(handler)
  }

  emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
    this.listeners.get(event)?.forEach((handler) => handler(data))
  }
}

// Usage
const events = new TypedEventEmitter()

events.on('user:login', (user) => {
  console.log(`Welcome, ${user.name}`)
})

events.emit('user:login', currentUser)
```

## TypeScript Review Checklist

When reviewing TypeScript usage:

- [ ] Is tsconfig.json configured appropriately?
- [ ] Are strict mode options enabled?
- [ ] Are types properly defined for domain entities?
- [ ] Are utility types used effectively?
- [ ] Is `any` usage minimized?
- [ ] Are type guards implemented where needed?
- [ ] Are discriminated unions used for state?
- [ ] Are generic types properly constrained?
- [ ] Are API types properly defined?
- [ ] Is runtime validation implemented (Zod, etc.)?
- [ ] Are error types properly structured?
- [ ] Are event types properly defined?
- [ ] Are component props properly typed?
- [ ] Are hooks/composables returning proper types?
- [ ] Are state management types complete?
- [ ] Is exhaustive checking used for unions?

## Common Issues and Fixes

### Issue: Excessive `any` Usage

**Find**:
```bash
# Search for any
grep -r "any" src/ --include="*.ts" --include="*.tsx"
```

**Fix strategies**:
1. Use `unknown` for truly unknown types
2. Create proper type definitions
3. Use generics for flexible types
4. Use union types for specific cases

### Issue: Type Assertions

**Problem**: Overusing `as` type assertions

```typescript
// Bad
const user = response as User

// Good
function isUser(value: unknown): value is User {
  // validation logic
}

if (isUser(response)) {
  const user = response
}
```

### Issue: Missing Null Checks

**Problem**: Not handling null/undefined

```typescript
// Bad
function getName(user: User | null): string {
  return user.name // Error: user might be null
}

// Good
function getName(user: User | null): string {
  return user?.name ?? 'Unknown'
}
```

### Issue: Incorrect Optional Chaining

**Problem**: Using optional chaining when types are guaranteed

```typescript
// Bad - unnecessary optional chaining
function processUser(user: User) {
  console.log(user?.name) // User is always defined
}

// Good
function processUser(user: User) {
  console.log(user.name)
}
```
