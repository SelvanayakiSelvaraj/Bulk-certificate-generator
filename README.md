Bulk Certificate Generator

A backend API for generating certificates in bulk from a predefined certificate template.

The system accepts a single certificate generation request containing multiple recipients, validates the recipient information, generates individual certificates, tracks the progress of the generation job, handles individual certificate failures without stopping the entire job, and provides APIs to retrieve the generated certificates.

---

📌 Overview

Organizations often need to generate certificates for a large number of participants after events, courses, workshops, training programs, or competitions.

Generating certificates individually is time-consuming and inefficient.

The Bulk Certificate Generator solves this problem by allowing the client to submit multiple recipients in a single API request.

Key capabilities

- Submit a bulk certificate generation request
- Validate recipient information
- Generate certificates using a predefined template
- Process multiple recipients as one job
- Track job progress
- Track successful and failed certificates
- Continue processing when an individual certificate fails
- Retrieve generated certificates
- Store job and certificate information in a relational database
- Provide automated tests for important application components

---

🏗️ System Architecture

flowchart TB

    CLIENT["🌐 Client / API Consumer"]

    API["⚡ FastAPI Backend"]

    VALIDATION["✅ Request Validation<br/>Pydantic"]

    JOB["📋 Job Manager"]

    QUEUE["📨 Background Job Processing"]

    GENERATOR["📄 Certificate Generator<br/>ReportLab"]

    TEMPLATE["🎨 Predefined Certificate Template"]

    STORAGE["📁 Certificate File Storage"]

    DB["🗄️ Relational Database"]

    STATUS["📊 Job & Certificate Status"]

    DOWNLOAD["⬇️ Certificate Retrieval"]

    CLIENT --> API
    API --> VALIDATION
    VALIDATION --> JOB

    JOB --> DB
    JOB --> QUEUE

    QUEUE --> GENERATOR
    GENERATOR --> TEMPLATE
    GENERATOR --> STORAGE

    GENERATOR --> STATUS
    STATUS --> DB

    API --> STATUS
    API --> DOWNLOAD
    DOWNLOAD --> STORAGE

    DB --> API
    STORAGE --> API

---

🔄 Application Workflow

flowchart LR

    A["👤 Client"] --> B["📤 Submit Recipients"]
    B --> C["⚡ Create Generation Job"]
    C --> D["✅ Validate Input"]

    D -->|Valid| E["📋 Create Job Records"]
    D -->|Invalid| F["❌ Return Validation Error"]

    E --> G["⚙️ Background Processing"]

    G --> H["👤 Process Recipient"]
    H --> I["🎨 Apply Certificate Template"]
    I --> J["📄 Generate Certificate"]

    J -->|Success| K["💾 Store Certificate"]
    J -->|Failure| L["❌ Mark Recipient Failed"]

    K --> M["📊 Update Progress"]
    L --> M

    M --> N{"More Recipients?"}

    N -->|Yes| H
    N -->|No| O["✅ Complete Job"]

    O --> P["📊 Client Checks Status"]
    P --> Q["⬇️ Retrieve Certificates"]

---

🧱 High-Level Block Diagram

flowchart TB

    subgraph USER["👤 CLIENT LAYER"]
        CLIENT["API Client"]
        POSTMAN["Postman / Frontend"]
    end

    subgraph BACKEND["⚡ APPLICATION LAYER"]
        API["FastAPI"]
        ROUTES["API Routes"]
        SERVICE["Business Logic"]
        VALIDATOR["Input Validation"]
        JOB_SERVICE["Job Service"]
        CERT_SERVICE["Certificate Service"]
    end

    subgraph PROCESSING["⚙️ PROCESSING LAYER"]
        WORKER["Background Worker"]
        GENERATOR["Certificate Generator"]
        TEMPLATE["Certificate Template"]
    end

    subgraph DATA["💾 DATA LAYER"]
        DATABASE["Relational Database"]
        FILES["Certificate Storage"]
    end

    CLIENT --> API
    POSTMAN --> API

    API --> ROUTES
    ROUTES --> VALIDATOR
    VALIDATOR --> SERVICE

    SERVICE --> JOB_SERVICE
    SERVICE --> CERT_SERVICE

    JOB_SERVICE --> WORKER
    WORKER --> GENERATOR
    GENERATOR --> TEMPLATE

    JOB_SERVICE --> DATABASE
    CERT_SERVICE --> DATABASE
    GENERATOR --> FILES

    API --> DATABASE
    API --> FILES

---

🧩 Component Architecture

