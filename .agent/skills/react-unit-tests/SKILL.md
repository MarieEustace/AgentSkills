---
name: react-unit-tests
description: Write comprehensive unit tests for React applications using Vitest and React Testing Library. Use this skill when the user wants to test React components, hooks, utilities, or set up React testing infrastructure. Trigger for requests like "write tests for my React component", "test this React hook", "set up React testing", "TDD for React", or any mention of testing React code, JSX components, or React Testing Library.
---

# React Unit Tests with Vitest

This skill helps you write maintainable, comprehensive unit tests for React applications using Vitest and React Testing Library, optimized for TDD workflows.

## Core Testing Philosophy

1. **Test behavior, not implementation** - Focus on what the component does, not how it does it
2. **User-centric testing** - Test how users interact with components (clicks, typing, etc.)
3. **Isolated units** - Mock external dependencies (APIs, context, routing)
4. **Clear failure messages** - Descriptive test names and assertions
5. **Edge cases first** - Cover error states, empty states, loading states

## Setup & Configuration

### Install Dependencies

```bash
npm install -D vitest @vitest/ui jsdom
npm install -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

### Vitest Configuration (`vitest.config.js`)

```javascript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.js',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.config.js',
        '**/*.d.ts',
        '**/index.js',
      ],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### Test Setup File (`src/test/setup.js`)

```javascript
import { expect, afterEach } from 'vitest';
import { cleanup } from '@testing-library/react';
import * as matchers from '@testing-library/jest-dom/matchers';

// Extend Vitest's expect with jest-dom matchers
expect.extend(matchers);

// Cleanup after each test
afterEach(() => {
  cleanup();
});
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

## Test Structure & Patterns

### File Organization

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   └── Button.test.jsx
│   └── UserProfile/
│       ├── UserProfile.jsx
│       └── UserProfile.test.jsx
├── hooks/
│   ├── useAuth.js
│   └── useAuth.test.js
└── utils/
    ├── formatters.js
    └── formatters.test.js
```

### Test File Template

```javascript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import ComponentName from './ComponentName';

describe('ComponentName', () => {
  // Setup common test data
  const defaultProps = {
    // ... props
  };

  // Group related tests
  describe('Rendering', () => {
    it('should render with required props', () => {
      // Test basic rendering
    });

    it('should display correct content when data is provided', () => {
      // Test data display
    });
  });

  describe('User Interactions', () => {
    it('should handle click events correctly', async () => {
      // Test interactions
    });
  });

  describe('Edge Cases', () => {
    it('should handle empty data gracefully', () => {
      // Test edge cases
    });

    it('should display error message when loading fails', () => {
      // Test error states
    });
  });
});
```

## Common Testing Patterns

### 1. Basic Component Rendering

```javascript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import Button from './Button';

describe('Button', () => {
  it('should render button with correct text', () => {
    render(<Button>Click me</Button>);
    
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('should apply custom className when provided', () => {
    render(<Button className="custom-class">Click</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('custom-class');
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Click</Button>);
    
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### 2. User Interactions

```javascript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import SearchBar from './SearchBar';

describe('SearchBar', () => {
  it('should call onSearch when user submits query', async () => {
    const user = userEvent.setup();
    const mockOnSearch = vi.fn();
    
    render(<SearchBar onSearch={mockOnSearch} />);
    
    const input = screen.getByRole('textbox');
    const submitButton = screen.getByRole('button', { name: /search/i });
    
    await user.type(input, 'test query');
    await user.click(submitButton);
    
    expect(mockOnSearch).toHaveBeenCalledWith('test query');
    expect(mockOnSearch).toHaveBeenCalledTimes(1);
  });

  it('should not submit empty search query', async () => {
    const user = userEvent.setup();
    const mockOnSearch = vi.fn();
    
    render(<SearchBar onSearch={mockOnSearch} />);
    
    await user.click(screen.getByRole('button', { name: /search/i }));
    
    expect(mockOnSearch).not.toHaveBeenCalled();
  });

  it('should clear input after successful submission', async () => {
    const user = userEvent.setup();
    const mockOnSearch = vi.fn();
    
    render(<SearchBar onSearch={mockOnSearch} />);
    
    const input = screen.getByRole('textbox');
    await user.type(input, 'test');
    await user.click(screen.getByRole('button', { name: /search/i }));
    
    expect(input).toHaveValue('');
  });
});
```

### 3. Async Operations & Loading States

```javascript
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from './UserProfile';

