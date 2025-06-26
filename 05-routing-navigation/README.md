# React Routing & Navigation - Senior Level Interview Questions

## 1. Advanced Routing Architecture: Dynamic Routes, Guards, and Middleware

**Q: Design a comprehensive routing system for a large-scale React application with role-based access control, dynamic route generation, and middleware support. Explain your architectural decisions and implement a solution that handles complex routing scenarios.**

### Answer
For enterprise applications, we need a routing system that supports dynamic route generation, authentication guards, and middleware patterns. This solution demonstrates advanced routing architecture.

**Core Router Configuration with TypeScript:**
```tsx
import React from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

// Type definitions for route configuration
interface RouteConfig {
  path: string;
  component: React.ComponentType<any>;
  exact?: boolean;
  protected?: boolean;
  roles?: string[];
  middleware?: MiddlewareFunction[];
  children?: RouteConfig[];
}

type MiddlewareFunction = (context: RouteContext) => boolean | Promise<boolean>;

interface RouteContext {
  user: User | null;
  location: Location;
  navigate: NavigateFunction;
}

interface User {
  id: string;
  roles: string[];
  permissions: string[];
}

// Route guard component for authentication and authorization
const RouteGuard: React.FC<{
  route: RouteConfig;
  children: React.ReactNode;
}> = ({ route, children }) => {
  const { user } = useAuth(); // Custom auth hook
  const location = useLocation();
  const navigate = useNavigate();

  // Check if route requires authentication
  if (route.protected && !user) {
    // Redirect to login with return URL
    return <Navigate to={`/login?redirect=${encodeURIComponent(location.pathname)}`} replace />;
  }

  // Check role-based access
  if (route.roles && user && !route.roles.some(role => user.roles.includes(role))) {
    return <Navigate to="/unauthorized" replace />;
  }

  // Execute middleware chain
  const executeMiddleware = async () => {
    if (!route.middleware) return true;
    
    for (const middleware of route.middleware) {
      const result = await middleware({ user, location, navigate });
      if (!result) return false;
    }
    return true;
  };

  // Use effect to handle async middleware
  React.useEffect(() => {
    executeMiddleware().then(allowed => {
      if (!allowed) {
        navigate('/forbidden');
      }
    });
  }, [route.path]);

  return <>{children}</>;
};

// Dynamic route generator based on user permissions
const generateRoutes = (user: User | null): RouteConfig[] => {
  const baseRoutes: RouteConfig[] = [
    {
      path: '/',
      component: HomePage,
      exact: true
    },
    {
      path: '/login',
      component: LoginPage
    }
  ];

  // Admin routes - only accessible by admin users
  if (user?.roles.includes('admin')) {
    baseRoutes.push({
      path: '/admin',
      component: AdminDashboard,
      protected: true,
      roles: ['admin'],
      middleware: [adminActivityMiddleware], // Custom middleware for admin activity logging
      children: [
        {
          path: 'users',
          component: UserManagement,
          protected: true,
          roles: ['admin']
        },
        {
          path: 'analytics',
          component: Analytics,
          protected: true,
          roles: ['admin']
        }
      ]
    });
  }

  // Manager routes - accessible by managers and admins
  if (user?.roles.some(role => ['admin', 'manager'].includes(role))) {
    baseRoutes.push({
      path: '/manager',
      component: ManagerDashboard,
      protected: true,
      roles: ['admin', 'manager'],
      middleware: [rateLimitMiddleware], // Rate limiting middleware
      children: [
        {
          path: 'reports',
          component: Reports,
          protected: true,
          roles: ['admin', 'manager']
        }
      ]
    });
  }

  return baseRoutes;
};

// Middleware implementations
const adminActivityMiddleware: MiddlewareFunction = async ({ user, location }) => {
  // Log admin activity for audit purposes
  if (user && location.pathname.startsWith('/admin')) {
    await logAdminActivity({
      userId: user.id,
      action: 'admin_access',
      path: location.pathname,
      timestamp: new Date()
    });
  }
  return true;
};

const rateLimitMiddleware: MiddlewareFunction = ({ user, location }) => {
  // Implement rate limiting logic
  const key = `rate_limit:${user?.id}:${location.pathname}`;
  const currentCount = getRateLimitCount(key);
  
  if (currentCount > 100) { // 100 requests per hour
    return false;
  }
  
  incrementRateLimitCount(key);
  return true;
};

// Main router component with dynamic route generation
const AppRouter: React.FC = () => {
  const { user } = useAuth();
  const routes = generateRoutes(user);

  // Recursive function to render nested routes
  const renderRoutes = (routeConfigs: RouteConfig[]): React.ReactNode => {
    return routeConfigs.map(route => (
      <Route
        key={route.path}
        path={route.path}
        element={
          <RouteGuard route={route}>
            <route.component />
          </RouteGuard>
        }
      >
        {route.children && renderRoutes(route.children)}
      </Route>
    ));
  };

  return (
    <BrowserRouter>
      <Routes>
        {renderRoutes(routes)}
        {/* Catch-all route for 404 */}
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
};
```