flowchart LR

    subgraph API["API"]
        A1["POST /jobs"]
        A2["GET /jobs/{job_id}"]
        A3["GET /jobs/{job_id}/certificates"]
        A4["GET /certificates/{certificate_id}"]
    end

    subgraph SERVICES["Services"]
        S1["Job Service"]
        S2["Validation Service"]
        S3["Certificate Service"]
        S4["Storage Service"]
    end

    subgraph DATABASE["Database"]
        D1["Generation Jobs"]
        D2["Recipients"]
        D3["Certificates"]
    end

    subgraph STORAGE["File Storage"]
        F1["Generated PDF Files"]
    end

    A1 --> S2
    A1 --> S1

    A2 --> S1
    A3 --> S3
    A4 --> S4

    S1 --> D1
    S1 --> D2
    S3 --> D3
    S4 --> F1

---

🛠️ Technology Stack

Technology| Purpose
Python| Backend programming language
FastAPI| REST API framework
Pydantic| Request and response validation
SQLAlchemy| Database ORM
SQLite / PostgreSQL| Relational database
ReportLab| PDF certificate generation
Background Worker| Bulk certificate processing
Pytest| Automated testing
Uvicorn| ASGI application server
Git & GitHub| Version control

«The assignment permits FastAPI, Django REST Framework, or Flask and requires a relational database. The certificate generation library and processing approach are developer choices.»

---

📁 Project Structure

bulk-certificate-generator/
│
├── app/
│   ├── main.py
│   │
│   ├── api/
│   │   ├── routes/
│   │   │   ├── jobs.py
│   │   │   └── certificates.py
│   │   │
│   │   └── dependencies.py
│   │
│   ├── models/
│   │   ├── job.py
│   │   ├── recipient.py
│   │   └── certificate.py
│   │
│   ├── schemas/
│   │   ├── job.py
│   │   ├── recipient.py
│   │   └── certificate.py
│   │
│   ├── services/
│   │   ├── job_service.py
│   │   ├── certificate_service.py
│   │   ├── validation_service.py
│   │   └── storage_service.py
│   │
│   ├── workers/
│   │   └── certificate_worker.py
│   │
│   ├── utils/
│   │   └── certificate_generator.py
│   │
│   └── database.py
│
├── templates/
│   └── certificate_template.pdf
│
├── generated/
│   └── certificates/
│
├── tests/
│   ├── test_jobs.py
│   ├── test_validation.py
│   ├── test_generation.py
│   ├── test_status.py
│   ├── test_failure_handling.py
│   └── test_retrieval.py
│
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md

---

🗄️ Database Design

The application uses a relational database to maintain job, recipient, and certificate information.

