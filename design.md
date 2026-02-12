# Design Document: BharatKrishi AI – Sustainable AI Farm Co-Pilot

## Overview

BharatKrishi AI is a mobile-first, offline-capable agricultural intelligence platform designed for small and marginal farmers in India. The system combines computer vision, natural language processing, predictive modeling, and federated learning to provide accessible, privacy-preserving agricultural advisory services.

The architecture prioritizes offline functionality through edge computing, supports multilingual voice interaction for low-literacy users, and integrates with India's agricultural digital infrastructure (AgriStack and e-NAM). The platform aims to deliver measurable impact: 20-30% yield increase, 20% water reduction, and 25% income uplift while enhancing climate resilience.

### Key Design Principles

1. **Offline-First**: Core functionality must work without network connectivity
2. **Privacy-Preserving**: Federated learning ensures farmer data never leaves their device in raw form
3. **Accessibility**: Voice-first interface with multilingual support for low-literacy users
4. **Sustainability**: Recommendations prioritize organic methods and resource conservation
5. **Scalability**: Cloud-native architecture with edge deployment for rural areas
6. **Integration-Ready**: API-first design for AgriStack and e-NAM connectivity

## Architecture

### System Architecture Overview

The system follows a hybrid edge-cloud architecture with three primary tiers:

```
┌─────────────────────────────────────────────────────────────┐
│                     Mobile Application                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   UI Layer   │  │ Voice Layer  │  │  Camera      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Edge Runtime Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Disease    │  │  Simulation  │  │   Voice      │      │
│  │   Detector   │  │   Engine     │  │   Advisory   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Local DB    │  │  FL Client   │  │   Cache      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                    (Sync when online)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     Cloud Services Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Model      │  │  FL Server   │  │  Advisory    │      │
│  │   Registry   │  │              │  │  Service     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  AgriStack   │  │   e-NAM      │  │  Analytics   │      │
│  │  Adapter     │  │   Adapter    │  │  Service     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Architecture Layers

**Mobile Application Layer**
- Native Android application (minimum Android 8.0)
- React Native for cross-platform development with native modules for performance-critical components
- Voice-first UI with large touch targets optimized for outdoor use
- Camera integration for crop image capture with quality validation

**Edge Runtime Layer**
- TensorFlow Lite models for on-device inference
- SQLite for local data persistence
- Background sync service for opportunistic cloud synchronization
- Federated learning client for privacy-preserving model updates
- Local cache for weather data, market prices, and advisory content

**Cloud Services Layer**
- Microservices architecture deployed on Kubernetes
- Model registry for versioned ML models
- Federated learning aggregation server
- Advisory content management system
- Integration adapters for AgriStack and e-NAM APIs
- Analytics and impact measurement service

### Data Flow Patterns

**Offline Operation Flow**
1. User captures crop image → Edge disease detector processes locally → Results displayed
2. User speaks query → Local speech-to-text → Local advisory lookup → Text-to-speech response
3. User creates simulation → Local engine computes using cached data → Results displayed

**Online Synchronization Flow**
1. Device connects to network → Background sync service activates
2. Upload: Local usage data, federated learning gradients, farmer feedback
3. Download: Model updates, fresh weather data, market prices, new advisory content
4. Conflict resolution: Server timestamp wins for reference data, merge for user data

## Components and Interfaces

### 1. Disease Detector Module

**Purpose**: Identify crop diseases and pests from images using computer vision

**Technology Stack**:
- TensorFlow Lite for mobile inference
- MobileNetV3 or EfficientNet-Lite as base architecture
- Custom classification head trained on Indian crop disease dataset
- Image preprocessing pipeline for quality enhancement

**Interfaces**:

```typescript
interface DiseaseDetector {
  // Analyze crop image and return disease predictions
  analyzeImage(image: CropImage): Promise<DiseaseAnalysisResult>
  
  // Validate image quality before analysis
  validateImageQuality(image: CropImage): ImageQualityReport
  
  // Get supported crop types
  getSupportedCrops(): CropType[]
  
  // Update model from federated learning
  updateModel(modelUpdate: ModelWeights): Promise<void>
}

interface CropImage {
  imageData: Uint8Array
  format: 'JPEG' | 'PNG'
  metadata: {
    timestamp: Date
    location?: GeoCoordinates
    cropType?: CropType
  }
}

interface DiseaseAnalysisResult {
  diseases: DetectedDisease[]
  processingTime: number
  modelVersion: string
}

interface DetectedDisease {
  diseaseId: string
  diseaseName: string
  confidence: number // 0.0 to 1.0
  severity: 'low' | 'medium' | 'high' | 'critical'
  affectedArea: BoundingBox
  recommendations: TreatmentRecommendation[]
}

interface ImageQualityReport {
  isAcceptable: boolean
  issues: QualityIssue[]
  suggestions: string[]
}

type QualityIssue = 'blur' | 'low_light' | 'overexposed' | 'wrong_subject' | 'too_far'
```

**Model Architecture**:
- Input: 224x224 RGB image
- Backbone: MobileNetV3-Small (optimized for mobile)
- Classification head: 20+ disease classes + healthy class
- Output: Probability distribution over disease classes
- Model size: <10MB for edge deployment

**Training Strategy**:
- Initial training on curated dataset of Tamil Nadu crop diseases
- Continuous improvement through federated learning
- Data augmentation: rotation, brightness, contrast, crop variations
- Class balancing to handle rare diseases

### 2. Voice Advisory Module

**Purpose**: Provide multilingual voice interaction for agricultural advisory

**Technology Stack**:
- Whisper-tiny for speech-to-text (multilingual support)
- Custom NLU model for agricultural intent classification
- Rule-based + retrieval-based advisory system
- Coqui TTS or similar for text-to-speech synthesis

**Interfaces**:

```typescript
interface VoiceAdvisory {
  // Transcribe farmer's voice query
  transcribeSpeech(audio: AudioBuffer, language: Language): Promise<TranscriptionResult>
  
