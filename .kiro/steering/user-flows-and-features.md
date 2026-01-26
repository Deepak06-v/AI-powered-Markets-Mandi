# User Flows and Features Documentation

## Primary User Personas

### 1. Local Mandi Vendor (Primary)
- **Profile**: Small-scale agricultural vendor, limited literacy, native language speaker
- **Goals**: Get fair prices, negotiate effectively, understand market trends
- **Challenges**: Language barriers, complex pricing, lack of market information
- **Preferred Interaction**: Voice-first with visual confirmation

### 2. Agricultural Trader (Secondary)
- **Profile**: Medium-scale trader, moderate tech literacy, multilingual
- **Goals**: Quick price discovery, market analysis, buyer connections
- **Challenges**: Time constraints, multiple market monitoring
- **Preferred Interaction**: Mixed voice and text input

### 3. Market Researcher (Tertiary)
- **Profile**: Agricultural analyst, high tech literacy, English proficient
- **Goals**: Market trends, data analysis, price forecasting
- **Challenges**: Data aggregation, trend analysis
- **Preferred Interaction**: Text-based with data visualization

## Core User Flows

### Flow 1: Guided Voice Form (Primary Flow)

#### Step 1: Language Selection
**User Action**: Opens application
**System Response**: 
- Displays language selection with flag icons
- Plays welcome message in multiple languages
- Auto-detects browser language as default

**Voice Interaction**:
```
System: "नमस्ते! Hello! আসসালামু আলাইকুম! Please select your language."
User: "Hindi" or taps Hindi flag
System: "हिंदी चुनी गई। आइए शुरू करते हैं।" (Hindi selected. Let's begin.)
```

#### Step 2: Name Collection
**User Action**: Hears question about name
**System Response**:
- Displays large microphone button
- Shows "Tap to speak" in selected language
- Provides text input fallback

**Voice Interaction**:
```
System: "आपका नाम क्या है?" (What is your name?)
User: "मेरा नाम राम है" (My name is Ram)
System: "राम, क्या यह सही है?" (Ram, is this correct?)
User: "हाँ" (Yes)
```

**Visual Feedback**:
- Form field updates with "राम" (Ram)
- Green checkmark appears
- Progress indicator shows step 1/6 complete

#### Step 3: Product Information
**Voice Interaction**:
```
System: "आप कौन सा उत्पाद बेचना चाहते हैं?" (Which product do you want to sell?)
User: "गेहूं" (Wheat)
System: "गेहूं, क्या यह सही है?" (Wheat, is this correct?)
User: "हाँ" (Yes)
```

**Smart Features**:
- Auto-suggests common products
- Handles variations ("गेहूं", "wheat", "गहूं")
- Shows product image for confirmation

#### Step 4: Quantity Details
**Voice Interaction**:
```
System: "आपके पास कितना गेहूं है?" (How much wheat do you have?)
User: "पचास क्विंटल" (Fifty quintals)
System: "50 क्विंटल, क्या यह सही है?" (50 quintals, is this correct?)
User: "हाँ" (Yes)
```

**Smart Processing**:
- Converts spoken numbers to digits
- Handles different units (kg, quintal, ton)
- Validates reasonable quantities

#### Step 5: Quality Assessment
**Voice Interaction**:
```
System: "आपके गेहूं की गुणवत्ता कैसी है?" (What is the quality of your wheat?)
User: "अच्छी गुणवत्ता" (Good quality)
System: "अच्छी गुणवत्ता, क्या यह सही है?" (Good quality, is this correct?)
User: "हाँ" (Yes)
```

**Quality Options**:
- Premium/उत्तम (Uttam)
- Good/अच्छा (Accha)
- Average/मध्यम (Madhyam)
- Below Average/निम्न (Nimn)

#### Step 6: Location Information
**Voice Interaction**:
```
System: "आप कहाँ से हैं?" (Where are you from?)
User: "दिल्ली" (Delhi)
System: "दिल्ली, क्या यह सही है?" (Delhi, is this correct?)
User: "हाँ" (Yes)
```

**Location Features**:
- Auto-detects GPS location (with permission)
- Handles city, district, state names
- Suggests nearby mandis

#### Step 7: Confirmation Summary
**System Response**:
- Shows complete form summary
- Reads back all information in user's language
- Provides edit options for each field

