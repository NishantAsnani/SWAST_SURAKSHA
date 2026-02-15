# SwasthChain – SURAKSHA
## System Design Document

**Version:** 1.0  
**Date:** February 15, 2026  
**Document Status:** Draft

---

## Table of Contents

1. System Architecture Overview
2. High-Level Architecture
3. Component Design
4. Data Architecture
5. API Design
6. Security Architecture
7. Blockchain Architecture
8. AI/ML Architecture
9. Integration Architecture
10. Deployment Architecture
11. Data Flow Diagrams
12. Database Schema Design
13. Technology Stack Details
14. Scalability & Performance Design
15. Disaster Recovery & Backup Strategy

---

## 1. System Architecture Overview

### 1.1 Architecture Style

SwasthChain follows a **microservices-based architecture** with the following key principles:

- **Service Independence**: Each microservice owns its data and business logic
- **API-First Design**: All services communicate via well-defined REST/GraphQL APIs
- **Event-Driven Communication**: Asynchronous messaging for non-critical operations
- **Decentralized Data Storage**: Hybrid approach using centralized DB + decentralized IPFS/blockchain
- **Cloud-Native**: Containerized services with orchestration support
- **Polyglot Persistence**: Different databases for different use cases

### 1.2 Design Principles

- **Security by Design**: Encryption, authentication, and authorization at every layer
- **Privacy First**: Patient data ownership with consent-based access
- **Interoperability**: FHIR-compliant APIs for healthcare ecosystem integration
- **Scalability**: Horizontal scaling capability for all services
- **Resilience**: Fault tolerance with graceful degradation
- **Observability**: Comprehensive logging, monitoring, and tracing

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                    │
├─────────────────────────────────────────────────────────────────────────┤
│  Web App (React)  │  Mobile App (React Native)  │  Doctor Portal       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      API GATEWAY & LOAD BALANCER                         │
│                    (Kong/AWS API Gateway + NGINX)                        │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                        MICROSERVICES LAYER                                │
├──────────────────────────────────────────────────────────────────────────┤
│ User Service │ Health Record │ FHIR Service │ Consent Mgmt │ Auth Service│
│ AI/ML Service│ Insurance Svc │ Telemedicine │ Notification │ OCR Service │
│ Voice Agent  │ Analytics Svc │ Appointment  │ Prescription │ Research Svc│
└──────────────────────────────────────────────────────────────────────────┘
                    │               │               │
        ┌───────────┼───────────────┼───────────────┼───────────┐
        ▼           ▼               ▼               ▼           ▼
┌─────────────┐ ┌──────────┐ ┌──────────────┐ ┌─────────┐ ┌──────────┐
│ PostgreSQL  │ │  Redis   │ │  HAPI-FHIR   │ │ Message │ │  MinIO   │
│  (Primary)  │ │ (Cache)  │ │   Server     │ │  Queue  │ │  (S3)    │
└─────────────┘ └──────────┘ └──────────────┘ └─────────┘ └──────────┘
        │                                                         │
        ▼                                                         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    DECENTRALIZED STORAGE LAYER                           │
├─────────────────────────────────────────────────────────────────────────┤
│         IPFS/Filecoin (Medical Documents & Imaging)                     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      BLOCKCHAIN LAYER                                    │
├─────────────────────────────────────────────────────────────────────────┤
│  Polygon Network │ Smart Contracts │ Audit Trail │ Consent Records     │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    EXTERNAL INTEGRATIONS                                 │
├─────────────────────────────────────────────────────────────────────────┤
│  ABDM APIs  │  Insurance APIs  │  Hospital Systems  │  Lab Networks    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Component Design

### 3.1 User Service

**Responsibility**: User authentication, authorization, profile management

**Technology**: Spring Boot (Java 17), PostgreSQL

**Key Features**:
- Aadhaar-based registration via ABDM
- Multi-factor authentication (OTP, biometric)
- Role-based access control (RBAC)
- Family account linking
- Session management

**APIs**:
- `POST /api/v1/users/register` - User registration
- `POST /api/v1/users/login` - User authentication
- `GET /api/v1/users/{userId}/profile` - Get user profile
- `PUT /api/v1/users/{userId}/profile` - Update profile
- `POST /api/v1/users/verify-aadhaar` - Aadhaar verification
- `POST /api/v1/users/mfa/enable` - Enable MFA
- `GET /api/v1/users/{userId}/family` - Get family members

**Database Tables**:
- `users` - User credentials and basic info
- `user_profiles` - Extended profile information
- `user_roles` - Role assignments
- `family_links` - Family relationships
- `sessions` - Active user sessions
- `audit_logs` - User activity logs

### 3.2 Health Record Service

**Responsibility**: Core health record management, CRUD operations

**Technology**: Spring Boot (Java 17), PostgreSQL, HAPI-FHIR

**Key Features**:
- FHIR-compliant record storage
- Medical history management
- Document upload/retrieval
- Version control
- Timeline generation

**APIs**:
- `POST /api/v1/records` - Create health record
- `GET /api/v1/records/{recordId}` - Get record details
- `PUT /api/v1/records/{recordId}` - Update record
- `GET /api/v1/patients/{patientId}/records` - Get patient records
- `GET /api/v1/patients/{patientId}/timeline` - Get medical timeline
- `POST /api/v1/records/{recordId}/documents` - Upload document
- `GET /api/v1/records/search` - Search records

**Database Tables**:
- `health_records` - Core health record data
- `diagnoses` - Diagnosis information (ICD-10)
- `prescriptions` - Medication prescriptions
- `lab_results` - Laboratory test results
- `vitals` - Vital signs measurements
- `allergies` - Patient allergies
- `immunizations` - Vaccination records
- `record_versions` - Version history

### 3.3 OCR & Document Digitization Service

**Responsibility**: Extract structured data from scanned/uploaded documents

**Technology**: FastAPI (Python), Tesseract OCR, spaCy, Hugging Face Transformers

**Key Features**:
- Multi-format document processing (PDF, JPG, PNG)
- Handwriting recognition
- Medical terminology extraction
- Structured data generation
- Confidence scoring
- Manual review flagging

**APIs**:
- `POST /api/v1/ocr/process` - Process document
- `GET /api/v1/ocr/jobs/{jobId}` - Get processing status
- `POST /api/v1/ocr/bulk-upload` - Bulk document processing
- `GET /api/v1/ocr/review-queue` - Get low-confidence extractions
- `PUT /api/v1/ocr/review/{extractionId}` - Manual correction

**Processing Pipeline**:
1. Document upload → MinIO/S3
2. Image preprocessing (deskew, denoise, enhance)
3. OCR extraction (Tesseract)
4. NLP processing (entity recognition)
5. Medical term mapping (SNOMED CT, ICD-10)
6. FHIR resource generation
7. Confidence scoring
8. Storage in Health Record Service

**Database Tables**:
- `ocr_jobs` - Processing job status
- `extracted_data` - Raw OCR output
- `review_queue` - Items needing manual review
- `extraction_confidence` - Confidence scores

### 3.4 AI/ML Service

