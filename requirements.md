# Requirements Document: Sahayak - AI Health Agent for Rural India

## Introduction

Sahayak is a multi-modal, agentic AI application designed for the Ayushman Bharat Digital Mission (ABDM) to bridge healthcare access gaps for rural patients in India.

In India, healthcare does not fail because of a lack of doctors or medicines — it fails because access and affordability break down before treatment even begins.

Every year, millions of patients delay care because they cannot explain symptoms in English, navigate digital health apps, or identify the right specialist. Even after reaching a doctor, many are pushed into buying expensive medicines simply because they cannot read prescriptions or ask for generic alternatives — despite government schemes like PMJAY and Jan Aushadhi existing precisely to prevent this.

Sahayak is a multi-modal, agentic AI built on the Ayushman Bharat Digital Mission (ABDM) that fixes both failures at once.
It enables any Indian — regardless of literacy or language — to find free or low-cost government healthcare using voice and reduce medicine costs by up to 80% using vision-based generic discovery.

By intelligently connecting patients to UHI-enabled government hospitals and Jan Aushadhi Kendras, Sahayak transforms healthcare from a confusing maze into a guided, voice-first public service.

 The system provides voice-first doctor discovery and vision-based medicine finding capabilities, targeting the Next Billion Users who are primarily non-English speakers in rural areas.

**Mission Statement:** "Breaking Language Barriers for Care, Breaking Cost Barriers for Cures."

The system enables users to:
1. Speak symptoms in their native language to discover the right doctors via the UHI(Unified Health Interface) network, explicitly highlighting nearby government hospitals offering free treatment under PMJAY(Pradhan Mantri Jan Arogya Yojana).
It also identifies potentially dangerous symptoms early and clearly alerts the user to seek urgent hospital care instead of delaying treatment.

2. Upload prescription photos to find the cheapest generic medicine alternatives at Jan Aushadhi Kendras, reducing post-treatment expenses.

## Glossary

- **Sahayak_System**: The complete AI health agent application including voice, vision, and discovery capabilities
- **Voice_Triage_Module**: Component that processes audio input, performs speech-to-text, translation, and specialist inference
- **Vision_Module**: Component that processes prescription images and extracts medicine information
- **UHI_Discovery_Service**: Service that searches the ABDM/UHI network for healthcare providers
- **Medicine_Price_Service**: Service that compares medicine prices between brands and Jan Aushadhi Kendras
- **Bedrock_Agent**: Amazon Bedrock agent orchestrating the agentic workflow with action groups
- **STT_Service**: Speech-to-Text service using Amazon Transcribe
- **TTS_Service**: Text-to-Speech service using Amazon Polly
- **OCR_Service**: Optical Character Recognition service using Amazon Textract
- **ABDM**: Ayushman Bharat Digital Mission - India's national digital health ecosystem
- **UHI**: Unified Health Interface - network protocol for healthcare provider discovery
- **PMJAY**: Pradhan Mantri Jan Arogya Yojana - government health insurance scheme
- **Jan_Aushadhi_Kendra**: Government-run generic medicine stores
- **Beckn_Protocol**: Open protocol for decentralized commerce, used by UHI
- **Government_Hospital**: Healthcare facility providing free services under PMJAY
- **Private_Clinic**: Healthcare facility charging fees for services
- **Generic_Medicine**: Medicine identified by its active pharmaceutical ingredient (salt name)
- **Brand_Medicine**: Medicine identified by its commercial brand name
- **Specialist_Type**: Medical specialty category (e.g., Cardiologist, Dermatologist, General Physician)
- **Symptom_Description**: User-provided description of health concerns in natural language
- **Native_Language**: User's preferred language (Hindi, Tamil, or other Indian languages)
- **Emergency_Mode**: UI mode prioritizing urgent care access (red theme)
- **Savings_Mode**: UI mode prioritizing cost-effective medicine options (green theme)

## Requirements

### Requirement 1: Voice Input Processing

**User Story:** As a rural patient who cannot read or write, I want to speak my symptoms in my native language, so that I can communicate my health concerns without language barriers.

#### Acceptance Criteria

