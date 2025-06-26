# React Testing - Senior Level Interview Questions

## 1. Advanced Testing Architecture: Custom Testing Utilities and Test Organization

**Q: Design a comprehensive testing architecture for a large-scale React application. Implement custom testing utilities, test data factories, and a strategy for testing complex business logic, API integrations, and user interactions. Explain your testing pyramid approach and how you ensure test maintainability.**

### Answer
A robust testing architecture requires custom utilities, proper test organization, and strategies for testing complex scenarios. This solution demonstrates enterprise-level testing practices.

**Custom Testing Utilities and Test Setup:**
```tsx
// test-utils/test-utils.tsx
import React, { ReactElement } from 'react';
import { render, RenderOptions, RenderResult } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { ThemeProvider } from '@mui/material/styles';
import { theme } from '../src/theme';
import { rootReducer } from '../src/store/rootReducer';

// Custom render function that includes all providers
interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  initialState?: any;
  route?: string;
  withRouter?: boolean;
  withRedux?: boolean;
  withTheme?: boolean;
}

// Test data factories for consistent test data
export const createUser = (overrides: Partial<User> = {}): User => ({
  id: 'user-1',
  name: 'John Doe',
  email: 'john@example.com',
  role: 'user',
  permissions: ['read'],
  createdAt: new Date('2023-01-01'),
  ...overrides
});

export const createProduct = (overrides: Partial<Product> = {}): Product => ({
  id: 'product-1',
  name: 'Test Product',
  price: 99.99,
  category: 'electronics',
  inStock: true,
  ...overrides
});

export const createOrder = (overrides: Partial<Order> = {}): Order => ({
  id: 'order-1',
  userId: 'user-1',
  products: [createProduct()],
  total: 99.99,
  status: 'pending',
  createdAt: new Date('2023-01-01'),
  ...overrides
});

// Custom render function with providers
export const customRender = (
  ui: ReactElement,
  options: CustomRenderOptions = {}
): RenderResult => {
  const {
    initialState = {},
    route = '/',
    withRouter = true,
    withRedux = true,
    withTheme = true,
    ...renderOptions
  } = options;

  // Create test store with initial state
  const store = withRedux 
    ? configureStore({
        reducer: rootReducer,
        preloadedState: initialState,
        middleware: (getDefaultMiddleware) =>
          getDefaultMiddleware({
            serializableCheck: false, // Disable for testing
          }),
      })
    : null;

  // Wrapper component that includes all necessary providers
  const AllTheProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => {
    let element = children;

    // Add theme provider
    if (withTheme) {
      element = <ThemeProvider theme={theme}>{element}</ThemeProvider>;
    }

    // Add Redux provider
    if (withRedux && store) {
      element = <Provider store={store}>{element}</Provider>;
    }

    // Add router provider
    if (withRouter) {
      element = (
        <BrowserRouter>
          {element}
        </BrowserRouter>
      );
    }

    return <>{element}</>;
  };

  // Set up initial route if provided
  if (withRouter && route !== '/') {
    window.history.pushState({}, 'Test page', route);
  }

  return render(ui, { wrapper: AllTheProviders, ...renderOptions });
};

// Custom hooks for testing
export const useTestStore = () => {
  const store = configureStore({
    reducer: rootReducer,
    middleware: (getDefaultMiddleware) =>
      getDefaultMiddleware({
        serializableCheck: false,
      }),
  });
  return store;
};

// Mock API utilities
export const mockApiResponse = <T>(data: T, delay = 100): Promise<T> => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(data), delay);
  });
};

export const mockApiError = (message: string, status = 500): Promise<never> => {
  return Promise.reject(new Error(message));
};

// Test helpers for common scenarios
export const waitForLoadingToFinish = async () => {
  await waitFor(() => {
    expect(screen.queryByTestId('loading-spinner')).not.toBeInTheDocument();
  });
};

export const waitForErrorToAppear = async (errorMessage: string) => {
  await waitFor(() => {
    expect(screen.getByText(errorMessage)).toBeInTheDocument();
  });
};

// Custom matchers for better test assertions
export const expectElementToBeVisible = (element: HTMLElement) => {
  expect(element).toBeInTheDocument();
  expect(element).toBeVisible();
};

export const expectElementToHaveText = (element: HTMLElement, text: string) => {
  expect(element).toHaveTextContent(text);
};

// Re-export everything from testing library
export * from '@testing-library/react';
export { default as userEvent } from '@testing-library/user-event';
```

