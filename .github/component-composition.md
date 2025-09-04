# Component Composition Patterns for React

## Introduction

Component composition is a fundamental React pattern that promotes reusability, maintainability, and flexibility. This guide covers proven composition patterns for building scalable React applications.

## Core Composition Principles

### 1. Favor Composition Over Inheritance
React components should be composed rather than inherited to achieve flexibility and reusability.

### 2. Single Responsibility Principle
Each component should have one clear purpose and responsibility.

### 3. Prop Interface Design
Design props that make components flexible without making them overly complex.

## Essential Composition Patterns

### Children Prop Pattern

This is the most fundamental composition pattern in React. It allows you to nest any content inside a reusable container component using the `children` prop.

```typescript
// ✅ Basic container component
interface ICardProps {
  children: React.ReactNode;
  className?: string;
  variant?: 'default' | 'elevated' | 'outlined';
}

const Card: React.FC<ICardProps> = ({ 
  children, 
  className = '', 
  variant = 'default' 
}) => (
  <div className={`card card--${variant} ${className}`}>
    {children}
  </div>
);

// Usage - Very flexible
<Card variant="elevated" className="user-card">
  <CardHeader>
    <UserAvatar user={user} />
    <UserName>{user.name}</UserName>
  </CardHeader>
  <CardBody>
    <UserDetails user={user} />
  </CardBody>
  <CardActions>
    <Button onClick={onEdit}>Edit</Button>
    <Button onClick={onDelete} variant="danger">Delete</Button>
  </CardActions>
</Card>
```
**Why use it?**  
- Enables maximum flexibility—any content can be passed as children.
- Ideal for layouts, wrappers, and reusable containers.
- Keeps component APIs simple and intuitive.

---

### Render Props Pattern

The render props pattern lets you share logic between components by passing a function as a child. This function receives state or logic from the parent and returns React elements, allowing for highly customizable rendering.

```typescript
// ✅ Data fetching with render props
interface IDataFetcherProps<T> {
  url: string;
  children: (state: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
}

const DataFetcher = <T,>({ url, children }: IDataFetcherProps<T>) => {
  // ...fetch logic...
  return <>{children({ data, loading, error, refetch: fetchData })}</>;
};

// Usage - Multiple rendering strategies
<DataFetcher<IUser[]> url="/api/users">
  {({ data: users, loading, error, refetch }) => {
    if (loading) return <LoadingSpinner />;
    if (error) return <ErrorMessage error={error} onRetry={refetch} />;
    if (!users?.length) return <EmptyState message="No users found" />;
    return (
      <UserList users={users}>
        {users.map(user => (
          <UserCard key={user.id} user={user} />
        ))}
      </UserList>
    );
  }}
</DataFetcher>
```
**Why use it?**  
- Allows logic reuse while giving full control over rendering.
- Useful for data fetching, animation, and state sharing.

---

### Higher-Order Components (HOCs)

A higher-order component is a function that takes a component and returns a new component with enhanced behavior or additional data. HOCs are great for cross-cutting concerns like authentication, error boundaries, or theming.

```typescript
// ✅ Authentication HOC
interface IWithAuthProps {
  user: IUser;
}

const withAuth = <P extends object>(
  WrappedComponent: React.ComponentType<P & IWithAuthProps>
) => {
  const WithAuthComponent = (props: P) => {
    const { user, loading } = useAuth();
    
    if (loading) return <AuthLoadingSpinner />;
    if (!user) return <LoginRedirect />;
    
    return <WrappedComponent {...props} user={user} />;
  };

  WithAuthComponent.displayName = `withAuth(${WrappedComponent.displayName || WrappedComponent.name})`;
  
  return WithAuthComponent;
};

// ✅ Error boundary HOC
const withErrorBoundary = <P extends object>(
  WrappedComponent: React.ComponentType<P>,
  fallback?: React.ComponentType<{ error: Error; resetError: () => void }>
) => {
  return (props: P) => (
    <ErrorBoundary fallback={fallback}>
      <WrappedComponent {...props} />
    </ErrorBoundary>
  );
};

// Usage
const ProtectedDashboard = withAuth(withErrorBoundary(Dashboard));
```

**Why use it?**  
- Encapsulates logic that should be shared across multiple components.
- Keeps your main components focused on their primary responsibility.

### Custom Hooks for Logic Composition

Custom hooks let you extract and share stateful logic between components. They help keep your components clean and focused on rendering, while logic lives in reusable hooks.

```typescript
// ✅ Form management hook
const useForm = <T extends Record<string, any>>({
  initialValues,
  validationSchema = {},
  onSubmit
}: IUseFormOptions<T>) => {
  // ...form state and handlers...
  return { values, errors, touched, isSubmitting, handleChange, handleBlur, handleSubmit, reset, isValid };
};
```
**Why use it?**  
- Cleanly separates logic from UI.
- Makes code more reusable, testable, and easier to maintain.

## Advanced Composition Patterns

### Compound Components Pattern

Compound components are a group of components that work together as a unit and share state via React context. This pattern is ideal for building complex, yet flexible, UI elements like tabs, accordions, or dropdowns.

