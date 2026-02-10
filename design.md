# Design Document: Sahayak Health Services Application

## 1. System Architecture

### 1.1 High-Level Architecture

The Sahayak application follows a mobile-first, cloud-backed architecture optimized for low-bandwidth environments:

```
┌─────────────────────────────────────────────────────────────┐
│                     Mobile Application                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ UI Layer     │  │ AI Agent     │  │ Local Cache  │      │
│  │ (React       │  │ Integration  │  │ (IndexedDB)  │      │
│  │  Native)     │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         │                  │                  │              │
│  ┌──────────────────────────────────────────────────┐      │
│  │         Service Layer (Business Logic)           │      │
│  └──────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Backend Services (Node.js)                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ API Gateway  │  │ Cache Layer  │  │ Data Cleanup │      │
│  │              │  │ (Redis)      │  │ Service      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ ABDM API     │  │ UHI API      │  │ External     │
│              │  │              │  │ Services     │
└──────────────┘  └──────────────┘  └──────────────┘
                                     │
                        ┌────────────┼────────────┐
                        ▼            ▼            ▼
                   ┌─────────┐ ┌─────────┐ ┌─────────┐
                   │ Amazon  │ │ AI Agent│ │ GPS     │
                   │ OCR     │ │ (TTS/   │ │ Service │
                   │         │ │  STT)   │ │         │
                   └─────────┘ └─────────┘ └─────────┘
```

### 1.2 Technology Stack (AWS-Optimized)

**Mobile Application:**
- Framework: React Native with Expo (rapid prototyping)
- State Management: Redux Toolkit
- Local Storage: AsyncStorage
- Offline Support: Redux Persist
- AWS Integration: AWS Amplify SDK

**Backend Services (Serverless AWS):**
- API Layer: AWS API Gateway (REST - Public, Rate-Limited)
- Compute: AWS Lambda (Node.js runtime)
- Cache: Amazon ElastiCache (Redis) with 24-hour TTL
- Database: Amazon DynamoDB with TTL attribute (24-hour auto-deletion)
- Task Scheduler: Amazon EventBridge (for cleanup jobs)
- File Storage: Amazon S3 (prescription images with lifecycle policies)

**AWS AI/ML Services:**
- Foundation Models: Anthropic Claude 3 Sonnet (via Bedrock)
- OCR: Amazon Textract (prescription image processing)
- Translation: Amazon Translate (10+ Indian languages)
- TTS: Amazon Polly (text-to-speech, Indian voices)
- STT: Amazon Transcribe (speech-to-text, Indian languages)
- Medical NLP: Amazon Comprehend Medical (symptom understanding)

**AWS Infrastructure:**
- CDN: Amazon CloudFront (static assets, low-latency)
- Monitoring: Amazon CloudWatch (logs, metrics, alarms)
- Secrets: AWS Secrets Manager (ABDM/UHI API keys)
- Deployment: AWS SAM or Serverless Framework
- Maps/Location: AWS Location Service (geocoding, routing)

**Security (No Authentication Required):**
- API Gateway: Rate limiting (100 req/min per IP) and throttling
- S3: Pre-signed URLs for uploads (15-minute expiry)
- DynamoDB: Automatic TTL-based deletion (24 hours)
- CloudWatch: Request logging for abuse detection
- WAF: AWS WAF for DDoS protection

**External Services:**
- ABDM API Integration (via Lambda)
- UHI API Integration (via Lambda)
- Emergency Services Integration

## 2. Component Design

### 2.1 Mobile Application Components

#### 2.1.1 Patient Profile Module
**Responsibility:** Manage patient demographic information

**Components:**
- `PatientProfileForm`: Input form for name, age, gender
- `PatientProfileStore`: Redux slice for patient data
- `ProfileValidator`: Validates patient input
- `ProfileCleanupService`: Schedules 24-hour data deletion

**Data Model:**
```typescript
interface PatientProfile {
  id: string;
  name: string;
  age: number;
  gender: 'male' | 'female' | 'other';
  createdAt: Date;
  expiresAt: Date; // createdAt + 24 hours
}
```

#### 2.1.2 Symptom Input Module
**Responsibility:** Capture and process patient symptoms

**Components:**
- `SymptomTextInput`: Text-based symptom entry with language support
- `SymptomVoiceInput`: Voice-based symptom capture using STT
- `LanguageSelector`: Choose native language
- `SymptomConfirmation`: Display understood symptoms for validation
- `AIAgentService`: Interface with AI agent for translation and analysis

**Data Model:**
```typescript
interface SymptomInput {
  id: string;
  patientId: string;
  rawText: string;
  language: string;
  translatedText: string; // English translation
  timestamp: Date;
  inputMethod: 'text' | 'voice';
  confirmed: boolean;
}
```

#### 2.1.3 Prescription Processing Module
**Responsibility:** Handle prescription upload and extraction

**Components:**
- `PrescriptionUploader`: Image/text upload interface
- `OCRService`: Amazon Textract integration
- `MedicineExtractor`: Parse medicine names from OCR output
- `ActiveIngredientResolver`: Map medicines to active ingredients
- `PrescriptionStore`: Redux slice for prescription data

