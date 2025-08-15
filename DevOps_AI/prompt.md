# i want to create a ai agent application which can take various type of input like ppt , excel , csv , word file contain data related to it company employee , which certification done , which are pending , which mandatory training are pending , project , supervisor , expertise etc. then the application store the files in azure file share , process the data using azure ai foundry application and use the data to create a employee profile containing all the data incorporated along with that store processed data in a separate Db which can provide list . entire infrastructure will be hosted in azure , terraform will be used as infrastructure creation , ansible will be used as configuration management , cicd pipeline will be created in github actions , code will be stored in github , application will be monitored by azure monitor . application should have fronend and backend .frontend should be react based and backend python based. from the application we can get the output of employee profile downloadable in ppt and pdf format , list of all employee and their details in spreadsheet also application frontend should have prompt to query about any employee , prepare a series of prompts for github copilot to create application , infrastructure , build and deployment pipeline , monitoring .
# chatgpt : 
Alright — this is a **full GitHub Copilot prompt playbook** for your AI-powered employee profile system, tailored for:

* **Frontend:** React (web app with file upload, search, query)
* **Backend:** Python FastAPI (file processing, Azure integration)
* **Storage:** Azure File Share, Azure SQL
* **Processing:** Azure AI Foundry (data enrichment)
* **Infra:** Terraform
* **Config Management:** Ansible
* **CI/CD:** GitHub Actions
* **Monitoring:** Azure Monitor
* **Exports:** PDF, PPT, Excel
* **Query:** Natural language search from UI

I’ll group prompts in **logical build order** so you can feed them to Copilot step-by-step without getting lost.

---

## **1️⃣ Application Backend (Python FastAPI)**

**Prompt 1 – Project Structure**

> \*"Generate a Python FastAPI project with:
>
> * Endpoints for uploading files (ppt, excel, csv, word) to Azure File Share
> * Endpoint to process uploaded files via Azure AI Foundry API
> * Endpoint to retrieve employee profile by ID or name
> * Endpoint to export employee profile as PPT and PDF
> * Endpoint to list all employees and export to Excel
>   Use environment variables for Azure credentials, SQL connection string, and AI Foundry API key. Structure in `/app` with `routers`, `services`, `models`, and `utils` folders."\*

**Prompt 2 – File Upload & Azure Integration**

> *"Write a FastAPI endpoint `/upload` to accept multiple file types (ppt, excel, csv, word), validate allowed types, and upload to Azure File Share using `azure-storage-file-share` SDK. Store file metadata in Azure SQL."*

**Prompt 3 – File Parsing Service**

> \*"Write Python service functions to extract structured employee data from:
>
> * PPT using `python-pptx`
> * Excel/CSV using `pandas`
> * Word using `python-docx`
>   Extract: employee name, certifications done/pending, mandatory training pending, project, supervisor, expertise. Return JSON."\*

**Prompt 4 – Azure AI Foundry Processing**

> *"Write a service function that sends extracted JSON to Azure AI Foundry for enrichment, receives enhanced profile data, and stores it in Azure SQL with SQLAlchemy models for `employee`, `certifications`, `training`, `projects`."*

**Prompt 5 – Export Functions**

> \*"Write Python functions to:
>
> * Generate a PPT employee profile using `python-pptx`
> * Generate a PDF profile using `reportlab`
> * Export all employee data to Excel using `pandas` DataFrame to Excel
>   Ensure proper formatting and company logo on reports."\*

---

## **2️⃣ Frontend (React)**

**Prompt 6 – React Project Setup**

> \*"Generate a React app with:
>
> * File upload form for PPT, Excel, CSV, Word
> * Table view for all employees with filters (supervisor, project, expertise)
> * Profile detail page with download buttons for PDF and PPT
> * Chat-like query box to ask about an employee and show AI response
>   Use Axios for API calls to FastAPI backend, Tailwind CSS for styling, and React Router for navigation."\*

**Prompt 7 – File Upload Component**

> *"Write a React component with drag-and-drop and manual file selection for uploading multiple files, showing progress bar, and calling `/upload` API endpoint."*

**Prompt 8 – AI Query Component**

> *"Write a React component that lets a user type a natural language question about an employee, sends it to backend `/query` endpoint, and displays AI-generated answer."*

---

## **3️⃣ Infrastructure (Terraform)**

**Prompt 9 – Azure Resource Provisioning**

> \*"Write Terraform to create:
>
> * Resource group
> * Storage account with Azure File Share
> * Azure SQL Database + Server with firewall rules
> * Azure AI Foundry workspace
> * Azure App Service for backend (Python)
> * Azure Static Web Apps for frontend (React)
> * Log Analytics workspace and Azure Monitor setup
>   Use variables for names and locations."\*

**Prompt 10 – Networking & Security**

> *"Extend Terraform to add VNet, subnets for App Service and SQL, private endpoint for SQL, NSGs to restrict access, and integrate all logs with Azure Monitor."*

---

## **4️⃣ Configuration Management (Ansible)**

**Prompt 11 – Backend Deployment**

> *"Write an Ansible playbook to deploy the FastAPI app to Azure App Service, set environment variables for Azure credentials, SQL connection string, AI Foundry API key, and enable managed identity."*

**Prompt 12 – Monitoring Agent Setup**

> *"Write an Ansible playbook to install and configure Azure Monitor Agent on any Azure VM used for background processing."*

---

## **5️⃣ CI/CD (GitHub Actions)**

**Prompt 13 – Backend Workflow**

> \*"Generate a GitHub Actions workflow that:
>
> * Triggers on push to `main`
> * Runs Python tests for backend
> * Runs Terraform plan & apply for infrastructure changes
> * Runs Ansible for config
> * Deploys backend to Azure App Service
> * Runs Alembic migrations for SQLAlchemy models."\*

**Prompt 14 – Frontend Workflow**

> *"Generate a GitHub Actions workflow that builds the React frontend, runs tests, and deploys to Azure Static Web Apps."*

---

## **6️⃣ Monitoring & Alerts**

**Prompt 15 – Terraform Alerts**

> \*"In Terraform, configure Azure Monitor alerts for:
>
> * App Service downtime
> * High CPU (>80%)
> * SQL DTU usage >80%
> * File Share storage usage >90%
>   Send alerts to an action group with email notifications."\*

**Prompt 16 – Dashboard Setup**

> *"In Terraform, create a custom Azure Monitor dashboard with tiles for: backend request count, SQL query performance, storage file usage, and AI Foundry processing latency."*

---

## **7️⃣ End-to-End Testing**

**Prompt 17 – Test Script**

> *"Write a Python script that uploads a sample employee data CSV, triggers AI Foundry processing, stores data in SQL, retrieves profile by name, and downloads the PDF and PPT."*

---

I can also prepare this as a **Copilot Execution Roadmap** where each step’s output feeds directly into the next, so you can literally go from **infra → backend → frontend → CI/CD → monitoring** without rewriting prompts or losing context.

If you want, I can make that **roadmap** so that your build is one smooth Copilot-assisted flow. This way, you avoid mismatches between Terraform, Ansible, backend, and frontend.


# deepseek

# AI Agent Application for Employee Data Management - Copilot Prompts

## Application Overview
Create a comprehensive AI agent application that processes employee data from various file formats, stores it in Azure, and provides queryable employee profiles with export capabilities.