---

## 2. Route-Based Code Splitting and Performance Optimization

**Q: Implement a sophisticated code splitting strategy that optimizes bundle size and loading performance. Include preloading strategies, error boundaries for lazy-loaded components, and intelligent loading states based on user behavior patterns.**

### Answer
Advanced code splitting requires understanding user behavior, implementing intelligent preloading, and handling loading states gracefully.

**Intelligent Code Splitting with Preloading:**
```tsx
import React, { Suspense, lazy, useEffect, useState } from 'react';
import { useLocation, useNavigate } from 'react-router-dom';

// Route-based lazy loading with error boundaries
const Dashboard = lazy(() => 
  import('./pages/Dashboard').catch(() => ({
    default: () => <ErrorFallback message="Failed to load Dashboard" />
  }))
);

const Analytics = lazy(() => 
  import('./pages/Analytics').catch(() => ({
    default: () => <ErrorFallback message="Failed to load Analytics" />
  }))
);

const Settings = lazy(() => 
  import('./pages/Settings').catch(() => ({
    default: () => <ErrorFallback message="Failed to load Settings" />
  }))
);

// Preloading strategy based on user behavior
const PreloadManager: React.FC = () => {
  const location = useLocation();
  const navigate = useNavigate();
  const [preloadedRoutes, setPreloadedRoutes] = useState<Set<string>>(new Set());

  // Track user navigation patterns
  const trackNavigation = (from: string, to: string) => {
    const navigationPattern = `${from}->${to}`;
    const patterns = JSON.parse(localStorage.getItem('navPatterns') || '{}');
    patterns[navigationPattern] = (patterns[navigationPattern] || 0) + 1;
    localStorage.setItem('navPatterns', JSON.stringify(patterns));
  };

  // Intelligent preloading based on navigation patterns
  const preloadLikelyRoutes = (currentPath: string) => {
    const patterns = JSON.parse(localStorage.getItem('navPatterns') || '{}');
    const likelyRoutes = Object.entries(patterns)
      .filter(([pattern]) => pattern.startsWith(currentPath))
      .sort(([, a], [, b]) => (b as number) - (a as number))
      .slice(0, 2) // Preload top 2 most likely routes
      .map(([pattern]) => pattern.split('->')[1]);

    likelyRoutes.forEach(route => {
      if (!preloadedRoutes.has(route)) {
        preloadRoute(route);
        setPreloadedRoutes(prev => new Set(prev).add(route));
      }
    });
  };

  // Preload specific route
  const preloadRoute = (route: string) => {
    const routeMap: Record<string, () => Promise<any>> = {
      '/analytics': () => import('./pages/Analytics'),
      '/settings': () => import('./pages/Settings'),
      '/dashboard': () => import('./pages/Dashboard')
    };

    if (routeMap[route]) {
      routeMap[route]().catch(console.error);
    }
  };

  // Preload on hover for better UX
  const handleRouteHover = (route: string) => {
    if (!preloadedRoutes.has(route)) {
      preloadRoute(route);
      setPreloadedRoutes(prev => new Set(prev).add(route));
    }
  };

  useEffect(() => {
    // Preload likely routes when component mounts
    preloadLikelyRoutes(location.pathname);
  }, [location.pathname]);

  return (
    <nav>
      <a 
        href="/dashboard" 
        onMouseEnter={() => handleRouteHover('/dashboard')}
        onClick={() => trackNavigation(location.pathname, '/dashboard')}
      >
        Dashboard
      </a>
      <a 
        href="/analytics" 
        onMouseEnter={() => handleRouteHover('/analytics')}
        onClick={() => trackNavigation(location.pathname, '/analytics')}
      >
        Analytics
      </a>
      <a 
        href="/settings" 
        onMouseEnter={() => handleRouteHover('/settings')}
        onClick={() => trackNavigation(location.pathname, '/settings')}
      >
        Settings
      </a>
    </nav>
  );
};

// Advanced loading component with skeleton screens
const AdvancedSuspense: React.FC<{ children: React.ReactNode; route: string }> = ({ 
  children, 
  route 
}) => {
  const [loadingTime, setLoadingTime] = useState(0);
  const startTime = React.useRef(Date.now());

  useEffect(() => {
    const timer = setInterval(() => {
      setLoadingTime(Date.now() - startTime.current);
    }, 100);

    return () => clearInterval(timer);
  }, []);

  // Show skeleton for first 500ms, then loading spinner
  const LoadingComponent = () => {
    if (loadingTime < 500) {
      return <SkeletonLoader route={route} />;
    }
    return <SpinnerLoader route={route} />;
  };

  return (
    <Suspense fallback={<LoadingComponent />}>
      {children}
    </Suspense>
  );
};

// Skeleton loader component
const SkeletonLoader: React.FC<{ route: string }> = ({ route }) => {
  const skeletonMap: Record<string, React.ReactNode> = {
    '/dashboard': (
      <div className="skeleton-dashboard">
        <div className="skeleton-header" />
        <div className="skeleton-cards">
          {[1, 2, 3].map(i => <div key={i} className="skeleton-card" />)}
        </div>
      </div>
    ),
    '/analytics': (
      <div className="skeleton-analytics">
        <div className="skeleton-chart" />
        <div className="skeleton-metrics" />
      </div>
    )
  };

  return skeletonMap[route] || <div className="skeleton-default" />;
};

// Error boundary for lazy-loaded components
class RouteErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error?: Error }
> {
  constructor(props: { children: React.ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error to monitoring service
    console.error('Route Error:', error, errorInfo);
    
    // Send to error tracking service
    logError({
      error: error.message,
      stack: error.stack,
      route: window.location.pathname,
      timestamp: new Date()
    });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }

    return this.props.children;
  }
}
```

