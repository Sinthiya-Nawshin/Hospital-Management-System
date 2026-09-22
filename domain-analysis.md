# Lab 1: Domain Analysis
## Introduction
### An Integrated Hospital Management System (HMS)

The Hospital Management System (HMS) is a specialized relational database solution designed to automate and unify the core clinical, administrative, and financial operations of a modern healthcare facility. In a complex environment involving multiple interconnected departments, such as Cardiology, Neurology, and Pediatrics - maintaining high standards of patient care and operational safety is paramount. The system acts as a central hub, integrating various entities including doctors, patients, admissions, and pharmaceutical resources into a cohesive digital structure.

Currently, many healthcare providers face significant operational bottlenecks due to manual, paper-based workflows. Fragmented record-keeping for patient medical history, physical doctor schedules, and medication inventories is not only inefficient but also prone to critical human errors. Challenges such as double-booking appointments, mismanaging room occupancy, and potential safety risks, like prescribing medication to a patient with a recorded allergy highlighting the urgent need for a robust digital intervention. Furthermore, manual billing and discharge processes often lead to administrative delays and financial inconsistencies.

The proposed HMS establishes a secure, normalized database framework (implemented in MS SQL Server) to streamline these workflows. By centering the system around the Patient entity, the project ensures that medical records, appointments, ward admissions, and prescriptions are synchronized in real-time. The implementation of ECA (Event-Condition-Action) rules and advanced triggers provides a layer of automated safety and logic. This includes enforcing business hours (09:00–17:00), validating patient ages to prevent future-dated entries, and ensuring room assignments strictly follow privacy and gender-matching protocols.

The expected outcomes of this project are measurable improvements in both clinical accuracy and organizational efficiency. By reducing data redundancy through Third Normal Form (3NF) normalization, the system provides a reliable "source of truth" for healthcare providers. Key results include faster appointment turnarounds, eliminated scheduling conflicts, and a rigorous financial clearance process that prevents patient discharge until all pending bills are settled. Ultimately, this system serves as a scalable foundation for future healthcare information systems, ensuring secure data storage and enhanced decision-making across the hospital.


## 1. Problem Definition

### Current State
The following diagram (Figure 1.1) presents that the Hospital currently relies on disparate, often paper-based or loosely connected systems to manage an ecosystem of internal staff and external actors. Process bottlenecks occur primarily in schedule coordination, patient safety verifications, and financial reconciliation.

![](./assets/hms-current-state.drawio.svg)

*Figure 1.1. The Rich Picture of the Current State of the Hospital Management System*

#### **Processes and interactions**

**Provide customer and administrative services**
- Managing patient registration while manually ensuring data validity (e.g., verifying dates of birth, recording fixed gender values).
- Handling visit bookings and scheduling, currently requiring manual review of physical calendars to avoid double-booking and out-of-hours appointments.
- Managing admissions and room allocations, ensuring room availability and compliance with privacy rules (e.g., matching patient gender to allowed room gender).
- Dispatching ambulances and ensuring no duplicate vehicle registrations.

**Provide medical services**
- Delivery of medical care and consultations through scheduled outpatient appointments and inpatient admissions.
- Prescribing medications, which requires manual cross-referencing of a patient’s medical history to prevent critical allergy conflicts.

**Financial and inventory management**
- Generating bills for services rendered (admissions, appointments, ambulance trips) and ensuring comprehensive financial tracking.
- Enforcing billing clearances so that patients with pending unpaid invoices cannot be formally discharged.
- Managing pharmaceutical and physical stock, requiring clerks to manually verify that medication prices are positive and stock remains non-negative.

**Internal actors**
- **Doctor**: Provides medical services, creates prescriptions, manages appointments.
- **Receptionist**: Manages patient registration, intakes, scheduling, and room allocation.
- **Billing Officer**: Reviews patient ledgers, generates bills, and processes financial clearances for discharge.
- **Supply Clerk**: Manages medication stocks and physical inventory records.
- **Ambulance Manager**: Oversees vehicle fleet and dispatches.
- **HR Manager**: Handles staff onboarding (e.g., enforcing official hospital email policies).
- **Admin**: Manages user roles, audits clinical and financial events, and maintains reference data.

