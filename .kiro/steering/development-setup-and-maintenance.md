# Development Setup and Maintenance Documentation

## Development Environment Setup

### Prerequisites
- **Node.js**: Version 18.x or higher (LTS recommended)
- **npm**: Version 9.x or higher (comes with Node.js)
- **Git**: Latest version for version control
- **VS Code**: Recommended IDE with extensions
- **Chrome/Firefox**: For testing voice features

### Required VS Code Extensions
```json
// .vscode/extensions.json
{
  "recommendations": [
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "ms-playwright.playwright",
    "vitest.explorer",
    "ms-vscode.vscode-json",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense"
  ]
}
```

### VS Code Settings
```json
// .vscode/settings.json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
    ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ],
  "files.associations": {
    "*.css": "tailwindcss"
  }
}
```

### Initial Setup Steps

#### 1. Clone and Install
```bash
# Clone the repository
git clone <repository-url>
cd multilingual-mandi-voice-assistant

# Install dependencies
npm install

# Copy environment variables
cp .env.example .env

# Install Playwright browsers (for E2E testing)
npx playwright install
```

#### 2. Environment Configuration
```bash
# .env.development
VITE_ENVIRONMENT=development
VITE_API_BASE_URL=http://localhost:3000
VITE_HUGGINGFACE_API_KEY=your_huggingface_token_here
VITE_AGMARKNET_API_KEY=your_agmarknet_key_here
VITE_ENABLE_MOCK_DATA=true
VITE_LOG_LEVEL=debug

# Optional: For testing real APIs
VITE_ENABLE_REAL_APIS=false
VITE_CACHE_DURATION=300000
```

#### 3. Development Scripts
```json
// package.json scripts
{
  "scripts": {
    "dev": "vite --host",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "type-check": "tsc --noEmit",
    "format": "prettier --write \"src/**/*.{ts,tsx,js,jsx,json,css,md}\"",
    "analyze": "npm run build && npx vite-bundle-analyzer dist/stats.html"
  }
}
```

### Development Workflow

#### Daily Development Process
1. **Start Development Server**
   ```bash
   npm run dev
   ```

2. **Run Tests in Watch Mode**
   ```bash
   npm run test
   ```

3. **Type Checking**
   ```bash
   npm run type-check
   ```

4. **Linting and Formatting**
   ```bash
   npm run lint:fix
   npm run format
   ```

#### Feature Development Workflow
1. **Create Feature Branch**
   ```bash
   git checkout -b feature/voice-enhancement
   ```

2. **Implement Feature with TDD**
   - Write failing tests first
   - Implement minimum code to pass tests
   - Refactor and optimize

3. **Test Across Browsers**
   ```bash
   npm run test:e2e
   ```

4. **Performance Check**
   ```bash
   npm run build
   npm run analyze
   ```

5. **Create Pull Request**
   - Ensure all tests pass
   - Include performance metrics
   - Document breaking changes

## Code Quality Standards

