# Multilingual Mandi Voice Assistant - Complete Project Documentation

## Project Overview

This is a sophisticated **Multilingual Mandi (Agricultural Market) Voice Assistant** designed to help local Indian vendors discover fair market prices, negotiate with buyers, and conduct business in their native languages. The application combines AI-driven price reasoning, multilingual translation, and voice interaction capabilities.

## Core Purpose
Help Indian agricultural vendors:
- Discover fair market prices using government data (AGMARKNET)
- Negotiate effectively with buyers
- Conduct business in their native languages (11+ Indian languages)
- Access real-time market intelligence
- Connect with verified buyers

## Key Features

### 1. Guided Voice Form System
- **6-step state machine**: Name → Product → Quantity → Quality → Location → Confirmation
- **One question at a time**: Predictable flow for low-literacy users
- **Dual speech recognition**: Whisper Large V3 + Web Speech API fallback
- **11+ language support**: Hindi, English, Bengali, Tamil, Telugu, Marathi, Gujarati, Kannada, Malayalam, Punjabi, Odia, Assamese
- **Manual text fallback**: Always available for accessibility
- **Real-time form preview**: Visual feedback as fields are filled

### 2. AI-Powered Price Discovery
- **Real mandi price integration**: AGMARKNET (Ministry of Agriculture) API
- **Government data source**: 900+ agricultural markets across India
- **Smart price analysis**: Quantity discounts, seasonal factors, market intelligence
- **Confidence scoring**: Data quality-based reliability indicators
- **Price range visualization**: Low/average/high prices with position indicator

### 3. Multilingual Translation System
- **Dictionary-based translation**: Common mandi terms (Hindi ↔ English)
- **Rule-based patterns**: Sentence structure for negotiation phrases
- **Language detection**: Script-based detection for 10+ Indian languages
- **Context-aware explanations**: Government data references in vendor's language

### 4. AI4Bharat Indic TTS
- **Best-in-class Indian language TTS**: IIT Madras research-backed
- **11+ language support**: Native Indian phonetics and pronunciation
- **High-quality audio**: 24kHz sampling with natural prosody
- **Multi-speaker support**: Male/female voices with adjustable speed
- **Open source**: MIT license, completely free

### 5. Negotiation Assistant
- **AI-generated counter-offers**: Based on real market data
- **Smart buyer connection**: Verified buyers in nearby mandis
- **Trade recording**: Transaction logging for market analysis
- **Market sentiment analysis**: Demand, supply, and quality indicators

### 6. Market Insights Dashboard
- **7-day price trends**: Historical data with volume information
- **Nearby mandi prices**: Regional market comparison
- **AI recommendations**: Predictive insights on optimal selling times
- **Custom mandi tracking**: Monitor specific markets

## Technology Stack
- **Frontend**: React 18, TypeScript, Vite
- **UI Framework**: shadcn-ui with Radix UI components
- **Styling**: Tailwind CSS with custom animations
- **State Management**: React Query, React Hooks
- **Routing**: React Router v6
- **Animations**: Framer Motion
- **Speech APIs**: Web Speech API, Whisper Large V3, AI4Bharat Indic TTS
- **Testing**: Vitest with comprehensive coverage
- **Backend**: Supabase integration
- **Build**: Vite with TypeScript support

## Development Guidelines
- Follow TypeScript best practices with strict type checking
- Use existing UI components from src/components/ui
- Maintain consistent error handling with graceful fallbacks
- Test voice features across different browsers and devices
- Keep accessibility in mind - voice-first with text fallback
- Ensure multilingual support in all new features
- Use state machines for predictable user flows
- Integrate real government data sources when possible
- Maintain open-source compatibility (no proprietary dependencies)