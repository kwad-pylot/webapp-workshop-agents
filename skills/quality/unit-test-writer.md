---
name: unit-test-writer
description: |
  Write comprehensive unit tests for functions, components, and hooks.
  Creates isolated tests with proper assertions and edge case coverage.
---

# Unit Test Writer Skill

## Purpose
Write thorough unit tests that verify individual units of code in isolation.

## Testing Setup

### Vitest Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    include: ['**/*.test.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules', 'test'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});

// vitest.setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
  cleanup();
});
```

## Testing Utility Functions

```typescript
// utils/format.test.ts
import { describe, it, expect } from 'vitest';
import { formatCurrency, formatDate, truncate, slugify } from './format';

describe('formatCurrency', () => {
  it('formats positive numbers correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
    expect(formatCurrency(0)).toBe('$0.00');
    expect(formatCurrency(1000000)).toBe('$1,000,000.00');
  });

  it('formats negative numbers', () => {
    expect(formatCurrency(-50)).toBe('-$50.00');
  });

  it('handles edge cases', () => {
    expect(formatCurrency(0.001)).toBe('$0.00');
    expect(formatCurrency(0.009)).toBe('$0.01'); // Rounds up
  });
});

describe('formatDate', () => {
  it('formats dates in default format', () => {
    const date = new Date('2024-01-15T12:00:00Z');
    expect(formatDate(date)).toBe('Jan 15, 2024');
  });

  it('handles string dates', () => {
    expect(formatDate('2024-01-15')).toBe('Jan 15, 2024');
  });

  it('returns empty string for invalid dates', () => {
    expect(formatDate('invalid')).toBe('');
    expect(formatDate(null as any)).toBe('');
  });
});

describe('truncate', () => {
  it('returns original string if shorter than limit', () => {
    expect(truncate('hello', 10)).toBe('hello');
  });

  it('truncates and adds ellipsis', () => {
    expect(truncate('hello world', 8)).toBe('hello...');
  });

  it('handles edge cases', () => {
    expect(truncate('', 10)).toBe('');
    expect(truncate('hi', 2)).toBe('hi');
    expect(truncate('hello', 3)).toBe('...');
  });
});

describe('slugify', () => {
  it('converts to lowercase', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  it('replaces spaces with hyphens', () => {
    expect(slugify('hello world')).toBe('hello-world');
  });

  it('removes special characters', () => {
    expect(slugify("Hello! What's up?")).toBe('hello-whats-up');
  });

  it('handles multiple spaces', () => {
    expect(slugify('hello   world')).toBe('hello-world');
  });
});
```

## Testing React Components

```tsx
// components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });

  it('handles click events', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click</Button>);

    fireEvent.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('can be disabled', () => {
    const handleClick = vi.fn();
    render(<Button disabled onClick={handleClick}>Click</Button>);

    const button = screen.getByRole('button');
    expect(button).toBeDisabled();

    fireEvent.click(button);
    expect(handleClick).not.toHaveBeenCalled();
  });

  it('shows loading state', () => {
    render(<Button loading>Submit</Button>);

    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
    expect(screen.getByTestId('spinner')).toBeInTheDocument();
  });

  it('applies variant styles', () => {
    const { rerender } = render(<Button variant="primary">Click</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-primary');

    rerender(<Button variant="destructive">Click</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-destructive');
  });

  it('forwards ref correctly', () => {
    const ref = vi.fn();
    render(<Button ref={ref}>Click</Button>);
    expect(ref).toHaveBeenCalledWith(expect.any(HTMLButtonElement));
  });

  it('spreads additional props', () => {
    render(<Button data-testid="custom-button" aria-label="Custom">Click</Button>);

    const button = screen.getByTestId('custom-button');
    expect(button).toHaveAttribute('aria-label', 'Custom');
  });
});
```

## Testing Custom Hooks

```tsx
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });

  it('sets specific value', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.set(42);
    });

    expect(result.current.count).toBe(42);
  });
});

// hooks/useLocalStorage.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { useLocalStorage } from './useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
    vi.spyOn(Storage.prototype, 'setItem');
    vi.spyOn(Storage.prototype, 'getItem');
  });

  it('returns initial value when localStorage is empty', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'default'));
    expect(result.current[0]).toBe('default');
  });

  it('returns stored value from localStorage', () => {
    localStorage.setItem('key', JSON.stringify('stored'));
    const { result } = renderHook(() => useLocalStorage('key', 'default'));
    expect(result.current[0]).toBe('stored');
  });

  it('updates localStorage when value changes', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'initial'));

    act(() => {
      result.current[1]('updated');
    });

    expect(result.current[0]).toBe('updated');
    expect(localStorage.setItem).toHaveBeenCalledWith('key', JSON.stringify('updated'));
  });

  it('handles objects', () => {
    const { result } = renderHook(() =>
      useLocalStorage('user', { name: 'John' })
    );

    act(() => {
      result.current[1]({ name: 'Jane' });
    });

    expect(result.current[0]).toEqual({ name: 'Jane' });
  });
});
```

## Test Patterns

### Arrange-Act-Assert
```typescript
it('calculates total correctly', () => {
  // Arrange
  const items = [{ price: 10 }, { price: 20 }];

  // Act
  const result = calculateTotal(items);

  // Assert
  expect(result).toBe(30);
});
```

### Testing Async Code
```typescript
it('fetches user data', async () => {
  const user = await fetchUser('123');
  expect(user.name).toBe('John');
});

it('handles fetch errors', async () => {
  await expect(fetchUser('invalid')).rejects.toThrow('User not found');
});
```

### Mocking
```typescript
import { vi } from 'vitest';

// Mock module
vi.mock('./api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ name: 'John' }),
}));

// Mock function
const mockFn = vi.fn().mockReturnValue(42);

// Spy on method
const spy = vi.spyOn(object, 'method');
```

## Quality Checklist

- [ ] Tests are isolated (no shared state)
- [ ] Each test has one assertion focus
- [ ] Edge cases covered
- [ ] Error cases tested
- [ ] Async code handled properly
- [ ] Mocks cleaned up between tests
- [ ] Test names are descriptive
- [ ] No implementation details tested
