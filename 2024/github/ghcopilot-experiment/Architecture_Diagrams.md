# Architecture Diagrams

This document contains visual diagrams for both architectures using Mermaid.

## Azure Native Architecture Diagram

```mermaid
graph TB
    subgraph "Frontend"
        WebApp[Azure Static Web Apps<br/>React.js]
    end
    
    subgraph "API Layer"
        APIM[Azure API Management<br/>Gateway & Security]
    end
    
    subgraph "Application Services"
        Upload[Upload Service<br/>Azure App Service]
        Process[Processing Service<br/>Azure Container Instances]
        AI[AI Detection Service<br/>Azure ML / AKS]
        Notify[Notification Service<br/>Azure Functions]
    end
    
    subgraph "Data Layer"
        Blob[Azure Blob Storage<br/>Images]
        DB[(Azure Cosmos DB<br/>Metadata)]
        Cache[(Azure Redis Cache<br/>Sessions)]
    end
    
    subgraph "Messaging"
        SB[Azure Service Bus<br/>Message Queues]
        EG[Azure Event Grid<br/>Events]
    end
    
    subgraph "Security & Monitoring"
        AAD[Azure AD<br/>Identity]
        KV[Azure Key Vault<br/>Secrets]
        AI_MON[Application Insights<br/>Monitoring]
        Monitor[Azure Monitor<br/>Metrics]
    end
    
    WebApp -->|API Calls| APIM
    APIM -->|Route| Upload
    APIM -->|Route| Process
    APIM -->|Route| AI
    
    Upload -->|Store Raw| Blob
    Upload -->|Metadata| DB
    Upload -->|Event| SB
    
    SB -->|Process Queue| Process
    Process -->|Store Processed| Blob
    Process -->|Event| SB
    
    SB -->|Detection Queue| AI
    AI -->|Results| DB
    AI -->|Event| SB
    
    SB -->|Notification Queue| Notify
    Notify -->|Update| DB
    
    APIM -->|Cache| Cache
    DB -->|Cache| Cache
    
    Upload -.->|Auth| AAD
    Process -.->|Secrets| KV
    AI -.->|Secrets| KV
    
    Upload -->|Logs/Metrics| AI_MON
    Process -->|Logs/Metrics| AI_MON
    AI -->|Logs/Metrics| AI_MON
    AI_MON -->|Aggregate| Monitor
    
    style WebApp fill:#0078d4
    style APIM fill:#0078d4
    style Upload fill:#00bcf2
    style Process fill:#00bcf2
    style AI fill:#00bcf2
    style Blob fill:#ff8c00
    style DB fill:#ff8c00
    style SB fill:#59b4d9
    style AAD fill:#f25022
    style KV fill:#f25022
```

## OpenShift on Azure Architecture Diagram

