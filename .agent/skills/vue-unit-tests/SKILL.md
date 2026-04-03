---
name: vue-unit-tests
description: Write comprehensive unit tests for Vue 3 applications with TypeScript using Vitest and Vue Test Utils. Use this skill when the user wants to test Vue components, composables, Vuetify components, or set up Vue testing infrastructure. Trigger for requests like "write tests for my Vue component", "test this composable", "set up Vue 3 testing", "TDD for Vue", "test Vuetify component", or any mention of testing Vue 3 code, SFC (Single File Components), or Vue Test Utils.
---

# Vue 3 Unit Tests with Vitest & TypeScript

This skill helps you write maintainable, comprehensive unit tests for Vue 3 applications with TypeScript and Vuetify, using Vitest and Vue Test Utils, optimized for TDD workflows.

## Core Testing Philosophy

1. **Test component behavior** - Focus on props, events, slots, and user interactions
2. **User-centric testing** - Test from the user's perspective
3. **Isolated units** - Mock composables, stores, and external dependencies
4. **Clear failure messages** - Descriptive test names and assertions
5. **Edge cases first** - Cover error states, empty states, loading states

## Setup & Configuration

### Install Dependencies

```bash
npm install -D vitest @vitest/ui jsdom
npm install -D @vue/test-utils @testing-library/vue @testing-library/user-event
npm install -D @testing-library/jest-dom
```

### Vitest Configuration (`vitest.config.ts`)

```typescript
import { defineConfig } from 'vitest/config';
import vue from '@vitejs/plugin-vue';
import vuetify from 'vite-plugin-vuetify';
import path from 'path';

export default defineConfig({
  plugins: [
    vue(),
    vuetify({ autoImport: true }),
  ],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.config.ts',
        '**/*.d.ts',
        '**/index.ts',
        '**/*.stories.ts',
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

### Test Setup File (`src/test/setup.ts`)

```typescript
import { expect, afterEach, vi } from 'vitest';
import { cleanup } from '@testing-library/vue';
import * as matchers from '@testing-library/jest-dom/matchers';
import { config } from '@vue/test-utils';

// Extend Vitest's expect with jest-dom matchers
expect.extend(matchers);

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock Vuetify components globally if needed
config.global.stubs = {
  // Only stub components that cause issues in tests
  'v-dialog': true,
};

// Suppress Vuetify warnings in tests
global.console.warn = vi.fn();
```

### Vuetify Test Helper (`src/test/vuetify.ts`)

```typescript
import { createVuetify } from 'vuetify';
import * as components from 'vuetify/components';
import * as directives from 'vuetify/directives';

export function createVuetifyForTest() {
  return createVuetify({
    components,
    directives,
  });
}
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
│   ├── AppButton/
│   │   ├── AppButton.vue
│   │   └── AppButton.test.ts
│   └── UserProfile/
│       ├── UserProfile.vue
│       └── UserProfile.test.ts
├── composables/
│   ├── useAuth.ts
│   └── useAuth.test.ts
└── utils/
    ├── formatters.ts
    └── formatters.test.ts
```

### Test File Template

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { mount, VueWrapper } from '@vue/test-utils';
import { createVuetifyForTest } from '@/test/vuetify';
import ComponentName from './ComponentName.vue';

describe('ComponentName', () => {
  let wrapper: VueWrapper;
  const vuetify = createVuetifyForTest();

  const defaultProps = {
    // ... props
  };

  const createWrapper = (props = {}) => {
    return mount(ComponentName, {
      props: { ...defaultProps, ...props },
      global: {
        plugins: [vuetify],
      },
    });
  };

  afterEach(() => {
    wrapper?.unmount();
  });

  describe('Rendering', () => {
    it('should render with required props', () => {
      wrapper = createWrapper();
      expect(wrapper.exists()).toBe(true);
    });
  });

  describe('User Interactions', () => {
    it('should emit event when user interacts', async () => {
      wrapper = createWrapper();
      // Test interactions
    });
  });

  describe('Edge Cases', () => {
    it('should handle empty data gracefully', () => {
      wrapper = createWrapper({ data: [] });
      // Test edge cases
    });
  });
});
```

## Common Testing Patterns

### 1. Basic Component Rendering

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import { createVuetifyForTest } from '@/test/vuetify';
import AppButton from './AppButton.vue';

