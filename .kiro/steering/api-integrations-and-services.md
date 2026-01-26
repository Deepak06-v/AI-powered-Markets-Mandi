# API Integrations and Services Documentation

## External API Integrations

### 1. AGMARKNET API (Primary Data Source)
**Purpose**: Real-time agricultural market prices from Government of India
**Provider**: Ministry of Agriculture & Farmers Welfare, Government of India
**Coverage**: 900+ agricultural markets across India

#### API Endpoints
```typescript
// Base URL
const AGMARKNET_BASE_URL = 'https://api.data.gov.in/resource/9ef84268-d588-465a-a308-a864a43d0070';

// Price Query Endpoint
GET /api/agmarknet/prices
Parameters:
- api-key: string (required)
- format: 'json' | 'xml'
- filters[commodity]: string
- filters[state]: string
- filters[district]: string
- filters[market]: string
- filters[arrival_date]: string (YYYY-MM-DD)
- limit: number (default: 100)
- offset: number (default: 0)
```

#### Response Format
```json
{
  "total": 1250,
  "count": 100,
  "records": [
    {
      "state": "Delhi",
      "district": "New Delhi",
      "market": "Azadpur",
      "commodity": "Wheat",
      "variety": "Dara",
      "arrival_date": "2024-01-15",
      "min_price": "1800",
      "max_price": "2200",
      "modal_price": "2050",
      "units": "Quintal"
    }
  ]
}
```

#### Integration Implementation
```typescript
class AGMARKNETService {
  private apiKey: string;
  private baseUrl: string;
  
  async fetchMarketPrices(params: MarketPriceQuery): Promise<MarketPriceResponse> {
    const url = new URL(this.baseUrl);
    url.searchParams.set('api-key', this.apiKey);
    url.searchParams.set('format', 'json');
    url.searchParams.set('filters[commodity]', params.commodity);
    url.searchParams.set('filters[state]', params.state);
    url.searchParams.set('limit', '100');
    
    const response = await fetch(url.toString());
    const data = await response.json();
    
    return this.transformResponse(data);
  }
  
  private transformResponse(data: any): MarketPriceResponse {
    return {
      prices: data.records.map(record => ({
        market: record.market,
        commodity: record.commodity,
        minPrice: parseFloat(record.min_price),
        maxPrice: parseFloat(record.max_price),
        modalPrice: parseFloat(record.modal_price),
        date: new Date(record.arrival_date),
        confidence: this.calculateConfidence(record)
      })),
      totalRecords: data.total,
      lastUpdated: new Date()
    };
  }
}
```

### 2. Hugging Face Whisper API (Speech Recognition)
**Purpose**: High-accuracy multilingual speech recognition
**Model**: Whisper Large V3
**Languages**: 99+ languages including all major Indian languages

#### API Configuration
```typescript
const WHISPER_CONFIG = {
  baseUrl: 'https://api-inference.huggingface.co/models/openai/whisper-large-v3',
  headers: {
    'Authorization': `Bearer ${process.env.VITE_HUGGINGFACE_API_KEY}`,
    'Content-Type': 'audio/wav'
  },
  options: {
    language: 'auto', // Auto-detect or specify: 'hi', 'en', 'bn', etc.
    task: 'transcribe', // 'transcribe' or 'translate'
    return_timestamps: true
  }
};
```

#### Integration Implementation
```typescript
class WhisperService {
  async transcribeAudio(audioBlob: Blob, language?: string): Promise<TranscriptionResult> {
    const formData = new FormData();
    formData.append('file', audioBlob, 'audio.wav');
    
    const response = await fetch(WHISPER_CONFIG.baseUrl, {
      method: 'POST',
      headers: {
        'Authorization': WHISPER_CONFIG.headers.Authorization
      },
      body: formData
    });
    
    const result = await response.json();
    
    return {
      text: result.text,
      language: result.language,
      confidence: result.confidence || 0.8,
      timestamps: result.chunks || []
    };
  }
}
```

### 3. AI4Bharat Indic TTS API (Text-to-Speech)
**Purpose**: High-quality Indian language text-to-speech
**Provider**: AI4Bharat (IIT Madras)
**Languages**: 13 Indian languages with native pronunciation