```mermaid
graph TB
    subgraph "OpenShift Frontend Project"
        WebDeploy[Frontend Deployment<br/>React + Nginx Pods]
        WebSvc[Service]
        WebRoute[Route<br/>TLS Termination]
    end
    
    subgraph "OpenShift API Project"
        APIGateway[3scale API Management<br/>or Kong Gateway]
        APIRoute[Route<br/>api.domain.com]
    end
    
    subgraph "OpenShift Application Projects"
        UploadDeploy[Upload Service<br/>Deployment + HPA]
        ProcessJob[Processing Service<br/>Jobs]
        AIDeploy[AI Service<br/>Deployment + GPU]
        NotifyKnative[Notification<br/>Knative Service]
    end
    
    subgraph "Azure Managed Services"
        AzureBlob[Azure Blob Storage<br/>Images]
        AzureDB[(Azure PostgreSQL<br/>Metadata)]
        AzureCache[(Azure Redis Cache<br/>Optional)]
    end
    
    subgraph "OpenShift Data Project"
        RedisOp[Redis Operator<br/>In-cluster Cache]
    end
    
    subgraph "OpenShift Messaging Project"
        Kafka[AMQ Streams<br/>Kafka Cluster]
        KafkaTopic1[upload-topic]
        KafkaTopic2[processing-topic]
        KafkaTopic3[detection-topic]
    end
    
    subgraph "OpenShift Monitoring"
        Prometheus[Prometheus<br/>Metrics]
        Grafana[Grafana<br/>Dashboards]
        Jaeger[Jaeger<br/>Tracing]
        EFK[EFK Stack<br/>Logging]
    end
    
    subgraph "Security"
        AzureAD[Azure AD<br/>Identity Provider]
        OSAuthOAuth[OpenShift OAuth<br/>Server]
        ExtSecrets[External Secrets<br/>+ Azure Key Vault]
        NetPol[Network Policies<br/>Firewall Rules]
    end
    
    WebRoute -->|HTTPS| WebSvc
    WebSvc --> WebDeploy
    WebDeploy -->|API Calls| APIRoute
    
    APIRoute --> APIGateway
    APIGateway -->|Route| UploadDeploy
    APIGateway -->|Route| ProcessJob
    APIGateway -->|Route| AIDeploy
    
    UploadDeploy -->|SDK| AzureBlob
    UploadDeploy -->|SQL| AzureDB
    UploadDeploy -->|Publish| KafkaTopic1
    
    KafkaTopic1 -->|Consume| ProcessJob
    ProcessJob -->|Store| AzureBlob
    ProcessJob -->|Publish| KafkaTopic2
    
    KafkaTopic2 -->|Consume| AIDeploy
    AIDeploy -->|Results| AzureDB
    AIDeploy -->|Publish| KafkaTopic3
    
    KafkaTopic3 -->|Event| NotifyKnative
    NotifyKnative -->|Update| AzureDB
    
    APIGateway -->|Cache| RedisOp
    UploadDeploy -->|Cache| RedisOp
    
    UploadDeploy -.->|Auth| OSAuthOAuth
    OSAuthOAuth -.->|Federate| AzureAD
    
    UploadDeploy -.->|Secrets| ExtSecrets
    ProcessJob -.->|Secrets| ExtSecrets
    AIDeploy -.->|Secrets| ExtSecrets
    
    UploadDeploy -->|Metrics| Prometheus
    ProcessJob -->|Metrics| Prometheus
    AIDeploy -->|Metrics| Prometheus
    
    Prometheus --> Grafana
    UploadDeploy -->|Traces| Jaeger
    ProcessJob -->|Traces| Jaeger
    AIDeploy -->|Traces| Jaeger
    
    UploadDeploy -->|Logs| EFK
    ProcessJob -->|Logs| EFK
    AIDeploy -->|Logs| EFK
    
    NetPol -.->|Protect| UploadDeploy
    NetPol -.->|Protect| ProcessJob
    NetPol -.->|Protect| AIDeploy
    
    style WebDeploy fill:#cc0000
    style UploadDeploy fill:#cc0000
    style ProcessJob fill:#cc0000
    style AIDeploy fill:#cc0000
    style APIGateway fill:#cc0000
    style Kafka fill:#000000,color:#fff
    style AzureBlob fill:#0078d4,color:#fff
    style AzureDB fill:#0078d4,color:#fff
    style Prometheus fill:#e6522c
    style Grafana fill:#f46800
```

## Simplified Comparison Diagram

```mermaid
graph LR
    subgraph "Azure Native"
        A1[Static Web Apps]
        A2[API Management]
        A3[App Service]
        A4[Container Instances]
        A5[Blob Storage]
        A6[Cosmos DB]
        A7[Service Bus]
        A8[App Insights]
        
        A1 --> A2
        A2 --> A3
        A3 --> A5
        A3 --> A6
        A3 --> A7
        A4 --> A5
        A3 --> A8
    end
    
    subgraph "OpenShift on Azure"
        O1[Deployment + Route]
        O2[3scale / Kong]
        O3[Deployment + HPA]
        O4[Job / Pod]
        O5[Azure Blob<br/>Hybrid]
        O6[Azure PostgreSQL<br/>Hybrid]
        O7[AMQ Streams]
        O8[Prometheus + Jaeger]
        
        O1 --> O2
        O2 --> O3
        O3 --> O5
        O3 --> O6
        O3 --> O7
        O4 --> O5
        O3 --> O8
    end
    
    A1 -.->|Migrate| O1
    A2 -.->|Migrate| O2
    A3 -.->|Migrate| O3
    A4 -.->|Migrate| O4
    A5 -.->|Keep/Hybrid| O5
    A6 -.->|Migrate| O6
    A7 -.->|Migrate| O7
    A8 -.->|Migrate| O8
    
    style A1 fill:#0078d4,color:#fff
    style A2 fill:#0078d4,color:#fff
    style A3 fill:#0078d4,color:#fff
    style A4 fill:#0078d4,color:#fff
    style A5 fill:#0078d4,color:#fff
    style A6 fill:#0078d4,color:#fff
    style A7 fill:#0078d4,color:#fff
    style A8 fill:#0078d4,color:#fff
    
    style O1 fill:#cc0000,color:#fff
    style O2 fill:#cc0000,color:#fff
    style O3 fill:#cc0000,color:#fff
    style O4 fill:#cc0000,color:#fff
    style O5 fill:#7f1084,color:#fff
    style O6 fill:#7f1084,color:#fff
    style O7 fill:#cc0000,color:#fff
    style O8 fill:#cc0000,color:#fff
```

