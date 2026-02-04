5.3 CI/CD
Testing pipelines
Visual regression
Bundle monitoring
Preview builds

========================================================================================================================================================================================================================


<!--  -->





# Phase 5.3: CI/CD - Complete Deep Dive Guide

## Table of Contents
1. [CI/CD Fundamentals](#cicd-fundamentals)
2. [Testing Pipelines](#testing-pipelines)
3. [Visual Regression Testing](#visual-regression-testing)
4. [Bundle Monitoring](#bundle-monitoring)
5. [Preview Builds](#preview-builds)
6. [Best Practices & Day-to-Day Coding](#best-practices)
7. [Common Interview Questions](#interview-questions)

---

## CI/CD Fundamentals

### What is CI/CD?

**Continuous Integration (CI)**: Automatically testing and validating code changes when they're pushed to version control.

**Continuous Deployment (CD)**: Automatically deploying validated code changes to production or staging environments.

### Core Concepts

```yaml
# .github/workflows/ci-cd.yml - Complete CI/CD Example
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '20.x'
  CACHE_VERSION: v1

jobs:
  # Job 1: Code Quality & Linting
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier
        run: npm run format:check
      
      - name: Type Check
        run: npm run type-check

  # Job 2: Unit & Integration Tests
  test:
    runs-on: ubuntu-latest
    needs: quality-check
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          flags: unittests
          name: codecov-umbrella

  # Job 3: E2E Tests
  e2e:
    runs-on: ubuntu-latest
    needs: quality-check
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Build application
        run: npm run build
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/

  # Job 4: Build & Bundle Analysis
  build:
    runs-on: ubuntu-latest
    needs: [test, e2e]
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          NODE_ENV: production
      
      - name: Analyze bundle
        run: npm run analyze
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-artifacts
          path: dist/
          retention-days: 7
      
      - name: Upload bundle stats
        uses: actions/upload-artifact@v3
        with:
          name: bundle-stats
          path: stats.json

  # Job 5: Security Scanning
  security:
    runs-on: ubuntu-latest
    needs: quality-check
    steps:
      - uses: actions/checkout@v4
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Run OWASP dependency check
        run: npx audit-ci --moderate

  # Job 6: Deploy to Preview
  deploy-preview:
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-artifacts
          path: dist/
      
      - name: Deploy to Vercel Preview
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          scope: ${{ secrets.VERCEL_SCOPE }}
      
      - name: Comment PR with preview URL
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview deployed! Check it out at: ${{ steps.deploy.outputs.preview-url }}'
            })

  # Job 7: Deploy to Production
  deploy-production:
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment:
      name: production
      url: https://myapp.com
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-artifacts
          path: dist/
      
      - name: Deploy to production
        run: |
          # Your deployment script
          npm run deploy:prod
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
      
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### CI/CD Platform Comparison

```javascript
// Platform-specific configurations comparison

// 1. GitHub Actions (Above example)
// Pros: Native GitHub integration, free for public repos
// Cons: Can be expensive for private repos with heavy usage

// 2. GitLab CI
// .gitlab-ci.yml
const gitlabConfig = `
stages:
  - test
  - build
  - deploy

cache:
  paths:
    - node_modules/

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run test
  coverage: '/Lines\\s*:\\s*(\\d+\\.\\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  only:
    - main
    - develop

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST $DEPLOY_WEBHOOK
  only:
    - main
`;

// 3. CircleCI
// .circleci/config.yml
const circleConfig = `
version: 2.1

orbs:
  node: circleci/node@5.1.0

workflows:
  build-test-deploy:
    jobs:
      - node/test:
          version: '20.0'
      - build:
          requires:
            - node/test
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: main

jobs:
  build:
    docker:
      - image: cimg/node:20.0
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm
      - run:
          name: Build application
          command: npm run build
      - persist_to_workspace:
          root: .
          paths:
            - dist

  deploy:
    docker:
      - image: cimg/node:20.0
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Deploy
          command: npm run deploy
`;

// 4. Jenkins (Jenkinsfile)
const jenkinsConfig = `
pipeline {
    agent {
        docker {
            image 'node:20-alpine'
        }
    }
    
    environment {
        CI = 'true'
    }
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('E2E Tests') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'npm run deploy'
            }
        }
    }
    
    post {
        always {
            junit 'test-results/**/*.xml'
            publishHTML([
                reportDir: 'coverage',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
    }
}
`;
```

---

## Testing Pipelines

### 1. Test Pyramid Structure

```javascript
// Test pyramid implementation in CI/CD

/*
        /\
       /E2E\        10%  - Slow, expensive, brittle
      /------\
     /  API  \      20%  - Medium speed, integration
    /----------\
   /   Unit    \    70%  - Fast, cheap, isolated
  /--------------\
*/

// package.json test scripts
{
  "scripts": {
    "test": "npm run test:unit && npm run test:integration && npm run test:e2e",
    "test:unit": "vitest run --coverage",
    "test:unit:watch": "vitest",
    "test:integration": "vitest run --config vitest.integration.config.ts",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:visual": "percy exec -- playwright test",
    "test:a11y": "playwright test --grep @accessibility",
    "test:smoke": "playwright test --grep @smoke",
    "test:ci": "npm run test:unit && npm run test:integration && npm run test:e2e -- --reporter=junit",
    "test:coverage": "vitest run --coverage && open coverage/index.html"
  }
}
```

### 2. Unit Testing Pipeline

```javascript
// vitest.config.ts - Complete configuration
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      include: ['src/**/*.{ts,tsx}'],
      exclude: [
        'src/**/*.test.{ts,tsx}',
        'src/**/*.spec.{ts,tsx}',
        'src/test/**',
        'src/**/*.d.ts',
        'src/main.tsx',
        'src/vite-env.d.ts'
      ],
      // Coverage thresholds - fail if below
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80
      }
    },
    // Parallel execution for speed
    poolOptions: {
      threads: {
        singleThread: false,
        minThreads: 1,
        maxThreads: 4
      }
    },
    // Test timeout
    testTimeout: 10000,
    hookTimeout: 10000,
    // Reporter configuration
    reporters: process.env.CI 
      ? ['junit', 'json', 'verbose']
      : ['verbose'],
    outputFile: {
      junit: './test-results/junit.xml',
      json: './test-results/results.json'
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@store': path.resolve(__dirname, './src/store')
    }
  }
});

// src/test/setup.ts - Test setup file
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, beforeAll, afterAll, vi } from 'vitest';
import { server } from './mocks/server';

// Mock environment variables
process.env.VITE_API_URL = 'http://localhost:3000/api';

// Start MSW server before all tests
beforeAll(() => {
  server.listen({ onUnhandledRequest: 'error' });
});

// Reset handlers after each test
afterEach(() => {
  cleanup();
  server.resetHandlers();
  vi.clearAllMocks();
  localStorage.clear();
  sessionStorage.clear();
});

// Clean up after all tests
afterAll(() => {
  server.close();
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  takeRecords() {
    return [];
  }
  unobserve() {}
};

// Mock ResizeObserver
global.ResizeObserver = class ResizeObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Example unit test with best practices
// src/components/Button/Button.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button Component', () => {
  // Test 1: Rendering
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  // Test 2: Props
  it('applies variant classes correctly', () => {
    const { rerender } = render(<Button variant="primary">Primary</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-primary');
    
    rerender(<Button variant="secondary">Secondary</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-secondary');
  });

  // Test 3: User interactions
  it('calls onClick handler when clicked', async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  // Test 4: Disabled state
  it('does not call onClick when disabled', async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick} disabled>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    expect(handleClick).not.toHaveBeenCalled();
    expect(screen.getByRole('button')).toBeDisabled();
  });

  // Test 5: Loading state
  it('shows loading spinner when loading', () => {
    render(<Button isLoading>Submit</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });

  // Test 6: Accessibility
  it('has correct aria attributes', () => {
    render(
      <Button aria-label="Delete item" aria-describedby="delete-description">
        Delete
      </Button>
    );
    
    const button = screen.getByRole('button');
    expect(button).toHaveAttribute('aria-label', 'Delete item');
    expect(button).toHaveAttribute('aria-describedby', 'delete-description');
  });

  // Test 7: Async operations
  it('handles async onClick correctly', async () => {
    const asyncClick = vi.fn().mockResolvedValue('success');
    const user = userEvent.setup();
    
    render(<Button onClick={asyncClick}>Submit</Button>);
    
    await user.click(screen.getByRole('button'));
    
    await waitFor(() => {
      expect(asyncClick).toHaveBeenCalled();
    });
  });
});
```

### 3. Integration Testing Pipeline

```javascript
// vitest.integration.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/integration-setup.ts'],
    include: ['src/**/*.integration.test.{ts,tsx}'],
    testTimeout: 30000, // Longer timeout for integration tests
    poolOptions: {
      threads: {
        singleThread: false,
        maxThreads: 2 // Fewer threads than unit tests
      }
    }
  }
});

