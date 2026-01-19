---
name: responsive-design
description: |
  Implement responsive layouts that work across all device sizes.
  Mobile-first approach with progressive enhancement.
---

# Responsive Design Skill

## Purpose
Create layouts that adapt beautifully to any screen size.

## Mobile-First Approach

Design for mobile first, then enhance for larger screens.

### Tailwind Breakpoints
| Prefix | Min Width | Target |
|--------|-----------|--------|
| (none) | 0px | Mobile (default) |
| `sm:` | 640px | Large phones |
| `md:` | 768px | Tablets |
| `lg:` | 1024px | Laptops |
| `xl:` | 1280px | Desktops |
| `2xl:` | 1536px | Large screens |

### Example Usage
```tsx
// Mobile-first: styles apply mobile and up, prefixed styles apply at that breakpoint and up
<div className="
  p-4           /* All sizes: padding 16px */
  md:p-6        /* Tablet+: padding 24px */
  lg:p-8        /* Laptop+: padding 32px */
">
```

## Layout Patterns

### Responsive Navigation
```tsx
function Navbar() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <nav className="bg-background border-b">
      <div className="container mx-auto px-4">
        <div className="flex h-16 items-center justify-between">
          {/* Logo */}
          <a href="/" className="text-xl font-bold">Logo</a>

          {/* Desktop Navigation */}
          <div className="hidden md:flex items-center gap-6">
            <a href="/products">Products</a>
            <a href="/about">About</a>
            <a href="/contact">Contact</a>
            <Button>Sign In</Button>
          </div>

          {/* Mobile Menu Button */}
          <button
            className="md:hidden p-2"
            onClick={() => setIsOpen(!isOpen)}
            aria-label="Toggle menu"
          >
            {isOpen ? <X /> : <Menu />}
          </button>
        </div>

        {/* Mobile Navigation */}
        {isOpen && (
          <div className="md:hidden py-4 border-t">
            <div className="flex flex-col gap-4">
              <a href="/products">Products</a>
              <a href="/about">About</a>
              <a href="/contact">Contact</a>
              <Button className="w-full">Sign In</Button>
            </div>
          </div>
        )}
      </div>
    </nav>
  );
}
```

### Responsive Grid
```tsx
// Card grid that adapts from 1 to 4 columns
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 md:gap-6">
  {items.map(item => (
    <Card key={item.id}>{item.title}</Card>
  ))}
</div>
```

### Sidebar Layout
```tsx
function DashboardLayout({ children }) {
  const [sidebarOpen, setSidebarOpen] = useState(false);

  return (
    <div className="min-h-screen">
      {/* Mobile sidebar overlay */}
      {sidebarOpen && (
        <div
          className="fixed inset-0 z-40 bg-black/50 lg:hidden"
          onClick={() => setSidebarOpen(false)}
        />
      )}

      {/* Sidebar */}
      <aside className={cn(
        "fixed inset-y-0 left-0 z-50 w-64 bg-background border-r transform transition-transform lg:translate-x-0 lg:static lg:z-auto",
        sidebarOpen ? "translate-x-0" : "-translate-x-full"
      )}>
        <div className="h-full overflow-y-auto p-4">
          <nav className="space-y-2">
            <a href="/dashboard">Dashboard</a>
            <a href="/projects">Projects</a>
            <a href="/settings">Settings</a>
          </nav>
        </div>
      </aside>

      {/* Main content */}
      <div className="lg:pl-64">
        {/* Mobile header */}
        <header className="sticky top-0 z-30 flex h-16 items-center gap-4 border-b bg-background px-4 lg:hidden">
          <button onClick={() => setSidebarOpen(true)}>
            <Menu className="h-6 w-6" />
          </button>
          <span className="font-semibold">Dashboard</span>
        </header>

        <main className="p-4 md:p-6 lg:p-8">
          {children}
        </main>
      </div>
    </div>
  );
}
```

### Responsive Table
```tsx
function ResponsiveTable({ data }) {
  return (
    <>
      {/* Desktop table */}
      <div className="hidden md:block overflow-x-auto">
        <table className="w-full">
          <thead>
            <tr className="border-b">
              <th className="px-4 py-3 text-left">Name</th>
              <th className="px-4 py-3 text-left">Email</th>
              <th className="px-4 py-3 text-left">Role</th>
              <th className="px-4 py-3 text-left">Actions</th>
            </tr>
          </thead>
          <tbody>
            {data.map(row => (
              <tr key={row.id} className="border-b">
                <td className="px-4 py-3">{row.name}</td>
                <td className="px-4 py-3">{row.email}</td>
                <td className="px-4 py-3">{row.role}</td>
                <td className="px-4 py-3">
                  <Button size="sm">Edit</Button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* Mobile cards */}
      <div className="md:hidden space-y-4">
        {data.map(row => (
          <div key={row.id} className="rounded-lg border p-4">
            <div className="font-medium">{row.name}</div>
            <div className="text-sm text-muted-foreground">{row.email}</div>
            <div className="mt-2 flex items-center justify-between">
              <span className="text-sm">{row.role}</span>
              <Button size="sm">Edit</Button>
            </div>
          </div>
        ))}
      </div>
    </>
  );
}
```

## Responsive Typography
```tsx
// Heading that scales with viewport
<h1 className="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
  Responsive Heading
</h1>

// Body text
<p className="text-sm md:text-base lg:text-lg">
  Body content that adapts
</p>
```

## Responsive Images
```tsx
// Next.js Image with responsive sizing
import Image from 'next/image';

<div className="relative aspect-video w-full">
  <Image
    src="/hero.jpg"
    alt="Hero"
    fill
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
    className="object-cover"
  />
</div>

// Picture element for different images
<picture>
  <source media="(min-width: 1024px)" srcSet="/hero-large.jpg" />
  <source media="(min-width: 640px)" srcSet="/hero-medium.jpg" />
  <img src="/hero-small.jpg" alt="Hero" className="w-full" />
</picture>
```

## Responsive Utilities
```tsx
// Show/hide based on screen size
<div className="block md:hidden">Mobile only</div>
<div className="hidden md:block">Tablet and up</div>
<div className="hidden lg:block">Desktop only</div>

// Different spacing
<div className="p-4 md:p-6 lg:p-8">

// Different flex direction
<div className="flex flex-col md:flex-row">

// Different widths
<div className="w-full md:w-1/2 lg:w-1/3">
```

## Testing Responsive Designs

### Breakpoint Indicator (Dev Only)
```tsx
function BreakpointIndicator() {
  if (process.env.NODE_ENV !== 'development') return null;

  return (
    <div className="fixed bottom-4 left-4 z-50 rounded bg-black px-2 py-1 text-xs text-white">
      <span className="sm:hidden">xs</span>
      <span className="hidden sm:inline md:hidden">sm</span>
      <span className="hidden md:inline lg:hidden">md</span>
      <span className="hidden lg:inline xl:hidden">lg</span>
      <span className="hidden xl:inline 2xl:hidden">xl</span>
      <span className="hidden 2xl:inline">2xl</span>
    </div>
  );
}
```

## Quality Checklist

- [ ] Works on 320px width (small phones)
- [ ] Touch targets at least 44px
- [ ] No horizontal scroll on mobile
- [ ] Readable text without zoom
- [ ] Images don't break layout
- [ ] Forms usable on mobile
- [ ] Navigation accessible on all sizes
- [ ] Critical content visible first