## Infrastructure Creation (Terraform)
```
Create Terraform modules for deploying the following Azure resources:
1. Azure File Share for storing input documents (PPT, Excel, CSV, Word)
2. Azure AI Foundry workspace for data processing
3. Azure SQL Database for processed employee data storage
4. Azure App Service for hosting Python backend API
5. Azure Static Web App for React frontend
6. Azure Monitor for application monitoring
7. Azure Key Vault for secrets management
8. Azure Virtual Network with proper subnets and NSGs
9. Azure Storage Account for application logs and artifacts
10. Azure Container Registry for Docker images

Ensure all resources follow Azure best practices for security, scalability, and cost optimization. Include variables for environment-specific configurations and output important endpoints and connection strings.
```

## Backend Development (Python)
```
Create a Python FastAPI application with the following components:
1. File upload endpoint that accepts PPT, Excel, CSV, and Word files and stores them in Azure File Share
2. Data processing service that uses Azure AI Foundry to:
   - Extract employee data from various file formats
   - Normalize data into a consistent employee profile schema
   - Identify certification/training gaps based on company policies
3. Database service that stores processed employee data in Azure SQL with tables for:
   - Employee profiles (basic info, contact details)
   - Certifications (completed, pending, mandatory)
   - Projects and supervisor relationships
   - Skills and expertise areas
4. Query service that supports:
   - Retrieving individual employee profiles
   - Generating reports (PDF/PPT) of employee profiles
   - Exporting employee lists in spreadsheet format
   - Natural language queries about employees using Azure AI
5. Authentication/authorization using Azure AD
6. Integration with Azure Monitor for logging and telemetry
7. Unit tests and integration tests using pytest
```

## Frontend Development (React)
```
Create a React application with the following features:
1. File upload component supporting drag-and-drop for multiple file types
2. Dashboard showing processing status and statistics
3. Employee profile viewer with tabbed interface for different data sections
4. Report generation panel with options for PDF/PPT export
5. Data table component for viewing all employees with filtering/sorting
6. Natural language query interface with:
   - Prompt input field
   - Query history
   - Results display with option to save as report
7. Responsive design using Material-UI or similar component library
8. Authentication flow using Azure AD
9. Integration with backend API services
10. Error handling and loading states
```

## CI/CD Pipeline (GitHub Actions)
```
Create GitHub Actions workflows for:
1. Infrastructure deployment (Terraform plan/apply) with environment separation (dev/stage/prod)
2. Backend build and test:
   - Python package installation
   - Unit test execution
   - Docker image build and push to Azure Container Registry
   - Deployment to Azure App Service
3. Frontend build and test:
   - Node.js dependencies installation
   - Linting and unit tests
   - Production build
   - Deployment to Azure Static Web Apps
4. Database migrations (if using ORM like SQLAlchemy)
5. Integration tests against deployed environments
6. Approval gates for production deployments
7. Automated rollback on failure detection
```

## Configuration Management (Ansible)
```
Create Ansible playbooks for:
1. Base OS configuration for any VMs in the infrastructure
2. Dependency installation (Python, Node.js, etc.)
3. Application configuration management
4. Secrets injection from Azure Key Vault
5. Monitoring agent setup for Azure Monitor
6. Security hardening and compliance checks
```

## Monitoring Setup (Azure Monitor)
```
Configure Azure Monitor for the application with:
1. Application performance monitoring for both frontend and backend
2. Custom metrics for:
   - File processing times
   - API response times
   - Query processing times
3. Alert rules for:
   - Failed file processing
   - High API error rates
   - Resource utilization thresholds
4. Log analytics queries for troubleshooting
5. Dashboard for operational visibility showing:
   - System health
   - Processing throughput
   - User activity
```

## Sample Employee Profile Schema
```
Create a comprehensive JSON schema for employee profiles that includes:
1. Personal information (name, employee ID, contact details)
2. Certification tracking (completed, in progress, required)
3. Training status (mandatory/completed/pending)
4. Project history (current/past projects, roles)
5. Reporting structure (supervisor, team members)
6. Skills and expertise areas (with proficiency levels)
7. Performance metrics and reviews
8. Career development goals
```

## Report Generation
```
Implement report generation features that:
1. Create visually appealing PPT presentations for individual employee profiles
   - Include charts for certification/training status
   - Project timeline visualization
   - Skills matrix
2. Generate PDF versions of profiles with printable formatting
3. Export employee lists to Excel with:
   - Filterable columns
   - Conditional formatting for certification/training status
   - Summary statistics
4. Support batch exports for multiple employees
```

## Natural Language Query Processing
```
Implement natural language query processing that:
1. Understands questions about:
   - Employee certifications ("Show employees missing AWS certification")
   - Training status ("List all employees with overdue mandatory training")
   - Project experience ("Who has experience with Azure AI projects?")
   - Skills ("Find Python experts with cloud experience")
2. Translates queries to appropriate database queries or aggregations
3. Presents results in tabular format with option to visualize
4. Allows saving frequent queries as templates
5. Learns from user corrections to improve future results
```

## Security Implementation
```
Implement comprehensive security measures including:
1. Azure AD authentication for both frontend and backend
2. Role-based access control (RBAC) for different user types
3. Data encryption at rest and in transit
4. Input validation and sanitization
5. Audit logging for all data access and modifications
6. Regular security scanning in CI/CD pipeline
7. Secret management using Azure Key Vault
```

These prompts should provide GitHub Copilot with detailed guidance to generate the necessary code and configuration files for your comprehensive employee data management application in Azure.

# grok

Here is a series of structured prompts you can use with GitHub Copilot to generate the code, configurations, and scripts for building your AI agent application. I've organized them into categories for clarity: Infrastructure (Terraform), Configuration (Ansible), CI/CD (GitHub Actions), Backend (Python), Frontend (React), Monitoring (Azure Monitor), and Integration/Additional Features. Each prompt is self-contained and detailed to guide Copilot effectively. Copy-paste them one by one into your IDE (e.g., VS Code with Copilot enabled) and let Copilot generate the code based on the prompt.

### 1. Infrastructure (Terraform)
Prompt 1: Set up Azure resources with Terraform
```
Create a Terraform configuration in main.tf to provision the following Azure resources for an AI agent application: 
- An Azure Resource Group named 'employee-ai-rg' in 'eastus' region.
- An Azure Storage Account with File Share for storing input files (PPT, Excel, CSV, Word).
- An Azure SQL Database (or Cosmos DB if better for structured employee data) to store processed employee profiles and lists.
- An Azure App Service for hosting the Python backend.
- An Azure Static Web App for hosting the React frontend.
- Integration with Azure AI services (use Azure AI Document Intelligence or Azure AI Studio for processing documents like PPT, Excel, CSV, Word to extract employee data such as certifications, trainings, projects, supervisors, expertise).
- Enable Azure Monitor for logging and metrics.
Use variables.tf for configurable values like subscription ID, and outputs.tf to output resource IDs. Include provider configuration for AzureRM version ~>3.0.
```

Prompt 2: Terraform modules for modularity
```
Write Terraform modules in separate directories (e.g., modules/storage, modules/db, modules/app-service) to encapsulate the Azure Storage Account with File Share, Azure SQL Database, and Azure App Service from the previous main.tf. Each module should have its own variables, resources, and outputs. In the root main.tf, call these modules with appropriate inputs.
```

