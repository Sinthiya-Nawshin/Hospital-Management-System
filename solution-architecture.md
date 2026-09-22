# Lab 2: Solution Architecture

## 1. Quality Goals
The Hospital Management System (HMS) operates in a clinical environment where incorrect, delayed, or inconsistent data can directly affect patient safety and financial integrity. Based on ISO/IEC 25010, the following quality attributes have been prioritised to guide all architectural decisions (Table 2.1).

*Table 2.1. Top Quality Goals of the HMS*

| Priority | Quality Attribute (ISO/IEC 25010) | Motivation in the HMS Context |
| :-- | :-- | :-- |
| 1 | **Reliability / Functional Correctness** | Clinical rules (allergy blocks, business-hour scheduling, gender-based room assignment) must execute deterministically. A single missed check can cause an adverse medication event. |
| 2 | **Security & Data Privacy** | HMS stores identifiable patient health data. Role-based access, encrypted transport, and strict separation of clinical and financial contexts are mandatory to comply with healthcare data-protection expectations (e.g., GDPR). |
| 3 | **Performance Efficiency** | Receptionists, doctors, and billing officers work concurrently; appointment look-ups, room searches, and discharge clearance checks must respond in under two seconds under peak load. |
| 4 | **Modifiability / Maintainability** | Regulations, insurance rules, and departmental structures change frequently. The architecture must isolate business rules so that updates (e.g., new billing policy) do not ripple through unrelated modules. |
| 5 | **Integrability** | The HMS must exchange data with external actors — insurance companies, pharmacy suppliers, and eventually laboratory/imaging systems — via well-defined, versioned interfaces. |

## 2. Solution Strategy
This section summarises the fundamental architectural decisions that shape the HMS and explains how each decision contributes to the quality goals listed above.

*Table 2.2. Solution Strategy Overview*