**External actors**
- **Patient**: Seeks care, provides medical history, attends appointments, gets admitted, and settles bills.
- **Insurance Company**: Provides coverage and financial support for patient care.

#### **Main resources**
- **Schedules & Records**: Patient files, doctor appointments, admission logs, structured allergy records.
- **Infrastructure**: Hospital rooms (categorized by type and gender rules).
- **Inventory**: Medication stock.
- **Fleet**: Ambulances.
- **Financial Data**: Summarized bills linked to respective clinical services.

#### Issues & Conflicts of Interest
*Table 1.1. Main Issues & Conflicts of Interest*
| ID  | Actors                          | Description                                                                                                    |
| :-- | :------------------------------ | :------------------------------------------------------------------------------------------------------------- |
| I01 | Receptionist → Patient          | Manual validation of patient data and room assignments is slow and prone to privacy violations.                |
| I02 | Doctor → Receptionist           | Poor schedule visibility leads to double-booked appointments and consultations scheduled outside business hours. |
| I03 | Doctor → Patient                | Lack of automated allergy checks during prescription poses a critical risk to patient safety.                  |
| I04 | Billing Officer → Patient       | Patients may be discharged before clearing pending bills due to disconnected clinical and financial records.   |
| I05 | Supply Clerk → Management       | Inaccurate tracking of medications leads to negative stock values and invalid pricing.                         |
| I06 | Ambulance Mgr → Administration  | Mismanagement of ambulance fleets leads to duplicate vehicle registrations and inefficient dispatch.           |

### Desired State
In the desired state, the new Hospital Management System (HMS) serves as the centralized source of truth. Data flows seamlessly through strict automated rules (Event-Condition-Action). 

![](./assets/hms-desired-state.drawio.svg)
*Figure 1.2. The Rich Picture of the Desired State of Hospital Management System.*

When a receptionist registers a patient, data constraints immediately ensure validity. When scheduling, the system actively blocks overlapping appointments and enforces business hours. During treatment, doctors are automatically blocked from prescribing medications if the patient has a recorded allergy. Finally, clinical and billing modules are deeply integrated so every bill corresponds to a concrete service, and the system automatically prevents the discharge of any patient with unpaid balances.


## 2. Goals and Objectives
This section presents an overview of the Hospital Managements primary business goals and objectives and the tactics and means used to achieve them. It is intended for hospital management, insurance companies and other stakeholders to align on strategic intent, expected outcomes and supporting initiatives.
    
### Goals–Objectives Table (companion view to BMM)
The business strategy focuses on automating hospital processes to improve patient care, safety, and operational efficiency. The strategic intent is fulfilled by strict implementation of business and information rules into the database layer.

*Table 1.2. Goals and Objectives*

| Goal                              | Objective                                                                                    | Supported by (Policy/Tactic/Strategy)                                                                        |
| :-------------------------------- | :------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| Ensure Patient Safety             | Achieve zero adverse medication events by enforcing automated allergy checks on prescription.| Tactic: Implement Clinical Safety Triggers; Medical Care Policy.                                             |
| Improve Operational Efficiency    | Eliminate scheduling conflicts and streamline room allocation based on gender rules.         | Tactic: System-enforced scheduling uniqueness and real-time room validation; Operational Policy.             |
| Optimize Financial Integrity      | Ensure 100% of generated bills are linked to valid services and block unpaid discharges.     | Tactic: Implement Integrated Billing & Clearance Workflows; Financial Policy.                                |
| Maintain Inventory Consistency    | Prevent negative stock and invalid pricing in the pharmacy.                                  | Tactic: Inventory Constraints and Validation; Supply Management Policy.                                      |

            
## 3. Stakeholders
This section identifies the internal and external stakeholders relevant to the hospital management domain and outlines their specific interests in the hospital management portal. The stakeholder perspective (Table 1.3) supports domain analysis by clarifying how different roles interact with, influence, or are affected by the portal. This enables the definition of portal requirements that support operational efficiency, service quality, regulatory compliance, and effective communication across the clinic’s ecosystem.