// Example integration test
// src/features/auth/auth.integration.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, beforeEach } from 'vitest';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';
import { LoginPage } from './LoginPage';
import { server } from '@/test/mocks/server';
import { http, HttpResponse } from 'msw';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        {children}
      </BrowserRouter>
    </QueryClientProvider>
  );
};

describe('Authentication Flow Integration', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('completes full login flow successfully', async () => {
    const user = userEvent.setup();
    
    // Mock successful login
    server.use(
      http.post('/api/auth/login', () => {
        return HttpResponse.json({
          token: 'fake-jwt-token',
          user: { id: 1, email: 'test@example.com' }
        });
      })
    );

    render(<LoginPage />, { wrapper: createWrapper() });

    // Fill in form
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    
    // Submit form
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    // Wait for success state
    await waitFor(() => {
      expect(localStorage.getItem('token')).toBe('fake-jwt-token');
    });

    // Verify redirect or success message
    expect(screen.getByText(/welcome back/i)).toBeInTheDocument();
  });

  it('handles login errors correctly', async () => {
    const user = userEvent.setup();
    
    // Mock failed login
    server.use(
      http.post('/api/auth/login', () => {
        return HttpResponse.json(
          { message: 'Invalid credentials' },
          { status: 401 }
        );
      })
    );

    render(<LoginPage />, { wrapper: createWrapper() });

    await user.type(screen.getByLabelText(/email/i), 'wrong@example.com');
    await user.type(screen.getByLabelText(/password/i), 'wrongpassword');
    await user.click(screen.getByRole('button', { name: /sign in/i }));

    await waitFor(() => {
      expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
    });

    expect(localStorage.getItem('token')).toBeNull();
  });
});
```

### 4. E2E Testing Pipeline

```javascript
// playwright.config.ts - Production-ready configuration
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: process.env.CI 
    ? [
        ['html'],
        ['junit', { outputFile: 'test-results/junit.xml' }],
        ['json', { outputFile: 'test-results/results.json' }],
        ['github']
      ]
    : [['html'], ['list']],
  
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
    navigationTimeout: 30000,
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});

// e2e/example.spec.ts - Comprehensive E2E test
import { test, expect } from '@playwright/test';

test.describe('User Authentication E2E', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should complete registration flow', async ({ page }) => {
    // Navigate to registration
    await page.click('text=Sign Up');
    await expect(page).toHaveURL('/register');

    // Fill registration form
    await page.fill('input[name="email"]', 'newuser@example.com');
    await page.fill('input[name="password"]', 'SecurePass123!');
    await page.fill('input[name="confirmPassword"]', 'SecurePass123!');
    
    // Submit form
    await page.click('button[type="submit"]');

    // Wait for success
    await expect(page.locator('text=Welcome!')).toBeVisible({ timeout: 5000 });
    await expect(page).toHaveURL('/dashboard');
  });

  test('should handle form validation', async ({ page }) => {
    await page.click('text=Sign Up');
    
    // Try to submit empty form
    await page.click('button[type="submit"]');

    // Check for validation errors
    await expect(page.locator('text=Email is required')).toBeVisible();
    await expect(page.locator('text=Password is required')).toBeVisible();
  });

  test('should persist login across page refreshes', async ({ page, context }) => {
    // Login
    await page.click('text=Sign In');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    
    await expect(page).toHaveURL('/dashboard');

    // Refresh page
    await page.reload();

    // Should still be logged in
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Dashboard')).toBeVisible();
  });

  test('should work offline', async ({ page, context }) => {
    // Go online first
    await context.setOffline(false);
    await page.goto('/');

    // Go offline
    await context.setOffline(true);
    
    // Check offline indicator
    await expect(page.locator('text=You are offline')).toBeVisible();

    // Try to navigate
    await page.click('text=Products');
    
    // Should show cached content
    await expect(page.locator('text=Offline Mode')).toBeVisible();
  });
});

// Page Object Model example
// e2e/pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('input[name="email"]');
    this.passwordInput = page.locator('input[name="password"]');
    this.submitButton = page.locator('button[type="submit"]');
    this.errorMessage = page.locator('[role="alert"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage() {
    return this.errorMessage.textContent();
  }
}

// Usage in test
test('login with page object', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('test@example.com', 'password123');
  await expect(page).toHaveURL('/dashboard');
});
```

### 5. Test Data Management

```javascript
// src/test/fixtures/user.fixture.ts
import { faker } from '@faker-js/faker';

export const createMockUser = (overrides = {}) => ({
  id: faker.string.uuid(),
  email: faker.internet.email(),
  firstName: faker.person.firstName(),
  lastName: faker.person.lastName(),
  avatar: faker.image.avatar(),
  createdAt: faker.date.past().toISOString(),
  ...overrides
});

export const createMockUsers = (count: number) => 
  Array.from({ length: count }, () => createMockUser());

// src/test/mocks/handlers.ts
import { http, HttpResponse } from 'msw';
import { createMockUser } from '../fixtures/user.fixture';

export const handlers = [
  // GET /api/users
  http.get('/api/users', () => {
    return HttpResponse.json({
      users: createMockUsers(10),
      total: 100,
      page: 1,
      pageSize: 10
    });
  }),

  // POST /api/users
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json(
      createMockUser(body),
      { status: 201 }
    );
  }),

  // Error scenarios
  http.get('/api/users/:id', ({ params }) => {
    if (params.id === 'error') {
      return HttpResponse.json(
        { message: 'User not found' },
        { status: 404 }
      );
    }
    return HttpResponse.json(createMockUser({ id: params.id }));
  }),
];

// src/test/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### 6. Parallel Test Execution

```javascript
// Advanced parallel testing strategies

// vitest.config.ts - Sharding for CI
export default defineConfig({
  test: {
    // Split tests across multiple CI jobs
    shard: process.env.CI 
      ? {
          current: parseInt(process.env.VITEST_SHARD_INDEX || '1'),
          total: parseInt(process.env.VITEST_SHARD_TOTAL || '1')
        }
      : undefined,
    
    // Isolation configuration
    isolate: true,
    pool: 'threads',
    poolOptions: {
      threads: {
        singleThread: false,
        minThreads: 1,
        maxThreads: process.env.CI ? 2 : 4
      }
    }
  }
});

// GitHub Actions matrix for parallel tests
// .github/workflows/test.yml
const parallelTestJob = `
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test
        env:
          VITEST_SHARD_INDEX: \${{ matrix.shard }}
          VITEST_SHARD_TOTAL: 4
`;
```

---

## Visual Regression Testing

### 1. Percy Setup & Configuration

```javascript
// Percy - Visual regression testing platform

// Install Percy
// npm install --save-dev @percy/cli @percy/playwright

// .percy.yml - Percy configuration
const percyConfig = `
version: 2
static:
  exclude:
    - "**/*.map"
    - "**/node_modules/**"
snapshot:
  widths:
    - 375   # Mobile
    - 768   # Tablet
    - 1280  # Desktop
    - 1920  # Large Desktop
  minHeight: 1024
  percyCSS: |
    /* Hide dynamic content */
    [data-testid="timestamp"] { visibility: hidden; }
    .animated-element { animation: none !important; }
    
discovery:
  allowedHostnames:
    - localhost
    - "*.example.com"
  networkIdleTimeout: 750
