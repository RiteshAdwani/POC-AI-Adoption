# React Best Practices for AI-Assisted Development

## Code Organization

### File Size Guidelines

- **Keep components under 200-250 lines**: Components exceeding this limit should be decomposed into smaller, focused sub-components. Each component should have a single, clear responsibility (Single Responsibility Principle). If a component handles multiple concerns (e.g., data fetching, UI rendering, and form validation), extract these into separate components or custom hooks.

- **Break large components into smaller, focused components**: When a component grows beyond 250 lines, identify logical boundaries for splitting. Extract sections like headers, forms, lists, or modals into their own components. Use composition patterns (children props, render props, or compound components) to maintain flexibility while reducing complexity.

- **Extract complex logic into custom hooks**: Move stateful logic, API calls, form handling, and side effects into custom hooks. This promotes reusability, testability, and separation of concerns. Custom hooks should handle one specific domain (e.g., `useUserData`, `useFormValidation`, `useLocalStorage`) and return a clear, typed interface.

- **Create utility functions for reusable operations**: Extract pure functions for data transformation, validation, formatting, and calculations into separate utility modules. Group related utilities together (e.g., `dateUtils.ts`, `stringUtils.ts`, `validationUtils.ts`). Ensure utilities are pure functions that don't depend on React context or external state.

- **Split complex forms into multiple step components**: Forms with more than 5-7 fields or complex validation logic should be broken into logical steps or sections. Use a wizard pattern for multi-step flows, or create section components for large forms. Each step should handle its own validation and state management while contributing to a unified form state.


### File Naming Conventions
| **File Type** | **Convention** | **Examples** |
|---------------|----------------|--------------|
| React Components | PascalCase | `UserProfile.tsx`, `NavigationBar.tsx` |
| Hooks | camelCase with 'use' prefix | `useUserData.ts`, `useLocalStorage.ts` |
| Utilities | camelCase | `dateUtils.ts`, `apiHelpers.ts` |
| Types/Interfaces | PascalCase | `User.ts`, `ApiResponse.ts` |
| Constants | camelCase file, UPPER_SNAKE_CASE exports | `apiConstants.ts` → `API_ENDPOINTS` |
| Services | camelCase | `userService.ts`, `apiClient.ts` |
| Tests | Match source + `.test` | `UserProfile.test.tsx` |
| Stories (Storybook) | Match source + `.stories` | `Button.stories.tsx` |

### File Structure Standards
```
src/
├── components/
│   ├── common/          # Reusable UI components
│   ├── ui/              # Basic design system components
│   ├── features/        # Feature-specific components
│   └── layout/          # Layout components
├── hooks/
│   ├── common/          # General-purpose hooks
│   ├── api/             # API-related hooks
│   └── features/        # Feature-specific hooks
├── services/
│   ├── api/             # API service functions
│   ├── storage/         # Local storage utilities
│   └── external/        # Third-party integrations
├── utils/
│   ├── constants/       # Application constants
│   ├── helpers/         # Pure utility functions
│   └── validators/      # Input validation functions
├── types/               # TypeScript type definitions
├── contexts/            # React contexts
└── assets/              # Static assets

```

#### Folder Organization Principles

**1. Feature-Based Grouping**
```
src/features/user-management/
├── components/
│   ├── UserList/
│   ├── UserForm/
│   └── UserDetails/
├── hooks/
│   ├── useUserCrud.ts
│   └── useUserValidation.ts
├── services/
│   └── userApi.ts
├── types/
│   └── user.types.ts
├── utils/
│   └── userHelpers.ts
└── index.ts
```

**2. Barrel Exports for Clean Imports**
```typescript
// components/common/index.ts
export { Button } from './Button';
export { Modal } from './Modal';
export { Table } from './Table';

// Usage in other files
import { Button, Modal, Table } from '@/components/common';
```