#### API Endpoints
```typescript
// Primary Endpoint (IIIT Hyderabad)
const INDIC_TTS_API = 'https://tts.ai4bharat.org/api/v1/synthesize';

// Alternative Endpoint (Client-side model)
const INDIC_F5_API = 'https://huggingface.co/ai4bharat/indic-f5-tts';
```

#### Request Format
```typescript
interface TTSRequest {
  text: string;
  language: 'hi' | 'bn' | 'ta' | 'te' | 'gu' | 'kn' | 'ml' | 'mr' | 'or' | 'pa' | 'as';
  speaker: 'male' | 'female';
  speed: number; // 0.5 to 2.0
  pitch: number; // 0.5 to 2.0
}
```

#### Integration Implementation
```typescript
class IndicTTSService {
  async synthesizeSpeech(request: TTSRequest): Promise<AudioBuffer> {
    const response = await fetch(INDIC_TTS_API, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        input: request.text,
        voice: {
          language_code: request.language,
          name: `${request.language}-${request.speaker}`,
          ssml_gender: request.speaker.toUpperCase()
        },
        audio_config: {
          audio_encoding: 'LINEAR16',
          sample_rate_hertz: 24000,
          speaking_rate: request.speed,
          pitch: request.pitch
        }
      })
    });
    
    const audioData = await response.arrayBuffer();
    return this.decodeAudioData(audioData);
  }
}
```

### 4. Browser Web Speech API (Fallback)
**Purpose**: Fallback speech recognition when Whisper unavailable
**Coverage**: Built-in browser support
**Languages**: Varies by browser, good support for major Indian languages

#### Implementation
```typescript
class BrowserSpeechService {
  private recognition: SpeechRecognition | null = null;
  
  initializeSpeechRecognition(language: string): void {
    if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      this.recognition = new SpeechRecognition();
      
      this.recognition.continuous = false;
      this.recognition.interimResults = true;
      this.recognition.lang = this.mapLanguageCode(language);
      this.recognition.maxAlternatives = 3;
    }
  }
  
  async startListening(): Promise<string> {
    return new Promise((resolve, reject) => {
      if (!this.recognition) {
        reject(new Error('Speech recognition not supported'));
        return;
      }
      
      this.recognition.onresult = (event) => {
        const result = event.results[0][0].transcript;
        resolve(result);
      };
      
      this.recognition.onerror = (event) => {
        reject(new Error(`Speech recognition error: ${event.error}`));
      };
      
      this.recognition.start();
    });
  }
}
```

## Internal Service Architecture

### 1. Price Calculation Service
**Purpose**: Intelligent price analysis and recommendation engine

```typescript
class PriceCalculationService {
  calculateFairPrice(params: PriceCalculationParams): PriceAnalysis {
    const basePrice = this.getBasePrice(params);
    const qualityAdjustment = this.calculateQualityPremium(params.quality);
    const quantityDiscount = this.calculateBulkDiscount(params.quantity);
    const seasonalFactor = this.getSeasonalAdjustment(params.commodity, params.date);
    const locationFactor = this.getLocationPremium(params.location);
    
    const adjustedPrice = basePrice * 
      (1 + qualityAdjustment) * 
      (1 - quantityDiscount) * 
      seasonalFactor * 
      locationFactor;
    
    return {
      recommendedPrice: Math.round(adjustedPrice),
      priceRange: {
        min: Math.round(adjustedPrice * 0.9),
        max: Math.round(adjustedPrice * 1.1)
      },
      confidence: this.calculateConfidence(params),
      factors: {
        quality: qualityAdjustment,
        quantity: quantityDiscount,
        seasonal: seasonalFactor,
        location: locationFactor
      }
    };
  }
  
  private calculateQualityPremium(quality: string): number {
    const qualityPremiums = {
      'premium': 0.15,  // 15% premium
      'good': 0.08,     // 8% premium
      'average': 0.0,   // No adjustment
      'below_average': -0.12 // 12% discount
    };
    return qualityPremiums[quality] || 0;
  }
  
  private calculateBulkDiscount(quantity: number): number {
    if (quantity >= 100) return 0.05;      // 5% discount for 100+ quintals
    if (quantity >= 50) return 0.03;       // 3% discount for 50+ quintals
    if (quantity >= 20) return 0.015;      // 1.5% discount for 20+ quintals
    return 0;
  }
}
```

### 2. Translation Service
**Purpose**: Multilingual support for mandi terminology