  // Process query and generate advisory
  processQuery(query: string, context: FarmerContext): Promise<AdvisoryResponse>
  
  // Synthesize speech from text
  synthesizeSpeech(text: string, language: Language): Promise<AudioBuffer>
  
  // Get supported languages
  getSupportedLanguages(): Language[]
}

interface TranscriptionResult {
  text: string
  confidence: number
  language: Language
  alternates?: string[]
}

interface AdvisoryResponse {
  answer: string
  sources: string[]
  relatedTopics: string[]
  actionable: boolean
  priority: 'low' | 'medium' | 'high'
}

interface FarmerContext {
  farmerId: string
  location: GeoCoordinates
  crops: CropInfo[]
  currentSeason: Season
  recentActivities: FarmActivity[]
}

type Language = 'ta' | 'te' | 'hi' | 'kn' | 'ml' // Tamil, Telugu, Hindi, Kannada, Malayalam
```

**Advisory Knowledge Base**:
- Structured agricultural knowledge graph
- Crop-specific best practices
- Disease treatment protocols (organic-first)
- Seasonal advisories
- Government scheme information
- Local expert contributions

**NLU Pipeline**:
1. Speech-to-text transcription
2. Language detection and normalization
3. Intent classification (disease_query, weather_query, market_query, general_advice)
4. Entity extraction (crop_name, disease_name, location, date)
5. Context-aware response generation
6. Text-to-speech synthesis

### 3. Simulation Engine

**Purpose**: Predict crop yields, water usage, and income for different farming scenarios

**Technology Stack**:
- Crop growth models (DSSAT-inspired simplified models)
- Weather data integration (IMD APIs when online)
- Soil-water balance calculations
- Economic modeling for income prediction

**Interfaces**:

```typescript
interface SimulationEngine {
  // Run what-if scenario simulation
  runSimulation(scenario: FarmingScenario): Promise<SimulationResult>
  
  // Compare multiple scenarios
  compareScenarios(scenarios: FarmingScenario[]): Promise<ScenarioComparison>
  
  // Get optimal recommendations
  getOptimalStrategy(constraints: FarmingConstraints): Promise<FarmingScenario>
}

interface FarmingScenario {
  scenarioId: string
  cropType: CropType
  plantingDate: Date
  irrigationSchedule: IrrigationPlan
  fertilizerPlan: FertilizerApplication[]
  pestManagement: PestManagementPlan
  landArea: number // in acres
  soilType: SoilType
}

interface SimulationResult {
  predictedYield: number // in kg
  waterUsage: number // in liters
  estimatedIncome: number // in INR
  estimatedCost: number // in INR
  netProfit: number // in INR
  sustainabilityScore: number // 0-100
  riskFactors: RiskFactor[]
  confidence: number // 0.0-1.0
}

interface ScenarioComparison {
  scenarios: SimulationResult[]
  bestScenario: string // scenarioId
  comparisonMetrics: {
    yieldDifference: number[]
    waterSavings: number[]
    profitDifference: number[]
  }
  recommendations: string[]
}

interface RiskFactor {
  type: 'weather' | 'pest' | 'market' | 'water'
  severity: 'low' | 'medium' | 'high'
  description: string
  mitigation: string
}
```

**Simulation Models**:

**Crop Growth Model**:
- Simplified degree-day accumulation model
- Water stress impact on yield
- Nutrient availability factors
- Pest/disease impact estimation

**Water Balance Model**:
- Evapotranspiration calculation (Penman-Monteith simplified)
- Soil moisture tracking
- Irrigation efficiency factors
- Rainfall incorporation

**Economic Model**:
- Input costs (seeds, fertilizer, pesticides, labor, water)
- Market price integration (e-NAM when available)
- Yield-to-income conversion
- Risk-adjusted returns

### 4. Sustainability Tracker

**Purpose**: Monitor and report water usage and carbon footprint

**Interfaces**:

```typescript
interface SustainabilityTracker {
  // Record farming activity
  recordActivity(activity: FarmActivity): Promise<void>
  
  // Calculate current footprints
  calculateFootprints(farmerId: string, period: DateRange): Promise<FootprintReport>
  
  // Get sustainability recommendations
  getRecommendations(farmerId: string): Promise<SustainabilityRecommendation[]>
  
  // Generate seasonal report
  generateSeasonalReport(farmerId: string, season: Season): Promise<SeasonalReport>
}

interface FarmActivity {
  activityId: string
  farmerId: string
  timestamp: Date
  activityType: 'irrigation' | 'fertilizer' | 'pesticide' | 'plowing' | 'harvesting'
  details: ActivityDetails
}

interface FootprintReport {
  waterFootprint: {
    total: number // liters
    perCrop: Map<CropType, number>
    efficiency: number // 0-100
    comparedToOptimal: number // percentage difference
  }
  carbonFootprint: {
    total: number // kg CO2e
    breakdown: {
      fertilizer: number
      pesticide: number
      irrigation: number
      machinery: number
    }
    comparedToBaseline: number // percentage difference
  }
  sustainabilityScore: number // 0-100
}