**3. Co-location of Related Files**
```
components/UserProfile/
├── UserProfile.tsx          # Main component
├── UserProfile.test.tsx     # Unit tests
├── UserProfile.stories.tsx  # Storybook stories
├── UserProfile.module.css   # Component-specific styles
├── hooks/                   # Component-specific hooks
│   └── useUserProfile.ts
├── types/                   # Component-specific types
│   └── UserProfile.types.ts
└── index.ts                 # Export barrel
```

#### Import Path Organization

**Absolute Imports with Path Mapping**
```typescript
// tsconfig.json paths configuration
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/services/*": ["./src/services/*"],
      "@/utils/*": ["./src/utils/*"],
      "@/types/*": ["./src/types/*"]
    }
  }
}

// Usage in components
import { Button } from '@/components/common';
import { useUserData } from '@/hooks/api';
import { userService } from '@/services/api';
import { IUser } from '@/types';
```

**Import Order Standards**
```typescript
// 1. External libraries
import React, { useState, useEffect } from 'react';
import { useQuery } from 'react-query';
import axios from 'axios';

// 2. Internal utilities and services
import { userService } from '@/services/api';
import { formatDate } from '@/utils/helpers';

// 3. Internal components and hooks
import { Button, Modal } from '@/components/common';
import { useUserData } from '@/hooks/api';

// 4. Types
import { IUser, IUserFormData } from '@/types';

// 5. Relative imports (if any)
import './UserProfile.css';
```

---

> **📋 Note**: This is a general proposed structure that provides a solid foundation for most React applications. You can adapt and insert project-specific structures based on your requirements, such as having dedicated `types/` folders for each module/sub-module under their respective feature folders, domain-specific service layers, or micro-frontend architectures. The key is maintaining consistency and clear separation of concerns while adapting to your project's specific needs and scale.

## Documentation and Comments

### Component Documentation
```typescript

interface IUserProfileProps {
  /** User data object */
  user: IUser;
  /** Callback fired when user data is updated */
  onUpdate: (userData: Partial<IUser>) => void;
  /** Whether the component is in read-only mode */
  readOnly?: boolean;
}

/**
 * @description Component for displaying and managing user profile information
 * @param {IUserProfileProps} props - Component props
 * @returns {JSX.Element} Rendered user profile component
 * 
 * Features:
 * - Editable user information
 * - Avatar upload functionality
 * - Form validation
 * - Accessibility compliance
 * 
 * Security considerations:
 * - Input sanitization for all user data
 * - File upload validation for avatars
 * - XSS prevention in user-generated content
 */
const UserProfile: React.FC<IUserProfileProps> = ({ user, onUpdate, readOnly = false }) => {
  // Component implementation
};
```

### Function Documentation
```typescript
/**
 * @description Validates and sanitizes user input data before API submission
 * @param {Record<string, unknown>} userData - Raw user input data
 * @returns {IValidatedUserData} Sanitized and validated user data
 * @throws {ValidationError} When required fields are missing or invalid
 */
const validateUserInput = (userData: Record<string, unknown>): IValidatedUserData => {
  // Implementation
};

/**
 * @description Extracts license assignment date from employee license data
 * @param {IEmployeeLicense} employeeLicense - Employee license object
 * @returns {Date | null} Assignment date or null if not found
 */
const extractAssignmentDate = (employeeLicense: IEmployeeLicense): Date | null => {
  // Implementation
};
```

### useEffect Documentation
```typescript
// Monitor background task status and show appropriate notifications
useEffect(() => {
  if (taskId && status === 'completed') {
    showSuccessNotification('Task completed successfully');
  }
}, [taskId, status, showSuccessNotification]);

// Cleanup event listeners on component unmount
useEffect(() => {
  const handleEscape = (event: KeyboardEvent) => {
    if (event.key === 'Escape') {
      onClose();
    }
  };

  document.addEventListener('keydown', handleEscape);
  return () => document.removeEventListener('keydown', handleEscape);
}, [onClose]);
```

## Routing and Navigation