### TypeScript Configuration
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/services/*": ["./src/services/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/utils/*": ["./src/utils/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### ESLint Configuration
```javascript
// eslint.config.js
import js from '@eslint/js'
import globals from 'globals'
import reactHooks from 'eslint-plugin-react-hooks'
import reactRefresh from 'eslint-plugin-react-refresh'
import tseslint from 'typescript-eslint'
import a11y from 'eslint-plugin-jsx-a11y'

export default tseslint.config(
  { ignores: ['dist'] },
  {
    extends: [js.configs.recommended, ...tseslint.configs.recommended],
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
      'jsx-a11y': a11y,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'warn',
      '@typescript-eslint/no-explicit-any': 'warn',
      'jsx-a11y/alt-text': 'error',
      'jsx-a11y/aria-props': 'error',
      'jsx-a11y/aria-proptypes': 'error',
      'jsx-a11y/aria-unsupported-elements': 'error',
      'jsx-a11y/role-has-required-aria-props': 'error',
      'jsx-a11y/role-supports-aria-props': 'error',
    },
  },
)
```

### Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

## Testing Setup and Guidelines

### Vitest Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        'dist/'
      ],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80
        }
      }
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

### Test Setup File
```typescript
// src/test/setup.ts
import '@testing-library/jest-dom'
import { vi } from 'vitest'

// Mock Web Speech API
Object.defineProperty(window, 'SpeechRecognition', {
  writable: true,
  value: vi.fn().mockImplementation(() => ({
    start: vi.fn(),
    stop: vi.fn(),
    abort: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
  })),
})

Object.defineProperty(window, 'webkitSpeechRecognition', {
  writable: true,
  value: window.SpeechRecognition,
})

// Mock Geolocation API
Object.defineProperty(navigator, 'geolocation', {
  writable: true,
  value: {
    getCurrentPosition: vi.fn().mockImplementation((success) => {
      success({
        coords: {
          latitude: 28.6139,
          longitude: 77.2090,
        },
      })
    }),
    watchPosition: vi.fn(),
    clearWatch: vi.fn(),
  },
})

// Mock MediaDevices API
Object.defineProperty(navigator, 'mediaDevices', {
  writable: true,
  value: {
    getUserMedia: vi.fn().mockResolvedValue({
      getTracks: () => [{ stop: vi.fn() }],
    }),
  },
})

// Global test utilities
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))
```

### Playwright Configuration
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:4173',
    trace: 'on-first-retry',
    video: 'retain-on-failure',
    screenshot: 'only-on-failure',
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
    command: 'npm run preview',
    port: 4173,
    reuseExistingServer: !process.env.CI,
  },
});
```

## Debugging and Development Tools

### Browser DevTools Setup
```typescript
// src/utils/devtools.ts
export const enableDevTools = (): void => {
  if (import.meta.env.DEV) {
    // Voice debugging
    (window as any).__VOICE_DEBUG__ = {
      logTranscription: (text: string, language: string) => {
        console.log(`🎤 [${language}] ${text}`);
      },
      logTTSRequest: (text: string, language: string) => {
        console.log(`🔊 [${language}] ${text}`);
      },
      logPriceCalculation: (params: any, result: any) => {
        console.log('💰 Price Calculation:', { params, result });
      }
    };

    // Performance monitoring
    (window as any).__PERF_MONITOR__ = {
      startTimer: (label: string) => {
        console.time(label);
      },
      endTimer: (label: string) => {
        console.timeEnd(label);
      }
    };
  }
};
```

### React DevTools Integration
```typescript
// src/main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'
import { enableDevTools } from './utils/devtools'

// Enable development tools
enableDevTools()

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### Voice Testing Tools
```typescript
// src/utils/voiceTestingUtils.ts
export class VoiceTestingUtils {
  static createMockAudioBlob(duration: number = 1000): Blob {
    const arrayBuffer = new ArrayBuffer(duration * 44.1);
    return new Blob([arrayBuffer], { type: 'audio/wav' });
  }

  static mockSpeechRecognitionResult(text: string, language: string = 'hi'): void {
    const mockEvent = {
      results: [[{
        transcript: text,
        confidence: 0.9
      }]],
      resultIndex: 0
    };

    // Trigger mock recognition result
    if (window.mockSpeechRecognition) {
      window.mockSpeechRecognition.onresult(mockEvent);
    }
  }

  static simulateVoiceInput(element: HTMLElement, text: string): void {
    const event = new CustomEvent('voiceInput', {
      detail: { transcript: text, confidence: 0.9 }
    });
    element.dispatchEvent(event);
  }
}
```

## Performance Monitoring and Optimization

### Performance Monitoring Setup
```typescript
// src/utils/performance.ts
export class PerformanceMonitor {
  private static instance: PerformanceMonitor;
  private metrics: Map<string, number[]> = new Map();

  static getInstance(): PerformanceMonitor {
    if (!PerformanceMonitor.instance) {
      PerformanceMonitor.instance = new PerformanceMonitor();
    }
    return PerformanceMonitor.instance;
  }

  startMeasurement(label: string): void {
    performance.mark(`${label}-start`);
  }

  endMeasurement(label: string): number {
    performance.mark(`${label}-end`);
    performance.measure(label, `${label}-start`, `${label}-end`);
    
    const measure = performance.getEntriesByName(label)[0];
    const duration = measure.duration;
    
    // Store metric
    if (!this.metrics.has(label)) {
      this.metrics.set(label, []);
    }
    this.metrics.get(label)!.push(duration);
    
    // Log in development
    if (import.meta.env.DEV) {
      console.log(`⏱️ ${label}: ${duration.toFixed(2)}ms`);
    }
    
    return duration;
  }

  getAverageTime(label: string): number {
    const times = this.metrics.get(label) || [];
    return times.reduce((sum, time) => sum + time, 0) / times.length;
  }

  getMetricsReport(): Record<string, { average: number; count: number; latest: number }> {
    const report: Record<string, any> = {};
    
    this.metrics.forEach((times, label) => {
      report[label] = {
        average: this.getAverageTime(label),
        count: times.length,
        latest: times[times.length - 1] || 0
      };
    });
    
    return report;
  }
}
```

### Bundle Analysis
```bash
# Analyze bundle size
npm run build
npx vite-bundle-analyzer dist/stats.html

# Check for unused dependencies
npx depcheck

# Audit for security vulnerabilities
npm audit
npm audit fix
```

## Maintenance and Updates

### Dependency Management
```bash
# Check for outdated packages
npm outdated

# Update dependencies safely
npm update

# Update major versions (with caution)
npx npm-check-updates -u
npm install

# Audit and fix security issues
npm audit fix
```

### Regular Maintenance Tasks

#### Weekly Tasks
1. **Dependency Updates**
   ```bash
   npm outdated
   npm update
   npm audit fix
   ```

2. **Performance Check**
   ```bash
   npm run build
   npm run analyze
   ```

3. **Test Coverage Review**
   ```bash
   npm run test:coverage
   ```

#### Monthly Tasks
1. **Major Dependency Updates**
   ```bash
   npx npm-check-updates -u
   npm install
   npm test
   ```

2. **Performance Benchmarking**
   ```bash
   npm run test:e2e
   # Review performance metrics
   ```

3. **Security Audit**
   ```bash
   npm audit
   npx audit-ci --moderate
   ```

#### Quarterly Tasks
1. **Technology Stack Review**
   - Evaluate new React features
   - Consider TypeScript updates
   - Review testing framework updates

2. **Performance Optimization**
   - Bundle size optimization
   - Code splitting improvements
   - Caching strategy review

3. **Accessibility Audit**
   ```bash
   npm run test:e2e -- --grep "accessibility"
   ```

### Troubleshooting Common Issues

#### Voice Recognition Not Working
```typescript
// Debug voice recognition issues
const debugVoiceRecognition = (): void => {
  console.log('Browser support:', {
    SpeechRecognition: 'SpeechRecognition' in window,
    webkitSpeechRecognition: 'webkitSpeechRecognition' in window,
    mediaDevices: 'mediaDevices' in navigator,
    getUserMedia: navigator.mediaDevices?.getUserMedia !== undefined
  });
  
  // Test microphone access
  navigator.mediaDevices?.getUserMedia({ audio: true })
    .then(() => console.log('✅ Microphone access granted'))
    .catch(err => console.error('❌ Microphone access denied:', err));
};
```

#### API Integration Issues
```typescript
// Debug API connectivity
const debugAPIConnectivity = async (): Promise<void> => {
  const apis = [
    { name: 'AGMARKNET', url: 'https://api.data.gov.in/resource/9ef84268-d588-465a-a308-a864a43d0070' },
    { name: 'Hugging Face', url: 'https://api-inference.huggingface.co/models/openai/whisper-large-v3' }
  ];
  
  for (const api of apis) {
    try {
      const response = await fetch(api.url, { method: 'HEAD' });
      console.log(`✅ ${api.name}: ${response.status}`);
    } catch (error) {
      console.error(`❌ ${api.name}:`, error);
    }
  }
};
```

#### Performance Issues
```typescript
// Performance debugging
const debugPerformance = (): void => {
  // Check bundle size
  console.log('Bundle info:', {
    totalSize: performance.getEntriesByType('navigation')[0]?.transferSize,
    domContentLoaded: performance.getEntriesByType('navigation')[0]?.domContentLoadedEventEnd,
    loadComplete: performance.getEntriesByType('navigation')[0]?.loadEventEnd
  });
  
  // Check memory usage
  if ('memory' in performance) {
    console.log('Memory usage:', (performance as any).memory);
  }
};
```

### Documentation Maintenance

#### Keeping Documentation Updated
1. **Code Changes**: Update relevant documentation when making code changes
2. **API Changes**: Document breaking changes and migration guides
3. **Performance Changes**: Update performance benchmarks
4. **New Features**: Add comprehensive feature documentation

#### Documentation Review Process
1. **Monthly Review**: Check for outdated information
2. **Version Updates**: Update version-specific information
3. **User Feedback**: Incorporate feedback from developers
4. **Accessibility**: Ensure documentation is accessible

This comprehensive development setup and maintenance documentation ensures smooth development workflow, high code quality, and long-term project maintainability for the Multilingual Mandi Voice Assistant.