**Complex Component Testing Example:**
```tsx
// __tests__/components/UserManagement.test.tsx
import React from 'react';
import { screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { customRender, createUser, createProduct, mockApiResponse } from '../../test-utils/test-utils';
import { UserManagement } from '../../components/UserManagement';
import { userApi } from '../../services/userApi';

// Mock the API service
jest.mock('../../services/userApi');
const mockedUserApi = userApi as jest.Mocked<typeof userApi>;

describe('UserManagement Component', () => {
  const user = userEvent.setup();

  beforeEach(() => {
    // Clear all mocks before each test
    jest.clearAllMocks();
  });

  describe('User List Display', () => {
    it('should display users with pagination and search functionality', async () => {
      // Arrange: Set up test data and API mocks
      const testUsers = [
        createUser({ id: '1', name: 'Alice Johnson', email: 'alice@example.com' }),
        createUser({ id: '2', name: 'Bob Smith', email: 'bob@example.com' }),
        createUser({ id: '3', name: 'Charlie Brown', email: 'charlie@example.com' })
      ];

      mockedUserApi.getUsers.mockResolvedValue({
        users: testUsers,
        total: 3,
        page: 1,
        limit: 10
      });

      // Act: Render component
      customRender(<UserManagement />, {
        initialState: {
          auth: { user: createUser({ role: 'admin' }) }
        }
      });

      // Assert: Check if users are displayed
      await waitFor(() => {
        expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
        expect(screen.getByText('Bob Smith')).toBeInTheDocument();
        expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
      });

      // Test search functionality
      const searchInput = screen.getByPlaceholderText('Search users...');
      await user.type(searchInput, 'Alice');

      await waitFor(() => {
        expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
        expect(screen.queryByText('Bob Smith')).not.toBeInTheDocument();
      });
    });

    it('should handle API errors gracefully', async () => {
      // Arrange: Mock API error
      mockedUserApi.getUsers.mockRejectedValue(new Error('Failed to fetch users'));

      // Act: Render component
      customRender(<UserManagement />);

      // Assert: Check error message
      await waitFor(() => {
        expect(screen.getByText('Failed to fetch users')).toBeInTheDocument();
        expect(screen.getByText('Retry')).toBeInTheDocument();
      });
    });
  });

  describe('User Actions', () => {
    it('should allow editing user information', async () => {
      // Arrange
      const testUser = createUser();
      mockedUserApi.getUsers.mockResolvedValue({
        users: [testUser],
        total: 1,
        page: 1,
        limit: 10
      });

      mockedUserApi.updateUser.mockResolvedValue({
        ...testUser,
        name: 'Updated Name'
      });

      customRender(<UserManagement />);

      // Wait for user to load
      await waitFor(() => {
        expect(screen.getByText(testUser.name)).toBeInTheDocument();
      });

      // Act: Click edit button
      const editButton = screen.getByTestId(`edit-user-${testUser.id}`);
      await user.click(editButton);

      // Assert: Check if edit modal opens
      expect(screen.getByText('Edit User')).toBeInTheDocument();

      // Act: Update user name
      const nameInput = screen.getByLabelText('Name');
      await user.clear(nameInput);
      await user.type(nameInput, 'Updated Name');

      // Submit form
      const saveButton = screen.getByText('Save');
      await user.click(saveButton);

      // Assert: Check if API was called and UI updated
      await waitFor(() => {
        expect(mockedUserApi.updateUser).toHaveBeenCalledWith(testUser.id, {
          ...testUser,
          name: 'Updated Name'
        });
        expect(screen.getByText('Updated Name')).toBeInTheDocument();
      });
    });

    it('should confirm before deleting users', async () => {
      // Arrange
      const testUser = createUser();
      mockedUserApi.getUsers.mockResolvedValue({
        users: [testUser],
        total: 1,
        page: 1,
        limit: 10
      });

      mockedUserApi.deleteUser.mockResolvedValue(undefined);

      customRender(<UserManagement />);

      await waitFor(() => {
        expect(screen.getByText(testUser.name)).toBeInTheDocument();
      });

      // Act: Click delete button
      const deleteButton = screen.getByTestId(`delete-user-${testUser.id}`);
      await user.click(deleteButton);

      // Assert: Check confirmation dialog
      expect(screen.getByText('Are you sure you want to delete this user?')).toBeInTheDocument();

      // Act: Confirm deletion
      const confirmButton = screen.getByText('Delete');
      await user.click(confirmButton);

      // Assert: Check if API was called
      await waitFor(() => {
        expect(mockedUserApi.deleteUser).toHaveBeenCalledWith(testUser.id);
      });
    });
  });

  describe('Access Control', () => {
    it('should hide admin actions for non-admin users', async () => {
      // Arrange: Render with non-admin user
      const regularUser = createUser({ role: 'user' });
      
      customRender(<UserManagement />, {
        initialState: {
          auth: { user: regularUser }
        }
      });

      // Assert: Check that admin actions are not visible
      expect(screen.queryByTestId('add-user-button')).not.toBeInTheDocument();
      expect(screen.queryByTestId('bulk-actions')).not.toBeInTheDocument();
    });

    it('should show admin actions for admin users', async () => {
      // Arrange: Render with admin user
      const adminUser = createUser({ role: 'admin' });
      
      customRender(<UserManagement />, {
        initialState: {
          auth: { user: adminUser }
        }
      });

      // Assert: Check that admin actions are visible
      expect(screen.getByTestId('add-user-button')).toBeInTheDocument();
      expect(screen.getByTestId('bulk-actions')).toBeInTheDocument();
    });
  });
});
```

