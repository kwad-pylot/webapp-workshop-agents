---
name: feature-integrator
description: |
  Integrate frontend and backend into cohesive features.
  Connects UI components to API endpoints with proper data flow.
---

# Feature Integrator Skill

## Purpose
Connect frontend UI to backend APIs for complete feature implementation.

## Integration Workflow

1. **Define Contract**: API interface between frontend and backend
2. **Implement Backend**: API endpoint with validation
3. **Create API Client**: Type-safe frontend API calls
4. **Build UI**: Components that consume the API
5. **Handle States**: Loading, error, success states
6. **Test Integration**: End-to-end verification

## API Contract Definition

```typescript
// types/api.ts

// Shared types between frontend and backend
export interface User {
  id: string;
  email: string;
  name: string | null;
  createdAt: string;
}

export interface CreateUserInput {
  email: string;
  name?: string;
  password: string;
}

export interface UpdateUserInput {
  name?: string;
  email?: string;
}

// API Response types
export interface ApiResponse<T> {
  data: T;
}

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

export interface ApiError {
  error: string;
  code: string;
  details?: Record<string, string[]>;
}
```

## Complete Feature Example: User Profile

### 1. Backend API
```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { updateUserSchema } from '@/lib/validations';
import { auth } from '@/lib/auth';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await prisma.user.findUnique({
    where: { id: params.id },
    select: { id: true, email: true, name: true, image: true, createdAt: true },
  });

  if (!user) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 });
  }

  return NextResponse.json(user);
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  // Auth check
  const session = await auth();
  if (!session || session.user.id !== params.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Validate input
  const body = await request.json();
  const result = updateUserSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json(
      { error: 'Validation failed', details: result.error.flatten().fieldErrors },
      { status: 400 }
    );
  }

  // Update user
  const user = await prisma.user.update({
    where: { id: params.id },
    data: result.data,
    select: { id: true, email: true, name: true, image: true, createdAt: true },
  });

  return NextResponse.json(user);
}
```

### 2. API Client
```typescript
// lib/api/users.ts
import { api } from '@/lib/api-client';
import type { User, UpdateUserInput } from '@/types/api';

export const usersApi = {
  get: (id: string) => api.get<User>(`/users/${id}`),
  update: (id: string, data: UpdateUserInput) => api.patch<User>(`/users/${id}`, data),
};
```

### 3. React Query Hooks
```typescript
// hooks/use-user.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '@/lib/api/users';
import type { UpdateUserInput } from '@/types/api';

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => usersApi.get(id),
    enabled: !!id,
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserInput }) =>
      usersApi.update(id, data),
    onSuccess: (user) => {
      // Update cache
      queryClient.setQueryData(['users', user.id], user);
    },
  });
}
```

### 4. UI Component
```tsx
// components/profile/profile-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useUser, useUpdateUser } from '@/hooks/use-user';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { toast } from 'sonner';

const profileSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email'),
});

type ProfileFormData = z.infer<typeof profileSchema>;

interface ProfileFormProps {
  userId: string;
}

export function ProfileForm({ userId }: ProfileFormProps) {
  // Fetch user data
  const { data: user, isLoading, error } = useUser(userId);
  const updateUser = useUpdateUser();

  // Form setup
  const form = useForm<ProfileFormData>({
    resolver: zodResolver(profileSchema),
    values: user ? { name: user.name || '', email: user.email } : undefined,
  });

  // Handle submit
  const onSubmit = async (data: ProfileFormData) => {
    try {
      await updateUser.mutateAsync({ id: userId, data });
      toast.success('Profile updated!');
    } catch (error) {
      toast.error('Failed to update profile');
    }
  };

  // Loading state
  if (isLoading) {
    return <ProfileFormSkeleton />;
  }

  // Error state
  if (error) {
    return (
      <div className="text-center py-8">
        <p className="text-destructive">Failed to load profile</p>
        <Button variant="outline" onClick={() => window.location.reload()}>
          Retry
        </Button>
      </div>
    );
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      <div className="space-y-2">
        <label htmlFor="name" className="text-sm font-medium">
          Name
        </label>
        <Input
          id="name"
          {...form.register('name')}
          aria-invalid={!!form.formState.errors.name}
        />
        {form.formState.errors.name && (
          <p className="text-sm text-destructive">
            {form.formState.errors.name.message}
          </p>
        )}
      </div>

      <div className="space-y-2">
        <label htmlFor="email" className="text-sm font-medium">
          Email
        </label>
        <Input
          id="email"
          type="email"
          {...form.register('email')}
          aria-invalid={!!form.formState.errors.email}
        />
        {form.formState.errors.email && (
          <p className="text-sm text-destructive">
            {form.formState.errors.email.message}
          </p>
        )}
      </div>

      <Button
        type="submit"
        disabled={updateUser.isPending || !form.formState.isDirty}
      >
        {updateUser.isPending ? 'Saving...' : 'Save Changes'}
      </Button>
    </form>
  );
}

function ProfileFormSkeleton() {
  return (
    <div className="space-y-4 animate-pulse">
      <div className="space-y-2">
        <div className="h-4 w-12 bg-muted rounded" />
        <div className="h-10 bg-muted rounded" />
      </div>
      <div className="space-y-2">
        <div className="h-4 w-12 bg-muted rounded" />
        <div className="h-10 bg-muted rounded" />
      </div>
      <div className="h-10 w-28 bg-muted rounded" />
    </div>
  );
}
```

### 5. Page Integration
```tsx
// app/profile/page.tsx
import { redirect } from 'next/navigation';
import { auth } from '@/lib/auth';
import { ProfileForm } from '@/components/profile/profile-form';

export default async function ProfilePage() {
  const session = await auth();

  if (!session) {
    redirect('/login');
  }

  return (
    <div className="container max-w-2xl py-8">
      <h1 className="text-2xl font-bold mb-6">Profile Settings</h1>
      <ProfileForm userId={session.user.id} />
    </div>
  );
}
```

## Integration Patterns

### Optimistic Updates
```typescript
useMutation({
  mutationFn: updateUser,
  onMutate: async (newData) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['users', id] });

    // Snapshot previous value
    const previous = queryClient.getQueryData(['users', id]);

    // Optimistically update
    queryClient.setQueryData(['users', id], (old: User) => ({
      ...old,
      ...newData,
    }));

    return { previous };
  },
  onError: (err, newData, context) => {
    // Rollback on error
    queryClient.setQueryData(['users', id], context?.previous);
    toast.error('Update failed');
  },
  onSettled: () => {
    // Refetch to ensure consistency
    queryClient.invalidateQueries({ queryKey: ['users', id] });
  },
});
```

### Form Reset After Success
```typescript
const form = useForm();
const mutation = useMutation({
  mutationFn: createItem,
  onSuccess: () => {
    form.reset();
    toast.success('Created!');
  },
});
```

## Quality Checklist

- [ ] API contract defined and shared
- [ ] Type safety end-to-end
- [ ] Loading states handled
- [ ] Error states handled
- [ ] Form validation on client
- [ ] Input validation on server
- [ ] Optimistic updates where appropriate
- [ ] Cache invalidation correct
- [ ] Accessibility maintained
