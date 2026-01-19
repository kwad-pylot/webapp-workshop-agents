---
name: api-consumer
description: |
  Build type-safe API clients for consuming backend services.
  Creates reusable hooks and utilities for data fetching.
---

# API Consumer Skill

## Purpose
Create robust, type-safe API consumption layer for frontend applications.

## Base API Client

```typescript
// lib/api-client.ts
const API_BASE = process.env.NEXT_PUBLIC_API_URL || '/api';

export class ApiError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string,
    public details?: Record<string, string[]>
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

interface RequestOptions extends RequestInit {
  params?: Record<string, string | number | boolean | undefined>;
}

async function request<T>(
  endpoint: string,
  options: RequestOptions = {}
): Promise<T> {
  const { params, ...init } = options;

  // Build URL with query params
  let url = `${API_BASE}${endpoint}`;
  if (params) {
    const searchParams = new URLSearchParams();
    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined) {
        searchParams.append(key, String(value));
      }
    });
    const queryString = searchParams.toString();
    if (queryString) {
      url += `?${queryString}`;
    }
  }

  // Default headers
  const headers = new Headers(init.headers);
  if (!headers.has('Content-Type') && init.body) {
    headers.set('Content-Type', 'application/json');
  }

  // Add auth token
  const token = getAuthToken();
  if (token) {
    headers.set('Authorization', `Bearer ${token}`);
  }

  const response = await fetch(url, { ...init, headers });

  // Handle errors
  if (!response.ok) {
    const body = await response.json().catch(() => ({}));
    throw new ApiError(
      response.status,
      body.code || 'UNKNOWN_ERROR',
      body.error || `Request failed with status ${response.status}`,
      body.details
    );
  }

  // Handle 204 No Content
  if (response.status === 204) {
    return undefined as T;
  }

  return response.json();
}

// Helper to get auth token
function getAuthToken(): string | null {
  if (typeof window === 'undefined') return null;
  return localStorage.getItem('accessToken');
}

// Typed API methods
export const api = {
  get: <T>(endpoint: string, params?: RequestOptions['params']) =>
    request<T>(endpoint, { method: 'GET', params }),

  post: <T>(endpoint: string, data?: unknown) =>
    request<T>(endpoint, {
      method: 'POST',
      body: data ? JSON.stringify(data) : undefined,
    }),

  put: <T>(endpoint: string, data: unknown) =>
    request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    }),

  patch: <T>(endpoint: string, data: unknown) =>
    request<T>(endpoint, {
      method: 'PATCH',
      body: JSON.stringify(data),
    }),

  delete: <T>(endpoint: string) =>
    request<T>(endpoint, { method: 'DELETE' }),
};
```

## Domain-Specific API Modules

```typescript
// lib/api/users.ts
import { api } from '@/lib/api-client';
import type { User, CreateUserInput, UpdateUserInput, PaginatedResponse } from '@/types';

export const usersApi = {
  list: (params?: { page?: number; limit?: number; search?: string }) =>
    api.get<PaginatedResponse<User>>('/users', params),

  get: (id: string) =>
    api.get<User>(`/users/${id}`),

  create: (data: CreateUserInput) =>
    api.post<User>('/users', data),

  update: (id: string, data: UpdateUserInput) =>
    api.patch<User>(`/users/${id}`, data),

  delete: (id: string) =>
    api.delete<void>(`/users/${id}`),
};

// lib/api/posts.ts
import { api } from '@/lib/api-client';
import type { Post, CreatePostInput, PaginatedResponse } from '@/types';

export const postsApi = {
  list: (params?: { page?: number; authorId?: string; status?: string }) =>
    api.get<PaginatedResponse<Post>>('/posts', params),

  get: (idOrSlug: string) =>
    api.get<Post>(`/posts/${idOrSlug}`),

  create: (data: CreatePostInput) =>
    api.post<Post>('/posts', data),

  update: (id: string, data: Partial<CreatePostInput>) =>
    api.patch<Post>(`/posts/${id}`, data),

  delete: (id: string) =>
    api.delete<void>(`/posts/${id}`),

  publish: (id: string) =>
    api.post<Post>(`/posts/${id}/publish`),
};
```

## React Query Hooks