1. WHEN a user provides audio input in Hindi or Tamil or any language, THE STT_Service SHALL transcribe the audio to text within 2 seconds
2. WHEN the transcribed text is in a native language, THE Voice_Triage_Module SHALL translate it to English for processing
3. WHEN audio quality is poor or speech is unclear, THE Sahayak_System SHALL request the user to repeat their input
4. WHEN background noise exceeds acceptable thresholds, THE STT_Service SHALL filter noise before transcription
5. THE Voice_Triage_Module SHALL support audio input in multiple languages

### Requirement 2: Symptom Analysis and Specialist Inference

**User Story:** As a patient with limited medical knowledge, I want the system to understand my symptoms and recommend the right type of doctor, so that I can get appropriate medical care.

#### Acceptance Criteria

1. WHEN a Symptom_Description is provided, THE Bedrock_Agent SHALL analyze the symptoms and infer the appropriate Specialist_Type
2. WHEN symptoms indicate potential emergency conditions, THE Bedrock_Agent SHALL prioritize Emergency_Mode and flag urgent care requirements
3. WHEN symptoms are ambiguous or insufficient, THE Bedrock_Agent SHALL ask clarifying questions in the user's Native_Language
4. THE Bedrock_Agent SHALL classify symptoms into at least 10 common Specialist_Type categories (General Physician, Cardiologist, Dermatologist, Pediatrician, Gynecologist, Orthopedic, ENT, Ophthalmologist, Dentist, Psychiatrist)
5. WHEN inferring specialist type, THE Bedrock_Agent SHALL prioritize patient safety by over-estimating urgency rather than under-estimating

### Requirement 3: Healthcare Provider Discovery via UHI Network

**User Story:** As a rural patient seeking affordable healthcare, I want to find nearby doctors who accept government insurance, so that I can access free or low-cost medical care.

#### Acceptance Criteria

1. WHEN a Specialist_Type and user location are provided, THE UHI_Discovery_Service SHALL search the ABDM/UHI network for matching healthcare providers within 50 kilometers
2. WHEN displaying search results, THE UHI_Discovery_Service SHALL prioritize Government_Hospital entries over Private_Clinic entries
3. WHEN a Government_Hospital is available, THE Sahayak_System SHALL clearly indicate that services are free under PMJAY
4. WHEN only Private_Clinic options are available, THE Sahayak_System SHALL display consultation fees prominently
5. THE UHI_Discovery_Service SHALL return results sorted by distance from user location, with Government_Hospital entries appearing before Private_Clinic entries at the same distance
6. WHEN no providers are found within 50 kilometers, THE UHI_Discovery_Service SHALL expand the search radius to 100 kilometers and notify the user
7. THE UHI_Discovery_Service SHALL use Beckn_Protocol schemas for all search requests and responses

### Requirement 4: Prescription Image Processing

**User Story:** As a patient with a handwritten prescription, I want to upload a photo of my prescription, so that the system can read it and help me find affordable medicines.

#### Acceptance Criteria

1. WHEN a user uploads a prescription image, THE OCR_Service SHALL extract text from the image within 3 seconds
2. WHEN the prescription contains handwritten text, THE OCR_Service SHALL recognize and extract medicine names with at least 85% accuracy
3. WHEN the prescription image is blurry or poorly lit, THE Vision_Module SHALL request a clearer image from the user
4. WHEN multiple medicines are listed in a prescription, THE Vision_Module SHALL extract all medicine names as a structured list
5. THE Vision_Module SHALL support common prescription image formats (JPEG, PNG, HEIC)
6. WHEN the prescription contains both brand names and generic names, THE Vision_Module SHALL identify and extract both

### Requirement 5: Medicine Name Normalization

**User Story:** As a patient who received a prescription with brand names, I want to know the generic alternatives, so that I can purchase cheaper medicines.

#### Acceptance Criteria

1. WHEN a Brand_Medicine name is extracted from a prescription, THE Medicine_Price_Service SHALL map it to the corresponding Generic_Medicine salt name
2. WHEN a medicine name is ambiguous or has multiple possible matches, THE Medicine_Price_Service SHALL present all options to the user for selection
3. WHEN a medicine name cannot be recognized, THE Medicine_Price_Service SHALL request clarification from the user
4. THE Medicine_Price_Service SHALL maintain a database mapping of at least 500 common Brand_Medicine names to Generic_Medicine salt names
5. WHEN a Generic_Medicine name is already provided in the prescription, THE Medicine_Price_Service SHALL use it directly without mapping