**Data Model:**
```typescript
interface Prescription {
  id: string;
  patientId: string;
  imageUrl?: string;
  rawText: string;
  medicines: Medicine[];
  createdAt: Date;
  expiresAt: Date;
}

interface Medicine {
  name: string;
  activeIngredient: string;
  dosage: string;
  frequency: string;
}
```

#### 2.1.4 Location Services Module
**Responsibility:** Capture and manage patient location

**Components:**
- `LocationCapture`: GPS-based location fetching
- `ManualLocationInput`: Manual location entry
- `LocationValidator`: Validate coordinates
- `LocationStore`: Redux slice for location data

**Data Model:**
```typescript
interface Location {
  id: string;
  patientId: string;
  latitude: number;
  longitude: number;
  address: string;
  source: 'gps' | 'manual';
  createdAt: Date;
  expiresAt: Date;
}
```

#### 2.1.5 Doctor Recommendation Module
**Responsibility:** Find and display nearby doctors/specialists

**Components:**
- `DoctorSearch`: Search interface
- `DoctorList`: Display search results
- `DoctorCard`: Individual doctor information
- `AppointmentBooking`: Book appointments
- `AIAgentAnalyzer`: Analyze symptoms to recommend specialists
- `ABDMService`: Query ABDM for doctor information

**Data Model:**
```typescript
interface Doctor {
  id: string;
  name: string;
  specialization: string[];
  hospitalName: string;
  hospitalType: 'government' | 'private';
  distance: number; // in km
  consultationFee: number;
  availability: Availability[];
  rating: number;
  location: {
    latitude: number;
    longitude: number;
    address: string;
  };
}

interface Availability {
  day: string;
  slots: TimeSlot[];
}

interface TimeSlot {
  startTime: string;
  endTime: string;
  available: boolean;
}
```

#### 2.1.6 Medicine Search Module
**Responsibility:** Find affordable medicine alternatives

**Components:**
- `MedicineSearch`: Search interface
- `MedicineList`: Display results with prices
- `MedicineDetail`: Detailed medicine information
- `PharmacyLocator`: Find nearby Jan Aushadhi Kendras and pharmacies
- `PriceComparator`: Compare prices across pharmacies

**Data Model:**
```typescript
interface MedicineResult {
  id: string;
  name: string;
  activeIngredient: string;
  type: 'generic' | 'branded';
  price: number;
  pharmacy: Pharmacy;
  chemicalComposition: string;
  sideEffects: string[];
  dosageInfo: string;
}

interface Pharmacy {
  id: string;
  name: string;
  type: 'jan_aushadhi' | 'regular';
  distance: number;
  location: {
    latitude: number;
    longitude: number;
    address: string;
  };
  contactNumber: string;
}
```

#### 2.1.7 Emergency Module
**Responsibility:** Handle emergency situations

**Components:**
- `EmergencyModeToggle`: Manual emergency activation
- `EmergencyThemeProvider`: Red theme for emergency mode
- `EmergencyAnalyzer`: AI-based emergency detection
- `HospitalBedFinder`: Find hospitals with available beds
- `EmergencyServiceConnector`: Contact emergency services
- `FacilityMatcher`: Match required facilities to hospitals

**Data Model:**
```typescript
interface EmergencyRequest {
  id: string;
  patientId: string;
  symptoms: string;
  isEmergency: boolean;
  requiredFacilities: string[];
  timestamp: Date;
}

interface Hospital {
  id: string;
  name: string;
  type: 'government' | 'private';
  distance: number;
  availableBeds: number;
  facilities: Facility[];
  emergencyContact: string;
  location: {
    latitude: number;
    longitude: number;
    address: string;
  };
}

interface Facility {
  name: string;
  available: boolean;
  count: number;
}
```

### 2.2 Backend Services

#### 2.2.1 API Gateway Service
**Responsibility:** Route requests and handle authentication

**Endpoints:**
- `POST /api/patients` - Create patient profile
- `PUT /api/patients/:id` - Update patient profile
- `POST /api/symptoms` - Submit symptoms
- `POST /api/prescriptions` - Upload prescription
- `POST /api/prescriptions/ocr` - Process prescription image
- `GET /api/doctors` - Search doctors
- `POST /api/appointments` - Book appointment
- `GET /api/medicines` - Search medicines
- `GET /api/pharmacies` - Find pharmacies
- `POST /api/emergency` - Initiate emergency request
- `GET /api/hospitals` - Find hospitals with beds

#### 2.2.2 Cache Service (Amazon ElastiCache)
**Responsibility:** Cache API responses using Amazon ElastiCache (Redis)

**Implementation:**
- Use ElastiCache Redis cluster 
- Within code, we connect to the Redis cluster
- On fetching an API response from the User, the ABDM or the UHI API, we will:
  - Cache it in Redis if it is a fresh value (check -> fetch from API -> store -> return)
  - Fetch it directly from the cache if available (check -> fetch from cache -> return)

**Cache Keys Strategy:**