erDiagram

    GENERATION_JOB ||--o{ RECIPIENT : contains
    RECIPIENT ||--o| CERTIFICATE : generates

    GENERATION_JOB {
        string id PK
        string status
        int total_count
        int successful_count
        int failed_count
        datetime created_at
        datetime completed_at
    }

    RECIPIENT {
        string id PK
        string job_id FK
        string name
        string email
        string status
        string error_message
    }

    CERTIFICATE {
        string id PK
        string recipient_id FK
        string file_path
        string status
        datetime generated_at
    }

Generation Job

Stores overall information about a bulk generation request.

Example statuses:

PENDING
PROCESSING
COMPLETED
COMPLETED_WITH_ERRORS
FAILED

Recipient

Stores recipient-specific information and generation status.

Example statuses:

PENDING
PROCESSING
SUCCESS
FAILED

Certificate

Stores the generated certificate information and file location.

---

📤 API Endpoints

1. Create Generation Job

POST /jobs

Creates a new bulk certificate generation job.

Example request

{
  "certificate_title": "Certificate of Participation",
  "event_name": "Python Workshop",
  "date": "2026-10-08",
  "recipients": [
    {
      "name": "John Doe",
      "email": "john@example.com"
    },
    {
      "name": "Jane Doe",
      "email": "jane@example.com"
    }
  ]
}

Example response

{
  "job_id": "JOB-001",
  "status": "PENDING",
  "total_count": 2
}

---

📊 2. Check Job Status

GET /jobs/{job_id}

Returns the current progress of the generation job.

Example response

{
  "job_id": "JOB-001",
  "status": "PROCESSING",
  "total": 100,
  "successful": 72,
  "failed": 2,
  "pending": 26,
  "progress": 74
}

This allows the client to monitor a large certificate generation request without submitting another generation request.

---

📄 3. List Generated Certificates

GET /jobs/{job_id}/certificates

Returns certificates associated with the specified job.

Example:

{
  "job_id": "JOB-001",
  "certificates": [
    {
      "certificate_id": "CERT-001",
      "recipient_name": "John Doe",
      "status": "SUCCESS"
    }
  ]
}

---

⬇️ 4. Retrieve Certificate

GET /certificates/{certificate_id}

Returns/downloads the generated certificate file.

---

🔍 Validation

The API validates recipient information before processing.

Example validation rules:

- Recipient name must be provided
- Recipient email must be valid when email is required
- Recipient list must not be empty
- Certificate information must be provided
- Required fields must have valid values

Invalid data should produce a meaningful API response instead of starting an invalid generation job.

---

❌ Failure Handling

A major requirement of the system is that one failed certificate must not unnecessarily stop the complete bulk job.

For example:

100 recipients
      │
      ├── Certificate 1  → ✅ Success
      ├── Certificate 2  → ✅ Success
      ├── Certificate 3  → ❌ Failed
      ├── Certificate 4  → ✅ Success
      ├── Certificate 5  → ✅ Success
      │
      └── Continue processing...

At the end of the job:

Total       : 100
Successful  : 97
Failed      : 3

The failed records should contain enough information to identify the reason for failure.

Example:

{
  "recipient": "John Doe",
  "status": "FAILED",
  "error_message": "Unable to generate certificate"
}

---

⚙️ Bulk Processing Strategy

The system is designed around a single bulk request rather than requiring one API request per recipient.

Why background processing?

Certificate generation can take time when the request contains a large number of recipients.

Therefore, the API can:

1. Accept the request
2. Create a job
3. Store recipient information
4. Return the job ID
5. Process certificates in the background
6. Update progress continuously
7. Allow the client to check status

sequenceDiagram

    participant C as Client
    participant API as FastAPI
    participant DB as Database
    participant W as Worker
    participant PDF as PDF Generator
    participant FS as File Storage

    C->>API: POST /jobs
    API->>DB: Create Job
    API->>DB: Store Recipients
    API-->>C: Return Job ID

    API->>W: Start Background Job

    loop Each Recipient
        W->>DB: Mark Processing
        W->>PDF: Generate Certificate
        PDF->>FS: Save PDF

        alt Generation Successful
            W->>DB: Mark Success
        else Generation Failed
            W->>DB: Mark Failed
        end
    end

    C->>API: GET /jobs/{id}
    API->>DB: Read Status
    DB-->>API: Progress
    API-->>C: Return Status

    C->>API: GET /certificates/{id}
    API->>FS: Read Certificate
    FS-->>API: Certificate File
    API-->>C: Download Certificate

---

🎨 Certificate Generation

The application uses one predefined certificate template.

The template contains fixed design elements such as:

┌─────────────────────────────────────────────┐
│                                             │
│              CERTIFICATE                    │
│             OF PARTICIPATION                │
│                                             │
│             This is awarded to              │
│                                             │
│               JOHN DOE                      │
│                                             │
│       for participating in Python Workshop  │
│                                             │
│                08-10-2026                   │
│                                             │
│          ____________________               │
│               Organizer                     │
│                                             │
└─────────────────────────────────────────────┘

Recipient-specific information is inserted dynamically during generation.

---

🔄 Certificate Generation Flow

flowchart TD

    START["📥 Recipient Data"]

    VALIDATE{"✅ Valid Data?"}

    TEMPLATE["🎨 Load Predefined Template"]

    INSERT["✏️ Insert Recipient Information"]

    GENERATE["📄 Generate PDF"]

    SAVE["💾 Save Certificate"]

    UPDATE["📊 Update Database"]

    SUCCESS["✅ SUCCESS"]

    FAILURE["❌ FAILED"]

    START --> VALIDATE

    VALIDATE -->|No| FAILURE
    VALIDATE -->|Yes| TEMPLATE

    TEMPLATE --> INSERT
    INSERT --> GENERATE

    GENERATE -->|Success| SAVE
    SAVE --> UPDATE
    UPDATE --> SUCCESS

    GENERATE -->|Error| FAILURE

---

📈 Job Progress Tracking

For every generation job, the system maintains:

Total Certificates
Successful Certificates
Failed Certificates
Pending Certificates
Current Status
Progress Percentage

Progress can be calculated as:

Progress =
((Successful + Failed) / Total) × 100

Example:

Total      = 50
Successful = 40
Failed     = 2
Pending    = 8

Progress = ((40 + 2) / 50) × 100
         = 84%

---

🧪 Testing

The project includes tests covering the important parts of the application.

Required test areas

✓ Creating a generation job
✓ Input validation
✓ Certificate generation
✓ Job status/progress
✓ Individual certificate failure
✓ Certificate retrieval

Example test structure:

tests/
│
├── test_jobs.py
│   └── test_create_generation_job()
│
├── test_validation.py
│   ├── test_valid_recipient()
│   └── test_invalid_recipient()
│
├── test_generation.py
│   └── test_certificate_generation()
│
├── test_status.py
│   └── test_job_progress()
│
├── test_failure_handling.py
│   └── test_single_certificate_failure()
│
└── test_retrieval.py
    └── test_certificate_retrieval()

Run tests:

pytest

---

🚀 Installation

1. Clone the repository

git clone https://github.com/<your-username>/bulk-certificate-generator.git
cd bulk-certificate-generator

2. Create a virtual environment

Windows

python -m venv venv
venv\Scripts\activate

Linux / macOS

python3 -m venv venv
source venv/bin/activate

3. Install dependencies

pip install -r requirements.txt

4. Configure environment variables

Create a ".env" file:

DATABASE_URL=sqlite:///./certificates.db
CERTIFICATE_STORAGE_PATH=generated/certificates

For PostgreSQL:

DATABASE_URL=postgresql://username:password@localhost/certificates

---

▶️ Running the Application

Start the FastAPI server:

uvicorn app.main:app --reload

The API will be available at:

http://127.0.0.1:8000

FastAPI interactive documentation:

http://127.0.0.1:8000/docs

---

📤 Example Usage

Step 1 — Create a job

POST /jobs

{
  "certificate_title": "Certificate of Participation",
  "event_name": "AI Workshop",
  "date": "2026-10-08",
  "recipients": [
    {
      "name": "Alice",
      "email": "alice@example.com"
    },
    {
      "name": "Bob",
      "email": "bob@example.com"
    }
  ]
}

Step 2 — Receive Job ID

{
  "job_id": "JOB-123",
  "status": "PENDING",
  "total_count": 2
}

Step 3 — Check Status

GET /jobs/JOB-123

Response:

{
  "job_id": "JOB-123",
  "status": "COMPLETED",
  "total": 2,
  "successful": 2,
  "failed": 0,
  "progress": 100
}

Step 4 — Retrieve Certificates

GET /jobs/JOB-123/certificates

Then retrieve an individual certificate:

GET /certificates/CERT-123

---

🔐 Error Handling

The API should return meaningful HTTP responses.

Situation| Response
Invalid request| "400 Bad Request"
Invalid recipient data| "422 Unprocessable Entity"
Job not found| "404 Not Found"
Certificate not found| "404 Not Found"
Successful job creation| "201 Created"
Successful status request| "200 OK"
Successful certificate retrieval| "200 OK"

---

📊 Design Decisions

1. Why FastAPI?

FastAPI provides:

- Simple REST API development
- Automatic request validation
- Automatic API documentation
- Good performance
- Type-hint based development

2. Why a relational database?

The application has structured relationships between:

Generation Job
      ↓
Recipients
      ↓
Certificates

A relational database provides a suitable structure for maintaining these relationships and tracking generation status.

3. Why background processing?

Bulk certificate generation may take longer as the number of recipients increases.

Background processing allows the API to return a job ID quickly while the certificates are generated separately.

4. Why individual status tracking?

If one certificate fails, the remaining certificates should continue processing.

Therefore, each recipient/certificate maintains its own status.

---

🛡️ Failure Isolation

The processing logic follows an isolated approach:

for recipient in recipients:

    try:
        generate_certificate(recipient)
        mark_success(recipient)

    except Exception as error:
        mark_failed(recipient, str(error))

    # Continue with the next recipient

This prevents a single certificate-generation error from stopping the entire bulk operation.

---

📂 Generated Files

Generated certificates can be organized by job:

generated/
└── certificates/
    ├── JOB-001/
    │   ├── john_doe.pdf
    │   ├── jane_doe.pdf
    │   └── robert_smith.pdf
    │
    └── JOB-002/
        ├── alice.pdf
        └── bob.pdf

The database stores the corresponding file path for retrieval.

---

🔮 Future Enhancements

The assignment does not require these features, but they can be added later:

- Email certificates automatically
- ZIP download for an entire job
- Certificate verification using unique IDs
- QR code on certificates
- Multiple certificate templates
- Admin dashboard
- Cloud storage such as AWS S3
- Authentication and authorization
- Retry failed certificates
- Job cancellation
- Scheduled certificate generation
- Detailed generation analytics

---

📌 Project Requirements

The implementation satisfies the core requirements of the assignment:

✓ Python backend
✓ REST API
✓ Relational database
✓ Bulk certificate generation
✓ Predefined certificate template
✓ Recipient validation
✓ Job creation
✓ Generation status tracking
✓ Progress tracking
✓ Individual failure handling
✓ Certificate retrieval
✓ Automated tests
✓ Project documentation

---

📜 License

This project is developed for educational and technical assessment purposes.

---

👩‍💻 Author

Selvanayaki S

GitHub: https://github.com/SelvanayakiSelvaraj
