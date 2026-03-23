# ⏱️ EvoluaPonto - Enterprise Time Tracking & Compliance System (Architecture Showcase)

> **Disclaimer:** This repository serves as an architectural showcase. The full source code is proprietary. The diagrams, system design choices, and architectural patterns described here demonstrate my role as a software engineer in building this platform.

## 📌 Executive Summary
**EvoluaPonto** is a comprehensive, multi-tenant workforce management ecosystem. Designed to handle strict governmental labor regulations, the system ensures 100% data integrity through spatial validation (Geofencing), cryptographic hashing (SHA-256), digital signatures, and real-time auditing.

**My Role:** Co-Creator & Frontend/Backend Developer
**Core Stack:** C# .NET 8, PostgreSQL, React Native (Expo), Docker, Nginx.

---

## 🏗️ System Architecture & Domain Driven Design

The system follows a microservices-oriented approach, heavily relying on containerization to decouple business rules from delivery mechanisms.

### 1. Cryptographic Audit Trails & Sequential Compliance
To meet strict labor laws, the system must prove timestamps were not tampered with and maintain a continuous chronological ledger.
* **Implementation:** Every punch-in generates a unique SHA-256 hash and a strict Sequential Audit Number (NSR) isolated per tenant. Receipts are dynamically generated via `QuestPDF` and digitally signed utilizing the `BouncyCastle` cryptography library.
* **Export Engines:** Engineered complex data parsers capable of generating standardized flat files (AFD/AEJ) required by federal tax and labor audit systems.

### 2. Multi-Tenant Data Isolation (PostgreSQL RLS)
Beyond application-level Role-Based Access Control (RBAC) utilizing JWT Bearer Authentication, data isolation between different corporate clients (tenants) is enforced directly at the database engine level.
* **Implementation:** PostgreSQL Row Level Security (RLS) policies were implemented to guarantee that queries can never leak cross-tenant data, providing defense-in-depth even if the application layer is compromised.

### 3. Spatial Validation & Geofencing Engine
* **Logic:** When an employee registers a punch-in, the React Native mobile app calculates the exact coordinates. The .NET Backend processes the Haversine distance between the device's location and the allowed perimeter of the company's facility. Out-of-bounds requests are systematically blocked.

---

## 🗄️ Core Database Schema (Entity-Relationship)

```mermaid
erDiagram
    %% Corporate Core
    Empresas ||--o{ Estabelecimentos : "owns"
    Empresas ||--o{ EventosProvas : "organizes"

    %% Time Tracking & Compliance
    Estabelecimentos ||--o{ Funcionarios : "employs"
    Estabelecimentos ||--o{ Escalas : "defines"
    Estabelecimentos ||--o{ Feriados : "controls"
    Estabelecimentos ||--o{ FeriadosPersonalizados : "controls"
    
    Escalas ||--o{ EscalaDias : "details routine"
    Escalas ||--o{ Funcionarios : "assigns"
    
    Funcionarios ||--o{ RegistroPontos : "creates"
    
    %% Educational / Event Management Module
    EventosProvas ||--o{ LocalProva : "occurs in"
    LocalProva ||--o{ SalaProva : "divided into"
    EventosProvas ||--o{ InscricaoAluno : "receives"
    
    %% Identity & Auth
    Usuarios {
        uuid id PK
        string email
        string role
    }

    Empresas {
        uuid id PK
        string cnpj
        string razao_social
    }

    Estabelecimentos {
        uuid id PK
        uuid empresa_id FK
        float center_latitude
        float center_longitude
        int raio_permitido_metros
    }

    Funcionarios {
        uuid id PK
        uuid estabelecimento_id FK
        uuid escala_id FK
        string cpf
        string matricula
    }

    Escalas {
        uuid id PK
        uuid estabelecimento_id FK
        string descricao
    }

    RegistroPontos {
        uuid id PK
        uuid funcionario_id FK
        datetime data_hora
        string hash_sha256
        string comprovante_url
        int nsr "Sequential Audit Number"
    }
