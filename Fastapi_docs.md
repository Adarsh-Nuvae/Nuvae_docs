# FastAPI AI Services Documentation

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture](#architecture)
3. [Core Components](#core-components)
4. [API Endpoints](#api-endpoints)
5. [Agent System](#agent-system)
6. [Chat System](#chat-system)
7. [Data Integration](#data-integration)
8. [Document Processing](#document-processing)
9. [Claims & Appeals](#claims--appeals)
10. [Contract Management](#contract-management)
11. [Tools & Utilities](#tools--utilities)
12. [Configuration](#configuration)
13. [Testing](#testing)
14. [Deployment](#deployment)

---

## System Overview

The Nuvae AI Services is a comprehensive healthcare revenue cycle management platform built with FastAPI. It integrates AI-powered agents, FHIR healthcare data standards, document processing, claims automation, and appeals generation to streamline healthcare administrative processes.

### Key Features
- **Multi-Agent AI System** for specialized healthcare workflows
- **FHIR-compliant** healthcare data integration
- **Automated appeals processing** with ML predictions
- **Real-time chat system** with healthcare context
- **Document processing** with OCR and semantic search
- **Contract analysis** and rate validation
- **Event-driven architecture** using Inngest
- **Voice call automation** for pre-authorizations

---

## Architecture

```
ai-services/
├── main.py                 # FastAPI application entry point
├── config.py               # Application configuration
├── models/                 # Database models
├── lib/                    # Core libraries (DB, storage, instrumentation)
├── auth/                   # Authentication system
├── agents/                 # AI agent system
├── chat/                   # Chat system (v1 and v2)
├── data_sync/              # FHIR data synchronization
├── appeals/                # Appeals processing
├── claims/                 # Claims workflow and ML
├── contract/               # Contract management
├── payer/                  # Payer document processing
├── pre_auth/               # Pre-authorization workflows
├── stats/                  # Analytics and chart generation
├── tools/                  # Various utility tools
├── ub04/                   # UB04 form analysis
├── utils/                  # Utility functions
├── stores/                 # Vector stores
└── tests/                  # Test suite
```

---

## Core Components

### 1. Main Application (`main.py`)

**Purpose**: Central FastAPI application serving as the AI-powered healthcare backend.

**Key Endpoints**:
- `/health` - System health check with dependency monitoring
- `/api/chat` - Create new chat sessions
- `/messages/{chat_id}` - Get/post messages to chat
- `/v2/messages/{chat_id}` - Enhanced chat service
- `/payer-plan-knowledge` - Process payer plan documents
- `/generate-widget` - AI-powered chart generation
- `/sync-fhir-resource` - FHIR data synchronization
- `/analyze-ub04` - UB04 form analysis
- `/phi/deidentify/` - PHI de-identification

**Dependencies**:
```python
# Key imports and integrations
from appeals.restapi.routes import router as appeals_router
from chat.routes import router as chat_router
from auth import JWTBearer
from inngest_client import get_all_inngest_functions
```

**Integration Points**:
- **Appeals System**: `/appeals` routes for automated appeal generation
- **Chat System**: `/chat` routes for AI conversations
- **FHIR Integration**: Webhook endpoint for healthcare data sync
- **Inngest Workflows**: Event-driven function orchestration
- **Authentication**: JWT-based security for all endpoints

---

### 2. Configuration System (`config.py`)

**Purpose**: Centralized configuration management for the entire application.

**Key Features**:
- Environment variable validation
- Azure OpenAI client setup
- Logging configuration
- Application settings

**Configuration Structure**:
```python
# Required environment variables
REQUIRED_ENV_VARS = [
    "AZURE_OPENAI_API_KEY", 
    "AZURE_OPENAI_ENDPOINT", 
    "AZURE_OPENAI_API_VERSION",
    "AZURE_OPENAI_DEPLOYMENT_NAME"
]

# Application settings
APP_NAME = "Nuvae AI"
ENVIRONMENT = os.getenv("ENVIRONMENT", "development")
```

**Usage**: All modules import configuration through this centralized system.

---

### 3. Database Layer (`lib/db.py`)

**Purpose**: Async PostgreSQL database connection management with connection pooling.

**Key Features**:
- Async SQLAlchemy engine
- Connection pooling (20 base, 30 overflow)
- Session management
- Health monitoring

**Connection Configuration**:
```python
engine = create_async_engine(
    url=DATABASE_URL,
    echo=True,
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True,
    pool_recycle=3600
)
```

**Dependency Injection**:
- `get_db()` - Standard database session
- `get_db_async()` - Enhanced async session
- Used across all data access layers

---

### 4. Models (`models/models.py`)

**Purpose**: SQLAlchemy database models defining the entire data schema.

**Key Model Categories**:

**User Management**:
```python
class User(Base):
    __tablename__ = "User"
    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True)
    # ... relationships to chats, sessions, roles
```

**Chat System**:
```python
class Chat(Base):
    __tablename__ = "Chat" 
    # Links to users and messages
    
class Message(Base):
    __tablename__ = "Message"
    role = Column(Enum(MessageRole))  # USER, ASSISTANT, SYSTEM
```

**Healthcare Data**:
- `Patient`, `Encounter`, `Observation`, `Procedure`, `Condition`
- `Claim`, `Denial`, `Appeal`
- `PayerPlan`, `InsurancePlan`

**Document Management**:
- `Document`, `DocumentVersion`, `FileResource`
- `KnowledgeReference` for embeddings

---

## API Endpoints

### Core Endpoints (`main.py`)

#### 1. Health Check Endpoint
```http
GET /health
```
**Purpose**: Comprehensive system health monitoring
**Returns**: Status of database, Azure Storage, OpenAI API
**Integration**: Monitors all critical dependencies

#### 2. Chat Management
```http
POST /api/chat
GET /messages/{chat_id}  
POST /messages/{chat_id}
POST /v2/messages/{chat_id}
```
**Purpose**: AI-powered chat system for healthcare queries
**Features**: 
- Streaming responses
- Context-aware conversations  
- Multi-agent routing
- Memory management

#### 3. Document Processing
```http
POST /payer-plan-knowledge
```
**Purpose**: Process payer plan documents (PDF/Excel) for embeddings
**Features**:
- Document text extraction
- Vector embedding generation
- Summary generation using AI
- Metadata storage

#### 4. Analytics & Visualization
```http
POST /generate-widget
```
**Purpose**: AI-powered chart generation from natural language
**Process**:
1. Generate SQL from description
2. Execute query against database
3. Create chart configuration
4. Return visualization data

#### 5. FHIR Integration
```http
POST /sync-fhir-resource
```
**Purpose**: Healthcare data synchronization webhook
**Security**: Signature validation
**Integration**: Routes to FHIR sync handlers

### Appeals System Endpoints (`appeals/restapi/routes.py`)

#### Base Routes: `/v1/denials`

#### 1. Debug Denial
```http
GET /{denial_id}/debug
```
**Purpose**: Debug denial information with EDI data
**Returns**: Denial metadata, patient info, EDI content

#### 2. Generate Appeal
```http
POST /{denial_id}/appeals
```
**Purpose**: Generate automated appeal letters
**Process**:
1. Retrieve denial information
2. Determine appeal type (EOB vs EDI_835)
3. Generate appeal letter using AI
4. Store and return appeal data

**Integration Points**:
- `AppealLetterService` - Core appeal generation logic
- `PDRFormService` - PDR form processing
- `OrganizationDao` - Organization data access
- `AzureBlobStorage` - Document storage

#### 3. Export Appeals
```http
POST /{denial_id}/appeals/{appeal_id}/export
```
**Purpose**: Export appeal letters and PDR forms
**Options**:
- Appeal letter only (Word document)
- PDR form only (PDF)
- Both (ZIP file)
**Features**: Streaming download, export tracking

---

## Agent System

### Agent Architecture (`agents/`)

The AI agent system is the core of the platform, providing specialized AI assistants for different healthcare workflows.

#### 1. Agent Factory (`agents/factory.py`)

**Purpose**: Creates and manages teams of specialized AI agents using AutoGen framework.

**Key Function**: `create_team_selector_group_chat()`
```python
async def create_team_selector_group_chat() -> Tuple[SelectorGroupChat, AzureOpenAIChatCompletionClient, Optional[ListMemory]]:
    # Creates team of healthcare agents
    participants = [
        patient_processor,
        insurance_analyst, 
        clinical_specialist,
        claim_processor,
        encounter_processor,
        medical_code_helper_agent,
        finance_assist_agent,
    ]
```

**Agent Selection Logic**: AI-powered selector chooses appropriate agent based on query context.

#### 2. Individual Agents

**Patient Processor** (`agents/patient_processor.py`):
- **Purpose**: Patient data retrieval and formatting
- **Capabilities**: Demographics, medical history, encounter summaries
- **Integration**: FHIR patient resources, encounter data

**Insurance Analyst** (`agents/insurance_analyst.py`):
- **Purpose**: Insurance coverage and reimbursement analysis  
- **Capabilities**: Coverage verification, rate analysis, authorization checks
- **Integration**: Payer plan data, contract terms

**Clinical Specialist** (`agents/clinical_specialist.py`):
- **Purpose**: Medical condition and diagnosis assistance
- **Capabilities**: ICD-10 coding, clinical decision support
- **Integration**: Medical knowledge bases, condition data

**Claims Processor** (`agents/claim_processor.py`):
- **Purpose**: Claims analysis and processing
- **Capabilities**: Claim validation, denial analysis, appeal recommendations
- **Integration**: EDI processing, ML prediction models

**Finance Assistant** (`agents/finance_assistant.py`):
- **Purpose**: Financial calculations and analysis
- **Capabilities**: Netdown calculations, payment analysis, financial reporting
- **Integration**: Claims data, contract rates, payment information

#### 3. Memory Management (`agents/nuvae_assistant_memory_manager.py`)

**Purpose**: Sophisticated memory system for maintaining conversation context across interactions.

**Key Features**:
- **Scoped Memory Access**: Different memory levels based on user permissions
- **Context Persistence**: Maintains conversation history and context
- **Memory Filtering**: Intelligent memory retrieval based on relevance
- **Multi-Session Support**: Handles multiple concurrent conversations

**Memory Structure**:
```python
class NuvaeAssistantMemoryManager:
    def __init__(self):
        self.base_memory = ListMemory()
        self.scoped_memories = {}
    
    def get_scoped_memory(self, scope: str) -> ListMemory:
        # Returns memory scoped to specific context
```

#### 4. Agent Constants (`agents/constants.py`)

**Purpose**: Centralized agent name definitions and routing configuration.

**Agent Names**:
```python
PATIENT_PROCESSOR = "patient_processor"
INSURANCE_ANALYST = "insurance_analyst"
CLINICAL_SPECIALIST = "clinical_specialist"
CLAIM_PROCESSOR = "claim_processor"
ENCOUNTER_PROCESSOR = "encounter_processor"
MEDICAL_CODE_HELPER_AGENT = "medical_code_helper_agent"
FINANCE_ASSISTANT = "finance_assistant"
```

---

## Chat System

### V1 Chat System (`chat/`)

#### 1. Chat System Core (`chat/chat_system.py`)

**Purpose**: OpenAI-integrated chat system with streaming responses and function calling.

**Key Function**: `send_message_generator()`
```python
async def send_message_generator(chat_id, message_text, contexts, db):
    # 1. Load chat context and history
    # 2. Generate user message with context
    # 3. Get OpenAI streaming response
    # 4. Process function calls
    # 5. Save messages to database
```

**Function Calling Integration**:
- `LookUpMissingInfo` - Medical information lookup
- `IdentifyFileType` - Document type identification
- File processing for UB04 forms and denial letters

#### 2. Context Management (`chat/system_context.py`)

**Purpose**: Manages contextual information for chat conversations.

**Context Types**:
- Patient context (demographics, medical history)
- Insurance context (plans, coverage, authorizations)
- Document context (uploaded files, references)
- Clinical context (diagnoses, procedures, medications)

#### 3. Chat Routes (`chat/routes.py`)

**Purpose**: FastAPI routes for chat functionality.

**Endpoints**:
```python
@router.post("/chat")
async def create_chat_session()

@router.get("/{chat_id}/messages")
async def get_chat_messages()

@router.post("/{chat_id}/message")
async def send_message()
```

### V2 Chat System (`chat/v2/`)

#### 1. Enhanced Chat Service (`chat/v2/chat_service.py`)

**Purpose**: Improved chat service with better performance and modular context providers.

**Improvements over V1**:
- Better error handling
- Modular context providers
- Enhanced performance
- Improved memory management

#### 2. Context Providers (`chat/v2/context_providers/`)

**Modular Context System**:
- `patient_context_provider.py` - Patient data context
- `claim_context_provider.py` - Claims information context
- `allergy_context_provider.py` - Allergy information context
- `condition_context_provider.py` - Medical condition context
- `encounter_context_provider.py` - Encounter details context
- `observation_context_provider.py` - Observation data context
- `procedure_context_provider.py` - Procedure information context

**Context Provider Interface**:
```python
class ContextProvider:
    async def get_context(self, context_request) -> ContextData:
        # Returns structured context data
```

---

## Data Integration

### FHIR Synchronization (`data_sync/fhir/`)

#### 1. FHIR Sync Handler (`data_sync/fhir/fhir_sync_handler.py`)

**Purpose**: Main entry point for FHIR resource synchronization.

```python
class FHIRSyncHandler:
    async def handle(self, resource: dict, session: AsyncSession):
        sync_source = SyncSourceType.from_config()
        if sync_source == SyncSourceType.HL7:
            self.sync_service = HL7FHIRSyncService()
        else:
            self.sync_service = FethrFHIRSyncService()
        return await self.sync_service.sync_fhir(resource, session=session)
```

**Integration**: Routes to appropriate sync service based on configuration.

#### 2. Resource Services (`data_sync/fhir/resource_services/`)

**Patient Services**:
- `patients/patient_service.py` - Patient resource processing
- `patients/patient_fhir_resource_processor.py` - FHIR patient data transformation
- `patients/hl7_patient_id_generator.py` - ID generation for HL7 patients

**Encounter Services**:
- `encounters/encounter_service.py` - Encounter processing
- `encounters/encounter_fhir_resource_processor.py` - FHIR encounter transformation

**Clinical Services**:
- `observations/observation_service.py` - Observation data processing
- `procedures/procedure_service.py` - Procedure data processing
- `conditions/condition_service.py` - Condition data processing
- `allergy_intolerances/allergy_intolerance_service.py` - Allergy processing

**Coverage Services**:
- `coverages/coverage_service.py` - Insurance coverage processing

#### 3. Bundle Processing (`data_sync/fhir/resource_services/bundle/`)

**Purpose**: Handles FHIR bundle resources containing multiple related resources.

```python
class BundleProcessor:
    async def process_bundle(self, bundle_resource: dict, session: AsyncSession):
        # Processes all entries in FHIR bundle
        # Maintains referential integrity
        # Handles transactions
```

#### 4. Data Access Objects (`data/`)

**Patient Data** (`data/patients.py`):
```python
async def get_patient_details(connection, patient_id):
    # Retrieves comprehensive patient information
    
async def get_patients_with_most_observations(connection, limit=10):
    # Analytics query for patient activity
```

**Clinical Data**:
- `data/encounters.py` - Encounter data access
- `data/observations.py` - Observation data access  
- `data/procedures.py` - Procedure data access
- `data/conditions.py` - Condition data access
- `data/medications.py` - Medication data access

**Claims Data** (`data/claims.py`):
```python
def get_claims(connection, encounter_ids):
    # Retrieves claims for specific encounters
```

---

## Document Processing

### Payer Document Processing (`payer/`)

#### 1. Document Embedding Manager (`payer/document_embedding_manager.py`)

**Purpose**: Processes payer plan documents (PDF/Excel) for semantic search.

**Key Functions**:

**PDF Processing**:
```python
async def add_payer_plan_document_embeddings(payload: PayerPlanProcessingRequest, file: str, db: AsyncSession):
    # 1. Extract text from PDF pages
    # 2. Detect images and tables
    # 3. Create document chunks
    # 4. Generate embeddings
    # 5. Store in vector store
```

**Excel Processing**:
```python
async def add_payer_plan_excel_embeddings(payload: PayerPlanProcessingRequest, file_path: str, db: AsyncSession):
    # 1. Extract text from Excel sheets
    # 2. Create metadata records
    # 3. Generate document chunks
    # 4. Store embeddings in Qdrant
```

**Search Functionality**:
```python
def find_similar_payer_plan_documents(query: str, payerPlanId: int):
    # Semantic search using Azure Cognitive Search
    # Returns relevant document chunks
```

#### 2. Plan Summary (`payer/plan_summary.py`)

**Purpose**: Generates AI-powered summaries of payer plan documents.

### UB04 Form Analysis (`ub04/`)

#### 1. UB04 Analyzer (`ub04/ub04_analyzer.py`)

**Purpose**: Comprehensive analysis of UB04 medical forms using OCR and AI.

**Key Function**: `get_ub04_form_analysis(form_id)`
```python
async def get_ub04_form_analysis(form_id):
    # 1. Download form image from storage
    # 2. Perform OCR analysis
    # 3. Extract medical codes and procedures
    # 4. Compare with patient clinical data
    # 5. Generate suggestions for ICD/CPT codes
    # 6. Predict procedure prices using contracts
```

**Integration Points**:
- **Patient Data**: Cross-references with FHIR patient records
- **Clinical Data**: Compares with existing conditions, procedures, observations
- **Contract Data**: Uses contract terms for price predictions
- **ML Models**: Employs AI for code suggestions and corrections

#### 2. Code Suggestors

**ICD Suggestor** (`ub04/icd_suggestor.py`):
- Suggests appropriate ICD-10 codes based on clinical data
- Cross-references conditions, observations, procedures

**CPT Suggestor** (`ub04/cpt_suggestor.py`):
- Suggests CPT codes based on procedures and treatments
- Integrates with procedure data and medication requests

### Document Tools (`tools/`)

#### 1. Document Extractor (`tools/document_extractor.py`)

**Purpose**: Multi-format document processing and text extraction.

**Supported Formats**:
- PDF documents
- Word documents (DOCX)
- Excel spreadsheets
- Image files (OCR)

#### 2. Document Chunker (`tools/document_chunker.py`)

**Purpose**: Intelligent text chunking for AI processing.

**Features**:
- Semantic chunking strategies
- Overlap management
- Embedding generation
- Metadata preservation

---

## Claims & Appeals

### Claims Processing (`claims/`)

#### 1. Claims Workflow (`claims/workflow.py`)

**Purpose**: Event-driven claims processing using Inngest workflows.

**Key Workflows**:

**File Processing Workflow**:
```python
@inngest_client.create_function(
    fn_id="claims-process-file",
    trigger=inngest.TriggerEvent(event="claims/process.file")
)
async def claims_process_file(ctx: inngest.Context):
    # 1. Process EDI file
    # 2. Extract claim data
    # 3. Validate claims
    # 4. Store extracted data
```

**ML Prediction Workflow**:
```python
@inngest_client.create_function(
    fn_id="claims-ml-predict",
    trigger=inngest.TriggerEvent(event="claims/ml.predict")
)
async def claims_ml_predict(ctx: inngest.Context):
    # 1. Load claim data
    # 2. Run ML prediction model
    # 3. Calculate rejection probability
    # 4. Generate recommendations
```

#### 2. ML Predictors

**Simple ML Predictor** (`claims/ml_predictor_simple.py`):
- Basic rejection probability calculation
- Risk factor identification
- Simple recommendation engine

**Advanced ML Predictor** (`claims/ml_predictor.py`):
- Ensemble model approach
- Sophisticated feature engineering
- Advanced risk analysis

**Model Trainer** (`claims/model_trainer.py`):
- Model training pipeline
- Feature selection
- Performance monitoring

#### 3. Suggestion Generator (`claims/suggestion_generator.py`)

**Purpose**: Generates actionable suggestions for claim improvements.

**Suggestion Types**:
- Code corrections (ICD/CPT)
- Documentation improvements
- Cost optimizations
- Compliance recommendations

#### 4. Contract Validator (`claims/contract_validator.py`)

**Purpose**: Validates claims against payer contract terms.

**Validation Areas**:
- Coverage verification
- Rate validation
- Authorization requirements
- Compliance checks

### Appeals System (`appeals/`)

#### 1. Appeal Services (`appeals/services/`)

**Appeal Letter Service** (`appeals/services/appeal_letter_service.py`):
- Automated appeal letter generation
- Template management
- Multi-format output (Word, PDF)

**PDR Form Service** (`appeals/services/pdr_form_service.py`):
- Prior Determination Request form processing
- Form field population
- PDF generation

#### 2. Appeal Clients (`appeals/clients/`)

**OpenAI Clients**:
- `AppealFromEobOpenAIClient` - EOB-based appeals
- `AppealFrom835OpenAIClient` - EDI 835-based appeals

#### 3. Appeal Data Access (`appeals/dao.py`)

**Purpose**: Data access layer for appeals processing.

**Key Functions**:
- Denial retrieval and management
- Appeal creation and updates
- Export status tracking
- Bulk operations

---

## Contract Management

### Contract Service (`contract/service.py`)

**Purpose**: Core contract evaluation and analysis service.

**Key Function**: `evaluate_claims()`
```python
async def evaluate_claims(self, contract_group: Providers, claim: ClaimInput):
    # 1. Get contract details for provider
    # 2. Retrieve rate cards for CPT codes
    # 3. Run AI evaluation agent
    # 4. Return evaluation results
```

### Contract Agents (`contract/agents/`)

**Claim Evaluation** (`contract/agents/claim_eval.py`):
- AI-powered claim evaluation against contract terms
- Rate validation and adjustments
- Compliance checking

**CPT Extraction** (`contract/agents/cpt_extraction.py`):
- Extracts CPT codes from contract documents
- Identifies rate schedules and fee structures

**Contract Segmenter** (`contract/agents/contract_segmenter.py`):
- Segments contract documents into logical sections
- Identifies key terms and conditions

### Contract Data (`contract/data/`)

**Models** (`contract/data/model.py`):
```python
class ContractBase(Enum):
    MEDICAL = "medical"
    DENTAL = "dental"
    
class Providers(Enum):
    MEDICARE = "medicare"
    MEDICAID = "medicaid"
```

**Session Management** (`contract/data/session.py`):
- Contract-specific database sessions
- Connection pooling for contract operations

---

## Tools & Utilities

### EDI Processing (`tools/edi_837_importer/`)

#### 1. Main Processor (`tools/edi_837_importer/main.py`)

**Purpose**: Entry point for EDI file processing.

```python
def main():
    # Process EDI 837 (claims) files
    EDI837Processor().process(num_files_to_process)
    # Process EDI 835 (remittance) files  
    EDI835Processor().process(num_files_to_process)
```

#### 2. EDI Processors (`tools/edi_837_importer/edi_importer/processor/`)

**EDI 837 Processor** (`edi_837_processor.py`):
- Processes healthcare claim files
- Extracts patient, provider, and claim data
- Validates claim information

**EDI 835 Processor** (`edi_835_processor.py`):
- Processes remittance advice files
- Extracts payment and adjustment information
- Links payments to original claims

#### 3. Temporal Workflows (`tools/edi_837_importer/temporal_functions/`)

**Purpose**: Reliable, long-running EDI processing workflows.

**Workflow Activities**:
- `edi_835_activities.py` - EDI 835 processing activities
- `patient_updater_activities.py` - Patient data update activities

### Medical Code Tools (`tools/`)

#### 1. ICD Code Tool (`tools/icd_code.py`)

**Purpose**: ICD-10 code lookup and validation.

**Features**:
- Code validation
- Description lookup
- Related code suggestions
- Clinical decision support

#### 2. CPT Code Tool (`tools/cpt_code.py`)

**Purpose**: CPT code processing and validation.

**Features**:
- Procedure code lookup
- Fee schedule integration
- Code validation
- Related procedure suggestions

### Azure Storage Tools (`tools/azure_storage_tools/`)

**Blob Management** (`tools/azure_storage_tools/blob.py`):
- Blob operations (upload, download, delete)
- Metadata management
- Access control

**File Uploader** (`tools/azure_storage_tools/uploader.py`):
- Multi-part upload support
- Progress tracking
- Error handling

### Mock Data Tools (`tools/mock_data/`)

**Patient Data Mocker** (`tools/mock_data/patient_data_mocker/`):
- Generates realistic FHIR patient data
- Anonymization tools
- Test data creation

**EDI Anonymizer** (`tools/mock_data/edi/anonymizer/`):
- Anonymizes EDI files for testing
- Preserves data structure
- HIPAA-compliant de-identification

---

## Statistics & Analytics (`stats/`)

### Stats Generator (`stats/generator.py`)

**Purpose**: AI-powered SQL generation and chart creation from natural language.

**Key Function**: `get_sql_query_from_ai()`
```python
def get_sql_query_from_ai(visualisation_description: str):
    # 1. Read Prisma schema
    # 2. Generate SQL from natural language
    # 3. Validate query structure
    # 4. Return SQL and chart type
```

**Chart Generators**:
- `bar_chart_generator.py` - Bar chart configurations
- `line_chart_generator.py` - Line chart configurations
- `pie_chart_generator.py` - Pie chart configurations
- `area_chart_generator.py` - Area chart configurations
- `radar_chart_generator.py` - Radar chart configurations
- `table_generator.py` - Table configurations

### Query Execution

```python
async def execute_query(sql_query: str):
    # 1. Create database connection
    # 2. Execute SQL query
    # 3. Process results
    # 4. Return structured data
```

---

## Pre-Authorization (`pre_auth/notifications/`)

### Voice Call Integration

#### 1. Vapi Webhook Handler (`pre_auth/notifications/vapi_webhook_handler.py`)

**Purpose**: Handles voice call webhooks from Vapi service.

**Event Types**:
- `end-of-call-report` - Call completion with transcript
- `hang` - Call failure notifications
- Status updates (in-progress, forwarding, ended)

**Webhook Processing**:
```python
class VapiWebhookEventHandler:
    async def handle_event(self, webhook_data: Dict[str, Any], signature: str):
        # 1. Store webhook event in database
        # 2. Verify signature
        # 3. Route to appropriate handler
        # 4. Send Inngest event for processing
```

#### 2. Inngest Handlers (`pre_auth/notifications/inngest_handlers.py`)

**Call Processing Workflows**:
- Call completion processing
- Transcript analysis
- Summary generation
- Follow-up actions

#### 3. Vapi Routes (`pre_auth/notifications/vapi_webhook_routes.py`)

**FastAPI Routes**:
```python
@router.post("/webhook")
async def handle_vapi_webhook()
    # Processes incoming Vapi webhooks
```

---

## Vector Stores (`stores/`)

### Qdrant Vector Store (`stores/qdrant_vector_store.py`)

**Purpose**: Vector database integration for semantic search.

**Features**:
```python
class QdrantVectorStore:
    def add_documents(self, documents: List[Document]):
        # 1. Generate embeddings using Azure OpenAI
        # 2. Store vectors in Qdrant
        # 3. Include metadata for filtering
    
    def search(self, query: str, k: int = 5):
        # 1. Embed query
        # 2. Perform vector search
        # 3. Return ranked results
```

**Integration**:
- Document embedding storage
- Semantic search capabilities
- Metadata filtering
- Similarity scoring

---

## Authentication & Security (`auth/`)

### JWT Bearer Authentication (`auth/jwt_bearer.py`)

**Purpose**: JWT-based authentication for all API endpoints.

```python
class JWTBearer:
    async def __call__(self, credentials: HTTPAuthorizationCredentials):
        # 1. Extract JWT token
        # 2. Validate token signature
        # 3. Check expiration
        # 4. Return user account info
```

**Integration**: Used as dependency in all protected routes.

### User Account (`auth/user_account.py`)

**Purpose**: User account representation and management.

---

## Configuration & Infrastructure

### Environment Configuration

**Required Variables**:
```env
# Database
DATABASE_URL=postgresql+asyncpg://...

# Azure OpenAI
AZURE_OPENAI_API_KEY=...
AZURE_OPENAI_ENDPOINT=...
AZURE_OPENAI_API_VERSION=...
AZURE_OPENAI_DEPLOYMENT_NAME=...

# Azure Storage
AZURE_STORAGE_CONNECTION_STRING=...
REJECTION_FILE_CONTAINER=...

# Authentication
NEXTJS_JWT_SECRET=...

# Inngest
INNGEST_EVENT_KEY=...
INNGEST_APP_ID=...

# Vector Store
AZURE_SEARCH_ENDPOINT=...
AZURE_SEARCH_ADMIN_KEY=...
```

### Logging & Monitoring (`lib/instrumentation.py`)

**Features**:
- Structured logging with Loguru
- OpenTelemetry tracing
- Loki log aggregation
- Performance monitoring

**Setup**:
```python
def setup_logging():
    # Configure Loguru
    # Filter out verbose library logs
    # Setup remote logging

def setup_tracing(app):
    # OpenTelemetry configuration
    # FastAPI instrumentation
    # Request tracing
```

### Workflow Orchestration (`inngest_client.py`)

**Purpose**: Event-driven workflow orchestration using Inngest.

**Features**:
- Reliable function execution
- Event-driven architecture
- Workflow state management
- Error handling and retries

**Function Registration**:
```python
@register_inngest_function
@inngest_client.create_function(
    fn_id="function-id",
    trigger=inngest.TriggerEvent(event="event/name")
)
async def workflow_function(ctx: inngest.Context):
    # Workflow logic
```

---

## Testing (`tests/`)

### Test Structure

```
tests/
├── unit/                    # Unit tests
│   ├── agents/             # Agent system tests
│   ├── chat/               # Chat system tests
│   ├── data_sync/          # FHIR sync tests
│   └── faker_factories/    # Test data factories
├── functional/             # Functional tests
└── fixtures/              # Test fixtures and data
```

### Faker Factories (`tests/unit/faker_factories/`)

**FHIR Resource Factories**:
- `patient_factory.py` - Realistic patient data
- `encounter_factory.py` - Encounter test data
- `observation_factory.py` - Observation test data

**Test Data Factories**:
- `chat_factory.py` - Chat session test data
- `message_factory.py` - Message test data
- `payer_factory.py` - Payer plan test data

### Running Tests

```bash
# Run all tests
python -m pytest tests/

# Run specific test category
python -m pytest tests/unit/
python -m pytest tests/functional/

# Run with coverage
python -m pytest --cov=. tests/
```

---

## Deployment

### Docker Configuration

The application can be containerized using Docker:

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 9000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "9000"]
```

### Production Considerations

1. **Environment Variables**: Ensure all required environment variables are set
2. **Database Migrations**: Run database migrations before deployment
3. **Health Checks**: Configure load balancer health checks to use `/health` endpoint
4. **Scaling**: The application is designed to be horizontally scalable
5. **Monitoring**: Setup monitoring for all critical dependencies
6. **Logging**: Configure centralized logging aggregation

### Performance Optimization

1. **Database Connection Pooling**: Configured for optimal performance
2. **Async Processing**: All I/O operations are asynchronous
3. **Caching**: Vector search results are cached where appropriate
4. **Streaming**: Large responses use streaming to reduce memory usage

---

## API Integration Examples

### Creating a Chat Session

```python
# Create chat session
POST /api/chat
{
    "userId": 1,
    "title": "Patient Discussion",
    "context": "Patient ID: 12345"
}

# Send message
POST /messages/{chat_id}
{
    "message": "What is the patient's current condition?",
    "contexts": [
        {
            "type": "PATIENT",
            "id": "12345",
            "name": "John Doe"
        }
    ]
}
```

### Processing Payer Documents

```python
POST /payer-plan-knowledge
{
    "fileResourceId": 123,
    "versionId": 456,
    "documentId": 789,
    "payerId": 1,
    "planId": 2
}
```

### Generating Appeal Letters

```python
POST /v1/denials/123/appeals
# Automatically generates appeal letter based on denial type
```

### FHIR Data Sync

```python
POST /sync-fhir-resource
Headers: {
    "x-synapse-signature": "signature"
}
Body: {
    # FHIR Bundle or Resource
}
```

---

## Troubleshooting

### Common Issues

1. **Database Connection Issues**:
   - Check DATABASE_URL environment variable
   - Verify database is accessible
   - Check connection pool settings

2. **Azure OpenAI Issues**:
   - Verify API key and endpoint
   - Check rate limits
   - Monitor token usage

3. **FHIR Sync Issues**:
   - Validate webhook signatures
   - Check FHIR resource structure
   - Monitor sync logs

4. **Agent System Issues**:
   - Check agent selection logic
   - Verify model client configuration
   - Monitor memory usage

### Logging

The application provides comprehensive logging:

```python
# Import logger
from loguru import logger

# Log levels
logger.debug("Debug information")
logger.info("General information")
logger.warning("Warning message")
logger.error("Error occurred")
```

### Health Monitoring

Monitor the `/health` endpoint for system status:

```json
{
    "status": "ok",
    "dependencies": {
        "database": {"status": "healthy"},
        "azure_storage": {"status": "healthy"},
        "azure_openai": {"status": "healthy"}
    }
}
```

---

## Contributing

### Development Setup

1. Clone the repository
2. Install dependencies: `pip install -r requirements.txt`
3. Set up environment variables
4. Run database migrations
5. Start the development server: `uvicorn main:app --reload`

### Code Standards

- Use async/await for all I/O operations
- Follow PEP 8 style guidelines
- Add comprehensive docstrings
- Write unit tests for new functionality
- Use type hints throughout

### Adding New Agents

1. Create agent file in `agents/`
2. Register in `agents/factory.py`
3. Add agent constants
4. Update selector prompt logic
5. Write tests

---

This documentation provides a comprehensive overview of the FastAPI AI Services system. Each component is designed to work together to provide a robust, scalable healthcare AI platform.