---

## 2. Integration Testing: API Testing and State Management

**Q: Implement comprehensive integration tests for React components that interact with APIs, Redux state management, and external services. Design a testing strategy that covers happy paths, error scenarios, loading states, and edge cases while maintaining test isolation and performance.**

### Answer
Integration testing requires careful setup of external dependencies, proper mocking strategies, and testing of complex interactions between components and services.

**API Integration Testing Setup:**
```tsx
// __tests__/integration/api-integration.test.tsx
import React from 'react';
import { screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { customRender, createUser, createProduct } from '../../test-utils/test-utils';
import { ProductManagement } from '../../components/ProductManagement';
import { store } from '../../store/store';

// Set up MSW server for API mocking
const server = setupServer(
  // Mock GET /api/products endpoint
  rest.get('/api/products', (req, res, ctx) => {
    const page = req.url.searchParams.get('page') || '1';
    const limit = req.url.searchParams.get('limit') || '10';
    const search = req.url.searchParams.get('search') || '';

    const products = [
      createProduct({ id: '1', name: 'Laptop', price: 999.99 }),
      createProduct({ id: '2', name: 'Mouse', price: 29.99 }),
      createProduct({ id: '3', name: 'Keyboard', price: 89.99 })
    ].filter(product => 
      search ? product.name.toLowerCase().includes(search.toLowerCase()) : true
    );

    return res(
      ctx.json({
        products: products.slice((Number(page) - 1) * Number(limit), Number(page) * Number(limit)),
        total: products.length,
        page: Number(page),
        limit: Number(limit)
      })
    );
  }),

  // Mock POST /api/products endpoint
  rest.post('/api/products', async (req, res, ctx) => {
    const productData = await req.json();
    const newProduct = {
      ...productData,
      id: Date.now().toString(),
      createdAt: new Date().toISOString()
    };

    return res(ctx.json(newProduct));
  }),

  // Mock PUT /api/products/:id endpoint
  rest.put('/api/products/:id', async (req, res, ctx) => {
    const { id } = req.params;
    const updateData = await req.json();
    
    return res(ctx.json({
      id,
      ...updateData,
      updatedAt: new Date().toISOString()
    }));
  }),

  // Mock DELETE /api/products/:id endpoint
  rest.delete('/api/products/:id', (req, res, ctx) => {
    return res(ctx.status(204));
  })
);

// Start server before all tests
beforeAll(() => server.listen());

// Reset handlers after each test
afterEach(() => {
  server.resetHandlers();
  // Clear Redux store state
  store.dispatch({ type: 'RESET_STATE' });
});

// Close server after all tests
afterAll(() => server.close());

describe('Product Management Integration Tests', () => {
  const user = userEvent.setup();

  describe('Product CRUD Operations', () => {
    it('should create, read, update, and delete products successfully', async () => {
      // Arrange: Render component
      customRender(<ProductManagement />, {
        withRedux: true,
        initialState: {
          auth: { user: createUser({ role: 'admin' }) }
        }
      });

      // Wait for initial load
      await waitFor(() => {
        expect(screen.getByText('Laptop')).toBeInTheDocument();
      });

      // Test CREATE operation
      const addButton = screen.getByTestId('add-product-button');
      await user.click(addButton);

      // Fill out product form
      const nameInput = screen.getByLabelText('Product Name');
      const priceInput = screen.getByLabelText('Price');
      const categoryInput = screen.getByLabelText('Category');

      await user.type(nameInput, 'New Product');
      await user.type(priceInput, '149.99');
      await user.selectOptions(categoryInput, 'electronics');

      // Submit form
      const saveButton = screen.getByText('Save Product');
      await user.click(saveButton);

      // Assert: Check if product was created
      await waitFor(() => {
        expect(screen.getByText('New Product')).toBeInTheDocument();
        expect(screen.getByText('$149.99')).toBeInTheDocument();
      });

      // Test UPDATE operation
      const editButton = screen.getByTestId('edit-product-New Product');
      await user.click(editButton);

      // Update product name
      const editNameInput = screen.getByDisplayValue('New Product');
      await user.clear(editNameInput);
      await user.type(editNameInput, 'Updated Product');

      const updateButton = screen.getByText('Update Product');
      await user.click(updateButton);

      // Assert: Check if product was updated
      await waitFor(() => {
        expect(screen.getByText('Updated Product')).toBeInTheDocument();
        expect(screen.queryByText('New Product')).not.toBeInTheDocument();
      });

      // Test DELETE operation
      const deleteButton = screen.getByTestId('delete-product-Updated Product');
      await user.click(deleteButton);

      // Confirm deletion
      const confirmButton = screen.getByText('Delete');
      await user.click(confirmButton);

      // Assert: Check if product was deleted
      await waitFor(() => {
        expect(screen.queryByText('Updated Product')).not.toBeInTheDocument();
      });
    });

    it('should handle API errors gracefully', async () => {
      // Arrange: Override default handler to simulate error
      server.use(
        rest.get('/api/products', (req, res, ctx) => {
          return res(ctx.status(500), ctx.json({ message: 'Internal server error' }));
        })
      );

      // Act: Render component
      customRender(<ProductManagement />);

      // Assert: Check error handling
      await waitFor(() => {
        expect(screen.getByText('Internal server error')).toBeInTheDocument();
        expect(screen.getByText('Retry')).toBeInTheDocument();
      });
    });

    it('should handle network timeouts', async () => {
      // Arrange: Simulate slow network
      server.use(
        rest.get('/api/products', async (req, res, ctx) => {
          await new Promise(resolve => setTimeout(resolve, 5000)); // 5 second delay
          return res(ctx.json({ products: [], total: 0 }));
        })
      );

      // Act: Render component
      customRender(<ProductManagement />);

      // Assert: Check loading state
      expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();

      // Wait for timeout (if implemented)
      await waitFor(() => {
        expect(screen.getByText('Request timeout')).toBeInTheDocument();
      }, { timeout: 6000 });
    });
  });

  describe('Search and Filtering', () => {
    it('should filter products by search term', async () => {
      // Arrange: Render component
      customRender(<ProductManagement />);

      await waitFor(() => {
        expect(screen.getByText('Laptop')).toBeInTheDocument();
      });

      // Act: Search for specific product
      const searchInput = screen.getByPlaceholderText('Search products...');
      await user.type(searchInput, 'Laptop');

      // Assert: Check filtered results
      await waitFor(() => {
        expect(screen.getByText('Laptop')).toBeInTheDocument();
        expect(screen.queryByText('Mouse')).not.toBeInTheDocument();
        expect(screen.queryByText('Keyboard')).not.toBeInTheDocument();
      });
    });

    it('should handle pagination correctly', async () => {
      // Arrange: Mock paginated response
      server.use(
        rest.get('/api/products', (req, res, ctx) => {
          const page = req.url.searchParams.get('page') || '1';
          
          const allProducts = Array.from({ length: 25 }, (_, i) => 
            createProduct({ 
              id: String(i + 1), 
              name: `Product ${i + 1}` 
            })
          );

          const startIndex = (Number(page) - 1) * 10;
          const endIndex = startIndex + 10;

          return res(ctx.json({
            products: allProducts.slice(startIndex, endIndex),
            total: 25,
            page: Number(page),
            limit: 10
          }));
        })
      );

      // Act: Render component
      customRender(<ProductManagement />);

      await waitFor(() => {
        expect(screen.getByText('Product 1')).toBeInTheDocument();
      });

      // Act: Navigate to next page
      const nextPageButton = screen.getByTestId('next-page-button');
      await user.click(nextPageButton);

      // Assert: Check if page 2 products are displayed
      await waitFor(() => {
        expect(screen.getByText('Product 11')).toBeInTheDocument();
        expect(screen.queryByText('Product 1')).not.toBeInTheDocument();
      });
    });
  });

  describe('State Management Integration', () => {
    it('should sync Redux state with API responses', async () => {
      // Arrange: Render component with Redux
      const { store } = customRender(<ProductManagement />, { withRedux: true });

      await waitFor(() => {
        expect(screen.getByText('Laptop')).toBeInTheDocument();
      });

      // Assert: Check if Redux state is updated
      const state = store.getState();
      expect(state.products.items).toHaveLength(3);
      expect(state.products.loading).toBe(false);
      expect(state.products.error).toBeNull();
    });

    it('should handle optimistic updates', async () => {
      // Arrange: Render component
      customRender(<ProductManagement />, { withRedux: true });

      await waitFor(() => {
        expect(screen.getByText('Laptop')).toBeInTheDocument();
      });

      // Act: Add product with optimistic update
      const addButton = screen.getByTestId('add-product-button');
      await user.click(addButton);

      const nameInput = screen.getByLabelText('Product Name');
      await user.type(nameInput, 'Optimistic Product');

      const saveButton = screen.getByText('Save Product');
      await user.click(saveButton);

      // Assert: Check optimistic update (immediate UI update)
      expect(screen.getByText('Optimistic Product')).toBeInTheDocument();

      // Wait for API response
      await waitFor(() => {
        expect(screen.getByText('Optimistic Product')).toBeInTheDocument();
      });
    });
  });
});
```