**Responsibility**: Disease prediction, pattern analysis, risk assessment

**Technology**: FastAPI (Python), PyTorch, TensorFlow, scikit-learn, MLflow

**Key Features**:
- Family history-based disease prediction
- Health pattern identification
- Anomaly detection
- Risk scoring
- Personalized recommendations
- Model versioning and monitoring

**APIs**:
- `POST /api/v1/ai/predict-disease-risk` - Disease risk prediction
- `POST /api/v1/ai/analyze-patterns` - Health pattern analysis
- `POST /api/v1/ai/family-risk-assessment` - Family history analysis
- `GET /api/v1/ai/recommendations/{patientId}` - Get personalized recommendations
- `POST /api/v1/ai/detect-anomalies` - Anomaly detection
- `POST /api/v1/ai/drug-interaction-check` - Drug interaction analysis
- `POST /api/v1/ai/image-analysis` - Medical image analysis

**ML Models**:

1. **Disease Risk Predictor**
   - Input: Demographics, family history, lifestyle, vitals, lab results
   - Output: Risk scores for diabetes, hypertension, heart disease, cancer
   - Algorithm: Gradient Boosting (XGBoost), Neural Networks
   - Training: Supervised learning on historical patient data

2. **Pattern Analyzer**
   - Input: Longitudinal patient data (time series)
   - Output: Trends, anomalies, correlations
   - Algorithm: LSTM, Time Series Analysis, Clustering
   - Features: Vital trends, lab value patterns, symptom frequency

3. **Family Risk Assessor**
   - Input: Multi-generational family health tree
   - Output: Genetic predisposition scores
   - Algorithm: Graph Neural Networks, Bayesian Networks
   - Features: Disease inheritance patterns, age of onset

4. **Drug Interaction Checker**
   - Input: Current medications, new prescription, allergies
   - Output: Interaction warnings, severity levels
   - Algorithm: Knowledge Graph, Rule-based System
   - Database: Drug interaction database

**Database Tables**:
- `ml_models` - Model metadata and versions
- `predictions` - Prediction results and history
- `model_performance` - Model metrics and monitoring
- `training_datasets` - Dataset metadata

### 3.5 Voice Agent & Chatbot Service

**Responsibility**: Conversational AI for symptom assessment and healthcare guidance

**Technology**: FastAPI (Python), Dialogflow/Rasa, Whisper (Speech-to-Text), gTTS (Text-to-Speech)

**Key Features**:
- Multi-language voice recognition
- Symptom assessment and triage
- Medical specialty recommendation
- Appointment booking
- Basic remedy suggestions
- Emergency escalation
- Doctor handoff with summary

**APIs**:
- `POST /api/v1/voice/session/start` - Start conversation session
- `POST /api/v1/voice/process-audio` - Process voice input
- `POST /api/v1/voice/process-text` - Process text input
- `GET /api/v1/voice/session/{sessionId}` - Get conversation history
- `POST /api/v1/voice/book-appointment` - Book doctor appointment
- `POST /api/v1/voice/emergency-escalate` - Escalate to emergency

**Conversation Flow**:
```
1. Greeting & Language Selection
   ↓
2. Symptom Collection (Multi-turn dialogue)
   ↓
3. Symptom Analysis (NLP + Medical Knowledge Base)
   ↓
4. Severity Assessment (Triage Algorithm)
   ↓
5. Decision Branch:
   - Minor: Home remedy suggestions
   - Moderate: Specialist recommendation + Appointment booking
   - Severe: Emergency escalation + Nearest hospital
   ↓
6. Summary Generation for Doctor
```

**NLP Pipeline**:
1. Speech-to-Text (Whisper/Google Speech API)
2. Language Detection
3. Intent Classification (Symptom reporting, Appointment, Query)
4. Entity Extraction (Symptoms, Duration, Severity)
5. Medical Term Mapping (SNOMED CT)
6. Context Management (Multi-turn conversation)
7. Response Generation
8. Text-to-Speech (gTTS/Azure Speech)

**Database Tables**:
- `conversation_sessions` - Active and historical sessions
- `symptom_reports` - Extracted symptoms
- `triage_decisions` - Triage outcomes
- `appointment_requests` - Booking requests

### 3.6 Insurance & Claims Service

**Responsibility**: Insurance integration, claim processing, verification

**Technology**: Spring Boot (Java 17), PostgreSQL

**Key Features**:
- Eligibility verification
- Automated claim submission
- Document compilation
- AI-assisted validation
- Claim tracking
- Fraud detection

**APIs**:
- `POST /api/v1/insurance/verify-eligibility` - Check insurance eligibility
- `POST /api/v1/insurance/claims` - Submit claim
- `GET /api/v1/insurance/claims/{claimId}` - Get claim status
- `POST /api/v1/insurance/claims/{claimId}/documents` - Attach documents
- `GET /api/v1/insurance/claims/{claimId}/probability` - Get approval probability
- `POST /api/v1/insurance/pre-auth` - Pre-authorization request
- `GET /api/v1/insurance/policies/{patientId}` - Get patient policies

**Claim Processing Workflow**:
```
1. Claim Initiation (Patient/Hospital)
   ↓
2. Document Auto-Compilation (Medical records, bills, prescriptions)
   ↓
3. AI Validation (Completeness check, missing document detection)
   ↓
4. Blockchain Verification (Record authenticity)
   ↓
5. Claim Narrative Generation (AI-generated summary)
   ↓
6. Approval Probability Prediction (ML model)
   ↓
7. Submission to Insurance Company (API integration)
   ↓
8. Status Tracking & Notifications
   ↓
9. Settlement Processing
```

**Database Tables**:
- `insurance_policies` - Patient insurance policies
- `claims` - Claim records
- `claim_documents` - Attached documents
- `claim_status_history` - Status tracking
- `fraud_alerts` - Suspicious activity flags

### 3.7 Research & Analytics Service

**Responsibility**: Anonymized data generation, pattern discovery, research support

**Technology**: FastAPI (Python), PostgreSQL, Apache Spark, Jupyter

**Key Features**:
- Data anonymization (differential privacy)
- Cohort identification
- Pattern discovery
- Statistical analysis
- Research sandbox
- Ethical approval tracking

**APIs**:
- `POST /api/v1/research/request-access` - Request research data access
- `GET /api/v1/research/datasets` - List available datasets
- `POST /api/v1/research/query` - Execute research query
- `GET /api/v1/research/patterns` - Discover health patterns
- `POST /api/v1/research/cohort-identify` - Identify patient cohorts
- `GET /api/v1/research/export/{datasetId}` - Export research data

**Anonymization Techniques**:
- K-anonymity (grouping similar records)
- L-diversity (diverse sensitive attributes)
- Differential privacy (noise addition)
- Data masking (PII removal)
- Aggregation (statistical summaries)

**Research Use Cases**:
1. Disease correlation studies
2. Drug efficacy analysis
3. Epidemiological research
4. Biomarker discovery
5. Clinical trial recruitment
6. Public health monitoring