### 2. Configuration (Ansible)
Prompt 1: Ansible playbook for backend configuration
```
Create an Ansible playbook (main.yml) to configure a Python backend on Azure App Service. Install dependencies like Flask or FastAPI, Azure SDKs (azure-storage-file-share, azure-ai-documentintelligence for processing files), pandas for data handling, and libraries for generating PPT/PDF (e.g., python-pptx, reportlab). Set up environment variables for Azure connections (storage, DB, AI services). Include roles for installing Python 3.10 and pip packages.
```

Prompt 2: Ansible playbook for frontend and overall setup
```
Write an Ansible playbook to configure the React frontend on Azure Static Web App and integrate with the backend. Install Node.js, npm, and build the React app. Configure Azure connections via environment variables. Include a role to deploy the built frontend artifacts to Azure Static Web App.
```

### 3. CI/CD (GitHub Actions)
Prompt 1: GitHub Actions workflow for build and deploy
```
Create a GitHub Actions workflow YAML file (.github/workflows/deploy.yml) that triggers on push to main branch. Steps: 
- Checkout code.
- Set up Terraform and apply infrastructure changes (use Azure login action).
- Run Ansible playbooks for configuration.
- Build Python backend (pip install requirements) and deploy to Azure App Service using Azure WebApp deploy action.
- Build React frontend (npm install, npm build) and deploy to Azure Static Web App.
- Use secrets for Azure credentials (AZURE_SUBSCRIPTION_ID, AZURE_TENANT_ID, etc.).
```

Prompt 2: GitHub Actions for testing and monitoring integration
```
Extend the GitHub Actions workflow to include testing: Run unit tests for Python backend (using pytest) and React frontend (using Jest). After deployment, integrate with Azure Monitor by enabling diagnostic settings via Azure CLI in the workflow. Add a step to validate Azure AI service endpoints.
```

### 4. Backend (Python)
Prompt 1: Python backend core structure
```
Write a Python backend using FastAPI for an AI agent app. Endpoints: 
- POST /upload: Accept file uploads (PPT, Excel, CSV, Word), store in Azure File Share, process with Azure AI Document Intelligence to extract employee data (certifications done/pending, trainings pending, projects, supervisors, expertise), create employee profile JSON, store in Azure SQL DB.
- GET /employees: Return list of all employees and details as JSON (for spreadsheet export).
- GET /profile/{employee_id}: Generate and return employee profile as PPT or PDF (use query param for format).
- POST /query: Accept natural language query about employees, use Azure AI (e.g., OpenAI integration) to query DB and return results.
Use SQLAlchemy for DB interactions, azure-storage-file-share for storage, azure-ai-documentintelligence for file processing.
```

Prompt 2: Python utilities for file processing and exports
```
Add utility functions to the Python backend: 
- def process_file(file_path): Use Azure AI to extract data from PPT/Excel/CSV/Word into a dict (e.g., {'certifications': [...], 'trainings_pending': [...]}).
- def generate_profile(data, format): Create PPT using python-pptx or PDF using reportlab from employee data.
- def export_spreadsheet(employees): Generate Excel/CSV with all employee details using pandas.
Integrate error handling and logging to Azure Monitor.
```

### 5. Frontend (React)
Prompt 1: React frontend structure
```
Create a React app structure with components: 
- UploadForm: For uploading files (PPT, Excel, CSV, Word) to backend /upload endpoint.
- EmployeeList: Display table of all employees fetched from /employees, with download button for spreadsheet.
- ProfileViewer: For viewing/downloading individual profiles (PPT/PDF) via /profile endpoint.
- QueryPrompt: Text input to query about employees (e.g., "List employees with pending certifications"), send to /query endpoint, display results.
Use React Router for navigation, Axios for API calls, and Material-UI for styling.
```

Prompt 2: React integration with backend and exports
```
Extend the React app: Add download functionality in EmployeeList for spreadsheet (fetch blob from backend and save as XLSX). In ProfileViewer, handle PPT/PDF downloads. Implement authentication if needed (e.g., Azure AD). Add loading states and error messages.
```

### 6. Monitoring (Azure Monitor)
Prompt 1: Set up Azure Monitor in Terraform and code
```
Update the Terraform main.tf to enable Azure Monitor: Create Log Analytics Workspace, Application Insights, and diagnostic settings for App Service, Storage, and DB. Output instrumentation keys. In Python backend, integrate Application Insights SDK for telemetry. In React, add client-side logging if applicable.
```

Prompt 2: Dashboards and alerts for monitoring
```
Write Azure ARM templates or extend Terraform to create Azure Monitor dashboards for the app: Track metrics like API response times, file uploads, DB queries, AI processing errors. Set up alerts for high CPU, failed deployments, or low storage. Include Ansible tasks to configure agent-based monitoring if needed.
```

### 7. Integration/Additional Features
Prompt 1: Overall project setup and README
```
Generate a README.md for the GitHub repo describing the AI agent app: Architecture (Azure-hosted, React frontend, Python backend), how to set up (Terraform apply, Ansible run, GitHub Actions deploy), features (file upload/processing, profile generation, queries, exports), and prerequisites (Azure account, GitHub secrets).
```

Prompt 2: Testing and security prompts
```
Create pytest tests for Python backend endpoints (mock Azure services). Add Jest tests for React components. Implement security: Use Azure AD for auth in backend/frontend, validate file uploads, encrypt DB data.
```

Use these prompts sequentially, refining the generated code as needed. Start with infrastructure to build the foundation, then move to backend/frontend, and finally CI/CD and monitoring. If Copilot generates incomplete code, add more details to the prompt or iterate.

# gemini

Of course. Here is a series of structured prompts for GitHub Copilot to help you build your AI-powered employee profile application from the ground up.

This guide is broken down by the major components of your project: Backend, AI Processing, Frontend, Infrastructure, CI/CD, and Monitoring. For best results with Copilot, provide these prompts in the context of the relevant files (e.g., give Python prompts in a `.py` file).

**A Quick Word of Advice:** GitHub Copilot works best as a **pair programmer**. Use these prompts to generate a strong first draft, but expect to review, refine, and debug the code it produces. Think of it as an accelerator, not a one-click solution. 🚀

***

### 1. Backend Application (Python with FastAPI)

First, let's create the core API structure. FastAPI is a modern, high-performance web framework for building APIs with Python.

* **Prompt 1.1: Project Structure**
    > "Create a directory structure for a Python FastAPI project. Include a root directory with subdirectories for `app`, `tests`, and `scripts`. Inside `app`, create files for `main.py`, `models.py`, `schemas.py`, `crud.py`, and a directory for `api/v1/endpoints` containing `employees.py`."

* **Prompt 1.2: Main API Endpoints** (In `app/api/v1/endpoints/employees.py`)
    > "Create a FastAPI router with the following endpoints:
    > 1.  A POST endpoint at `/upload` that accepts a file upload. It should save the file temporarily and return a file ID.
    > 2.  A GET endpoint at `/employees` that returns a list of all employees from a database.
    > 3.  A GET endpoint at `/employees/{employee_id}` that returns the detailed profile of a single employee.
    > 4.  A POST endpoint at `/employees/{employee_id}/download` that generates and returns a PDF and a PPT of the employee's profile.
    > 5.  A POST endpoint at `/query` that accepts a JSON object with a text `query` and returns an answer about an employee."

* **Prompt 1.3: Dockerization** (In `backend/Dockerfile`)
    > "Create a multi-stage Dockerfile for this FastAPI application. The first stage should install dependencies from `requirements.txt`. The second stage should copy the application code and run it using uvicorn. The container should expose port 8000."