- Cache keys will be named according to the source/entity associated with the data. Example:
  - `doctors:{lat}:{lng}:{specialization}` - Doctor search results
  - `medicines:{activeIngredient}:{lat}:{lng}` - Medicine search results
  - `hospitals:{lat}:{lng}:{facilities}` - Hospital search results
  - `abdm:response:{endpoint}:{hash}` - ABDM API responses
  - `uhi:response:{endpoint}:{hash}` - UHI API responses
  - `translation:{hash}:{targetLang}` - Translation cache


#### 2.2.3 Data Cleanup Service (AWS-Native)
**Responsibility:** Automatically delete data after 24 hours using AWS services

**DynamoDB TTL Configuration:**

To automatically delete the data, we can use:
- Time to live (TTL) property of DynamoDB
- Expiration property configuration of S3 bucker

#### 2.2.4 ABDM Integration Service
**Responsibility:** Interface with ABDM APIs

**Methods:**
- `searchDoctors(location, specialization)` - Find doctors
- `searchHospitals(location, facilities)` - Find hospitals
- `checkBedAvailability(hospitalId)` - Check bed availability
- `bookAppointment(doctorId, patientId, slot)` - Book appointment

#### 2.2.5 UHI Integration Service
**Responsibility:** Interface with UHI APIs

**Methods:**
- `searchHealthcareProviders(location, type)` - Find providers
- `getServiceAvailability(providerId)` - Check availability
- `initiateEmergencyRequest(location, requirements)` - Emergency services

#### 2.2.6 OCR Service (Amazon Textract)
**Responsibility:** Extract text from prescription images using Amazon Textract

**Implementation:**
```typescript
import { TextractClient, DetectDocumentTextCommand, AnalyzeDocumentCommand } from "@aws-sdk/client-textract";

class TextractOCRService {
  private textract: TextractClient;

  constructor() {
    this.textract = new TextractClient({ region: "us-east-1" });
  }

  async extractPrescription(s3Key: string): Promise<PrescriptionData> {
    // Use DetectDocumentText for simple text extraction
    const command = new DetectDocumentTextCommand({
      Document: {
        S3Object: {
          Bucket: process.env.S3_BUCKET_NAME,
          Name: s3Key
        }
      }
    });
    
    const result = await this.textract.send(command);
    const extractedText = this.concatenateBlocks(result.Blocks);
    
    // Use Bedrock Agent to parse medicine names from extracted text
    const medicines = await bedrockAgent.extractMedicines(extractedText);
    
    return {
      rawText: extractedText,
      medicines: medicines,
      confidence: this.calculateConfidence(result.Blocks)
    };
  }

  async extractPrescriptionAdvanced(s3Key: string): Promise<PrescriptionData> {
    // Use AnalyzeDocument for structured data extraction (tables, forms)
    const command = new AnalyzeDocumentCommand({
      Document: {
        S3Object: {
          Bucket: process.env.S3_BUCKET_NAME,
          Name: s3Key
        }
      },
      FeatureTypes: ["TABLES", "FORMS"]
    });
    
    const result = await this.textract.send(command);
    return this.parseStructuredPrescription(result);
  }

  private concatenateBlocks(blocks: any[]): string {
    if (!blocks) return '';
    
    return blocks
      .filter(block => block.BlockType === 'LINE')
      .map(block => block.Text)
      .join('\n');
  }

  private calculateConfidence(blocks: any[]): number {
    if (!blocks || blocks.length === 0) return 0;
    
    const confidences = blocks
      .filter(block => block.Confidence)
      .map(block => block.Confidence);
    
    return confidences.reduce((a, b) => a + b, 0) / confidences.length;
  }

  private parseStructuredPrescription(result: any): PrescriptionData {
    // Parse tables and forms from prescription
    // Extract: Medicine name, dosage, frequency, duration
    const medicines = [];
    
    // Process TABLES feature type
    if (result.Blocks) {
      // Implementation to extract structured data
    }
    
    return {
      rawText: this.concatenateBlocks(result.Blocks),
      medicines: medicines,
      confidence: this.calculateConfidence(result.Blocks)
    };
  }
}

interface PrescriptionData {
  rawText: string;
  medicines: Medicine[];
  confidence: number;
}
```

**S3 Pre-Signed URL for Uploads:**
```typescript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

async function generateUploadUrl(fileName: string): Promise<string> {
  const s3Client = new S3Client({ region: "us-east-1" });
  const key = `prescriptions/${Date.now()}-${fileName}`;
  
  const command = new PutObjectCommand({
    Bucket: process.env.S3_BUCKET_NAME,
    Key: key,
    ContentType: 'image/jpeg',
    // Metadata for automatic deletion
    Metadata: {
      'upload-time': Date.now().toString()
    }
  });
  
  // URL expires in 15 minutes
  const uploadUrl = await getSignedUrl(s3Client, command, { expiresIn: 900 });
  
  return uploadUrl;
}
```

#### 2.2.7 Amazon Bedrock Agent Service
**Responsibility:** Orchestrate AI tasks using Amazon Bedrock Agents

**Amazon Bedrock Agents Configuration:**