describe('AppButton', () => {
  const vuetify = createVuetifyForTest();

  it('should render button with correct text', () => {
    const wrapper = mount(AppButton, {
      props: { text: 'Click me' },
      global: { plugins: [vuetify] },
    });

    expect(wrapper.text()).toContain('Click me');
  });

  it('should apply primary color when color prop is primary', () => {
    const wrapper = mount(AppButton, {
      props: { color: 'primary' },
      global: { plugins: [vuetify] },
    });

    expect(wrapper.find('.v-btn').classes()).toContain('bg-primary');
  });

  it('should be disabled when disabled prop is true', () => {
    const wrapper = mount(AppButton, {
      props: { disabled: true },
      global: { plugins: [vuetify] },
    });

    expect(wrapper.find('button').attributes('disabled')).toBeDefined();
  });

  it('should render icon when icon prop is provided', () => {
    const wrapper = mount(AppButton, {
      props: { icon: 'mdi-check' },
      global: { plugins: [vuetify] },
    });

    expect(wrapper.find('.mdi-check').exists()).toBe(true);
  });
});
```

### 2. Testing Emitted Events

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import SearchBar from './SearchBar.vue';

describe('SearchBar', () => {
  it('should emit search event with query when user submits', async () => {
    const wrapper = mount(SearchBar);

    const input = wrapper.find('input[type="text"]');
    await input.setValue('test query');

    const form = wrapper.find('form');
    await form.trigger('submit');

    expect(wrapper.emitted('search')).toBeTruthy();
    expect(wrapper.emitted('search')?.[0]).toEqual(['test query']);
  });

  it('should emit update:modelValue when input changes', async () => {
    const wrapper = mount(SearchBar, {
      props: { modelValue: '' },
    });

    await wrapper.find('input').setValue('new value');

    expect(wrapper.emitted('update:modelValue')).toBeTruthy();
    expect(wrapper.emitted('update:modelValue')?.[0]).toEqual(['new value']);
  });

  it('should not emit search event when query is empty', async () => {
    const wrapper = mount(SearchBar);

    await wrapper.find('form').trigger('submit');

    expect(wrapper.emitted('search')).toBeFalsy();
  });

  it('should emit clear event when clear button is clicked', async () => {
    const wrapper = mount(SearchBar, {
      props: { modelValue: 'test' },
    });

    await wrapper.find('[data-test="clear-button"]').trigger('click');

    expect(wrapper.emitted('clear')).toBeTruthy();
  });
});
```

### 3. Testing Props & Reactivity

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import UserCard from './UserCard.vue';

interface User {
  id: number;
  name: string;
  email: string;
}

describe('UserCard', () => {
  const mockUser: User = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
  };

  it('should display user name from props', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
    });

    expect(wrapper.text()).toContain('John Doe');
  });

  it('should update display when user prop changes', async () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser },
    });

    const newUser: User = { id: 2, name: 'Jane Smith', email: 'jane@example.com' };
    await wrapper.setProps({ user: newUser });

    expect(wrapper.text()).toContain('Jane Smith');
    expect(wrapper.text()).not.toContain('John Doe');
  });

  it('should apply custom class from props', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser, customClass: 'highlighted' },
    });

    expect(wrapper.classes()).toContain('highlighted');
  });

  it('should use default avatar when user has no avatar', () => {
    const wrapper = mount(UserCard, {
      props: { user: { ...mockUser, avatar: null } },
    });

    expect(wrapper.find('img').attributes('src')).toContain('default-avatar.png');
  });
});
```

### 4. Testing Slots

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import Card from './Card.vue';

describe('Card', () => {
  it('should render default slot content', () => {
    const wrapper = mount(Card, {
      slots: {
        default: '<div>Card content</div>',
      },
    });

    expect(wrapper.text()).toContain('Card content');
  });

  it('should render named slots correctly', () => {
    const wrapper = mount(Card, {
      slots: {
        header: '<h2>Card Title</h2>',
        default: '<p>Card body</p>',
        footer: '<button>Action</button>',
      },
    });

    expect(wrapper.find('h2').text()).toBe('Card Title');
    expect(wrapper.find('p').text()).toBe('Card body');
    expect(wrapper.find('button').text()).toBe('Action');
  });

  it('should pass slot props correctly', () => {
    const wrapper = mount(Card, {
      slots: {
        default: `
          <template #default="{ item }">
            <span>{{ item.name }}</span>
          </template>
        `,
      },
    });

    // Test scoped slot props are passed correctly
    expect(wrapper.html()).toContain('name');
  });

  it('should render fallback content when slot is not provided', () => {
    const wrapper = mount(Card);

    expect(wrapper.text()).toContain('Default card content');
  });
});
```