### Centralized Route Management
**Use constants for navigation routes** to maintain consistency and avoid hardcoded strings throughout your application.

```typescript
// Define centralized navigation routes
export const NavigationRoutes = {
  // Static routes
  home: '/',
  dashboard: '/dashboard',
  profile: '/profile',
  
  // Dynamic routes with helper functions
  userDetails: (userId: string) => `/users/${userId}`,
  licenseDetails: (licenseId: string, activeTab?: string) => 
    `/licenses/${licenseId}${activeTab ? `?activeTab=${activeTab}` : ''}`,
  editUser: (userId: string) => `/users/${userId}/edit`,
  
  // Nested routes for modules
  admin: {
    dashboard: '/admin/dashboard',
    users: '/admin/users',
    settings: '/admin/settings',
  }
} as const;

// Usage in components
const navigateToUser = (userId: string) => {
  navigate(NavigationRoutes.userDetails(userId));
};

// ✅ Do this:
navigate(NavigationRoutes.licenseDetails(licenseId, activeTab));

// ❌ Instead of this:
navigate(`/licenses/${licenseId}?activeTab=${activeTab}`);
```

### Route Parameters Best Practices
- **Path parameters**: For essential resource identifiers (`/users/:userId`)
- **Query parameters**: For optional filters, pagination, or view state (`?page=2&filter=active`)
- **Consistent naming**: Use the same parameter names across similar routes
- **Validation**: Always validate route parameters before using them

```typescript
// Route parameter validation
const UserDetailsPage = () => {
  const { userId } = useParams<{ userId: string }>();
  const navigate = useNavigate();

  useEffect(() => {
    if (!userId || !isValidUserId(userId)) {
      navigate(NavigationRoutes.home);
      return;
    }
  }, [userId, navigate]);

  // Component implementation
};
```

## State Management

### Context Usage Guidelines

**When to Use Context:**
- Sharing state across multiple components at different nesting levels
- Avoiding prop drilling for frequently used data (theme, authentication, language)
- Managing global application state that doesn't change frequently
- Providing configuration or services to component trees

**Context Best Practices:**
- **Keep contexts focused** - Each context should handle one specific concern
- **Separate frequently changing data** - Don't mix stable data with rapidly changing state
- **Provide custom hooks** - Always create typed hooks for context access
- **Use multiple contexts** - Split large contexts into smaller, focused ones
- **Memoize context values** - Prevent unnecessary re-renders with useMemo

**When NOT to Use Context:**
- Passing props down only 2-3 levels (prop drilling is acceptable here)
- Frequently changing state that affects many components (consider state management libraries)
- Server state management (use data fetching libraries instead)
- Simple component communication between siblings

```typescript
// ✅ Focused context for authentication
interface IAuthContext {
  user: IUser | null;
  login: (credentials: ILoginCredentials) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

const AuthContext = createContext<IAuthContext | undefined>(undefined);

// Custom hook for type-safe context access
export const useAuth = (): IAuthContext => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

// Provider implementation with memoized value
export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<IUser | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  const login = useCallback(async (credentials: ILoginCredentials) => {
    setIsLoading(true);
    try {
      const user = await authService.login(credentials);
      setUser(user);
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    authService.logout();
  }, []);

  // Memoize context value to prevent unnecessary re-renders
  const value = useMemo(() => ({
    user,
    login,
    logout,
    isLoading,
  }), [user, login, logout, isLoading]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Local vs. Global State Decision Matrix

| State Type | Scope | Management | Examples |
|------------|-------|------------|----------|
| Component UI State | Single component | `useState` | Form inputs, modal open/close, loading states |
| Shared UI State | Multiple components | Context | Theme, sidebar collapsed state |
| Server State | Application-wide | React Query | User data, API responses, cached data |
| Application State | Application-wide | Context + useReducer | Authentication, global settings |

## Type Safety and Enums

### TypeScript Best Practices

**Benefits of Strong Typing:**
- **Compile-time error detection** - Catch errors before runtime
- **Better IDE support** - Improved autocomplete, refactoring, and navigation
- **Self-documenting code** - Types serve as inline documentation
- **Safer refactoring** - TypeScript helps identify breaking changes
- **Team collaboration** - Clear contracts between components and functions

**When to Use Enums:**
- **Fixed sets of related constants** - User roles, status values, categories
- **API response fields** - When backend returns specific string values
- **Configuration options** - Theme modes, sorting options, view types
- **State machine states** - Workflow statuses, loading states

**Enum vs Constants Guidelines:**
- **Use enums** for values that represent a concept (UserRole, TaskStatus)
- **Use const objects** for configuration values (API endpoints, UI constants)
- **Use string literal unions** for simple sets of string values

### Enum Implementation Patterns

```typescript
// ✅ String enums for API compatibility
enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

