# Testing and Deployment Documentation

## Testing Strategy

### Testing Philosophy
- **Voice-First Testing**: All voice interactions must be tested across different accents and languages
- **Accessibility Testing**: Ensure all features work with screen readers and keyboard navigation
- **Cross-Browser Compatibility**: Test across Chrome, Firefox, Safari, and Edge
- **Mobile-First**: Primary testing on mobile devices with fallback to desktop
- **Real-World Scenarios**: Test with actual agricultural vendors and realistic data

### Test Coverage Requirements
- **Unit Tests**: 90%+ coverage for all services and utilities
- **Integration Tests**: All API integrations and service interactions
- **E2E Tests**: Complete user flows from voice input to price discovery
- **Performance Tests**: Voice processing latency and API response times
- **Accessibility Tests**: WCAG 2.1 AA compliance

## Unit Testing (Vitest)

### Service Layer Testing
```typescript
// src/test/guidedVoice.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { GuidedVoiceService, GuidedVoiceState } from '../services/guidedVoiceService';

describe('GuidedVoiceService', () => {
  let service: GuidedVoiceService;
  
  beforeEach(() => {
    service = new GuidedVoiceService();
  });
  
  describe('State Machine', () => {
    it('should start in ASKING_NAME state', () => {
      expect(service.getCurrentState()).toBe(GuidedVoiceState.ASKING_NAME);
    });
    
    it('should transition to ASKING_PRODUCT after name input', () => {
      service.processVoiceInput('राम', GuidedVoiceState.ASKING_NAME);
      expect(service.getCurrentState()).toBe(GuidedVoiceState.ASKING_PRODUCT);
    });
    
    it('should handle invalid state transitions gracefully', () => {
      expect(() => {
        service.processVoiceInput('गेहूं', GuidedVoiceState.ASKING_NAME);
      }).not.toThrow();
    });
  });
  
  describe('Voice Input Processing', () => {
    it('should normalize product names correctly', () => {
      const result = service.normalizeProductName('गेहूं');
      expect(result).toBe('wheat');
    });
    
    it('should handle quantity variations', () => {
      expect(service.parseQuantity('पचास क्विंटल')).toBe(50);
      expect(service.parseQuantity('50 quintals')).toBe(50);
      expect(service.parseQuantity('५० क्विंटल')).toBe(50);
    });
    
    it('should detect confirmation responses', () => {
      expect(service.isConfirmation('हाँ')).toBe(true);
      expect(service.isConfirmation('yes')).toBe(true);
      expect(service.isConfirmation('नहीं')).toBe(false);
    });
  });
});
```

### AI Service Testing
```typescript
// src/test/aiService.test.ts
import { describe, it, expect, vi } from 'vitest';
import { MultilingualMandiAI } from '../services/aiService';

describe('MultilingualMandiAI', () => {
  let aiService: MultilingualMandiAI;
  
  beforeEach(() => {
    aiService = new MultilingualMandiAI();
  });
  
  describe('Price Analysis', () => {
    it('should calculate fair price with quality premium', async () => {
      const mockPriceData = {
        commodity: 'wheat',
        quantity: 50,
        quality: 'good',
        location: 'delhi'
      };
      
      const result = await aiService.processMandiQuery(mockPriceData);
      
      expect(result.recommendedPrice).toBeGreaterThan(0);
      expect(result.confidence).toBeGreaterThan(0.5);
      expect(result.priceRange.min).toBeLessThan(result.recommendedPrice);
      expect(result.priceRange.max).toBeGreaterThan(result.recommendedPrice);
    });
    
    it('should apply bulk discounts correctly', () => {
      const service = new PriceCalculationService();
      
      expect(service.calculateBulkDiscount(20)).toBe(0.015);
      expect(service.calculateBulkDiscount(50)).toBe(0.03);
      expect(service.calculateBulkDiscount(100)).toBe(0.05);
    });
  });
  
  describe('Translation Service', () => {
    it('should translate mandi terms correctly', () => {
      const translation = aiService.translateText('wheat price', 'en', 'hi');
      expect(translation).toContain('गेहूं');
      expect(translation).toContain('कीमत');
    });
    
    it('should handle unknown terms gracefully', () => {
      const translation = aiService.translateText('unknown term', 'en', 'hi');
      expect(translation).toBe('unknown term'); // Should return original
    });
  });
});
```