**Voice Interaction**:
```
System: "राम जी, आपकी जानकारी: 50 क्विंटल अच्छी गुणवत्ता का गेहूं, दिल्ली से। क्या यह सब सही है?" 
(Ram ji, your information: 50 quintals of good quality wheat from Delhi. Is this all correct?)
User: "हाँ, सही है" (Yes, it's correct)
System: "धन्यवाद! अब मैं आपके लिए कीमत की जानकारी लाता हूँ।" 
(Thank you! Now I'll get price information for you.)
```

### Flow 2: AI Price Discovery and Analysis

#### Step 1: Data Collection
**System Process**:
- Fetches real-time data from AGMARKNET API
- Queries 900+ agricultural markets across India
- Applies location-based filtering
- Calculates confidence scores

**API Integration**:
```typescript
const priceData = await mandiPriceService.fetchPrices({
  commodity: 'wheat',
  location: 'delhi',
  quantity: 50,
  quality: 'good'
});
```

#### Step 2: Price Analysis
**AI Processing**:
- Analyzes historical trends (7-day, 30-day)
- Applies quantity discounts for bulk sales
- Considers seasonal price variations
- Factors in quality premiums/discounts

**Price Calculation Logic**:
```typescript
const basePrice = getMarketPrice('wheat', 'delhi');
const quantityDiscount = calculateBulkDiscount(50); // 2-3% for 50 quintals
const qualityPremium = getQualityPremium('good'); // 5-10% for good quality
const seasonalFactor = getSeasonalAdjustment('wheat', currentMonth);

const finalPrice = basePrice * (1 + qualityPremium) * (1 - quantityDiscount) * seasonalFactor;
```

#### Step 3: Results Presentation
**Visual Display**:
- Price range slider (₹1,800 - ₹2,200 per quintal)
- Current position indicator (₹2,050)
- Confidence meter (85% reliable)
- Nearby mandi comparison table

**Voice Response**:
```
System: "राम जी, दिल्ली में गेहूं की कीमत ₹2,050 प्रति क्विंटल है। यह अच्छी कीमत है। आपके 50 क्विंटल का कुल मूल्य ₹1,02,500 होगा।"
(Ram ji, wheat price in Delhi is ₹2,050 per quintal. This is a good price. Your 50 quintals total value would be ₹1,02,500.)
```

### Flow 3: Negotiation Assistant

#### Step 1: Counter-Offer Generation
**AI Analysis**:
- Reviews buyer's initial offer
- Compares with market rates
- Generates strategic counter-offers
- Provides negotiation talking points

**Example Scenario**:
```
Buyer Offer: ₹1,900 per quintal
Market Rate: ₹2,050 per quintal
AI Counter-Offer: ₹2,000 per quintal (2.4% below market)

Talking Points:
- "Government data shows ₹2,050 average"
- "Good quality deserves premium"
- "Bulk quantity should get fair price"
```

#### Step 2: Buyer Connection
**System Features**:
- Shows verified buyers in nearby mandis
- Displays buyer ratings and history
- Provides contact options (phone/WhatsApp)
- Shows buyer's typical purchase volumes

**Buyer Profile Display**:
```
Verified Buyer: Sharma Traders
Location: Azadpur Mandi, Delhi
Rating: 4.8/5 (127 transactions)
Typical Purchase: 20-100 quintals
Contact: +91-98765-43210
Preferred Payment: Cash/Bank Transfer
```

#### Step 3: Trade Recording
**Transaction Logging**:
- Records agreed price and quantity
- Stores buyer information
- Updates market intelligence
- Generates transaction receipt

### Flow 4: Market Insights Dashboard

#### Price Trends Visualization
**7-Day Trend Chart**:
- Line graph showing daily price movements
- Volume bars showing market activity
- Seasonal comparison with previous year
- Price volatility indicators

**Data Points**:
```
Day 1: ₹2,020 (Volume: 1,200 quintals)
Day 2: ₹2,035 (Volume: 1,450 quintals)
Day 3: ₹2,050 (Volume: 1,300 quintals)
Day 4: ₹2,045 (Volume: 1,100 quintals)
Day 5: ₹2,060 (Volume: 1,350 quintals)
Day 6: ₹2,055 (Volume: 1,250 quintals)
Day 7: ₹2,050 (Volume: 1,400 quintals)
```

