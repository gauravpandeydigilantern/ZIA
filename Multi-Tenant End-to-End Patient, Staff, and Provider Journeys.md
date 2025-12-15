## Multi-Tenant End-to-End Patient, Staff, and Provider Journeys

---

## 1. Patient End-to-End Workflow

**Omnichannel Journey: First Contact → AI Interaction → Scheduling → Encounter → Follow-Up**

```mermaid
flowchart TD
    A[Patient Contacts Clinic] --> B{Channel?}
    B -->|Phone AI| C[Telnyx Voice + AI Agent A]
    B -->|SMS| D[Telnyx SMS + AI Agent A]
    B -->|Web Chat| E[WebRTC/SignalR + AI Agent A]
    B -->|Portal| F[Next.js Patient Login]
    B -->|Walk-In| G[Staff In-Person Check-in]
    
    C --> H[HIPAA Disclosure + Org Context]
    D --> H
    E --> H
    F --> I[Azure AD B2C + Org Context]
    G --> J[Staff Dashboard - Org Filtered]
    
    H --> K[Identity Verification]
    I --> L[Load Patient Profile - Org Scoped]
    J --> M[Manual Patient Entry]
    
    K --> N{New or Existing?}
    N -->|New| O[Send JotForm Intake Links via SMS]
    N -->|Existing| L
    
    O --> P[Patient Completes Forms]
    P --> Q[Webhook → AI Profile Generation GPT-4o]
    Q --> R[Staff Approves AI Profile]
    R --> S[Patient Ready for Service]
    
    L --> T{What do you need?}
    S --> T
    M --> T
    
    T -->|Clinical Question| U[Agent B Clinical AI + Org RAG]
    T -->|Schedule Appointment| V[Capture Scheduling Request]
    T -->|Emergency Keywords| W[Immediate Emergency Protocol]
    T -->|Forms/Insurance| X[Route to Staff Queue]
    
    U --> Y[AI Generates Response with Citations]
    Y --> Z[100% Judge LLM Evaluation]
    Z --> AA[5 Criteria: Safety, Accuracy, Citations, Empathy, Compliance]
    AA --> AB{Judge Decision?}
    AB -->|Pass ≥85| AC[Send Response to Patient]
    AB -->|Revise 70-84| AD[Auto-Revision Loop Max 2x]
    AB -->|Escalate <70| AE[Provider Escalation Queue]
    AD --> Z
    
    AE --> AF[Provider Reviews + Corrects]
    AF --> AG[Provider Sends Corrected Response]
    AG --> AH[Log Full Lineage for Learning]
    
    V --> AI[Staff Reviews in Retool Dashboard]
    AI --> AJ[Check Insurance RTE - Office Ally]
    AJ --> AK[Staff Confirms Appointment]
    AK --> AL[Send Multi-Channel Confirmation]
    AL --> AM[Schedule 24h + 2h Reminders]
    
    AM --> AN{Visit Type?}
    AN -->|Telehealth| AO[MS Teams Meeting Link]
    AN -->|In-Person| AP[Exam Room Booking]
    
    AO --> AQ[Provider Encounter with Twofold/HealOS]
    AP --> AQ
    AQ --> AR[Real-Time AI Transcription]
    AR --> AS[AI Generates SOAP + ICD-10]
    AS --> AT[Provider Reviews & Approves]
    AT --> AU[Generate Visit Summary]
    AU --> AV[Send to Patient Portal]
    AV --> AW[24h Post-Visit Follow-Up SMS/Email]
    AW --> AX[Generate Claim → Office Ally]
    
    W --> AY[SMS Alert to Provider + Dashboard]
    AY --> AZ[Provider Immediate Action]
    AZ --> BA[Call Patient / ER Referral]
    
    AC --> BB[Patient Continues or Satisfied]
    AH --> BB
    AX --> BC[Patient Care Journey Complete]
    BA --> BC
```

**Key Patient Touchpoints:**
- **Omnichannel Access**: 5 channels (Phone, SMS, Web, Portal, Walk-In)
- **100% AI Safety**: Judge LLM validates every clinical response
- **Unified Profile**: One profile across all channels (no data silos)
- **Smart Intake**: AI-generated clinical profiles from forms
- **Evidence-Based**: Responses cite ACOG, CDC, DynaMed, UpToDate
- **Post-Visit Engagement**: Automated follow-up templates

