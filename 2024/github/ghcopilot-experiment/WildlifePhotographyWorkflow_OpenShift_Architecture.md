# Wildlife Photography Workflow Architecture - Red Hat OpenShift on Azure

## Overview
This document describes the equivalent architecture for the Wildlife Photography Workflow when deployed on Red Hat OpenShift running on Microsoft Azure (Azure Red Hat OpenShift - ARO). This architecture leverages OpenShift's container orchestration capabilities while integrating with Azure services where appropriate.

## Architecture Components

### 1. Frontend Layer
**Component**: Web Application
- **Original**: Azure Static Web Apps
- **OpenShift Equivalent**: 
  - **Deployment**: Frontend container deployed as OpenShift Deployment
  - **Technology**: React.js / Next.js containerized with Nginx
  - **Service**: OpenShift Service (ClusterIP)
  - **Route**: OpenShift Route with TLS termination
  - **Scaling**: Horizontal Pod Autoscaler (HPA)

### 2. API Gateway
**Component**: API Management
- **Original**: Azure API Management
- **OpenShift Equivalent**:
  - **Option 1**: Red Hat 3scale API Management (OpenShift Operator)
  - **Option 2**: Kong API Gateway (deployed as OpenShift Deployment)
  - **Option 3**: OpenShift Service Mesh (Istio-based) for advanced routing
  - **Features**:
    - Route-based API versioning
    - Rate limiting via 3scale or custom middleware
    - OAuth 2.0/JWT validation
    - Certificate management via cert-manager

### 3. Application Services

#### Upload Service
- **Original**: Azure App Service / Azure Functions
- **OpenShift Equivalent**:
  - **Deployment**: OpenShift Deployment/StatefulSet
  - **Technology**: Node.js / Python FastAPI in container
  - **Service**: OpenShift Service
  - **Scaling**: HPA based on CPU/memory or custom metrics
  - **Storage**: PersistentVolumeClaim for temporary storage
  - **Alternative**: Knative Serving for serverless capabilities

#### Processing Service
- **Original**: Azure Container Instances / AKS
- **OpenShift Equivalent**:
  - **Deployment**: OpenShift Job or Deployment
  - **Technology**: Python with OpenCV/Pillow
  - **Pattern**: Job-based processing triggered by events
  - **Scaling**: HPA or Job parallelism
  - **Resource Management**: ResourceQuotas and LimitRanges

#### AI Detection Service
- **Original**: Azure Machine Learning / AKS
- **OpenShift Equivalent**:
  - **Deployment**: OpenShift Deployment with GPU node affinity
  - **Technology**: Python with TensorFlow/PyTorch
  - **ML Platform**: Red Hat OpenShift Data Science (RHODS) or Kubeflow
  - **Model Serving**: Seldon Core or KServe (formerly KFServing)
  - **GPU Support**: Node labels for GPU-enabled nodes
  - **Scaling**: HPA with custom metrics from model serving

### 4. Storage Layer