#### Nearby Mandi Comparison
**Market Comparison Table**:
```
Mandi Name          | Distance | Price/Quintal | Quality
--------------------|----------|---------------|----------
Azadpur, Delhi      | 5 km     | ₹2,050       | Good
Ghazipur, Delhi     | 12 km    | ₹2,040       | Good
Najafgarh, Delhi    | 18 km    | ₹2,035       | Good
Sonipat, Haryana    | 45 km    | ₹2,070       | Good
Panipat, Haryana    | 65 km    | ₹2,065       | Good
```

#### AI Recommendations
**Smart Insights**:
```
🔍 Market Analysis:
- Prices trending upward (+1.5% this week)
- High demand in Delhi region
- Quality premium of 8% for good wheat

💡 Recommendations:
- Best time to sell: Next 2-3 days
- Consider Sonipat mandi for ₹20 premium
- Bulk discount available for 50+ quintals

📊 Confidence Level: 85%
Data Source: AGMARKNET (Updated 2 hours ago)
```

## Alternative User Flows

### Flow A: Quick Price Check (Power Users)
1. **Direct Input**: User types "wheat 50 quintals Delhi"
2. **Instant Analysis**: AI processes immediately
3. **Quick Results**: Shows price range and confidence
4. **Optional Details**: Expandable sections for trends/comparison

### Flow B: Voice Assistant Mode (Hands-Free)
1. **Wake Word**: "Hey Mandi" (future feature)
2. **Natural Query**: "What's the wheat price in Delhi today?"
3. **Conversational Response**: Full voice interaction
4. **Follow-up Questions**: "Show me nearby mandis"

### Flow C: Offline Mode (Limited Connectivity)
1. **Cached Data**: Uses last downloaded prices
2. **Offline Indicator**: Shows data freshness
3. **Sync When Online**: Updates when connection restored
4. **Essential Features**: Basic price calculation available

## Error Handling Flows

### Voice Recognition Failures
1. **First Attempt**: "Sorry, I didn't catch that. Please try again."
2. **Second Attempt**: "Let me try a different way. Can you speak slowly?"
3. **Third Attempt**: "Would you like to type your answer instead?"
4. **Fallback**: Shows text input with voice option still available

### Network Connectivity Issues
1. **Detection**: Monitors network status
2. **Notification**: "Connection lost. Using offline mode."
3. **Cached Data**: Shows last known prices with timestamp
4. **Retry Logic**: Attempts reconnection every 30 seconds
5. **Recovery**: "Connection restored. Updating prices..."

### API Service Failures
1. **Primary Failure**: AGMARKNET API unavailable
2. **Fallback 1**: Use cached government data
3. **Fallback 2**: Rule-based pricing model
4. **User Notification**: "Using estimated prices. Accuracy may vary."
5. **Confidence Adjustment**: Lower confidence scores for estimates

## Accessibility Features

### Visual Impairment Support
- **Screen Reader**: Full ARIA label support
- **High Contrast**: Alternative color scheme
- **Large Text**: Scalable font sizes
- **Voice Navigation**: Complete voice-only interaction

### Hearing Impairment Support
- **Visual Feedback**: All audio has visual equivalents
- **Text Alternatives**: Captions for all voice responses
- **Vibration**: Haptic feedback for mobile devices
- **Sign Language**: Future feature consideration

### Motor Impairment Support
- **Large Touch Targets**: 64px minimum for voice buttons
- **Voice Control**: Hands-free operation
- **Dwell Clicking**: Future feature for eye tracking
- **Switch Navigation**: Keyboard-only operation

### Cognitive Accessibility
- **Simple Language**: Clear, concise instructions
- **Consistent Layout**: Predictable interface patterns
- **Progress Indicators**: Clear step-by-step guidance
- **Error Prevention**: Confirmation before actions
- **Help Text**: Context-sensitive assistance

## Performance Optimization

### Voice Processing
- **Local Processing**: Browser Web Speech API for basic recognition
- **Cloud Processing**: Whisper Large V3 for high accuracy
- **Caching**: Store common phrases and responses
- **Compression**: Optimize audio data transmission

### Data Loading
- **Progressive Loading**: Essential data first
- **Background Sync**: Update prices in background
- **Intelligent Caching**: Cache based on user patterns
- **Preloading**: Anticipate user needs

### Mobile Optimization
- **Touch Optimization**: Large, accessible touch targets
- **Battery Efficiency**: Minimize background processing
- **Data Usage**: Compress API responses
- **Offline Capability**: Essential features work offline

This comprehensive user flow documentation ensures all interactions are well-defined, accessible, and optimized for the target user base of Indian agricultural vendors.