```typescript
class TranslationService {
  private mandiDictionary: Map<string, Map<string, string>>;
  
  constructor() {
    this.initializeDictionary();
  }
  
  translateText(text: string, fromLang: string, toLang: string): string {
    // Dictionary-based translation for mandi terms
    const translated = this.translateMandiTerms(text, fromLang, toLang);
    
    // Rule-based sentence structure translation
    return this.applyGrammarRules(translated, fromLang, toLang);
  }
  
  private initializeDictionary(): void {
    this.mandiDictionary = new Map([
      ['wheat', new Map([
        ['hi', 'गेहूं'],
        ['bn', 'গম'],
        ['ta', 'கோதுமை'],
        ['te', 'గోధుమ'],
        ['gu', 'ઘઉં']
      ])],
      ['rice', new Map([
        ['hi', 'चावल'],
        ['bn', 'চাল'],
        ['ta', 'அரிசி'],
        ['te', 'బియ్యం'],
        ['gu', 'ચોખા']
      ])],
      ['price', new Map([
        ['hi', 'कीमत'],
        ['bn', 'দাম'],
        ['ta', 'விலை'],
        ['te', 'ధర'],
        ['gu', 'કિંમત']
      ])]
    ]);
  }
}
```

### 3. Geolocation Service
**Purpose**: Location detection and mandi mapping

```typescript
class GeolocationService {
  async getCurrentLocation(): Promise<LocationData> {
    return new Promise((resolve, reject) => {
      if (!navigator.geolocation) {
        reject(new Error('Geolocation not supported'));
        return;
      }
      
      navigator.geolocation.getCurrentPosition(
        async (position) => {
          const { latitude, longitude } = position.coords;
          const locationData = await this.reverseGeocode(latitude, longitude);
          resolve(locationData);
        },
        (error) => {
          reject(new Error(`Geolocation error: ${error.message}`));
        },
        {
          enableHighAccuracy: true,
          timeout: 10000,
          maximumAge: 300000 // 5 minutes
        }
      );
    });
  }
  
  async findNearbyMandis(location: LocationData, radius: number = 50): Promise<MandiInfo[]> {
    // Query mandi database for nearby markets
    const mandis = await this.queryMandiDatabase({
      latitude: location.latitude,
      longitude: location.longitude,
      radius: radius
    });
    
    return mandis.map(mandi => ({
      ...mandi,
      distance: this.calculateDistance(location, mandi.location)
    })).sort((a, b) => a.distance - b.distance);
  }
}
```

## Data Models and Interfaces

### Core Data Types
```typescript
interface MarketPriceData {
  commodity: string;
  variety?: string;
  market: string;
  state: string;
  district: string;
  minPrice: number;
  maxPrice: number;
  modalPrice: number;
  arrivalDate: Date;
  units: 'Quintal' | 'Kg' | 'Ton';
  confidence: number;
}

interface VendorProfile {
  id: string;
  name: string;
  location: LocationData;
  preferredLanguage: string;
  commodities: string[];
  averageQuantity: number;
  qualityRating: number;
  transactionHistory: Transaction[];
}

interface Transaction {
  id: string;
  vendorId: string;
  buyerId: string;
  commodity: string;
  quantity: number;
  agreedPrice: number;
  marketPrice: number;
  date: Date;
  location: string;
  paymentMethod: 'cash' | 'bank_transfer' | 'upi';
  status: 'pending' | 'completed' | 'cancelled';
}

interface AIInsight {
  type: 'price_trend' | 'market_opportunity' | 'negotiation_tip';
  message: string;
  confidence: number;
  actionable: boolean;
  expiresAt?: Date;
}
```

### API Response Types
```typescript
interface PriceAnalysisResponse {
  recommendedPrice: number;
  priceRange: {
    min: number;
    max: number;
  };
  marketComparison: MarketComparison[];
  trends: PriceTrend[];
  insights: AIInsight[];
  confidence: number;
  dataSource: 'agmarknet' | 'cached' | 'estimated';
  lastUpdated: Date;
}

interface VoiceProcessingResponse {
  transcription: string;
  language: string;
  confidence: number;
  processedValue: any;
  fieldType: 'name' | 'product' | 'quantity' | 'quality' | 'location';
  isValid: boolean;
  suggestions?: string[];
}
```