enum TaskStatus {
  PENDING = 'pending',
  IN_PROGRESS = 'in_progress',
  COMPLETED = 'completed',
  FAILED = 'failed',
}

// ✅ Use for API response fields with specific values
enum LicenseType {
  PREMIUM = 'premium',
  STANDARD = 'standard',
  TRIAL = 'trial',
}
```

### Constants Organization Principles

**Grouping Strategy:**
- **By domain** - Group related constants together
- **By usage** - Separate API, UI, and validation constants
- **By mutability** - Use `as const` for immutable values
- **By scope** - Global vs feature-specific constants

```typescript
// ✅ Group related constants by domain
export const API_ENDPOINTS = {
  USERS: '/api/users',
  LICENSES: '/api/licenses',
  TASKS: '/api/tasks',
} as const;

export const UI_CONSTANTS = {
  PAGE_SIZES: [10, 25, 50, 100],
  DEBOUNCE_DELAY: 300,
  ANIMATION_DURATION: 200,
} as const;

export const VALIDATION_RULES = {
  EMAIL_REGEX: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  PASSWORD_MIN_LENGTH: 8,
  USERNAME_MAX_LENGTH: 50,
} as const;
```

### Type Definition Guidelines

**Interface Design Principles:**
- **Single responsibility** - Each interface should represent one concept
- **Clear naming** - Use descriptive names that explain the data structure
- **Optional vs required** - Be explicit about what's required
- **Composition over inheritance** - Extend interfaces when appropriate
- **API alignment** - Match backend response structures when possible

```typescript
// ✅ API response type matching backend structure
interface IUserApiResponse {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  createdAt: string;
  updatedAt: string;
}

// ✅ Component props with clear optional/required distinction
interface IUserCardProps {
  /** User object to display - required */
  user: IUser;
  /** Optional edit callback */
  onEdit?: (userId: string) => void;
  /** Optional delete callback */
  onDelete?: (userId: string) => void;
  /** Additional CSS classes */
  className?: string;
  /** Whether to show action buttons */
  showActions?: boolean;
}

// ✅ Discriminated unions for complex state management
type LoadingState = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: unknown }
  | { status: 'error'; error: string };

// ✅ Generic types for reusable patterns
interface IApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

interface IPaginatedResponse<T> {
  items: T[];
  totalCount: number;
  pageSize: number;
  currentPage: number;
}
```

### Type Safety Best Practices

**Type Guards and Validation:**
- **Runtime type checking** - Validate data from external sources
- **Type guards** - Create functions to narrow types safely
- **Assertion functions** - Use for critical type validations
- **Unknown over any** - Prefer `unknown` for truly dynamic data

```typescript
// ✅ Type guard for API responses
const isValidUser = (data: unknown): data is IUser => {
  return (
    typeof data === 'object' &&
    data !== null &&
    typeof (data as IUser).id === 'string' &&
    typeof (data as IUser).email === 'string' &&
    Object.values(UserRole).includes((data as IUser).role)
  );
};

