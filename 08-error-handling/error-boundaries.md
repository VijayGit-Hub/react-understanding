# React Error Boundaries: Comprehensive Guide

Error Boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed.

---

## What are Error Boundaries?

Error Boundaries are **class components** that implement one or both of these lifecycle methods:
- `static getDerivedStateFromError()` - Renders fallback UI after an error
- `componentDidCatch()` - Logs error information

**Key Points:**
- Only work in class components (not function components)
- Catch errors in the component tree below them
- Don't catch errors in event handlers, async code, or server-side rendering
- Work like JavaScript `catch {}` blocks but for components

---

## Basic Error Boundary Structure

```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    
    // Initialize state to track if an error has occurred
    this.state = { 
      hasError: false,
      error: null,
      errorInfo: null
    };
  }

  // This lifecycle method is called when an error occurs in any child component
  // It receives the error that was thrown and should return new state
  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { 
      hasError: true,
      error: error
    };
  }

  // This lifecycle method is called after an error has been thrown
  // It receives the error and error info (stack trace)
  componentDidCatch(error, errorInfo) {
    // Log the error to console, error reporting service, etc.
    console.error('Error caught by boundary:', error);
    console.error('Error info:', errorInfo);
    
    // You can also log to an error reporting service like:
    // logErrorToService(error, errorInfo);
    
    // Update state with error info for debugging
    this.setState({
      errorInfo: errorInfo
    });
  }

  render() {
    // If an error occurred, render fallback UI
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '20px', 
          border: '1px solid #ff6b6b', 
          borderRadius: '8px',
          backgroundColor: '#fff5f5',
          color: '#d63031'
        }}>
          <h2>Something went wrong!</h2>
          <p>We're sorry, but something unexpected happened.</p>
          
          {/* Show error details in development */}
          {process.env.NODE_ENV === 'development' && (
            <details style={{ marginTop: '10px' }}>
              <summary>Error Details (Development Only)</summary>
              <pre style={{ 
                fontSize: '12px', 
                backgroundColor: '#f8f9fa',
                padding: '10px',
                borderRadius: '4px',
                overflow: 'auto'
              }}>
                {this.state.error && this.state.error.toString()}
                <br />
                {this.state.errorInfo && this.state.errorInfo.componentStack}
              </pre>
            </details>
          )}
          
          <button 
            onClick={() => {
              // Reset the error state to try rendering again
              this.setState({ 
                hasError: false, 
                error: null, 
                errorInfo: null 
              });
            }}
            style={{
              marginTop: '10px',
              padding: '8px 16px',
              backgroundColor: '#0984e3',
              color: 'white',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer'
            }}
          >
            Try Again
          </button>
        </div>
      );
    }

    // If no error, render children normally
    return this.props.children;
  }
}

export default ErrorBoundary;
```

---

## Understanding Error Boundary Lifecycle Methods

Error Boundaries have two key lifecycle methods that serve different purposes:

### `static getDerivedStateFromError(error)`

