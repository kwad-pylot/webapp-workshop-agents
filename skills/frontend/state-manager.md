---
name: state-manager
description: |
  Implement client-side state management using modern patterns.
  Covers local state, global state, and server state.
---

# State Manager Skill

## Purpose
Implement appropriate state management patterns for different use cases.

## State Categories

| Category | Description | Solution |
|----------|-------------|----------|
| Local State | Component-specific state | useState, useReducer |
| Global State | Shared across components | Zustand, Jotai, Context |
| Server State | Data from APIs | React Query, SWR |
| URL State | State in URL | useSearchParams, nuqs |
| Form State | Form values/validation | react-hook-form |

## Local State (useState/useReducer)

### useState
```tsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

### useReducer (Complex State)
```tsx
interface State {
  items: Item[];
  loading: boolean;
  error: string | null;
}

type Action =
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: Item[] }
  | { type: 'FETCH_ERROR'; payload: string }
  | { type: 'ADD_ITEM'; payload: Item }
  | { type: 'REMOVE_ITEM'; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, loading: false, items: action.payload };
    case 'FETCH_ERROR':
      return { ...state, loading: false, error: action.payload };
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.payload] };
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter(i => i.id !== action.payload) };
    default:
      return state;
  }
}

function ItemList() {
  const [state, dispatch] = useReducer(reducer, {
    items: [],
    loading: false,
    error: null,
  });

  // Usage
  dispatch({ type: 'ADD_ITEM', payload: newItem });
}
```

## Global State (Zustand)

### Store Setup
```tsx
// stores/app-store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
}

interface AppState {
  // State
  user: User | null;
  theme: 'light' | 'dark';
  sidebarOpen: boolean;

  // Actions
  setUser: (user: User | null) => void;
  setTheme: (theme: 'light' | 'dark') => void;
  toggleSidebar: () => void;
  logout: () => void;
}

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({
      // Initial state
      user: null,
      theme: 'light',
      sidebarOpen: true,

      // Actions
      setUser: (user) => set({ user }),
      setTheme: (theme) => set({ theme }),
      toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
      logout: () => set({ user: null }),
    }),
    {
      name: 'app-storage', // localStorage key
      partialize: (state) => ({ theme: state.theme }), // Only persist theme
    }
  )
);
```

### Using the Store
```tsx
function Header() {
  // Select only what you need (prevents unnecessary re-renders)
  const user = useAppStore((state) => state.user);
  const logout = useAppStore((state) => state.logout);

  return (
    <header>
      {user ? (
        <>
          <span>{user.name}</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <a href="/login">Login</a>
      )}
    </header>
  );
}
```

### Computed Values with Selectors
```tsx
// Define selectors outside component
const selectIsAuthenticated = (state: AppState) => state.user !== null;
const selectUserInitials = (state: AppState) =>
  state.user?.name
    .split(' ')
    .map((n) => n[0])
    .join('')
    .toUpperCase() ?? '';

function Avatar() {
  const isAuthenticated = useAppStore(selectIsAuthenticated);
  const initials = useAppStore(selectUserInitials);

  if (!isAuthenticated) return null;
  return <div className="avatar">{initials}</div>;
}
```

## Server State (TanStack Query)

### Setup
```tsx
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      retry: 1,
    },
  },
});

// app/providers.tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/lib/query-client';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

### Query Hooks
```tsx
// hooks/use-users.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/api';

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => api.get<User[]>('/users'),
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => api.get<User>(`/users/${id}`),
    enabled: !!id, // Only fetch if id exists
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserInput) => api.post<User>('/users', data),
    onSuccess: () => {
      // Invalidate and refetch users list
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<User> }) =>
      api.patch<User>(`/users/${id}`, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['users', id] });
    },
  });
}
```

### Usage
```tsx
function UserList() {
  const { data: users, isLoading, error } = useUsers();
  const createUser = useCreateUser();

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <button
        onClick={() => createUser.mutate({ email: 'new@example.com' })}
        disabled={createUser.isPending}
      >
        {createUser.isPending ? 'Creating...' : 'Add User'}
      </button>

      <ul>
        {users?.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Context API (Simple Global State)

```tsx
// contexts/theme-context.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface ThemeContextValue {
  theme: 'light' | 'dark';
  setTheme: (theme: 'light' | 'dark') => void;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme((t) => (t === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

## Choosing the Right Approach

| Use Case | Recommended |
|----------|-------------|
| Component UI state | useState |
| Complex local state | useReducer |
| Theme, auth, settings | Zustand |
| API data | React Query |
| Form data | react-hook-form |
| URL params | useSearchParams |

## Quality Checklist

- [ ] No prop drilling (use context/stores)
- [ ] Server state separate from UI state
- [ ] Selectors for derived state
- [ ] Proper loading/error states
- [ ] Optimistic updates where appropriate
- [ ] Persist necessary state
- [ ] Clean up subscriptions
