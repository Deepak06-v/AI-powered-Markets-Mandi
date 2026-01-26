# Technical Architecture

## Core Services Architecture

### 1. aiService.ts - Main AI Orchestrator
**Purpose**: Central coordinator for all AI operations
- `MultilingualMandiAI` class: Main service orchestrator
- `PriceReasoningService`: Market intelligence and pricing logic
- `TranslationService`: Multilingual translation capabilities
- `VoiceService`: Speech synthesis integration

**Key Methods**:
- `processMandiQuery()`: Main entry point for price queries
- `translateText()`: Language translation with fallbacks
- `generateVoiceResponse()`: Text-to-speech generation

### 2. guidedVoiceService.ts - Voice Form Filling
**Purpose**: State machine-driven voice form interaction
- 6-step state machine: Name → Product → Quantity → Quality → Location → Confirmation
- Dual speech recognition: Whisper Large V3 + Browser Web Speech API
- Language-specific question handling
- Form data validation and normalization

**State Machine**:
```typescript
enum GuidedVoiceState {
  ASKING_NAME = 'asking_name',
  ASKING_PRODUCT = 'asking_product', 
  ASKING_QUANTITY = 'asking_quantity',
  ASKING_QUALITY = 'asking_quality',
  ASKING_LOCATION = 'asking_location',
  CONFIRMATION = 'confirmation'
}
```

### 3. mandiPriceService.ts - Price Data Integration
**Purpose**: Real market price data from government sources
- AGMARKNET API integration (Ministry of Agriculture, Govt. of India)
- 900+ agricultural markets across India
- Mock data generation for demo/fallback
- Price aggregation and statistical analysis
- Market confidence calculation

**Data Sources**:
- Primary: AGMARKNET open data API
- Fallback: Rule-based pricing with seasonal adjustments
- Confidence scoring based on data freshness and completeness

### 4. indicTTSService.ts - Text-to-Speech
**Purpose**: High-quality Indian language speech synthesis
- AI4Bharat Indic TTS integration (IIT Madras research)
- 11+ Indian language support with native phonetics
- Client-side IndicF5 model integration
- Server-side IIIT Indic TTS API
- Browser TTS fallback system

**Supported Languages**:
- Hindi, English, Bengali, Tamil, Telugu, Marathi
- Gujarati, Kannada, Malayalam, Punjabi, Odia, Assamese

### 5. whisperService.ts - Advanced Speech Recognition
**Purpose**: High-accuracy multilingual speech recognition
- Hugging Face Whisper Large V3 integration
- Audio recording and real-time transcription
- Language mapping and auto-detection
- Comprehensive error handling with fallbacks

**Features**:
- 24kHz audio sampling
- Real-time transcription
- Language confidence scoring
- Automatic fallback to browser Web Speech API

### 6. voiceAssistantService.ts - Alternative Voice Interface
**Purpose**: Step-by-step voice guidance system
- Alternative to guided voice for different interaction patterns
- Language detection from speech input
- Field value processing and normalization
- Comprehensive error handling and retry logic

## UI Component Architecture

### Core Components

#### 1. GuidedVoiceForm.tsx
**Purpose**: Main voice form interface with state machine
- Voice controls with real-time visual feedback
- Manual input fallback for accessibility
- API key configuration for Whisper integration
- Progress tracking with step indicators
- Form field updates in real-time

**Key Features**:
- Microphone permission handling
- Audio recording visualization
- Language selection dropdown
- Form preview with editing capabilities

#### 2. PriceInsightsPage.tsx
**Purpose**: Market discovery and analysis interface
- Price range visualization with position indicators
- Nearby mandi comparison tables
- 7-day trend charts with volume data
- Trade recording dialog system
- Buyer connection interface

**Components**:
- Price range slider with confidence indicators
- Market comparison table with sorting
- Historical trend charts
- Buyer profile cards with contact options

#### 3. AIInsightsCard.tsx
**Purpose**: AI response display and interaction
- Price analysis presentation with confidence scores
- Translation results with source/target languages
- Voice playback controls with progress indicators
- Copy-to-clipboard functionality
- Data source attribution