**Database Tables**:
- `research_requests` - Access requests
- `anonymized_datasets` - Generated datasets
- `research_queries` - Query history
- `ethical_approvals` - IRB approvals
- `data_access_logs` - Access audit trail

### 3.8 Consent Management Service

**Responsibility**: Patient consent tracking, access control

**Technology**: Spring Boot (Java 17), PostgreSQL, Blockchain

**Key Features**:
- Granular consent management
- Time-bound access
- Purpose-based consent
- Revocation support
- Blockchain-backed proof

**APIs**:
- `POST /api/v1/consent/grant` - Grant consent
- `POST /api/v1/consent/revoke` - Revoke consent
- `GET /api/v1/consent/{patientId}` - Get active consents
- `POST /api/v1/consent/verify` - Verify consent validity
- `GET /api/v1/consent/history/{patientId}` - Consent history

**Consent Model**:
```json
{
  "consentId": "uuid",
  "patientId": "uuid",
  "grantedTo": "doctor/hospital/researcher",
  "purpose": "treatment/research/insurance",
  "dataTypes": ["diagnoses", "lab_results", "prescriptions"],
  "validFrom": "timestamp",
  "validUntil": "timestamp",
  "status": "active/revoked/expired",
  "blockchainHash": "0x..."
}
```

**Database Tables**:
- `consents` - Consent records
- `consent_history` - Audit trail
- `access_attempts` - Access logs

### 3.9 FHIR Service

**Responsibility**: FHIR-compliant data exchange, interoperability

**Technology**: HAPI-FHIR Server, Java

**Key Features**:
- FHIR R4 compliance
- Resource CRUD operations
- Search capabilities
- Bulk data export
- HL7 v2 conversion

**Supported FHIR Resources**:
- Patient
- Practitioner
- Organization
- Encounter
- Observation
- Condition
- MedicationRequest
- DiagnosticReport
- DocumentReference
- AllergyIntolerance
- Immunization

**APIs** (FHIR Standard):
- `GET /fhir/Patient/{id}` - Get patient resource
- `POST /fhir/Observation` - Create observation
- `GET /fhir/Condition?patient={id}` - Search conditions
- `GET /fhir/Patient/$everything` - Get all patient data
- `POST /fhir/$export` - Bulk data export

### 3.10 Notification Service

**Responsibility**: Multi-channel notifications and alerts

**Technology**: Spring Boot (Java 17), RabbitMQ/Kafka, Firebase Cloud Messaging

**Key Features**:
- SMS notifications (Twilio/AWS SNS)
- Email notifications (SendGrid/AWS SES)
- Push notifications (FCM)
- In-app notifications
- Notification preferences
- Delivery tracking

**APIs**:
- `POST /api/v1/notifications/send` - Send notification
- `GET /api/v1/notifications/{userId}` - Get user notifications
- `PUT /api/v1/notifications/{notificationId}/read` - Mark as read
- `PUT /api/v1/notifications/preferences` - Update preferences

**Notification Types**:
- Appointment reminders
- Lab result alerts
- Medication reminders
- Consent expiry warnings
- Health risk alerts
- Insurance claim updates
- Emergency access notifications

**Database Tables**:
- `notifications` - Notification records
- `notification_preferences` - User preferences
- `delivery_status` - Delivery tracking

---

## 4. Data Architecture

### 4.1 Data Storage Strategy

**Relational Database (PostgreSQL)**:
- User profiles and authentication
- Health records metadata
- Transactional data
- Relationships and references

**FHIR Server (HAPI-FHIR)**:
- Standardized clinical data
- Interoperability layer
- FHIR resource storage

**Cache (Redis)**:
- Session management
- Frequently accessed data
- Rate limiting counters
- Real-time analytics

**Object Storage (MinIO/S3)**:
- Medical documents (PDF, images)
- Backup storage
- Hot storage for recent files

**Decentralized Storage (IPFS/Filecoin)**:
- Long-term document archival
- Medical imaging (DICOM files)
- Immutable storage
- Cold storage for old records

**Blockchain (Polygon)**:
- Audit trail records
- Consent proofs
- Document hashes
- Insurance claim verification

**Message Queue (RabbitMQ/Kafka)**:
- Asynchronous task processing
- Event-driven communication
- Service decoupling

### 4.2 Data Flow

**Write Path**:
```
Client → API Gateway → Microservice → PostgreSQL (metadata)
                                    → MinIO/S3 (documents)
                                    → IPFS (archival)
                                    → Blockchain (audit)
                                    → Message Queue (async tasks)
```

**Read Path**:
```
Client → API Gateway → Microservice → Redis (cache hit)
                                    → PostgreSQL (cache miss)
                                    → MinIO/S3 (documents)
                                    → IPFS (archived files)
```

### 4.3 Data Retention Policy

| Data Type | Hot Storage | Warm Storage | Cold Storage | Retention |
|-----------|-------------|--------------|--------------|-----------|
| Active Records | PostgreSQL | - | - | Indefinite |
| Recent Documents | MinIO/S3 | - | - | 2 years |
| Archived Documents | - | MinIO/S3 | IPFS | Indefinite |
| Medical Images | MinIO/S3 | - | IPFS | Indefinite |
| Audit Logs | PostgreSQL | S3 | - | 7 years |
| Cache Data | Redis | - | - | 24 hours |
| Blockchain Records | - | - | Polygon | Permanent |

---

## 5. API Design

### 5.1 API Standards

- **Protocol**: REST over HTTPS
- **Format**: JSON (application/json)
- **Versioning**: URI versioning (/api/v1/, /api/v2/)
- **Authentication**: JWT Bearer tokens
- **Rate Limiting**: 1000 requests/hour per user
- **Pagination**: Cursor-based for large datasets

### 5.2 API Request/Response Format

**Standard Request**:
```json
{
  "requestId": "uuid",
  "timestamp": "2026-02-15T10:30:00Z",
  "data": {
    // Request payload
  }
}
```

**Standard Success Response**:
```json
{
  "status": "success",
  "requestId": "uuid",
  "timestamp": "2026-02-15T10:30:01Z",
  "data": {
    // Response payload
  },
  "metadata": {
    "page": 1,
    "pageSize": 20,
    "totalRecords": 100
  }
}
```

**Standard Error Response**:
```json
{
  "status": "error",
  "requestId": "uuid",
  "timestamp": "2026-02-15T10:30:01Z",
  "error": {
    "code": "INVALID_INPUT",
    "message": "Patient ID is required",
    "details": {
      "field": "patientId",
      "constraint": "not_null"
    }
  }
}
```

### 5.3 Authentication Flow