| Decision Area | Chosen Approach | Rationale |
| :-- | :-- | :-- |
| Architectural style | **Modular monolith** on the server side, decomposed by bounded context (Admission & Appointments, Billing, Inventory, HR/Admin), with a clear path to extract services later. | Matches a small hospital deployment, keeps transactional integrity across clinical and billing workflows, and avoids premature distribution complexity. |
| Top-level decomposition | Organised around the **bounded contexts** identified in Lab 1 (Context Map, Fig. 1.4). Each context owns its tables, rules, and APIs. | Aligns code structure with the domain, protects core clinical logic from generic/support changes. |
| Persistence | **MS SQL Server** (deployed as **Azure SQL Database** in cloud environments) with 3NF schema, ECA triggers, and stored procedures for invariants that must be guaranteed regardless of the calling service. | Continuity with Lab 1 design; database-level enforcement provides defence-in-depth for patient safety; managed cloud DB removes DBA overhead. |
| Backend technology | **ASP.NET Core Web API** (C#) exposing REST endpoints; Entity Framework Core for data access; hosted on **Azure App Service**. | Native integration with SQL Server, strong typing, mature security stack (Identity, policy-based authorization); managed PaaS in the same cloud as the DB. |
| Frontend technology | **React + TypeScript** SPA hosted on **Vercel** (CDN edge, GitHub-integrated deploys). | Rich, role-specific dashboards for receptionists, doctors, billing officers, supply clerks, ambulance managers, HR, and admins; TypeScript reduces defects on forms with many constraints; edge delivery for low-latency loads. |
| API style | **REST + JSON**, versioned (`/api/v1/...`); internal events via an in-process mediator (MediatR) to keep contexts loosely coupled. | Predictable for integrators (insurance, pharmacy), simple to secure; events prepare the ground for future service extraction. |
| Authentication & Authorization | **OAuth 2.0 / OpenID Connect** with role claims (Doctor, Receptionist, Billing Officer, Supply Clerk, Ambulance Manager, HR, Admin — exactly the internal-actor set from Lab 1 §3). Fine-grained policy checks at the API layer; row-level filters at the data layer. | Supports the Security quality goal and the HR-enforced hospital-email policy. |
| Quality-attribute tactics | Allergy/scheduling/discharge rules enforced **both** in the application layer (friendly errors) **and** as SQL triggers (last line of defence). Caching of reference data (departments, rooms, medication catalogue). Structured logging + health checks. | Reliability and Performance are achieved together: cheap reads for reference data, authoritative writes guarded by the database. |
| Organizational decisions | Single small team; **trunk-based development** with pull requests, automated CI (build, unit + integration tests against a real SQL Server container). | Small team, fast feedback, matches the "no mocks for DB" stance implied by trigger-centric design. |

## 3. C4 Model – Context View
The System Context diagram (Figure 2.1) shows the HMS as a single system and the people and external systems it interacts with. Internal actors identified in Lab 1 use the HMS through one role-aware staff portal; external systems (insurance, pharmacy supplier, email/SMS gateway) integrate via REST APIs and notifications.

![](./assets/hms-c4-context.drawio.svg)
*Figure 2.1. C4 Context Diagram of the Hospital Management System*

**Key external interactions**
- **Doctor / Receptionist / Billing Officer / Supply Clerk / Ambulance Manager / HR / Admin** – operate the HMS through a single role-filtered web UI.
- **Patient** – interacts indirectly: appointments are booked at Reception, bills are settled at the Billing desk. No patient-facing portal is in scope; patient-visible notifications are delivered by SMS/email.
- **Insurance Company** – consumes billing and eligibility data through a secured REST endpoint.
- **Pharmacy Supplier** – receives restock orders and confirms deliveries.
- **Notification Gateway (Email/SMS)** – delivers appointment reminders and discharge notices.

## 4. C4 Model – Containers View
The Containers diagram (Figure 2.2) decomposes the HMS into independently deployable units. The choice of a modular monolith is reflected in a single `HMS API` container whose internal modules mirror the bounded contexts.

![](./assets/hms-c4-containers.drawio.svg)
*Figure 2.2. C4 Containers Diagram of the HMS*

*Table 2.3. Containers of the HMS*

| Container | Technology | Responsibility |
| :-- | :-- | :-- |
| Web Client (Staff Portal) | React + TypeScript SPA, hosted on Vercel | Single role-aware UI for doctors, receptionists, billing officers, supply clerks, ambulance managers, HR, and admins. The working prototype at `hospital-management-system-demo.vercel.app` is the current artefact of this container. |
| HMS API | ASP.NET Core Web API, hosted on Azure App Service | Orchestrates all use cases; hosts bounded-context modules (Admission & Appointments, Billing, Inventory, Staff & Administration). |
| HMS Database | Azure SQL Database (MS SQL Server) | Authoritative store; enforces ECA triggers, referential integrity, and business invariants. |
| Identity Provider | ASP.NET Core Identity / OIDC | Issues tokens, manages roles and hospital-email policy. **Co-located with the HMS API on the same App Service plan in v1**; documented as a candidate for extraction. |
| Notification Service | Background worker (Hangfire) | Sends appointment reminders, discharge notices, restock alerts via email/SMS gateway. Runs in the same App Service plan as the API. |

## 5. C4 Model – Components View
Figure 2.3 zooms into the **Admission & Appointments** module of the HMS API, the core domain identified in Lab 1. Components are organised by responsibility: inbound adapters (Controllers), application services (use cases), domain services (invariants), and outbound adapters (EF Core repositories, notification dispatcher).

![](./assets/hms-c4-components.drawio.svg)
*Figure 2.3. C4 Components Diagram – Admission & Appointments module*

*Table 2.4. Key components*

| Component | Responsibility |
| :-- | :-- |
| `AppointmentController` | REST endpoints for booking, rescheduling, cancelling. |
| `AdmissionController` | Endpoints for admit, transfer, and discharge requests. |
| `AppointmentScheduler` (application service) | Validates business hours, detects overlaps, coordinates doctor availability. |
| `AdmissionManager` (application service) | Assigns rooms respecting gender rules, delegates to billing on discharge. |
| `PrescriptionService` | Cross-references `PatientAllergy` before persisting a prescription. |
| `RoomAllocationPolicy` (domain service) | Encapsulates gender/occupancy rules. |
| `AppointmentRepository`, `AdmissionRepository`, `PatientRepository` | EF Core repositories mapping aggregates to SQL Server tables. |
| `BillingGateway` (anti-corruption layer) | Emits domain events consumed by the Billing module; checks outstanding balances before discharge. |
| `NotificationPublisher` | Hands off reminder/discharge events to the Notification Service. |

## 6. Information Architecture
The Information Architecture (Figure 2.4) organises the HMS portal into role-scoped areas. Each top-level section corresponds to a bounded context and exposes only the features the signed-in role is authorised to use.

![](./assets/hms-information-architecture.drawio.svg)
*Figure 2.4. Information Architecture (SiteMap) of the HMS Portal*

**Top-level navigation** (matches the sidebar of the working prototype):
- **Dashboard** – role-specific KPIs and alerts (e.g., allergy blocks today, rooms free, bills pending clearance).
- **Patients** – search, register, view medical history, allergies, admissions.
- **Appointments** – calendar view, booking, rescheduling, out-of-hours warnings.
- **Admissions** – room board (gender-filtered), admit/transfer/discharge flows.
- **Prescriptions** – issue prescription (with allergy check), view history.
- **Pharmacy** – medication catalogue, stock levels, restock orders.
- **Billing** – invoice list, discharge clearance, insurance exports.
- **Ambulance** – vehicle registry, dispatch board.
- **Staff** – staff directory, departments, hospital-email policy enforcement, role assignment and audit log (Admin-only sub-views).

Navigation rules:
1. Side navigation is filtered by role claims (per the role-to-nav mapping defined in the staff portal); hidden items are also blocked server-side.
2. Breadcrumbs follow the Patient as the primary axis of the core domain (Patient → Admission → Prescription → Bill).
3. Global search spans Patients (always) and Appointments / Rooms when the active role can see them.

---
[back](../README.md)