describe('UserProfile', () => {
  it('should display loading state while fetching user data', () => {
    render(<UserProfile userId={1} />);
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('should display user data after successful fetch', async () => {
    const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };
    
    // Mock fetch
    global.fetch = vi.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve(mockUser),
      })
    );
    
    render(<UserProfile userId={1} />);
    
    // Wait for loading to finish
    await waitFor(() => {
      expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
    });
    
    expect(screen.getByText(mockUser.name)).toBeInTheDocument();
    expect(screen.getByText(mockUser.email)).toBeInTheDocument();
  });

  it('should display error message when fetch fails', async () => {
    global.fetch = vi.fn(() => Promise.reject(new Error('Network error')));
    
    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
      expect(screen.getByText(/error loading user/i)).toBeInTheDocument();
    });
  });

  it('should handle 404 response gracefully', async () => {
    global.fetch = vi.fn(() =>
      Promise.resolve({
        ok: false,
        status: 404,
      })
    );
    
    render(<UserProfile userId={999} />);
    
    await waitFor(() => {
      expect(screen.getByText(/user not found/i)).toBeInTheDocument();
    });
  });
});
```

### 4. Mocking External Dependencies

```javascript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginForm from './LoginForm';

// Mock API module
vi.mock('@/api/auth', () => ({
  login: vi.fn(),
}));

import { login } from '@/api/auth';

describe('LoginForm', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should call login API with correct credentials', async () => {
    const user = userEvent.setup();
    login.mockResolvedValue({ token: 'abc123' });
    
    render(<LoginForm />);
    
    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(login).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'password123',
    });
  });

  it('should display error message when login fails', async () => {
    const user = userEvent.setup();
    login.mockRejectedValue(new Error('Invalid credentials'));
    
    render(<LoginForm />);
    
    await user.type(screen.getByLabelText(/email/i), 'wrong@example.com');
    await user.type(screen.getByLabelText(/password/i), 'wrong');
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    await waitFor(() => {
      expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
    });
  });

  it('should disable submit button while logging in', async () => {
    const user = userEvent.setup();
    login.mockImplementation(() => new Promise(resolve => setTimeout(resolve, 100)));
    
    render(<LoginForm />);
    
    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    
    const submitButton = screen.getByRole('button', { name: /log in/i });
    await user.click(submitButton);
    
    expect(submitButton).toBeDisabled();
  });
});
```

### 5. Testing Custom Hooks

```javascript
import { describe, it, expect, vi } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { act } from 'react';
import useCounter from './useCounter';

describe('useCounter', () => {
  it('should initialize with default value of 0', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });

  it('should initialize with custom initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });

  it('should increment count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('should decrement count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });

  it('should not decrement below minimum value', () => {
    const { result } = renderHook(() => useCounter(0, { min: 0 }));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(0);
  });

  it('should reset to initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });
    
    expect(result.current.count).toBe(5);
  });
});
```

### 6. Testing Context & Providers

```javascript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { ThemeProvider } from '@/context/ThemeContext';
import ThemedButton from './ThemedButton';

describe('ThemedButton', () => {
  it('should apply light theme styles by default', () => {
    render(
      <ThemeProvider>
        <ThemedButton>Click me</ThemedButton>
      </ThemeProvider>
    );
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('theme-light');
  });

  it('should apply dark theme styles when theme is dark', () => {
    render(
      <ThemeProvider initialTheme="dark">
        <ThemedButton>Click me</ThemedButton>
      </ThemeProvider>
    );
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('theme-dark');
  });
});
```

### 7. Testing Forms with Validation

```javascript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import ContactForm from './ContactForm';

describe('ContactForm', () => {
  it('should show validation error for invalid email', async () => {
    const user = userEvent.setup();
    
    render(<ContactForm />);
    
    const emailInput = screen.getByLabelText(/email/i);
    await user.type(emailInput, 'invalid-email');
    await user.tab(); // Trigger blur event
    
    expect(screen.getByText(/please enter a valid email/i)).toBeInTheDocument();
  });

  it('should not submit form when required fields are empty', async () => {
    const user = userEvent.setup();
    const mockOnSubmit = vi.fn();
    
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(mockOnSubmit).not.toHaveBeenCalled();
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
  });

  it('should submit form with valid data', async () => {
    const user = userEvent.setup();
    const mockOnSubmit = vi.fn();
    
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    await user.type(screen.getByLabelText(/name/i), 'John Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/message/i), 'Hello!');
    await user.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(mockOnSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com',
      message: 'Hello!',
    });
  });
});
```

## Edge Cases to Always Test

1. **Empty/Null Data**
   - Empty arrays/objects
   - Null/undefined props
   - Missing required data

2. **Loading States**
   - Initial loading
   - Refetching
   - Skeleton/placeholder content

3. **Error States**
   - Network failures
   - API errors (400, 404, 500)
   - Validation errors
   - Permission errors

4. **Boundary Conditions**
   - Minimum/maximum values
   - Very long text/strings
   - Special characters
   - Zero/negative numbers

5. **User Input Edge Cases**
   - Empty input
   - Whitespace-only input
   - Very long input
   - Special characters/SQL injection attempts

## Query Priority (React Testing Library)

Use queries in this order of preference:

1. **getByRole** - Most accessible (button, textbox, heading, etc.)
2. **getByLabelText** - For form fields
3. **getByPlaceholderText** - When label isn't available
4. **getByText** - For non-interactive content
5. **getByTestId** - Last resort (requires adding data-testid)

```javascript
// ✅ GOOD
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText(/email address/i);