**JWT Token Structure**:
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "userId",
    "role": "patient/doctor/admin",
    "aadhaarVerified": true,
    "iat": 1708000000,
    "exp": 1708086400
  }
}
```

**Authentication Endpoints**:
- `POST /api/v1/auth/login` - Login with credentials
- `POST /api/v1/auth/refresh` - Refresh access token
- `POST /api/v1/auth/logout` - Logout and invalidate token
- `POST /api/v1/auth/mfa/verify` - Verify MFA code

### 5.4 Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| SUCCESS | 200 | Request successful |
| CREATED | 201 | Resource created |
| INVALID_INPUT | 400 | Invalid request data |
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |
| SERVICE_UNAVAILABLE | 503 | Service temporarily unavailable |

---

## 6. Security Architecture

### 6.1 Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 7: Application Security (Input Validation, OWASP)    │
├─────────────────────────────────────────────────────────────┤
│ Layer 6: Authentication & Authorization (JWT, RBAC, MFA)   │
├─────────────────────────────────────────────────────────────┤
│ Layer 5: API Security (Rate Limiting, API Keys, OAuth)     │
├─────────────────────────────────────────────────────────────┤
│ Layer 4: Data Encryption (AES-256 at rest, TLS 1.3)        │
├─────────────────────────────────────────────────────────────┤
│ Layer 3: Network Security (VPC, Firewall, WAF)             │
├─────────────────────────────────────────────────────────────┤
│ Layer 2: Infrastructure Security (Container Security, IDS)  │
├─────────────────────────────────────────────────────────────┤
│ Layer 1: Physical Security (Cloud Provider Data Centers)    │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Encryption Strategy

**Data at Rest**:
- Database: AES-256 encryption (PostgreSQL TDE)
- File Storage: Server-side encryption (S3/MinIO)
- Sensitive Fields: Application-level encryption (patient SSN, Aadhaar)
- Backups: Encrypted backups with separate keys

**Data in Transit**:
- TLS 1.3 for all API communications
- Certificate pinning for mobile apps
- VPN for internal service communication
- End-to-end encryption for sensitive documents

**Key Management**:
- AWS KMS / Azure Key Vault for key storage
- Key rotation every 90 days
- Separate keys for different data types
- Hardware Security Module (HSM) for critical keys

### 6.3 Access Control

**Role-Based Access Control (RBAC)**:

| Role | Permissions |
|------|-------------|
| Patient | View own records, grant consent, manage profile |
| Doctor | View consented records, create records, prescribe |
| Hospital Admin | Manage hospital users, view hospital records |
| Insurance Officer | View claim-related records, process claims |
| Researcher | Access anonymized data, run queries |
| System Admin | System configuration, user management |

**Consent-Based Access**:
- All data access requires valid consent
- Consent verification before every read operation
- Automatic consent expiry
- Audit trail for all access attempts

### 6.4 Security Measures

**Application Security**:
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection (Content Security Policy)
- CSRF protection (tokens)
- Secure password hashing (bcrypt)
- Session timeout (15 minutes inactivity)

**API Security**:
- Rate limiting (1000 req/hour per user)
- API key authentication for external integrations
- Request signing for sensitive operations
- IP whitelisting for admin operations
- DDoS protection (CloudFlare/AWS Shield)

**Monitoring & Detection**:
- Real-time security monitoring (SIEM)
- Intrusion detection system (IDS)
- Anomaly detection (unusual access patterns)
- Failed login attempt tracking
- Automated security alerts

**Compliance**:
- DPDP Act 2023 compliance
- ISO 27001 certification
- Regular security audits (quarterly)
- Penetration testing (bi-annual)
- Vulnerability scanning (weekly)

---

## 7. Blockchain Architecture

### 7.1 Blockchain Network

**Network**: Polygon (Mainnet for production, Sepolia for testing)

**Why Polygon**:
- Low transaction costs (~$0.01 per transaction)
- Fast confirmation times (~2 seconds)
- Ethereum compatibility
- High throughput (7000+ TPS)
- Active ecosystem

### 7.2 Smart Contracts

**Contract 1: AuditTrail.sol**
```solidity
// Stores immutable audit records
contract AuditTrail {
    struct AuditRecord {
        bytes32 recordHash;
        address actor;
        string action;
        uint256 timestamp;
        bytes32 consentHash;
    }
    
    mapping(bytes32 => AuditRecord[]) public auditTrail;
    
    function logAccess(
        bytes32 recordId,
        bytes32 recordHash,
        string memory action,
        bytes32 consentHash
    ) public;
    
    function getAuditTrail(bytes32 recordId) 
        public view returns (AuditRecord[] memory);
}
```

**Contract 2: ConsentManagement.sol**
```solidity
// Manages patient consent records
contract ConsentManagement {
    struct Consent {
        address patient;
        address grantedTo;
        string purpose;
        uint256 validFrom;
        uint256 validUntil;
        bool isActive;
    }
    
    mapping(bytes32 => Consent) public consents;
    
    function grantConsent(
        bytes32 consentId,
        address grantedTo,
        string memory purpose,
        uint256 duration
    ) public;
    
    function revokeConsent(bytes32 consentId) public;
    
    function verifyConsent(bytes32 consentId) 
        public view returns (bool);
}
```

**Contract 3: InsuranceClaim.sol**
```solidity
// Verifies insurance claims
contract InsuranceClaim {
    struct Claim {
        bytes32 claimId;
        address patient;
        address hospital;
        address insurer;
        uint256 amount;
        bytes32[] documentHashes;
        ClaimStatus status;
        uint256 timestamp;
    }
    
    enum ClaimStatus { Submitted, Verified, Approved, Rejected, Settled }
    
    mapping(bytes32 => Claim) public claims;
    
    function submitClaim(
        bytes32 claimId,
        address hospital,
        address insurer,
        uint256 amount,
        bytes32[] memory documentHashes
    ) public;
    
    function verifyClaim(bytes32 claimId) public;
    
    function approveClaim(bytes32 claimId) public;
}
```

### 7.3 Blockchain Integration Flow

**Record Creation Flow**:
```
1. User creates health record in backend
2. Backend generates record hash (SHA-256)
3. Backend calls smart contract to log creation
4. Transaction submitted to Polygon network
5. Transaction confirmed (~2 seconds)
6. Blockchain transaction hash stored in database
7. User receives confirmation
```

**Access Verification Flow**:
```
1. Doctor requests patient record
2. Backend checks consent in database
3. Backend verifies consent on blockchain
4. If valid, grant access
5. Log access event on blockchain
6. Return record to doctor
```

### 7.4 Gas Optimization

- Batch multiple operations into single transaction
- Use events for non-critical data storage
- Minimize on-chain storage (store hashes, not full data)
- Use Layer 2 solutions for high-frequency operations
- Implement transaction queuing for cost optimization

---

## 8. AI/ML Architecture

### 8.1 ML Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA COLLECTION LAYER                        │
│  Patient Records │ Lab Results │ Family History │ Lifestyle Data │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   DATA PREPROCESSING LAYER                       │
│  Cleaning │ Normalization │ Feature Engineering │ Anonymization │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      FEATURE STORE                               │
│              (Centralized feature repository)                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     MODEL TRAINING LAYER                         │
│  Disease Predictor │ Pattern Analyzer │ Risk Assessor │ NLP     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    MODEL REGISTRY (MLflow)                       │
│         Version Control │ Metrics │ Artifacts │ Metadata        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   MODEL SERVING LAYER                            │
│    REST API │ Batch Inference │ Real-time Prediction            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   MONITORING & FEEDBACK                          │
│  Performance Tracking │ Drift Detection │ Retraining Triggers   │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Feature Engineering

**Patient Demographics**:
- Age, gender, BMI, location
- Socioeconomic indicators
- Lifestyle factors (smoking, alcohol, exercise)

**Clinical Features**:
- Vital signs (BP, heart rate, temperature)
- Lab values (glucose, cholesterol, HbA1c)
- Medication history
- Diagnosis history (ICD-10 codes)

**Family History Features**:
- Disease occurrence in family tree
- Age of onset for family members
- Genetic risk scores
- Multi-generational patterns

**Temporal Features**:
- Time since last visit
- Frequency of hospitalizations
- Trend analysis (improving/declining)
- Seasonal patterns

**Derived Features**:
- Risk scores (Framingham, ASCVD)
- Comorbidity indices
- Medication adherence scores
- Health trajectory indicators

### 8.3 Model Details

**Disease Risk Prediction Model**:
- **Architecture**: Gradient Boosting (XGBoost) + Neural Network ensemble
- **Input**: 150+ features (demographics, vitals, labs, family history)
- **Output**: Risk scores for 10 major diseases (0-100 scale)
- **Training Data**: 500K+ anonymized patient records
- **Performance**: AUC-ROC > 0.85 for major diseases
- **Retraining**: Monthly with new data
- **Inference Time**: <100ms

**Health Pattern Analyzer**:
- **Architecture**: LSTM (Long Short-Term Memory) network
- **Input**: Time series data (6-12 months of vitals, labs)
- **Output**: Trend predictions, anomaly flags
- **Training Data**: 1M+ longitudinal patient records
- **Performance**: 90% accuracy in trend prediction
- **Retraining**: Quarterly
- **Inference Time**: <200ms

**Family Risk Assessor**:
- **Architecture**: Graph Neural Network (GNN)
- **Input**: Family health tree (3 generations)
- **Output**: Genetic predisposition scores
- **Training Data**: 100K+ family health trees
- **Performance**: 80% accuracy in hereditary disease prediction
- **Retraining**: Bi-annually
- **Inference Time**: <500ms

**Symptom NLP Model**:
- **Architecture**: BERT-based transformer (BioBERT)
- **Input**: Patient-described symptoms (text/voice)
- **Output**: Medical entities, severity, specialty recommendation
- **Training Data**: 1M+ symptom descriptions
- **Performance**: 92% entity extraction accuracy
- **Retraining**: Quarterly
- **Inference Time**: <300ms

### 8.4 Model Monitoring

**Metrics Tracked**:
- Prediction accuracy
- Model drift (data distribution changes)
- Inference latency
- Error rates
- Feature importance changes

**Alerting Thresholds**:
- Accuracy drop > 5%
- Latency increase > 50%
- Data drift score > 0.3
- Error rate > 2%

**Retraining Triggers**:
- Scheduled (monthly/quarterly)
- Performance degradation
- Significant data drift
- New disease patterns emerge

---

## 9. Integration Architecture

### 9.1 ABDM Integration

**Integration Points**:
- Health ID creation and verification
- Healthcare Professional Registry (HPR)
- Health Facility Registry (HFR)
- Health Data Exchange (consent-based)

**ABDM APIs Used**:
- `/v1/registration/aadhaar/generateOtp` - Aadhaar OTP generation
- `/v1/registration/aadhaar/verifyOtp` - OTP verification
- `/v1/registration/aadhaar/createHealthId` - Health ID creation
- `/v1/consent/request` - Consent request
- `/v1/consent/fetch` - Fetch consent status
- `/v1/health-information/cm/request` - Health data request
- `/v1/health-information/transfer` - Data transfer

**Data Flow**:
```
SwasthChain → ABDM Gateway → Healthcare Provider
           ← FHIR Bundle ←
