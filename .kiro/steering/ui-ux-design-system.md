# UI/UX Design System

## Design Philosophy

### Voice-First Design Principles
- **Accessibility First**: Every voice interaction has text fallback
- **Progressive Enhancement**: Works without JavaScript, enhanced with it
- **Inclusive Design**: Supports low-literacy users with visual cues
- **Cultural Sensitivity**: Respects Indian cultural contexts and languages
- **Predictable Interactions**: State machine ensures consistent behavior

### Visual Design Language
- **Clean & Minimal**: Reduces cognitive load for users
- **High Contrast**: Ensures readability in various lighting conditions
- **Large Touch Targets**: Mobile-friendly for field use
- **Native Script Support**: Proper fonts for all Indian languages
- **Status Indicators**: Clear visual feedback for all states

## Color System

### Primary Colors
```css
--primary: 142 76% 36%        /* Green #2d7d32 - Agricultural theme */
--primary-foreground: 0 0% 98% /* White text on primary */
--secondary: 210 40% 98%       /* Light gray #f8fafc */
--secondary-foreground: 222 84% 5% /* Dark text on secondary */
```

### Semantic Colors
```css
--success: 142 76% 36%         /* Green - Successful operations */
--warning: 38 92% 50%          /* Orange - Warnings and alerts */
--error: 0 84% 60%             /* Red - Errors and failures */
--info: 221 83% 53%            /* Blue - Information */
```

### Voice State Colors
```css
--voice-listening: 142 76% 36%  /* Green - Actively listening */
--voice-processing: 38 92% 50%  /* Orange - Processing speech */
--voice-error: 0 84% 60%        /* Red - Voice recognition error */
--voice-success: 142 76% 36%    /* Green - Successfully captured */
```

## Typography System

### Font Families
```css
/* Primary - Inter for English/Latin */
font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;

/* Hindi/Devanagari */
font-family: 'Noto Sans Devanagari', 'Inter', sans-serif;

/* Bengali */
font-family: 'Noto Sans Bengali', 'Inter', sans-serif;

/* Tamil */
font-family: 'Noto Sans Tamil', 'Inter', sans-serif;

/* Telugu */
font-family: 'Noto Sans Telugu', 'Inter', sans-serif;

/* Gujarati */
font-family: 'Noto Sans Gujarati', 'Inter', sans-serif;
```

### Type Scale
```css
--text-xs: 0.75rem    /* 12px - Small labels */
--text-sm: 0.875rem   /* 14px - Body text small */
--text-base: 1rem     /* 16px - Body text */
--text-lg: 1.125rem   /* 18px - Large body text */
--text-xl: 1.25rem    /* 20px - Small headings */
--text-2xl: 1.5rem    /* 24px - Medium headings */
--text-3xl: 1.875rem  /* 30px - Large headings */
--text-4xl: 2.25rem   /* 36px - Extra large headings */
```

## Component Design Patterns

### Voice Controls
```typescript
interface VoiceControlProps {
  isListening: boolean;
  isProcessing: boolean;
  hasError: boolean;
  onStartListening: () => void;
  onStopListening: () => void;
  disabled?: boolean;
}
```

**Visual States**:
- **Idle**: Gray microphone icon with "Tap to speak" text
- **Listening**: Animated green microphone with pulsing ring
- **Processing**: Orange microphone with spinner
- **Error**: Red microphone with error message
- **Success**: Green checkmark with confirmation

### Form Field States
```typescript
interface FormFieldState {
  value: string;
  isValid: boolean;
  isRequired: boolean;
  hasVoiceInput: boolean;
  validationMessage?: string;
}
```

**Visual Indicators**:
- **Empty Required**: Red border with asterisk
- **Voice Filled**: Green border with microphone icon
- **Text Filled**: Blue border with keyboard icon
- **Invalid**: Red border with error message
- **Valid**: Green border with checkmark