**Purpose:** Update state to render fallback UI
**When called:** During render phase, after an error is thrown
**Return value:** Must return an object to update state, or null
**Side effects:** None allowed (pure function)

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  // This method is called during render phase
  // It should return new state to show fallback UI
  static getDerivedStateFromError(error) {
    console.log('getDerivedStateFromError called with:', error);
    
    // Return new state - this will trigger a re-render with fallback UI
    return { 
      hasError: true, 
      error: error.message 
    };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Error Boundary caught an error!</h2>
          <p>Error: {this.state.error}</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### `componentDidCatch(error, errorInfo)`

**Purpose:** Log errors and perform side effects
**When called:** During commit phase, after fallback UI is rendered
**Return value:** None (void)
**Side effects:** Perfect place for logging, analytics, etc.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error: error.message };
  }

  // This method is called after the fallback UI is rendered
  // Perfect for logging, sending to error reporting services, etc.
  componentDidCatch(error, errorInfo) {
    console.log('componentDidCatch called');
    console.log('Error:', error);
    console.log('Error Info:', errorInfo);
    
    // Log to error reporting service
    this.logErrorToService(error, errorInfo);
    
    // Send analytics
    this.trackError(error, errorInfo);
  }

  logErrorToService = (error, errorInfo) => {
    // Example: Send to error reporting service
    if (window.errorReportingService) {
      window.errorReportingService.captureException(error, {
        extra: {
          componentStack: errorInfo.componentStack,
          timestamp: new Date().toISOString()
        }
      });
    }
  };

  trackError = (error, errorInfo) => {
    // Example: Send to analytics
    if (window.analytics) {
      window.analytics.track('Error Boundary Caught Error', {
        errorMessage: error.message,
        componentStack: errorInfo.componentStack
      });
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <p>We've been notified and are working on a fix.</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### Key Differences Summary

| Aspect | `getDerivedStateFromError` | `componentDidCatch` |
|--------|---------------------------|---------------------|
| **Purpose** | Update state for fallback UI | Log errors and side effects |
| **When Called** | During render phase | During commit phase |
| **Return Value** | State object or null | None (void) |
| **Side Effects** | Not allowed (pure) | Allowed and encouraged |
| **Parameters** | `error` only | `error` and `errorInfo` |
| **Use Case** | Show error UI | Logging, analytics, reporting |

### Complete Example Using Both Methods

```jsx
class ComprehensiveErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null,
      errorId: null,
      retryCount: 0
    };
  }

  // Step 1: Update state to show fallback UI
  static getDerivedStateFromError(error) {
    console.log('Step 1: getDerivedStateFromError - updating state');
    
    return { 
      hasError: true, 
      error: error.message,
      errorId: Date.now() // Generate unique error ID
    };
  }

  // Step 2: Log error and perform side effects
  componentDidCatch(error, errorInfo) {
    console.log('Step 2: componentDidCatch - logging and side effects');
    
    // Log to console
    console.error('Error Boundary caught error:', error);
    console.error('Error info:', errorInfo);
    
    // Log to external service
    this.logToErrorService(error, errorInfo);
    
    // Track error for analytics
    this.trackErrorForAnalytics(error, errorInfo);
    
    // Increment retry count
    this.setState(prevState => ({
      retryCount: prevState.retryCount + 1
    }));
  }

  logToErrorService = (error, errorInfo) => {
    const errorData = {
      id: this.state.errorId,
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent
    };

    // Send to your error reporting service
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorData)
    }).catch(console.error);
  };

  trackErrorForAnalytics = (error, errorInfo) => {
    // Track error for analytics/monitoring
    if (window.gtag) {
      window.gtag('event', 'error_boundary_caught', {
        error_message: error.message,
        component_stack: errorInfo.componentStack
      });
    }
  };

  handleRetry = () => {
    console.log('Retrying component...');
    this.setState({ 
      hasError: false, 
      error: null,
      errorId: null
    });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '20px',
          backgroundColor: '#fef2f2',
          border: '1px solid #fecaca',
          borderRadius: '8px',
          margin: '10px'
        }}>
          <h3>Error #{this.state.errorId}</h3>
          <p><strong>Error:</strong> {this.state.error}</p>
          <p><strong>Retry Count:</strong> {this.state.retryCount}</p>
          
          <button 
            onClick={this.handleRetry}
            style={{
              padding: '8px 16px',
              backgroundColor: '#3b82f6',
              color: 'white',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
              marginRight: '10px'
            }}
          >
            Try Again
          </button>
          
          <button 
            onClick={() => window.location.reload()}
            style={{
              padding: '8px 16px',
              backgroundColor: '#ef4444',
              color: 'white',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer'
            }}
          >
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

---

## Usage Examples

### 1. Wrapping Individual Components

```jsx
import React from 'react';
import ErrorBoundary from './ErrorBoundary';

// Component that might throw an error
function BuggyCounter({ count }) {
  // Simulate an error when count is 5
  if (count === 5) {
    throw new Error('I crashed! Count is 5');
  }
  
  return <h1>{count}</h1>;
}

// Component that uses the buggy component
function CounterApp() {
  const [count, setCount] = React.useState(0);
  
  return (
    <div>
      <h2>Counter Example</h2>
      
      {/* Wrap the buggy component in an error boundary */}
      <ErrorBoundary>
        <BuggyCounter count={count} />
      </ErrorBoundary>
      
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      
      <p>Current count: {count}</p>
    </div>
  );
}
```

### 2. Wrapping Route Components

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import ErrorBoundary from './ErrorBoundary';

// Different error boundaries for different sections
class UserProfileErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>User Profile Error</h2>
          <p>Unable to load user profile. Please try again later.</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Wrap each route in an error boundary */}
        <Route 
          path="/profile" 
          element={
            <UserProfileErrorBoundary>
              <UserProfile />
            </UserProfileErrorBoundary>
          } 
        />
        <Route 
          path="/dashboard" 
          element={
            <ErrorBoundary>
              <Dashboard />
            </ErrorBoundary>
          } 
        />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## Advanced Error Boundary Patterns

### 1. Error Boundary with Retry Logic

```jsx
class RetryErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      retryCount: 0,
      maxRetries: props.maxRetries || 3
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error in component:', error);
    
    // Log to error reporting service
    this.logError(error, errorInfo);
  }

  logError = (error, errorInfo) => {
    // Example: Send to error reporting service
    if (window.errorReportingService) {
      window.errorReportingService.captureException(error, {
        extra: {
          componentStack: errorInfo.componentStack,
          retryCount: this.state.retryCount
        }
      });
    }
  };

  handleRetry = () => {
    const { retryCount, maxRetries } = this.state;
    
    if (retryCount < maxRetries) {
      // Increment retry count and reset error state
      this.setState(prevState => ({
        hasError: false,
        retryCount: prevState.retryCount + 1
      }));
    } else {
      // Max retries reached, show permanent error
      this.setState({ 
        hasError: true,
        maxRetriesReached: true 
      });
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '20px', 
          textAlign: 'center',
          backgroundColor: '#fff5f5',
          border: '1px solid #fed7d7',
          borderRadius: '8px'
        }}>
          <h3>Something went wrong</h3>
          
          {this.state.maxRetriesReached ? (
            <div>
              <p>We've tried multiple times but couldn't load this content.</p>
              <button 
                onClick={() => window.location.reload()}
                style={{
                  padding: '10px 20px',
                  backgroundColor: '#e53e3e',
                  color: 'white',
                  border: 'none',
                  borderRadius: '4px',
                  cursor: 'pointer'
                }}
              >
                Reload Page
              </button>
            </div>
          ) : (
            <div>
              <p>Retry attempt {this.state.retryCount + 1} of {this.state.maxRetries}</p>
              <button 
                onClick={this.handleRetry}
                style={{
                  padding: '10px 20px',
                  backgroundColor: '#38a169',
                  color: 'white',
                  border: 'none',
                  borderRadius: '4px',
                  cursor: 'pointer'
                }}
              >
                Try Again
              </button>
            </div>
          )}
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 2. Error Boundary with Different Fallbacks

```jsx
class FlexibleErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Use custom fallback if provided, otherwise use default
      if (this.props.fallback) {
        return this.props.fallback(this.state.error);
      }
      
      // Default fallback UI
      return (
        <div style={{ 
          padding: '15px', 
          backgroundColor: '#fef2f2',
          border: '1px solid #fecaca',
          borderRadius: '6px',
          color: '#dc2626'
        }}>
          <h4>An error occurred</h4>
          <p>Please try refreshing the page.</p>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage with custom fallback
function App() {
  return (
    <div>
      {/* Default fallback */}
      <FlexibleErrorBoundary>
        <ComponentThatMightError />
      </FlexibleErrorBoundary>
      
      {/* Custom fallback */}
      <FlexibleErrorBoundary
        fallback={(error) => (
          <div style={{ color: 'red' }}>
            <h3>Custom Error UI</h3>
            <p>Error: {error.message}</p>
            <button onClick={() => window.location.reload()}>
              Reload
            </button>
          </div>
        )}
      >
        <AnotherComponent />
      </FlexibleErrorBoundary>
    </div>
  );
}
```

---

## Common Pitfalls and How to Avoid Them

### 1. **Pitfall: Error Boundaries Don't Catch All Errors**

**What they DON'T catch:**
- Event handlers
- Asynchronous code (setTimeout, promises, async/await)
- Server-side rendering
- Errors thrown in the error boundary itself

**Solution: Use try-catch for these cases**

```jsx
// ❌ This won't be caught by Error Boundary
function BuggyComponent() {
  const handleClick = () => {
    throw new Error('Event handler error');
  };
  
  return <button onClick={handleClick}>Click me</button>;
}

// ✅ Use try-catch for event handlers
function SafeComponent() {
  const handleClick = () => {
    try {
      // Risky operation
      throw new Error('Event handler error');
    } catch (error) {
      console.error('Caught error in event handler:', error);
      // Handle error appropriately
    }
  };
  
  return <button onClick={handleClick}>Click me</button>;
}

// ✅ Use try-catch for async operations
function AsyncComponent() {
  const [data, setData] = React.useState(null);
  const [error, setError] = React.useState(null);

  React.useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/data');
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        const result = await response.json();
        setData(result);
      } catch (error) {
        setError(error);
        console.error('Async error:', error);
      }
    };

    fetchData();
  }, []);

  if (error) {
    return <div>Error loading data: {error.message}</div>;
  }

  return <div>{data ? JSON.stringify(data) : 'Loading...'}</div>;
}
```

### 2. **Pitfall: Not Handling Error Boundary Errors**

**Problem:** If an error occurs in the error boundary itself, it can cause infinite loops.

**Solution:** Keep error boundaries simple and test them thoroughly.

```jsx
// ❌ Complex error boundary that might fail
class BadErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    // This might throw an error itself
    const errorInfo = this.processError(error); // This could fail
    return { hasError: true, errorInfo };
  }

  // ❌ Complex processing that might fail
  processError(error) {
    // Complex logic that might throw
    return JSON.parse(error.toString()); // Could fail
  }
}

// ✅ Simple, reliable error boundary
class GoodErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    // Keep it simple - just return the error
    return { hasError: true, error: error.message };
  }

  componentDidCatch(error, errorInfo) {
    // Log error safely
    console.error('Error:', error.message);
    console.error('Stack:', errorInfo.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <p>Error: {this.state.error}</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### 3. **Pitfall: Not Providing Recovery Mechanisms**

**Problem:** Users get stuck with error UI and can't recover.

**Solution:** Always provide ways to recover from errors.

```jsx
class RecoverableErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null,
      retryKey: 0 // Force re-mount of children
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  handleRetry = () => {
    // Reset error state and force re-mount
    this.setState({ 
      hasError: false, 
      error: null,
      retryKey: this.state.retryKey + 1 
    });
  };

  handleReload = () => {
    window.location.reload();
  };

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '20px',
          textAlign: 'center',
          backgroundColor: '#fef2f2',
          border: '1px solid #fecaca',
          borderRadius: '8px'
        }}>
          <h3>Oops! Something went wrong</h3>
          <p>We encountered an unexpected error.</p>
          
          <div style={{ marginTop: '15px' }}>
            <button 
              onClick={this.handleRetry}
              style={{
                marginRight: '10px',
                padding: '8px 16px',
                backgroundColor: '#3b82f6',
                color: 'white',
                border: 'none',
                borderRadius: '4px',
                cursor: 'pointer'
              }}
            >
              Try Again
            </button>
            
            <button 
              onClick={this.handleReload}
              style={{
                padding: '8px 16px',
                backgroundColor: '#ef4444',
                color: 'white',
                border: 'none',
                borderRadius: '4px',
                cursor: 'pointer'
              }}
            >
              Reload Page
            </button>
          </div>
        </div>
      );
    }

    // Use key to force re-mount when retrying
    return (
      <div key={this.state.retryKey}>
        {this.props.children}
      </div>
    );
  }
}
```

### 4. **Pitfall: Not Logging Errors Properly**

**Problem:** Errors occur but you don't know about them.

**Solution:** Implement proper error logging.

```jsx
class LoggingErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log to console
    console.error('Error Boundary caught an error:', error);
    console.error('Error info:', errorInfo);

    // Log to external service
    this.logToService(error, errorInfo);
  }

  logToService = (error, errorInfo) => {
    // Example: Send to error reporting service
    const errorData = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href
    };

    // Send to your error reporting service
    fetch('/api/log-error', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorData)
    }).catch(console.error); // Don't let logging errors break the app
  };

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong</h2>
          <p>We've been notified and are working on a fix.</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