### Requirement 6: Medicine Price Comparison

**User Story:** As a cost-conscious patient, I want to compare prices between branded medicines and Jan Aushadhi generics, so that I can make informed purchasing decisions.

#### Acceptance Criteria

1. WHEN a Generic_Medicine is identified, THE Medicine_Price_Service SHALL retrieve prices from both brand pharmacies and Jan_Aushadhi_Kendra locations
2. WHEN displaying price comparisons, THE Sahayak_System SHALL show the percentage savings when purchasing from Jan_Aushadhi_Kendra
3. WHEN Jan_Aushadhi_Kendra prices are significantly lower (>50% savings), THE Sahayak_System SHALL highlight this in Savings_Mode with visual emphasis
4. THE Medicine_Price_Service SHALL display prices for the nearest 3 Jan_Aushadhi_Kendra locations based on user location
5. WHEN a medicine is not available at Jan_Aushadhi_Kendra, THE Medicine_Price_Service SHALL indicate this and show only brand pharmacy prices
6. THE Sahayak_System SHALL display both per-unit price and total prescription cost for comparison

### Requirement 7: Location Services

**User Story:** As a mobile user, I want the system to automatically detect my location, so that I can find nearby healthcare providers and pharmacies without manual entry.

#### Acceptance Criteria

1. WHEN the application starts, THE Sahayak_System SHALL request location permissions from the user
2. WHEN location permissions are granted, THE Sahayak_System SHALL use Amazon Location Service to determine the user's current coordinates
3. WHEN location permissions are denied, THE Sahayak_System SHALL allow manual location entry via city/district/pincode

### Requirement 8: Multilingual Audio Output

**User Story:** As a non-English speaking user, I want to hear responses in my native language, so that I can understand the information provided by the system.

#### Acceptance Criteria

1. WHEN the system generates a response, THE TTS_Service SHALL convert text to speech in the user's Native_Language
2. THE TTS_Service SHALL support both Hindi and Tamil language output with natural-sounding voices
3. WHEN providing doctor recommendations, THE TTS_Service SHALL speak the doctor's name, specialty, hospital name, distance, and cost information
4. WHEN providing medicine information, THE TTS_Service SHALL speak the generic name, brand name, and price comparison
5. THE TTS_Service SHALL deliver audio responses within 2 seconds of text generation
6. THE Sahayak_System SHALL allow users to replay audio responses without regenerating them

### Requirement 9: Bedrock Agent Orchestration

**User Story:** As a system architect, I want the AI agent to coordinate multiple services intelligently, so that the system provides coherent and contextual responses.

#### Acceptance Criteria

1. THE Bedrock_Agent SHALL implement an action group with tool_find_doctor function accepting symptom and location parameters
2. THE Bedrock_Agent SHALL implement an action group with tool_find_medicine function accepting image_text and location parameters
3. WHEN a user query requires multiple steps, THE Bedrock_Agent SHALL execute action groups in the appropriate sequence
4. WHEN an action group fails, THE Bedrock_Agent SHALL provide a meaningful error message in the user's Native_Language
5. THE Bedrock_Agent SHALL maintain conversation context across multiple user interactions within a session
6. THE Bedrock_Agent SHALL use Claude 3.5 Sonnet model for natural language understanding and generation

### Requirement 10: Emergency Mode Handling

**User Story:** As a patient with urgent symptoms, I want the system to recognize emergencies and prioritize immediate care options, so that I can get help quickly.

#### Acceptance Criteria

1. WHEN symptoms indicate potential emergency conditions (chest pain, severe bleeding, difficulty breathing, loss of consciousness), THE Bedrock_Agent SHALL activate Emergency_Mode
2. WHEN Emergency_Mode is activated, THE Sahayak_System SHALL display a red-themed UI with prominent emergency indicators
3. WHEN in Emergency_Mode, THE UHI_Discovery_Service SHALL prioritize hospitals with emergency departments over clinics
4. WHEN in Emergency_Mode, THE Sahayak_System SHALL provide emergency helpline numbers (108 ambulance service)
5. WHEN Emergency_Mode is activated, THE Sahayak_System SHALL skip non-essential questions and expedite the search process