---

## 3. E2E Testing with Playwright and Visual Regression Testing

**Q: Design an end-to-end testing strategy using Playwright for a complex React application. Implement visual regression testing, accessibility testing, and performance testing. Explain how you would handle test data management, parallel execution, and CI/CD integration.**

### Answer
E2E testing requires comprehensive coverage of user workflows, visual consistency checks, and performance monitoring. This solution demonstrates enterprise-level E2E testing practices.

**Playwright E2E Testing Setup:**
```typescript
// tests/e2e/setup/global-setup.ts
import { chromium, FullConfig } from '@playwright/test';
import { createTestUser, cleanupTestData } from '../utils/test-data';

async function globalSetup(config: FullConfig) {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  // Set up test environment
  await page.goto('http://localhost:3000');
  
  // Create test user for E2E tests
  const testUser = await createTestUser({
    email: 'e2e-test@example.com',
    password: 'testpassword123',
    role: 'admin'
  });

  // Store authentication state
  await page.context().storageState({ path: 'playwright/.auth/user.json' });

  await browser.close();
}

export default globalSetup;
```

**Main E2E Test Suite:**
```typescript
// tests/e2e/user-management.spec.ts
import { test, expect } from '@playwright/test';
import { createTestUser, deleteTestUser } from './utils/test-data';

test.describe('User Management E2E Tests', () => {
  let testUser: any;

  test.beforeEach(async ({ page }) => {
    // Navigate to user management page
    await page.goto('/admin/users');
    
    // Wait for page to load
    await page.waitForSelector('[data-testid="user-management-table"]');
  });

  test('should create, edit, and delete users through complete workflow', async ({ page }) => {
    // Test CREATE user workflow
    await test.step('Create new user', async () => {
      // Click add user button
      await page.click('[data-testid="add-user-button"]');
      
      // Fill out user form
      await page.fill('[data-testid="user-name-input"]', 'John Doe');
      await page.fill('[data-testid="user-email-input"]', 'john.doe@example.com');
      await page.selectOption('[data-testid="user-role-select"]', 'manager');
      
      // Submit form
      await page.click('[data-testid="save-user-button"]');
      
      // Verify user was created
      await expect(page.locator('text=John Doe')).toBeVisible();
      await expect(page.locator('text=john.doe@example.com')).toBeVisible();
    });

    // Test EDIT user workflow
    await test.step('Edit existing user', async () => {
      // Click edit button for the created user
      await page.click('[data-testid="edit-user-john.doe@example.com"]');
      
      // Update user information
      await page.fill('[data-testid="user-name-input"]', 'John Smith');
      await page.selectOption('[data-testid="user-role-select"]', 'admin');
      
      // Save changes
      await page.click('[data-testid="update-user-button"]');
      
      // Verify changes were saved
      await expect(page.locator('text=John Smith')).toBeVisible();
      await expect(page.locator('text=admin')).toBeVisible();
    });

    // Test DELETE user workflow
    await test.step('Delete user', async () => {
      // Click delete button
      await page.click('[data-testid="delete-user-john.doe@example.com"]');
      
      // Confirm deletion
      await page.click('[data-testid="confirm-delete-button"]');
      
      // Verify user was deleted
      await expect(page.locator('text=John Smith')).not.toBeVisible();
    });
  });

  test('should handle bulk operations correctly', async ({ page }) => {
    // Select multiple users
    await page.click('[data-testid="select-all-users"]');
    
    // Perform bulk action
    await page.click('[data-testid="bulk-actions-menu"]');
    await page.click('[data-testid="bulk-delete-option"]');
    
    // Confirm bulk deletion
    await page.click('[data-testid="confirm-bulk-delete"]');
    
    // Verify all users were deleted
    await expect(page.locator('[data-testid="user-row"]')).toHaveCount(0);
  });

  test('should handle search and filtering', async ({ page }) => {
    // Search for specific user
    await page.fill('[data-testid="user-search-input"]', 'admin');
    
    // Verify filtered results
    await expect(page.locator('[data-testid="user-row"]')).toHaveCount(1);
    
    // Clear search
    await page.clear('[data-testid="user-search-input"]');
    
    // Verify all users are shown
    await expect(page.locator('[data-testid="user-row"]')).toHaveCount(3);
  });

  test('should handle pagination correctly', async ({ page }) => {
    // Navigate to next page
    await page.click('[data-testid="next-page-button"]');
    
    // Verify page change
    await expect(page.locator('[data-testid="current-page"]')).toHaveText('2');
    
    // Navigate back to first page
    await page.click('[data-testid="prev-page-button"]');
    
    // Verify back to first page
    await expect(page.locator('[data-testid="current-page"]')).toHaveText('1');
  });
});
```