```

### 9.2 Insurance Company Integration

**Integration Methods**:
- REST APIs (preferred)
- SFTP file exchange (legacy systems)
- EDI (Electronic Data Interchange)

**Data Exchange**:
- Eligibility verification (real-time)
- Claim submission (batch/real-time)
- Pre-authorization requests
- Claim status updates
- Settlement notifications

**Standard Formats**:
- FHIR Claim resource
- X12 837 (Healthcare Claim)
- Custom JSON schemas

### 9.3 Hospital System Integration

**Integration Approaches**:
- FHIR APIs (modern systems)
- HL7 v2 messages (legacy systems)
- Database replication (read-only)
- File-based exchange (CSV, XML)

**Bidirectional Sync**:
```
Hospital EMR ←→ SwasthChain
- Patient demographics
- Appointments
- Lab results
- Prescriptions
- Discharge summaries
```

**Integration Patterns**:
- Real-time: Webhook notifications
- Batch: Scheduled sync (hourly/daily)
- On-demand: API calls during consultation

### 9.4 Laboratory Network Integration

**Integration Points**:
- Lab test ordering
- Result retrieval
- Report download

**Standards**:
- LOINC codes for test identification
- HL7 v2 ORU messages for results
- FHIR DiagnosticReport resource

---

## 10. Deployment Architecture

### 10.1 Cloud Infrastructure

**Cloud Provider**: AWS (primary), Azure (backup)

**Services Used**:
- **Compute**: EKS (Kubernetes), EC2, Lambda
- **Database**: RDS (PostgreSQL), ElastiCache (Redis)
- **Storage**: S3, EFS
- **Networking**: VPC, CloudFront (CDN), Route 53
- **Security**: KMS, WAF, Shield, GuardDuty
- **Monitoring**: CloudWatch, X-Ray

### 10.2 Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        INGRESS LAYER                             │
│              NGINX Ingress Controller + AWS ALB                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      NAMESPACE: production                       │
├─────────────────────────────────────────────────────────────────┤
│  Deployment: user-service (3 replicas)                          │
│  Deployment: health-record-service (5 replicas)                 │
│  Deployment: ai-ml-service (3 replicas, GPU nodes)              │
│  Deployment: ocr-service (2 replicas)                           │
│  Deployment: voice-agent-service (3 replicas)                   │
│  Deployment: insurance-service (2 replicas)                     │
│  Deployment: notification-service (2 replicas)                  │
│  Deployment: consent-service (2 replicas)                       │
│  Deployment: research-service (2 replicas)                      │
│  StatefulSet: hapi-fhir-server (3 replicas)                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    PERSISTENT STORAGE                            │
│  RDS PostgreSQL │ ElastiCache Redis │ EFS │ S3                  │
└─────────────────────────────────────────────────────────────────┘
```

**Resource Allocation**:
- **User Service**: 2 CPU, 4GB RAM per pod
- **Health Record Service**: 2 CPU, 4GB RAM per pod
- **AI/ML Service**: 4 CPU, 16GB RAM, 1 GPU per pod
- **OCR Service**: 4 CPU, 8GB RAM per pod
- **Voice Agent**: 2 CPU, 4GB RAM per pod

**Auto-scaling**:
- Horizontal Pod Autoscaler (HPA)
- Target CPU utilization: 70%
- Min replicas: 2, Max replicas: 10
- Scale-up: 30 seconds, Scale-down: 5 minutes

