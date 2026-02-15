# SwasthChain – SURAKSHA
## System Design Document

**Version:** 1.0  
**Date:** February 15, 2026

---

## 1. Architecture Overview

### Architecture Style
Microservices-based architecture with:
- Service independence with dedicated databases
- API-first design (REST/GraphQL)
- Event-driven communication
- Cloud-native containerization
- Polyglot persistence

### Design Principles
- Security by design
- Privacy first with consent-based access
- FHIR interoperability
- Horizontal scalability
- Comprehensive observability

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              CLIENT LAYER                                    │
│  Web (React) │ Mobile (React Native) │ Doctor Portal       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│         NGINX REVERSE PROXY (Load Balancer)                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│              MICROSERVICES LAYER (Node.js/Python)            │
│ User │ Health Record │ AI/ML │ OCR │ Voice │ Insurance      │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│              DATA & STORAGE LAYER                            │
│         MySQL Database │ AWS S3 (File Storage)              │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│           EXTERNAL INTEGRATIONS                              │
│ ABDM │ Insurance APIs │ Hospital Systems                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Core Microservices

### 3.1 User Service
**Tech**: Node.js (Express.js), MySQL (Sequelize)  
**Responsibilities**: Authentication, authorization, profile management  
**Key APIs**:
- `POST /api/v1/users/register` - User registration
- `POST /api/v1/users/login` - Authentication
- `GET /api/v1/users/{userId}/profile` - Get profile
- `POST /api/v1/users/verify-aadhaar` - Aadhaar verification

### 3.2 Health Record Service
**Tech**: Node.js (Express.js), MySQL  
**Responsibilities**: CRUD operations, medical history, timeline  
**Key APIs**:
- `POST /api/v1/records` - Create record
- `GET /api/v1/records/{recordId}` - Get record
- `GET /api/v1/patients/{patientId}/timeline` - Medical timeline
- `POST /api/v1/records/{recordId}/documents` - Upload document

### 3.3 OCR Service
**Tech**: FastAPI (Python), Tesseract, spaCy  
**Responsibilities**: Document digitization, data extraction  
**Key APIs**:
- `POST /api/v1/ocr/process` - Process document
- `GET /api/v1/ocr/jobs/{jobId}` - Get status

**Processing Pipeline**:
1. Document upload → AWS S3
2. Image preprocessing
3. OCR extraction (Tesseract)
4. NLP entity recognition
5. Medical term mapping
6. Confidence scoring

### 3.4 AI/ML Service
**Tech**: FastAPI (Python), scikit-learn, TensorFlow  
**Responsibilities**: Predictions, pattern analysis, recommendations  
**Key APIs**:
- `POST /api/v1/ai/predict-disease-risk` - Risk prediction
- `POST /api/v1/ai/analyze-patterns` - Pattern analysis
- `POST /api/v1/ai/family-risk-assessment` - Family history analysis

**ML Models**:
1. **Disease Risk Predictor**: Random Forest / Logistic Regression
2. **Pattern Analyzer**: Time series analysis
3. **Symptom NLP**: spaCy NER

### 3.5 Voice Agent Service
**Tech**: FastAPI (Python), Google Speech API, gTTS  
**Responsibilities**: Symptom assessment, triage  
**Key APIs**:
- `POST /api/v1/voice/process-audio` - Process voice input
- `POST /api/v1/voice/book-appointment` - Book appointment

### 3.6 Insurance Service
**Tech**: Node.js (Express.js), MySQL  
**Responsibilities**: Claim processing, verification  
**Key APIs**:
- `POST /api/v1/insurance/claims` - Submit claim
- `GET /api/v1/insurance/claims/{claimId}` - Get status

### 3.7 FHIR Service
**Tech**: Node.js (Express.js), MySQL  
**Responsibilities**: FHIR-compliant data exchange  
**Supported Resources**: Patient, Observation, Condition, MedicationRequest

---

## 4. Data Architecture

### Database Strategy

**MySQL (Primary)**:
- User profiles, authentication
- Health records metadata
- Transactional data
- Session management

**AWS S3 (Object Storage)**:
- Medical documents
- Medical images
- Backups

### Key Database Tables