---

## 2. Staff End-to-End Workflow

**Centralized Operations: Patient Onboarding → Scheduling → Insurance → Claims → Monitoring**

```mermaid
flowchart TD
    A[Staff Login - Azure AD B2C] --> B[Retool Operations Dashboard]
    B --> C[Org-Filtered View Loads]
    C --> D[Today's Overview KPIs]
    
    D --> E[New Patient Onboarding Queue]
    D --> F[Scheduling Requests Queue]
    D --> G[Callback Queue]
    D --> H[Insurance Verification Queue]
    D --> I[Claims Submission Queue]
    D --> J[Intake Monitoring Dashboard]
    D --> K[Emergency Alerts]
    
    E --> L[Open New Patient Record]
    L --> M[Verify Identity - Name, DOB, Phone]
    M --> N[Run Office Ally RTE Check]
    N --> O{Insurance Valid?}
    O -->|No| P[Flag Issue + Contact Patient]
    O -->|Yes| Q[Store Insurance Details]
    P --> R[Patient Updates Insurance]
    R --> N
    Q --> S[Assign Primary Provider]
    S --> T[Generate JotForm Links - Batch SMS]
    T --> U[HIPAA + Telehealth + Intake + Clinical History]
    U --> V[Monitor Form Completion]
    V --> W{Complete in 48h?}
    W -->|No| X[Send Reminder SMS at 48h, 72h]
    W -->|Yes| Y[AI Profile Auto-Generated]
    X --> Z{Complete in 5 Days?}
    Z -->|No| AA[Escalate to Staff Call]
    Z -->|Yes| Y
    Y --> AB[Staff Reviews AI Clinical Profile]
    AB --> AC{Profile Accurate?}
    AC -->|Yes| AD[Approve - Patient Ready]
    AC -->|No| AE[Manual Edits]
    AE --> AD
    AD --> AF[Send Welcome Message]
    
    F --> AG[Open Scheduling Request]
    AG --> AH[Load Patient Clinical Profile]
    AH --> AI[View: Symptoms, Urgency, Preferences]
    AI --> AJ{Urgent or Routine?}
    AJ -->|Urgent| AK[Priority Scheduling - Contact Provider]
    AJ -->|Routine| AL[Standard Scheduling Flow]
    AK --> AM[Find Immediate Slot]
    AL --> AN[Check Provider Calendar]
    AN --> AO[Match Date/Time Preferences]
    AM --> AP[Confirm Appointment]
    AO --> AP
    AP --> AQ{Visit Type?}
    AQ -->|Telehealth| AR[Generate MS Teams Link]
    AQ -->|In-Person| AS[Book Exam Room]
    AR --> AT[Store Meeting Details]
    AS --> AT
    AT --> AU[Send Multi-Channel Confirmation]
    AU --> AV[SMS + Email + Portal Update]
    AV --> AW[Schedule 24h + 2h Reminders]
    
    G --> AX[Review Callback Request]
    AX --> AY[Assign to Provider or Staff]
    AY --> AZ[Schedule Callback Time]
    AZ --> BA[Send Confirmation to Patient]
    
    H --> BB[Batch RTE Checks for Upcoming]
    BB --> BC[Run Office Ally RTE API]
    BC --> BD{All Valid?}
    BD -->|Yes| BE[Update Insurance Status]
    BD -->|No| BF[Flag Issues for Resolution]
    BF --> BG[Contact Patients with Issues]
    
    I --> BH[Review Post-Visit Claims]
    BH --> BI[Validate ICD-10 + CPT Codes]
    BI --> BJ[From Twofold/HealOS AI Documentation]
    BJ --> BK[Review Claim Accuracy]
    BK --> BL{Claim Valid?}
    BL -->|Yes| BM[Submit to Office Ally API]
    BL -->|No| BN[Flag for Correction]
    BN --> BO[Edit Codes]
    BO --> BK
    BM --> BP[Track Claim Status]
    BP --> BQ{Status?}
    BQ -->|Accepted| BR[Monitor Payment]
    BQ -->|Rejected| BS[Review Rejection Reason]
    BS --> BT[Correct & Resubmit]
    
    J --> BU[Monitor Intake Completion Rates]
    BU --> BV[Filter: Incomplete > 72h]
    BV --> BW[Proactive Outreach]
    
    K --> BX[Emergency Alert - Red Banner]
    BX --> BY[Review Emergency Context]
    BY --> BZ[Coordinate with Provider]
    BZ --> CA[Document Actions Taken]
    
    AF --> CB[Patient Onboarded Successfully]
    AW --> CC[Appointment Confirmed]
    BA --> CD[Callback Scheduled]
    BE --> CE[Insurance Verified]
    BR --> CF[Claim Payment Tracked]
    BW --> CG[Intake Completion Improved]
    CA --> CH[Emergency Handled]
    
    CB --> CI[Staff Operations Complete]
    CC --> CI
    CD --> CI
    CE --> CI
    CF --> CI
    CG --> CI
    CH --> CI
```

