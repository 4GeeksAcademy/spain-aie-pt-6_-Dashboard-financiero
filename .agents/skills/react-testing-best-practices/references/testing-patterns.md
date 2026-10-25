# Advanced React Testing Patterns

Reference for less common but important testing scenarios.

---

## Snapshot Testing

Use snapshots sparingly — only for stable, intentionally static UI.

```tsx
it('matches snapshot', () => {
  const { container } = render(<Badge variant="success">Active</Badge>)
  expect(container.firstChild).toMatchSnapshot()
})
```

**Rules:**
- Commit snapshots to version control
- Update with `vitest --update-snapshots` only when intentional
- Prefer inline snapshots (`toMatchInlineSnapshot`) for small components
- Never snapshot entire page layouts — too brittle

---

## Testing React Router Navigation

```tsx
import { MemoryRouter, Route, Routes } from 'react-router-dom'

const renderWithRouter = (ui, { initialEntries = ['/'] } = {}) => {
  return render(
    <MemoryRouter initialEntries={initialEntries}>
      {ui}
    </MemoryRouter>
  )
}

it('navigates to profile on click', async () => {
  const user = userEvent.setup()

  renderWithRouter(
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/profile" element={<div>Profile Page</div>} />
    </Routes>
  )

  await user.click(screen.getByRole('link', { name: /profile/i }))
  expect(screen.getByText('Profile Page')).toBeInTheDocument()
})
```

---

## Testing with React Query / TanStack Query

Wrap with a fresh `QueryClient` per test to avoid cache pollution:

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },       // Don't retry on failure in tests
      mutations: { retry: false },
    },
  })

const renderWithQuery = (ui) => {
  const queryClient = createTestQueryClient()
  return render(
    <QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>
  )
}

it('displays fetched data', async () => {
  // MSW will intercept the fetch made by React Query
  renderWithQuery(<UserProfile userId="1" />)

  expect(await screen.findByText('Alice')).toBeInTheDocument()
})
```

---

## Testing Drag and Drop

For libraries like `@dnd-kit` or `react-beautiful-dnd`, simulate pointer events:

```tsx
it('reorders items via drag', async () => {
  const user = userEvent.setup()
  render(<SortableList items={['A', 'B', 'C']} />)

  const itemA = screen.getByText('A')
  const itemC = screen.getByText('C')

  // Simulate drag via pointer events
  await user.pointer([
    { keys: '[MouseLeft>]', target: itemA },
    { target: itemC },
    { keys: '[/MouseLeft]' },
  ])

  // Verify new order
  const items = screen.getAllByRole('listitem')
  expect(items[0]).toHaveTextContent('B')
  expect(items[2]).toHaveTextContent('A')
})
```

For complex DnD, consider Playwright E2E instead of unit tests.

---

## Visual Regression Testing with Playwright

For critical UI stability across deployments:

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test'

export default defineConfig({
  testMatch: '**/*.visual.ts',
  use: { baseURL: 'http://localhost:5173' },
})

// Button.visual.ts
import { test, expect } from '@playwright/test'

test('Button variants look correct', async ({ page }) => {
  await page.goto('/storybook/iframe.html?id=button--primary')
  await expect(page).toHaveScreenshot('button-primary.png', {
    maxDiffPixelRatio: 0.02,
  })
})
```

Run visual tests in CI only; update screenshots intentionally:
```bash
npx playwright test --update-snapshots
```

---

## Testing Error Boundaries

```tsx
// Suppress expected console.error in tests
beforeEach(() => {
  vi.spyOn(console, 'error').mockImplementation(() => {})
})

afterEach(() => {
  vi.restoreAllMocks()
})

const ThrowError = ({ shouldThrow }) => {
  if (shouldThrow) throw new Error('Test error')
  return <div>No error</div>
}

it('renders fallback UI on error', () => {
  render(
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <ThrowError shouldThrow />
    </ErrorBoundary>
  )
  expect(screen.getByText('Something went wrong')).toBeInTheDocument()
})
```

---

## Coverage Configuration

```ts
// vite.config.ts
test: {
  coverage: {
    provider: 'v8',
    reporter: ['text', 'lcov', 'html'],
    exclude: [
      'src/test/**',
      'src/**/*.stories.*',
      'src/main.tsx',
    ],
    thresholds: {
      lines: 80,
      functions: 80,
      branches: 75,
    },
  },
}
```

Run: `npx vitest --coverage`

---

## CI Integration (GitHub Actions)

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run test -- --coverage
      - uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
```
