---
name: component-builder
description: |
  Build reusable, accessible UI components following best practices.
  Creates well-structured React/Vue components with TypeScript.
---

# Component Builder Skill

## Purpose
Create reusable, maintainable, and accessible UI components.

## Component Structure

### File Organization
```
components/
├── Button/
│   ├── Button.tsx          # Main component
│   ├── Button.test.tsx     # Tests
│   ├── Button.stories.tsx  # Storybook (optional)
│   └── index.ts            # Export
├── Card/
│   ├── Card.tsx
│   ├── CardHeader.tsx
│   ├── CardBody.tsx
│   └── index.ts
└── ui/                      # Base UI primitives
    ├── input.tsx
    ├── label.tsx
    └── ...
```

### Component Template

```tsx
// components/Button/Button.tsx
import { forwardRef, ButtonHTMLAttributes, ReactNode } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

// Define variants using cva
const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

// Props interface
export interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  /** Show loading spinner */
  loading?: boolean;
  /** Icon to show before text */
  leftIcon?: ReactNode;
  /** Icon to show after text */
  rightIcon?: ReactNode;
}

// Component with forwardRef for ref forwarding
export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      className,
      variant,
      size,
      loading = false,
      leftIcon,
      rightIcon,
      disabled,
      children,
      ...props
    },
    ref
  ) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size, className }))}
        disabled={disabled || loading}
        {...props}
      >
        {loading && (
          <svg
            className="mr-2 h-4 w-4 animate-spin"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
            />
          </svg>
        )}
        {!loading && leftIcon && <span className="mr-2">{leftIcon}</span>}
        {children}
        {rightIcon && <span className="ml-2">{rightIcon}</span>}
      </button>
    );
  }
);

Button.displayName = 'Button';

// Export variants for external use
export { buttonVariants };
```

### Index Export
```tsx
// components/Button/index.ts
export { Button, buttonVariants } from './Button';
export type { ButtonProps } from './Button';
```

## Component Patterns

### Compound Components
```tsx
// components/Card/index.tsx
import { createContext, useContext, ReactNode } from 'react';

interface CardContextValue {
  variant: 'default' | 'elevated';
}

const CardContext = createContext<CardContextValue | null>(null);

function useCardContext() {
  const context = useContext(CardContext);
  if (!context) throw new Error('Card components must be used within Card');
  return context;
}

interface CardProps {
  children: ReactNode;
  variant?: 'default' | 'elevated';
  className?: string;
}

export function Card({ children, variant = 'default', className }: CardProps) {
  return (
    <CardContext.Provider value={{ variant }}>
      <div className={cn('rounded-lg border bg-card p-6', className)}>
        {children}
      </div>
    </CardContext.Provider>
  );
}

export function CardHeader({ children, className }: { children: ReactNode; className?: string }) {
  return <div className={cn('mb-4', className)}>{children}</div>;
}

export function CardTitle({ children, className }: { children: ReactNode; className?: string }) {
  return <h3 className={cn('text-lg font-semibold', className)}>{children}</h3>;
}

export function CardContent({ children, className }: { children: ReactNode; className?: string }) {
  return <div className={cn('text-sm text-muted-foreground', className)}>{children}</div>;
}

// Usage:
// <Card>
//   <CardHeader>
//     <CardTitle>Title</CardTitle>
//   </CardHeader>
//   <CardContent>Content here</CardContent>
// </Card>
```

### Controlled vs Uncontrolled
```tsx
interface InputProps {
  // Controlled
  value?: string;
  onChange?: (value: string) => void;
  // Uncontrolled
  defaultValue?: string;
}

function Input({ value, onChange, defaultValue }: InputProps) {
  // Internal state for uncontrolled mode
  const [internalValue, setInternalValue] = useState(defaultValue ?? '');

  // Use controlled value if provided, otherwise internal
  const isControlled = value !== undefined;
  const currentValue = isControlled ? value : internalValue;

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    if (!isControlled) {
      setInternalValue(newValue);
    }
    onChange?.(newValue);
  };

  return <input value={currentValue} onChange={handleChange} />;
}
```

## Accessibility Requirements

### Keyboard Navigation
```tsx
// Handle keyboard events
const handleKeyDown = (e: KeyboardEvent) => {
  switch (e.key) {
    case 'Enter':
    case ' ':
      e.preventDefault();
      onClick?.();
      break;
    case 'Escape':
      onClose?.();
      break;
  }
};
```

### ARIA Attributes
```tsx
<button
  role="button"
  aria-label={ariaLabel}
  aria-pressed={isPressed}
  aria-disabled={disabled}
  aria-expanded={isExpanded}
  aria-haspopup={hasPopup}
>
```

### Focus Management
```tsx
// Focus trap for modals
import { FocusTrap } from '@headlessui/react';

function Modal({ open, children }) {
  return (
    <Dialog open={open}>
      <FocusTrap>
        {children}
      </FocusTrap>
    </Dialog>
  );
}
```

## Quality Checklist

- [ ] TypeScript types for all props
- [ ] Supports ref forwarding
- [ ] Has displayName
- [ ] Handles disabled state
- [ ] Handles loading state (if applicable)
- [ ] Keyboard accessible
- [ ] Screen reader friendly
- [ ] Supports className override
- [ ] Has sensible defaults
- [ ] Documented with JSDoc