**Visual Regression Testing:**
```typescript
// tests/e2e/visual-regression.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Regression Tests', () => {
  test('should maintain visual consistency across pages', async ({ page }) => {
    // Test dashboard page
    await page.goto('/dashboard');
    await page.waitForLoadState('networkidle');
    
    // Take screenshot and compare with baseline
    await expect(page).toHaveScreenshot('dashboard-page.png', {
      threshold: 0.1, // Allow 10% difference
      fullPage: true
    });

    // Test user management page
    await page.goto('/admin/users');
    await page.waitForLoadState('networkidle');
    
    await expect(page).toHaveScreenshot('user-management-page.png', {
      threshold: 0.1,
      fullPage: true
    });

    // Test responsive design
    await page.setViewportSize({ width: 768, height: 1024 });
    await expect(page).toHaveScreenshot('user-management-mobile.png', {
      threshold: 0.1,
      fullPage: true
    });
  });

  test('should handle dynamic content without visual regressions', async ({ page }) => {
    await page.goto('/admin/users');
    
    // Add a new user
    await page.click('[data-testid="add-user-button"]');
    await page.fill('[data-testid="user-name-input"]', 'Test User');
    await page.fill('[data-testid="user-email-input"]', 'test@example.com');
    
    // Take screenshot of form
    await expect(page).toHaveScreenshot('add-user-form.png', {
      threshold: 0.05
    });
    
    // Submit form and check result
    await page.click('[data-testid="save-user-button"]');
    await page.waitForSelector('text=Test User');
    
    await expect(page).toHaveScreenshot('user-added-success.png', {
      threshold: 0.05
    });
  });
});
```