***

### 2. AI Data Processing Core (Azure AI Services)

This is the brain of your application. It will use Azure AI services to parse documents and extract structured information.

* **Prompt 2.1: Document Parsing & Extraction** (In a new file, e.g., `app/processing/parser.py`)
    > "Create a Python function `process_employee_document(file_path)` that takes a local file path as input. This function should:
    > 1.  Use the **Azure AI Document Intelligence** SDK (`azure-ai-documentintelligence`) to analyze the document (PPT, Word, PDF, Excel).
    > 2.  Extract all text content, tables, and key-value pairs from the analysis result.
    > 3.  Take the extracted text and pass it to the **Azure OpenAI** service with a system prompt to structure the information.
    > 4.  The prompt should instruct the OpenAI model to return a JSON object with keys like `employee_name`, `supervisor`, `certifications_completed`, `certifications_pending`, `mandatory_training_pending`, `projects`, and `expertise`.
    > 5.  The function should return this structured JSON object."

* **Prompt 2.2: Database Integration** (In `app/crud.py`)
    > "Using the **SQLAlchemy** ORM, create CRUD (Create, Read, Update, Delete) functions for an `Employee` model. The `create_employee` function should take the JSON data from the AI parser and save it to a PostgreSQL or Azure SQL database."

***

### 3. Frontend Application (React)

Now for the user interface. This will be a single-page application (SPA) built with React.

* **Prompt 3.1: Component Structure** (In your terminal with `npx create-react-app frontend`)
    > "Inside the `src/components` directory of a new React app, create the following functional components: `FileUpload.js`, `EmployeeList.js`, `EmployeeProfile.js`, and `QueryPrompt.js`."

* **Prompt 3.2: File Upload Component** (In `src/components/FileUpload.js`)
    > "Create a React component that allows a user to select or drag-and-drop a file. On file submission, it should use **axios** to send the file to the backend `/upload` endpoint. It should display a loading indicator while uploading and a success or error message upon completion."

* **Prompt 3.3: Employee Profile & Download** (In `src/components/EmployeeProfile.js`)
    > "Create a React component that fetches and displays the data for a single employee given an ID. The component should show all profile details (certifications, projects, etc.) in a clean layout. Include two buttons: 'Download PDF' and 'Download PPT'. Clicking these buttons should call the backend endpoint to trigger the file download."

* **Prompt 3.4: Query Interface** (In `src/components/QueryPrompt.js`)
    > "Create a chat-like interface component with a text input field and a submit button. When the user enters a question and clicks submit, the component should make a POST request to the backend `/query` endpoint with the question. The response from the AI should then be displayed on the screen."
    

***

### 4. Infrastructure as Code (Terraform)

This set of prompts will create the necessary Azure resources to host your entire application.

* **Prompt 4.1: Basic Setup** (In `terraform/main.tf`)
    > "Create a Terraform configuration file for Azure. It should configure the `azurerm` provider and create a resource group named `employee-ai-rg` in the `East US` region."

* **Prompt 4.2: Core Services** (In `terraform/main.tf`)
    > "Extend the Terraform configuration to provision the following Azure resources:
    > 1.  An **Azure App Service Plan** (Linux, B1 SKU).
    > 2.  An **Azure App Service** for the Python backend, configured to use a Docker container from Azure Container Registry.
    > 3.  An **Azure Static Web App** for the React frontend.
    > 4.  An **Azure Storage Account** with an **Azure File Share** to store the uploaded raw documents.
    > 5.  An **Azure Cosmos DB** (or Azure Database for PostgreSQL) for storing the processed employee data.
    > 6.  An **Azure AI Services** multi-service account for Document Intelligence and OpenAI.
    > 7.  An **Azure Container Registry** to store the backend Docker image."

***

### 5. CI/CD Pipelines (GitHub Actions)

Automate the build, testing, and deployment of your application.

* **Prompt 5.1: Backend CI/CD Pipeline** (In `.github/workflows/backend-ci-cd.yml`)
    > "Create a GitHub Actions workflow that triggers on push to the `main` branch for changes in the `backend/` directory. The workflow should have jobs to:
    > 1.  Log in to Azure.
    > 2.  Build the Docker image for the FastAPI application.
    > 3.  Push the Docker image to Azure Container Registry.
    > 4.  Deploy the new image to the Azure App Service."

* **Prompt 5.2: Frontend CI/CD Pipeline** (In `.github/workflows/frontend-ci-cd.yml`)
    > "Create a GitHub Actions workflow that triggers on push to the `main` branch for changes in the `frontend/` directory. Use the official **Azure Static Web Apps** action to build and deploy the React application."

***

### 6. Monitoring & Observability (Azure Monitor)

Finally, ensure you can monitor the health and performance of your application.

* **Prompt 6.1: Backend Instrumentation** (In `app/main.py`)
    > "Integrate the **OpenTelemetry Azure Monitor Exporter** library into the FastAPI application. Configure it to automatically collect and send request traces, logs, and metrics to Azure Application Insights."

* **Prompt 6.2: Terraform Alerting Rules** (In a new file, `terraform/monitoring.tf`)
    > "Using Terraform, create two alert rules in **Azure Monitor**:
    > 1.  An alert that triggers if the backend App Service has an HTTP Server Error (5xx) rate greater than 5% over 15 minutes.
    > 2.  An alert that triggers if the CPU percentage of the App Service Plan is over 85% for 30 minutes.
    > Both alerts should send an email to an administrator."

# le chat

Indra, your vision for an AI agent application that centralizes and processes employee data—using Azure, Terraform, Ansible, GitHub Actions, and a React/Python stack—is comprehensive and well-structured. Below, I’ll break down a **series of prompts** you can use with GitHub Copilot (or your team) to scaffold and build each component of your application. These prompts are designed to be modular, so you can address each part step-by-step.

---

## **1. Infrastructure as Code (Terraform)**
**Goal:** Define Azure resources (storage, AI services, databases, monitoring, etc.) using Terraform.

**Prompt for GitHub Copilot:**
> "Generate a Terraform configuration for an Azure environment that includes:
> - Azure Storage Account with File Share for storing uploaded files (PPT, Excel, CSV, Word)
> - Azure AI Services (Form Recognizer, Cognitive Search, or AI Foundry) for processing documents
> - Azure Database for PostgreSQL/MySQL/Cosmos DB for storing processed employee data
> - Azure App Service for hosting the backend API and frontend
> - Azure Monitor and Application Insights for logging and monitoring
> - Azure Key Vault for secrets management
> - Networking (VNet, Subnets, NSGs) for security
> - Output variables for connection strings and endpoints
> Ensure the configuration is modular, follows best practices, and includes variables for environment-specific settings."

**Follow-up:**
> "Generate a Terraform module for Azure AI Foundry integration, including resource provisioning and IAM roles for the application to access AI services."

---

## **2. Configuration Management (Ansible)**
**Goal:** Automate the configuration of VMs or containers (if used) and application deployment settings.

**Prompt for GitHub Copilot:**
> "Write Ansible playbooks to:
> - Configure Azure VMs/containers with required dependencies (Python, Node.js, Docker, etc.)
> - Deploy the backend (Python/Flask/FastAPI) and frontend (React) applications
> - Set up environment variables and secrets from Azure Key Vault
> - Ensure idempotency and use roles for reusability"

---

## **3. Backend (Python)**
**Goal:** Build a Python backend to process files, interact with Azure AI, and manage data.