**Agent 1: Symptom Analysis Agent**
- Foundation Model: Anthropic Claude 3 Sonnet
- Action Groups:
  - `analyzeSymptoms`: Analyze patient symptoms and determine severity
  - `detectEmergency`: Identify if symptoms indicate emergency
  - `recommendSpecialist`: Suggest appropriate medical specialist
- Knowledge Base: Medical symptom database (optional)
- Guardrails: Medical content filtering

**Agent 2: Medicine Extraction Agent**
- Foundation Model: Anthropic Claude 3 Sonnet
- Action Groups:
  - `extractMedicines`: Parse prescription text for medicine names
  - `identifyActiveIngredients`: Map medicines to active ingredients
  - `findGenericAlternatives`: Suggest generic equivalents
- Knowledge Base: Indian Pharmacopoeia, Jan Aushadhi medicine list

**Agent 3: Translation & Communication Agent**
- Foundation Model: Anthropic Claude 3 Sonnet
- Action Groups:
  - `translateToEnglish`: Translate Indian languages to English
  - `translateFromEnglish`: Translate English to Indian languages
  - `simplifyMedicalTerms`: Convert medical jargon to simple language
- Integration: Amazon Translate for additional language support

**Agent 4: Emergency Coordinator Agent**
- Foundation Model: Anthropic Claude 3 Sonnet
- Action Groups:
  - `assessEmergency`: Rapid emergency assessment
  - `determineFacilities`: Identify required hospital facilities
  - `prioritizeHospitals`: Rank hospitals by suitability
- Optimized for: Low-latency responses (<5 seconds)

## 3. Data Flow Diagrams

### 3.1 Doctor Search Flow

```
Patient → Enter Symptoms → AI Agent Analysis → Translate to English
                                    ↓
                          Determine Specialist Type
                                    ↓
                          Query Location Service
                                    ↓
                    Check Cache for Doctor Results
                                    ↓
                          Cache Miss? Query ABDM
                                    ↓
                          Filter by Specialization
                                    ↓
                          Sort by Distance & Type
                                    ↓
                    Prioritize Government Facilities
                                    ↓
                          Cache Results (24h TTL)
                                    ↓
                          Return to Patient
```

### 3.2 Medicine Search Flow

```
Patient → Upload Prescription → OCR Processing → Extract Text
                                    ↓
                          AI Agent Extraction
                                    ↓
                          Identify Medicine Names
                                    ↓
                          Map to Active Ingredients
                                    ↓
                          Query Location Service
                                    ↓
                    Check Cache for Medicine Results
                                    ↓
                    Search Jan Aushadhi Kendras First
                                    ↓
                    No Results? Search Regular Pharmacies
                                    ↓
                          Sort by Price (Ascending)
                                    ↓
                          Cache Results (24h TTL)
                                    ↓
                          Return to Patient
```

### 3.3 Emergency Flow

```
Patient → Activate Emergency Mode → Enter Symptoms → AI Agent Analysis
                                                            ↓
                                                  Confirm Emergency?
                                                            ↓
                                                    Yes → Switch to Red Theme
                                                            ↓
                                                  Determine Required Facilities
                                                            ↓
                                                  Query Location Service
                                                            ↓
                                                  Query ABDM + UHI (Parallel)
                                                            ↓
                                                  Filter by Facilities
                                                            ↓
                                                  Check Bed Availability
                                                            ↓
                                                  Sort by Distance
                                                            ↓
                                                  Contact Emergency Services
                                                            ↓
                                                  Return Results (< 30s)
```

## 4. Performance Optimization

### 4.1 Low-Bandwidth Optimization

**Strategies:**
1. **Data Compression:** Gzip all API responses
2. **Image Optimization:** Compress prescription images before upload
3. **Lazy Loading:** Load UI components on demand
4. **Pagination:** Limit results to 10-20 items per page
5. **Minimal Payloads:** Return only necessary fields
6. **Progressive Enhancement:** Basic functionality works on 2G

**Implementation:**
```typescript
// API Response Compression
app.use(compression({
  level: 6,
  threshold: 1024 // Only compress responses > 1KB
}));

// Image Compression
const compressImage = async (image: File): Promise<Blob> => {
  return await imageCompression(image, {
    maxSizeMB: 0.5,
    maxWidthOrHeight: 1920,
    useWebWorker: true
  });
};
```

### 4.2 Caching Strategy

**Multi-Level Caching:**
1. **Client-Side Cache:** IndexedDB for offline access
2. **CDN Cache:** Static assets (images, scripts)
3. **Server-Side Cache:** Redis for API responses
4. **API Response Cache:** 24-hour TTL

**Cache Invalidation:**
- Time-based: Automatic expiry after 24 hours
- Event-based: Invalidate on data updates
- Manual: Admin can force cache refresh

### 4.3 Emergency Mode Optimization

**Priority Optimizations:**
1. **Parallel API Calls:** Query ABDM and UHI simultaneously
2. **Reduced Validation:** Skip non-critical validations
3. **Prioritized Network:** Mark emergency requests as high priority
4. **Preloaded Data:** Cache common emergency facilities
5. **Timeout Handling:** 30-second hard timeout