**Accessibility Testing:**
```typescript
// tests/e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Accessibility Tests', () => {
  test('should meet WCAG 2.1 AA standards', async ({ page }) => {
    await page.goto('/admin/users');
    
    // Run accessibility audit
    const accessibilityReport = await page.accessibility.snapshot();
    
    // Check for critical accessibility issues
    expect(accessibilityReport.violations).toHaveLength(0);
    
    // Check specific accessibility requirements
    await expect(page).toHaveScreenshot('accessibility-audit.png');
  });

  test('should be navigable with keyboard only', async ({ page }) => {
    await page.goto('/admin/users');
    
    // Test keyboard navigation
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toHaveAttribute('data-testid', 'add-user-button');
    
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toHaveAttribute('data-testid', 'user-search-input');
    
    // Test keyboard shortcuts
    await page.keyboard.press('Control+/');
    await expect(page.locator('[data-testid="keyboard-shortcuts-modal"]')).toBeVisible();
  });

  test('should have proper ARIA labels and roles', async ({ page }) => {
    await page.goto('/admin/users');
    
    // Check for proper ARIA attributes
    await expect(page.locator('[data-testid="user-management-table"]')).toHaveAttribute('role', 'table');
    await expect(page.locator('[data-testid="user-search-input"]')).toHaveAttribute('aria-label', 'Search users');
    
    // Check for screen reader support
    const screenReaderText = await page.locator('[data-testid="screen-reader-only"]').textContent();
    expect(screenReaderText).toBeTruthy();
  });
});
```