## Error Handling and Fallbacks

### API Failure Handling
```typescript
class APIFallbackManager {
  async executeWithFallback<T>(
    primaryFn: () => Promise<T>,
    fallbackFn: () => Promise<T>,
    cacheKey?: string
  ): Promise<T> {
    try {
      const result = await primaryFn();
      if (cacheKey) {
        this.cacheResult(cacheKey, result);
      }
      return result;
    } catch (primaryError) {
      console.warn('Primary API failed, trying fallback:', primaryError);
      
      try {
        return await fallbackFn();
      } catch (fallbackError) {
        console.error('Fallback also failed:', fallbackError);
        
        // Try cached data as last resort
        if (cacheKey) {
          const cached = this.getCachedResult(cacheKey);
          if (cached) {
            return cached;
          }
        }
        
        throw new Error('All API options exhausted');
      }
    }
  }
}
```

### Service Health Monitoring
```typescript
class ServiceHealthMonitor {
  private healthStatus: Map<string, ServiceHealth> = new Map();
  
  async checkServiceHealth(serviceName: string): Promise<ServiceHealth> {
    const health: ServiceHealth = {
      service: serviceName,
      status: 'unknown',
      lastCheck: new Date(),
      responseTime: 0,
      errorRate: 0
    };
    
    const startTime = Date.now();
    
    try {
      await this.pingService(serviceName);
      health.status = 'healthy';
      health.responseTime = Date.now() - startTime;
    } catch (error) {
      health.status = 'unhealthy';
      health.error = error.message;
    }
    
    this.healthStatus.set(serviceName, health);
    return health;
  }
  
  getServiceRecommendation(serviceName: string): 'use' | 'fallback' | 'cache' {
    const health = this.healthStatus.get(serviceName);
    
    if (!health || health.status === 'unknown') {
      return 'use'; // Try primary service
    }
    
    if (health.status === 'healthy' && health.responseTime < 5000) {
      return 'use';
    }
    
    if (health.status === 'degraded' || health.responseTime > 5000) {
      return 'fallback';
    }
    
    return 'cache'; // Service is down
  }
}
```

## Performance Optimization

### Caching Strategy
```typescript
class CacheManager {
  private cache: Map<string, CacheEntry> = new Map();
  private readonly TTL = {
    prices: 30 * 60 * 1000,      // 30 minutes
    translations: 24 * 60 * 60 * 1000, // 24 hours
    locations: 60 * 60 * 1000,   // 1 hour
    voice: 5 * 60 * 1000         // 5 minutes
  };
  
  set(key: string, value: any, type: keyof typeof this.TTL): void {
    const entry: CacheEntry = {
      value,
      timestamp: Date.now(),
      ttl: this.TTL[type]
    };
    
    this.cache.set(key, entry);
  }
  
  get(key: string): any | null {
    const entry = this.cache.get(key);
    
    if (!entry) return null;
    
    if (Date.now() - entry.timestamp > entry.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.value;
  }
}
```

### Request Batching
```typescript
class RequestBatcher {
  private batches: Map<string, BatchRequest[]> = new Map();
  private timers: Map<string, NodeJS.Timeout> = new Map();
  
  addToBatch(type: string, request: BatchRequest): Promise<any> {
    return new Promise((resolve, reject) => {
      const batch = this.batches.get(type) || [];
      batch.push({ ...request, resolve, reject });
      this.batches.set(type, batch);
      
      // Set timer to execute batch
      if (!this.timers.has(type)) {
        const timer = setTimeout(() => {
          this.executeBatch(type);
        }, 100); // 100ms batch window
        
        this.timers.set(type, timer);
      }
    });
  }
  
  private async executeBatch(type: string): Promise<void> {
    const batch = this.batches.get(type) || [];
    this.batches.delete(type);
    
    const timer = this.timers.get(type);
    if (timer) {
      clearTimeout(timer);
      this.timers.delete(type);
    }
    
    if (batch.length === 0) return;
    
    try {
      const results = await this.processBatch(type, batch);
      batch.forEach((request, index) => {
        request.resolve(results[index]);
      });
    } catch (error) {
      batch.forEach(request => {
        request.reject(error);
      });
    }
  }
}
```

This comprehensive API integration documentation ensures reliable, performant, and scalable service integration for the Multilingual Mandi Voice Assistant.