### Progress Indicators
```typescript
interface StepIndicatorProps {
  currentStep: number;
  totalSteps: number;
  completedSteps: number[];
  stepLabels: string[];
}
```

**Visual Design**:
- Horizontal progress bar with step markers
- Completed steps: Green circles with checkmarks
- Current step: Blue circle with number
- Future steps: Gray circles with numbers
- Step labels below in user's language

## Layout System

### Grid System
```css
/* Mobile-first responsive grid */
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

/* Breakpoints */
@media (min-width: 640px) { /* sm */ }
@media (min-width: 768px) { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }
```

### Spacing Scale
```css
--space-1: 0.25rem   /* 4px */
--space-2: 0.5rem    /* 8px */
--space-3: 0.75rem   /* 12px */
--space-4: 1rem      /* 16px */
--space-6: 1.5rem    /* 24px */
--space-8: 2rem      /* 32px */
--space-12: 3rem     /* 48px */
--space-16: 4rem     /* 64px */
```

## Voice Interaction Patterns

### Guided Voice Flow
1. **Language Selection**
   - Visual: Flag icons with language names in native script
   - Voice: "Please select your language" in multiple languages
   - Fallback: English with visual language picker

2. **Question Presentation**
   - Visual: Large text in user's language with voice icon
   - Audio: Question spoken in selected language
   - Progress: Step indicator showing current position

3. **Response Capture**
   - Visual: Animated microphone with waveform
   - Audio: Confirmation beep when recording starts/stops
   - Feedback: Real-time transcription display

4. **Confirmation**
   - Visual: Filled form field with edit option
   - Audio: "I heard [response], is this correct?"
   - Actions: Confirm, retry, or edit manually

### Error Handling Patterns
1. **Speech Recognition Failure**
   - Visual: Red microphone with error message
   - Audio: "Sorry, I didn't catch that. Please try again."
   - Fallback: Manual text input option appears

2. **Network Connectivity Issues**
   - Visual: Offline indicator with retry button
   - Message: "Connection lost. Using offline mode."
   - Fallback: Local processing where possible

3. **API Service Failures**
   - Visual: Warning icon with service status
   - Message: "Service temporarily unavailable"
   - Fallback: Alternative service or cached data

## Mobile-First Design

### Touch Targets
- Minimum 44px × 44px for all interactive elements
- Voice buttons: 64px × 64px for easy access
- Form inputs: 48px height minimum
- Navigation elements: 48px height minimum

### Responsive Breakpoints
```css
/* Mobile: 320px - 639px */
.mobile-layout {
  flex-direction: column;
  padding: 1rem;
}

/* Tablet: 640px - 1023px */
.tablet-layout {
  flex-direction: row;
  padding: 2rem;
}

/* Desktop: 1024px+ */
.desktop-layout {
  max-width: 1200px;
  padding: 3rem;
}
```

### Gesture Support
- **Swipe**: Navigate between form steps
- **Tap and Hold**: Start voice recording
- **Double Tap**: Repeat last audio
- **Pinch to Zoom**: Increase text size for accessibility

## Accessibility Features

### Screen Reader Support
```typescript
// ARIA labels for voice controls
<button
  aria-label="Start voice recording for product name"
  aria-describedby="voice-help-text"
  aria-pressed={isListening}
>
  <MicrophoneIcon />
</button>
```

### Keyboard Navigation
- Tab order follows logical flow
- Enter key activates voice recording
- Escape key cancels current operation
- Arrow keys navigate between form fields

### High Contrast Mode
```css
@media (prefers-contrast: high) {
  :root {
    --primary: 0 0% 0%;
    --background: 0 0% 100%;
    --border: 0 0% 0%;
  }
}
```

### Reduced Motion
```css
@media (prefers-reduced-motion: reduce) {
  .voice-animation {
    animation: none;
  }
  
  .transition-all {
    transition: none;
  }
}
```

## Language-Specific Design