### 5. Testing Composables

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { ref } from 'vue';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should initialize with default value of 0', () => {
    const { count } = useCounter();

    expect(count.value).toBe(0);
  });

  it('should initialize with custom initial value', () => {
    const { count } = useCounter(10);

    expect(count.value).toBe(10);
  });

  it('should increment count', () => {
    const { count, increment } = useCounter();

    increment();

    expect(count.value).toBe(1);
  });

  it('should decrement count', () => {
    const { count, decrement } = useCounter(5);

    decrement();

    expect(count.value).toBe(4);
  });

  it('should not decrement below minimum value', () => {
    const { count, decrement } = useCounter(0, { min: 0 });

    decrement();

    expect(count.value).toBe(0);
  });

  it('should reset to initial value', () => {
    const { count, increment, reset } = useCounter(5);

    increment();
    increment();
    reset();

    expect(count.value).toBe(5);
  });

  it('should handle async operations correctly', async () => {
    const { data, isLoading, fetchData } = useAsyncData();

    expect(isLoading.value).toBe(false);

    const promise = fetchData();
    expect(isLoading.value).toBe(true);

    await promise;
    expect(isLoading.value).toBe(false);
    expect(data.value).toBeDefined();
  });
});
```

### 6. Testing Async Operations & API Calls

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { mount, flushPromises } from '@vue/test-utils';
import UserProfile from './UserProfile.vue';

// Mock API
vi.mock('@/api/users', () => ({
  fetchUser: vi.fn(),
}));

import { fetchUser } from '@/api/users';

describe('UserProfile', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should display loading state while fetching user', () => {
    const wrapper = mount(UserProfile, {
      props: { userId: 1 },
    });

    expect(wrapper.text()).toContain('Loading');
  });

  it('should display user data after successful fetch', async () => {
    const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };
    (fetchUser as any).mockResolvedValue(mockUser);

    const wrapper = mount(UserProfile, {
      props: { userId: 1 },
    });

    await flushPromises();

    expect(wrapper.text()).toContain('John Doe');
    expect(wrapper.text()).toContain('john@example.com');
    expect(wrapper.text()).not.toContain('Loading');
  });

  it('should display error message when fetch fails', async () => {
    (fetchUser as any).mockRejectedValue(new Error('Network error'));

    const wrapper = mount(UserProfile, {
      props: { userId: 1 },
    });

    await flushPromises();

    expect(wrapper.text()).toContain('Error loading user');
  });

  it('should handle 404 response gracefully', async () => {
    (fetchUser as any).mockRejectedValue({ status: 404 });

    const wrapper = mount(UserProfile, {
      props: { userId: 999 },
    });

    await flushPromises();

    expect(wrapper.text()).toContain('User not found');
  });

  it('should refetch user data when userId prop changes', async () => {
    const mockUser1 = { id: 1, name: 'User 1' };
    const mockUser2 = { id: 2, name: 'User 2' };
    (fetchUser as any).mockResolvedValueOnce(mockUser1);

    const wrapper = mount(UserProfile, {
      props: { userId: 1 },
    });

    await flushPromises();
    expect(wrapper.text()).toContain('User 1');

    (fetchUser as any).mockResolvedValueOnce(mockUser2);
    await wrapper.setProps({ userId: 2 });
    await flushPromises();

    expect(wrapper.text()).toContain('User 2');
    expect(fetchUser).toHaveBeenCalledTimes(2);
  });
});
```

### 7. Testing Vuetify Components

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import { createVuetifyForTest } from '@/test/vuetify';
import DataTable from './DataTable.vue';