**Prompt for GitHub Copilot:**
> "Create a Python FastAPI/Flask application with the following features:
> - Endpoints to upload files (PPT, Excel, CSV, Word) to Azure File Share
> - Integration with Azure AI Foundry/Form Recognizer to extract data from files
> - Logic to parse extracted data and create employee profiles (certifications, trainings, projects, etc.)
> - CRUD APIs for employee data stored in Azure Database
> - Endpoints to generate downloadable employee profiles in PPT and PDF formats using libraries like `python-pptx` and `reportlab`
> - Endpoint to export all employee data as a spreadsheet (Excel/CSV)
> - Integration with Azure Monitor for logging and telemetry
> - Use async/await for file processing to improve performance
> - Include unit and integration tests using `pytest`"

**Follow-up:**
> "Write a Python script to process employee data from Azure AI Foundry, transform it into a structured format, and store it in Azure Database. Handle edge cases like missing data or duplicate entries."

---

## **4. Frontend (React)**
**Goal:** Develop a user interface for uploading files, querying employees, and downloading reports.

**Prompt for GitHub Copilot:**
> "Create a React application with:
> - A file upload interface (drag-and-drop) for PPT, Excel, CSV, Word files
> - A dashboard to display employee profiles and lists
> - A search/prompt interface to query employee data (e.g., 'List all employees with pending mandatory training')
> - Download buttons for employee profiles (PPT/PDF) and spreadsheets
> - State management using Redux or Context API
> - Responsive design using Material-UI or TailwindCSS
> - Integration with the Python backend API"

**Follow-up:**
> "Generate React components for:
> - Employee profile cards with filters (by certification, training status, project, etc.)
> - A data table for the employee list with sorting/pagination
> - A chat-like interface for querying employee data using natural language"

---

## **5. CI/CD Pipeline (GitHub Actions)**
**Goal:** Automate testing, building, and deployment of both frontend and backend.

**Prompt for GitHub Copilot:**
> "Write GitHub Actions workflows for:
> - Backend: Linting, testing, Docker image build, and deployment to Azure App Service
> - Frontend: Linting, testing, build, and deployment to Azure Static Web Apps/App Service
> - Infrastructure: Terraform plan/apply on push to main branch
> - Secrets management using GitHub Secrets and Azure Key Vault
> - Environment-specific deployments (dev, staging, prod)"

**Follow-up:**
> "Add a workflow to run integration tests after deployment and notify Slack/Teams on failure."

---

## **6. Monitoring (Azure Monitor)**
**Goal:** Set up monitoring, alerts, and logging for the application.

**Prompt for GitHub Copilot:**
> "Provide a Terraform configuration to set up Azure Monitor and Application Insights for:
> - Backend API performance and error tracking
> - Frontend user interaction analytics
> - Alerts for failed file processing or database errors
> - Custom dashboards for key metrics (file uploads, profile generations, API latency)"

**Follow-up:**
> "Write a Python script to send custom metrics and logs to Azure Monitor from the backend."

---

## **7. Prompt Engineering for Employee Queries**
**Goal:** Enable natural language queries about employees (e.g., "Who has pending safety training?").

**Prompt for GitHub Copilot:**
> "Generate a list of example prompts for querying employee data, such as:
> - 'List all employees in the Delhi office with pending certifications'
> - 'Show me employees with expertise in Python and Azure'
> - 'Who reports to [Supervisor Name] and what are their pending trainings?'
> - 'Generate a report of all employees due for recertification in the next 3 months'
> - 'What projects is [Employee Name] working on?'
> Include code to integrate these prompts with the backend API and Azure AI for natural language processing."

---

## **8. Documentation**
**Goal:** Document the architecture, APIs, and deployment process.

**Prompt for GitHub Copilot:**
> "Generate a README.md with:
> - Architecture diagram (using Mermaid.js)
> - Setup instructions for local development
> - API documentation (OpenAPI/Swagger)
> - Deployment guide for Azure using Terraform and Ansible
> - Example queries for the employee prompt interface"

---

## **9. Security and Compliance**
**Goal:** Ensure data privacy and access control.

**Prompt for GitHub Copilot:**
> "Provide best practices and code snippets for:
> - Role-based access control (RBAC) in Azure and the application
> - Encryption of data at rest and in transit
> - Audit logging for file uploads and data access
> - Compliance with GDPR or local data protection laws"

---

## **10. Testing**
**Goal:** Ensure reliability and data accuracy.

**Prompt for GitHub Copilot:**
> "Write unit and integration tests for:
> - File processing and data extraction logic
> - API endpoints (upload, query, download)
> - Database operations and data consistency
> - Frontend component rendering and user interactions"

---

### **Next Steps**
- Start with the **Terraform** and **Ansible** prompts to set up your infrastructure.
- Then, scaffold the **backend** and **frontend** using the provided prompts.
- Finally, set up **CI/CD** and **monitoring**.

Would you like me to refine any of these prompts or provide more detailed examples for a specific component? Or do you want to dive deeper into the architecture or tech stack choices?

# claude

# GitHub Copilot Prompts: AI Agent Employee Profile Management System

## 🏗️ **Phase 1: Infrastructure Setup (Terraform)**

### **Prompt 1.1: Azure Resource Group & Storage**
```terraform
# Create Azure infrastructure for AI Agent Employee Profile Management System
# Components: Resource Group, Storage Account with File Share, Key Vault
# Region: East US 2 for AI Foundry availability
# Environment: Production-ready with proper naming conventions

# Create resource group for employee profile management system
resource "azurerm_resource_group" "employee_profile_rg" {
  # Name should follow: rg-employee-profile-prod-eastus2
  # Location: East US 2
  # Tags: project, environment, owner, cost-center
```

### **Prompt 1.2: Azure AI Foundry & Cognitive Services**
```terraform
# Create Azure AI Foundry workspace and Cognitive Services
# Requirements:
# - AI Foundry workspace for document processing
# - Computer Vision API for image/document analysis  
# - Form Recognizer for structured data extraction
# - Text Analytics for content processing
# - All services should be in same resource group and region

resource "azurerm_cognitive_account" "ai_services" {
  # Create multi-service cognitive services account
  # SKU: S0 for production workloads
  # Enable all required AI capabilities
```

### **Prompt 1.3: Database & Web App Infrastructure**
```terraform
# Create database and hosting infrastructure
# Components:
# - Azure Database for PostgreSQL Flexible Server
# - Azure Container Apps for backend API
# - Azure Static Web Apps for React frontend
# - Application Insights for monitoring
# - Log Analytics workspace

resource "azurerm_postgresql_flexible_server" "employee_db" {
  # Configuration:
  # - Version: 14
  # - SKU: GP_Standard_D2s_v3 (2 vCores, 8GB RAM)
  # - Storage: 32GB with auto-grow enabled
  # - Backup retention: 7 days
  # - High availability: Zone redundant
```

### **Prompt 1.4: Networking & Security**
```terraform
# Create networking and security components
# Requirements:
# - Virtual Network with subnets for different tiers
# - Network Security Groups with appropriate rules
# - Private endpoints for secure database access
# - Application Gateway for load balancing
# - Azure Key Vault for secrets management

resource "azurerm_virtual_network" "main_vnet" {
  # CIDR: 10.0.0.0/16
  # Subnets: web (10.0.1.0/24), app (10.0.2.0/24), data (10.0.3.0/24)
  # Enable service endpoints for storage and key vault
```