`;

// playwright-percy.config.ts
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
  testDir: './e2e/visual',
  use: {
    baseURL: 'http://localhost:5173',
    // Disable animations for consistent screenshots
    timezoneId: 'UTC',
    locale: 'en-US',
  },
  webServer: {
    command: 'npm run dev',
    port: 5173,
    reuseExistingServer: !process.env.CI,
  },
};

export default config;

// e2e/visual/homepage.visual.spec.ts
import { test } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test.describe('Visual Regression Tests', () => {
  test.beforeEach(async ({ page }) => {
    // Set consistent viewport
    await page.setViewportSize({ width: 1280, height: 720 });
    
    // Mock time-dependent data
    await page.addInitScript(() => {
      // Override Date to return consistent time
      const constantDate = new Date('2024-01-01T00:00:00Z');
      Date.now = () => constantDate.getTime();
    });
  });

  test('homepage renders correctly', async ({ page }) => {
    await page.goto('/');
    
    // Wait for all images to load
    await page.waitForLoadState('networkidle');
    
    // Hide dynamic elements
    await page.addStyleTag({
      content: `
        .timestamp, [data-testid="timestamp"] { visibility: hidden !important; }
        * { animation: none !important; transition: none !important; }
      `
    });

    // Take snapshot
    await percySnapshot(page, 'Homepage - Default State');
  });

  test('mobile navigation menu', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('/');
    
    // Open mobile menu
    await page.click('[aria-label="Open menu"]');
    await page.waitForSelector('[role="dialog"]', { state: 'visible' });

    await percySnapshot(page, 'Mobile Navigation - Opened');
  });

  test('product listing with various states', async ({ page }) => {
    await page.goto('/products');

    // Default state
    await percySnapshot(page, 'Products - Default');

    // Filtered state
    await page.click('text=Electronics');
    await page.waitForLoadState('networkidle');
    await percySnapshot(page, 'Products - Electronics Filter');

    // Empty state
    await page.fill('[placeholder="Search products"]', 'nonexistentproduct123');
    await page.waitForTimeout(500); // Debounce
    await percySnapshot(page, 'Products - Empty State');
  });

  test('dark mode comparison', async ({ page }) => {
    await page.goto('/');

    // Light mode
    await percySnapshot(page, 'Homepage - Light Mode');

    // Dark mode
    await page.click('[aria-label="Toggle dark mode"]');
    await page.waitForTimeout(300); // Wait for theme transition
    await percySnapshot(page, 'Homepage - Dark Mode');
  });

  test('form validation states', async ({ page }) => {
    await page.goto('/contact');

    // Pristine state
    await percySnapshot(page, 'Contact Form - Pristine');

    // Try to submit empty form
    await page.click('button[type="submit"]');
    await page.waitForSelector('[role="alert"]');
    await percySnapshot(page, 'Contact Form - Validation Errors');

    // Fill valid data
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="message"]', 'Test message');
    await percySnapshot(page, 'Contact Form - Valid State');
  });

  test('responsive grid layouts', async ({ page }) => {
    const viewports = [
      { width: 375, name: 'Mobile' },
      { width: 768, name: 'Tablet' },
      { width: 1280, name: 'Desktop' },
      { width: 1920, name: 'Large Desktop' }
    ];

    for (const viewport of viewports) {
      await page.setViewportSize({ width: viewport.width, height: 1024 });
      await page.goto('/dashboard');
      await page.waitForLoadState('networkidle');
      
      await percySnapshot(page, `Dashboard - ${viewport.name}`);
    }
  });
});

// package.json scripts
{
  "scripts": {
    "test:visual": "percy exec -- playwright test --config playwright-percy.config.ts",
    "test:visual:update": "percy exec -- playwright test --update-snapshots"
  }
}

// CI/CD Integration
// .github/workflows/visual-regression.yml
const visualRegressionWorkflow = `
name: Visual Regression Tests

on:
  pull_request:
    branches: [main, develop]

jobs:
  visual-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps chromium
      
      - name: Run visual regression tests
        run: npm run test:visual
        env:
          PERCY_TOKEN: \${{ secrets.PERCY_TOKEN }}
      
      - name: Upload Percy build link
        if: always()
        uses: actions/github-script@v6
        with:
          script: |
            const percyBuildUrl = process.env.PERCY_BUILD_URL;
            if (percyBuildUrl) {
              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: \`🎨 Percy visual tests completed! View results: \${percyBuildUrl}\`
              });
            }
`;
```

### 2. Playwright Built-in Screenshot Testing

```javascript
// Alternative: Playwright's native screenshot comparison

// playwright.config.ts
export default defineConfig({
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 100,
      threshold: 0.2,
      animations: 'disabled',
      caret: 'hide',
    },
  },
});

// e2e/visual/screenshots.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Built-in Screenshot Tests', () => {
  test('homepage matches baseline', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    
    // Full page screenshot
    await expect(page).toHaveScreenshot('homepage.png', {
      fullPage: true,
    });
  });

  test('component screenshot', async ({ page }) => {
    await page.goto('/components');
    
    // Screenshot specific element
    const button = page.locator('[data-testid="primary-button"]');
    await expect(button).toHaveScreenshot('primary-button.png');
  });

  test('screenshot with mask', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Mask dynamic elements
    await expect(page).toHaveScreenshot('dashboard.png', {
      mask: [
        page.locator('[data-testid="timestamp"]'),
        page.locator('.user-avatar'),
      ],
    });
  });

  test('update screenshots', async ({ page }) => {
    // Run with --update-snapshots flag to update baselines
    await page.goto('/');
    await expect(page).toHaveScreenshot();
  });
});

// Updating snapshots
// npm run test:e2e -- --update-snapshots
```

### 3. Chromatic (Storybook Integration)

```javascript
// For component-level visual testing

// Install Chromatic
// npm install --save-dev chromatic

// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
};

export default config;

// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    // Chromatic specific parameters
    chromatic: {
      viewports: [320, 768, 1200],
      delay: 300,
      pauseAnimationAtEnd: true,
    },
  },
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
    },
  },
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

export const Loading: Story = {
  args: {
    isLoading: true,
    children: 'Loading...',
  },
  parameters: {
    chromatic: { disableSnapshot: true }, // Skip this story
  },
};

export const AllSizes: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', flexDirection: 'column', alignItems: 'flex-start' }}>
      <Button size="small">Small</Button>
      <Button size="medium">Medium</Button>
      <Button size="large">Large</Button>
    </div>
  ),
};

// chromatic.config.json
{
  "projectId": "your-project-id",
  "buildScriptName": "build-storybook",
  "exitZeroOnChanges": true,
  "exitOnceUploaded": true,
  "onlyChanged": true,
  "externals": ["public/**"],
  "skip": "dependabot/**"
}

// package.json
{
  "scripts": {
    "chromatic": "chromatic --project-token=$CHROMATIC_PROJECT_TOKEN",
    "chromatic:ci": "chromatic --exit-zero-on-changes --exit-once-uploaded"
  }
}

// .github/workflows/chromatic.yml
const chromaticWorkflow = `
name: Chromatic

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  visual-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      
      - run: npm ci
      
      - name: Run Chromatic
        uses: chromaui/action@v1
        with:
          projectToken: \${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          exitZeroOnChanges: true
          onlyChanged: true
`;
```

### 4. Visual Testing Best Practices

```javascript
// Best practices for consistent visual tests

// 1. Stabilize Dynamic Content
class VisualTestHelpers {
  static async stabilizePage(page) {
    // Wait for fonts to load
    await page.evaluate(() => document.fonts.ready);
    
    // Wait for images
    await page.evaluate(() => {
      return Promise.all(
        Array.from(document.images)
          .filter(img => !img.complete)
          .map(img => new Promise(resolve => {
            img.onload = img.onerror = resolve;
          }))
      );
    });
    
    // Disable animations
    await page.addStyleTag({
      content: `
        *, *::before, *::after {
          animation-duration: 0s !important;
          animation-delay: 0s !important;
          transition-duration: 0s !important;
          transition-delay: 0s !important;
        }
      `
    });
    
    // Hide cursor/caret
    await page.addStyleTag({
      content: `
        * {
          caret-color: transparent !important;
          cursor: none !important;
        }
      `
    });
  }

  // 2. Mock Time-Dependent Data
  static async freezeTime(page, date = '2024-01-01T00:00:00Z') {
    await page.addInitScript(`{
      const targetDate = new Date('${date}');
      Date = class extends Date {
        constructor(...args) {
          if (args.length === 0) {
            super(targetDate);
          } else {
            super(...args);
          }
        }
        static now() {
          return targetDate.getTime();
        }
      };
    }`);
  }

  // 3. Wait for Network Idle
  static async waitForStability(page) {
    await page.waitForLoadState('networkidle');
    await page.waitForLoadState('domcontentloaded');
    await page.waitForTimeout(300); // Extra buffer
  }

  // 4. Handle Flaky Elements
  static async hideVolatileElements(page) {
    await page.addStyleTag({
      content: `
        [data-volatile],
        .timestamp,
        .live-update,
        .random-content {
          visibility: hidden !important;
        }
      `
    });
  }
}