#### 4. IndicTTSShowcase.tsx
**Purpose**: TTS demonstration and testing
- Language selection with native script display
- Text input with character limits
- Audio playback controls with speed adjustment
- Quality comparison between different TTS engines
- Voice selection (male/female options)

#### 5. VoiceAssistantForm.tsx
**Purpose**: Alternative voice interface
- Step-by-step guidance with clear instructions
- Voice/text input toggle options
- Form preview with real-time editing
- Progress indicators and navigation controls

### UI Component Library (shadcn-ui)

**Base Components** (src/components/ui/):
- `button.tsx`: Consistent button styling with variants
- `card.tsx`: Container components with elevation
- `input.tsx`: Form inputs with validation states
- `select.tsx`: Dropdown selections with search
- `dialog.tsx`: Modal dialogs for confirmations
- `badge.tsx`: Status indicators and labels
- `progress.tsx`: Progress bars and indicators
- `toast.tsx`: Notification system
- `tabs.tsx`: Tabbed interfaces
- `table.tsx`: Data tables with sorting

## Hooks and Utilities

### Custom Hooks

#### 1. useGuidedVoice.ts
**Purpose**: React hook for guided voice service integration
- State management for voice form flow
- Speech recognition lifecycle management
- Error handling and retry logic
- Form data synchronization

#### 2. useMandiAI.ts
**Purpose**: React hook for AI service integration
- AI query processing and state management
- Response caching and optimization
- Error boundary integration
- Loading state management

#### 3. useGeolocation.ts
**Purpose**: Location detection and management
- Browser geolocation API integration
- Location permission handling
- Fallback location selection
- Privacy-conscious implementation

### Utility Functions

#### languageUtils.ts
**Purpose**: Language processing and formatting utilities
- Language detection from text/script
- Font selection for different scripts
- Text formatting and normalization
- Language code mapping and conversion

**Key Functions**:
- `detectLanguage()`: Script-based language detection
- `getLanguageFont()`: Appropriate font selection
- `formatText()`: Text normalization for different languages
- `getLanguageDirection()`: RTL/LTR text direction

## Data Flow Architecture

### 1. Voice Input Flow
```
User Speech → Whisper/Web Speech API → Language Detection → 
Text Normalization → Field Validation → State Update → 
Form Preview Update → Confirmation
```

### 2. Price Discovery Flow
```
Form Data → Location Detection → AGMARKNET API Query → 
Price Analysis → Confidence Calculation → AI Reasoning → 
Translation → TTS Generation → UI Display
```

### 3. Negotiation Flow
```
Price Data → Market Analysis → Counter-offer Generation → 
Buyer Matching → Contact Information → Trade Recording → 
Market Intelligence Update
```

## State Management

### React Query Integration
- API response caching with intelligent invalidation
- Background refetching for real-time data
- Optimistic updates for better UX
- Error boundary integration

### Local State Management
- React hooks for component-level state
- Context providers for shared state
- State machines for predictable flows
- Persistent storage for user preferences

## Error Handling Strategy

### Graceful Degradation
- Multiple fallback options for each service
- Progressive enhancement based on browser capabilities
- Offline functionality where possible
- Clear error messages in user's language

### Error Boundaries
- Component-level error isolation
- Service-level error recovery
- User-friendly error reporting
- Automatic retry mechanisms

## Performance Optimization

### Code Splitting
- Route-based code splitting
- Component lazy loading
- Service worker integration
- Bundle size optimization

### Caching Strategy
- API response caching
- Static asset caching
- Browser storage utilization
- CDN integration for assets

## Security Considerations

### API Key Management
- Environment variable configuration
- Client-side key validation
- Rate limiting implementation
- Secure key rotation

### Data Privacy
- Minimal data collection
- Local processing where possible
- User consent management
- GDPR compliance considerations

## Testing Architecture

### Unit Testing (Vitest)
- Service layer testing with mocks
- Component testing with React Testing Library
- Hook testing with custom test utilities
- Utility function testing

### Integration Testing
- API integration testing
- Voice service testing
- End-to-end user flow testing
- Cross-browser compatibility testing

### Test Coverage
- 90%+ code coverage target
- Critical path testing priority
- Edge case handling validation
- Performance regression testing