```typescript
const emergencySearch = async (symptoms: string, location: Location) => {
  const timeout = 30000; // 30 seconds
  
  const [abdmResults, uhiResults] = await Promise.race([
    Promise.all([
      abdmService.searchHospitals(location, facilities),
      uhiService.searchHealthcareProviders(location, 'emergency')
    ]),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), timeout)
    )
  ]);
  
  return mergeAndSortResults(abdmResults, uhiResults);
};
```

## 5. Security and Privacy

### 5.1 Data Encryption

**In Transit:**
- TLS 1.3 for all API communications
- Certificate pinning in mobile app

**At Rest:**
- AES-256 encryption for database
- Encrypted S3 buckets for prescription images

### 5.2 Data Retention Policy

**Automatic Deletion:**
```typescript
interface DataRetentionPolicy {
  patientProfiles: '24 hours';
  prescriptions: '24 hours';
  prescriptionImages: '24 hours';
  locations: '24 hours';
  symptoms: '24 hours';
  searchHistory: '24 hours';
  cacheData: '24 hours';
}
```

**Implementation:**
- Database triggers for automatic deletion
- Scheduled cleanup jobs (hourly)
- Audit logs for compliance

### 5.3 API Security (No User Authentication)

**Public Access Design:**
- No user authentication required (removes barrier to healthcare access)
- Focus on rate limiting and abuse prevention
- Session-less architecture for simplicity

**AWS API Gateway Security:**
```typescript
// API Gateway configuration
const apiGatewayConfig = {
  throttle: {
    rateLimit: 100,  // requests per second per IP
    burstLimit: 200  // maximum concurrent requests
  },
  quota: {
    limit: 10000,    // requests per day per IP
    period: 'DAY'
  }
};
```

**AWS WAF (Web Application Firewall):**
- Rate-based rules: Block IPs exceeding 1000 requests/5 minutes
- Geo-blocking: Restrict to India region (optional)
- SQL injection protection
- XSS protection
- Bot detection and mitigation

**CloudWatch Monitoring:**
```typescript
// Alert on suspicious patterns
const cloudWatchAlarm = {
  MetricName: 'Count',
  Namespace: 'AWS/ApiGateway',
  Statistic: 'Sum',
  Period: 300, // 5 minutes
  EvaluationPeriods: 1,
  Threshold: 5000, // Alert if > 5000 requests in 5 min from single IP
  ComparisonOperator: 'GreaterThanThreshold'
};
```

**S3 Security:**
- Pre-signed URLs with 15-minute expiry for prescription uploads
- No public read access
- Server-side encryption (SSE-S3)
- Automatic deletion after 24 hours via lifecycle policy

**DynamoDB Security:**
- No direct public access (only via Lambda)
- Encryption at rest enabled
- Point-in-time recovery enabled
- Automatic TTL-based deletion

**Lambda Security:**
- Minimal IAM permissions (principle of least privilege)
- VPC configuration for ElastiCache access
- Environment variables in AWS Secrets Manager
- X-Ray tracing for debugging

## 6. Error Handling

### 6.1 Network Errors

**Strategies:**
1. **Retry Logic:** Exponential backoff for failed requests
2. **Offline Queue:** Queue requests when offline
3. **Graceful Degradation:** Show cached data when API fails
4. **User Feedback:** Clear error messages

```typescript
const apiCall = async (endpoint: string, retries = 3): Promise<any> => {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(endpoint);
    } catch (error) {
      if (i === retries - 1) throw error;
      await sleep(Math.pow(2, i) * 1000); // Exponential backoff
    }
  }
};
```

### 6.2 AI Agent Errors

**Fallback Strategies:**
1. **Translation Failure:** Show original text with warning
2. **Analysis Failure:** Prompt user to select specialist manually
3. **Emergency Detection Failure:** Default to emergency mode for safety
4. **TTS/STT Failure:** Fall back to text input/output

### 6.3 External Service Errors

**ABDM/UHI API Failures:**
- Return cached results if available
- Show partial results from available APIs
- Notify user of limited data
- Provide manual search option

## 7. Testing Strategy

### 7.1 Unit Testing

**Coverage Requirements:**
- Minimum 80% code coverage
- All business logic components
- All utility functions
- All data transformations

**Framework:** Jest + React Native Testing Library

### 7.2 Integration Testing

**Test Scenarios:**
- ABDM API integration
- UHI API integration
- OCR service integration
- AI Agent integration
- Cache service integration

**Framework:** Jest + Supertest

### 7.3 End-to-End Testing

**Critical User Flows:**
1. Complete doctor search and booking
2. Complete medicine search
3. Emergency mode activation and hospital search
4. Prescription upload and processing
5. Multilingual symptom input

**Framework:** Detox (React Native)

### 7.4 Performance Testing

**Metrics:**
- API response time < 2 seconds (normal mode)
- Emergency search < 30 seconds
- App launch time < 3 seconds
- Search results render < 1 second

**Tools:** Lighthouse, React Native Performance Monitor

### 7.5 Accessibility Testing