```typescript
// hooks/api/use-users.ts
import { useQuery, useMutation, useQueryClient, useInfiniteQuery } from '@tanstack/react-query';
import { usersApi } from '@/lib/api/users';
import type { CreateUserInput, UpdateUserInput } from '@/types';

// Query keys factory
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (params: object) => [...userKeys.lists(), params] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

// List users with pagination
export function useUsers(params?: { page?: number; limit?: number; search?: string }) {
  return useQuery({
    queryKey: userKeys.list(params ?? {}),
    queryFn: () => usersApi.list(params),
  });
}

// Infinite scroll users
export function useInfiniteUsers(limit = 20) {
  return useInfiniteQuery({
    queryKey: userKeys.lists(),
    queryFn: ({ pageParam = 1 }) => usersApi.list({ page: pageParam, limit }),
    getNextPageParam: (lastPage) =>
      lastPage.meta.page < lastPage.meta.totalPages
        ? lastPage.meta.page + 1
        : undefined,
    initialPageParam: 1,
  });
}

// Single user
export function useUser(id: string) {
  return useQuery({
    queryKey: userKeys.detail(id),
    queryFn: () => usersApi.get(id),
    enabled: !!id,
  });
}

// Create user
export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserInput) => usersApi.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}

// Update user
export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserInput }) =>
      usersApi.update(id, data),
    onSuccess: (user) => {
      queryClient.setQueryData(userKeys.detail(user.id), user);
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}

// Delete user
export function useDeleteUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => usersApi.delete(id),
    onSuccess: (_, id) => {
      queryClient.removeQueries({ queryKey: userKeys.detail(id) });
      queryClient.invalidateQueries({ queryKey: userKeys.lists() });
    },
  });
}
```

## Error Handling in Components

```tsx
// components/user-list.tsx
import { useUsers } from '@/hooks/api/use-users';
import { ApiError } from '@/lib/api-client';

export function UserList() {
  const { data, isLoading, error, refetch } = useUsers();

  if (isLoading) {
    return <UserListSkeleton />;
  }

  if (error) {
    return (
      <div className="text-center py-8">
        <p className="text-destructive mb-4">
          {error instanceof ApiError
            ? error.message
            : 'Failed to load users'}
        </p>
        <Button variant="outline" onClick={() => refetch()}>
          Try Again
        </Button>
      </div>
    );
  }

  if (!data?.data.length) {
    return (
      <div className="text-center py-8 text-muted-foreground">
        No users found
      </div>
    );
  }

  return (
    <ul className="divide-y">
      {data.data.map((user) => (
        <UserListItem key={user.id} user={user} />
      ))}
    </ul>
  );
}
```

## Mutation with Error Handling

```tsx
import { useCreateUser } from '@/hooks/api/use-users';
import { ApiError } from '@/lib/api-client';
import { toast } from 'sonner';

export function CreateUserForm() {
  const createUser = useCreateUser();
  const [errors, setErrors] = useState<Record<string, string[]>>({});

  const handleSubmit = async (data: CreateUserInput) => {
    setErrors({});

    try {
      await createUser.mutateAsync(data);
      toast.success('User created!');
      // Reset form or redirect
    } catch (error) {
      if (error instanceof ApiError) {
        if (error.code === 'VALIDATION_ERROR' && error.details) {
          setErrors(error.details);
        } else {
          toast.error(error.message);
        }
      } else {
        toast.error('Something went wrong');
      }
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <Input name="email" error={errors.email?.[0]} />
      <Input name="name" error={errors.name?.[0]} />
      <Button loading={createUser.isPending}>Create</Button>
    </form>
  );
}
```

## Query Client Setup

```tsx
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      gcTime: 5 * 60 * 1000, // 5 minutes
      retry: (failureCount, error) => {
        // Don't retry on 4xx errors
        if (error instanceof ApiError && error.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
    },
  },
});
```

## Quality Checklist

- [ ] Base client handles all HTTP methods
- [ ] Auth token automatically attached
- [ ] Errors typed and handled
- [ ] Query keys organized with factory
- [ ] Cache invalidation on mutations
- [ ] Loading states exposed
- [ ] Retry logic appropriate
- [ ] TypeScript types end-to-end