---

## 3. Deep Linking and URL State Management

**Q: Design a system for managing complex application state in URLs, including deep linking support, browser history management, and synchronization between URL parameters and application state. Implement a solution that handles nested state, filters, and pagination.**

### Answer
URL state management is crucial for shareable links, browser navigation, and maintaining application state across sessions.

**Advanced URL State Management:**
```tsx
import React, { useEffect, useCallback, useMemo } from 'react';
import { useSearchParams, useNavigate, useLocation } from 'react-router-dom';

// Type definitions for URL state
interface URLState {
  page: number;
  filters: Record<string, string[]>;
  sortBy: string;
  sortOrder: 'asc' | 'desc';
  view: 'grid' | 'list';
  selectedItems: string[];
}

// Custom hook for URL state management
const useURLState = <T extends Record<string, any>>(
  initialState: T,
  options: {
    serialize?: (state: T) => Record<string, string>;
    deserialize?: (params: URLSearchParams) => T;
    debounceMs?: number;
  } = {}
) => {
  const [searchParams, setSearchParams] = useSearchParams();
  const navigate = useNavigate();
  const location = useLocation();

  // Default serialization/deserialization
  const serialize = options.serialize || ((state: T) => {
    const params: Record<string, string> = {};
    
    Object.entries(state).forEach(([key, value]) => {
      if (value !== undefined && value !== null) {
        if (Array.isArray(value)) {
          params[key] = value.join(',');
        } else {
          params[key] = String(value);
        }
      }
    });
    
    return params;
  });

  const deserialize = options.deserialize || ((params: URLSearchParams) => {
    const state: Partial<T> = {};
    
    Object.keys(initialState).forEach(key => {
      const paramValue = params.get(key);
      if (paramValue !== null) {
        const initialValue = initialState[key];
        
        if (Array.isArray(initialValue)) {
          state[key] = paramValue.split(',').filter(Boolean) as any;
        } else if (typeof initialValue === 'number') {
          state[key] = Number(paramValue) as any;
        } else if (typeof initialValue === 'boolean') {
          state[key] = (paramValue === 'true') as any;
        } else {
          state[key] = paramValue as any;
        }
      }
    });
    
    return { ...initialState, ...state };
  });

  // Debounced state update to prevent excessive URL updates
  const debouncedSetSearchParams = useCallback(
    debounce((params: Record<string, string>) => {
      setSearchParams(params, { replace: true });
    }, options.debounceMs || 300),
    [setSearchParams, options.debounceMs]
  );

  // Get current state from URL
  const currentState = useMemo(() => {
    return deserialize(searchParams);
  }, [searchParams, deserialize]);

  // Update URL state
  const updateState = useCallback((updates: Partial<T>) => {
    const newState = { ...currentState, ...updates };
    const serializedParams = serialize(newState);
    debouncedSetSearchParams(serializedParams);
  }, [currentState, serialize, debouncedSetSearchParams]);

  // Reset state to initial values
  const resetState = useCallback(() => {
    setSearchParams({}, { replace: true });
  }, [setSearchParams]);

  return {
    state: currentState,
    updateState,
    resetState,
    searchParams
  };
};

// Complex data table with URL state management
const AdvancedDataTable: React.FC<{ data: any[] }> = ({ data }) => {
  const initialState: URLState = {
    page: 1,
    filters: {},
    sortBy: 'name',
    sortOrder: 'asc',
    view: 'grid',
    selectedItems: []
  };

  const { state, updateState, resetState } = useURLState(initialState, {
    // Custom serialization for complex filters
    serialize: (state: URLState) => {
      const params: Record<string, string> = {
        page: String(state.page),
        sortBy: state.sortBy,
        sortOrder: state.sortOrder,
        view: state.view
      };

      // Serialize filters as JSON for complex nested structures
      if (Object.keys(state.filters).length > 0) {
        params.filters = JSON.stringify(state.filters);
      }

      // Serialize selected items
      if (state.selectedItems.length > 0) {
        params.selected = state.selectedItems.join(',');
      }

      return params;
    },
    // Custom deserialization
    deserialize: (params: URLSearchParams) => {
      const state: Partial<URLState> = {
        page: Number(params.get('page')) || 1,
        sortBy: params.get('sortBy') || 'name',
        sortOrder: (params.get('sortOrder') as 'asc' | 'desc') || 'asc',
        view: (params.get('view') as 'grid' | 'list') || 'grid',
        filters: {},
        selectedItems: []
      };

      // Deserialize filters
      const filtersParam = params.get('filters');
      if (filtersParam) {
        try {
          state.filters = JSON.parse(filtersParam);
        } catch (e) {
          console.warn('Failed to parse filters from URL');
        }
      }

      // Deserialize selected items
      const selectedParam = params.get('selected');
      if (selectedParam) {
        state.selectedItems = selectedParam.split(',').filter(Boolean);
      }

      return { ...initialState, ...state };
    }
  });

  // Computed values based on current state
  const filteredData = useMemo(() => {
    let result = [...data];

    // Apply filters
    Object.entries(state.filters).forEach(([key, values]) => {
      if (values.length > 0) {
        result = result.filter(item => 
          values.some(value => 
            String(item[key]).toLowerCase().includes(value.toLowerCase())
          )
        );
      }
    });

    // Apply sorting
    result.sort((a, b) => {
      const aValue = a[state.sortBy];
      const bValue = b[state.sortBy];
      
      if (state.sortOrder === 'asc') {
        return aValue > bValue ? 1 : -1;
      } else {
        return aValue < bValue ? 1 : -1;
      }
    });

    return result;
  }, [data, state.filters, state.sortBy, state.sortOrder]);

  // Pagination logic
  const itemsPerPage = 20;
  const totalPages = Math.ceil(filteredData.length / itemsPerPage);
  const startIndex = (state.page - 1) * itemsPerPage;
  const paginatedData = filteredData.slice(startIndex, startIndex + itemsPerPage);

  // Event handlers
  const handlePageChange = (page: number) => {
    updateState({ page });
  };

  const handleSort = (sortBy: string) => {
    const sortOrder = state.sortBy === sortBy && state.sortOrder === 'asc' ? 'desc' : 'asc';
    updateState({ sortBy, sortOrder, page: 1 }); // Reset to first page when sorting
  };

  const handleFilter = (key: string, values: string[]) => {
    const newFilters = { ...state.filters };
    if (values.length === 0) {
      delete newFilters[key];
    } else {
      newFilters[key] = values;
    }
    updateState({ filters: newFilters, page: 1 }); // Reset to first page when filtering
  };

  const handleViewChange = (view: 'grid' | 'list') => {
    updateState({ view });
  };

  const handleSelectionChange = (selectedItems: string[]) => {
    updateState({ selectedItems });
  };

  return (
    <div className="advanced-data-table">
      {/* URL state debug info */}
      <div className="url-state-debug">
        <small>Current URL State: {JSON.stringify(state)}</small>
        <button onClick={resetState}>Reset State</button>
      </div>

      {/* Filters */}
      <FilterPanel 
        filters={state.filters}
        onFilterChange={handleFilter}
      />

      {/* Table controls */}
      <TableControls
        sortBy={state.sortBy}
        sortOrder={state.sortOrder}
        view={state.view}
        onSort={handleSort}
        onViewChange={handleViewChange}
      />

      {/* Data table */}
      <DataTable
        data={paginatedData}
        selectedItems={state.selectedItems}
        onSelectionChange={handleSelectionChange}
        view={state.view}
      />

      {/* Pagination */}
      <Pagination
        currentPage={state.page}
        totalPages={totalPages}
        onPageChange={handlePageChange}
      />
    </div>
  );
};

// Utility function for debouncing
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout;
  return (...args: Parameters<T>) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), wait);
  };
}
```

**Key Features of This Implementation:**
- **Deep Linking**: All state is preserved in URL for shareable links
- **Browser Navigation**: Back/forward buttons work correctly
- **Debounced Updates**: Prevents excessive URL updates during rapid changes
- **Complex State Serialization**: Handles nested objects and arrays
- **Type Safety**: Full TypeScript support with generic types
- **Performance**: Memoized computations and efficient updates 