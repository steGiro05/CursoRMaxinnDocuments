# HealthMaxxing System - Class Overview

This document provides a detailed description of every class in the HealthMaxxing domain model, including their responsibilities, attributes, methods, and key relationships.

---

## Table of Contents

1. [HealthMaxxinSystem](#1-healthmaxxinsystem)
2. [User](#2-user)
3. [Patient](#3-patient)
4. [Doctor](#4-doctor)
5. [HeadOfDepartment](#5-headofdepartment)
6. [HeadOfHospital](#6-headofhospital)
7. [SystemAdmin](#7-systemadmin)
8. [Hospital](#8-hospital)
9. [Department](#9-department)
10. [Shift](#10-shift)
11. [Calendar](#11-calendar)
12. [TemporaryPassword](#12-temporarypassword)
13. [Appointment](#13-appointment)
14. [Payment](#14-payment)
15. [Treatment](#15-treatment)
16. [TreatmentCatalog](#16-treatmentcatalog)
17. [Prescription](#17-prescription)
18. [Exemption](#18-exemption)
19. [InsurancePolicy](#19-insurancepolicy)
20. [InsurancePolicyRecord](#20-insurancepolicyrecord)
21. [OutstandingDebt](#21-outstandingdebt)
22. [AiAssistant](#22-aiassistant)
23. [MailService](#23-mailservice)

---

## 1. HealthMaxxinSystem

### Description
The central singleton class representing the entire HealthMaxxing platform. It acts as the top-level container and entry point for the system, tying together the major subsystems: the hospital network, the user base, the treatment catalog, and insurance policies. Its existence ensures that all components operate within a single, coherent system context.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `systemName` | String | The display name of the platform |
| `version` | String | The current running version of the software |

### Relationships
- Manages (1-to-1) one `Hospital` network entry point
- Is associated with `User`, `TreatmentCatalog`, and `InsurancePolicy` at a system-wide level
- Contains (1-to-1) one `SystemAdmin`

---

## 2. User

### Description
The base class for all human actors who interact with the HealthMaxxing platform (excluding `SystemAdmin`, which is a separate administrative entity). It encapsulates the shared identity and authentication data common to all user roles, patients, doctors, department heads, and hospital heads. Every user has a unique account, can authenticate, and can manage their own profile. The `fiscalCode` attribute plays an important role in the Italian healthcare context, serving as a universal patient identifier.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | String | Unique identifier for the user |
| `firstName` | String | The user's first name |
| `lastName` | String | The user's last name |
| `email` | String | The user's email address, used for login and notifications |
| `fiscalCode` | String | Italian tax code, used as a universal health identifier |
| `passwordHash` | String | Hashed version of the user's password |
| `isVerified` | Boolean | Whether the user's email address has been verified |
| `createdAt` | DateTime | Timestamp of account creation |

### Methods
| Method | Description |
|---|---|
| `login()` | Authenticates the user and initiates a session |
| `logout()` | Terminates the active session |
| `viewProfile()` | Returns the user's profile information |
| `updateProfile()` | Allows the user to modify their personal details |
| `requestPasswordReset()` | Triggers a password reset flow (email with temporary credentials) |
| `deleteAccount()` | Permanently removes the user account from the system |

### Relationships
- Superclass of `Patient`, `Doctor`, `HeadOfDepartment`, and `HeadOfHospital`
- Owns exactly one `Calendar`
- May have zero or one `TemporaryPassword` at any given time
- Is assisted by exactly one `AiAssistant`
- Is notified by `MailService` for account-related events

---

## 3. Patient

### Description
A specialization of `User` representing individuals who seek and receive medical care within the system. Patients are the primary consumers of healthcare services: they book appointments for treatments, manage their insurance coverage, and accumulate a health history made up of prescriptions, examinations, and exemptions. The `fiscalCode` inherited from `User` is especially important for patients, as it is the key used by doctors to access their medical records.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `address` | String | The patient's residential address |
| `dateOfBirth` | Date | The patient's date of birth |

*(Inherits all attributes from `User`)*

### Methods
| Method | Description |
|---|---|
| `bookAppointment()` | Creates a new appointment request for a treatment |
| `cancelAppointment()` | Cancels an existing appointment |
| `rescheduleAppointment()` | Moves an existing appointment to a new date/time |
| `registerInsurance(code: String)` | Links an insurance policy to the patient's profile using a unique code |

### Relationships
- Is the "Booked By" party for `Appointment` (1 patient to 0..N appointments)
- Receives `Prescription` records (issued to 0..N)
- Holds `Exemption` records (belongs to 0..N)
- Linked to `InsurancePolicyRecord` entries (0..N)
- Is notified by `MailService` for appointments, prescriptions, exemptions, and examinations

---

## 4. Doctor

### Description
A specialization of `User` representing licensed medical professionals registered in the system. Doctors are the primary healthcare providers: they conduct appointments, issue prescriptions and exemptions, upload examination results, and access patient records. They are assigned to one or more departments and operate according to scheduled shifts. Doctors form the basis for the more privileged `HeadOfDepartment` and `HeadOfHospital` roles.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `specialty` | String | The medical specialty of the doctor (e.g., Cardiology, Radiology) |

*(Inherits all attributes from `User`)*

### Methods
| Method | Description |
|---|---|
| `viewAppointments()` | Retrieves the list of appointments assigned to this doctor |
| `cancelAppointment(reason: String)` | Cancels an appointment and records the reason |
| `createPrescription()` | Issues a new prescription for a patient |
| `createExemption()` | Issues a new exemption record for a patient |
| `accessPatientRecord(fiscalCode: String)` | Looks up a patient's full medical history using their fiscal code |
| `uploadExamination()` | Attaches examination results to a patient's record |
| `viewChronologicalReport()` | Generates a chronological view of all clinical activities |

### Relationships
- Works at a `Department` (0..1 department at a time)
- Assigned to `Appointment` records (0..N)
- Issues `Prescription` records (0..N)
- Creates `Exemption` records (0..N)
- Schedules follow a `Shift` pattern (0..N)
- Is notified by `MailService` for shift changes and credentials

---

## 5. HeadOfDepartment

### Description
A specialization of `Doctor` with additional administrative authority scoped to a single department. The Head of Department oversees the operational management of their unit: they configure capacity, define treatment offerings, manage doctor assignments, and maintain shift patterns. They also have visibility into all appointments occurring within their department and can intervene by cancelling them when necessary.

### Methods
| Method | Description |
|---|---|
| `assignDoctorToDepartment()` | Assigns a doctor to work within this department |
| `updateDepartmentTreatments()` | Modifies the list of treatments offered by the department |
| `configureDepartmentCapacity()` | Sets or updates the maximum patient capacity for the department |
| `setShiftPattern(doctor: Doctor)` | Defines or updates the work shift schedule for a specific doctor |
| `viewDepartmentAppointments()` | Retrieves all appointments booked within the department |
| `cancelDepartmentAppointment(reason: String)` | Cancels an appointment at the department level with a stated reason |

*(Inherits all attributes and methods from `Doctor` and `User`)*

### Relationships
- Manages exactly one `Department` (1-to-1)
- Manages doctor schedules via `Shift` associations

---

## 6. HeadOfHospital

### Description
A specialization of `Doctor` with the highest level of administrative authority within a specific hospital. The Head of Hospital oversees the appointment and allocation of staff at the departmental and hospital-wide level. They are also the key actor in financial management, having visibility into insurance-related debts and the ability to settle outstanding balances with insurance companies.

### Methods
| Method | Description |
|---|---|
| `appointHeadOfDepartment(doctor: Doctor, dept: Department)` | Designates a doctor as the Head of a specific department |
| `allocateDoctors(doctors: List<Doctor>, dept: Department)` | Assigns a group of doctors to a department |
| `viewInsuranceDebt()` | Displays the outstanding debt owed by insurance companies |
| `settleInsuranceDebt(company: InsuranceCompany, amount: Decimal)` | Records a partial or full debt settlement with an insurance company |

*(Inherits all attributes and methods from `Doctor` and `User`)*

### Relationships
- Manages exactly one `Hospital` (1-to-1)
- Interacts with `OutstandingDebt` to monitor and settle insurance balances

---

## 7. SystemAdmin

### Description
A privileged, platform-level administrator who sits outside the `User` hierarchy. The System Admin is responsible for the configuration and lifecycle management of the entire platform infrastructure: creating and closing hospitals and departments, registering doctors, and managing insurance policies and treatments globally. This role is typically held by platform operators rather than healthcare professionals and has no patient-facing responsibilities. It has its own separate authentication mechanism.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier for the admin account |
| `username` | String | The login username |
| `passwordHash` | String | Hashed password for admin authentication |

### Methods
| Method | Description |
|---|---|
| `createHospital()` | Registers a new hospital in the system |
| `editHospital()` | Updates hospital details |
| `closeHospital()` | Marks a hospital as closed and inactive |
| `createDepartment()` | Creates a new department within a hospital |
| `editDepartment()` | Modifies department attributes |
| `closeDepartment()` | Deactivates a department |
| `registerDoctor()` | Adds a new doctor to the platform |
| `assignDoctorToHospital()` | Associates a doctor with a specific hospital |
| `appointHeadOfHospital(doctor: Doctor, hospital: Hospital)` | Designates a doctor as the Head of a Hospital |
| `createInsurancePolicy()` | Adds a new insurance policy to the system |
| `editInsurancePolicy()` | Updates an existing insurance policy |
| `deactivateInsurancePolicy()` | Disables an insurance policy |
| `generateInsuranceCode(policy: InsurancePolicy, patient: Patient)` | Generates a unique insurance code linking a policy to a patient |
| `createTreatment()` | Adds a new treatment to the global catalog |
| `editTreatment()` | Modifies an existing treatment |
| `deactivateTreatment()` | Removes a treatment from active availability |
| `adminLogin()` | Authenticates the system administrator |
| `adminLogout()` | Terminates the admin session |

### Relationships
- Operates at the `HealthMaxxinSystem` level (1-to-1)
- Manages `Hospital`, `Department`, `Doctor`, `InsurancePolicy`, and `Treatment` lifecycle

---

## 8. Hospital

### Description
Represents a physical hospital facility registered within the platform. A hospital is the top-level organizational unit in the care delivery hierarchy, containing multiple departments and employing doctors. Its status lifecycle (active/closed) is managed by the `SystemAdmin`. Each hospital is headed by a `HeadOfHospital` and can accumulate insurance-related debt tracked via `OutstandingDebt`.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `name` | String | Official name of the hospital |
| `address` | String | Physical address of the facility |
| `status` | HospitalStatus | Current operational status (e.g., ACTIVE, CLOSED) |
| `createdAt` | DateTime | Timestamp of registration in the system |

### Methods
| Method | Description |
|---|---|
| `close()` | Transitions the hospital to a closed state |

### Relationships
- Contains 1..N `Department` entities
- Managed by exactly one `HeadOfHospital`
- Appointments are located at a hospital (via the "Located at" association on `Appointment`)
- Owes debt tracked by `OutstandingDebt`

---

## 9. Department

### Description
Represents a specialized medical unit within a hospital (e.g., Cardiology, Oncology, Radiology). A department defines what treatments it offers, how many patients it can handle (capacity), and which doctors are working within it. Its operational state is managed through status flags, and it can be closed when no longer active. Departments are the direct context in which appointments and clinical activities take place.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `name` | String | Name of the department (e.g., "Cardiology") |
| `capacity` | Integer | Maximum number of patients/appointments the department can handle |
| `status` | DepartmentStatus | Current operational status (e.g., ACTIVE, CLOSED) |
| `createdAt` | DateTime | Timestamp of creation |

### Methods
| Method | Description |
|---|---|
| `close()` | Transitions the department to a closed/inactive state |

### Relationships
- Belongs to exactly one `Hospital`
- Managed by exactly one `HeadOfDepartment`
- Offers 0..N `Treatment` types
- Houses 0..N `Doctor` workers
- Is the site of 0..N `Appointment` records

---

## 10. Shift

### Description
Defines a recurring working time block for a specific doctor. A shift is characterized by a day of the week and a time window, and is valid within a defined date range. This allows the system to model part-time schedules, rotations, and temporary duty assignments. The validity check ensures the system only considers shifts that fall within their active period when computing doctor availability for appointment booking.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `startTime` | Time | The time the shift begins |
| `endTime` | Time | The time the shift ends |
| `dayOfWeek` | DayOfWeek | The day of the week this shift applies to |
| `validFrom` | Date | Start date of this shift's validity period |
| `validUntil` | Date | End date of this shift's validity period |

### Methods
| Method | Description |
|---|---|
| `isValid()` | Returns true if the current date falls within the validFrom–validUntil range |

### Relationships
- Schedules 0..N `Doctor` working patterns (each doctor has 0..N shifts)
- Set by `HeadOfDepartment` via `setShiftPattern()`

---

## 11. Calendar

### Description
A personal scheduling container owned by each `User`. It aggregates all appointments associated with that user and provides a secure export mechanism for integration with external calendar applications (e.g., Google Calendar, iCal). The export is token-protected and time-limited to prevent unauthorized access.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `exportToken` | String | A secure token for external calendar export/sync |
| `tokenExpiresAt` | DateTime | Expiry timestamp of the export token |

### Methods
| Method | Description |
|---|---|
| `generateExportToken()` | Creates or refreshes the export token with a new expiry date |
| `getAppointments()` | Returns the list of all appointments associated with this calendar |

### Relationships
- Owned by exactly one `User` (1-to-1)
- Displays 0..N `Appointment` records

---

## 12. TemporaryPassword

### Description
A short-lived credential object generated during the password reset flow or when a new doctor account is created by the `SystemAdmin`. It ensures that users can gain initial or recovery access to the system without exposing their permanent password. The system enforces expiry to minimize the security window of exposure.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `value` | String | The plaintext or hashed temporary password value |
| `createdAt` | DateTime | Timestamp when the temporary password was generated |
| `expiresAt` | DateTime | Timestamp after which the password is no longer valid |

### Methods
| Method | Description |
|---|---|
| `isExpired()` | Returns true if the current time has passed `expiresAt` |

### Relationships
- Belongs to zero or one `User` (a user either has an active temporary password or does not)

---

## 13. Appointment

### Description
The central transactional object of the system, representing a scheduled encounter between a patient and a doctor for a specific treatment in a department. An appointment has a well-defined lifecycle (e.g., PENDING, CONFIRMED, CANCELLED, COMPLETED) and can be created, rescheduled, or cancelled by various actors. Cancellation always requires a reason for audit purposes. Each appointment is associated with a `Payment`.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `date` | Date | The scheduled date of the appointment |
| `time` | Time | The scheduled time of the appointment |
| `status` | AppointmentStatus | Current status (e.g., PENDING, CONFIRMED, CANCELLED, COMPLETED) |
| `cancellationReason` | String | The reason provided when the appointment is cancelled |
| `createdAt` | DateTime | Timestamp of creation |

### Methods
| Method | Description |
|---|---|
| `create()` | Initializes and persists a new appointment |
| `reschedule(newDate: Date)` | Moves the appointment to a new date |
| `cancel(reason: String)` | Cancels the appointment and stores the reason |

### Relationships
- Booked by exactly one `Patient`
- Assigned to exactly one `Doctor`
- Located at exactly one `Hospital` (within a `Department`)
- Paid via 0..N `Payment` records
- Displayed in the user's `Calendar`
- Triggers notifications via `MailService` (confirmation, cancellation, rescheduling)

---

## 14. Payment

### Description
Records the financial transaction associated with a single appointment. The payment model supports a layered discount structure: a base price is defined by the treatment, which can be reduced by applicable exemptions and/or insurance coverage, yielding a final amount. The class supports multiple payment methods and tracks transaction status (e.g., PENDING, COMPLETED, REFUNDED). Refunds are handled for appointments that are cancelled after payment.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `basePrice` | Decimal | The original cost of the treatment before discounts |
| `exemptionDiscount` | Decimal | Amount deducted due to applied exemptions |
| `insuranceDiscount` | Decimal | Amount covered by insurance policies |
| `finalAmount` | Decimal | The net amount charged to the patient |
| `paymentMethod` | PaymentMethod | The method used (e.g., CREDIT_CARD, BANK_TRANSFER) |
| `paidAt` | DateTime | Timestamp when payment was completed |
| `status` | PaymentStatus | Current payment state (e.g., PENDING, PAID, REFUNDED) |
| `transactionId` | String | External transaction reference for reconciliation |

### Methods
| Method | Description |
|---|---|
| `process()` | Executes the payment transaction |
| `refund()` | Reverses the payment and issues a refund |
| `applyExemptions(exemptions: List<Exemption>)` | Calculates and applies exemption discounts to the base price |
| `applyInsurance(policies: List<InsurancePolicy>)` | Calculates and applies insurance coverage discounts |

### Relationships
- Associated with one `Appointment` (via "Paid via")
- Uses 0..N `Exemption` records for discount calculation
- Uses 0..N `InsurancePolicy` records for coverage calculation

---

## 15. Treatment

### Description
Represents a specific medical procedure or service that can be offered by departments and booked by patients as appointments. Each treatment has a defined base price, a required room type, and the type of medical professional needed to perform it. Treatments can be deactivated to remove them from availability without deleting their historical records, preserving audit integrity.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `name` | String | Name of the treatment (e.g., "ECG", "Blood Test") |
| `roomType` | String | Type of room required (e.g., "Operating Room", "Consultation Room") |
| `specializedFigure` | String | The medical role required to perform the treatment |
| `basePrice` | Decimal | The standard cost of the treatment |
| `isActive` | Boolean | Whether the treatment is currently available for booking |
| `deactivatedAt` | DateTime | Timestamp of deactivation, if applicable |

### Methods
| Method | Description |
|---|---|
| `activate()` | Re-enables a previously deactivated treatment |
| `deactivate()` | Marks the treatment as inactive and records the deactivation time |

### Relationships
- Catalogued in one `TreatmentCatalog` (via "Catalogs")
- Offered by 0..N `Department` entities
- Referenced by `Prescription` (prescribes 0..N treatments)
- Referenced by `Exemption` (applicable to specific treatments)

---

## 16. TreatmentCatalog

### Description
A registry that aggregates all treatments defined in the system. The catalog serves as the authoritative source of available medical services, providing methods to query active treatments and manage their lifecycle. It acts as an intermediary between the `SystemAdmin` (who manages the global treatment list) and the departments that offer subsets of those treatments.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |

### Methods
| Method | Description |
|---|---|
| `addTreatment()` | Registers a new treatment in the catalog |
| `getActiveTreatments()` | Returns all treatments where `isActive = true` |
| `deactivateTreatment(treatment: Treatment)` | Deactivates a specific treatment within the catalog |

### Relationships
- Catalogs 1..N `Treatment` entries
- Associated with the `HealthMaxxinSystem` at a global level

---

## 17. Prescription

### Description
A formal medical document issued by a doctor to a patient, authorizing the patient to receive a specific treatment or medication. Prescriptions have a validity window and a fulfillment flag, enabling the system to track whether they have been acted upon. Expired or already-fulfilled prescriptions are distinguishable, preventing misuse.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `issueDate` | Date | The date the prescription was created |
| `expirationDate` | Date | The date after which the prescription is no longer valid |
| `isFulfilled` | Boolean | Whether the prescription has been used/acted upon |
| `notes` | String | Optional clinical notes from the issuing doctor |

### Methods
| Method | Description |
|---|---|
| `markFulfilled()` | Sets `isFulfilled` to true once the prescription has been used |
| `isExpired()` | Returns true if the current date is past `expirationDate` |

### Relationships
- Issued to exactly one `Patient`
- Issued by exactly one `Doctor`
- Prescribes 1..N `Treatment` entries
- Triggers a notification to the patient via `MailService`

---

## 18. Exemption

### Description
Represents a discount entitlement granted to a patient, typically based on medical, social, or economic criteria (as is common in the Italian National Health Service). An exemption reduces the cost of applicable treatments by a defined percentage and is valid for a specific time period. Exemptions are treatment-specific, meaning they only apply to a designated set of treatments rather than all services.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `discountPercentage` | Decimal | The percentage reduction applied to the base price |
| `validFrom` | Date | Start date of the exemption's validity |
| `validUntil` | Date | End date of the exemption's validity |
| `notes` | String | Clinical or administrative notes about the exemption |
| `applicableTreatments` | List\<Treatment\> | The set of treatments to which this exemption applies |

### Methods
| Method | Description |
|---|---|
| `isValid()` | Returns true if the current date falls within the validity window |
| `applyDiscount(basePrice: Decimal)` | Computes and returns the discounted price after applying the exemption percentage |

### Relationships
- Belongs to exactly one `Patient`
- Created by exactly one `Doctor`
- Applicable to 1..N `Treatment` types
- Used by `Payment` to calculate the `exemptionDiscount`
- Triggers a notification to the patient via `MailService`

---

## 19. InsurancePolicy

### Description
Represents a health insurance product offered by an external insurance company, registered in the system by the `SystemAdmin`. A policy defines the coverage conditions and the discount percentage it provides on treatments. Policies can be linked to individual patients via `InsurancePolicyRecord` entries, allowing the system to automatically apply insurance discounts during payment. Policies can be deactivated to prevent new linkages while preserving historical records.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `name` | String | Display name of the insurance product |
| `uniqueIdentifier` | String | A system-wide unique reference code for the policy |
| `coverageConditions` | String | Description of the conditions under which coverage applies |
| `discountPercentage` | Decimal | The percentage of treatment costs covered by this policy |
| `isActive` | Boolean | Whether the policy is currently available for new enrolments |
| `companyName` | String | The name of the insurance company offering this policy |

### Methods
| Method | Description |
|---|---|
| `deactivate()` | Marks the policy as inactive |
| `generateCode()` | Generates a new unique enrolment code for linking the policy to a patient |

### Relationships
- Records 1..N `InsurancePolicyRecord` entries (one per patient linked)
- Owed to an `OutstandingDebt` tracking the balance owed by the insurance company
- Offers coverage across 0..N `Department`-`Treatment` combinations
- Used by `Payment` to calculate the `insuranceDiscount`

---

## 20. InsurancePolicyRecord

### Description
A join entity that captures the specific instance of an insurance policy being linked to a patient. It records when the policy was registered, whether it has been consumed (used for a payment), and the total financial amount it covered. This provides a granular audit trail of insurance usage per patient and per policy.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `uniqueCode` | String | The unique code used by the patient to register this policy |
| `linkedAt` | DateTime | Timestamp when the patient registered the policy |
| `consumedAt` | DateTime | Timestamp when the insurance was applied to a payment |
| `totalAmountCovered` | Decimal | The cumulative monetary amount covered by this policy record |

### Relationships
- Records the use of exactly one `InsurancePolicy`
- Belongs to one `Patient`
- Forms part of the history tracked by `OutstandingDebt`

---

## 21. OutstandingDebt

### Description
Tracks the running financial debt that an insurance company owes to the hospital system as a result of insurance discounts applied to patient payments. As insurance covers portions of appointment costs, a debt accumulates against the insurance company. The Head of Hospital monitors this balance and can record settlements. A full history of insurance policy records is accessible for reconciliation purposes.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier |
| `totalBalance` | Decimal | The current total amount owed by the insurance company |
| `lastUpdated` | DateTime | Timestamp of the most recent balance update |
| `companyName` | String | The name of the insurance company that owes the debt |

### Methods
| Method | Description |
|---|---|
| `settle(amount: Decimal)` | Records a (partial or full) payment from the insurance company, reducing the balance |
| `getHistory()` | Returns the list of all `InsurancePolicyRecord` entries that contributed to this debt |

### Relationships
- Owed to one `Hospital` (via the insurance company's obligations)
- Has a history of 1..N `InsurancePolicyRecord` entries
- Linked to one `InsurancePolicy`
- Managed by `HeadOfHospital` via `viewInsuranceDebt()` and `settleInsuranceDebt()`

---

## 22. AiAssistant

### Description
An AI-powered conversational assistant embedded within the platform, associated with each user's session. It provides contextual support by processing natural language queries from users and suggesting relevant quick actions within the application. The assistant is session-scoped, meaning it has context within a single user interaction. It can help patients understand appointment options, guide doctors through workflows, or assist administrators with system tasks.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier for the assistant instance |
| `sessionId` | String | Identifier tying the assistant to the current user session |

### Methods
| Method | Description |
|---|---|
| `processQuery(query: String)` | Accepts a natural language input and returns a contextual response |
| `suggestActions()` | Returns a list of `QuickAction` objects representing recommended next steps for the user |

### Relationships
- Assists exactly one `User` (1-to-1), scoped per session
- Associated with the `HealthMaxxinSystem` at a platform level

---

## 23. MailService

### Description
A service class responsible for all outbound email communications sent by the system to users and doctors. It centralizes notification logic, ensuring that every significant event, account creation, appointment updates, prescriptions, shift changes, and more, triggers a standardized, well-formatted email to the relevant recipient. By encapsulating all mail-sending logic in one class, the system maintains a consistent communication layer decoupled from business logic.

### Attributes
| Attribute | Type | Description |
|---|---|---|
| `id` | UUID | Unique identifier for the mail service instance |
| `senderAddress` | String | The "from" email address used for all outgoing messages |

### Methods
| Method | Description |
|---|---|
| `sendVerificationEmail(user: User)` | Sends an email verification link to a newly registered user |
| `sendPasswordReset(user: User)` | Sends a password reset link or temporary credentials |
| `sendCredentials(doctor: Doctor)` | Sends initial login credentials to a newly registered doctor |
| `sendAppointmentConfirmation(appointment: Appointment)` | Confirms a successfully booked appointment |
| `sendCancellationNotice(appointment: Appointment)` | Notifies relevant parties of an appointment cancellation |
| `sendRescheduleNotice(appointment: Appointment)` | Informs the patient and doctor of an appointment rescheduling |
| `sendPrescriptionNotice(patient: Patient)` | Alerts a patient that a new prescription has been issued to them |
| `sendExemptionNotice(patient: Patient)` | Alerts a patient that a new exemption has been granted |
| `sendExaminationNotice(patient: Patient)` | Notifies a patient that new examination results have been uploaded |
| `sendShiftNotice(doctor: Doctor)` | Informs a doctor of a new or updated shift assignment |
| `sendAccountDeletionConfirmation(user: User)` | Sends a final confirmation email when a user deletes their account |

### Relationships
- Notifies `User` for account lifecycle events (verification, password reset, deletion)
- Notifies `Doctor` for credential delivery and shift updates
- Notifies about `Appointment` events (confirmation, cancellation, rescheduling)
- Notifies `Patient` for clinical events (prescriptions, exemptions, examinations)
