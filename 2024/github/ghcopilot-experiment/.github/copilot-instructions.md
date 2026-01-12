# GitHub Copilot Instructions

## Project Overview
This project contains architecture documentation for a Wildlife Photography Workflow system, including both traditional Azure cloud deployments and Red Hat OpenShift on Azure deployments.

## Project Structure
```
2024/github/ghcopilot-experiment/
├── WildlifePhotographyWorkflow_Architecture.md
├── WildlifePhotographyWorkflow_OpenShift_Architecture.md
└── .github/
    └── copilot-instructions.md
```

## Architecture Documentation Standards

### When working with architecture documents:
1. **Cloud-Native Focus**: Prioritize containerized, microservices-based architectures
2. **Scalability**: All components should support horizontal scaling
3. **High Availability**: Design for multi-zone/multi-region deployments
4. **Security First**: Include identity, secrets management, and network security
5. **Observability**: Include comprehensive monitoring, logging, and tracing

### Azure Services Conventions
- Use Azure managed services where appropriate for reduced operational overhead
- Consider cost optimization strategies (storage tiers, reserved instances, auto-scaling)
- Implement proper network isolation using VNets and NSGs
- Use Azure AD for identity and access management

### OpenShift Conventions
- Leverage Operators for application lifecycle management
- Use OpenShift Routes for external access (not standard Ingress)
- Implement proper RBAC using Projects and Roles
- Use BuildConfigs and ImageStreams for CI/CD
- Prefer Operators from OperatorHub for common services
- Use Horizontal Pod Autoscalers (HPA) for auto-scaling

### Hybrid Architecture Principles
When creating hybrid architectures (OpenShift + Azure):
1. Use Azure managed services (PostgreSQL, Blob Storage) for data persistence
2. Run application workloads in OpenShift for portability
3. Configure Azure AD as OpenShift Identity Provider
4. Use Azure AD Workload Identity for pod-to-Azure service authentication
5. Implement network connectivity between ARO and Azure services

## Documentation Style
- Use Markdown with proper headings hierarchy
- Include component diagrams where beneficial
- Provide concrete examples (YAML manifests, configuration snippets)
- Create comparison tables for migration scenarios
- Include both "what" and "why" for architectural decisions

## Technologies to Reference

### Preferred Azure Services
- Azure Red Hat OpenShift (ARO)
- Azure Blob Storage
- Azure Database for PostgreSQL
- Azure Cache for Redis
- Azure Key Vault
- Azure Monitor
- Azure Application Insights
- Azure AD

### Preferred OpenShift Components
- OpenShift Operators (Strimzi, Crunchy PostgreSQL, etc.)
- OpenShift Pipelines (Tekton)
- OpenShift GitOps (ArgoCD)
- OpenShift Service Mesh (Istio)
- OpenShift Data Foundation (ODF)
- Red Hat 3scale API Management
- Red Hat AMQ Streams (Kafka)
- Knative Serving and Eventing

### Application Stack
- **Frontend**: React.js, Next.js, Vue.js
- **Backend**: Node.js, Python (FastAPI), Go
- **Databases**: PostgreSQL, MongoDB, Redis
- **Message Queues**: Kafka, RabbitMQ, ActiveMQ
- **ML/AI**: TensorFlow, PyTorch, scikit-learn

## Code Generation Guidelines

### For Kubernetes/OpenShift YAML:
```yaml
# Always include:
# - Resource requests and limits
# - Labels and selectors
# - Proper namespace references
# - Health checks (readiness/liveness probes)
# - Security context where applicable

apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
  namespace: example-namespace
  labels:
    app: example-app
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
        version: v1
    spec:
      containers:
      - name: app
        image: example/app:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### For Architecture Decisions:
- Explain trade-offs between options
- Provide cost considerations
- Include operational complexity assessment
- Reference industry best practices
- Consider compliance and security requirements

## Common Patterns

### Storage Pattern
- **Objects/Files**: Azure Blob Storage (with lifecycle policies)
- **Structured Data**: Azure Database for PostgreSQL/MySQL
- **Cache**: Redis (in-cluster or Azure Cache)
- **Temporary**: PersistentVolumeClaims with appropriate StorageClass

### Messaging Pattern
- **Event Streaming**: AMQ Streams (Kafka)
- **Message Queue**: AMQ Broker or RabbitMQ
- **Event-Driven**: Knative Eventing

### Scaling Pattern
- **Horizontal**: HPA based on metrics
- **Vertical**: VPA for resource optimization
- **Cluster**: Machine autoscaler for node scaling

### Security Pattern
- **Secrets**: External Secrets Operator + Azure Key Vault
- **Network**: NetworkPolicy + Service Mesh
- **Identity**: Azure AD + OpenShift OAuth
- **RBAC**: Projects, Roles, RoleBindings

## Environment-Specific Guidance

### Development
- Use OperatorHub operators for quick setup
- Single-replica deployments acceptable
- In-cluster databases for simplicity
- Relaxed resource limits

### Production
- Multi-replica for all stateless services
- Azure managed databases for data layer
- Strict resource quotas and limits
- Multi-zone deployment
- Comprehensive monitoring and alerting
- Automated backup and disaster recovery

## Conventions for AI/ML Workloads
- Use GPU-enabled node pools for training
- Implement model serving with KServe or Seldon
- Use Red Hat OpenShift Data Science (RHODS) for ML workflows
- Store models in S3-compatible storage (ODF or Azure Blob)
- Implement A/B testing for model deployments
- Monitor model drift and performance metrics

## Best Practices Checklist
When reviewing or generating architecture:
- [ ] All services have defined resource limits
- [ ] High availability is addressed
- [ ] Security components are included
- [ ] Monitoring and logging are specified
- [ ] Scalability strategy is clear
- [ ] Cost optimization is considered
- [ ] Backup and disaster recovery are defined
- [ ] Network security is implemented
- [ ] Secrets management is secure
- [ ] CI/CD pipeline is described