---

## Best Practices

### 1. **Place Error Boundaries Strategically**

```jsx
function App() {
  return (
    <div>
      {/* Top-level boundary for critical errors */}
      <ErrorBoundary>
        <Header />
        
        {/* Route-level boundaries */}
        <Routes>
          <Route 
            path="/dashboard" 
            element={
              <ErrorBoundary>
                <Dashboard />
              </ErrorBoundary>
            } 
          />
          <Route 
            path="/profile" 
            element={
              <ErrorBoundary>
                <Profile />
              </ErrorBoundary>
            } 
          />
        </Routes>
        
        <Footer />
      </ErrorBoundary>
    </div>
  );
}
```

### 2. **Use Different Error Boundaries for Different Contexts**

```jsx
// Error boundary for data-heavy components
class DataErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h3>Data Loading Error</h3>
          <p>Unable to load the requested data.</p>
          <button onClick={() => window.location.reload()}>
            Refresh Page
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Error boundary for UI components
class UIErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '10px',
          backgroundColor: '#fef2f2',
          border: '1px solid #fecaca',
          borderRadius: '4px'
        }}>
          <p>This component is temporarily unavailable.</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### 3. **Test Your Error Boundaries**

```jsx
// Component that throws an error for testing
function BuggyComponent({ shouldThrow = false }) {
  if (shouldThrow) {
    throw new Error('Test error');
  }
  return <div>Working component</div>;
}