```typescript
// ✅ Tabs compound component
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (id: string) => void;
} | null>(null);

const useTabs = () => {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tabs components must be used within a Tabs provider');
  }
  return context;
};

const Tabs: React.FC<{ defaultTab: string; children: React.ReactNode }> & {
  List: typeof TabList;
  Tab: typeof Tab;
  Panels: typeof TabPanels;
  Panel: typeof TabPanel;
} = ({ defaultTab, children }) => {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs" role="tablist">
        {children}
      </div>
    </TabsContext.Provider>
  );
};

const TabList: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="tabs__list" role="tablist">
    {children}
  </div>
);

const Tab: React.FC<{ id: string; children: React.ReactNode }> = ({ id, children }) => {
  const { activeTab, setActiveTab } = useTabs();
  const isActive = activeTab === id;
  
  return (
    <button
      className={`tabs__tab ${isActive ? 'tabs__tab--active' : ''}`}
      role="tab"
      aria-selected={isActive}
      tabIndex={isActive ? 0 : -1}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
};

const TabPanels: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="tabs__panels">
    {children}
  </div>
);

const TabPanel: React.FC<{ id: string; children: React.ReactNode }> = ({ id, children }) => {
  const { activeTab } = useTabs();
  
  if (activeTab !== id) return null;
  
  return (
    <div className="tabs__panel" role="tabpanel">
      {children}
    </div>
  );
};

// Attach sub-components
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panels = TabPanels;
Tabs.Panel = TabPanel;

// Usage - Clean and intuitive API
<Tabs defaultTab="profile">
  <Tabs.List>
    <Tabs.Tab id="profile">Profile</Tabs.Tab>
    <Tabs.Tab id="settings">Settings</Tabs.Tab>
    <Tabs.Tab id="notifications">Notifications</Tabs.Tab>
  </Tabs.List>
  
  <Tabs.Panels>
    <Tabs.Panel id="profile">
      <ProfileSettings />
    </Tabs.Panel>
    <Tabs.Panel id="settings">
      <GeneralSettings />
    </Tabs.Panel>
    <Tabs.Panel id="notifications">
      <NotificationSettings />
    </Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

**Why use it?**  
- Provides a clean, intuitive API for related components.
- Allows for flexible composition and extension.

### Provider Pattern for State Management

The provider pattern uses React context to share state and actions across your app. This is especially useful for global state like authentication, themes, or shopping carts.


```typescript
// ✅ Shopping cart provider
<CartProvider>
  <ProductList />
  <CartSummary />
</CartProvider>
```
**Why use it?**  
- Avoids prop drilling by making state accessible anywhere in the component tree.
- Centralizes logic and state management for related features.

---

## Composition Best Practices

### 1. Component Interface Design

Design props that are clear, focused, and flexible. Avoid making interfaces overly complex.

```typescript
// ✅ Clear, focused props interface
interface IButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  loading?: boolean;
  disabled?: boolean;
  className?: string;
}

// ❌ Overly complex interface
interface IBadButtonProps {
  text: string;
  icon?: string;
  iconPosition?: 'left' | 'right';
  color?: string;
  backgroundColor?: string;
  borderColor?: string;
  fontSize?: number;
  padding?: string;
  margin?: string;
  // ... too many styling props
}
```
- Keep interfaces simple and focused on the component's main responsibility.

### 2. Avoid Prop Drilling

Prop drilling is when you pass data through many layers of components. Use context to avoid this and keep your code clean.

```typescript
// ❌ Prop drilling through multiple levels
const App = () => {
  const [user, setUser] = useState(null);
  return <Dashboard user={user} setUser={setUser} />;
};

const Dashboard = ({ user, setUser }) => {
  return <Sidebar user={user} setUser={setUser} />;
};

const Sidebar = ({ user, setUser }) => {
  return <UserMenu user={user} setUser={setUser} />;
};

// ✅ Use context for deeply nested state
const App = () => (
  <UserProvider>
    <Dashboard />
  </UserProvider>
);

const Dashboard = () => <Sidebar />;
const Sidebar = () => <UserMenu />;
const UserMenu = () => {
  const { user, setUser } = useUser();
  // Component logic
};
```

- Use context for global or deeply nested state to simplify your component tree.


### 3. Composition vs Configuration

Favor composition (nesting components) over passing large configuration objects as props. This leads to more flexible and readable code.

```typescript
// ❌ Configuration-heavy approach
<DataTable
  columns={[
    { key: 'name', title: 'Name', sortable: true },
    { key: 'email', title: 'Email', sortable: false },
    { key: 'actions', title: 'Actions', render: (item) => <ActionsMenu item={item} /> }
  ]}
  data={users}
  pagination={{ pageSize: 10, showSizeChanger: true }}
  filters={{ search: true, dateRange: true }}
/>

// ✅ Composition-based approach
<DataTable data={users}>
  <DataTable.Column key="name" title="Name" sortable>
    {(user) => <UserName user={user} />}
  </DataTable.Column>
  <DataTable.Column key="email" title="Email">
    {(user) => <EmailLink email={user.email} />}
  </DataTable.Column>
  <DataTable.Column key="actions" title="Actions">
    {(user) => <UserActions user={user} />}
  </DataTable.Column>
  <DataTable.Pagination pageSize={10} showSizeChanger />
  <DataTable.Filters>
    <DataTable.SearchFilter />
    <DataTable.DateRangeFilter />
  </DataTable.Filters>
</DataTable>
```

- Composition makes your components more flexible and easier to extend.

## Summary

Effective component composition is crucial for building maintainable React applications. Key principles:

- **Use children props** for basic composition
- **Implement render props** for sharing logic with different UIs
- **Create custom hooks** for stateful logic reuse
- **Build compound components** for related component groups
- **Design clean interfaces** that are focused and flexible
- **Avoid prop drilling** with appropriate context usage

By following these patterns, you'll create components that are more reusable, testable, and maintainable.