// ❌ AVOID
screen.getByTestId('submit-button');
```

## Mocking Best Practices

### Mock External API Calls

```javascript
// Mock entire module
vi.mock('@/api/users', () => ({
  fetchUser: vi.fn(),
  updateUser: vi.fn(),
}));

// Or mock globally in setup file
beforeEach(() => {
  global.fetch = vi.fn();
});
```

### Mock Timers

```javascript
import { vi, beforeEach, afterEach } from 'vitest';

beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.restoreAllMocks();
});

it('should debounce search input', async () => {
  const user = userEvent.setup({ delay: null }); // Disable built-in delay
  const mockOnSearch = vi.fn();
  
  render(<SearchInput onSearch={mockOnSearch} debounceMs={500} />);
  
  await user.type(screen.getByRole('textbox'), 'test');
  
  expect(mockOnSearch).not.toHaveBeenCalled();
  
  vi.advanceTimersByTime(500);
  
  expect(mockOnSearch).toHaveBeenCalledWith('test');
});
```

### Mock Router (React Router)

```javascript
import { MemoryRouter } from 'react-router-dom';

const renderWithRouter = (component, { route = '/' } = {}) => {
  return render(
    <MemoryRouter initialEntries={[route]}>
      {component}
    </MemoryRouter>
  );
};

it('should navigate to user profile on click', async () => {
  const user = userEvent.setup();
  
  renderWithRouter(<UserList />, { route: '/users' });
  
  await user.click(screen.getByText(/john doe/i));
  
  expect(window.location.pathname).toBe('/users/1');
});
```

## Test Naming Convention

Use descriptive names that explain WHAT is being tested and WHAT the expected outcome is:

```javascript
// ✅ GOOD - Clear and descriptive
it('should display error message when email is invalid', () => {});
it('should disable submit button while form is submitting', () => {});
it('should call onSuccess callback after successful API response', () => {});

// ❌ BAD - Vague or implementation-focused
it('should work', () => {});
it('should render correctly', () => {});
it('should call handleClick', () => {});
```

## TDD Workflow

1. **Write the test first** (it will fail - that's expected)
2. **Write minimal code** to make the test pass
3. **Refactor** while keeping tests green
4. **Repeat** for next feature/edge case

Example TDD session:

```javascript
// Step 1: Write failing test
it('should increment counter when + button is clicked', async () => {
  const user = userEvent.setup();
  render(<Counter />);
  
  await user.click(screen.getByRole('button', { name: '+' }));
  
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

// Step 2: Implement minimal code to pass test
// (Create Counter component with increment functionality)

// Step 3: Refactor if needed while keeping test green
```

## Common Pitfalls to Avoid

1. **Don't test implementation details**
   ```javascript
   // ❌ BAD - Testing internal state
   expect(component.state.count).toBe(1);
   
   // ✅ GOOD - Testing user-visible output
   expect(screen.getByText('Count: 1')).toBeInTheDocument();
   ```

2. **Don't use test IDs unnecessarily**
   ```javascript
   // ❌ BAD
   screen.getByTestId('submit-btn');
   
   // ✅ GOOD
   screen.getByRole('button', { name: /submit/i });
   ```

3. **Don't forget to cleanup mocks**
   ```javascript
   beforeEach(() => {
     vi.clearAllMocks(); // Reset mock call counts
   });
   ```

4. **Don't make tests too coupled**
   - Each test should be independent
   - Tests shouldn't rely on execution order
   - Use beforeEach for shared setup

## Output Guidelines

When writing tests for users:

1. **Ask about the component/functionality** to understand:
   - What does it do?
   - What are the user interactions?
   - What are the edge cases?
   - Are there external dependencies (API, context, etc.)?

2. **Generate comprehensive test file** with:
   - Clear describe blocks grouping related tests
   - All edge cases covered
   - Proper mocking setup
   - Descriptive test names

3. **Include setup instructions** if needed:
   - Required dependencies
   - Mock setup
   - Test utilities

4. **Explain the tests** briefly so the user understands what's being tested and why
