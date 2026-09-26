# Wildfire Operations Center (EOC) — Documentation Master Index

Welcome to the official **Project Handover & Documentation Package** for the **Edge-to-Cloud IoT Early Wildfire Detection and Tactical Operations Center Platform**.

This documentation package is curated and verified against the live codebase for a final-year B.Tech engineering student preparing for project demonstration, viva defense, and formal handover.

---

## 📑 Core Documentation Modules

| Section | Document Title | Description | Direct Link |
| :---: | :--- | :--- | :---: |
| **00** | **Handover Root Overview** | System quickstart, 3-minute run guide, architecture diagram, and summary. | [README.html](./README.html) |
| **01** | **Project Overview** | Executive summary, problem definition, objectives, architecture, and scope. | [Project_Overview.html](./01_Project_Overview/Project_Overview.html) |
| **02** | **Technical Documentation** | Hardware schematics, GPIO pinouts, LoRa RF framing, Python bridge, and DB triggers. | [Technical_Documentation.html](./02_Technical_Documentation/Technical_Documentation.html) |
| **03** | **User Operations Manual** | Tactical operations manual, GIS map navigation, incident workflows, and audio alerts. | [User_Manual.html](./03_User_Manual/User_Manual.html) |
| **04** | **Installation & Setup Guide** | Complete guide to flashing ESP32 firmware, configuring Supabase, and running the bridge. | [Installation_and_Setup_Guide.html](./04_Installation/Installation_and_Setup_Guide.html) |
| **05** | **Database Documentation** | PostgreSQL 15 schema, ER diagrams, PL/pgSQL stored procedures, deduplication, and indexes. | [Database_Documentation.html](./05_Database/Database_Documentation.html) |
| **06** | **REST API Specification** | Full REST API specification with endpoints, request/response bodies, and status codes. | [API_Documentation.html](./06_API/API_Documentation.html) |
| **07** | **Deployment Guide** | Vercel serverless deployment, Supabase Realtime setup, and Raspberry Pi systemd bridge service. | [Deployment_Guide.html](./07_Deployment/Deployment_Guide.html) |
| **08** | **Configuration & Env Guide** | Reference for all environment variables across web frontend (`.env.local`) and bridge (`bridge/.env`). | [Configuration_and_Environment_Guide.html](./08_Configuration/Configuration_and_Environment_Guide.html) |
| **09** | **Known Issues & Limitations** | Transparent documentation of bugs, physical constraints, and future enhancements. | [Known_Issues_and_Limitations.html](./09_Known_Issues/Known_Issues_and_Limitations.html) |
| **11** | **Project Handover & Sign-Off** | Formal deliverables verification matrix, acceptance criteria, and supervisor sign-off sheet. | [Project_Handover_and_Acceptance.html](./11_Handover/Project_Handover_and_Acceptance.html) |
| **12** | **Third-Party Libraries & Licenses** | Comprehensive inventory of all open-source libraries, versions, and software licenses. | [Third_Party_Libraries_and_Licenses.html](./12_Third_Party/Third_Party_Libraries_and_Licenses.html) |

---

## 🎓 Viva & Review Panel Preparation Package (Section 10)

| Guide # | Viva Topic | Focus Areas & Key Content | Direct Link |
| :---: | :--- | :--- | :---: |
| **10.1** | **Core Questions & Master Answers** | Problem statement, motivation, project objectives, target users, and complete end-to-end data journey. | [Viva_Questions_and_Answers.html](./10_Viva/Viva_Questions_and_Answers.html) |
| **10.2** | **Technical Deep-Dive** | ESP32 ADC1 vs ADC2, DHT22 single-wire protocol, SX1278 SPI, Ra-02 modulation, and Python serial buffering. | [Technical_Questions.html](./10_Viva/Technical_Questions.html) |
| **10.3** | **Architecture & System Design** | 4-tier decoupled model, 433 MHz LoRa vs 2.4 GHz WiFi/Cellular, Database Triggers vs cron, and Direct vs API Ingestion. | [Architecture_Questions.html](./10_Viva/Architecture_Questions.html) |
| **10.4** | **Database Design & Normalization** | Relational schema, 3NF normalization, PL/pgSQL trigger engine, deduplication constraints, and WAL replication. | [Database_Questions.html](./10_Viva/Database_Questions.html) |
| **10.5** | **Technology Stack Justifications** | Why ESP32, SX1278, Next.js 15, React 19, Tailwind v4, Leaflet, Recharts, Python, and Bun were chosen. | [Technology_Questions.html](./10_Viva/Technology_Questions.html) |
| **10.6** | **Security & Boundary Validation** | Fail-closed token authentication, 64 KB request size limits, physical boundary checks, and SQL injection immunity. | [Security_Questions.html](./10_Viva/Security_Questions.html) |
| **10.7** | **Difficult Panel Questions & Defense** | 20+ challenging cross-examinations, wet forest RF physics, antenna damage physics, and 30s elevator pitches. | [Difficult_Panel_Questions.html](./10_Viva/Difficult_Panel_Questions.html) |

---

## ⚡ Quick Navigation Tree

* 📂 **[01 Project Overview](./01_Project_Overview/Project_Overview.html)**
* 📂 **[02 Technical Documentation](./02_Technical_Documentation/Technical_Documentation.html)**
* 📂 **[03 User Manual](./03_User_Manual/User_Manual.html)**
* 📂 **[04 Installation and Setup](./04_Installation/Installation_and_Setup_Guide.html)**
* 📂 **[05 Database Architecture](./05_Database/Database_Documentation.html)**
* 📂 **[06 API Documentation](./06_API/API_Documentation.html)**
* 📂 **[07 Deployment Guide](./07_Deployment/Deployment_Guide.html)**
* 📂 **[08 Configuration and Environment](./08_Configuration/Configuration_and_Environment_Guide.html)**
* 📂 **[09 Known Issues and Limitations](./09_Known_Issues/Known_Issues_and_Limitations.html)**
* 📂 **10 Viva Defense Preparation:**
  * 📄 [10.1 Viva Questions & Answers](./10_Viva/Viva_Questions_and_Answers.html)
  * 📄 [10.2 Technical Questions](./10_Viva/Technical_Questions.html)
  * 📄 [10.3 Architecture Questions](./10_Viva/Architecture_Questions.html)
  * 📄 [10.4 Database Questions](./10_Viva/Database_Questions.html)
  * 📄 [10.5 Technology Questions](./10_Viva/Technology_Questions.html)
  * 📄 [10.6 Security Questions](./10_Viva/Security_Questions.html)
  * 📄 [10.7 Difficult Panel Questions](./10_Viva/Difficult_Panel_Questions.html)
* 📂 **[11 Handover and Acceptance](./11_Handover/Project_Handover_and_Acceptance.html)**
* 📂 **[12 Third-Party Libraries and Licenses](./12_Third_Party/Third_Party_Libraries_and_Licenses.html)**