interface SustainabilityRecommendation {
  category: 'water' | 'carbon' | 'soil' | 'biodiversity'
  recommendation: string
  potentialSavings: {
    water?: number
    carbon?: number
    cost?: number
  }
  difficulty: 'easy' | 'medium' | 'hard'
  priority: number
}
```

**Calculation Methods**:

**Water Footprint**:
- Blue water: Irrigation water consumption
- Green water: Rainwater consumption (evapotranspiration)
- Grey water: Water pollution from fertilizers/pesticides
- Efficiency metrics: Water productivity (kg yield per m³ water)

**Carbon Footprint**:
- Fertilizer production and application emissions
- Pesticide production and application emissions
- Irrigation energy (pump fuel/electricity)
- Machinery operation emissions
- Soil carbon sequestration (credit)

### 5. Federated Learning Module

**Purpose**: Enable privacy-preserving collaborative learning across farmer community

**Technology Stack**:
- TensorFlow Federated or Flower framework
- Differential privacy mechanisms
- Secure aggregation protocols
- Model compression for efficient transmission

**Interfaces**:

```typescript
interface FederatedLearningClient {
  // Train local model on farmer's data
  trainLocalModel(data: LocalTrainingData): Promise<ModelUpdate>
  
  // Receive and apply global model update
  applyGlobalUpdate(update: GlobalModelUpdate): Promise<void>
  
  // Get training status
  getTrainingStatus(): TrainingStatus
  
  // Opt in/out of federated learning
  setParticipation(participate: boolean): Promise<void>
}

interface FederatedLearningServer {
  // Aggregate model updates from clients
  aggregateUpdates(updates: ModelUpdate[]): Promise<GlobalModelUpdate>
  
  // Validate update quality
  validateUpdate(update: ModelUpdate): ValidationResult
  
  // Get aggregation statistics
  getAggregationStats(): AggregationStats
}

interface ModelUpdate {
  clientId: string // anonymized
  modelWeights: Float32Array
  trainingMetrics: {
    loss: number
    accuracy: number
    sampleCount: number
  }
  privacyBudget: number
}

interface GlobalModelUpdate {
  version: string
  modelWeights: Float32Array
  participantCount: number
  aggregationTimestamp: Date
  performanceMetrics: {
    averageAccuracy: number
    convergence: number
  }
}
```

**Federated Learning Workflow**:

1. **Initialization**: Server distributes initial global model to clients
2. **Local Training**: Each client trains on their local data (crop images, outcomes)
3. **Update Generation**: Client computes model weight updates (gradients)
4. **Privacy Protection**: Apply differential privacy noise to updates
5. **Secure Aggregation**: Server aggregates updates without seeing individual contributions
6. **Global Update**: Server computes new global model and distributes to clients
7. **Iteration**: Repeat process periodically (e.g., weekly)

**Privacy Guarantees**:
- Differential privacy with ε = 1.0 (strong privacy)
- Secure multi-party computation for aggregation
- No raw data transmission (only model updates)
- Minimum aggregation threshold (100 participants)
- Anonymized client identifiers

### 6. Edge Runtime

**Purpose**: Enable offline operation of core features on mobile devices

**Technology Stack**:
- TensorFlow Lite for model inference
- SQLite for local database
- Service Workers for background sync
- IndexedDB for large data caching

**Interfaces**:

```typescript
interface EdgeRuntime {
  // Initialize edge runtime
  initialize(config: EdgeConfig): Promise<void>
  
  // Check if feature available offline
  isFeatureAvailable(feature: Feature): boolean
  
  // Sync with cloud when online
  syncWithCloud(): Promise<SyncResult>
  
  // Get offline capability status
  getOfflineStatus(): OfflineStatus
  
  // Manage local storage
  manageStorage(): Promise<StorageReport>
}

interface EdgeConfig {
  modelPath: string
  databasePath: string
  cacheSize: number // in MB
  syncInterval: number // in minutes
  priorityCrops: CropType[]
  language: Language
}

interface SyncResult {
  success: boolean
  itemsSynced: {
    modelsUpdated: number
    dataUploaded: number
    dataDownloaded: number
  }
  errors: SyncError[]
  nextSyncTime: Date
}

interface OfflineStatus {
  isOnline: boolean
  lastSyncTime: Date
  cachedDataAge: {
    weather: number // days
    marketPrices: number // days
    advisoryContent: number // days
  }
  availableFeatures: Feature[]
  storageUsed: number // in MB
  storageAvailable: number // in MB
}
```

**Storage Management**:
- Priority-based caching (user's crops prioritized)
- Automatic cleanup of old data
- Compression for advisory content
- Incremental model updates (delta compression)

**Sync Strategy**:
- Opportunistic sync when WiFi available
- Batch uploads to minimize data usage
- Conflict resolution (server wins for reference data)
- Retry with exponential backoff for failures

### 7. Integration Adapters

**Purpose**: Connect with AgriStack and e-NAM government platforms

**AgriStack Adapter**:

```typescript
interface AgriStackAdapter {
  // Authenticate farmer
  authenticateFarmer(credentials: FarmerCredentials): Promise<AuthResult>
  
  // Fetch farmer profile
  getFarmerProfile(farmerId: string): Promise<FarmerProfile>
  
  // Fetch land records
  getLandRecords(farmerId: string): Promise<LandRecord[]>
  
  // Get eligible schemes
  getEligibleSchemes(farmerId: string): Promise<GovernmentScheme[]>
}

interface FarmerProfile {
  farmerId: string
  name: string
  contactNumber: string
  address: Address
  landHoldings: number // in acres
  crops: CropType[]
  registrationDate: Date
}

interface LandRecord {
  surveyNumber: string
  area: number // in acres
  soilType: SoilType
  irrigationSource: IrrigationSource
  currentCrop?: CropType
}
```

**e-NAM Adapter**:

```typescript
interface ENAMAdapter {
  // Get current market prices
  getMarketPrices(commodity: CropType, market?: string): Promise<MarketPrice[]>
  