describe('DataTable', () => {
  const vuetify = createVuetifyForTest();

  const defaultProps = {
    headers: [
      { title: 'Name', key: 'name' },
      { title: 'Email', key: 'email' },
    ],
    items: [
      { id: 1, name: 'John', email: 'john@example.com' },
      { id: 2, name: 'Jane', email: 'jane@example.com' },
    ],
  };

  it('should render Vuetify data table with items', () => {
    const wrapper = mount(DataTable, {
      props: defaultProps,
      global: { plugins: [vuetify] },
    });

    expect(wrapper.findComponent({ name: 'v-data-table' }).exists()).toBe(true);
    expect(wrapper.text()).toContain('John');
    expect(wrapper.text()).toContain('Jane');
  });

  it('should emit row-click event when row is clicked', async () => {
    const wrapper = mount(DataTable, {
      props: defaultProps,
      global: { plugins: [vuetify] },
    });

    const firstRow = wrapper.find('tbody tr');
    await firstRow.trigger('click');

    expect(wrapper.emitted('row-click')).toBeTruthy();
    expect(wrapper.emitted('row-click')?.[0]).toEqual([defaultProps.items[0]]);
  });

  it('should show empty state when items array is empty', () => {
    const wrapper = mount(DataTable, {
      props: { ...defaultProps, items: [] },
      global: { plugins: [vuetify] },
    });

    expect(wrapper.text()).toContain('No data available');
  });

  it('should apply sorting when header is clicked', async () => {
    const wrapper = mount(DataTable, {
      props: defaultProps,
      global: { plugins: [vuetify] },
    });

    const nameHeader = wrapper.find('th:first-child');
    await nameHeader.trigger('click');

    expect(wrapper.emitted('update:sort-by')).toBeTruthy();
  });
});
```

### 8. Testing Forms with Validation

```typescript
import { describe, it, expect, vi } from 'vitest';
import { mount, VueWrapper } from '@vue/test-utils';
import { createVuetifyForTest } from '@/test/vuetify';
import LoginForm from './LoginForm.vue';

describe('LoginForm', () => {
  const vuetify = createVuetifyForTest();
  let wrapper: VueWrapper;

  afterEach(() => {
    wrapper?.unmount();
  });

  it('should show validation error for invalid email', async () => {
    wrapper = mount(LoginForm, {
      global: { plugins: [vuetify] },
    });

    const emailInput = wrapper.find('input[type="email"]');
    await emailInput.setValue('invalid-email');
    await emailInput.trigger('blur');

    expect(wrapper.text()).toContain('Please enter a valid email');
  });

  it('should show required field errors when form is empty', async () => {
    wrapper = mount(LoginForm, {
      global: { plugins: [vuetify] },
    });

    const form = wrapper.findComponent({ name: 'v-form' });
    await form.trigger('submit.prevent');

    expect(wrapper.text()).toContain('Email is required');
    expect(wrapper.text()).toContain('Password is required');
  });

  it('should emit submit event with valid credentials', async () => {
    wrapper = mount(LoginForm, {
      global: { plugins: [vuetify] },
    });

    await wrapper.find('input[type="email"]').setValue('user@example.com');
    await wrapper.find('input[type="password"]').setValue('password123');
    await wrapper.find('form').trigger('submit.prevent');

    expect(wrapper.emitted('submit')).toBeTruthy();
    expect(wrapper.emitted('submit')?.[0]).toEqual([{
      email: 'user@example.com',
      password: 'password123',
    }]);
  });

  it('should disable submit button while form is invalid', async () => {
    wrapper = mount(LoginForm, {
      global: { plugins: [vuetify] },
    });

    const submitButton = wrapper.find('button[type="submit"]');
    expect(submitButton.attributes('disabled')).toBeDefined();
  });

  it('should enable submit button when form is valid', async () => {
    wrapper = mount(LoginForm, {
      global: { plugins: [vuetify] },
    });

    await wrapper.find('input[type="email"]').setValue('user@example.com');
    await wrapper.find('input[type="password"]').setValue('password123');

    const submitButton = wrapper.find('button[type="submit"]');
    expect(submitButton.attributes('disabled')).toBeUndefined();
  });
});
```

### 9. Testing Pinia Stores

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { setActivePinia, createPinia } from 'pinia';
import { useUserStore } from './userStore';

describe('useUserStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('should initialize with empty user state', () => {
    const store = useUserStore();

    expect(store.user).toBeNull();
    expect(store.isAuthenticated).toBe(false);
  });

  it('should set user on login', () => {
    const store = useUserStore();
    const mockUser = { id: 1, name: 'John', email: 'john@example.com' };

    store.setUser(mockUser);

    expect(store.user).toEqual(mockUser);
    expect(store.isAuthenticated).toBe(true);
  });

  it('should clear user on logout', () => {
    const store = useUserStore();
    const mockUser = { id: 1, name: 'John' };

    store.setUser(mockUser);
    store.logout();

    expect(store.user).toBeNull();
    expect(store.isAuthenticated).toBe(false);
  });

  it('should compute full name correctly', () => {
    const store = useUserStore();

    store.setUser({ firstName: 'John', lastName: 'Doe' });

    expect(store.fullName).toBe('John Doe');
  });
});
```

### 10. Testing with Provide/Inject