---

## 🔧 **Phase 2: Configuration Management (Ansible)**

### **Prompt 2.1: Database Setup Playbook**
```yaml
# Create Ansible playbook for PostgreSQL database configuration
# Tasks:
# - Create employee_profiles database
# - Create tables: employees, certifications, trainings, projects, supervisors
# - Set up database users with proper permissions
# - Configure backup policies
# - Install required extensions (uuid-ossp, pgcrypto)

---
- name: Configure PostgreSQL for Employee Profile System
  hosts: localhost
  vars:
    db_name: employee_profiles
    db_user: app_user
  tasks:
    - name: Create database and tables
      # Use postgresql_db and postgresql_user modules
```

### **Prompt 2.2: Azure Container Apps Configuration**
```yaml
# Create Ansible playbook for Container Apps deployment
# Requirements:
# - Configure Container Apps Environment
# - Deploy Python FastAPI backend container
# - Set environment variables for AI services
# - Configure scaling rules (CPU and memory based)
# - Set up health checks and probes
# - Configure custom domains and SSL certificates

- name: Deploy Backend to Azure Container Apps
  hosts: localhost
  vars:
    container_image: "{{ registry_url }}/employee-backend:{{ build_number }}"
  tasks:
    - name: Deploy container app
      # Configure with proper resource limits and environment variables
```

### **Prompt 2.3: Static Web App Configuration**
```yaml
# Configure Azure Static Web Apps for React frontend
# Tasks:
# - Configure build settings for React app
# - Set up custom domains
# - Configure API integration with backend
# - Set up authentication with Azure AD
# - Configure CDN and caching policies

- name: Configure Static Web App
  hosts: localhost
  tasks:
    - name: Configure SWA settings
      # Set up staticwebapp.config.json with routing rules
```

---

## 🐍 **Phase 3: Backend Development (Python FastAPI)**

### **Prompt 3.1: Project Structure & Dependencies**
```python
# Create FastAPI project structure for Employee Profile Management
# Structure:
# app/
#   ├── main.py (FastAPI app initialization)
#   ├── api/ (API routes)
#   ├── models/ (database models)
#   ├── services/ (business logic)
#   ├── utils/ (utility functions)
#   └── config/ (configuration)
# 
# Dependencies needed:
# - fastapi, uvicorn
# - sqlalchemy, psycopg2-binary
# - azure-storage-file-share, azure-ai-formrecognizer
# - python-multipart, openpyxl, python-pptx
# - reportlab, fpdf2

# Create requirements.txt with all necessary dependencies
```

### **Prompt 3.2: Database Models**
```python
# Create SQLAlchemy models for employee profile system
# Models needed:
# - Employee (id, name, email, department, position, hire_date, supervisor_id)
# - Certification (id, name, provider, validity_period, mandatory)
# - EmployeeCertification (employee_id, certification_id, completion_date, expiry_date, status)
# - Training (id, name, description, mandatory, deadline)
# - Project (id, name, description, start_date, end_date)
# - EmployeeProject (employee_id, project_id, role)
# - Expertise (id, employee_id, skill, level, years_experience)

from sqlalchemy import Column, Integer, String, DateTime, Boolean, ForeignKey, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Employee(Base):
    __tablename__ = "employees"
    
    # Define all fields with proper relationships
```

### **Prompt 3.3: File Processing Service**
```python
# Create service class for processing uploaded files
# Requirements:
# - Support PPT, Excel, CSV, Word files
# - Extract employee data using Azure AI services
# - Store files in Azure File Share
# - Parse different file formats and extract structured data
# - Handle errors and validate data quality
# - Return standardized employee data format

from azure.storage.fileshare import ShareFileClient
from azure.ai.formrecognizer import DocumentAnalysisClient
import pandas as pd
from pptx import Presentation
from docx import Document

class FileProcessingService:
    def __init__(self, storage_connection_string: str, ai_endpoint: str, ai_key: str):
        # Initialize Azure clients for file storage and AI processing
        
    async def process_uploaded_file(self, file_content: bytes, filename: str, file_type: str):
        """
        Process uploaded file and extract employee data
        Returns: List of employee records with standardized format
        """
```

### **Prompt 3.4: Employee Profile API Endpoints**
```python
# Create FastAPI endpoints for employee profile management
# Endpoints needed:
# POST /api/files/upload - Upload and process files
# GET /api/employees - List all employees with filters
# GET /api/employees/{id} - Get specific employee profile
# PUT /api/employees/{id} - Update employee profile
# GET /api/employees/{id}/profile/pdf - Generate PDF profile
# GET /api/employees/{id}/profile/ppt - Generate PPT profile
# POST /api/query - Natural language query about employees
# GET /api/export/spreadsheet - Export all employee data

from fastapi import APIRouter, UploadFile, File, Depends, Query
from typing import List, Optional
from pydantic import BaseModel

router = APIRouter(prefix="/api", tags=["employees"])

class EmployeeResponse(BaseModel):
    # Define response schema for employee data
    
@router.post("/files/upload")
async def upload_and_process_files(files: List[UploadFile] = File(...)):
    """
    Upload multiple files and process employee data
    """
```

### **Prompt 3.5: AI Query Service**
```python
# Create service for natural language queries about employees
# Requirements:
# - Use Azure OpenAI for query understanding
# - Convert natural language to database queries
# - Support queries like "Who has expired certifications?", "List Python experts"
# - Return structured responses with employee data
# - Handle complex queries with multiple filters

from azure.ai.textanalytics import TextAnalyticsClient
from sqlalchemy.orm import Session
import json

class EmployeeQueryService:
    def __init__(self, ai_endpoint: str, ai_key: str):
        # Initialize AI services for query processing
        
    async def process_natural_language_query(self, query: str, db: Session):
        """
        Process natural language query and return relevant employee data
        Example queries:
        - "Show me all employees with expired certifications"
        - "Who are the Python experts in the engineering team?"
        - "List employees with pending mandatory training"
        """
```

### **Prompt 3.6: Report Generation Service**
```python
# Create service for generating PDF and PPT employee profiles
# Requirements:
# - Generate professional PDF profiles with all employee data
# - Create PowerPoint presentations with employee information
# - Include charts and visualizations for skills, certifications
# - Support templates and customization
# - Handle batch generation for multiple employees

from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table
from pptx import Presentation
from pptx.util import Inches
import matplotlib.pyplot as plt

class ReportGenerationService:
    def __init__(self):
        # Initialize report generation templates
        
    async def generate_employee_pdf_profile(self, employee_data: dict) -> bytes:
        """
        Generate comprehensive PDF profile for employee
        Include: basic info, certifications, trainings, projects, expertise
        """
        
    async def generate_employee_ppt_profile(self, employee_data: dict) -> bytes:
        """
        Generate PowerPoint presentation for employee profile
        Include slides for: overview, certifications, skills, projects
        """
```

---

## ⚛️ **Phase 4: Frontend Development (React)**

### **Prompt 4.1: Project Setup & Structure**
```bash
# Create React TypeScript project with modern tooling
# Requirements:
# - Create React app with TypeScript
# - Set up Material-UI for components
# - Configure Axios for API calls
# - Set up React Router for navigation
# - Add React Query for data fetching
# - Configure environment variables for different environments

npx create-react-app employee-portal --template typescript
cd employee-portal

# Install additional dependencies
npm install @mui/material @emotion/react @emotion/styled @mui/icons-material
npm install axios react-router-dom @tanstack/react-query
npm install react-dropzone recharts

# Create project structure:
# src/
#   ├── components/ (reusable components)
#   ├── pages/ (page components)
#   ├── services/ (API services)
#   ├── hooks/ (custom hooks)
#   ├── types/ (TypeScript types)
#   └── utils/ (utility functions)
```