#### Object Storage
- **Original**: Azure Blob Storage
- **OpenShift Equivalent**:
  - **Option 1 (Hybrid)**: Continue using Azure Blob Storage via SDK/API
    - Deploy Azure Storage SDK in containers
    - Use Azure Managed Identity via OpenShift Workload Identity
  - **Option 2 (Cloud-Native)**: OpenShift Data Foundation (ODF) with NooBaa
    - S3-compatible object storage
    - Multi-cloud bucket abstraction
  - **Option 3**: Minio deployed on OpenShift
  - **Recommendation**: Hybrid approach - Azure Blob for cost-effectiveness
  
  **PVC Structure**:
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: image-processing-cache
  spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 100Gi
    storageClassName: ocs-storagecluster-cephfs
  ```

#### Metadata Database
- **Original**: Azure Cosmos DB / Azure SQL Database
- **OpenShift Equivalent**:
  - **Option 1 (Managed)**: Azure Database for PostgreSQL/MySQL via service binding
  - **Option 2 (In-cluster)**: PostgreSQL/MySQL Operator-managed database
    - Crunchy PostgreSQL Operator
    - Percona Operator for MySQL
  - **Option 3**: MongoDB deployed via Operator
  - **Backup**: Velero for database backups
  - **Recommendation**: Hybrid - Azure Database for PostgreSQL for production

#### Cache Layer
- **Original**: Azure Redis Cache
- **OpenShift Equivalent**:
  - **Deployment**: Redis deployed via Redis Operator
  - **Alternative**: Continue using Azure Cache for Redis (managed service)
  - **Configuration**: ConfigMap for Redis configuration
  - **Persistence**: PVC for Redis data persistence
  - **High Availability**: Redis Sentinel or Redis Cluster mode

### 5. Message Queue / Event Processing

#### Message Broker
- **Original**: Azure Service Bus / Event Grid
- **OpenShift Equivalent**:
  - **Option 1**: Red Hat AMQ Streams (Kafka-based)
    - Strimzi Kafka Operator
    - Topics for different processing stages
  - **Option 2**: Red Hat AMQ Broker (ActiveMQ Artemis)
  - **Option 3**: RabbitMQ via Operator
  - **Event-Driven**: Knative Eventing for event processing
  
  **Topics/Queues**:
  - `upload-topic`: New image uploads
  - `processing-topic`: Images awaiting processing
  - `detection-topic`: Images for AI analysis
  - `notification-topic`: User notifications

### 6. Monitoring & Logging

#### Application Monitoring
- **Original**: Azure Application Insights
- **OpenShift Equivalent**:
  - **Built-in**: OpenShift built-in Prometheus and Grafana
  - **APM**: Jaeger for distributed tracing (included in OpenShift Service Mesh)
  - **Advanced**: Elastic APM or Datadog agent deployment
  - **User Workload Monitoring**: Custom ServiceMonitor resources

#### Logging
- **Original**: Azure Log Analytics
- **OpenShift Equivalent**:
  - **Built-in**: OpenShift Logging (EFK Stack)
    - Elasticsearch for log storage
    - Fluentd for log collection
    - Kibana for visualization
  - **Alternative**: Red Hat OpenShift Logging Operator with Loki
  - **ClusterLogForwarder**: Forward logs to external systems

#### Metrics
- **Original**: Azure Monitor
- **OpenShift Equivalent**:
  - **Prometheus**: Built-in metrics collection
  - **Grafana**: Dashboard visualization
  - **AlertManager**: Alert routing and management
  - **Custom Metrics**: ServiceMonitor and PodMonitor CRDs

### 7. Security Components

#### Identity Management
- **Original**: Azure Active Directory
- **OpenShift Equivalent**:
  - **OpenShift OAuth**: Built-in OAuth server
  - **Identity Providers**: Configure Azure AD as OpenShift IdP
  - **RBAC**: OpenShift Role-Based Access Control
  - **Service Accounts**: For pod-to-pod authentication
  - **Azure Integration**: Azure AD Workload Identity for accessing Azure resources

#### Secrets Management
- **Original**: Azure Key Vault
- **OpenShift Equivalent**:
  - **Built-in**: OpenShift Secrets
  - **External Secrets**: External Secrets Operator with Azure Key Vault backend
  - **Sealed Secrets**: For GitOps-based secret management
  - **Vault**: HashiCorp Vault Operator integration

#### Network Security
- **Original**: Azure Virtual Network, NSGs
- **OpenShift Equivalent**:
  - **Network Policies**: Kubernetes NetworkPolicy resources
  - **Egress Firewall**: OpenShift EgressNetworkPolicy
  - **Service Mesh**: Istio/Service Mesh for mTLS between services
  - **Azure Integration**: ARO cluster in Azure VNet with NSG rules

## OpenShift-Specific Features

### 1. Operators
Use OpenShift Operators for managing applications:
- **Strimzi Kafka Operator**: Message queue
- **Crunchy PostgreSQL Operator**: Database
- **Redis Operator**: Caching
- **Seldon Operator**: ML model serving
- **Cert-Manager Operator**: Certificate management

### 2. Routes vs Ingress
- **Routes**: OpenShift-native (used for external access)
- **Automatic TLS**: Let's Encrypt integration via cert-manager
- **Path-based Routing**: Different routes for different services

### 3. Image Streams and Build Configs
- **BuildConfig**: Build container images from source
- **ImageStream**: Track and trigger deployments on image updates
- **S2I (Source-to-Image)**: Build images directly from source code

### 4. Projects (Namespaces)
Organize components into OpenShift Projects:
- `wildlife-frontend`: Frontend application
- `wildlife-api`: API services
- `wildlife-processing`: Image processing services
- `wildlife-data`: Data layer (databases, caching)
- `wildlife-ml`: ML/AI services
- `wildlife-monitoring`: Monitoring stack

## Deployment Architecture

```yaml
# Example Deployment for Upload Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: upload-service
  namespace: wildlife-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: upload-service
  template:
    metadata:
      labels:
        app: upload-service
    spec:
      containers:
      - name: upload-service
        image: quay.io/wildlife/upload-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: AZURE_STORAGE_CONNECTION
          valueFrom:
            secretKeyRef:
              name: azure-storage-secret
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: upload-service
  namespace: wildlife-api