```typescript
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

describe('ChildComponent with Provide/Inject', () => {
  it('should receive injected value from parent', () => {
    const theme = ref('dark');

    const wrapper = mount(ChildComponent, {
      global: {
        provide: {
          theme,
        },
      },
    });

    expect(wrapper.text()).toContain('dark');
  });

  it('should react to changes in injected value', async () => {
    const theme = ref('light');

    const wrapper = mount(ChildComponent, {
      global: {
        provide: {
          theme,
        },
      },
    });

    expect(wrapper.text()).toContain('light');

    theme.value = 'dark';
    await wrapper.vm.$nextTick();

    expect(wrapper.text()).toContain('dark');
  });
});
```

## Edge Cases to Always Test

1. **Empty/Null Data**
   - Empty arrays/objects
   - Null/undefined props
   - Missing optional props

2. **Loading States**
   - Initial loading
   - Refetching data
   - Skeleton screens

3. **Error States**
   - Network failures
   - API errors (400, 404, 500)
   - Validation errors

4. **Boundary Conditions**
   - Min/max values
   - Very long text
   - Special characters
   - Zero/negative numbers

5. **Reactive Updates**
   - Prop changes
   - Store state changes
   - Computed property updates

## Query Selectors Priority

1. **Component selectors** - `findComponent({ name: 'v-btn' })`
2. **Role/semantic** - `find('button')`, `find('input[type="email"]')`
3. **Data attributes** - `find('[data-test="submit-btn"]')`
4. **Text content** - `wrapper.text()` for verification
5. **Classes** - Last resort

```typescript
// ✅ GOOD
wrapper.findComponent({ name: 'v-btn' });
wrapper.find('button[type="submit"]');

// ❌ AVOID
wrapper.find('.v-btn--primary');
```

## Mocking Best Practices

### Mock API Calls

```typescript
// Mock entire module
vi.mock('@/api/users', () => ({
  fetchUsers: vi.fn(),
  createUser: vi.fn(),
}));
```

### Mock Composables

```typescript
vi.mock('@/composables/useAuth', () => ({
  useAuth: vi.fn(() => ({
    user: ref({ id: 1, name: 'Test User' }),
    isAuthenticated: ref(true),
    login: vi.fn(),
    logout: vi.fn(),
  })),
}));
```

### Mock Router

```typescript
import { mount } from '@vue/test-utils';
import { createRouter, createMemoryHistory } from 'vue-router';

const router = createRouter({
  history: createMemoryHistory(),
  routes: [
    { path: '/', component: { template: '<div>Home</div>' } },
    { path: '/about', component: { template: '<div>About</div>' } },
  ],
});

const wrapper = mount(Component, {
  global: {
    plugins: [router],
  },
});
```

## Test Naming Convention

Use descriptive names explaining the scenario and expected outcome:

```typescript
// ✅ GOOD
it('should display error message when email format is invalid', () => {});
it('should emit update event when user changes input value', () => {});
it('should disable submit button while API request is pending', () => {});

// ❌ BAD
it('works correctly', () => {});
it('renders', () => {});
it('calls method', () => {});
```

## TDD Workflow for Vue

1. **Write failing test** for the feature/component
2. **Implement minimum code** to pass the test
3. **Refactor** while keeping tests green
4. **Add edge case tests** and repeat

## Common Pitfalls to Avoid

1. **Don't test Vue internals**
   ```typescript
   // ❌ BAD
   expect(wrapper.vm.internalState).toBe(true);
   
   // ✅ GOOD
   expect(wrapper.find('.active').exists()).toBe(true);
   ```

2. **Always await async operations**
   ```typescript
   // ❌ BAD
   wrapper.find('button').trigger('click');
   expect(wrapper.emitted('click')).toBeTruthy();
   
   // ✅ GOOD
   await wrapper.find('button').trigger('click');
   expect(wrapper.emitted('click')).toBeTruthy();
   ```

3. **Clean up after tests**
   ```typescript
   afterEach(() => {
     wrapper?.unmount();
     vi.clearAllMocks();
   });
   ```

## Output Guidelines

When writing tests:

1. **Understand the component** - Ask about:
   - Props and their types
   - Events emitted
   - External dependencies (stores, composables, APIs)
   - Vuetify components used

2. **Generate comprehensive test file** with:
   - Proper TypeScript types
   - Vuetify setup if needed
   - All edge cases
   - Clear test organization

3. **Include setup code** for:
   - Vuetify configuration
   - Mock stores/composables
   - Router if needed

4. **Explain tests briefly** so the user understands what's covered