### Voice Processing Testing
```typescript
// src/test/voiceProcessing.test.ts
describe('Voice Processing', () => {
  describe('Language Detection', () => {
    it('should detect Hindi from Devanagari script', () => {
      expect(detectLanguage('गेहूं की कीमत')).toBe('hi');
    });
    
    it('should detect Bengali from Bengali script', () => {
      expect(detectLanguage('গমের দাম')).toBe('bn');
    });
    
    it('should fallback to English for unknown scripts', () => {
      expect(detectLanguage('wheat price')).toBe('en');
    });
  });
  
  describe('Number Processing', () => {
    it('should convert Hindi numerals to Arabic', () => {
      expect(convertHindiNumerals('५० क्विंटल')).toBe('50 क्विंटल');
    });
    
    it('should parse spoken numbers', () => {
      expect(parseSpokenNumber('पचास')).toBe(50);
      expect(parseSpokenNumber('fifty')).toBe(50);
    });
  });
});
```

## Integration Testing

### API Integration Tests
```typescript
// src/test/integration/apiIntegration.test.ts
describe('API Integration Tests', () => {
  describe('AGMARKNET API', () => {
    it('should fetch real market prices', async () => {
      const service = new MandiPriceService();
      const prices = await service.fetchPrices({
        commodity: 'wheat',
        state: 'delhi',
        limit: 10
      });
      
      expect(prices.length).toBeGreaterThan(0);
      expect(prices[0]).toHaveProperty('modalPrice');
      expect(prices[0]).toHaveProperty('market');
    });
    
    it('should handle API failures gracefully', async () => {
      const service = new MandiPriceService();
      
      // Mock API failure
      vi.spyOn(global, 'fetch').mockRejectedValueOnce(new Error('Network error'));
      
      const prices = await service.fetchPrices({
        commodity: 'wheat',
        state: 'delhi'
      });
      
      // Should return cached or estimated prices
      expect(prices).toBeDefined();
      expect(prices.length).toBeGreaterThan(0);
    });
  });
  
  describe('Whisper API Integration', () => {
    it('should transcribe Hindi audio correctly', async () => {
      const service = new WhisperService();
      const mockAudioBlob = new Blob(['mock audio data'], { type: 'audio/wav' });
      
      const result = await service.transcribeAudio(mockAudioBlob, 'hi');
      
      expect(result).toHaveProperty('text');
      expect(result).toHaveProperty('language');
      expect(result.confidence).toBeGreaterThan(0);
    });
  });
});
```

### Service Integration Tests
```typescript
describe('Service Integration', () => {
  it('should complete full voice-to-price flow', async () => {
    const guidedVoice = new GuidedVoiceService();
    const aiService = new MultilingualMandiAI();
    
    // Simulate voice inputs
    guidedVoice.processVoiceInput('राम', GuidedVoiceState.ASKING_NAME);
    guidedVoice.processVoiceInput('गेहूं', GuidedVoiceState.ASKING_PRODUCT);
    guidedVoice.processVoiceInput('पचास क्विंटल', GuidedVoiceState.ASKING_QUANTITY);
    guidedVoice.processVoiceInput('अच्छी गुणवत्ता', GuidedVoiceState.ASKING_QUALITY);
    guidedVoice.processVoiceInput('दिल्ली', GuidedVoiceState.ASKING_LOCATION);
    
    const formData = guidedVoice.getFormData();
    const priceAnalysis = await aiService.processMandiQuery(formData);
    
    expect(priceAnalysis.recommendedPrice).toBeGreaterThan(0);
    expect(priceAnalysis.insights.length).toBeGreaterThan(0);
  });
});
```

## End-to-End Testing (Playwright)

### E2E Test Setup
```typescript
// e2e/setup.ts
import { test as base, expect } from '@playwright/test';

export const test = base.extend({
  // Custom fixture for voice testing
  voiceContext: async ({ page }, use) => {
    // Grant microphone permissions
    await page.context().grantPermissions(['microphone']);
    
    // Mock Web Speech API
    await page.addInitScript(() => {
      window.SpeechRecognition = class MockSpeechRecognition {
        start() {
          setTimeout(() => {
            this.onresult({
              results: [[{ transcript: 'मॉक वॉइस इनपुट' }]]
            });
          }, 100);
        }
      };
    });
    
    await use(page);
  }
});
```