// Usage in tests
test('stable visual test', async ({ page }) => {
  await page.goto('/');
  
  await VisualTestHelpers.freezeTime(page);
  await VisualTestHelpers.stabilizePage(page);
  await VisualTestHelpers.hideVolatileElements(page);
  await VisualTestHelpers.waitForStability(page);
  
  await percySnapshot(page, 'Stable Homepage');
});
```

---

## Bundle Monitoring

### 1. Bundle Size Analysis

```javascript
// vite.config.ts - Bundle analysis setup
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';
import { compression } from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    
    // Bundle visualization
    visualizer({
      filename: './dist/stats.html',
      open: false,
      gzipSize: true,
      brotliSize: true,
      template: 'treemap', // or 'sunburst', 'network'
    }),
    
    // Compression for production
    compression({
      algorithm: 'gzip',
      ext: '.gz',
    }),
    compression({
      algorithm: 'brotliCompress',
      ext: '.br',
    }),
  ],
  
  build: {
    rollupOptions: {
      output: {
        // Manual chunking strategy
        manualChunks: {
          // Vendor chunks
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          'utils-vendor': ['lodash-es', 'date-fns'],
          
          // Feature-based chunks
          'admin': [
            './src/features/admin/AdminDashboard',
            './src/features/admin/UserManagement',
          ],
          'analytics': [
            './src/features/analytics/Charts',
            './src/features/analytics/Reports',
          ],
        },
        
        // Asset naming
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: ({ name }) => {
          if (/\.(gif|jpe?g|png|svg)$/.test(name ?? '')) {
            return 'assets/images/[name]-[hash][extname]';
          }
          if (/\.css$/.test(name ?? '')) {
            return 'assets/css/[name]-[hash][extname]';
          }
          return 'assets/[name]-[hash][extname]';
        },
      },
    },
    
    // Chunk size warnings
    chunkSizeWarningLimit: 500,
    
    // Source maps for production debugging
    sourcemap: true,
    
    // Minification
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
});

// package.json scripts
{
  "scripts": {
    "build": "vite build",
    "build:analyze": "vite build --mode analyze && open dist/stats.html",
    "analyze": "npx vite-bundle-visualizer",
    "size-limit": "size-limit",
    "size-limit:why": "size-limit --why"
  }
}
```

### 2. Size-limit Configuration

```javascript
// .size-limit.json - Budget enforcement
[
  {
    "name": "Main Bundle",
    "path": "dist/assets/index-*.js",
    "limit": "150 KB",
    "gzip": true,
    "running": false
  },
  {
    "name": "Vendor Bundle",
    "path": "dist/assets/vendor-*.js",
    "limit": "200 KB",
    "gzip": true
  },
  {
    "name": "CSS",
    "path": "dist/assets/*.css",
    "limit": "50 KB",
    "gzip": true
  },
  {
    "name": "Total Initial Load",
    "path": "dist/assets/*.{js,css}",
    "limit": "400 KB",
    "gzip": true,
    "running": false
  }
]

// GitHub Action for size monitoring
// .github/workflows/size-limit.yml
const sizeLimitWorkflow = `
name: Size Limit

on:
  pull_request:
    branches: [main]

jobs:
  size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Check bundle size
        uses: andresz1/size-limit-action@v1
        with:
          github_token: \${{ secrets.GITHUB_TOKEN }}
          skip_step: install
`;
```

### 3. Bundle Performance Monitoring

```javascript
// scripts/analyze-bundle.js - Custom bundle analysis
import fs from 'fs';
import path from 'path';
import { gzipSizeSync } from 'gzip-size';
import { brotliCompressSync } from 'zlib';

interface BundleStats {
  file: string;
  size: number;
  gzipSize: number;
  brotliSize: number;
}

function analyzeBundles(distDir: string): BundleStats[] {
  const files = fs.readdirSync(distDir, { recursive: true });
  const stats: BundleStats[] = [];

  files.forEach((file) => {
    if (typeof file !== 'string') return;
    
    const filePath = path.join(distDir, file);
    const stat = fs.statSync(filePath);

    if (stat.isFile() && /\.(js|css)$/.test(file)) {
      const content = fs.readFileSync(filePath);
      const size = stat.size;
      const gzipSize = gzipSizeSync(content);
      const brotliSize = brotliCompressSync(content).length;

      stats.push({
        file,
        size,
        gzipSize,
        brotliSize,
      });
    }
  });

  return stats;
}

function formatBytes(bytes: number): string {
  if (bytes === 0) return '0 B';
  const k = 1024;
  const sizes = ['B', 'KB', 'MB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return `${parseFloat((bytes / Math.pow(k, i)).toFixed(2))} ${sizes[i]}`;
}

function generateReport(stats: BundleStats[]): void {
  console.log('\n📦 Bundle Analysis Report\n');
  console.log('━'.repeat(80));
  console.log(
    `${'File'.padEnd(40)} ${'Size'.padEnd(12)} ${'Gzip'.padEnd(12)} ${'Brotli'.padEnd(12)}`
  );
  console.log('━'.repeat(80));

  stats.forEach((stat) => {
    console.log(
      `${stat.file.padEnd(40)} ${formatBytes(stat.size).padEnd(12)} ${formatBytes(
        stat.gzipSize
      ).padEnd(12)} ${formatBytes(stat.brotliSize).padEnd(12)}`
    );
  });

  console.log('━'.repeat(80));

  const totalSize = stats.reduce((acc, s) => acc + s.size, 0);
  const totalGzip = stats.reduce((acc, s) => acc + s.gzipSize, 0);
  const totalBrotli = stats.reduce((acc, s) => acc + s.brotliSize, 0);

  console.log(
    `${'TOTAL'.padEnd(40)} ${formatBytes(totalSize).padEnd(12)} ${formatBytes(
      totalGzip
    ).padEnd(12)} ${formatBytes(totalBrotli).padEnd(12)}`
  );
  console.log('\n');

  // Save to JSON for CI
  fs.writeFileSync(
    'bundle-stats.json',
    JSON.stringify({ stats, totals: { size: totalSize, gzip: totalGzip, brotli: totalBrotli } }, null, 2)
  );
}

const stats = analyzeBundles('./dist');
generateReport(stats);

// Compare with previous build
function compareWithBaseline(current: BundleStats[], baseline: BundleStats[]): void {
  console.log('\n📊 Bundle Size Comparison\n');
  console.log('━'.repeat(80));

  current.forEach((currentStat) => {
    const baselineStat = baseline.find((b) => b.file === currentStat.file);
    
    if (baselineStat) {
      const diff = currentStat.gzipSize - baselineStat.gzipSize;
      const percentChange = ((diff / baselineStat.gzipSize) * 100).toFixed(2);
      const icon = diff > 0 ? '📈' : diff < 0 ? '📉' : '➖';
      const sign = diff > 0 ? '+' : '';

      console.log(`${icon} ${currentStat.file}`);
      console.log(`   ${sign}${formatBytes(diff)} (${sign}${percentChange}%)`);
    }
  });
}
```

### 4. Webpack Bundle Analyzer (Alternative)

```javascript
// For Webpack projects

// webpack.config.js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: process.env.ANALYZE ? 'server' : 'disabled',
      generateStatsFile: true,
      statsFilename: 'bundle-stats.json',
      reportFilename: 'bundle-report.html',
      openAnalyzer: false,
      statsOptions: { source: false },
    }),
  ],
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name(module) {
            const packageName = module.context.match(
              /[\\/]node_modules[\\/](.*?)([\\/]|$)/
            )[1];
            return `vendor.${packageName.replace('@', '')}`;
          },
          priority: 10,
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },
    
    runtimeChunk: 'single',
    
    moduleIds: 'deterministic',
  },
  
  performance: {
    maxEntrypointSize: 400000,
    maxAssetSize: 300000,
    hints: 'warning',
  },
};
```

### 5. Runtime Bundle Monitoring

```javascript
// Monitor bundle performance in production

// src/utils/performance-monitor.ts
class BundlePerformanceMonitor {
  private static instance: BundlePerformanceMonitor;
  private metrics: Map<string, number> = new Map();

  static getInstance(): BundlePerformanceMonitor {
    if (!BundlePerformanceMonitor.instance) {
      BundlePerformanceMonitor.instance = new BundlePerformanceMonitor();
    }
    return BundlePerformanceMonitor.instance;
  }

  // Track chunk loading time
  trackChunkLoad(chunkName: string): void {
    if (!('performance' in window)) return;

    const entries = performance.getEntriesByType('resource') as PerformanceResourceTiming[];
    const chunkEntry = entries.find((entry) => entry.name.includes(chunkName));

    if (chunkEntry) {
      this.metrics.set(chunkName, chunkEntry.duration);
      this.reportMetric(chunkName, chunkEntry.duration);
    }
  }

  // Monitor total bundle size loaded
  getTotalBundleSize(): number {
    if (!('performance' in window)) return 0;

    const entries = performance.getEntriesByType('resource') as PerformanceResourceTiming[];
    const jsEntries = entries.filter((entry) => entry.name.endsWith('.js'));

    return jsEntries.reduce((total, entry) => total + (entry.transferSize || 0), 0);
  }

  // Report to analytics
  private reportMetric(name: string, value: number): void {
    // Send to analytics service
    if (typeof gtag !== 'undefined') {
      gtag('event', 'chunk_load', {
        chunk_name: name,
        load_time: Math.round(value),
        event_category: 'Performance',
      });
    }
  }

  // Generate performance report
  generateReport(): void {
    const report = {
      totalSize: this.getTotalBundleSize(),
      chunkMetrics: Array.from(this.metrics.entries()),
      timestamp: new Date().toISOString(),
    };

    console.table(report.chunkMetrics);
    return report;
  }
}