## Data Flow Comparison

### Azure Native Data Flow
```mermaid
sequenceDiagram
    participant User
    participant WebApp as Azure Static Web Apps
    participant APIM as Azure API Management
    participant Upload as Upload Service
    participant Blob as Azure Blob Storage
    participant SB as Service Bus
    participant Process as Processing Service
    participant AI as AI Service
    participant DB as Cosmos DB
    
    User->>WebApp: Upload Image
    WebApp->>APIM: POST /upload
    APIM->>Upload: Route request
    Upload->>Blob: Store raw image
    Upload->>DB: Save metadata
    Upload->>SB: Publish upload event
    SB->>Process: Consume event
    Process->>Blob: Retrieve raw image
    Process->>Blob: Store processed image
    Process->>SB: Publish process event
    SB->>AI: Consume event
    AI->>Blob: Retrieve image
    AI->>DB: Store detection results
    AI->>SB: Publish completion
    DB->>APIM: Return status
    APIM->>WebApp: Response
    WebApp->>User: Display result
```

### OpenShift on Azure Data Flow
```mermaid
sequenceDiagram
    participant User
    participant Route as OpenShift Route
    participant Frontend as Frontend Pod
    participant Gateway as 3scale Gateway
    participant Upload as Upload Pod
    participant Blob as Azure Blob Storage
    participant Kafka as AMQ Streams
    participant Process as Processing Job
    participant AI as AI Pod
    participant DB as Azure PostgreSQL
    
    User->>Route: Upload Image
    Route->>Frontend: HTTPS
    Frontend->>Gateway: API Call
    Gateway->>Upload: Route request
    Upload->>Blob: Store raw image (SDK)
    Upload->>DB: Save metadata (SQL)
    Upload->>Kafka: Publish to upload-topic
    Kafka->>Process: Job triggered
    Process->>Blob: Retrieve raw image
    Process->>Blob: Store processed image
    Process->>Kafka: Publish to process-topic
    Kafka->>AI: Consume event
    AI->>Blob: Retrieve image
    AI->>DB: Store detection results
    AI->>Kafka: Publish completion
    DB->>Gateway: Return status
    Gateway->>Frontend: Response
    Frontend->>User: Display result
```

## Network Architecture - OpenShift on Azure

```mermaid
graph TB
    subgraph "Internet"
        Users[Users/Clients]
    end
    
    subgraph "Azure Public Services"
        CDN[Azure CDN<br/>Optional]
    end
    
    subgraph "Azure VNet - 10.0.0.0/16"
        subgraph "ARO Cluster Subnet - 10.0.1.0/24"
            Master[Master Nodes<br/>Control Plane]
            Worker1[Worker Node 1]
            Worker2[Worker Node 2]
            Worker3[Worker Node 3]
            Router[OpenShift Router<br/>Ingress]
        end
        
        subgraph "Private Endpoint Subnet - 10.0.2.0/24"
            PE1[Private Endpoint<br/>Azure Blob]
            PE2[Private Endpoint<br/>Azure PostgreSQL]
            PE3[Private Endpoint<br/>Azure Redis]
        end
        
        subgraph "Application Gateway Subnet - 10.0.3.0/24"
            AppGW[Azure Application Gateway<br/>Optional WAF]
        end
    end
    
    subgraph "Azure PaaS Services"
        AzBlob[Azure Blob Storage]
        AzDB[Azure PostgreSQL]
        AzRedis[Azure Redis Cache]
        AzKV[Azure Key Vault]
    end
    
    subgraph "Security Controls"
        NSG1[NSG - ARO Subnet]
        NSG2[NSG - PE Subnet]
        NetPol[Network Policies<br/>Within OpenShift]
        FW[Azure Firewall<br/>Optional]
    end
    
    Users -->|HTTPS| CDN
    CDN -->|HTTPS| AppGW
    AppGW -->|HTTPS| Router
    Users -.->|Direct| Router
    
    Router --> Worker1
    Router --> Worker2
    Router --> Worker3
    
    Worker1 -->|Private| PE1
    Worker2 -->|Private| PE2
    Worker3 -->|Private| PE3
    
    PE1 -->|Private Link| AzBlob
    PE2 -->|Private Link| AzDB
    PE3 -->|Private Link| AzRedis
    
    Worker1 -.->|Secrets| AzKV
    Worker2 -.->|Secrets| AzKV
    Worker3 -.->|Secrets| AzKV
    
    NSG1 -.->|Protect| Worker1
    NSG1 -.->|Protect| Worker2
    NSG1 -.->|Protect| Worker3
    NSG2 -.->|Protect| PE1
    NSG2 -.->|Protect| PE2
    
    NetPol -.->|Pod Security| Worker1
    NetPol -.->|Pod Security| Worker2
    NetPol -.->|Pod Security| Worker3
    
    style Master fill:#cc0000,color:#fff
    style Worker1 fill:#cc0000,color:#fff
    style Worker2 fill:#cc0000,color:#fff
    style Worker3 fill:#cc0000,color:#fff
    style Router fill:#ee0000,color:#fff
    style AzBlob fill:#0078d4,color:#fff
    style AzDB fill:#0078d4,color:#fff
    style AzRedis fill:#0078d4,color:#fff
    style AppGW fill:#0078d4,color:#fff
```

