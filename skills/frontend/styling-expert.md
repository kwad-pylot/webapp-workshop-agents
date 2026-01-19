---
name: styling-expert
description: |
  Apply modern CSS styling using Tailwind, CSS Modules, or CSS-in-JS.
  Creates consistent, maintainable styles.
---

# Styling Expert Skill

## Purpose
Implement consistent, maintainable styling using modern CSS approaches.

## Tailwind CSS

### Configuration
```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'class',
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      fontFamily: {
        sans: ['var(--font-sans)', 'system-ui', 'sans-serif'],
        mono: ['var(--font-mono)', 'monospace'],
      },
      keyframes: {
        'fade-in': {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        'slide-in': {
          '0%': { transform: 'translateY(-10px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
      },
      animation: {
        'fade-in': 'fade-in 0.2s ease-out',
        'slide-in': 'slide-in 0.3s ease-out',
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
};
```

### CSS Variables
```css
/* globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
  }
}
```

### Utility Function
```ts
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Common Patterns

#### Layout
```tsx
// Flex container
<div className="flex items-center justify-between gap-4">

// Grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// Centering
<div className="flex min-h-screen items-center justify-center">

// Container
<div className="container mx-auto px-4 md:px-6 lg:px-8">
```

#### Cards
```tsx
<div className="rounded-lg border bg-card p-6 shadow-sm hover:shadow-md transition-shadow">
  <h3 className="text-lg font-semibold">Title</h3>
  <p className="mt-2 text-sm text-muted-foreground">Description</p>
</div>
```

#### Buttons
```tsx
// Primary
<button className="inline-flex items-center justify-center rounded-md bg-primary px-4 py-2 text-sm font-medium text-primary-foreground hover:bg-primary/90 focus-visible:outline-none focus-visible:ring-2">

// Secondary
<button className="inline-flex items-center justify-center rounded-md border bg-background px-4 py-2 text-sm font-medium hover:bg-accent">

// Ghost
<button className="inline-flex items-center justify-center rounded-md px-4 py-2 text-sm font-medium hover:bg-accent">
```

#### Forms
```tsx
<div className="space-y-4">
  <div className="space-y-2">
    <label className="text-sm font-medium leading-none">Email</label>
    <input className="flex h-10 w-full rounded-md border bg-background px-3 py-2 text-sm placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring" />
  </div>
</div>
```

## CSS Modules

```css
/* Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 0.375rem;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 500;
  transition: colors 0.2s;
}

.primary {
  background-color: var(--color-primary);
  color: white;
}

.primary:hover {
  background-color: var(--color-primary-dark);
}

.secondary {
  background-color: var(--color-secondary);
  color: var(--color-secondary-foreground);
}
```

```tsx
import styles from './Button.module.css';

function Button({ variant = 'primary', children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

## CSS-in-JS (styled-components)

```tsx
import styled from 'styled-components';

const Button = styled.button<{ variant?: 'primary' | 'secondary' }>`
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  border-radius: 0.375rem;
  font-weight: 500;
  transition: all 0.2s;

  ${({ variant }) =>
    variant === 'primary'
      ? `
        background-color: var(--color-primary);
        color: white;
        &:hover {
          background-color: var(--color-primary-dark);
        }
      `
      : `
        background-color: var(--color-secondary);
        color: var(--color-foreground);
      `}
`;
```

## Animation Patterns

```tsx
// Tailwind animations
<div className="animate-fade-in">Fades in</div>
<div className="animate-slide-in">Slides in</div>
<div className="animate-spin">Spins</div>
<div className="animate-pulse">Pulses</div>

// Transition on hover
<div className="transition-transform hover:scale-105">
<div className="transition-colors hover:bg-accent">
<div className="transition-shadow hover:shadow-lg">
```

## Quality Checklist

- [ ] Consistent spacing scale
- [ ] Dark mode support
- [ ] Responsive breakpoints
- [ ] Accessible color contrast
- [ ] Smooth transitions
- [ ] Focus states visible
- [ ] No hardcoded colors
- [ ] Uses design tokens