  // Get price trends
  getPriceTrends(commodity: CropType, period: DateRange): Promise<PriceTrend>
  
  // Get nearby markets
  getNearbyMarkets(location: GeoCoordinates, radius: number): Promise<Market[]>
}

interface MarketPrice {
  commodity: CropType
  market: string
  modalPrice: number // in INR per quintal
  minPrice: number
  maxPrice: number
  date: Date
  arrivals: number // quantity in quintals
}

interface PriceTrend {
  commodity: CropType
  dataPoints: PriceDataPoint[]
  trend: 'increasing' | 'decreasing' | 'stable'
  forecast?: number // predicted price
}
```

## Data Models

### Core Entities

**Farmer**:
```typescript
interface Farmer {
  id: string
  name: string
  phoneNumber: string
  preferredLanguage: Language
  location: GeoCoordinates
  registrationDate: Date
  agriStackId?: string
  farms: Farm[]
  preferences: FarmerPreferences
}

interface FarmerPreferences {
  voiceEnabled: boolean
  offlineMode: boolean
  sustainabilityGoals: SustainabilityGoal[]
  notificationSettings: NotificationSettings
  federatedLearningOptIn: boolean
}
```

**Farm**:
```typescript
interface Farm {
  id: string
  farmerId: string
  name: string
  location: GeoCoordinates
  totalArea: number // in acres
  soilType: SoilType
  irrigationSource: IrrigationSource
  plots: Plot[]
}

interface Plot {
  id: string
  farmId: string
  area: number // in acres
  currentCrop?: CropType
  plantingDate?: Date
  expectedHarvestDate?: Date
  activities: FarmActivity[]
}
```

**Crop Disease Record**:
```typescript
interface CropDiseaseRecord {
  id: string
  farmerId: string
  plotId: string
  timestamp: Date
  image: CropImage
  detectedDiseases: DetectedDisease[]
  farmerFeedback?: {
    accuracyRating: number // 1-5
    actualDisease?: string
    treatmentApplied?: string
    outcome?: string
  }
  modelVersion: string
}
```

**Advisory Session**:
```typescript
interface AdvisorySession {
  id: string
  farmerId: string
  timestamp: Date
  queryType: 'voice' | 'text'
  query: string
  language: Language
  response: AdvisoryResponse
  feedback?: {
    helpful: boolean
    rating: number // 1-5
    comments?: string
  }
}
```

**Simulation Record**:
```typescript
interface SimulationRecord {
  id: string
  farmerId: string
  timestamp: Date
  scenarios: FarmingScenario[]
  results: SimulationResult[]
  selectedScenario?: string
  actualOutcome?: {
    yield: number
    waterUsed: number
    income: number
    recordedDate: Date
  }
}
```

### Database Schema

**Local SQLite Schema** (Edge Device):
```sql
-- Farmer profile (synced from cloud)
CREATE TABLE farmers (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  phone_number TEXT,
  preferred_language TEXT,
  location_lat REAL,
  location_lon REAL,
  synced_at TIMESTAMP
);

-- Local farm data
CREATE TABLE farms (
  id TEXT PRIMARY KEY,
  farmer_id TEXT REFERENCES farmers(id),
  name TEXT,
  total_area REAL,
  soil_type TEXT,
  irrigation_source TEXT
);

-- Disease detection history
CREATE TABLE disease_records (
  id TEXT PRIMARY KEY,
  farmer_id TEXT,
  plot_id TEXT,
  timestamp TIMESTAMP,
  image_path TEXT,
  detected_diseases TEXT, -- JSON
  model_version TEXT,
  synced BOOLEAN DEFAULT 0
);

-- Advisory history
CREATE TABLE advisory_sessions (
  id TEXT PRIMARY KEY,
  farmer_id TEXT,
  timestamp TIMESTAMP,
  query TEXT,
  response TEXT, -- JSON
  language TEXT,
  synced BOOLEAN DEFAULT 0
);

-- Cached reference data
CREATE TABLE cached_weather (
  location_key TEXT PRIMARY KEY,
  data TEXT, -- JSON
  cached_at TIMESTAMP
);

CREATE TABLE cached_market_prices (
  commodity TEXT,
  market TEXT,
  data TEXT, -- JSON
  cached_at TIMESTAMP,
  PRIMARY KEY (commodity, market)
);