### Requirement 11: Error Handling and User Feedback

**User Story:** As a user, I want clear feedback when something goes wrong, so that I know what to do next.

#### Acceptance Criteria

1. WHEN an API call fails, THE Sahayak_System SHALL display an error message in the user's Native_Language explaining the issue
2. WHEN the STT_Service cannot transcribe audio, THE Sahayak_System SHALL ask the user to speak more clearly or reduce background noise
3. WHEN the OCR_Service cannot read a prescription, THE Sahayak_System SHALL provide tips for taking a better photo and offer the option to manually enter the prescription
4. WHEN no healthcare providers are found, THE Sahayak_System SHALL suggest expanding the search radius or trying a different specialist type
5. WHEN the system encounters an unexpected error, THE Sahayak_System SHALL log the error details and display a generic error message to the user
6. THE Sahayak_System SHALL provide a retry option for all failed operations

### Requirement 12: Beckn Protocol Compliance

**User Story:** As a system integrator, I want the system to use standard Beckn Protocol schemas, so that it can interoperate with the UHI network.

#### Acceptance Criteria

1. WHEN making UHI search requests, THE UHI_Discovery_Service SHALL format requests according to Beckn Protocol search schema
2. WHEN receiving UHI responses, THE UHI_Discovery_Service SHALL parse responses according to Beckn Protocol on_search schema
3. THE UHI_Discovery_Service SHALL include required Beckn Protocol fields (context, message) in all requests
4. THE UHI_Discovery_Service SHALL validate Beckn Protocol responses before processing them
5. WHEN Beckn Protocol validation fails, THE UHI_Discovery_Service SHALL log the validation error and return a user-friendly error message

### Requirement 13: Mobile-First User Interface

**User Story:** As a mobile user, I want an intuitive interface optimized for smartphones, so that I can easily access healthcare services on my device.

#### Acceptance Criteria

1. THE Sahayak_System SHALL provide a mobile application built with React Native or Flutter
2. THE Sahayak_System SHALL support both portrait and landscape orientations
3. WHEN in Emergency_Mode, THE Sahayak_System SHALL display a red-themed interface with large, easily tappable buttons
4. WHEN in Savings_Mode, THE Sahayak_System SHALL display a green-themed interface highlighting cost savings
5. THE Sahayak_System SHALL provide voice input buttons prominently on the main screen
6. THE Sahayak_System SHALL provide camera/gallery access buttons for prescription upload
7. THE Sahayak_System SHALL display results in a scrollable list with clear visual hierarchy

### Requirement 14: Security and Privacy

**User Story:** As a patient, I want my health information to be kept private and secure, so that my medical data is not misused.

#### Acceptance Criteria

1. THE Sahayak_System SHALL not store audio recordings after transcription is complete
2. THE Sahayak_System SHALL not store prescription images after OCR processing is complete
3. THE Sahayak_System SHALL encrypt all data in transit using HTTPS/TLS
4. THE Sahayak_System SHALL encrypt sensitive data at rest in DynamoDB
5. THE Sahayak_System SHALL not share user health data with third parties without explicit consent
6. THE Sahayak_System SHALL comply with Indian data protection regulations and ABDM privacy guidelines

### Requirement 15: Data Persistence and Caching

**User Story:** As a system administrator, I want to cache frequently accessed data, so that the system responds quickly even with limited network connectivity.

#### Acceptance Criteria

1. THE Sahayak_System SHALL cache UHI network responses in DynamoDB for up to 24 hours
2. THE Sahayak_System SHALL cache medicine price data in DynamoDB for up to 7 days
3. WHEN cached data is available and less than the expiration time, THE Sahayak_System SHALL use cached data instead of making new API calls
4. WHEN cached data is stale or unavailable, THE Sahayak_System SHALL fetch fresh data and update the cache
5. THE Sahayak_System SHALL store user session data in DynamoDB with automatic expiration after 1 hour of inactivity