**Requirements:**
- Screen reader support
- High contrast mode
- Large text support
- Voice navigation

## 8. Deployment Strategy

### 8.1 Mobile App Deployment

**Platforms:**
- iOS: App Store
- Android: Google Play Store

**Release Process:**
1. Beta testing with 100 users
2. Staged rollout (10% → 50% → 100%)
3. Monitor crash reports and feedback
4. Hotfix capability for critical issues

### 8.2 Backend Deployment (AWS Serverless)

**Infrastructure (Serverless):**
- AWS Lambda functions (auto-scaling, pay-per-use)
- API Gateway (managed, auto-scaling)
- DynamoDB (serverless database)
- ElastiCache (managed Redis)
- S3 (object storage)
- No servers to manage!

**Infrastructure as Code (AWS SAM):**
```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  SahayakAPI:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
      Cors:
        AllowOrigin: "'*'"
      ThrottleSettings:
        RateLimit: 100
        BurstLimit: 200

  SymptomAnalysisFunction:
    Type: AWS::Serverless::Function
    Properties:
      Runtime: nodejs18.x
      Handler: symptom-analysis.handler
      Timeout: 30
      MemorySize: 512
      Environment:
        Variables:
          BEDROCK_AGENT_ID: !Ref SymptomAgentId
          DYNAMODB_TABLE: !Ref PatientDataTable
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref PatientDataTable
        - Statement:
          - Effect: Allow
            Action:
              - bedrock:InvokeAgent
            Resource: '*'

  PatientDataTable:
    Type: AWS::DynamoDB::Table
    Properties:
      BillingMode: PAY_PER_REQUEST
      TimeToLiveSpecification:
        Enabled: true
        AttributeName: ttl
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
```

**CI/CD Pipeline (AWS CodePipeline):**
```yaml
# buildspec.yml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 18
  pre_build:
    commands:
      - npm install
      - npm run test
  build:
    commands:
      - sam build
      - sam package --output-template-file packaged.yaml
  post_build:
    commands:
      - sam deploy --template-file packaged.yaml --stack-name sahayak-prod
```

**Deployment Stages:**
1. Code commit → GitHub/CodeCommit
2. CodeBuild runs tests
3. SAM builds and packages Lambda functions
4. Deploy to staging (automatic)
5. Run integration tests
6. Deploy to production (manual approval)
7. CloudWatch monitors metrics

**Multi-Region Deployment (Optional):**
- Primary: ap-south-1 (Mumbai)
- Backup: ap-southeast-1 (Singapore)
- Route53 health checks for failover
- DynamoDB Global Tables for replication

### 8.3 Monitoring and Logging (AWS-Native)

**CloudWatch Metrics:**
- Lambda invocations, duration, errors
- API Gateway requests, latency, 4xx/5xx errors
- DynamoDB read/write capacity, throttles
- ElastiCache CPU, memory, cache hits
- Bedrock Agent invocations, latency

**CloudWatch Dashboards:**
```typescript
const dashboard = {
  widgets: [
    {
      type: 'metric',
      properties: {
        metrics: [
          ['AWS/Lambda', 'Duration', { stat: 'Average' }],
          ['AWS/ApiGateway', 'Latency', { stat: 'p99' }],
          ['AWS/DynamoDB', 'UserErrors', { stat: 'Sum' }]
        ],
        period: 300,
        stat: 'Average',
        region: 'ap-south-1',
        title: 'Sahayak Performance Metrics'
      }
    }
  ]
};
```

**CloudWatch Alarms:**
- Lambda errors > 10 in 5 minutes → SNS notification
- API Gateway 5xx errors > 50 in 5 minutes → SNS notification
- DynamoDB throttling → Auto-scale capacity
- Emergency endpoint latency > 5 seconds → SNS notification

**AWS X-Ray Tracing:**
- End-to-end request tracing
- Service map visualization
- Performance bottleneck identification
- Error analysis

**CloudWatch Logs Insights:**
```sql
-- Find slow Bedrock Agent calls
fields @timestamp, @message
| filter @message like /Bedrock/
| filter duration > 5000
| sort @timestamp desc
| limit 20
```

**Cost Monitoring:**
- AWS Cost Explorer for daily cost tracking
- Budget alerts for unexpected spending
- Lambda cost optimization (right-sizing memory)
- S3 lifecycle policies to reduce storage costs

## 9. Future Enhancements

### 9.1 Phase 2 Features
- Telemedicine video consultations
- Medicine delivery integration
- Health records management (with user consent)
- Insurance claim assistance
- Medication reminders

### 9.2 Technical Improvements
- Progressive Web App (PWA) version
- Offline-first architecture
- AI model optimization for edge devices
- Blockchain for health records
- Advanced analytics and insights

## 10. Correctness Properties

### 10.1 Data Integrity Properties

**Property 1.1: Data Expiration**
- **Description:** All patient data must be automatically deleted after exactly 24 hours
- **Validation:** For any data record with timestamp T, the record must not exist at T + 24 hours + 1 minute
- **Test Strategy:** Property-based test with random timestamps