spec:
  selector:
    app: upload-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: upload-service
  namespace: wildlife-api
spec:
  to:
    kind: Service
    name: upload-service
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

## Workflow Flow on OpenShift

1. **Image Upload**:
   - User uploads via frontend (OpenShift Route → Service → Pod)
   - Upload service validates and stores to Azure Blob Storage (using SDK)
   - Publishes event to Kafka topic `upload-topic`

2. **Processing Pipeline**:
   - Processing service (Kafka consumer) picks up from `upload-topic`
   - Launches OpenShift Job for image processing
   - Job processes image and stores results in Azure Blob
   - Publishes to `detection-topic`

3. **AI Detection**:
   - AI service (running on GPU-enabled nodes) consumes from `detection-topic`
   - Performs inference using model served via KServe
   - Stores results in PostgreSQL database
   - Publishes to `notification-topic`

4. **Notification**:
   - Notification service consumes from `notification-topic`
   - Updates image status in database
   - Sends notification to user

5. **Gallery Display**:
   - User accesses via frontend Route
   - API calls routed through 3scale API Management
   - Data retrieved from Redis cache or PostgreSQL
   - Images served from Azure Blob Storage via CDN

## Scalability Considerations

- **Horizontal Pod Autoscaler (HPA)**: Auto-scale based on CPU, memory, or custom metrics
- **Vertical Pod Autoscaler (VPA)**: Automatically adjust resource requests/limits
- **Cluster Autoscaler**: Add/remove nodes based on demand (ARO supports this)
- **Machine Sets**: Define different node types (GPU, high-memory, etc.)
- **Storage Scaling**: OpenShift Data Foundation can scale dynamically

## High Availability

- **Multi-AZ Deployment**: Deploy ARO across multiple Azure Availability Zones
- **Pod Disruption Budgets**: Ensure minimum replicas during updates
- **StatefulSets**: For stateful applications requiring stable network identities
- **Anti-Affinity Rules**: Spread pods across nodes/zones
- **Database HA**: PostgreSQL replication or Azure managed service
- **Load Balancing**: OpenShift Router (HAProxy) provides load balancing

## Cost Optimization

- **Cluster Autoscaling**: Scale down during off-peak hours
- **Spot Instances**: Use Azure Spot VMs for worker nodes (non-critical workloads)
- **Resource Quotas**: Prevent resource over-allocation
- **Limit Ranges**: Set default limits for containers
- **Storage Tiers**: Use Azure Blob Storage tiers for archival
- **Serverless**: Knative for event-driven, pay-per-use workloads

## CI/CD Integration