### 10.3 Environment Strategy

**Environments**:
1. **Development**: Single-node cluster, minimal resources
2. **Staging**: Production-like, reduced capacity
3. **Production**: Multi-AZ, high availability

**Environment Configuration**:
| Environment | Nodes | Database | Blockchain | Monitoring |
|-------------|-------|----------|------------|------------|
| Development | 1 | PostgreSQL (local) | Sepolia | Basic |
| Staging | 3 | RDS (single-AZ) | Sepolia | Full |
| Production | 9+ | RDS (multi-AZ) | Polygon | Full |

### 10.4 CI/CD Pipeline

```
Code Commit (GitHub)
    ↓
GitHub Actions Trigger
    ↓
Build & Unit Tests
    ↓
Docker Image Build
    ↓
Push to Container Registry (ECR)
    ↓
Integration Tests
    ↓
Security Scan (Trivy, Snyk)
    ↓
Deploy to Staging
    ↓
Automated E2E Tests
    ↓
Manual Approval
    ↓
Deploy to Production (Blue-Green)
    ↓
Health Checks & Smoke Tests
    ↓
Traffic Switch
```

**Deployment Strategies**:
- **Blue-Green Deployment**: Zero-downtime deployments
- **Canary Releases**: Gradual rollout (10% → 50% → 100%)
- **Rollback**: Automated rollback on health check failures

---

## 11. Data Flow Diagrams

### 11.1 Patient Registration Flow

```
Patient → Mobile App
    ↓
Enter Aadhaar Number
    ↓
API Gateway → User Service
    ↓
User Service → ABDM API (Generate OTP)
    ↓
OTP sent to Patient's Mobile
    ↓
Patient enters OTP
    ↓
User Service → ABDM API (Verify OTP)
    ↓
Create Health ID
    ↓
User Service → PostgreSQL (Store user data)
    ↓
User Service → Blockchain (Log registration)
    ↓
Return JWT Token to Patient
```

### 11.2 Document Digitization Flow

```
Patient uploads scanned document → Mobile App
    ↓
API Gateway → OCR Service
    ↓
OCR Service → MinIO (Store original)
    ↓
OCR Service → Image Preprocessing
    ↓
OCR Service → Tesseract (Text extraction)
    ↓
OCR Service → NLP Model (Entity extraction)
    ↓
OCR Service → Medical Term Mapping (SNOMED CT)
    ↓
OCR Service → FHIR Resource Generation
    ↓
OCR Service → Health Record Service (Store structured data)
    ↓
Health Record Service → PostgreSQL
    ↓
Health Record Service → Blockchain (Log creation)
    ↓
If confidence < 80% → Add to Review Queue
    ↓
Notify Patient (Document processed)
```

### 11.3 AI Disease Prediction Flow

```
Doctor requests risk assessment → Web Portal
    ↓
API Gateway → AI/ML Service
    ↓
AI/ML Service → Health Record Service (Fetch patient data)
    ↓
AI/ML Service → Health Record Service (Fetch family history)
    ↓
AI/ML Service → Feature Engineering
    ↓
AI/ML Service → Load ML Models (Disease Predictor, Family Risk Assessor)
    ↓
AI/ML Service → Run Inference
    ↓
AI/ML Service → Generate Risk Scores
    ↓
AI/ML Service → Generate Recommendations
    ↓
AI/ML Service → PostgreSQL (Store prediction)
    ↓
Return Risk Report to Doctor
    ↓
Notification Service → Notify Patient (if high risk)
```

### 11.4 Voice Agent Symptom Assessment Flow

```
Patient speaks symptoms → Mobile App
    ↓
Mobile App → Speech-to-Text (Whisper API)
    ↓
API Gateway → Voice Agent Service
    ↓
Voice Agent → Language Detection
    ↓
Voice Agent → Intent Classification (Symptom reporting)
    ↓
Voice Agent → Entity Extraction (Symptoms, duration, severity)
    ↓
Voice Agent → Medical Term Mapping (SNOMED CT)
    ↓
Voice Agent → Triage Algorithm (Severity assessment)
    ↓
Decision:
  - Minor → Generate home remedy suggestions
  - Moderate → Recommend specialist + Book appointment
  - Severe → Emergency escalation + Nearest hospital
    ↓
Voice Agent → Generate Response
    ↓
Voice Agent → Text-to-Speech (gTTS)
    ↓
Return audio response to Patient
    ↓
If appointment booked → Notification Service → Notify Doctor
    ↓
Voice Agent → PostgreSQL (Store conversation)
```

### 11.5 Insurance Claim Submission Flow

```
Patient initiates claim → Mobile App
    ↓
API Gateway → Insurance Service
    ↓
Insurance Service → Health Record Service (Fetch relevant records)
    ↓
Insurance Service → AI/ML Service (Validate completeness)
    ↓
If incomplete → Return missing document list
    ↓
Insurance Service → Compile documents (Medical records, bills, prescriptions)
    ↓
Insurance Service → AI/ML Service (Generate claim narrative)
    ↓
Insurance Service → Blockchain (Verify record authenticity)
    ↓
Insurance Service → AI/ML Service (Predict approval probability)
    ↓
Insurance Service → Insurance Company API (Submit claim)
    ↓
Insurance Service → Blockchain (Log claim submission)
    ↓
Insurance Service → PostgreSQL (Store claim)
    ↓
Notification Service → Notify Patient (Claim submitted)
    ↓
Periodic polling → Insurance Company API (Check status)
    ↓
On status change → Notification Service → Notify Patient
```

### 11.6 Research Data Access Flow

```
Researcher submits request → Research Portal
    ↓
API Gateway → Research Service
    ↓
Research Service → PostgreSQL (Store request)
    ↓
Manual Review → Ethical Review Board
    ↓
If approved → Research Service → Update request status
    ↓
Researcher executes query → Research Portal
    ↓
Research Service → Data Anonymization (Differential privacy)
    ↓
Research Service → PostgreSQL (Execute query on anonymized data)
    ↓
Research Service → Generate dataset
    ↓
Research Service → PostgreSQL (Log access)
    ↓
Return anonymized dataset to Researcher
    ↓
Research Service → Analytics (Track data usage)
```

---

## 12. Database Schema Design

### 12.1 Core Tables

**users**
```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aadhaar_hash VARCHAR(64) UNIQUE NOT NULL,
    health_id VARCHAR(50) UNIQUE,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(15) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    mfa_enabled BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);

CREATE INDEX idx_users_health_id ON users(health_id);
CREATE INDEX idx_users_phone ON users(phone);
```

**user_profiles**
```sql
CREATE TABLE user_profiles (
    profile_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender VARCHAR(20),
    blood_group VARCHAR(5),
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    pincode VARCHAR(10),
    emergency_contact_name VARCHAR(100),
    emergency_contact_phone VARCHAR(15),
    profile_picture_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_profiles_user_id ON user_profiles(user_id);
```