// Test your error boundary
function TestErrorBoundary() {
  const [shouldThrow, setShouldThrow] = React.useState(false);

  return (
    <div>
      <button onClick={() => setShouldThrow(!shouldThrow)}>
        {shouldThrow ? 'Hide Error' : 'Show Error'}
      </button>
      
      <ErrorBoundary>
        <BuggyComponent shouldThrow={shouldThrow} />
      </ErrorBoundary>
    </div>
  );
}
```

---

## Real-World Example: E-commerce App

```jsx
// Main app with error boundaries
function EcommerceApp() {
  return (
    <div>
      {/* Top-level boundary */}
      <CriticalErrorBoundary>
        <Header />
        
        <main>
          {/* Product listing with its own boundary */}
          <ProductListErrorBoundary>
            <ProductList />
          </ProductListErrorBoundary>
          
          {/* Shopping cart with its own boundary */}
          <CartErrorBoundary>
            <ShoppingCart />
          </CartErrorBoundary>
        </main>
        
        <Footer />
      </CriticalErrorBoundary>
    </div>
  );
}

// Specialized error boundaries
class CriticalErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log critical errors immediately
    console.error('Critical error:', error);
    
    // Send to monitoring service
    this.sendToMonitoring(error, errorInfo);
  }

  sendToMonitoring = (error, errorInfo) => {
    // Implementation for error monitoring service
  };

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '40px',
          textAlign: 'center',
          backgroundColor: '#fef2f2'
        }}>
          <h1>We're experiencing technical difficulties</h1>
          <p>Please try again in a few minutes.</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

class ProductListErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '20px',
          backgroundColor: '#f0f9ff',
          border: '1px solid #bae6fd',
          borderRadius: '8px'
        }}>
          <h3>Unable to load products</h3>
          <p>Please try refreshing the page or contact support.</p>
        </div>
      );
    }
    return this.props.children;
  }
}

class CartErrorBoundary extends React.Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ 
          padding: '15px',
          backgroundColor: '#fef3c7',
          border: '1px solid #fde68a',
          borderRadius: '6px'
        }}>
          <h4>Cart temporarily unavailable</h4>
          <p>Your items are safe. Please try again.</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

---

## Summary

Error Boundaries are essential for building robust React applications:

**Key Points:**
- Only work in class components
- Catch errors in the component tree below them
- Provide fallback UI when errors occur
- Should be placed strategically throughout your app
- Need proper error logging and recovery mechanisms

**Best Practices:**
- Keep error boundaries simple and reliable
- Provide multiple recovery options
- Log errors appropriately
- Test your error boundaries
- Use different boundaries for different contexts

**Common Pitfalls:**
- Not catching all types of errors
- Complex error boundaries that might fail
- No recovery mechanisms
- Poor error logging

By implementing Error Boundaries correctly, you can create more resilient React applications that gracefully handle errors and provide better user experiences.