// Usage in app
const monitor = BundlePerformanceMonitor.getInstance();

// Track lazy-loaded chunks
const AdminDashboard = lazy(async () => {
  const start = performance.now();
  const module = await import('./features/admin/AdminDashboard');
  monitor.trackChunkLoad('admin-dashboard');
  return module;
});
```

### 6. Bundle Budget CI Check

```javascript
// scripts/check-bundle-budget.js
import fs from 'fs';

const BUDGETS = {
  'main': 150 * 1024,      // 150KB
  'vendor': 200 * 1024,    // 200KB
  'total': 400 * 1024,     // 400KB
};

function checkBudgets() {
  const stats = JSON.parse(fs.readFileSync('bundle-stats.json', 'utf-8'));
  const failures = [];

  // Check individual bundles
  stats.stats.forEach(stat => {
    const name = stat.file.includes('vendor') ? 'vendor' : 'main';
    const budget = BUDGETS[name];
    
    if (stat.gzipSize > budget) {
      failures.push({
        file: stat.file,
        size: stat.gzipSize,
        budget,
        overage: stat.gzipSize - budget,
      });
    }
  });

  // Check total
  if (stats.totals.gzip > BUDGETS.total) {
    failures.push({
      file: 'Total Bundle',
      size: stats.totals.gzip,
      budget: BUDGETS.total,
      overage: stats.totals.gzip - BUDGETS.total,
    });
  }

  // Report results
  if (failures.length > 0) {
    console.error('\n❌ Bundle budget exceeded!\n');
    failures.forEach(failure => {
      console.error(`${failure.file}:`);
      console.error(`  Current: ${(failure.size / 1024).toFixed(2)}KB`);
      console.error(`  Budget:  ${(failure.budget / 1024).toFixed(2)}KB`);
      console.error(`  Over by: ${(failure.overage / 1024).toFixed(2)}KB\n`);
    });
    process.exit(1);
  } else {
    console.log('✅ All bundles within budget!');
  }
}

checkBudgets();
```

---

## Preview Builds

### 1. Vercel Preview Deployments

```yaml
# vercel.json - Vercel configuration
{
  "version": 2,
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "vite",
  "outputDirectory": "dist",
  
  "env": {
    "NODE_VERSION": "20"
  },
  
  "build": {
    "env": {
      "VITE_API_URL": "@api-url",
      "VITE_ANALYTICS_ID": "@analytics-id"
    }
  },
  
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ],
  
  "redirects": [
    {
      "source": "/old-route",
      "destination": "/new-route",
      "permanent": true
    }
  ],
  
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api.example.com/:path*"
    }
  ],
  
  "regions": ["iad1"],
  
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 10
    }
  }
}
```

```javascript
// .github/workflows/preview.yml - Automated preview deployments
const previewWorkflow = `
name: Preview Deployment

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
        env:
          VITE_APP_ENV: preview
          VITE_API_URL: https://preview-api.example.com
      
      - name: Deploy to Vercel
        id: deploy
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: \${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: \${{ secrets.ORG_ID }}
          vercel-project-id: \${{ secrets.PROJECT_ID }}
          scope: \${{ secrets.VERCEL_SCOPE }}
          alias-domains: |
            pr-\${{ github.event.pull_request.number }}.preview.example.com
      
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            \${{ steps.deploy.outputs.preview-url }}
          uploadArtifacts: true
          temporaryPublicStorage: true
      
      - name: Comment PR with preview URL
        uses: actions/github-script@v6
        with:
          script: |
            const previewUrl = '\${{ steps.deploy.outputs.preview-url }}';
            const lighthouseUrl = '\${{ steps.lighthouse.outputs.manifest-url }}';
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: \`
## 🚀 Preview Deployment Ready!

### Deployment URLs
- **Preview**: [\${previewUrl}](\${previewUrl})
- **PR Alias**: [pr-\${{ github.event.pull_request.number }}.preview.example.com](https://pr-\${{ github.event.pull_request.number }}.preview.example.com)

### Performance
- **Lighthouse Report**: [View Results](\${lighthouseUrl})

### Quick Actions
- 🔍 [View Deployment Logs](https://vercel.com/\${{ secrets.VERCEL_SCOPE }}/deployments)
- 📊 [Bundle Analysis](/actions/runs/\${{ github.run_id }})
- 🎨 [Visual Regression Tests](/actions/runs/\${{ github.run_id }})

---
*Deployed from commit \${{ github.sha }}*
              \`
            });
      
      - name: Run E2E tests on preview
        run: npx playwright test
        env:
          BASE_URL: \${{ steps.deploy.outputs.preview-url }}
      
      - name: Update deployment status
        if: always()
        uses: actions/github-script@v6
        with:
          script: |
            const status = '\${{ job.status }}' === 'success' ? 'success' : 'failure';
            github.rest.repos.createDeploymentStatus({
              owner: context.repo.owner,
              repo: context.repo.repo,
              deployment_id: '\${{ steps.deploy.outputs.deployment-id }}',
              state: status,
              environment_url: '\${{ steps.deploy.outputs.preview-url }}',
              description: 'Preview deployment \${{ job.status }}'
            });
`;

// Enhanced preview with feature flags
// src/config/preview.ts
export const getPreviewConfig = () => {
  const isPreviewing mode = window.location.hostname.includes('preview');
  const prNumber = window.location.hostname.match(/pr-(\d+)/)?.[1];
  
  return {
    isPreview: isPreviewMode,
    prNumber,
    features: {
      // Enable all features in preview
      newDashboard: true,
      betaFeatures: true,
      debugMode: true,
    },
    api: {
      baseUrl: isPreviewMode 
        ? 'https://preview-api.example.com'
        : 'https://api.example.com'
    },
    analytics: {
      enabled: !isPreviewMode, // Disable analytics in preview
    }
  };
};
```

### 2. Netlify Preview Deployments

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"
  functions = "netlify/functions"

[build.environment]
  NODE_VERSION = "20"
  NPM_FLAGS = "--legacy-peer-deps"

# Preview context
[context.deploy-preview]
  command = "npm run build:preview"
  
[context.deploy-preview.environment]
  VITE_APP_ENV = "preview"
  VITE_API_URL = "https://preview-api.example.com"

# Branch deploys
[context.develop]
  command = "npm run build:staging"
  
[context.develop.environment]
  VITE_APP_ENV = "staging"

# Headers
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

# Redirects
[[redirects]]
  from = "/old-path"
  to = "/new-path"
  status = 301

[[redirects]]
  from = "/api/*"
  to = "https://api.example.com/:splat"
  status = 200

# SPA fallback
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# Plugin configuration
[[plugins]]
  package = "@netlify/plugin-lighthouse"
  
  [plugins.inputs.thresholds]
    performance = 0.9
    accessibility = 0.9
    best-practices = 0.9
    seo = 0.9

[[plugins]]
  package = "netlify-plugin-checklinks"
  
  [plugins.inputs]
    skipPatterns = ["https://external-site.com"]
```

```javascript
// scripts/netlify-preview-comment.js
// Netlify build plugin for PR comments

module.exports = {
  async onSuccess({ utils, constants }) {
    const { DEPLOY_PRIME_URL, PULL_REQUEST } = process.env;
    
    if (!PULL_REQUEST) return;

    const comment = `
## 🚀 Preview Deployment Ready!

**Preview URL**: ${DEPLOY_PRIME_URL}

### Lighthouse Scores
- Performance: ${utils.cache.get('lighthouse:performance')}
- Accessibility: ${utils.cache.get('lighthouse:accessibility')}
- Best Practices: ${utils.cache.get('lighthouse:best-practices')}
- SEO: ${utils.cache.get('lighthouse:seo')}

*Deployed from commit ${constants.COMMIT_REF}*
    `;

    // Post comment to GitHub
    await utils.git.comment(PULL_REQUEST, comment);
  }
};
```

### 3. CloudFlare Pages Preview

```javascript
// wrangler.toml - CloudFlare Pages configuration
const wranglerConfig = `
name = "my-app"
compatibility_date = "2024-01-01"
pages_build_output_dir = "dist"

[build]
command = "npm run build"

[build.environment]
NODE_VERSION = "20"

[[env.preview.build.environment]]
VITE_APP_ENV = "preview"
VITE_API_URL = "https://preview-api.example.com"

[[env.production.build.environment]]
VITE_APP_ENV = "production"
VITE_API_URL = "https://api.example.com"

# Deploy hooks
[deploy]
  hook = "https://api.cloudflare.com/client/v4/pages/webhooks/deploy_hooks/YOUR_HOOK_ID"