**health_records**
```sql
CREATE TABLE health_records (
    record_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    record_type VARCHAR(50) NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    record_date DATE NOT NULL,
    created_by UUID REFERENCES users(user_id),
    hospital_id UUID,
    fhir_resource_id VARCHAR(100),
    blockchain_hash VARCHAR(66),
    ipfs_hash VARCHAR(100),
    version INT DEFAULT 1,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_records_patient_id ON health_records(patient_id);
CREATE INDEX idx_records_date ON health_records(record_date);
CREATE INDEX idx_records_type ON health_records(record_type);
```

**diagnoses**
```sql
CREATE TABLE diagnoses (
    diagnosis_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    record_id UUID REFERENCES health_records(record_id) ON DELETE CASCADE,
    icd10_code VARCHAR(10) NOT NULL,
    diagnosis_name VARCHAR(255) NOT NULL,
    severity VARCHAR(20),
    status VARCHAR(20),
    onset_date DATE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_diagnoses_record_id ON diagnoses(record_id);
CREATE INDEX idx_diagnoses_icd10 ON diagnoses(icd10_code);
```

**prescriptions**
```sql
CREATE TABLE prescriptions (
    prescription_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    record_id UUID REFERENCES health_records(record_id) ON DELETE CASCADE,
    medication_name VARCHAR(255) NOT NULL,
    dosage VARCHAR(100),
    frequency VARCHAR(100),
    duration VARCHAR(100),
    instructions TEXT,
    prescribed_by UUID REFERENCES users(user_id),
    start_date DATE,
    end_date DATE,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_prescriptions_record_id ON prescriptions(record_id);
CREATE INDEX idx_prescriptions_patient ON prescriptions(record_id);
```

**family_health_history**
```sql
CREATE TABLE family_health_history (
    family_history_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    relation VARCHAR(50) NOT NULL,
    condition VARCHAR(255) NOT NULL,
    icd10_code VARCHAR(10),
    age_of_onset INT,
    is_deceased BOOLEAN DEFAULT false,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_family_history_patient ON family_health_history(patient_id);
CREATE INDEX idx_family_history_condition ON family_health_history(condition);
```

**consents**
```sql
CREATE TABLE consents (
    consent_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    granted_to_id UUID REFERENCES users(user_id),
    granted_to_type VARCHAR(50) NOT NULL,
    purpose VARCHAR(100) NOT NULL,
    data_types TEXT[] NOT NULL,
    valid_from TIMESTAMP NOT NULL,
    valid_until TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    blockchain_hash VARCHAR(66),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    revoked_at TIMESTAMP
);

CREATE INDEX idx_consents_patient ON consents(patient_id);
CREATE INDEX idx_consents_granted_to ON consents(granted_to_id);
CREATE INDEX idx_consents_status ON consents(status);
```

**insurance_claims**
```sql
CREATE TABLE insurance_claims (
    claim_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    policy_number VARCHAR(100) NOT NULL,
    insurer_id UUID,
    hospital_id UUID,
    claim_amount DECIMAL(10, 2) NOT NULL,
    claim_type VARCHAR(50),
    status VARCHAR(50) DEFAULT 'submitted',
    submission_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    approval_date TIMESTAMP,
    settlement_date TIMESTAMP,
    settlement_amount DECIMAL(10, 2),
    blockchain_hash VARCHAR(66),
    approval_probability DECIMAL(5, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_claims_patient ON insurance_claims(patient_id);
CREATE INDEX idx_claims_status ON insurance_claims(status);
```

**ocr_jobs**
```sql
CREATE TABLE ocr_jobs (
    job_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    document_url TEXT NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    confidence_score DECIMAL(5, 2),
    extracted_data JSONB,
    needs_review BOOLEAN DEFAULT false,
    processed_at TIMESTAMP,
    reviewed_at TIMESTAMP,
    reviewed_by UUID REFERENCES users(user_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_ocr_jobs_patient ON ocr_jobs(patient_id);
CREATE INDEX idx_ocr_jobs_status ON ocr_jobs(status);
CREATE INDEX idx_ocr_jobs_review ON ocr_jobs(needs_review);
```

**ai_predictions**
```sql
CREATE TABLE ai_predictions (
    prediction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    prediction_type VARCHAR(50) NOT NULL,
    model_version VARCHAR(20) NOT NULL,
    input_features JSONB NOT NULL,
    prediction_result JSONB NOT NULL,
    confidence_score DECIMAL(5, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_predictions_patient ON ai_predictions(patient_id);
CREATE INDEX idx_predictions_type ON ai_predictions(prediction_type);
```

**conversation_sessions**
```sql
CREATE TABLE conversation_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    language VARCHAR(10) NOT NULL,
    conversation_history JSONB NOT NULL,
    symptoms_extracted JSONB,
    triage_result VARCHAR(50),
    recommended_specialty VARCHAR(100),
    appointment_booked BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ended_at TIMESTAMP
);

CREATE INDEX idx_sessions_patient ON conversation_sessions(patient_id);
CREATE INDEX idx_sessions_created ON conversation_sessions(created_at);
```