- **OpenShift Pipelines**: Tekton-based CI/CD
- **GitOps**: ArgoCD or OpenShift GitOps Operator
- **Image Registry**: OpenShift internal registry or Quay
- **Build Process**: BuildConfig with S2I or Docker builds

## Migration Mapping Table

| Azure Service | OpenShift on Azure Equivalent | Notes |
|---------------|------------------------------|-------|
| Azure Static Web Apps | OpenShift Deployment + Route | Container-based deployment |
| Azure API Management | 3scale API Management / Kong | Can also use Azure APIM for hybrid |
| Azure App Service | OpenShift Deployment | Containerized applications |
| Azure Functions | Knative Serving | Serverless on OpenShift |
| Azure Container Instances | OpenShift Job/Pod | On-demand container execution |
| AKS | Azure Red Hat OpenShift (ARO) | Managed OpenShift cluster |
| Azure Machine Learning | RHODS + KServe/Seldon | ML platform and model serving |
| Azure Blob Storage | Azure Blob (hybrid) or ODF/Minio | Recommend keeping Azure Blob |
| Cosmos DB | Azure PostgreSQL (managed) | Or in-cluster PostgreSQL via Operator |
| Azure SQL Database | Azure PostgreSQL/MySQL | Or in-cluster via Operator |
| Azure Redis Cache | Redis Operator or Azure Redis | Managed or self-hosted |
| Azure Service Bus | AMQ Streams (Kafka) | Red Hat managed Kafka |
| Event Grid | Knative Eventing | Event-driven architecture |
| Application Insights | Prometheus + Jaeger | Built-in monitoring |
| Log Analytics | EFK Stack (Elasticsearch-Fluentd-Kibana) | Built-in logging |
| Azure Monitor | Prometheus + Grafana | Built-in metrics |
| Azure AD | Azure AD as IdP + OpenShift OAuth | Integrated authentication |
| Azure Key Vault | External Secrets Operator | With Key Vault backend |
| Virtual Network | ARO VNet + NetworkPolicy | Network isolation |
| NSG | NetworkPolicy + EgressFirewall | Network security rules |

## Key Advantages of OpenShift on Azure

1. **Container-First**: All components run in containers for consistency
2. **Kubernetes-Native**: Leverage Kubernetes ecosystem and tooling
3. **Built-in CI/CD**: Tekton-based pipelines integrated
4. **Operator Framework**: Automated application management
5. **Multi-Cloud**: Easier to migrate between clouds
6. **GitOps Ready**: Native support for GitOps workflows
7. **Developer Experience**: Web console, CLI, IDE integration
8. **Enterprise Support**: Red Hat enterprise support for OpenShift

## Key Considerations

1. **Hybrid Approach**: Use Azure managed services (Blob Storage, PostgreSQL) where cost-effective
2. **Networking**: Ensure proper network connectivity between ARO and Azure services
3. **Identity**: Configure Azure AD Workload Identity for pod authentication to Azure services
4. **Monitoring**: Combine OpenShift monitoring with Azure Monitor for comprehensive visibility
5. **Backup**: Use Velero for cluster-level backups
6. **Compliance**: Leverage OpenShift compliance operator for policy enforcement

## Recommended Architecture Pattern

**Best Practice**: Hybrid approach
- **Compute**: OpenShift (ARO) for all application workloads
- **Storage**: Azure Blob Storage for objects (cost-effective, durable)
- **Database**: Azure Database for PostgreSQL (managed, HA)
- **Cache**: Redis on OpenShift (low latency) or Azure Redis (managed)
- **Message Queue**: AMQ Streams on OpenShift (integrated)
- **Monitoring**: OpenShift built-in + Azure Monitor for Azure services
- **Identity**: Azure AD as OpenShift IdP + Workload Identity for Azure access

This hybrid approach provides the best balance of:
- **Performance**: In-cluster processing and messaging
- **Cost**: Managed Azure services for storage and database
- **Operational Simplicity**: Reduce operational overhead
- **Scalability**: Leverage both OpenShift and Azure scaling capabilities
