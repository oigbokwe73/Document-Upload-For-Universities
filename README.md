# Document-Upload-For-Universities


Below is a **detailed, implementation-ready use case** tailored to the workflow you requested, aligned with your Azure architecture style: serverless, event-driven, scalable, and integrated with Azure Document Intelligence + SQL + REST API.

---

# 🎓 **Use Case: Certificate Verification & Metadata Extraction for Colleges/Universities**

## **Business Problem**

Colleges and universities need a secure, automated method for verifying student academic certificates (transcripts, diplomas, vocational certificates). Many institutions still manually upload documents into systems for validation, creating bottlenecks and increasing operational costs.

The university wants to **automate certificate ingestion, scanning, metadata extraction, verification, and storage**, allowing administrators and students to view certificate metadata and the original document through a secure REST API.

---

# 🚀 **End-to-End Workflow Summary**

1. **Colleges generate & upload certificate PDFs** via SFTP.
2. **Files automatically land in Azure Blob Storage** (SFTP-enabled container).
3. **Azure Event Grid** detects new uploads and triggers downstream processing.
4. **Azure Function (Document Processing Function)** runs asynchronously:

   * Pulls the PDF document from Blob Storage
   * Sends it to Azure Document Intelligence (Document AI)
   * Extracts structured metadata (Name, DOB, GPA, Degree, Graduation Year, Issuing Department, etc.)
   * Stores extracted metadata + blob URL reference in **Azure SQL Database**
5. **RESTful API (via Azure App Service / Azure Functions)** enables:

   * Querying certificate metadata (by student ID, year, name, etc.)
   * Viewing or downloading the original document from Blob Storage (Managed Identity + SAS)
6. Optional: **Power BI dashboard** for certificate verification analytics.

---

# 🧭 **Detailed Use Case Scenario**

A university receives thousands of academic certificates annually from partner colleges. Certificates arrive in PDF format with variations in layout, structure, and quality. Each must be:

* **Authenticated**
* **Parsed**
* **Indexed**
* **Stored for future retrieval**

Instead of building manual intake workflows, the university deploys the following automated solution using Azure.

---

# 🧩 **Azure Architecture Components**

| Layer      | Azure Service                        | Purpose                                                     |
| ---------- | ------------------------------------ | ----------------------------------------------------------- |
| Ingestion  | **SFTP-enabled Azure Blob Storage**  | Colleges upload PDFs securely using SFTP credentials        |
| Eventing   | **Event Grid**                       | Detects file uploads and triggers processing                |
| Processing | **Azure Functions**                  | Runs Document Intelligence, extracts metadata, saves to SQL |
| AI         | **Azure Document Intelligence**      | OCR + key-value pair extraction                             |
| Storage    | **Azure Blob Storage**               | Holds original certificate files                            |
| Database   | **Azure SQL Database**               | Stores metadata, processing logs, verification status       |
| API        | **Azure App Service / Function API** | Presents metadata & document-access endpoints               |
| Security   | **Managed Identity, RBAC, SAS**      | Secure document access, restricted metadata retrieval       |

---

# 🌐 **Mermaid Architecture Diagram**

```mermaid
flowchart TD

A[SFTP Client (College/University)] -->|Upload Certificate PDF| B[Azure Blob Storage <br> SFTP Enabled Container]

B -->|Blob Created Event| C[Event Grid]

C -->|Trigger| D[Azure Function: ProcessCertificate]

D -->|Read PDF| B
D -->|Send to Document AI| E[Azure Document Intelligence]

E -->|Extracted Metadata (JSON)| D

D -->|Insert Metadata + Blob URL| F[Azure SQL Database]

G[REST API (Azure App Service/Function)] -->|Query Metadata| F
G -->|Generate SAS URI| B

H[Admin/Student Portal] -->|View Certificate Metadata| G
H -->|Download Certificate| G
```

---

# 📄 **Metadata Extracted from Document Intelligence**

* StudentName
* DateOfBirth
* StudentID
* CertificateType
* Degree / Program
* GPA (optional)
* Issuing College
* GraduationDate
* TranscriptNumber
* DocumentConfidenceScore
* BlobStorageURL

---

# 🗃 **SQL Table Example**

```sql
CREATE TABLE CertificateMetadata (
    CertificateId INT IDENTITY PRIMARY KEY,
    StudentName NVARCHAR(200),
    StudentId NVARCHAR(100),
    DateOfBirth DATE,
    CertificateType NVARCHAR(100),
    DegreeProgram NVARCHAR(200),
    GPA NVARCHAR(20),
    IssuingUniversity NVARCHAR(200),
    GraduationDate DATE,
    BlobUrl NVARCHAR(MAX),
    ProcessedDate DATETIME DEFAULT GETDATE()
);
```

---

# ⚙️ **Azure Function – Processing Flow**

### **1. Trigger**

```json
{
  "eventType": "Microsoft.Storage.BlobCreated",
  "data": {
    "url": "<blob-url>",
    ...
  }
}
```

### **2. Function Steps**

1. Function retrieves blob using Managed Identity
2. Sends the PDF to Document Intelligence `prebuilt-document` or custom model
3. Receives structured JSON
4. Maps JSON fields → SQL data model
5. Inserts into SQL
6. Logs errors or AI exceptions

---

# 🔗 **REST API Endpoints**

### **1. GET: List Certificates**

`GET /api/certificates?studentId=12345`

### **2. GET: Certificate by ID**

`GET /api/certificates/{certificateId}`

### **Response:**

```json
{
  "certificateId": 101,
  "studentName": "Jane Doe",
  "degreeProgram": "Computer Science",
  "graduationDate": "2023-06-15",
  "blobUrl": "<SAS URL>",
  "confidenceScore": 0.98
}
```

### **3. GET: Download Certificate**

`GET /api/certificates/{certificateId}/download`

* API returns a **short-lived SAS token**
* Client downloads directly from Blob Storage

---

# 🔐 **Security Highlights**

* Colleges authenticate using **SFTP username/password or SSH keypair**
* Blob Storage uses **private access only**
* API uses **Azure AD authentication**
* Blob access uses **User Delegation SAS** generated by API
* Azure Functions & API use **Managed Identity**
* SQL also secured with **AAD authentication**

---

# 📊 **Benefits**

### **For Universities**

* Automated, paperless certificate intake
* AI-based extraction reduces manual review
* Fast validation + improved accuracy
* Standardized metadata for reporting
* Easily integrates with SIS (Student Information System)

