# Requirements Document: Sahayak Health Services Application

## 1. Overview

Sahayak is a healthcare application designed to help patients in India find affordable healthcare services and medicines by integrating with the Ayushman Bharat Digital Mission (ABDM) and United Health Interface (UHI) services. The application prioritizes accessibility for rural users with limited internet connectivity and medical knowledge.

## 2. Target Users

### 2.1 Primary Users
- Patients seeking affordable healthcare and medicines
- Users with limited medical knowledge
- Users in remote/rural areas with poor internet connectivity
- Users who may need to communicate in their native language

### 2.2 Secondary Users
- Hospitals registered in the ABDM system
- Doctors and specialists
- Pharmacies and Jan Aushadhi Kendras

## 3. User Stories and Acceptance Criteria

### 3.1 Patient Profile Management

**User Story:** As a patient, I want to enter my personal details so that I can receive personalized healthcare recommendations.

**Acceptance Criteria:**
- 3.1.1 Patient can enter name, age, and gender
- 3.1.2 Patient details are stored securely
- 3.1.3 Patient details are automatically deleted after 24 hours
- 3.1.4 Patient can update their details at any time

### 3.2 Symptom Input

**User Story:** As a patient, I want to describe my symptoms in my native language so that I can get appropriate medical recommendations.

**Acceptance Criteria:**
- 3.2.1 Patient can type symptoms in their native language
- 3.2.2 Patient can speak symptoms using voice input (Speech-to-Text)
- 3.2.3 AI Agent translates native language input to English
- 3.2.4 Patient can input post-medication symptoms
- 3.2.5 System validates and confirms understood symptoms with the patient

### 3.3 Prescription Upload

**User Story:** As a patient, I want to upload my prescription so that I can find affordable medicine alternatives.

**Acceptance Criteria:**
- 3.3.1 Patient can upload prescription as an image
- 3.3.2 Patient can enter prescription as text
- 3.3.3 System uses Amazon OCR to extract text from prescription images
- 3.3.4 System extracts medicine names from prescription
- 3.3.5 System identifies active ingredients from medicine names

### 3.4 Location Services

**User Story:** As a patient, I want to provide my location so that I can find nearby healthcare services.

**Acceptance Criteria:**
- 3.4.1 System automatically fetches location using smartphone GPS
- 3.4.2 Patient can manually enter location if GPS is unavailable
- 3.4.3 System validates location data
- 3.4.4 Location data is deleted after 24 hours

### 3.5 Doctor and Specialist Recommendations

**User Story:** As a patient, I want to find nearby doctors and specialists based on my symptoms so that I can get appropriate treatment.

**Acceptance Criteria:**
- 3.5.1 AI Agent analyzes symptoms and patient details
- 3.5.2 System recommends appropriate doctors/specialists
- 3.5.3 System displays doctors sorted by distance from patient location
- 3.5.4 System prioritizes government healthcare facilities
- 3.5.5 System shows doctor availability and consultation fees
- 3.5.6 Patient can view doctor profiles and specializations
- 3.5.7 Patient can book appointments with selected doctors

### 3.6 Affordable Medicine Search

**User Story:** As a patient, I want to find cheaper generic alternatives to prescribed medicines so that I can save money.

**Acceptance Criteria:**
- 3.6.1 System extracts active ingredients from prescription
- 3.6.2 System searches for generic medicines at nearby Jan Aushadhi Kendras
- 3.6.3 If no Jan Aushadhi Kendra is available, system shows brand-name alternatives at nearby pharmacies
- 3.6.4 System displays medicine prices prominently
- 3.6.5 System sorts medicines by price (cheapest first)
- 3.6.6 Patient can tap on medicine to view details (chemical composition, side effects, dosage)
- 3.6.7 System shows distance to pharmacy/Jan Aushadhi Kendra

### 3.7 Emergency Hospital Bed Locator

**User Story:** As a patient in an emergency, I want to quickly find nearby hospitals with available beds and required facilities so that I can receive immediate treatment.