---

**Reference:** [React Error Boundaries Documentation](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) 

---

## Error Boundaries in Functional Components

Since Error Boundaries require class components, you have several options for using them with functional components:

### Option 1: Use a Class Component Wrapper

The simplest approach is to create a class-based Error Boundary and use it to wrap your functional components.

```jsx
// Class-based Error Boundary (required)
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div>
          <h2>Something went wrong</h2>
          <p>Error: {this.state.error?.message}</p>
        </div>
      );
    }
    return this.props.children;
  }
}

// Functional component that might throw an error
function BuggyFunctionalComponent({ count }) {
  if (count === 5) {
    throw new Error('Functional component crashed!');
  }
  
  return <div>Count: {count}</div>;
}

// Usage: Wrap functional component with class-based Error Boundary
function App() {
  const [count, setCount] = React.useState(0);
  
  return (
    <div>
      <ErrorBoundary 
        fallback={<div>Custom fallback for functional component</div>}
      >
        <BuggyFunctionalComponent count={count} />
      </ErrorBoundary>
      
      <button onClick={() => setCount(count + 1)}>
        Increment ({count})
      </button>
    </div>
  );
}
```

### Option 2: Use Third-Party Libraries

Several libraries provide Error Boundary functionality for functional components:

#### Using `react-error-boundary`

```jsx
import { ErrorBoundary } from 'react-error-boundary';

// Functional component that might error
function UserProfile({ userId }) {
  const [user, setUser] = React.useState(null);
  
  React.useEffect(() => {
    if (userId === 'invalid') {
      throw new Error('Invalid user ID');
    }
    // Fetch user data
  }, [userId]);
  
  return <div>User: {user?.name}</div>;
}

// Error fallback component
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div style={{ 
      padding: '20px',
      backgroundColor: '#fef2f2',
      border: '1px solid #fecaca',
      borderRadius: '8px'
    }}>
      <h2>Something went wrong:</h2>
      <pre style={{ color: 'red' }}>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// Usage
function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, errorInfo) => {
        console.error('Error caught:', error, errorInfo);
      }}
      onReset={() => {
        // Reset the state of your app here
        console.log('Error boundary reset');
      }}
    >
      <UserProfile userId="invalid" />
    </ErrorBoundary>
  );
}
```

#### Using `@sentry/react` (for error monitoring)

```jsx
import { ErrorBoundary } from '@sentry/react';

function MyApp() {
  return (
    <ErrorBoundary
      fallback={({ error, componentStack, resetError }) => (
        <div>
          <h2>An error occurred</h2>
          <p>Error: {error.message}</p>
          <button onClick={resetError}>Try again</button>
        </div>
      )}
    >
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

### Option 3: Custom Hook + Error State

For simpler cases, you can use a custom hook to manage error state:

```jsx
// Custom hook for error handling
function useErrorHandler() {
  const [error, setError] = React.useState(null);
  
  const handleError = React.useCallback((error) => {
    console.error('Error caught by hook:', error);
    setError(error);
  }, []);
  
  const resetError = React.useCallback(() => {
    setError(null);
  }, []);
  
  return { error, handleError, resetError };
}