**Performance Testing:**
```typescript
// tests/e2e/performance.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('should load pages within performance budget', async ({ page }) => {
    // Measure page load time
    const startTime = Date.now();
    await page.goto('/admin/users');
    await page.waitForLoadState('networkidle');
    const loadTime = Date.now() - startTime;
    
    // Assert load time is within budget (2 seconds)
    expect(loadTime).toBeLessThan(2000);
    
    // Check Core Web Vitals
    const metrics = await page.evaluate(() => {
      return {
        FCP: performance.getEntriesByName('first-contentful-paint')[0]?.startTime,
        LCP: performance.getEntriesByName('largest-contentful-paint')[0]?.startTime,
        CLS: performance.getEntriesByName('layout-shift')[0]?.value
      };
    });
    
    expect(metrics.FCP).toBeLessThan(1000); // First Contentful Paint < 1s
    expect(metrics.LCP).toBeLessThan(2500); // Largest Contentful Paint < 2.5s
    expect(metrics.CLS).toBeLessThan(0.1); // Cumulative Layout Shift < 0.1
  });

  test('should handle large datasets efficiently', async ({ page }) => {
    await page.goto('/admin/users?limit=1000');
    
    // Measure time to render large table
    const renderStart = Date.now();
    await page.waitForSelector('[data-testid="user-row"]');
    const renderTime = Date.now() - renderStart;
    
    // Should render 1000 rows within 3 seconds
    expect(renderTime).toBeLessThan(3000);
    
    // Check for virtual scrolling implementation
    const visibleRows = await page.locator('[data-testid="user-row"]').count();
    expect(visibleRows).toBeLessThan(100); // Only visible rows should be rendered
  });
});
```

**Playwright Configuration:**
```typescript
// playwright.config.ts
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
  testDir: './tests/e2e',
  timeout: 30000,
  expect: {
    timeout: 5000
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/results.xml' }]
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] }
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] }
    }
  ],
  globalSetup: require.resolve('./tests/e2e/setup/global-setup.ts'),
  globalTeardown: require.resolve('./tests/e2e/setup/global-teardown.ts')
};

export default config;
```

**Key Features of This Testing Strategy:**
- **Comprehensive Coverage**: Unit, integration, and E2E tests
- **Visual Regression**: Automated visual consistency checks
- **Accessibility**: WCAG compliance testing
- **Performance**: Core Web Vitals monitoring
- **Cross-browser**: Testing across multiple browsers
- **CI/CD Integration**: Automated testing in deployment pipeline
- **Test Data Management**: Proper setup and cleanup
- **Parallel Execution**: Efficient test execution 