**Property 1.2: Location Accuracy**
- **Description:** GPS coordinates must be valid (latitude: -90 to 90, longitude: -180 to 180)
- **Validation:** All location data must pass coordinate validation
- **Test Strategy:** Property-based test with boundary values

### 10.2 Search Result Properties

**Property 2.1: Distance Sorting**
- **Description:** Search results must be sorted by distance in ascending order
- **Validation:** For any two consecutive results R1 and R2, distance(R1) ≤ distance(R2)
- **Test Strategy:** Property-based test with random locations

**Property 2.2: Government Priority**
- **Description:** Government facilities must appear before private facilities at equal distances
- **Validation:** For results at same distance, government type comes first
- **Test Strategy:** Property-based test with mixed facility types

**Property 2.3: Price Sorting**
- **Description:** Medicine results must be sorted by price in ascending order
- **Validation:** For any two consecutive medicines M1 and M2, price(M1) ≤ price(M2)
- **Test Strategy:** Property-based test with random prices

### 10.3 Emergency Mode Properties

**Property 3.1: Emergency Response Time**
- **Description:** Emergency search must complete within 30 seconds
- **Validation:** Time from emergency activation to results display ≤ 30 seconds
- **Test Strategy:** Performance test with timeout monitoring

**Property 3.2: Facility Matching**
- **Description:** Emergency results must only include hospitals with required facilities
- **Validation:** All returned hospitals must have facilities matching emergency requirements
- **Test Strategy:** Property-based test with various facility combinations

### 10.4 Cache Properties

**Property 4.1: Cache Consistency**
- **Description:** Cached data must match source data at time of caching
- **Validation:** Cache entry equals API response at cache time
- **Test Strategy:** Property-based test comparing cache and source

**Property 4.2: Cache Expiration**
- **Description:** Cache entries must expire after 24 hours
- **Validation:** Cache entry with timestamp T must not be retrievable at T + 24 hours + 1 minute
- **Test Strategy:** Property-based test with time manipulation

### 10.5 Translation Properties

**Property 5.1: Translation Reversibility**
- **Description:** Translating text from language A to B and back to A should preserve meaning
- **Validation:** Semantic similarity score > 0.8 after round-trip translation
- **Test Strategy:** Property-based test with sample medical terms

**Property 5.2: Language Support**
- **Description:** All supported Indian languages must be translatable to English
- **Validation:** Translation succeeds for all supported language codes
- **Test Strategy:** Property-based test with language code enumeration

### 10.6 Prescription Processing Properties

**Property 6.1: OCR Accuracy**
- **Description:** OCR must extract text with >90% accuracy for clear images
- **Validation:** Extracted text matches ground truth with >90% similarity
- **Test Strategy:** Property-based test with sample prescriptions

**Property 6.2: Medicine Extraction**
- **Description:** All medicine names in prescription must be extracted
- **Validation:** Extracted medicines list contains all medicines from prescription
- **Test Strategy:** Property-based test with known prescriptions

## 11. Implementation Phases (2-Day Hackathon with AWS)

### Day 1: Morning (Hours 0-4) - AWS Foundation & Core Setup
**Goal:** Set up AWS infrastructure and basic patient flow

**Tasks:**
- Initialize React Native project with Expo and AWS Amplify
- Set up AWS account and configure AWS CLI
- Create AWS SAM template for Lambda functions
- Deploy DynamoDB table with TTL enabled
- Create patient profile form (name, age, gender)
- Implement basic location capture (GPS or manual)
- Set up mock ABDM/UHI API responses for testing
- Deploy first Lambda function (patient data storage)

**AWS Services Used:**
- AWS Amplify (mobile app setup)
- AWS Lambda (serverless functions)
- Amazon DynamoDB (data storage with TTL)
- AWS SAM (infrastructure as code)

**Deliverable:** App can capture patient details and location, store in DynamoDB

### Day 1: Afternoon (Hours 4-8) - Bedrock Agents & Doctor Search
**Goal:** Enable symptom entry and doctor recommendations using Bedrock

**Tasks:**
- Build symptom input form (text only, English)
- Create Amazon Bedrock Agent for symptom analysis
- Configure Claude 3 Sonnet model in Bedrock
- Implement doctor search Lambda function with mock data
- Create doctor list UI with distance sorting
- Add government facility prioritization logic
- Set up ElastiCache Redis for caching (or use DynamoDB for simplicity)

**AWS Services Used:**
- Amazon Bedrock Agents (symptom analysis)
- Anthropic Claude 3 Sonnet (foundation model)
- AWS Lambda (doctor search logic)
- Amazon ElastiCache or DynamoDB (caching)

**Deliverable:** Users can enter symptoms and see nearby doctors via Bedrock Agent

### Day 1: Evening (Hours 8-12) - Medicine Search with Bedrock
**Goal:** Basic prescription processing and medicine search

**Tasks:**
- Create prescription text input (skip image upload for MVP)
- Create Bedrock Agent for medicine extraction
- Implement medicine name extraction using Bedrock
- Build medicine search Lambda function with mock pharmacy data
- Create medicine list UI with price sorting
- Add Jan Aushadhi Kendra prioritization
- Implement medicine detail view
- Set up API Gateway endpoints