*Table 1.3. Internal and external stakeholders*
| **Name** | **Org Unit** | **Interest in the Hospital Management System** |
| --- | --- | --- |
| Receptionist | Administration | Expects streamlined data entry with automatic validation for dates of birth, gender, and efficient room filtering. |
| Doctor | Clinical Practice | Needs a reliable scheduling module free of overlaps, and safety nets (like allergy alerts) when prescribing medication. |
| Billing Officer | Finance | Requires an integrated view ensuring all clinical actions (admissions, appointments) automatically generate traceable financial records. |
| Supply Clerk | Pharmacy/Inventory | Expects reliable tracking of medication logic to prevent pricing and stock errors. |
| Ambulance Manager | Administration / Fleet | Needs a unique-plate registry and a dispatch board that records every trip against the originating patient. |
| HR Manager | Administration | Interested in enforcing personnel policies via the system (e.g., domain verification for emails). |
| Admin | IT / Compliance | Manages role assignments, accesses the audit log of allergy blocks and discharge-clearance events, and maintains reference data (departments, rooms, medication catalogue). |
| Patient | External | Seeks efficient bookings, safe treatment (no allergy mishaps), and accurate, consolidated billing. Interacts indirectly via Reception, Doctor and Billing staff (no self-service portal in scope). |
| Insurance Company | External | Indirect interest in accurate, transparent patient bills linked to verified appointments and admissions. |


## 4. Essential Features
The following table (Table 1.4) briefly describes the essential features of the proposed Hospital Management Portal, derived from the business process and conceptual model analysis.

*Table 1.4. Essential features of HMS*

| Title                         | Domain (core/supporting/generic) | Description                                                                                                                          |
| :--                           | :---                             | :---                                                                                                                                 |
| Patient Registration          | Core                             | Enforces valid profile creation, ensuring correct formats (Age, Gender) for clinical documentation.                                  |
| Appointment Scheduling        | Core                             | Manages doctor schedules, actively preventing double-booking and out-of-hours consultations.                                         |
| Inpatient Admission logic     | Core                             | Connects patients to rooms while strictly enforcing gender-based privacy availability.                                               |
| Prescription & Safety Control | Core                             | Allows medication prescriptions with an active safety check against recorded Patient Allergies.                                      |
| Billing & Discharge Clearance | Supporting                       | Consolidates charges for services. Actively prevents patient discharge if pending bills remain.                                      |
| Pharmacy Inventory Management | Supporting                       | Tracks medication stocks and enforces positive pricing and non-negative stock counts.                                                |
| Ambulance Fleet Tracking      | Supporting                       | Manages dispatch capabilities and enforces unique vehicle registration.                                                              |
| Staff & Department Tracking   | Generic (Internal)               | Manages the doctor roster and departmental allocations.              

This feature set defines the functional scope baseline for architecture decomposition, module design, and deployment planning.


## 5. Context Map
The Context Map (Figure 1.4) visualises the domain boundaries and integration relationships for the Hospital Management Systems Portal. It identifies ownership, primary responsibilities, and key data flows so teams can design interfaces, data mappings and integration patterns with clear service contracts and compliance controls.

![](./assets/hms-context-map.drawio.svg)

*Figure 1.4. The Context Map of the Hospital Management Systems Portal.*

The Context Map is organised into the following contexts:

**Core Domain — Admission & Appointments:** Owns the primary healthcare lifecycle. This includes the Patient registry, Appointment scheduling, Inpatient Admissions, and Prescriptions. It is responsible for critical business logic such as preventing appointment overlaps, enforcing room privacy constraints, and cross-referencing prescriptions with the Patient Allergy records.

**Supporting Subdomains — Financial & Billing:** Owns the Billing context. It consumes data from the Core Domain (Admissions, Appointments, Ambulance trips) to generate invoices. This context governs the discharge clearance process by locking Core Domain admission updates if unpaid bills exist, and it produces the periodic billing exports consumed by Insurance Companies.

**Supporting Subdomains — Resource & Inventory Management:** Includes Room management, Pharmacy (Medications) and the Ambulance fleet. This context ensures resources are accurately tracked, priced, and maintained without negative values or duplicate registrations.

**Generic (Internal) — Staff & Administration:** Includes Department structuring, Doctor profiles (with hospital-email validation), and the cross-cutting Admin functions of role management and audit logging.

**Generic (External) — Insurance Companies:** Consumes verified billing data linked to admissions, appointments and ambulance trips; settles patient invoices through the Billing context.


---
[back](../README.md)