**Key Staff Workflows:**
- **Org-Scoped Dashboard**: Multi-tenant data isolation in Retool
- **RTE Integration**: Real-time eligibility checks via Office Ally
- **Claims Management**: Complete submission and tracking workflow
- **Proactive Monitoring**: Intake completion, insurance verification
- **Emergency Coordination**: Real-time alerts and provider communication
- **Efficiency Metrics**: Time to schedule, insurance verification rate, intake completion %

---

## 3. Provider End-to-End Workflow

**Clinical Excellence: AI Escalations → Emergency Response → AI-Native Encounters → Learning Loop**

```mermaid
flowchart TD
    A[Provider Login - Azure AD B2C] --> B[Retool Provider Dashboard]
    B --> C[Org-Filtered Provider View]
    C --> D[Today's Schedule]
    C --> E[Clinical Escalation Queue - Badge Count]
    C --> F[Emergency Alerts - Red Banner]
    C --> G[Patient Messages]
    
    E --> H[Judge LLM Escalated Case]
    H --> I[Load Complete Patient Context]
    I --> J[Demographics + Clinical Profile]
    I --> K[Full Conversation History]
    I --> L[Recent Encounters + Meds]
    
    J --> M[View AI Response Attempt]
    K --> M
    L --> M
    M --> N[Judge LLM 5-Criteria Scores]
    N --> O[Safety: X/100 + Reasoning]
    N --> P[Accuracy: X/100 + Reasoning]
    N --> Q[Citations: X/100 + Reasoning]
    N --> R[Empathy: X/100 + Reasoning]
    N --> S[Compliance: X/100 + Reasoning]
    
    O --> T[Review RAG Citations]
    P --> T
    Q --> T
    R --> T
    S --> T
    T --> U[ACOG, CDC, DynaMed, UpToDate, Org Protocols]
    
    U --> V{Provider Decision?}
    V -->|Approve AI Response| W[Approve with Comment]
    V -->|Minor Edit| X[Inline Edit - Track Changes]
    V -->|Major Revision| Y[Complete Rewrite]
    V -->|Escalate Further| Z[Forward to Senior Provider]
    
    W --> AA[Tag: Approved Despite Flag]
    X --> AB[Tag Correction Category]
    Y --> AB
    AB --> AC[Safety / Accuracy / Citations / Empathy / Compliance]
    
    AA --> AD[Log Complete Data Lineage]
    AC --> AD
    AD --> AE[Original Q → AI Attempt → Judge Scores → Provider Edit → Final Message]
    AE --> AF[Apply 5-Criteria Labels for Learning]
    AF --> AG[Store in LearningDatasets Table]
    AG --> AH[Send Corrected Response to Patient]
    AH --> AI[Close Escalation Case]
    
    F --> AJ[Emergency Alert - SMS + Dashboard]
    AJ --> AK[Review Emergency Context]
    AK --> AL[Patient Message + Keywords Detected]
    AL --> AM{Action Required?}
    AM -->|Call Patient Now| AN[Immediate Phone Call]
    AM -->|Instruct 911| AO[Tell Patient Call 911]
    AM -->|Same-Day Urgent| AP[Schedule Emergency Slot]
    AM -->|ER Referral| AQ[Direct to Emergency Room]
    AM -->|False Positive| AR[Dismiss with Note]
    
    AN --> AS[Document Emergency Call]
    AO --> AS
    AP --> AS
    AQ --> AS
    AR --> AS
    AS --> AT[Update EmergencyIncidents Table]
    AT --> AU[Close Emergency Case]
    
    D --> AV[View Today's Appointments]
    AV --> AW[Select Upcoming Encounter]
    AW --> AX[Pre-Visit Preparation 5-10 Min]
    AX --> AY[Load Patient Clinical Profile]
    AX --> AZ[AI-Generated Visit Summary]
    AX --> BA[Recent AI Conversations Last 7 Days]
    AX --> BB[Previous Encounter Notes]
    AX --> BC[Current Meds + Allergies + Labs]
    
    AY --> BD{Visit Type?}
    AZ --> BD
    BA --> BD
    BB --> BD
    BC --> BD
    
    BD -->|Telehealth| BE[Launch MS Teams Meeting]
    BD -->|In-Person| BF[Enter Exam Room]
    
    BE --> BG[Start Twofold/HealOS AI Documentation]
    BF --> BG
    BG --> BH[Select Encounter Type]
    BH --> BI[Initial / Follow-Up / Annual / Urgent]
    BI --> BJ[Real-Time Audio Capture Activated]
    
    BJ --> BK[Begin Clinical Encounter]
    BK --> BL[Provider-Patient Conversation]
    BL --> BM[Twofold/HealOS Processing]
    BM --> BN[Real-Time Speech-to-Text]
    BM --> BO[Medical Entity Recognition]
    BM --> BP[Structured Data Extraction]
    
    BN --> BQ[Live SOAP Note Building]
    BO --> BQ
    BP --> BQ
    BQ --> BR[Provider Can View During Call]
    
    BL --> BS[Encounter Complete]
    BS --> BT[AI Generates Full SOAP Note]
    BT --> BU[Subjective + Objective + Assessment + Plan]
    BU --> BV[AI Suggests ICD-10 Codes]
    BV --> BW[AI Suggests CPT Codes]
    BW --> BX[Provider Reviews SOAP in UI]
    
    BX --> BY{SOAP Accurate?}
    BY -->|Yes| BZ[Approve SOAP - Sign Off]
    BY -->|No| CA[Edit SOAP - Corrections]
    CA --> CB[Make Changes]
    CB --> BZ
    
    BZ --> CC[Finalize Encounter Documentation]
    CC --> CD[Generate Patient Visit Summary]
    CD --> CE[Plain Language Instructions]
    CE --> CF[Medications + Follow-Up Plan]
    CF --> CG[Send to Patient Portal]
    CG --> CH[Store All Documents in Azure Blob]
    CH --> CI[SOAP PDF + Visit Summary + Transcript]
    CI --> CJ[Generate FHIR Encounter Resource]
    CJ --> CK[Auto-Generate Claim Data]
    CK --> CL[Staff Reviews Claim → Office Ally]
    CL --> CM[Trigger 24h Post-Visit Follow-Up]
    
    AI --> CN[Escalation Workflow Complete]
    AU --> CO[Emergency Workflow Complete]
    CM --> CP[Encounter Workflow Complete]
    
    CN --> CQ[Provider Daily Tasks Complete]
    CO --> CQ
    CP --> CQ
```

**Key Provider Workflows:**
- **Detailed Escalation Review**: Full context + Judge scores + citations + correction tagging
- **Emergency Response**: Real-time alerts with immediate action options
- **AI-Native Documentation**: Twofold/HealOS for ambient transcription (telehealth + in-person)
- **Evidence-Based Care**: RAG citations from DynaMed, UpToDate, ACOG, CDC
- **Complete Data Lineage**: Full chain from AI attempt to final message for learning
- **5-Criteria Labeling**: Safety, Accuracy, Citations, Empathy, Compliance tags
- **Seamless Billing**: Auto-generated claims from AI documentation
- **Post-Visit Automation**: Visit summary to portal + 24h follow-up triggered