-- Model metadata
CREATE TABLE models (
  model_name TEXT PRIMARY KEY,
  version TEXT,
  file_path TEXT,
  size_bytes INTEGER,
  updated_at TIMESTAMP
);
```

**Cloud PostgreSQL Schema**:
```sql
-- Farmers (master data)
CREATE TABLE farmers (
  id UUID PRIMARY KEY,
  agristack_id TEXT UNIQUE,
  name TEXT NOT NULL,
  phone_number TEXT,
  preferred_language TEXT,
  location GEOGRAPHY(POINT),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Farms
CREATE TABLE farms (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  name TEXT,
  total_area DECIMAL,
  soil_type TEXT,
  irrigation_source TEXT,
  location GEOGRAPHY(POINT),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Disease detection records (aggregated)
CREATE TABLE disease_detections (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  plot_id UUID,
  timestamp TIMESTAMP,
  image_url TEXT,
  detected_diseases JSONB,
  farmer_feedback JSONB,
  model_version TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Advisory sessions (aggregated)
CREATE TABLE advisory_sessions (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  timestamp TIMESTAMP,
  query_type TEXT,
  query TEXT,
  language TEXT,
  response JSONB,
  feedback JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Simulation records
CREATE TABLE simulations (
  id UUID PRIMARY KEY,
  farmer_id UUID REFERENCES farmers(id),
  timestamp TIMESTAMP,
  scenarios JSONB,
  results JSONB,
  selected_scenario TEXT,
  actual_outcome JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Federated learning metadata
CREATE TABLE fl_rounds (
  id UUID PRIMARY KEY,
  round_number INTEGER,
  model_version TEXT,
  participant_count INTEGER,
  aggregation_timestamp TIMESTAMP,
  performance_metrics JSONB
);

-- Impact metrics (aggregated)
CREATE TABLE impact_metrics (
  id UUID PRIMARY KEY,
  period_start DATE,
  period_end DATE,
  total_farmers INTEGER,
  avg_yield_improvement DECIMAL,
  avg_water_reduction DECIMAL,
  avg_income_uplift DECIMAL,
  calculated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_farmers_location ON farmers USING GIST(location);
CREATE INDEX idx_disease_detections_farmer ON disease_detections(farmer_id);
CREATE INDEX idx_disease_detections_timestamp ON disease_detections(timestamp);
CREATE INDEX idx_advisory_sessions_farmer ON advisory_sessions(farmer_id);
```


## Error Handling

### Error Categories and Strategies

**1. Network Errors**
- **Strategy**: Graceful degradation to offline mode
- **Implementation**: 
  - Detect network unavailability immediately
  - Switch to cached data automatically
  - Queue operations for later sync
  - Display clear offline indicator to user
  - Retry with exponential backoff when network returns

**2. Model Inference Errors**
- **Strategy**: Fallback and user guidance
- **Implementation**:
  - If disease detection fails, request better image with specific guidance
  - If confidence is low (<60%), present multiple possibilities with disclaimer
  - Log errors for model improvement
  - Provide manual advisory option as fallback

**3. Data Quality Errors**
- **Strategy**: Validation and user feedback
- **Implementation**:
  - Validate image quality before processing (blur detection, lighting check)
  - Validate voice input quality (noise level, duration)
  - Provide immediate feedback on data quality issues
  - Guide user to capture better data

**4. Storage Errors**
- **Strategy**: Proactive management and user notification
- **Implementation**:
  - Monitor storage usage continuously
  - Warn user when storage is 80% full
  - Automatically clean old cached data
  - Prioritize essential models and user data
  - Provide manual storage management UI

**5. Integration Errors (AgriStack/e-NAM)**
- **Strategy**: Cached data fallback
- **Implementation**:
  - Use cached data when APIs are unavailable
  - Display data age to user ("Prices from 2 days ago")
  - Retry API calls with timeout limits
  - Log integration failures for monitoring
  - Degrade gracefully (e.g., simulations work without latest prices)

**6. Authentication Errors**
- **Strategy**: Clear user guidance and recovery
- **Implementation**:
  - Provide clear error messages in user's language
  - Offer password reset flow
  - Support offline authentication with cached credentials
  - Lock account after 5 failed attempts with recovery option

**7. Federated Learning Errors**
- **Strategy**: Silent failure with monitoring
- **Implementation**:
  - FL failures should not impact user experience
  - Log FL errors for debugging
  - Skip FL rounds if device resources are constrained
  - Validate model updates before applying

### Error Response Format

```typescript
interface ErrorResponse {
  errorCode: string
  errorMessage: string // in user's language
  severity: 'low' | 'medium' | 'high' | 'critical'
  recoverable: boolean
  suggestedAction?: string
  technicalDetails?: string // for logging
}

// Example error codes
const ErrorCodes = {
  // Network errors
  NETWORK_UNAVAILABLE: 'ERR_NET_001',
  API_TIMEOUT: 'ERR_NET_002',
  
  // Model errors
  MODEL_INFERENCE_FAILED: 'ERR_MODEL_001',
  MODEL_NOT_FOUND: 'ERR_MODEL_002',
  LOW_CONFIDENCE: 'ERR_MODEL_003',
  
  // Data quality errors
  IMAGE_QUALITY_LOW: 'ERR_DATA_001',
  AUDIO_QUALITY_LOW: 'ERR_DATA_002',
  INVALID_INPUT: 'ERR_DATA_003',
  
  // Storage errors
  STORAGE_FULL: 'ERR_STORAGE_001',
  DATABASE_ERROR: 'ERR_STORAGE_002',
  
  // Integration errors
  AGRISTACK_UNAVAILABLE: 'ERR_INT_001',
  ENAM_UNAVAILABLE: 'ERR_INT_002',
  
  // Authentication errors
  AUTH_FAILED: 'ERR_AUTH_001',
  SESSION_EXPIRED: 'ERR_AUTH_002',
  ACCOUNT_LOCKED: 'ERR_AUTH_003'
}
```

### Logging and Monitoring

**Client-Side Logging**:
- Log errors locally with context (user action, device state, timestamp)
- Batch upload logs when online (privacy-preserving, no PII)
- Configurable log levels (ERROR, WARN, INFO, DEBUG)

**Server-Side Monitoring**:
- Real-time error rate monitoring
- Alert on critical errors (>5% error rate)
- Track API latency and availability
- Monitor federated learning convergence
- Track impact metrics (yield, water, income)

**Error Recovery Metrics**:
- Time to recovery for network errors
- Success rate of retry operations
- User abandonment rate after errors
- Fallback usage frequency

## Testing Strategy

### Overview

The testing strategy employs a dual approach combining traditional unit/integration testing with property-based testing to ensure correctness, reliability, and scalability of the BharatKrishi AI platform.

**Testing Principles**:
1. **Offline-First Testing**: All core features must pass tests in offline mode
2. **Multilingual Testing**: Test all features across supported languages
3. **Performance Testing**: Validate response times on low-end devices
4. **Privacy Testing**: Verify no raw data leakage in federated learning
5. **Integration Testing**: Test AgriStack and e-NAM integrations with mocks
6. **Property-Based Testing**: Verify universal correctness properties across all inputs

### Unit Testing

**Scope**: Individual components and functions

**Framework**: Jest for JavaScript/TypeScript, pytest for Python services

**Coverage Target**: 80% code coverage minimum

**Key Areas**:
- Disease detector preprocessing pipeline
- Voice transcription accuracy
- Simulation engine calculations
- Sustainability footprint calculations
- Data validation functions
- Error handling logic

**Example Unit Tests**:
```typescript
describe('DiseaseDetector', () => {
  test('should reject blurry images', () => {
    const blurryImage = loadTestImage('blurry_leaf.jpg')
    const quality = detector.validateImageQuality(blurryImage)
    expect(quality.isAcceptable).toBe(false)
    expect(quality.issues).toContain('blur')
  })
  
  test('should handle empty image gracefully', () => {
    const emptyImage = new CropImage({ imageData: new Uint8Array(0) })
    expect(() => detector.analyzeImage(emptyImage)).toThrow('Invalid image data')
  })
})
```

### Integration Testing

**Scope**: Component interactions and external integrations

**Framework**: Jest with test containers for databases, Postman/Newman for API testing

**Key Areas**:
- Edge runtime sync with cloud services
- AgriStack API integration (with mocks)
- e-NAM API integration (with mocks)
- Database operations (SQLite and PostgreSQL)
- Federated learning client-server communication

**Example Integration Tests**:
```typescript
describe('EdgeRuntime Sync', () => {
  test('should sync disease records when online', async () => {
    const runtime = new EdgeRuntime(testConfig)
    await runtime.initialize()
    
    // Create local disease record
    const record = createTestDiseaseRecord()
    await runtime.saveDiseaseRecord(record)
    
    // Simulate going online
    mockNetworkOnline()
    const syncResult = await runtime.syncWithCloud()
    
    expect(syncResult.success).toBe(true)
    expect(syncResult.itemsSynced.dataUploaded).toBe(1)
    
    // Verify record exists in cloud
    const cloudRecord = await cloudService.getDiseaseRecord(record.id)
    expect(cloudRecord).toMatchObject(record)
  })
})
```

### Property-Based Testing

**Scope**: Universal correctness properties that must hold for all valid inputs

**Framework**: fast-check for JavaScript/TypeScript, Hypothesis for Python

**Configuration**: Minimum 100 iterations per property test

**Tagging Convention**: Each test must reference its design document property
```typescript
// Feature: bharatkrishi-ai, Property 1: Disease detection preserves image metadata
```

**Key Properties** (detailed in Correctness Properties section):
- Image processing preserves metadata
- Simulation results are deterministic for same inputs
- Water footprint calculations are monotonic
- Federated learning maintains privacy bounds
- Voice transcription round-trip consistency
- Offline-online data consistency

**Example Property Test Structure**:
```typescript
import fc from 'fast-check'

describe('Property Tests', () => {
  test('Property 1: Disease detection preserves image metadata', () => {
    // Feature: bharatkrishi-ai, Property 1: Disease detection preserves image metadata
    fc.assert(
      fc.asyncProperty(
        arbitraryCropImage(),
        async (image) => {
          const result = await detector.analyzeImage(image)
          expect(result.metadata.timestamp).toEqual(image.metadata.timestamp)
          expect(result.metadata.location).toEqual(image.metadata.location)
        }
      ),
      { numRuns: 100 }
    )
  })
})
```

### End-to-End Testing

**Scope**: Complete user workflows

**Framework**: Detox for mobile app testing, Playwright for web interfaces

**Key Workflows**:
1. Farmer captures crop image → receives disease diagnosis → views treatment recommendations
2. Farmer asks voice query in Tamil → receives voice response
3. Farmer creates what-if scenario → compares results → selects optimal strategy
4. Farmer views sustainability report → receives recommendations
5. App goes offline → core features continue working → syncs when online

### Performance Testing

**Scope**: Response times, resource usage, scalability

**Framework**: k6 for load testing, Android Profiler for mobile performance

**Performance Targets**:
- Disease detection: <5 seconds on device
- Voice transcription: <3 seconds
- Simulation: <15 seconds
- API response time: <2 seconds (95th percentile)
- App startup time: <3 seconds
- Memory usage: <200MB on device

**Load Testing Scenarios**:
- 10,000 concurrent users
- 1 million disease detections per month
- 100,000 voice queries per day
- Federated learning with 1,000 participants

### Security Testing

**Scope**: Authentication, authorization, data privacy, encryption

**Key Areas**:
- Penetration testing for API endpoints
- Encryption validation (at rest and in transit)
- Federated learning privacy verification
- SQL injection and XSS prevention
- Rate limiting and DDoS protection

### Accessibility Testing

**Scope**: Voice interface, visual design, multilingual support

**Key Areas**:
- Voice recognition accuracy across accents
- Text-to-speech naturalness in regional languages
- UI readability in bright sunlight
- Large touch targets for outdoor use
- Color contrast for visual impairments

### Field Testing

**Scope**: Real-world usage with farmers in Tamil Nadu

**Approach**:
- Pilot with 100 farmers across 5 villages
- Monitor usage patterns and error rates
- Collect qualitative feedback through interviews
- Measure impact metrics (yield, water, income)
- Iterate based on farmer feedback

**Success Criteria**:
- 80% farmer satisfaction rate
- <5% error rate in disease detection
- 90% voice recognition accuracy in field conditions
- Measurable improvements in yield/water/income

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

The following properties define the correctness criteria for BharatKrishi AI. Each property is universally quantified and must hold for all valid inputs. These properties will be implemented as property-based tests with a minimum of 100 iterations each.

### Disease Detection Properties

**Property 1: Disease detection output completeness**
*For any* valid crop image that is analyzed, the disease detection result SHALL include disease name, severity level, and confidence score for each detected disease.
**Validates: Requirements 1.2**

**Property 2: Disease detection language consistency**
*For any* detected disease and any supported regional language, treatment recommendations SHALL be provided in the requested language.
**Validates: Requirements 1.3**

**Property 3: Image quality validation**
*For any* crop image with quality issues (blur, low light, overexposure), the system SHALL reject the image and provide specific guidance on how to capture a better image.
**Validates: Requirements 1.4**

**Property 4: Disease severity ordering**
*For any* crop image with multiple detected diseases, the diseases SHALL be ordered by severity level (critical > high > medium > low).
**Validates: Requirements 1.7**

**Property 5: Offline disease detection**
*For any* valid crop image, disease detection SHALL produce results in offline mode that are equivalent to online mode (same model version).
**Validates: Requirements 1.6**

### Voice Advisory Properties

**Property 6: Voice output language consistency**
*For any* advisory recommendation and any supported regional language, the synthesized speech SHALL be in the requested language.
**Validates: Requirements 2.3**

**Property 7: Offline voice functionality**
*For any* voice query in a supported language, the voice advisory SHALL function in offline mode using local models.
**Validates: Requirements 2.4**

**Property 8: Voice advisory response provision**
*For any* valid agricultural query, the system SHALL provide an advisory response containing actionable guidance.
**Validates: Requirements 2.5**

**Property 9: Noisy audio handling**
*For any* audio input with background noise exceeding acceptable thresholds, the system SHALL request the farmer to repeat the query.
**Validates: Requirements 2.7**

### Simulation Engine Properties

**Property 10: Simulation output completeness**
*For any* valid farming scenario, the simulation result SHALL include predicted yield, water usage, and estimated income.
**Validates: Requirements 3.1, 3.2, 3.3**

**Property 11: Simulation determinism**
*For any* farming scenario with identical inputs (crop, soil, weather, irrigation, fertilizer), running the simulation multiple times SHALL produce identical results.
**Validates: Requirements 3.1, 3.2, 3.3**

**Property 12: Scenario comparison consistency**
*For any* two farming scenarios where scenario A has higher predicted yield than scenario B, the comparison SHALL indicate scenario A as better for yield.
**Validates: Requirements 3.5**

**Property 13: Weather data influence**
*For any* two farming scenarios that differ only in weather data, the simulation results SHALL differ (weather impacts predictions).
**Validates: Requirements 3.6**

**Property 14: Offline simulation with historical data**
*For any* farming scenario in offline mode, the simulation SHALL use historical weather patterns and produce valid results.
**Validates: Requirements 3.7**

**Property 15: Input factor influence**
*For any* two farming scenarios that differ in soil type, crop variety, or climate conditions, the simulation results SHALL differ (these factors impact predictions).
**Validates: Requirements 3.8**

### Sustainability Tracking Properties

**Property 16: Footprint calculation completeness**
*For any* farming activity, the sustainability tracker SHALL calculate both water footprint and carbon footprint.
**Validates: Requirements 4.1, 4.2**

**Property 17: Sustainability report structure**
*For any* completed farming season, the sustainability report SHALL include actual resource usage, optimal resource usage, and comparison metrics.
**Validates: Requirements 4.3**

**Property 18: Water reduction recommendations**
*For any* farmer with tracked activities, the sustainability tracker SHALL provide at least one recommendation with quantified potential water savings.
**Validates: Requirements 4.4**

**Property 19: Chemical impact calculation**
*For any* farming activity involving chemical fertilizers or pesticides, the sustainability tracker SHALL calculate and include environmental impact scores.
**Validates: Requirements 4.5**

**Property 20: Multi-season tracking persistence**
*For any* farmer, sustainability data SHALL be retained and accessible across multiple farming seasons.
**Validates: Requirements 4.6**

### Federated Learning Properties

**Property 21: Privacy-preserving data transmission**
*For any* federated learning update transmitted from client to server, the update SHALL contain only model weights (gradients) and SHALL NOT contain raw farmer data (images, text, personal information).
**Validates: Requirements 5.1**

**Property 22: Differential privacy application**
*For any* model update shared in federated learning, the update SHALL have differential privacy noise applied with epsilon ≤ 1.0.
**Validates: Requirements 5.2**

**Property 23: Minimum aggregation threshold**
*For any* community insights presented to farmers, the insights SHALL be derived from at least 100 participating farmers.
**Validates: Requirements 5.3**

**Property 24: Community insights presentation**
*For any* advisory recommendation that includes community insights, the insights SHALL be clearly labeled as supplementary and community-derived.
**Validates: Requirements 5.4**

**Property 25: Opt-out respect**
*For any* farmer who has opted out of federated learning, their local data SHALL NOT be used for model training and no updates SHALL be transmitted.
**Validates: Requirements 5.5**

**Property 26: Background synchronization**
*For any* federated learning client that transitions from offline to online, model synchronization SHALL be initiated in the background without user intervention.
**Validates: Requirements 5.6**

### Offline Edge Capability Properties

**Property 27: Core features offline availability**
*For any* device in offline mode, disease detection, voice advisory, and simulation engine SHALL all function without network connectivity.
**Validates: Requirements 6.1**

**Property 28: Offline mode indication**
*For any* device without network connectivity, the system SHALL indicate offline mode status to the user.
**Validates: Requirements 6.2**

**Property 29: Online sync trigger**
*For any* device that transitions from offline to online, background synchronization with cloud services SHALL be automatically initiated.
**Validates: Requirements 6.3**

**Property 30: Offline feature availability**
*For any* device in offline mode, at least 90% of core features (disease detection, voice advisory, simulation, sustainability tracking) SHALL be accessible.
**Validates: Requirements 6.5**

**Property 31: Cache retention policy**
*For any* cached data (weather, market prices, advisory content), data SHALL be retained for at least 30 days.
**Validates: Requirements 6.6**

**Property 32: Storage prioritization**
*For any* device with limited storage (<500MB available), the edge runtime SHALL prioritize models and data for the farmer's registered crops and region over other data.
**Validates: Requirements 6.7**

### Integration Properties

**Property 33: Market price integration**
*For any* simulation that includes income prediction, if e-NAM market prices are available, the simulation SHALL use current e-NAM prices rather than historical averages.
**Validates: Requirements 7.3**

**Property 34: Scheme filtering by profile**
*For any* farmer profile retrieved from AgriStack, the displayed government schemes SHALL be filtered to show only schemes for which the farmer is eligible based on their profile attributes.
**Validates: Requirements 7.4**

**Property 35: Integration fallback**
*For any* API call to AgriStack or e-NAM that fails or times out, the system SHALL continue functioning using cached data and SHALL indicate data age to the user.
**Validates: Requirements 7.5**

**Property 36: Land record synchronization**
*For any* farmer whose land records are updated in AgriStack, the local system SHALL reflect the updated records after the next successful synchronization.
**Validates: Requirements 7.7**

### Security and Privacy Properties

**Property 37: Authentication enforcement**
*For any* request to access farmer-specific data, the system SHALL require valid authentication credentials and SHALL reject unauthenticated requests.
**Validates: Requirements 9.3**

**Property 38: Consent-based data sharing**
*For any* attempt to share farmer data with third parties, the system SHALL verify explicit farmer consent exists and SHALL block sharing if consent is not present.
**Validates: Requirements 9.6**

**Property 39: Security alert triggering**
*For any* detected suspicious access pattern (multiple failed logins, unusual access times, unexpected locations), the system SHALL alert the farmer and temporarily lock the account.
**Validates: Requirements 9.7**

**Property 40: Error message localization**
*For any* error that occurs, the error message SHALL be displayed in the farmer's chosen regional language and SHALL use simple, non-technical terminology.
**Validates: Requirements 8.7**

### Advisory Quality Properties

**Property 41: Context-aware recommendations**
*For any* two advisory requests that differ in crop type, growth stage, soil conditions, or climate, the recommendations SHALL differ (context influences advice).
**Validates: Requirements 11.1**

**Property 42: Organic method prioritization**
*For any* treatment recommendations where both organic and chemical options exist, organic methods SHALL be listed before chemical methods.
**Validates: Requirements 11.2**

**Property 43: Treatment recommendation completeness**
*For any* treatment recommendation, the recommendation SHALL include dosage, timing, and application method.
**Validates: Requirements 11.3**

**Property 44: Treatment ranking**
*For any* set of multiple treatment options, the treatments SHALL be ranked by a combination of effectiveness, cost, and environmental impact.
**Validates: Requirements 11.4**

**Property 45: Recommendation source attribution**
*For any* advisory recommendation, when source information is requested, the system SHALL provide sources or reasoning for the recommendation.
**Validates: Requirements 11.5**

**Property 46: Community insight labeling**
*For any* recommendation derived from community insights, the recommendation SHALL be clearly labeled as community-based.
**Validates: Requirements 11.6**

**Property 47: Recommendation updates**
*For any* advisory recommendation that becomes outdated due to new information (weather changes, disease outbreaks), the system SHALL update the recommendation and notify the farmer.
**Validates: Requirements 11.7**

### Impact Measurement Properties

**Property 48: Multi-season yield tracking**
*For any* farmer using the platform, yield data SHALL be tracked and retained across multiple farming seasons.
**Validates: Requirements 12.1**

**Property 49: Water usage tracking**
*For any* farmer using the platform, water usage data SHALL be tracked and compared against baseline measurements.
**Validates: Requirements 12.2**

**Property 50: Income tracking**
*For any* farmer using the platform, income data SHALL be tracked across farming seasons.
**Validates: Requirements 12.3**

**Property 51: Impact report generation**
*For any* aggregated impact report, the report SHALL include metrics for yield improvement, water reduction, and income uplift with progress toward target goals (20-30% yield, 20% water, 25% income).
**Validates: Requirements 12.4, 12.5, 12.6**

**Property 52: Report anonymization**
*For any* impact report generated, individual farmer data SHALL NOT be identifiable and only aggregated statistics SHALL be presented.
**Validates: Requirements 12.7**

### Property Test Implementation Notes

**Test Configuration**:
- Each property test MUST run a minimum of 100 iterations
- Each test MUST be tagged with: `Feature: bharatkrishi-ai-farm-copilot, Property N: [property title]`
- Tests SHOULD use appropriate generators for Indian agricultural context (Tamil Nadu crops, regional languages, typical farm sizes)

**Generator Requirements**:
- Crop images: Various quality levels, lighting conditions, disease types
- Languages: All 5 supported regional languages (Tamil, Telugu, Hindi, Kannada, Malayalam)
- Farming scenarios: Realistic ranges for Tamil Nadu delta region (soil types, crop varieties, irrigation methods)
- Farm activities: Typical activities with realistic resource usage
- Weather data: Historical patterns for Tamil Nadu region

**Edge Cases to Include**:
- Empty or minimal inputs
- Maximum boundary values (large farms, many activities)
- Offline/online transitions
- Network failures and timeouts
- Storage constraints
- Multiple concurrent operations

