# SWASTH SURAKSHA 🏥

**Smart Wellness Analytics & Secure Technology for Structured Unified Record Architecture**

A unified digital health record platform for India, enabling secure, interoperable, and AI-powered healthcare delivery.

---

## 🎯 Problem Statement

India's healthcare system faces critical challenges:
- Patient records scattered across hospitals
- Decades of paper-based records remain undigitized
- Manual insurance claim verification is time-consuming
- Critical patient info unavailable during emergencies
- No systematic tracking of family medical history

---

## ✨ Key Features

- 🔐 **Aadhaar-Based Authentication** - Secure login via ABDM integration
- 📄 **Document Digitization** - OCR-powered conversion of paper records to structured data
- 🤖 **AI Disease Prediction** - Risk assessment using family history and health patterns
- 🗣️ **Voice Agent** - Multi-language symptom checker and appointment booking
- 💼 **Insurance Automation** - AI-assisted claim processing and verification
- 🏥 **FHIR Interoperability** - Seamless data exchange with hospitals

---

## 🛠️ Technology Stack

**Frontend**: React  
**Backend**: Node.js, Express.js, MySQL  
**AI/ML**: Python, FastAPI, scikit-learn, TensorFlow  
**Storage**: AWS S3  
**Hosting**: AWS EC2, RDS MySQL  

---

## � Quick Start

### Prerequisites
- Node.js 18+
- Python 3.10+
- MySQL 8.0+

### Installation

```bash
# Clone repository
git clone https://github.com/your-org/swasth-suraksha.git
cd swasth-suraksha

# Backend setup
cd backend
npm install
cp .env.example .env
npm start

# AI services setup
cd ../ai-services
pip install -r requirements.txt
uvicorn main:app --reload

# Frontend setup
cd ../frontend
npm install
npm start
```

---

## 📁 Project Structure

```
swasth-suraksha/
├── backend/           # Node.js services
├── ai-services/       # Python AI/ML services
├── frontend/          # React web app
└── docs/              # Documentation
```

---

## � Documentation

- [Requirements Document](docs/requirements.md)
- [Design Document](docs/design.md)

---

## � Team

- [Nishant Asnani] - Project Lead
- [Nishant Asnani] - Backend Developer
- [Deep Bhat, Manmeet Patel] - AI/ML Engineer
- [Vansh Shinde] - Frontend Developer

---

## � License

MIT License

---

**Made with ❤️ for India's Healthcare**