// ✅ Safe API response handling
const fetchUser = async (userId: string): Promise<IUser> => {
  const response = await api.get(API_ENDPOINTS.USER_DETAILS(userId));
  
  if (!isValidUser(response.data)) {
    throw new Error(ERROR_MESSAGES.INVALID_USER_DATA);
  }
  
  return response.data;
};
```

### Safe Property Access with Optional Chaining

**Use optional chaining (`?.`) for safe property access** when dealing with potentially undefined or null values, especially from API responses or user input.

```typescript
// ✅ Safe property access with optional chaining
const UserProfile = ({ user }: { user: IUser | null }) => {
  return (
    <div>
      <h1>{user?.name || 'Unknown User'}</h1>
      <p>{user?.profile?.bio || 'No bio available'}</p>
      <img src={user?.avatar?.url} alt={user?.name} />
      
      {/* Safe method calls */}
      <p>Joined: {user?.createdAt?.toLocaleDateString()}</p>
      
      {/* Safe array access */}
      <p>Skills: {user?.skills?.[0] || 'No skills listed'}</p>
    </div>
  );
};

// ✅ Optional chaining with nullish coalescing
const formatUserDisplayName = (user: IUser | undefined) => {
  return user?.firstName && user?.lastName 
    ? `${user.firstName} ${user.lastName}`
    : user?.email?.split('@')[0] ?? 'Anonymous';
};

// ✅ Safe API response handling
const processUserData = (response: IApiResponse<IUser[]> | undefined) => {
  const users = response?.data ?? [];
  const totalCount = response?.meta?.totalCount ?? 0;
  const hasNextPage = response?.pagination?.hasNext ?? false;
  
  return { users, totalCount, hasNextPage };
};

// ❌ Avoid: Unsafe property access
const displayUserInfo = (user: IUser | null) => {
  // This will throw if user is null
  return user.name + ' - ' + user.email;
};

// ✅ Do: Safe property access
const displayUserInfo = (user: IUser | null) => {
  return `${user?.name ?? 'Unknown'} - ${user?.email ?? 'No email'}`;
};

// ✅ Complex nested optional chaining
const getLicenseExpiry = (user: IUser | undefined) => {
  return user?.licenses?.[0]?.expiryDate?.toISOString() ?? null;
};

// ✅ Optional chaining with function calls
const notifyUser = (user: IUser | undefined, message: string) => {
  user?.preferences?.notifications?.email && 
    sendEmailNotification?.(user.email, message);
};
```

**Optional Chaining Best Practices:**
- **Use with API responses** - Always assume external data might be missing
- **Combine with nullish coalescing (`??`)** - Provide meaningful defaults
- **Don't overuse** - If you know a property exists, regular access is fine
- **Chain safely** - Each `?.` stops execution if the value is nullish
- **Consider type guards** - For complex objects, type guards might be clearer

**When to Use Optional Chaining:**
- API response data that might be incomplete
- User-generated content or optional form fields
- Deeply nested object properties
- Dynamic property access
- Method calls on potentially undefined objects

### Advanced Type Patterns

**Utility Types for Common Scenarios:**
- **Partial<T>** - For update operations
- **Pick<T, K>** - For selecting specific properties
- **Omit<T, K>** - For excluding properties
- **Record<K, V>** - For key-value mappings

```typescript
// ✅ Using utility types for different operations
type CreateUserRequest = Omit<IUser, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateUserRequest = Partial<Pick<IUser, 'firstName' | 'lastName' | 'email'>>;
type UserSearchFilters = Record<string, string | number | boolean>;

// ✅ Custom utility types for specific needs
type RequiredFields<T, K extends keyof T> = T & Required<Pick<T, K>>;
type OptionalFields<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// Usage example
type UserWithRequiredEmail = RequiredFields<IUser, 'email'>;
```

## Performance Optimization

### Key Performance Guidelines

**Measurement First:**
- Profile before optimizing - identify real bottlenecks
- Use React DevTools Profiler to measure performance
- Focus on user-perceived performance

**Memoization Best Practices:**
- **Use memoization when:** Expensive calculations, large data operations, preventing child re-renders
- **Avoid memoization for:** Simple operations, primitive values, frequently changing dependencies

```typescript
// ✅ Good: Expensive operation with stable dependencies
const processedUsers = useMemo(() => {
  return users
    .filter(user => user.isActive)
    .sort((a, b) => a.name.localeCompare(b.name));
}, [users]);