**users**
```sql
CREATE TABLE users (
    user_id CHAR(36) PRIMARY KEY,
    aadhaar_hash VARCHAR(64) UNIQUE NOT NULL,
    health_id VARCHAR(50) UNIQUE,
    email VARCHAR(255),
    phone VARCHAR(15) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

**health_records**
```sql
CREATE TABLE health_records (
    record_id CHAR(36) PRIMARY KEY,
    patient_id CHAR(36) NOT NULL,
    record_type VARCHAR(50) NOT NULL,
    title VARCHAR(255) NOT NULL,
    record_date DATE NOT NULL,
    fhir_resource_id VARCHAR(100),
    document_hash VARCHAR(64),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES users(user_id)
) ENGINE=InnoDB;
```

**consents**
```sql
CREATE TABLE consents (
    consent_id CHAR(36) PRIMARY KEY,
    patient_id CHAR(36) NOT NULL,
    granted_to_id CHAR(36),
    purpose VARCHAR(100) NOT NULL,
    data_types JSON NOT NULL,
    valid_from TIMESTAMP NOT NULL,
    valid_until TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    FOREIGN KEY (patient_id) REFERENCES users(user_id)
) ENGINE=InnoDB;
```

---

## 5. API Design

### Standards
- Protocol: REST over HTTPS
- Format: JSON
- Versioning: URI (/api/v1/)
- Authentication: JWT Bearer tokens
- Rate Limiting: 1000 req/hour per user

### Request/Response Format

**Success Response**:
```json
{
  "status": "success",
  "data": { /* payload */ },
  "metadata": { "page": 1, "totalRecords": 100 }
}
```

**Error Response**:
```json
{
  "status": "error",
  "error": {
    "code": "INVALID_INPUT",
    "message": "Patient ID is required"
  }
}
```

---

## 6. Security Architecture

### Security Layers
1. Application Security (Input validation, OWASP)
2. Authentication & Authorization (JWT, RBAC, MFA)
3. API Security (Rate limiting, API keys)
4. Data Encryption (AES-256 at rest, TLS 1.3)
5. Network Security (VPC, Firewall, WAF)

### Encryption
- **At Rest**: AES-256 (MySQL TDE, S3 encryption)
- **In Transit**: TLS 1.3
- **Sensitive Fields**: Application-level encryption
- **Key Management**: AWS KMS/Azure Key Vault

### Access Control (RBAC)
- **Patient**: View own records, grant consent
- **Doctor**: View consented records, create records
- **Hospital Admin**: Manage hospital users
- **Insurance Officer**: View claim-related records
- **Researcher**: Access anonymized data

---

## 7. AI/ML Architecture

### ML Pipeline
```
Data Collection → Preprocessing → Feature Engineering
    → Model Training → Model Registry (MLflow)
    → Model Serving → Monitoring
```

### Feature Engineering
- Demographics: Age, gender, BMI, location
- Clinical: Vitals, lab values, medications, diagnoses
- Family History: Disease occurrence, age of onset
- Temporal: Time since last visit, trends
- Derived: Risk scores, comorbidity indices

### Model Performance
- Disease Risk Predictor: AUC-ROC > 0.85
- Pattern Analyzer: 90% accuracy
- Family Risk Assessor: 80% accuracy
- Symptom NLP: 92% entity extraction

---

## 8. Deployment Architecture

### AWS EC2 Setup
```
Internet → Route 53 (DNS) → EC2 Load Balancer
    → EC2 Instances (Node.js Services)
    → RDS MySQL Database
    → S3 Bucket (File Storage)
```

### EC2 Instance Configuration
- **Web Server**: t3.medium (2 vCPU, 4GB RAM) - Node.js services
- **AI Server**: t3.xlarge (4 vCPU, 16GB RAM) - Python AI/ML services
- **Database**: RDS MySQL t3.medium
- **Storage**: S3 Standard

### Environments
- **Development**: Local MySQL, local file storage
- **Production**: AWS EC2 + RDS + S3

### Deployment Process
```
Code Commit → GitHub → Manual Deploy to EC2
    → SSH into EC2 → Pull code → npm install
    → Restart Node.js services (PM2)
```

---

## 9. Scalability & Performance

### Scaling Strategy
- Vertical scaling: Upgrade EC2 instance types as needed
- Load balancing: AWS ELB for distributing traffic
- Database optimization: Proper indexing, query optimization

### Performance Targets
- API response: < 500ms
- Page load: < 3s
- Database query: < 100ms
- Concurrent users: 1,000+

---

## 10. Disaster Recovery

### Backup Strategy
- **Database**: Daily full, 6-hour incremental, 30-day retention
- **Files**: Daily incremental, weekly full, 90-day retention
- **Storage**: S3 with versioning, Glacier for archival

### Recovery Objectives
- RPO (Recovery Point Objective): < 1 hour
- RTO (Recovery Time Objective): < 4 hours
- Uptime SLA: 99.9%

---

## 11. Technology Stack Summary

| Layer | Technology |
|-------|-----------|
| Frontend | React 18+, React Native, JavaScript |
| Backend | Node.js, Express.js, JavaScript |
| AI/ML | Python, FastAPI, scikit-learn, TensorFlow |
| Database | MySQL 8.0+, Sequelize ORM |
| Storage | AWS S3 |
| Hosting | AWS EC2, RDS MySQL |
| Load Balancer | NGINX, AWS ELB |
| Version Control | GitHub |
| Process Manager | PM2 (for Node.js) |

---

## 12. Integration Points

### ABDM Integration
- Health ID creation and verification
- Healthcare Professional Registry (HPR)
- Health Facility Registry (HFR)
- Consent-based Health Data Exchange

### Insurance Integration
- REST APIs for eligibility verification
- Claim submission and tracking
- Real-time status updates

### Hospital Integration
- FHIR APIs for modern systems
- HL7 v2 for legacy systems
- Bidirectional sync (demographics, appointments, results)

---

**End of Document**