### **Prompt 4.2: File Upload Component**
```typescript
// Create drag-and-drop file upload component
// Requirements:
// - Support multiple file types (PPT, Excel, CSV, Word)
// - Drag and drop interface with visual feedback
// - Progress indicators for upload and processing
// - File validation and error handling
// - Preview of files before upload
// - Batch processing status display

import React, { useState, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';
import { Box, Typography, LinearProgress, Alert, Card } from '@mui/material';
import { CloudUpload } from '@mui/icons-material';

interface FileUploadProps {
  onFilesProcessed: (data: any) => void;
  onError: (error: string) => void;
}

const FileUploadComponent: React.FC<FileUploadProps> = ({ onFilesProcessed, onError }) => {
  // Implement drag-and-drop file upload with progress tracking
```

### **Prompt 4.3: Employee List & Search Component**
```typescript
// Create employee list component with search and filters
// Features:
// - Data table with sorting and pagination
// - Search by name, department, skills
// - Filters for certification status, training completion
// - Export to Excel functionality
// - Bulk actions (select multiple employees)
// - Real-time search with debouncing

import React, { useState, useEffect, useMemo } from 'react';
import { DataGrid, GridColDef, GridToolbar } from '@mui/x-data-grid';
import { TextField, FormControl, Select, MenuItem, Button, Chip } from '@mui/material';
import { useQuery } from '@tanstack/react-query';

interface Employee {
  id: number;
  name: string;
  department: string;
  position: string;
  certifications: Certification[];
  trainings: Training[];
}

const EmployeeListComponent: React.FC = () => {
  // Implement searchable, filterable employee list
```

### **Prompt 4.4: Employee Profile Detail View**
```typescript
// Create detailed employee profile view
// Sections:
// - Basic information (editable)
// - Certifications timeline with status indicators
// - Training progress and completion status
// - Skills and expertise radar chart
// - Project history and current assignments
// - Download profile as PDF/PPT buttons

import React, { useState } from 'react';
import { Card, CardContent, Typography, Tabs, Tab, Box, Button, Chip, Avatar } from '@mui/material';
import { Timeline, TimelineItem, TimelineContent, TimelineSeparator, TimelineDot } from '@mui/lab';
import { Radar, RadarChart, PolarGrid, PolarAngleAxis, ResponsiveContainer } from 'recharts';
import { Download, Edit } from '@mui/icons-material';

interface EmployeeProfileProps {
  employeeId: number;
}

const EmployeeProfileDetail: React.FC<EmployeeProfileProps> = ({ employeeId }) => {
  // Implement comprehensive employee profile view
```

### **Prompt 4.5: AI Query Interface**
```typescript
// Create natural language query interface
// Features:
// - Chat-like interface for asking questions
// - Query suggestions and examples
// - Results displayed in appropriate format (table, cards, charts)
// - Query history and saved queries
// - Export query results
// - Loading states and error handling

import React, { useState } from 'react';
import { TextField, Paper, Typography, List, ListItem, Chip, IconButton } from '@mui/material';
import { Send, History, Bookmark } from '@mui/icons-material';
import { useQuery } from '@tanstack/react-query';

interface QueryResult {
  query: string;
  results: any[];
  timestamp: Date;
}

const AIQueryInterface: React.FC = () => {
  const [query, setQuery] = useState('');
  const [queryHistory, setQueryHistory] = useState<QueryResult[]>([]);
  
  // Example queries to show users
  const exampleQueries = [
    "Who has expired certifications?",
    "List all Python developers in engineering",
    "Show employees with pending mandatory training",
    "Who are the project managers with PMP certification?"
  ];
  
  // Implement natural language query interface
```

### **Prompt 4.6: Dashboard & Analytics**
```typescript
// Create dashboard with employee analytics
// Components:
// - Total employee count by department
// - Certification compliance rates
// - Training completion statistics
// - Skills distribution charts
// - Upcoming certification expirations
// - Department-wise expertise mapping

import React from 'react';
import { Grid, Card, CardContent, Typography } from '@mui/material';
import { PieChart, Pie, Cell, BarChart, Bar, XAxis, YAxis, ResponsiveContainer } from 'recharts';
import { useQuery } from '@tanstack/react-query';

const DashboardComponent: React.FC = () => {
  // Implement analytics dashboard with charts
```

---

## 🚀 **Phase 5: CI/CD Pipeline (GitHub Actions)**

### **Prompt 5.1: Backend Deployment Pipeline**
```yaml
# Create GitHub Actions workflow for backend deployment
# Requirements:
# - Trigger on push to main branch and pull requests
# - Run tests and code quality checks
# - Build Docker image for FastAPI app
# - Push to Azure Container Registry
# - Deploy to Azure Container Apps
# - Run integration tests after deployment

name: Backend CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
    paths: [ 'backend/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'backend/**' ]

env:
  REGISTRY: employeeportal.azurecr.io
  IMAGE_NAME: employee-backend

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        # Configure Python environment and run tests
```

### **Prompt 5.2: Frontend Deployment Pipeline**
```yaml
# Create GitHub Actions workflow for React frontend
# Requirements:
# - Build React TypeScript application
# - Run unit tests and linting
# - Build production bundle
# - Deploy to Azure Static Web Apps
# - Run E2E tests with Playwright
# - Update CDN cache

name: Frontend CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
    paths: [ 'frontend/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'frontend/**' ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      # Build and test React application
```

### **Prompt 5.3: Infrastructure Pipeline**
```yaml
# Create GitHub Actions workflow for infrastructure deployment
# Requirements:
# - Use Terraform for infrastructure provisioning
# - Validate Terraform configuration
# - Plan infrastructure changes
# - Apply changes with approval for production
# - Run Ansible playbooks for configuration
# - Store Terraform state in Azure Storage

name: Infrastructure CI/CD Pipeline

on:
  push:
    branches: [ main ]
    paths: [ 'infrastructure/**' ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'dev'
        type: choice
        options:
        - dev
        - staging
        - prod

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0
      # Terraform plan and apply workflow
```

### **Prompt 5.4: End-to-End Testing Pipeline**
```yaml
# Create comprehensive testing pipeline
# Requirements:
# - Unit tests for both frontend and backend
# - Integration tests for API endpoints
# - E2E tests for critical user journeys
# - Performance testing with load tests
# - Security scanning with CodeQL
# - Generate test reports and coverage

name: Comprehensive Testing Pipeline

on:
  schedule:
    - cron: '0 2 * * *'  # Run nightly
  workflow_dispatch:

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    # Run unit tests for all components
    
  integration-tests:
    runs-on: ubuntu-latest
    # Run API integration tests
    
  e2e-tests:
    runs-on: ubuntu-latest
    # Run Playwright E2E tests
    
  performance-tests:
    runs-on: ubuntu-latest
    # Run load tests with k6
    
  security-scan:
    runs-on: ubuntu-latest
    # Run security scans
```

---

## 📊 **Phase 6: Monitoring & Observability (Azure Monitor)**