### Text Direction Support
```css
/* RTL support for Urdu/Arabic */
[dir="rtl"] .form-field {
  text-align: right;
}

[dir="rtl"] .voice-button {
  margin-left: 0;
  margin-right: 1rem;
}
```

### Script-Specific Styling
```css
/* Devanagari (Hindi) */
.devanagari {
  font-family: 'Noto Sans Devanagari';
  line-height: 1.6;
  letter-spacing: 0.02em;
}

/* Bengali */
.bengali {
  font-family: 'Noto Sans Bengali';
  line-height: 1.7;
}

/* Tamil */
.tamil {
  font-family: 'Noto Sans Tamil';
  line-height: 1.6;
}
```

## Animation System

### Voice Feedback Animations
```css
/* Listening pulse animation */
@keyframes listening-pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.1); opacity: 0.8; }
}

/* Processing spinner */
@keyframes processing-spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Success checkmark */
@keyframes success-draw {
  0% { stroke-dashoffset: 100; }
  100% { stroke-dashoffset: 0; }
}
```

### Page Transitions
```css
/* Slide in from right */
.page-enter {
  transform: translateX(100%);
  opacity: 0;
}

.page-enter-active {
  transform: translateX(0);
  opacity: 1;
  transition: all 300ms ease-out;
}
```

## Data Visualization

### Price Range Display
```typescript
interface PriceRangeProps {
  minPrice: number;
  maxPrice: number;
  currentPrice: number;
  confidence: number;
  currency: string;
}
```

**Visual Design**:
- Horizontal bar with gradient (red → yellow → green)
- Current price indicator with confidence badge
- Min/max labels with currency formatting
- Confidence meter below main bar

### Market Trends Chart
```typescript
interface TrendChartProps {
  data: PricePoint[];
  timeRange: '7d' | '30d' | '90d';
  showVolume: boolean;
}
```

**Visual Elements**:
- Line chart with area fill
- Volume bars below price line
- Interactive tooltips with price details
- Time range selector buttons

## Component Library Usage

### shadcn-ui Components
All components follow the established design system:

```typescript
// Button variants
<Button variant="default">Primary Action</Button>
<Button variant="secondary">Secondary Action</Button>
<Button variant="outline">Outline Button</Button>
<Button variant="ghost">Ghost Button</Button>

// Card layouts
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>Content</CardContent>
  <CardFooter>Footer</CardFooter>
</Card>

// Form controls
<Input placeholder="Enter text" />
<Select>
  <SelectTrigger>
    <SelectValue placeholder="Select option" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="option1">Option 1</SelectItem>
  </SelectContent>
</Select>
```

### Custom Component Extensions
```typescript
// Voice-enabled input
<VoiceInput
  placeholder="Speak or type product name"
  onVoiceInput={handleVoiceInput}
  onTextInput={handleTextInput}
  language="hi"
/>

// Language-aware button
<LanguageButton
  text="Submit"
  language="hi"
  onClick={handleSubmit}
/>

// Price display
<PriceDisplay
  amount={1500}
  currency="INR"
  confidence={0.85}
  language="hi"
/>
```

## Performance Considerations

### Image Optimization
- WebP format with JPEG fallback
- Responsive images with srcset
- Lazy loading for non-critical images
- Optimized SVG icons

### Font Loading
```css
/* Preload critical fonts */
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/fonts/noto-sans-devanagari.woff2" as="font" type="font/woff2" crossorigin>

/* Font display strategy */
@font-face {
  font-family: 'Inter';
  font-display: swap;
  src: url('/fonts/inter.woff2') format('woff2');
}
```

### CSS Optimization
- Critical CSS inlined
- Non-critical CSS loaded asynchronously
- CSS custom properties for theming
- Minimal CSS bundle size

This design system ensures consistent, accessible, and culturally appropriate user experiences across all components and interactions in the Multilingual Mandi Voice Assistant.