`;

// GitHub Action for CloudFlare
const cloudflareWorkflow = `
name: CloudFlare Pages Deploy

on:
  push:
    branches: [main]
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
      pull-requests: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build
        run: npm ci && npm run build
      
      - name: Publish to CloudFlare Pages
        id: deploy
        uses: cloudflare/pages-action@v1
        with:
          apiToken: \${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: \${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: my-app
          directory: dist
          gitHubToken: \${{ secrets.GITHUB_TOKEN }}
      
      - name: Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚀 Preview: \${{ steps.deploy.outputs.url }}'
            });
`;
```

### 4. Self-Hosted Preview Infrastructure

```javascript
// docker-compose.preview.yml - Self-hosted preview environment
const dockerCompose = `
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
      - previews:/usr/share/nginx/html
    depends_on:
      - preview-manager
  
  preview-manager:
    build: ./preview-manager
    environment:
      - GITHUB_TOKEN=\${GITHUB_TOKEN}
      - PREVIEW_DOMAIN=preview.example.com
    volumes:
      - previews:/previews
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "3000:3000"

volumes:
  previews:
`;

// preview-manager/server.js - Custom preview deployment service
const express = require('express');
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

const app = express();
app.use(express.json());

const PREVIEWS_DIR = '/previews';

// Webhook endpoint for GitHub
app.post('/webhook/github', async (req, res) => {
  const { action, pull_request } = req.body;
  
  if (!['opened', 'synchronize', 'reopened'].includes(action)) {
    return res.status(200).send('Ignored');
  }

  const prNumber = pull_request.number;
  const branch = pull_request.head.ref;
  const repoUrl = pull_request.head.repo.clone_url;

  try {
    await deployPreview(prNumber, repoUrl, branch);
    await commentOnPR(prNumber, `Preview deployed: https://pr-${prNumber}.preview.example.com`);
    res.status(200).send('Deployment started');
  } catch (error) {
    console.error('Deployment failed:', error);
    await commentOnPR(prNumber, 'Preview deployment failed. Check logs for details.');
    res.status(500).send('Deployment failed');
  }
});

async function deployPreview(prNumber, repoUrl, branch) {
  const previewDir = path.join(PREVIEWS_DIR, `pr-${prNumber}`);
  
  // Create directory
  if (!fs.existsSync(previewDir)) {
    fs.mkdirSync(previewDir, { recursive: true });
  }

  // Clone and build
  await execPromise(`
    cd ${previewDir} &&
    git clone --branch ${branch} ${repoUrl} . &&
    npm ci &&
    npm run build &&
    cp -r dist/* .
  `);

  // Update nginx config
  await updateNginxConfig(prNumber);
}

async function updateNginxConfig(prNumber) {
  const config = `
server {
    listen 80;
    server_name pr-${prNumber}.preview.example.com;
    
    root /previews/pr-${prNumber};
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    location /assets {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
  `;

  fs.writeFileSync(`/etc/nginx/sites-enabled/pr-${prNumber}`, config);
  await execPromise('nginx -s reload');
}

function execPromise(command) {
  return new Promise((resolve, reject) => {
    exec(command, (error, stdout, stderr) => {
      if (error) reject(error);
      else resolve(stdout);
    });
  });
}

app.listen(3000, () => {
  console.log('Preview manager running on port 3000');
});
```

### 5. Preview Environment Management

```javascript
// src/utils/preview-banner.tsx - Preview environment indicator
import { useState, useEffect } from 'react';
import { AlertCircle, X } from 'lucide-react';

export function PreviewBanner() {
  const [isPreview, setIsPreview] = useState(false);
  const [prNumber, setPrNumber] = useState<string | null>(null);
  const [dismissed, setDismissed] = useState(false);

  useEffect(() => {
    const hostname = window.location.hostname;
    const isPreviewEnv = hostname.includes('preview') || hostname.includes('pr-');
    const match = hostname.match(/pr-(\d+)/);
    
    setIsPreview(isPreviewEnv);
    setPrNumber(match ? match[1] : null);

    // Check if previously dismissed
    const wasDismissed = sessionStorage.getItem('preview-banner-dismissed');
    setDismissed(wasDismissed === 'true');
  }, []);

  const handleDismiss = () => {
    setDismissed(true);
    sessionStorage.setItem('preview-banner-dismissed', 'true');
  };

  if (!isPreview || dismissed) return null;

  return (
    <div className="preview-banner">
      <div className="preview-banner-content">
        <AlertCircle className="icon" />
        <div className="preview-banner-text">
          <strong>Preview Environment</strong>
          {prNumber && <span> - PR #{prNumber}</span>}
          <p>This is a preview deployment. Data may not be persisted.</p>
        </div>
        <button onClick={handleDismiss} className="dismiss-btn">
          <X size={18} />
        </button>
      </div>
    </div>
  );
}

// Conditional feature flags for preview
export function usePreviewFeatures() {
  const isPreview = window.location.hostname.includes('preview');
  
  return {
    debugMode: isPreview,
    showDevTools: isPreview,
    enableExperimentalFeatures: isPreview,
    bypassAuth: isPreview && import.meta.env.VITE_PREVIEW_BYPASS_AUTH === 'true',
  };
}
```

---

## Best Practices & Day-to-Day Coding

### 1. Daily CI/CD Workflow

```javascript
// Daily development best practices

// 1. Pre-commit hooks with Husky
// .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."

# Run linting
npm run lint-staged

# Run type checking
npm run type-check

# Run unit tests for changed files
npm run test:changed

echo "✅ Pre-commit checks passed!"

// 2. Lint-staged configuration
// .lintstagedrc.json
{
  "*.{ts,tsx}": [
    "eslint --fix",
    "prettier --write",
    () => "tsc --noEmit"
  ],
  "*.{json,md,yml,yaml}": [
    "prettier --write"
  ],
  "*.{css,scss}": [
    "stylelint --fix",
    "prettier --write"
  ]
}

// 3. Commit message validation
// .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no -- commitlint --edit $1

// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // New feature
        'fix',      // Bug fix
        'docs',     // Documentation
        'style',    // Formatting
        'refactor', // Code restructuring
        'test',     // Adding tests
        'chore',    // Maintenance
        'perf',     // Performance
        'ci',       // CI/CD changes
      ],
    ],
    'subject-case': [2, 'never', ['start-case', 'pascal-case', 'upper-case']],
    'subject-max-length': [2, 'always', 72],
  },
};

// Example good commits:
// feat: add user profile avatar upload
// fix: resolve race condition in checkout flow
// test: add integration tests for payment gateway
// chore: update dependencies to latest versions
```

### 2. Code Quality Gates

```javascript
// .github/workflows/quality-gates.yml
const qualityGatesWorkflow = `
name: Quality Gates

on:
  pull_request:
    branches: [main, develop]

jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for SonarCloud
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      
      - name: Install dependencies
        run: npm ci
      
      # Code Coverage Gate
      - name: Run tests with coverage
        run: npm run test:coverage
      
      - name: Check coverage thresholds
        run: |
          COVERAGE=\$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( \$(echo "\$COVERAGE < 80" | bc -l) )); then
            echo "❌ Coverage below 80%: \$COVERAGE%"
            exit 1
          fi
          echo "✅ Coverage: \$COVERAGE%"
      
      # Code Quality Gate (SonarCloud)
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: \${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: \${{ secrets.SONAR_TOKEN }}
      
      # Bundle Size Gate
      - name: Check bundle size
        run: npm run size-limit
      
      # Security Gate
      - name: Run security audit
        run: npm audit --audit-level=moderate
      
      # Accessibility Gate
      - name: Run accessibility tests
        run: npm run test:a11y
      
      # Type Safety Gate
      - name: TypeScript strict check
        run: npm run type-check
`;

// sonar-project.properties
const sonarConfig = `
sonar.projectKey=my-app
sonar.organization=my-org

sonar.sources=src
sonar.tests=src
sonar.test.inclusions=**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx
sonar.exclusions=**/*.test.ts,**/*.test.tsx,**/*.spec.ts,**/*.spec.tsx,**/node_modules/**

sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.testExecutionReportPaths=test-results/sonar-report.xml

# Quality Gates
sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300

# Coverage
sonar.coverage.exclusions=**/*.test.ts,**/*.spec.ts,**/*.config.ts

# Code Smells
sonar.issue.ignore.multicriteria=e1
sonar.issue.ignore.multicriteria.e1.ruleKey=typescript:S1128
sonar.issue.ignore.multicriteria.e1.resourceKey=**/*.tsx
`;
```

### 3. Performance Best Practices

```typescript
// Daily performance coding practices

// 1. Lazy loading & Code Splitting
import { lazy, Suspense } from 'react';