### Complete User Flow Tests
```typescript
// e2e/guidedVoiceFlow.spec.ts
import { test, expect } from './setup';

test.describe('Guided Voice Flow', () => {
  test('should complete full voice form submission', async ({ page, voiceContext }) => {
    await page.goto('/');
    
    // Start guided voice form
    await page.click('[data-testid="guided-voice-button"]');
    
    // Language selection
    await page.click('[data-testid="language-hindi"]');
    await expect(page.locator('[data-testid="current-language"]')).toContainText('हिंदी');
    
    // Name input
    await page.click('[data-testid="voice-input-button"]');
    await expect(page.locator('[data-testid="form-field-name"]')).toContainText('राम');
    
    // Product input
    await page.click('[data-testid="voice-input-button"]');
    await expect(page.locator('[data-testid="form-field-product"]')).toContainText('गेहूं');
    
    // Continue through all steps...
    
    // Final confirmation
    await page.click('[data-testid="confirm-button"]');
    
    // Verify price results
    await expect(page.locator('[data-testid="price-result"]')).toBeVisible();
    await expect(page.locator('[data-testid="recommended-price"]')).toContainText('₹');
  });
  
  test('should handle voice recognition errors gracefully', async ({ page }) => {
    await page.goto('/');
    
    // Mock speech recognition failure
    await page.evaluate(() => {
      window.SpeechRecognition = class {
        start() {
          setTimeout(() => {
            this.onerror({ error: 'no-speech' });
          }, 100);
        }
      };
    });
    
    await page.click('[data-testid="guided-voice-button"]');
    await page.click('[data-testid="voice-input-button"]');
    
    // Should show error message and text input fallback
    await expect(page.locator('[data-testid="voice-error"]')).toBeVisible();
    await expect(page.locator('[data-testid="text-input-fallback"]')).toBeVisible();
  });
});
```

### Accessibility Testing
```typescript
// e2e/accessibility.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility Tests', () => {
  test('should pass WCAG 2.1 AA compliance', async ({ page }) => {
    await page.goto('/');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
  
  test('should support keyboard navigation', async ({ page }) => {
    await page.goto('/');
    
    // Tab through all interactive elements
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toHaveAttribute('data-testid', 'guided-voice-button');
    
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toHaveAttribute('data-testid', 'manual-entry-button');
    
    // Enter should activate voice recording
    await page.keyboard.press('Enter');
    await expect(page.locator('[data-testid="voice-recording"]')).toBeVisible();
  });
  
  test('should work with screen readers', async ({ page }) => {
    await page.goto('/');
    
    // Check ARIA labels
    const voiceButton = page.locator('[data-testid="voice-input-button"]');
    await expect(voiceButton).toHaveAttribute('aria-label', /start voice recording/i);
    
    // Check live regions for dynamic content
    const liveRegion = page.locator('[aria-live="polite"]');
    await expect(liveRegion).toBeVisible();
  });
});
```

## Performance Testing

### Load Testing
```typescript
// performance/loadTest.js
import { check } from 'k6';
import http from 'k6/http';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Ramp up to 200
    { duration: '5m', target: 200 }, // Stay at 200
    { duration: '2m', target: 0 },   // Ramp down
  ],
};

export default function() {
  // Test price API endpoint
  let response = http.post('http://localhost:3000/api/prices', {
    commodity: 'wheat',
    location: 'delhi',
    quantity: 50,
    quality: 'good'
  });
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 2000ms': (r) => r.timings.duration < 2000,
    'has price data': (r) => JSON.parse(r.body).recommendedPrice > 0,
  });
}
```