### **Prompt 6.1: Application Insights Setup**
```python
# Configure Application Insights for Python backend
# Requirements:
# - Track API performance and errors
# - Monitor database query performance
# - Log AI service usage and costs
# - Track user actions and file processing
# - Set up custom metrics and alerts
# - Configure distributed tracing

from azure.monitor.opentelemetry import configure_azure_monitor
from opentelemetry import trace, metrics
import logging

# Configure Application Insights with custom telemetry
def setup_application_insights(connection_string: str):
    # Configure OpenTelemetry with Azure Monitor
    configure_azure_monitor(connection_string=connection_string)
    
    # Custom metrics for business logic
    meter = metrics.get_meter(__name__)
    file_processed_counter = meter.create_counter(
        "files_processed_total",
        description="Total number of files processed"
    )
    
    # Custom logging for employee operations
```

### **Prompt 6.2: React Application Monitoring**
```typescript
// Configure Application Insights for React frontend
// Requirements:
// - Track page views and user interactions
// - Monitor API call performance from frontend
// - Capture JavaScript errors and exceptions
// - Track custom events (file uploads, searches, downloads)
// - Monitor user session data
// - Set up performance metrics

import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactPlugin } from '@microsoft/applicationinsights-react-js';

const reactPlugin = new ReactPlugin();

const appInsights = new ApplicationInsights({
  config: {
    connectionString: process.env.REACT_APP_APPLICATIONINSIGHTS_CONNECTION_STRING,
    extensions: [reactPlugin],
    extensionConfig: {
      [reactPlugin.identifier]: {}
    },
    enableAutoRouteTracking: true,
    enableCorsCorrelation: true
  }
});

// Initialize and configure monitoring
```

### **Prompt 6.3: Azure Monitor Alerts & Dashboards**
```json
// Create Azure Monitor alert rules and dashboards
// Requirements:
// - Alert on API response time > 5 seconds
// - Alert on error rate > 5%
// - Alert on failed file processing
// - Alert on database connection issues
// - Alert on Azure AI service quota limits
// - Create comprehensive monitoring dashboard

{
  "alert_rules": [
    {
      "name": "High API Response Time",
      "condition": "avg_response_time > 5000ms",
      "action": "email + teams notification"
    },
    {
      "name": "High Error Rate", 
      "condition": "error_rate > 5%",
      "action": "page on-call engineer"
    }
  ],
  "dashboard_components": [
    "API performance metrics",
    "File processing statistics", 
    "User activity heatmap",
    "Database performance",
    "AI service usage and costs"
  ]
}
```

### **Prompt 6.4: Log Analytics Queries**
```kql
// Create KQL queries for log analysis and monitoring
// Requirements:
// - Query for failed file processing operations
// - Analyze user search patterns and popular queries
// - Monitor AI service API usage and costs
// - Track employee data access patterns
// - Generate performance reports
// - Create custom workbooks for business insights

// Query 1: Failed file processing operations
requests
| where name contains "upload"
| where success == false
| extend fileName = tostring(customDimensions.fileName)
| summarize FailureCount = count() by fileName, resultCode
| order by FailureCount desc

// Query 2: Most popular employee searches
customEvents
| where name == "employee_search"
| extend searchTerm = tostring(customDimensions.searchTerm)
| summarize SearchCount = count() by searchTerm
| top 20 by SearchCount desc

// Query 3: AI service usage and performance
dependencies
| where type == "HTTP"
| where target contains "cognitiveservices"
| summarize 
    RequestCount = count(),
    AvgDuration = avg(duration),
    SuccessRate = countif(success == true) * 100.0 / count()
by name
| order by RequestCount desc
```

---

## 🗂️ **Phase 7: Project Organization & Documentation**

### **Prompt 7.1: Repository Structure**
```bash
# Create comprehensive repository structure
# Requirements:
# - Organized folder structure for monorepo
# - Clear separation of concerns
# - Documentation for each component
# - Environment configuration templates
# - Development setup instructions

employee-profile-ai-system/
├── .github/
│   └── workflows/          # CI/CD pipelines
├── infrastructure/
│   ├── terraform/          # Infrastructure as code
│   └── ansible/           # Configuration management
├── backend/
│   ├── app/               # FastAPI application
│   ├── tests/             # Unit and integration tests
│   ├── Dockerfile         # Container configuration
│   └── requirements.txt   # Python dependencies
├── frontend/
│   ├── src/               # React TypeScript source
│   ├── public/            # Static assets
│   └── package.json       # Node.js dependencies
├── docs/                  # Project documentation
├── docker-compose.yml     # Local development setup
├── README.md              # Project overview and setup
└── .env.template         # Environment variables template
```

### **Prompt 7.2: Comprehensive README**
```markdown
# Create detailed README.md for the project
# Sections needed:
# - Project overview and objectives
# - Architecture diagram and technology stack
# - Prerequisites and system requirements
# - Local development setup instructions
# - Deployment guide for different environments
# - API documentation links
# - Contributing guidelines
# - Troubleshooting common issues

# AI Agent Employee Profile Management System

## Overview
This system processes employee data from various file formats (PPT, Excel, CSV, Word) using Azure AI services to create comprehensive employee profiles with certifications, trainings, projects, and expertise tracking.

## Architecture
[Include system architecture diagram]

## Technology Stack
- **Frontend**: React with TypeScript, Material-UI
- **Backend**: Python FastAPI, SQLAlchemy, PostgreSQL
- **AI Processing**: Azure AI Foundry, Form Recognizer, OpenAI
- **Infrastructure**: Azure (Container Apps, Static Web Apps, PostgreSQL)
- **DevOps**: Terraform, Ansible, GitHub Actions, Azure Monitor

## Quick Start
[Detailed setup instructions]
```

---

## 💡 **Bonus: Advanced Features Prompts**

### **Prompt 8.1: Real-time Notifications**
```python
# Implement real-time notifications using SignalR
# Features:
# - Notify users when file processing is complete
# - Alert managers about expiring certifications
# - Broadcast system maintenance messages
# - Send notifications for new training assignments

from azure.core.messaging import CloudEvent
from azure.eventgrid import EventGridPublisherClient
import asyncio

class NotificationService:
    def __init__(self, eventgrid_endpoint: str, eventgrid_key: str):
        # Set up EventGrid for real-time notifications
        
    async def notify_file_processing_complete(self, user_id: str, file_name: str, employee_count: int):
        """Send notification when file processing is complete"""
        
    async def alert_certification_expiry(self, manager_id: str, employees: list):
        """Alert managers about employees with expiring certifications"""
```

### **Prompt 8.2: Advanced Analytics & ML**
```python
# Implement machine learning for employee insights
# Features:
# - Predict skill gaps in teams
# - Recommend training based on career paths
# - Identify high-performing employee patterns
# - Suggest optimal team compositions for projects

from sklearn.ensemble import RandomForestClassifier
from sklearn.cluster import KMeans
import pandas as pd

class EmployeeAnalyticsML:
    def __init__(self):
        # Initialize ML models for employee analytics
        
    def predict_skill_gaps(self, department: str, project_requirements: list):
        """Predict skill gaps for upcoming projects"""
        
    def recommend_training_path(self, employee_id: int, career_goal: str):
        """Recommend personalized training path for career development"""
        
    def analyze_team_performance_patterns(self, project_data: pd.DataFrame):
        """Identify patterns in high-performing teams"""
```

This comprehensive prompt series will guide GitHub Copilot through creating your entire AI agent application system step by step, from infrastructure to deployment and monitoring!