**audit_logs**
```sql
CREATE TABLE audit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id UUID,
    ip_address INET,
    user_agent TEXT,
    status VARCHAR(20),
    blockchain_hash VARCHAR(66),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_resource ON audit_logs(resource_type, resource_id);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

### 12.2 Database Optimization

**Indexing Strategy**:
- Primary keys: UUID with B-tree index
- Foreign keys: Indexed for join performance
- Search fields: GIN index for JSONB columns
- Date fields: B-tree index for range queries

**Partitioning**:
- `health_records`: Partitioned by year
- `audit_logs`: Partitioned by month
- `ai_predictions`: Partitioned by quarter

**Replication**:
- Master-slave replication for read scaling
- Read replicas in multiple regions
- Automatic failover to standby

---

## 13. Technology Stack Details

### 13.1 Frontend Stack

**Web Application**:
- **Framework**: React 18.2+
- **Language**: TypeScript 5.0+
- **State Management**: Redux Toolkit, React Query
- **UI Library**: Material-UI (MUI) v5
- **Routing**: React Router v6
- **Forms**: React Hook Form + Yup validation
- **Charts**: Recharts, D3.js
- **HTTP Client**: Axios
- **Build Tool**: Vite
- **Testing**: Jest, React Testing Library, Cypress

**Mobile Application**:
- **Framework**: React Native 0.72+
- **Language**: TypeScript 5.0+
- **Navigation**: React Navigation v6
- **UI Components**: Native Base, React Native Paper
- **State Management**: Redux Toolkit
- **Storage**: AsyncStorage, MMKV
- **Camera**: react-native-camera
- **Biometrics**: react-native-biometrics
- **Voice**: react-native-voice
- **Testing**: Jest, Detox

### 13.2 Backend Stack

**Java Microservices**:
- **Framework**: Spring Boot 3.1+
- **Language**: Java 17
- **Security**: Spring Security, OAuth2, JWT
- **Data Access**: Spring Data JPA, Hibernate
- **API Documentation**: SpringDoc OpenAPI
- **Validation**: Jakarta Bean Validation
- **Testing**: JUnit 5, Mockito, TestContainers

**Python Microservices**:
- **Framework**: FastAPI 0.104+
- **Language**: Python 3.10+
- **Async**: asyncio, aiohttp
- **Data Validation**: Pydantic
- **ORM**: SQLAlchemy
- **Testing**: pytest, pytest-asyncio

**FHIR Server**:
- **Implementation**: HAPI-FHIR 6.8+
- **Version**: FHIR R4
- **Database**: PostgreSQL
- **Search**: Elasticsearch (optional)

### 13.3 AI/ML Stack

**Machine Learning**:
- **Deep Learning**: PyTorch 2.0+, TensorFlow 2.13+
- **Classical ML**: scikit-learn 1.3+, XGBoost 2.0+
- **NLP**: Hugging Face Transformers, spaCy 3.6+
- **Computer Vision**: OpenCV 4.8+, Pillow
- **Time Series**: Prophet, statsmodels
- **Graph ML**: PyTorch Geometric

**MLOps**:
- **Experiment Tracking**: MLflow 2.7+
- **Model Serving**: TorchServe, TensorFlow Serving
- **Feature Store**: Feast
- **Workflow**: Apache Airflow, Kubeflow
- **Monitoring**: Evidently AI, WhyLabs

**OCR & Speech**:
- **OCR**: Tesseract 5.0+, EasyOCR
- **Speech-to-Text**: OpenAI Whisper, Google Speech API
- **Text-to-Speech**: gTTS, Azure Speech Services
- **NLP**: BioBERT, ClinicalBERT

### 13.4 Database & Storage Stack

**Relational Database**:
- **Primary**: PostgreSQL 15+
- **Extensions**: pgcrypto, uuid-ossp, pg_trgm
- **Connection Pool**: HikariCP, pgBouncer
- **Migration**: Flyway, Liquibase

**Cache**:
- **In-Memory**: Redis 7.0+
- **Use Cases**: Session storage, rate limiting, caching
- **Clustering**: Redis Sentinel for HA

**Object Storage**:
- **Cloud**: AWS S3, Azure Blob Storage
- **Self-Hosted**: MinIO
- **CDN**: CloudFront, Cloudflare

**Decentralized Storage**:
- **Protocol**: IPFS (InterPlanetary File System)
- **Incentive Layer**: Filecoin
- **Pinning**: Pinata, Web3.Storage
- **Gateway**: Infura IPFS Gateway

**Message Queue**:
- **Primary**: RabbitMQ 3.12+
- **Alternative**: Apache Kafka (for high-throughput)
- **Patterns**: Pub/Sub, Work Queues, RPC

### 13.5 Blockchain Stack

**Smart Contract Development**:
- **Language**: Solidity 0.8.20+
- **Framework**: Hardhat 2.17+
- **Testing**: Hardhat, Chai, Waffle
- **Security**: Slither, MythX

**Blockchain Interaction**:
- **Library**: ethers.js 6.7+, web3.js
- **Wallet**: MetaMask, WalletConnect
- **Node Provider**: Infura, Alchemy

**Network**:
- **Mainnet**: Polygon (MATIC)
- **Testnet**: Sepolia, Mumbai
- **Explorer**: PolygonScan

### 13.6 DevOps Stack

**Containerization**:
- **Runtime**: Docker 24+
- **Orchestration**: Kubernetes 1.28+
- **Registry**: AWS ECR, Docker Hub

**CI/CD**:
- **Pipeline**: GitHub Actions, Jenkins
- **GitOps**: ArgoCD, Flux
- **Testing**: SonarQube, Snyk

**Monitoring & Observability**:
- **Metrics**: Prometheus, Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Tracing**: Jaeger, OpenTelemetry
- **APM**: New Relic, Datadog

**Infrastructure as Code**:
- **Provisioning**: Terraform 1.5+
- **Configuration**: Ansible
- **Secrets**: HashiCorp Vault, AWS Secrets Manager

---

## 14. Scalability & Performance Design

### 14.1 Horizontal Scaling Strategy

**Stateless Services**:
- All microservices designed to be stateless
- Session data stored in Redis (shared state)
- Easy to add/remove instances

**Load Balancing**:
- Application Load Balancer (ALB) at entry point
- Round-robin distribution
- Health check-based routing
- Sticky sessions for WebSocket connections

**Database Scaling**:
- Read replicas for read-heavy operations
- Connection pooling (max 100 connections per service)
- Query optimization and indexing
- Caching frequently accessed data

### 14.2 Caching Strategy

**Multi-Level Caching**:

1. **Browser Cache** (Client-side)
   - Static assets: 1 year
   - API responses: 5 minutes (with ETag)

2. **CDN Cache** (Edge)
   - Images, documents: 30 days
   - API responses (public): 1 hour

3. **Application Cache** (Redis)
   - User sessions: 24 hours
   - User profiles: 1 hour
   - Health records metadata: 30 minutes
   - FHIR resources: 15 minutes

4. **Database Cache** (PostgreSQL)
   - Query result cache
   - Materialized views for analytics

**Cache Invalidation**:
- Time-based expiration (TTL)
- Event-based invalidation (on data update)
- Cache-aside pattern for consistency

### 14.3 Performance Optimization

**API Optimization**:
- Response compression (gzip)
- Pagination for large datasets (page size: 20)
- Field filtering (return only requested fields)
- Batch API endpoints for multiple operations
- GraphQL for flexible data fetching

**Database Optimization**:
- Query optimization (EXPLAIN ANALYZE)
- Proper indexing strategy
- Denormalization for read-heavy tables
- Partitioning for large tables
- Connection pooling

**Asynchronous Processing**:
- OCR processing: Async with job queue
- Email/SMS notifications: Async
- Blockchain transactions: Async with confirmation polling
- Report generation: Async with download link

**Resource Optimization**:
- Image compression (WebP format)
- Lazy loading for images
- Code splitting for frontend
- Tree shaking for unused code
- Minification and bundling

### 14.4 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| API Response Time (p95) | < 200ms | Prometheus |
| API Response Time (p99) | < 500ms | Prometheus |
| Page Load Time | < 2s | Lighthouse |
| Database Query Time | < 50ms | PostgreSQL logs |
| Cache Hit Rate | > 80% | Redis metrics |
| Concurrent Users | 10,000+ | Load testing |
| Throughput | 5,000 req/s | Load testing |
| Error Rate | < 0.1% | Application logs |

---

## 15. Disaster Recovery & Backup Strategy

### 15.1 Backup Strategy

**Database Backups**:
- **Frequency**: 
  - Full backup: Daily at 2 AM IST
  - Incremental backup: Every 6 hours
  - Transaction logs: Continuous
- **Retention**: 
  - Daily backups: 30 days
  - Weekly backups: 3 months
  - Monthly backups: 1 year
- **Storage**: AWS S3 with versioning enabled
- **Encryption**: AES-256 encryption at rest

**File Storage Backups**:
- **Frequency**: Daily incremental, Weekly full
- **Retention**: 90 days
- **Storage**: S3 with lifecycle policies
- **Verification**: Monthly restore tests

**Blockchain Data**:
- No backup needed (immutable, distributed)
- Node redundancy for availability