**AWS Services Used:**
- Amazon Bedrock Agents (medicine extraction)
- AWS Lambda (medicine search)
- AWS API Gateway (REST endpoints)

**Deliverable:** Users can enter prescription and find affordable medicines

### Day 2: Morning (Hours 12-16) - Emergency Mode & Bedrock Optimization
**Goal:** Add emergency features with fast Bedrock responses

**Tasks:**
- Implement emergency mode toggle
- Create Bedrock Agent for emergency detection
- Optimize Bedrock Agent for <5 second response time
- Create red theme for emergency mode
- Build hospital bed search with facility filtering
- Add emergency service contact simulation
- Implement parallel ABDM/UHI API calls
- Add CloudWatch logging for monitoring

**AWS Services Used:**
- Amazon Bedrock Agents (emergency detection)
- AWS Lambda (hospital search)
- Amazon CloudWatch (monitoring)

**Deliverable:** Emergency mode functional with hospital search (<30 seconds)

### Day 2: Afternoon (Hours 16-20) - Multilingual with AWS AI Services
**Goal:** Add language support using AWS Translate, Polly, Transcribe

**Tasks:**
- Integrate Amazon Translate (2-3 Indian languages: Hindi, Tamil)
- Add language selector to UI
- Implement Amazon Transcribe for STT (Hindi voice input)
- Implement Amazon Polly for TTS (optional, if time permits)
- Connect to real ABDM/UHI APIs (if available) or refined mocks
- Add error handling and loading states
- Implement basic offline support (AsyncStorage cache)
- Set up CloudWatch alarms for errors

**AWS Services Used:**
- Amazon Translate (multilingual support)
- Amazon Transcribe (speech-to-text)
- Amazon Polly (text-to-speech, optional)
- Amazon CloudWatch (error monitoring)

**Deliverable:** Multilingual support and API integration complete

### Day 2: Evening (Hours 20-24) - Testing, Demo Prep & AWS Deployment
**Goal:** Finalize MVP and deploy to AWS

**Tasks:**
- End-to-end testing of all user flows
- Fix critical bugs
- Deploy all Lambda functions to production
- Configure API Gateway rate limiting
- Set up CloudWatch dashboard for demo
- Add demo data for presentation
- Create demo script covering all features
- Record demo video (backup for live demo)
- Prepare pitch deck highlighting AWS services and impact
- Deploy mobile app to Expo for testing

**AWS Services Used:**
- AWS SAM (deployment)
- AWS API Gateway (rate limiting)
- Amazon CloudWatch (dashboard)
- AWS Amplify (mobile app hosting, optional)

**Deliverable:** Working MVP deployed on AWS, ready for demo

### MVP Scope Adjustments for Hackathon (AWS-Optimized)

**Included (Must-Have):**
- Patient profile and symptom input (text)
- Amazon Bedrock Agents for AI (symptom analysis, medicine extraction, emergency detection)
- Doctor search with distance sorting
- Medicine search with price comparison
- Emergency mode with hospital search
- Amazon Translate for 2-3 Indian languages
- Government facility prioritization
- DynamoDB with TTL for data storage
- Lambda functions for all backend logic
- API Gateway for REST endpoints

**Simplified (Good-Enough):**
- Mock ABDM/UHI data (real API if time permits)
- DynamoDB for caching (instead of ElastiCache)
- Text-only prescription input (no Textract OCR)
- Amazon Transcribe for STT (skip Polly TTS if time-constrained)
- Manual CloudWatch monitoring (no automated alarms)
- No authentication (public API with rate limiting)

**Deferred (Post-Hackathon):**
- Amazon Textract for prescription image OCR
- Appointment booking functionality
- Amazon Polly for TTS output
- Comprehensive offline mode
- ElastiCache Redis for advanced caching
- Automated CloudWatch alarms and dashboards
- AWS WAF for DDoS protection
- Multi-region deployment
- Production-grade security hardening

### Hackathon Success Metrics

**Technical:**
- All 3 core flows working (doctor search, medicine search, emergency)
- Amazon Bedrock Agents successfully analyze symptoms and extract medicines
- Amazon Translate works for at least 2 Indian languages (Hindi + 1 more)
- Emergency mode responds within 30 seconds
- All data stored in DynamoDB with 24-hour TTL
- API Gateway rate limiting prevents abuse

**Demo:**
- Clear demonstration of problem and solution
- Live demo of all core features
- Showcase AWS services integration (Bedrock, Translate, Lambda, DynamoDB)
- Compelling story about impact on rural healthcare
- Evidence of government facility prioritization and cost savings
- CloudWatch dashboard showing real-time metrics

**Technical:**
- All 3 core flows working (doctor search, medicine search, emergency)
- AI Agent successfully analyzes symptoms
- Translation works for at least 2 Indian languages
- Emergency mode responds within 30 seconds

**Demo:**
- Clear demonstration of problem and solution
- Live demo of all core features
- Compelling story about impact on rural healthcare
- Evidence of government facility prioritization and cost savings