// ❌ Avoid: Simple operation that doesn't need memoization
const userCount = useMemo(() => users.length, [users]);

// ✅ Good: Event handler for child components
const handleUserSelect = useCallback((userId: string) => {
  setSelectedUsers(prev => [...prev, userId]);
}, []);
```

**Common Anti-patterns:**
- Creating objects/arrays in render
- Using array index as keys for dynamic lists
- Over-memoizing simple components

---

## Accessibility and UX

### Basic Accessibility Requirements

**The Four Principles of Accessibility (WCAG):**
- **Perceivable** - Information must be presentable in ways users can perceive
- **Operable** - Interface components must be operable by all users
- **Understandable** - Information and UI operation must be understandable
- **Robust** - Content must work with various assistive technologies

### 1. Semantic HTML
```typescript
// ✅ Use proper HTML elements
<main>
  <header>
    <h1>User Dashboard</h1>
  </header>
  <section>
    <h2>User List</h2>
    <article>
      <h3>John Doe</h3>
    </article>
  </section>
</main>
```

### 2. Keyboard Navigation
```typescript
// ✅ Handle keyboard events
const handleKeyDown = (event: KeyboardEvent) => {
  switch (event.key) {
    case 'Enter':
    case ' ':
      handleAction();
      break;
    case 'Escape':
      closeModal();
      break;
  }
};
```

### 3. Focus Management
```typescript
// ✅ Manage focus in modals
useEffect(() => {
  if (isOpen) {
    modalRef.current?.focus();
  }
}, [isOpen]);
```

### 4. ARIA Labels
```typescript
// ✅ Provide accessible names
<button aria-label="Delete user">🗑️</button>
<input aria-describedby="help-text" />
<div id="help-text">Enter your full name</div>
```

### 5. Form Labels
```typescript
// ✅ Associate labels with inputs
<label htmlFor="email">Email</label>
<input id="email" type="email" required />
```

### 6. Color and Contrast
```typescript
// ✅ Don't rely only on color
<span className="status-error">
  ❌ Error: Form submission failed
</span>
```

### 7. Alt Text for Images
```typescript
// ✅ Descriptive alt text
<img src={user.avatar} alt={`${user.name}'s profile picture`} />

// ✅ Decorative images
<img src="/decoration.png" alt="" />
```

### 8. Table Headers
```typescript
// ✅ Proper table structure
<table>
  <thead>
    <tr>
      <th scope="col">Name</th>
      <th scope="col">Email</th>
    </tr>
  </thead>
</table>
```

### 9. Live Announcements
```typescript
// ✅ Announce dynamic changes
<div aria-live="polite">
  {loading ? 'Loading...' : `${results.length} users found`}
</div>
```

### User Experience Essentials

```typescript
// ✅ Loading states
if (isLoading) return <div>Loading...</div>;
if (error) return <div>Error: {error.message}</div>;

// ✅ Confirmation dialogs
{showConfirm && (
  <div role="dialog">
    <p>Are you sure?</p>
    <button onClick={onConfirm}>Yes</button>
    <button onClick={onCancel}>Cancel</button>
  </div>
)}
```

## Conclusion

Following these enhanced best practices will significantly improve code quality, maintainability, and developer productivity. These guidelines should be regularly reviewed and updated as the project evolves and new patterns emerge.

Key takeaways:
- Prioritize type safety and clear documentation
- Optimize for performance without premature optimization
- Ensure accessibility is built-in, not an afterthought
- Use appropriate state management for each use case
- Maintain consistent patterns across the codebase
