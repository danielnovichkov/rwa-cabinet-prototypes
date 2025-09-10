# RWA Capital Cabinet System - Developer Specification

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Authentication System](#authentication-system)
4. [Project Creation System](#project-creation-system)
5. [Technical Requirements](#technical-requirements)
6. [API Specifications](#api-specifications)
7. [Database Schema](#database-schema)
8. [Security & Compliance](#security--compliance)
9. [Implementation Guidelines](#implementation-guidelines)

---

## 🎯 Project Overview

### Purpose
The RWA Capital Cabinet System is a comprehensive platform for real estate tokenization projects, enabling both retail investors and corporate clients to participate in the tokenization ecosystem.

### Key Features
- **Dual User Authentication** - Separate flows for retail investors and corporate clients
- **Project Creation Workflow** - Complete tokenization project setup and submission
- **Document Management** - Secure file upload and processing for compliance
- **Internal Audit System** - Project review and approval workflow

### Live Prototypes
**GitHub Pages (Recommended):**
- **Registration & Login:** [View Prototype](https://danielnovichkov.github.io/rwa-cabinet-prototypes/registration-login-prototype.html)
- **Project Creation:** [View Prototype](https://danielnovichkov.github.io/rwa-cabinet-prototypes/project-creation-prototype.html)

**Alternative Links:**
- **Registration & Login:** [View Prototype](https://raw.githack.com/danielnovichkov/rwa-cabinet-prototypes/main/registration-login-prototype.html)
- **Project Creation:** [View Prototype](https://raw.githack.com/danielnovichkov/rwa-cabinet-prototypes/main/project-creation-prototype.html)

---

## 🏗️ System Architecture

### Frontend Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    RWA Capital Cabinet                      │
├─────────────────────────────────────────────────────────────┤
│  Authentication Layer                                       │
│  ├── Retail Investor Auth                                   │
│  └── Corporate Client Auth                                  │
├─────────────────────────────────────────────────────────────┤
│  Project Management Layer                                   │
│  ├── Project Creation                                       │
│  ├── Document Upload                                        │
│  └── Status Tracking                                        │
├─────────────────────────────────────────────────────────────┤
│  Compliance & Audit Layer                                   │
│  ├── KYC/AML Processing                                     │
│  ├── Document Verification                                  │
│  └── Internal Review                                        │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack
- **Frontend:** React.js / Vue.js with TypeScript
- **Backend:** Node.js with Express.js / Python with FastAPI
- **Database:** PostgreSQL with Redis for caching
- **File Storage:** AWS S3 / Google Cloud Storage
- **Authentication:** JWT with refresh tokens
- **Document Processing:** AWS Textract / Google Document AI

---

## 🔐 Authentication System

### User Types

#### 1. Retail Investors
**Registration Requirements:**
- Personal information (name, email, phone)
- Country of residence
- Password with complexity requirements
- Terms of service acceptance
- Newsletter subscription (optional)

**Login Flow:**
- Email/password authentication
- Remember me functionality
- Password reset capability

#### 2. Corporate Clients
**Registration Requirements:**
- Company information (name, tax ID, legal address)
- Contact person details
- Asset type for tokenization
- Approximate asset value
- KYC/AML compliance confirmation
- Terms of service acceptance

**Login Flow:**
- Email/password authentication
- Corporate account management
- Multi-user access (future enhancement)

### Authentication API Endpoints

```typescript
// Registration
POST /api/auth/register/retail
POST /api/auth/register/corporate

// Login
POST /api/auth/login

// Password Management
POST /api/auth/forgot-password
POST /api/auth/reset-password
POST /api/auth/change-password

// Token Management
POST /api/auth/refresh-token
POST /api/auth/logout
```

### Security Requirements
- **Password Policy:** Minimum 8 characters, mixed case, numbers, special characters
- **Rate Limiting:** 5 attempts per 15 minutes for login
- **Session Management:** JWT with 24-hour expiration, refresh tokens
- **Email Verification:** Required for account activation
- **2FA:** Optional for corporate accounts

---

## 🏢 Project Creation System

### Project Creation Workflow

#### Step 1: Project Basic Information
```typescript
interface ProjectBasicInfo {
  name: string;
  description: string;
  propertyType: 'residential' | 'commercial' | 'office' | 'retail' | 'industrial' | 'hospitality' | 'mixed-use' | 'land';
  status: 'existing' | 'development' | 'planned';
}
```

#### Step 2: Property Details
```typescript
interface PropertyDetails {
  address: string;
  totalArea: number; // sq ft
  unitsFloors: string;
  yearBuilt?: number;
  lastRenovation?: number;
  occupancyRate?: number; // percentage
}
```

#### Step 3: Financial Information
```typescript
interface FinancialInfo {
  valuation: number; // USD
  valuationDate: string; // ISO date
  annualRentalIncome?: number;
  annualOperatingExpenses?: number;
  debtInformation?: string;
}
```

#### Step 4: Tokenization Parameters
```typescript
interface TokenizationParams {
  tokenizationPercentage: number; // 1-100%
  totalTokens: number;
  tokenPrice: number; // USD
  minimumInvestment: number; // USD
  investmentTerms?: string;
}
```

#### Step 5: Legal & Compliance
```typescript
interface LegalCompliance {
  ownershipStructure: 'individual' | 'corporation' | 'llc' | 'partnership' | 'trust' | 'reit';
  regulatoryJurisdiction: string;
  complianceNotes?: string;
}
```

#### Step 6: Document Upload
```typescript
interface DocumentUpload {
  titleDeed: File[];
  valuationReport: File[];
  insuranceDocuments?: File[];
  financialStatements?: File[];
  propertyPhotos?: File[];
  legalDocuments?: File[];
}
```

#### Step 7: Contact Information
```typescript
interface ContactInfo {
  primaryContactName: string;
  title: string;
  email: string;
  phone: string;
  company?: string;
}
```

### Project API Endpoints

```typescript
// Project Management
POST /api/projects
GET /api/projects
GET /api/projects/:id
PUT /api/projects/:id
DELETE /api/projects/:id

// Document Upload
POST /api/projects/:id/documents
GET /api/projects/:id/documents
DELETE /api/projects/:id/documents/:docId

// Project Status
POST /api/projects/:id/submit
GET /api/projects/:id/status
POST /api/projects/:id/audit-review
```

---

## 💾 Database Schema

### Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  user_type ENUM('retail', 'corporate') NOT NULL,
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Retail Investors Table
```sql
CREATE TABLE retail_investors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  country VARCHAR(100) NOT NULL,
  newsletter_subscribed BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Corporate Clients Table
```sql
CREATE TABLE corporate_clients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  company_name VARCHAR(255) NOT NULL,
  tax_id VARCHAR(50) NOT NULL,
  legal_address TEXT NOT NULL,
  contact_person VARCHAR(100) NOT NULL,
  position VARCHAR(100) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  asset_type VARCHAR(100) NOT NULL,
  asset_value_range VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Projects Table
```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  name VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  property_type VARCHAR(50) NOT NULL,
  status VARCHAR(50) NOT NULL,
  address TEXT NOT NULL,
  total_area INTEGER,
  units_floors VARCHAR(100),
  year_built INTEGER,
  last_renovation INTEGER,
  occupancy_rate INTEGER,
  valuation DECIMAL(15,2) NOT NULL,
  valuation_date DATE NOT NULL,
  annual_rental_income DECIMAL(15,2),
  annual_operating_expenses DECIMAL(15,2),
  debt_information TEXT,
  tokenization_percentage INTEGER NOT NULL,
  total_tokens BIGINT NOT NULL,
  token_price DECIMAL(10,2) NOT NULL,
  minimum_investment DECIMAL(10,2) NOT NULL,
  investment_terms TEXT,
  ownership_structure VARCHAR(50) NOT NULL,
  regulatory_jurisdiction VARCHAR(100) NOT NULL,
  compliance_notes TEXT,
  primary_contact_name VARCHAR(100) NOT NULL,
  contact_title VARCHAR(100) NOT NULL,
  contact_email VARCHAR(255) NOT NULL,
  contact_phone VARCHAR(20) NOT NULL,
  contact_company VARCHAR(255),
  project_status ENUM('draft', 'submitted', 'under_review', 'approved', 'rejected') DEFAULT 'draft',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Documents Table
```sql
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id),
  document_type VARCHAR(50) NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_path VARCHAR(500) NOT NULL,
  file_size INTEGER NOT NULL,
  mime_type VARCHAR(100) NOT NULL,
  uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🔒 Security & Compliance

### Data Protection
- **Encryption:** All sensitive data encrypted at rest and in transit
- **GDPR Compliance:** Data retention policies and user consent management
- **PCI DSS:** If payment processing is implemented
- **SOC 2:** Security controls and monitoring

### File Upload Security
- **File Type Validation:** Only allowed file types (PDF, JPG, PNG)
- **File Size Limits:** Maximum 10MB per file
- **Virus Scanning:** All uploaded files scanned for malware
- **Secure Storage:** Files stored in encrypted cloud storage

### Audit Trail
- **User Actions:** Log all user interactions and changes
- **Document Access:** Track who accessed which documents
- **System Events:** Log authentication, authorization, and system events
- **Compliance Reporting:** Generate audit reports for regulatory requirements

---

## 🚀 Implementation Guidelines

### Frontend Development

#### Component Structure
```
src/
├── components/
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── RegisterForm.tsx
│   │   └── UserTypeSelector.tsx
│   ├── projects/
│   │   ├── ProjectForm.tsx
│   │   ├── DocumentUpload.tsx
│   │   └── ProgressTracker.tsx
│   └── common/
│       ├── FileUpload.tsx
│       ├── FormValidation.tsx
│       └── LoadingSpinner.tsx
├── pages/
│   ├── LoginPage.tsx
│   ├── RegisterPage.tsx
│   ├── ProjectCreationPage.tsx
│   └── DashboardPage.tsx
└── services/
    ├── api.ts
    ├── auth.ts
    └── fileUpload.ts
```

#### State Management
- **Redux Toolkit** for global state management
- **React Query** for server state and caching
- **Formik** or **React Hook Form** for form management

### Backend Development

#### API Structure
```
src/
├── controllers/
│   ├── authController.ts
│   ├── projectController.ts
│   └── documentController.ts
├── services/
│   ├── authService.ts
│   ├── projectService.ts
│   ├── documentService.ts
│   └── emailService.ts
├── middleware/
│   ├── auth.ts
│   ├── validation.ts
│   └── fileUpload.ts
├── models/
│   ├── User.ts
│   ├── Project.ts
│   └── Document.ts
└── routes/
    ├── auth.ts
    ├── projects.ts
    └── documents.ts
```

### Testing Strategy
- **Unit Tests:** Jest for component and service testing
- **Integration Tests:** API endpoint testing
- **E2E Tests:** Cypress for user flow testing
- **Security Tests:** OWASP ZAP for security scanning

### Deployment
- **Frontend:** Vercel / Netlify for static hosting
- **Backend:** AWS ECS / Google Cloud Run for containerized deployment
- **Database:** AWS RDS / Google Cloud SQL for managed database
- **CDN:** CloudFront / CloudFlare for content delivery

---

## 📊 Performance Requirements

### Response Times
- **Page Load:** < 2 seconds
- **API Response:** < 500ms for most endpoints
- **File Upload:** Progress indication for large files
- **Search/Filter:** < 1 second for project listings

### Scalability
- **Concurrent Users:** Support 1000+ simultaneous users
- **File Storage:** Scalable cloud storage solution
- **Database:** Read replicas for improved performance
- **Caching:** Redis for session and data caching

---

## 🔄 Development Phases

### Phase 1: Core Authentication (2-3 weeks)
- User registration and login
- Basic user management
- Email verification
- Password reset functionality

### Phase 2: Project Creation (3-4 weeks)
- Project form implementation
- Document upload system
- Form validation and progress tracking
- Draft saving functionality

### Phase 3: Audit System (2-3 weeks)
- Internal review workflow
- Status tracking and notifications
- Document verification
- Approval/rejection process

### Phase 4: Enhancements (2-3 weeks)
- Advanced search and filtering
- User dashboard improvements
- Mobile responsiveness optimization
- Performance optimizations

---

## 📞 Support & Maintenance

### Development Team
- **Frontend Developer:** React.js/TypeScript specialist
- **Backend Developer:** Node.js/Python specialist
- **DevOps Engineer:** AWS/Google Cloud specialist
- **QA Engineer:** Testing and quality assurance

### Monitoring & Logging
- **Application Monitoring:** New Relic / DataDog
- **Error Tracking:** Sentry for error monitoring
- **Log Management:** ELK Stack or cloud logging
- **Uptime Monitoring:** Pingdom or similar service

---

## 📚 Additional Resources

- **GitHub Repository:** [rwa-cabinet-prototypes](https://github.com/danielnovichkov/rwa-cabinet-prototypes)
- **Live Prototypes:** [View Interactive Prototypes](https://danielnovichkov.github.io/rwa-cabinet-prototypes/)
- **Design System:** To be created based on prototype feedback
- **API Documentation:** To be generated using Swagger/OpenAPI

---

*This specification is a living document and will be updated as requirements evolve and feedback is incorporated from stakeholders and development team.*