// Functional component with error handling
function SafeFunctionalComponent() {
  const { error, handleError, resetError } = useErrorHandler();
  const [data, setData] = React.useState(null);
  
  React.useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/data');
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        handleError(err);
      }
    };
    
    fetchData();
  }, [handleError]);
  
  if (error) {
    return (
      <div>
        <h3>Error occurred</h3>
        <p>{error.message}</p>
        <button onClick={resetError}>Try Again</button>
      </div>
    );
  }
  
  return <div>Data: {JSON.stringify(data)}</div>;
}
```

### Option 4: Higher-Order Component (HOC) Approach

Create an HOC that wraps functional components with error handling:

```jsx
// HOC for error handling
function withErrorBoundary(WrappedComponent, FallbackComponent) {
  return class ErrorBoundaryWrapper extends React.Component {
    constructor(props) {
      super(props);
      this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error) {
      return { hasError: true, error };
    }

    componentDidCatch(error, errorInfo) {
      console.error('Error in wrapped component:', error, errorInfo);
    }

    render() {
      if (this.state.hasError) {
        return FallbackComponent ? (
          <FallbackComponent error={this.state.error} />
        ) : (
          <div>
            <h3>Component Error</h3>
            <p>{this.state.error?.message}</p>
          </div>
        );
      }
      
      return <WrappedComponent {...this.props} />;
    }
  };
}

// Functional component
function MyFunctionalComponent({ name }) {
  if (name === 'error') {
    throw new Error('Name cannot be "error"');
  }
  return <div>Hello, {name}!</div>;
}

// Custom fallback component
function CustomFallback({ error }) {
  return (
    <div style={{ 
      padding: '15px',
      backgroundColor: '#f0f9ff',
      border: '1px solid #bae6fd',
      borderRadius: '6px'
    }}>
      <h4>Component Error</h4>
      <p>Error: {error?.message}</p>
      <button onClick={() => window.location.reload()}>
        Reload Page
      </button>
    </div>
  );
}

// Usage
const SafeMyComponent = withErrorBoundary(MyFunctionalComponent, CustomFallback);

function App() {
  return (
    <div>
      <SafeMyComponent name="John" />
      <SafeMyComponent name="error" /> {/* This will show error UI */}
    </div>
  );
}
```

### Option 5: React 18+ with Error Boundary Hook (Experimental)

React 18 introduced experimental hooks for error handling:

```jsx
// Note: This is experimental and may not be available in all React versions
import { useErrorBoundary } from 'react-error-boundary';

function FunctionalComponentWithErrorBoundary() {
  const { ErrorBoundary } = useErrorBoundary();
  
  return (
    <ErrorBoundary fallback={<div>Error occurred</div>}>
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

### Best Practices for Functional Components

1. **Use `react-error-boundary` library** for production apps
2. **Keep class-based Error Boundaries simple** when you need to create them
3. **Handle async errors separately** with try-catch blocks
4. **Provide meaningful fallback UI** for better user experience
5. **Log errors appropriately** for debugging and monitoring

### Comparison: Class vs Functional Error Boundaries

| Aspect | Class Component | Functional Component |
|--------|----------------|---------------------|
| **Native Support** | ✅ Built-in | ❌ Requires workarounds |
| **Implementation** | Direct | Wrapper/library needed |
| **Complexity** | Medium | Low (with libraries) |
| **Flexibility** | High | High (with libraries) |
| **Bundle Size** | Small | Larger (with libraries) |

---

## Summary

**For `getDerivedStateFromError` vs `componentDidCatch`:**
- Use `getDerivedStateFromError` to update state and show fallback UI
- Use `componentDidCatch` for logging, analytics, and side effects
- Both work together: first updates state, then performs side effects

**For Functional Components:**
- Use `react-error-boundary` library for the best experience
- Or create simple class-based wrappers
- Handle async errors with try-catch blocks
- Provide good fallback UI and error recovery options

The key is choosing the right approach based on your project's needs and constraints.

--- 