### Voice Processing Performance
```typescript
// src/test/performance/voicePerformance.test.ts
describe('Voice Processing Performance', () => {
  it('should process voice input within 2 seconds', async () => {
    const service = new WhisperService();
    const mockAudio = generateMockAudio(3000); // 3 second audio
    
    const startTime = Date.now();
    const result = await service.transcribeAudio(mockAudio);
    const endTime = Date.now();
    
    expect(endTime - startTime).toBeLessThan(2000);
    expect(result.text).toBeDefined();
  });
  
  it('should handle concurrent voice requests', async () => {
    const service = new WhisperService();
    const requests = Array(10).fill(null).map(() => 
      service.transcribeAudio(generateMockAudio(1000))
    );
    
    const results = await Promise.all(requests);
    
    expect(results).toHaveLength(10);
    results.forEach(result => {
      expect(result.text).toBeDefined();
    });
  });
});
```

## Deployment Configuration

### Environment Setup
```bash
# .env.production
VITE_ENVIRONMENT=production
VITE_API_BASE_URL=https://api.mandiai.com
VITE_HUGGINGFACE_API_KEY=hf_prod_key_here
VITE_AGMARKNET_API_KEY=agmarknet_prod_key
VITE_SENTRY_DSN=https://sentry.io/project/dsn
VITE_ANALYTICS_ID=GA_MEASUREMENT_ID

# Performance optimizations
VITE_ENABLE_PWA=true
VITE_ENABLE_SERVICE_WORKER=true
VITE_CACHE_DURATION=3600000
```

### Build Configuration
```typescript
// vite.config.production.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.data\.gov\.in/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'agmarknet-cache',
              expiration: {
                maxEntries: 100,
                maxAgeSeconds: 30 * 60 // 30 minutes
              }
            }
          }
        ]
      }
    })
  ],
  build: {
    target: 'es2015',
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          voice: ['./src/services/whisperService', './src/services/indicTTSService'],
          ai: ['./src/services/aiService', './src/services/mandiPriceService']
        }
      }
    }
  },
  server: {
    headers: {
      'Cross-Origin-Embedder-Policy': 'require-corp',
      'Cross-Origin-Opener-Policy': 'same-origin'
    }
  }
});
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### CI/CD Pipeline (GitHub Actions)
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:integration
      - run: npm run test:e2e
      
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to CDN
        run: |
          aws s3 sync dist/ s3://${{ secrets.S3_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_ID }} --paths "/*"
```

### Monitoring and Analytics
```typescript
// src/utils/monitoring.ts
import * as Sentry from '@sentry/react';

// Error tracking
Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.VITE_ENVIRONMENT,
  tracesSampleRate: 0.1,
  beforeSend(event) {
    // Filter out sensitive data
    if (event.extra?.voiceData) {
      delete event.extra.voiceData;
    }
    return event;
  }
});

// Performance monitoring
export function trackVoiceProcessingTime(duration: number, language: string) {
  Sentry.addBreadcrumb({
    category: 'voice',
    message: `Voice processing completed in ${duration}ms`,
    data: { duration, language },
    level: 'info'
  });
}

// User analytics (privacy-conscious)
export function trackUserFlow(step: string, language: string) {
  // Only track aggregated, anonymous usage patterns
  if (typeof gtag !== 'undefined') {
    gtag('event', 'user_flow', {
      step,
      language,
      timestamp: Date.now()
    });
  }
}
```

### Health Checks and Monitoring
```typescript
// src/utils/healthCheck.ts
export class HealthCheckService {
  async performHealthCheck(): Promise<HealthStatus> {
    const checks = await Promise.allSettled([
      this.checkAGMARKNETAPI(),
      this.checkWhisperAPI(),
      this.checkIndicTTS(),
      this.checkLocalStorage(),
      this.checkGeolocation()
    ]);
    
    return {
      overall: checks.every(check => check.status === 'fulfilled') ? 'healthy' : 'degraded',
      services: {
        agmarknet: checks[0].status === 'fulfilled' ? 'healthy' : 'unhealthy',
        whisper: checks[1].status === 'fulfilled' ? 'healthy' : 'unhealthy',
        indicTTS: checks[2].status === 'fulfilled' ? 'healthy' : 'unhealthy',
        localStorage: checks[3].status === 'fulfilled' ? 'healthy' : 'unhealthy',
        geolocation: checks[4].status === 'fulfilled' ? 'healthy' : 'unhealthy'
      },
      timestamp: new Date().toISOString()
    };
  }
}
```

This comprehensive testing and deployment documentation ensures the Multilingual Mandi Voice Assistant is thoroughly tested, performant, and reliably deployed to production.