// ❌ Bad: Import everything upfront
// import AdminDashboard from './features/admin/AdminDashboard';
// import UserManagement from './features/admin/UserManagement';
// import Reports from './features/reports/Reports';

// ✅ Good: Lazy load route components
const AdminDashboard = lazy(() => import('./features/admin/AdminDashboard'));
const UserManagement = lazy(() => import('./features/admin/UserManagement'));
const Reports = lazy(() => import('./features/reports/Reports'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/admin" element={<AdminDashboard />} />
        <Route path="/users" element={<UserManagement />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}

// 2. Dynamic imports for heavy libraries
async function exportToExcel(data: any[]) {
  // ❌ Bad: Import at top level
  // import * as XLSX from 'xlsx';
  
  // ✅ Good: Import only when needed
  const XLSX = await import('xlsx');
  const worksheet = XLSX.utils.json_to_sheet(data);
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Data');
  XLSX.writeFile(workbook, 'export.xlsx');
}

// 3. Optimize dependencies
// package.json
{
  "dependencies": {
    // ❌ Bad: Import entire lodash (70KB)
    // "lodash": "^4.17.21"
    
    // ✅ Good: Import specific lodash methods
    "lodash-es": "^4.17.21"  // Or use lodash.debounce, lodash.throttle individually
  }
}

// In code
// ❌ Bad
// import _ from 'lodash';
// const debounced = _.debounce(fn, 300);

// ✅ Good
import debounce from 'lodash-es/debounce';
const debounced = debounce(fn, 300);

// 4. Image optimization
// ❌ Bad: Large unoptimized images
// <img src="/images/hero.jpg" alt="Hero" />

// ✅ Good: Responsive images with modern formats
<picture>
  <source srcSet="/images/hero.webp" type="image/webp" />
  <source srcSet="/images/hero.jpg" type="image/jpeg" />
  <img 
    src="/images/hero.jpg" 
    alt="Hero"
    loading="lazy"
    width="1200"
    height="600"
  />
</picture>

// 5. Memoization
import { useMemo, useCallback } from 'react';

function ExpensiveComponent({ data, filter }) {
  // ❌ Bad: Recalculates on every render
  // const filteredData = data.filter(item => item.category === filter);
  
  // ✅ Good: Only recalculates when dependencies change
  const filteredData = useMemo(() => {
    return data.filter(item => item.category === filter);
  }, [data, filter]);

  // ✅ Good: Stable function reference
  const handleClick = useCallback((id: string) => {
    console.log('Clicked', id);
  }, []);

  return <List data={filteredData} onItemClick={handleClick} />;
}

// 6. Virtual scrolling for large lists
import { useVirtualizer } from '@tanstack/react-virtual';

function LargeList({ items }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 4. Testing Best Practices

```typescript
// Daily testing practices

// 1. Test naming conventions
describe('UserProfile Component', () => {
  // ❌ Bad: Vague test name
  // it('works', () => {})
  
  // ✅ Good: Descriptive test name
  it('should display user name and email when data is loaded', () => {
    render(<UserProfile userId="123" />);
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  // ✅ Good: Behavior-driven naming
  it('should show loading spinner while fetching user data', () => {
    render(<UserProfile userId="123" />);
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });

  // ✅ Good: Error case
  it('should display error message when user fetch fails', async () => {
    server.use(
      http.get('/api/users/:id', () => {
        return HttpResponse.error();
      })
    );

    render(<UserProfile userId="123" />);
    
    await waitFor(() => {
      expect(screen.getByText(/failed to load user/i)).toBeInTheDocument();
    });
  });
});

// 2. Arrange-Act-Assert pattern
it('should update cart total when item quantity changes', async () => {
  // Arrange
  const user = userEvent.setup();
  const mockUpdateCart = vi.fn();
  render(<CartItem item={mockItem} onUpdate={mockUpdateCart} />);
  
  // Act
  const quantityInput = screen.getByLabelText('Quantity');
  await user.clear(quantityInput);
  await user.type(quantityInput, '3');
  
  // Assert
  expect(mockUpdateCart).toHaveBeenCalledWith({
    ...mockItem,
    quantity: 3,
  });
});

// 3. Test user interactions, not implementation
// ❌ Bad: Testing implementation details
it('should call setState when button clicked', () => {
  const setState = vi.fn();
  render(<Component setState={setState} />);
  fireEvent.click(screen.getByRole('button'));
  expect(setState).toHaveBeenCalled();
});

// ✅ Good: Testing user-facing behavior
it('should show success message after form submission', async () => {
  const user = userEvent.setup();
  render(<ContactForm />);
  
  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.type(screen.getByLabelText(/message/i), 'Hello world');
  await user.click(screen.getByRole('button', { name: /submit/i }));
  
  await waitFor(() => {
    expect(screen.getByText(/message sent successfully/i)).toBeInTheDocument();
  });
});

// 4. Accessibility testing
import { axe, toHaveNoViolations
} from 'jest-axe';
expect.extend(toHaveNoViolations);

it('should have no accessibility violations', async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// 5. Test coverage best practices
// Focus on critical paths, not 100% coverage
// ✅ Good: Test happy path and edge cases
describe('calculateDiscount', () => {
  it('should return 0 for orders under $50', () => {
    expect(calculateDiscount(40)).toBe(0);
  });

  it('should return 10% for orders $50-$100', () => {
    expect(calculateDiscount(75)).toBe(7.5);
  });

  it('should return 20% for orders over $100', () => {
    expect(calculateDiscount(150)).toBe(30);
  });

  it('should handle negative amounts gracefully', () => {
    expect(calculateDiscount(-10)).toBe(0);
  });

  it('should handle zero amount', () => {
    expect(calculateDiscount(0)).toBe(0);
  });
});
```

### 5. Monitoring & Observability

```typescript
// Production monitoring setup

// 1. Error tracking (Sentry)
// src/utils/error-tracking.ts
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

export function initErrorTracking() {
  if (import.meta.env.PROD) {
    Sentry.init({
      dsn: import.meta.env.VITE_SENTRY_DSN,
      integrations: [
        new BrowserTracing(),
        new Sentry.Replay({
          maskAllText: false,
          blockAllMedia: false,
        }),
      ],
      tracesSampleRate: 0.1,
      replaysSessionSampleRate: 0.1,
      replaysOnErrorSampleRate: 1.0,
      environment: import.meta.env.VITE_APP_ENV,
      beforeSend(event, hint) {
        // Filter out known issues
        if (event.exception) {
          const error = hint.originalException;
          if (error instanceof Error && error.message.includes('ResizeObserver')) {
            return null;
          }
        }
        return event;
      },
    });
  }
}

// 2. Performance monitoring
// src/utils/performance.ts
export class PerformanceMonitor {
  static measurePageLoad() {
    if ('performance' in window) {
      window.addEventListener('load', () => {
        const perfData = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
        
        const metrics = {
          dns: perfData.domainLookupEnd - perfData.domainLookupStart,
          tcp: perfData.connectEnd - perfData.connectStart,
          ttfb: perfData.responseStart - perfData.requestStart,
          download: perfData.responseEnd - perfData.responseStart,
          domInteractive: perfData.domInteractive,
          domComplete: perfData.domComplete,
          loadComplete: perfData.loadEventEnd,
        };

        // Send to analytics
        this.sendMetrics('page_load', metrics);
      });
    }
  }

  static measureWebVitals() {
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS((metric) => this.sendMetrics('CLS', metric));
      getFID((metric) => this.sendMetrics('FID', metric));
      getFCP((metric) => this.sendMetrics('FCP', metric));
      getLCP((metric) => this.sendMetrics('LCP', metric));
      getTTFB((metric) => this.sendMetrics('TTFB', metric));
    });
  }

  private static sendMetrics(name: string, value: any) {
    if (typeof gtag !== 'undefined') {
      gtag('event', name, {
        event_category: 'Web Vitals',
        value: Math.round(value.value),
        event_label: value.id,
        non_interaction: true,
      });
    }
  }
}

// 3. Custom metrics tracking
export function trackFeatureUsage(featureName: string) {
  if (typeof gtag !== 'undefined') {
    gtag('event', 'feature_used', {
      feature_name: featureName,
      timestamp: Date.now(),
    });
  }
}

// 4. API monitoring
// src/api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

apiClient.interceptors.request.use((config) => {
  config.metadata = { startTime: Date.now() };
  return config;
});

apiClient.interceptors.response.use(
  (response) => {
    const duration = Date.now() - response.config.metadata.startTime;
    
    // Track successful requests
    if (typeof gtag !== 'undefined') {
      gtag('event', 'api_call', {
        endpoint: response.config.url,
        method: response.config.method,
        status: response.status,
        duration,
      });
    }

    return response;
  },
  (error) => {
    const duration = Date.now() - error.config.metadata.startTime;
    
    // Track failed requests
    Sentry.captureException(error, {
      tags: {
        endpoint: error.config.url,
        method: error.config.method,
      },
      extra: {
        status: error.response?.status,
        duration,
      },
    });

    return Promise.reject(error);
  }
);
```

---

## Common Interview Questions

### Theory Questions

**Q1: What is the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?**

**Answer:**
- **Continuous Integration (CI)**: Automatically building and testing code changes as soon as they're pushed to version control. Ensures code integrates correctly with the main branch.
- **Continuous Delivery (CD)**: Extends CI by automatically preparing code for release. Deployments require manual approval.
- **Continuous Deployment**: Fully automated - every change that passes tests is automatically deployed to production without manual intervention.

**Q2: Explain the test pyramid and why it's important.**

**Answer:**
The test pyramid is a testing strategy with three layers:
- **Base (70%)**: Unit tests - Fast, isolated, test individual functions/components
- **Middle (20%)**: Integration tests - Test how different parts work together
- **Top (10%)**: E2E tests - Slow, test entire user workflows

This distribution provides fast feedback, good coverage, and manageable maintenance costs. Too many E2E tests make the suite slow and brittle; too few miss integration issues.

**Q3: What strategies would you use to reduce CI/CD pipeline execution time?**

**Answer:**
1. **Parallel execution**: Run tests across multiple machines
2. **Smart test selection**: Only run tests affected by changes
3. **Caching**: Cache dependencies, build artifacts
4. **Incremental builds**: Only rebuild changed modules
5. **Docker layer caching**: Optimize Dockerfile layer order
6. **Matrix builds**: Test different configurations simultaneously
7. **Fail fast**: Run quickest tests first, stop on first failure

**Q4: How do you handle secrets and sensitive data in CI/CD?**

**Answer:**
1. Use platform secret management (GitHub Secrets, GitLab Variables)
2. Encrypt secrets at rest
3. Never commit secrets to repository
4. Use environment variables
5. Rotate secrets regularly
6. Implement least-privilege access
7. Use secret scanning tools (GitGuardian, TruffleHog)
8. Consider external secret managers (HashiCorp Vault, AWS Secrets Manager)

**Q5: Explain visual regression testing and when you would use it.**

**Answer:**
Visual regression testing automatically captures screenshots of UI and compares them against baseline images to detect unintended visual changes. 

**When to use:**
- Component libraries (ensure consistent styling)
- Responsive layouts (test multiple viewports)
- Cross-browser compatibility
- CSS refactoring
- Theme changes

**Tools:** Percy, Chromatic, Playwright screenshots

**Challenges:** Can be flaky due to animations, fonts, dynamic content. Requires stabilization techniques.

**Q6: What metrics would you track to monitor bundle size and performance?**

**Answer:**
1. **Bundle Metrics:**
   - Total bundle size (gzipped)
   - Individual chunk sizes
   - Vendor vs application code ratio
   - First-party vs third-party code

2. **Performance Metrics (Web Vitals):**
   - LCP (Largest Contentful Paint) - < 2.5s
   - FID (First Input Delay) - < 100ms
   - CLS (Cumulative Layout Shift) - < 0.1
   - TTFB (Time to First Byte) - < 600ms
   - FCP (First Contentful Paint) - < 1.8s

3. **Custom Metrics:**
   - Time to Interactive
   - Route change performance
   - API response times
   - Error rates

**Q7: How would you implement feature flags in a CI/CD pipeline?**

**Answer:**
```typescript
// 1. Configuration-based flags
const features = {
  newDashboard: import.meta.env.VITE_FEATURE_NEW_DASHBOARD === 'true',
  betaPayments: import.meta.env.VITE_FEATURE_BETA_PAYMENTS === 'true',
};

// 2. Runtime flags (LaunchDarkly, Split.io)
import { useLDClient } from 'launchdarkly-react-client-sdk';

function App() {
  const ldClient = useLDClient();
  const showNewFeature = ldClient.variation('new-feature', false);
  
  return showNewFeature ? <NewFeature /> : <OldFeature />;
}

// 3. CI/CD Integration
// Deploy same build with different flags per environment
// .github/workflows/deploy.yml
env:
  VITE_FEATURE_NEW_DASHBOARD: ${{ github.ref == 'refs/heads/main' }}
```

**Benefits:**
- Deploy features without releasing them
- Gradual rollouts
- A/B testing
- Quick rollback without redeployment

**Q8: Describe your approach to testing a CI/CD pipeline.**

**Answer:**
1. **Pipeline as Code Review:**
   - Validate YAML syntax
   - Check for security issues
   - Ensure proper secret handling

2. **Dry Runs:**
   - Test pipeline logic locally (act for GitHub Actions)
   - Validate scripts in isolation

3. **Progressive Rollout:**
   - Test in feature branch first
   - Deploy to staging before production
   - Use preview environments

4. **Monitoring:**
   - Track pipeline execution time
   - Monitor success/failure rates
   - Alert on anomalies

5. **Rollback Plan:**
   - Keep previous pipeline versions
   - Document emergency procedures
   - Test rollback process

---

### Practical Scenario Questions

**Q9: Your CI pipeline suddenly starts failing. How would you debug it?**

**Answer:**
1. **Check recent changes:**
   - Review recent commits
   - Check for dependency updates
   - Verify configuration changes

2. **Examine logs:**
   - Read error messages carefully
   - Look for environment differences
   - Check for timeout issues

3. **Reproduce locally:**
   - Run tests on your machine
   - Use same Node version as CI
   - Check environment variables

4. **Isolate the problem:**
   - Comment out jobs to find culprit
   - Run tests individually
   - Check for flaky tests

5. **Fix and verify:**
   - Apply fix
   - Test in feature branch
   - Monitor next few runs

**Q10: How would you optimize a build that takes 30 minutes?**

**Answer:**
1. **Profile the build:**
   ```bash
   npm run build -- --profile
   ```
   Identify slowest parts

2. **Implement caching:**
   ```yaml
   - uses: actions/cache@v3
     with:
       path: |
         node_modules
         .next/cache
       key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
   ```

3. **Parallelize jobs:**
   ```yaml
   jobs:
     test:
       strategy:
         matrix:
           shard: [1, 2, 3, 4]
   ```

4. **Use faster alternatives:**
   - Switch to pnpm or yarn for faster installs
   - Use esbuild instead of webpack
   - Use swc instead of babel

5. **Optimize dependencies:**
   - Remove unused dependencies
   - Use tree-shaking
   - Lazy load large libraries

**Target:** Reduce to <10 minutes

**Q11: Design a deployment strategy for a high-traffic e-commerce site.**

**Answer:**
1. **Blue-Green Deployment:**
   - Maintain two identical environments
   - Deploy to inactive (green)
   - Switch traffic after verification
   - Keep blue as instant rollback

2. **Canary Releases:**
   - Deploy to 5% of users first
   - Monitor metrics (errors, latency, conversions)
   - Gradually increase to 25%, 50%, 100%
   - Auto-rollback on anomalies

3. **Feature Flags:**
   - Enable features gradually
   - A/B test new checkout flow
   - Quick disable without redeployment

4. **Database Migrations:**
   - Backward-compatible changes
   - Deploy code before schema changes
   - Use feature flags for data model changes

5. **Monitoring:**
   - Real-time error tracking
   - Performance monitoring
   - Business metrics (conversion rate, revenue)
   - Automated alerts

6. **Rollback Plan:**
   - Instant switch to blue environment
   - Database rollback scripts ready
   - Feature flag kill switches

**Q12: How would you set up CI/CD for a monorepo with multiple apps?**

**Answer:**
```yaml
# .github/workflows/monorepo-ci.yml
name: Monorepo CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  # Detect changed packages
  changes:
    runs-on: ubuntu-latest
    outputs:
      web: ${{ steps.filter.outputs.web }}
      api: ${{ steps.filter.outputs.api }}
      mobile: ${{ steps.filter.outputs.mobile }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            web:
              - 'packages/web/**'
            api:
              - 'packages/api/**'
            mobile:
              - 'packages/mobile/**'

  # Test web app only if changed
  test-web:
    needs: changes
    if: needs.changes.outputs.web == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test --workspace=web

  # Test API only if changed
  test-api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test --workspace=api

  # Deploy affected packages
  deploy:
    needs: [test-web, test-api]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy changed packages
        run: |
          npx turbo run deploy --filter=[HEAD^1]
```

**Tools:**
- Turborepo or Nx for build orchestration
- Path filtering for selective testing
- Shared pipelines for common tasks
- Workspace-aware dependency caching

---

This comprehensive guide covers CI/CD from basics to advanced topics, including practical examples, best practices, and interview preparation. You should now be well-equipped to implement robust CI/CD pipelines and ace system design interviews!