## Technology Stack Layers

```mermaid
graph TD
    subgraph "Azure Native Stack"
        AZ_L1[Presentation: Static Web Apps]
        AZ_L2[API: API Management]
        AZ_L3[Application: App Service + Functions + ACI]
        AZ_L4[Data: Blob + Cosmos DB + Redis]
        AZ_L5[Integration: Service Bus + Event Grid]
        AZ_L6[Platform: Azure]
        
        AZ_L1 --> AZ_L2
        AZ_L2 --> AZ_L3
        AZ_L3 --> AZ_L4
        AZ_L3 --> AZ_L5
        AZ_L5 --> AZ_L6
    end
    
    subgraph "OpenShift Stack"
        OS_L1[Presentation: Deployment + Route]
        OS_L2[API: 3scale + Service Mesh]
        OS_L3[Application: Deployments + Jobs + Knative]
        OS_L4[Data: Azure Blob + PostgreSQL + Redis Op]
        OS_L5[Integration: AMQ Streams + Knative Eventing]
        OS_L6[Platform: OpenShift on Azure ARO]
        
        OS_L1 --> OS_L2
        OS_L2 --> OS_L3
        OS_L3 --> OS_L4
        OS_L3 --> OS_L5
        OS_L5 --> OS_L6
    end
    
    AZ_L1 -.->|Transform| OS_L1
    AZ_L2 -.->|Transform| OS_L2
    AZ_L3 -.->|Transform| OS_L3
    AZ_L4 -.->|Hybrid| OS_L4
    AZ_L5 -.->|Transform| OS_L5
    AZ_L6 -.->|Platform Change| OS_L6
    
    style AZ_L1 fill:#0078d4,color:#fff
    style AZ_L2 fill:#0078d4,color:#fff
    style AZ_L3 fill:#0078d4,color:#fff
    style AZ_L4 fill:#0078d4,color:#fff
    style AZ_L5 fill:#0078d4,color:#fff
    style AZ_L6 fill:#0078d4,color:#fff
    
    style OS_L1 fill:#cc0000,color:#fff
    style OS_L2 fill:#cc0000,color:#fff
    style OS_L3 fill:#cc0000,color:#fff
    style OS_L4 fill:#7f1084,color:#fff
    style OS_L5 fill:#cc0000,color:#fff
    style OS_L6 fill:#cc0000,color:#fff
```

## Notes

- **Blue**: Azure-native services
- **Red**: OpenShift-native components
- **Purple**: Hybrid (Azure services used from OpenShift)
- **Arrows**: Data/control flow
- **Dotted lines**: Security/authentication relationships

These diagrams provide a visual representation of:
1. The complete architecture for both platforms
2. Component relationships and data flow
3. Network topology for OpenShift on Azure
4. Migration path from Azure to OpenShift
5. Technology stack layering

The diagrams can be rendered in any Markdown viewer that supports Mermaid, including GitHub, GitLab, and various documentation platforms.