**Acceptance Criteria:**
- 3.7.1 Patient can manually activate emergency mode
- 3.7.2 System requires symptom input even in emergency mode
- 3.7.3 AI Agent analyzes symptoms to determine if situation is an emergency
- 3.7.4 If emergency is confirmed, app switches to red color theme
- 3.7.5 System queries ABDM and UHI services for nearby hospitals with available beds
- 3.7.6 System filters hospitals based on required facilities (e.g., ventilator for respiratory emergency)
- 3.7.7 System displays hospitals sorted by distance
- 3.7.8 System shows bed availability and facility information
- 3.7.9 System contacts emergency services automatically
- 3.7.10 Emergency processing completes within 30 seconds

### 3.8 Multilingual AI Agent

**User Story:** As a patient who speaks a native Indian language, I want to communicate with an AI agent in my language so that I can accurately describe my symptoms.

**Acceptance Criteria:**
- 3.8.1 AI Agent supports major Indian languages
- 3.8.2 AI Agent uses Speech-to-Text (STT) for voice input
- 3.8.3 AI Agent uses Text-to-Speech (TTS) for voice output
- 3.8.4 AI Agent translates between native language and English
- 3.8.5 AI Agent reads and interprets prescriptions
- 3.8.6 AI Agent provides responses in patient's native language

## 4. Technical Requirements

### 4.1 Performance
- 4.1.1 Application must be optimized for low-bandwidth environments
- 4.1.2 Application must work on 2G/3G networks
- 4.1.3 Emergency mode processing must complete within 30 seconds
- 4.1.4 Search results must load within 5 seconds on 3G connection

### 4.2 Data Privacy and Security
- 4.2.1 All user data must be encrypted in transit and at rest
- 4.2.2 Patient details must be automatically deleted after 24 hours
- 4.2.3 Location data must be automatically deleted after 24 hours
- 4.2.4 Prescription images must be automatically deleted after 24 hours
- 4.2.5 Application must comply with Indian data protection regulations

### 4.3 API Integration
- 4.3.1 Application must integrate with ABDM services
- 4.3.2 Application must integrate with UHI services
- 4.3.3 API responses must be cached for 24 hours to avoid overloading
- 4.3.4 Application must handle API failures gracefully

### 4.4 External Services
- 4.4.1 Application must use Amazon OCR for prescription image processing
- 4.4.2 Application must integrate with emergency services
- 4.4.3 Application must use GPS services for location detection
- 4.4.4 Application must use TTS (Text-to-Speech) and STT (Speech-to-Text) services
- 4.4.5 TTS/STT may be provided by Amazon services or AI Agent's built-in functionality
- 4.4.6 AI Agent selection should be based on suitability for rural Indian users and language support

### 4.5 Offline Capability
- 4.5.1 Application should cache previously searched data for offline access
- 4.5.2 Application should indicate when operating in offline mode
- 4.5.3 Application should queue actions when offline and sync when online

## 5. Success Metrics

### 5.1 Healthcare Access
- 5.1.1 Patient successfully finds and books appointment with doctor/specialist
- 5.1.2 Patient successfully locates hospital bed in emergency
- 5.1.3 Patient receives treatment within acceptable timeframe

### 5.2 Cost Savings
- 5.2.1 Patient saves money by using government healthcare facilities
- 5.2.2 Patient saves money by purchasing generic medicines from Jan Aushadhi
- 5.2.3 Average cost savings per patient is measurable and significant

### 5.3 Emergency Response
- 5.3.1 Emergency cases are identified within 30 seconds
- 5.3.2 Emergency services are contacted automatically
- 5.3.3 Patient receives lifesaving treatment quickly

### 5.4 User Awareness
- 5.4.1 Patient discovers new healthcare providers in their area
- 5.4.2 Patient becomes aware of government healthcare options
- 5.4.3 Patient learns about Jan Aushadhi Kendras and generic medicines

## 6. Constraints

### 6.1 User Constraints
- Users may have limited medical knowledge
- Users may be in remote areas with poor connectivity
- Users may only speak native Indian languages
- Users may have limited smartphone literacy

### 6.2 Technical Constraints
- Must work efficiently on low-bandwidth networks
- Must minimize data storage (24-hour retention limit)
- Must avoid overloading ABDM and UHI APIs
- Must process emergencies quickly

### 6.3 Regulatory Constraints
- Must comply with Indian healthcare regulations
- Must comply with data protection laws
- Must handle medical data responsibly

## 7. Out of Scope

- Direct medicine ordering/delivery
- Payment processing for consultations
- Telemedicine video consultations
- Health records management beyond 24 hours
- Insurance claim processing
