# Wildlife Photography Workflow - Architecture Documentation

This directory contains comprehensive architecture documentation for deploying a Wildlife Photography Workflow system on Microsoft Azure, with both Azure-native and Red Hat OpenShift on Azure approaches.

## 📁 Documentation Structure

### Core Architecture Documents

1. **[WildlifePhotographyWorkflow_Architecture.md](./WildlifePhotographyWorkflow_Architecture.md)**
   - Complete Azure-native architecture using PaaS services
   - Components: Static Web Apps, API Management, App Service, Blob Storage, Cosmos DB, Service Bus
   - Traditional cloud-native approach with Azure managed services

2. **[WildlifePhotographyWorkflow_OpenShift_Architecture.md](./WildlifePhotographyWorkflow_OpenShift_Architecture.md)**
   - Equivalent architecture for Red Hat OpenShift on Azure (ARO)
   - Detailed component mapping from Azure services to OpenShift equivalents
   - Hybrid approach leveraging both OpenShift and Azure managed services
   - Includes YAML examples, operator recommendations, and best practices

3. **[Architecture_Comparison_Summary.md](./Architecture_Comparison_Summary.md)**
   - Side-by-side comparison of both architectures
   - Migration complexity assessment
   - Cost comparison and optimization strategies
   - Recommended migration path and success criteria
   - Quick reference tables for decision-making

4. **[Architecture_Diagrams.md](./Architecture_Diagrams.md)**
   - Visual diagrams using Mermaid
   - Architecture flows for both platforms
   - Network topology for OpenShift on Azure
   - Data flow sequence diagrams
   - Technology stack visualization

### Supporting Documentation

5. **[.github/copilot-instructions.md](./.github/copilot-instructions.md)**
   - GitHub Copilot coding standards for this project
   - Architecture documentation conventions
   - Preferred technologies and patterns
   - Code generation guidelines

## 🎯 Quick Start Guide

### Understanding the Architectures

**If you're starting from scratch:**
1. Read [WildlifePhotographyWorkflow_Architecture.md](./WildlifePhotographyWorkflow_Architecture.md) to understand the baseline architecture
2. Review [Architecture_Diagrams.md](./Architecture_Diagrams.md) for visual representation
3. Read [WildlifePhotographyWorkflow_OpenShift_Architecture.md](./WildlifePhotographyWorkflow_OpenShift_Architecture.md) for the OpenShift equivalent

**If you're migrating from Azure to OpenShift:**
1. Start with [Architecture_Comparison_Summary.md](./Architecture_Comparison_Summary.md) for the migration overview
2. Review the migration mapping table in [WildlifePhotographyWorkflow_OpenShift_Architecture.md](./WildlifePhotographyWorkflow_OpenShift_Architecture.md)
3. Follow the recommended migration path in the comparison document

**If you need specific information:**
- **Component mapping**: See "Migration Mapping Table" in OpenShift architecture doc
- **Cost analysis**: See "Cost Comparison" in comparison summary
- **Network design**: See "Network Architecture" diagram in diagrams doc
- **Data flow**: See sequence diagrams in diagrams doc

## 📊 Architecture Overview

### Azure Native Architecture
A traditional cloud-native architecture using Azure PaaS services:
- **Frontend**: Azure Static Web Apps
- **API**: Azure API Management
- **Compute**: Azure App Service, Functions, Container Instances
- **Data**: Azure Blob Storage, Cosmos DB, Redis Cache
- **Messaging**: Azure Service Bus, Event Grid
- **Monitoring**: Application Insights, Azure Monitor

### OpenShift on Azure Architecture
A container-first architecture using Red Hat OpenShift with selective Azure services:
- **Frontend**: OpenShift Deployment + Route
- **API**: Red Hat 3scale API Management or Kong
- **Compute**: OpenShift Deployments, Jobs, Knative
- **Data**: Azure Blob Storage (hybrid), Azure PostgreSQL (hybrid), Redis Operator
- **Messaging**: AMQ Streams (Kafka), Knative Eventing
- **Monitoring**: Prometheus, Grafana, Jaeger, EFK Stack

## 🔄 Key Transformation Mapping

| Azure Service | OpenShift Equivalent |
|---------------|---------------------|
| Azure Static Web Apps | Deployment + Service + Route |
| Azure API Management | 3scale API Management |
| Azure App Service | Deployment with HPA |
| Azure Functions | Knative Serving |
| Azure Container Instances | Job or Pod |
| Azure Blob Storage | **Keep Azure Blob** (hybrid) |
| Cosmos DB | Azure PostgreSQL (managed) |
| Azure Service Bus | AMQ Streams (Kafka) |
| Application Insights | Prometheus + Jaeger |

## 💡 Recommended Approach: Hybrid Architecture

The recommended approach is a **hybrid architecture** that combines:

✅ **OpenShift for Compute**
- All application workloads containerized
- Kubernetes-native orchestration
- Built-in CI/CD with OpenShift Pipelines
- Multi-cloud portability

✅ **Azure for Data**
- Azure Blob Storage for objects (cost-effective)
- Azure Database for PostgreSQL (managed, HA)
- Azure Cache for Redis (optional, managed)
- Azure Key Vault for secrets

### Benefits
1. **Cost-effective**: Leverage Azure managed services where cheaper
2. **Reduced complexity**: Less infrastructure to manage
3. **Best performance**: In-cluster processing, cloud storage
4. **Flexibility**: Easy to adjust based on needs
5. **Portability**: Can migrate compute to other clouds if needed

## 📋 Use Cases

### Workflow Description
The Wildlife Photography Workflow is designed to:

1. **Image Upload**: Photographers upload wildlife images through a web interface
2. **Processing**: Images are automatically processed (resize, thumbnails, watermarks)
3. **AI Detection**: Machine learning models detect animals and classify species
4. **Gallery**: Processed images are displayed in a searchable gallery
5. **Notifications**: Users receive notifications when processing is complete

### Technical Requirements
- Handle high-resolution image uploads (10MB+ per image)
- Process images within 30 seconds
- AI classification with 90%+ accuracy
- Support for 1000+ concurrent users
- 99.9% uptime SLA
- Global content delivery

## 🏗️ Component Deep Dive

### Frontend Layer
- **Azure**: Static Web Apps with built-in CDN
- **OpenShift**: Containerized React/Next.js with Nginx, exposed via Route
- **Common**: OAuth 2.0 authentication, responsive design

### API Gateway
- **Azure**: API Management for routing, rate limiting, auth
- **OpenShift**: 3scale API Management or Kong Gateway
- **Common**: JWT validation, API versioning, TLS termination

### Application Services

**Upload Service**
- Validates uploads (type, size, malware scan)
- Extracts EXIF metadata
- Stores raw images in object storage
- Publishes upload events to message queue

**Processing Service**
- Consumes upload events
- Generates multiple resolutions
- Creates thumbnails
- Applies watermarks
- Stores processed images

**AI Detection Service**
- Consumes processing events
- Runs TensorFlow/PyTorch models
- Detects animals in images
- Classifies species
- Stores results in database

### Data Layer
- **Object Storage**: Blob storage for raw, processed, and thumbnail images
- **Metadata Database**: PostgreSQL for image metadata, tags, user info
- **Cache**: Redis for sessions, API responses, popular images

### Messaging
- **Event-driven**: Asynchronous processing via message queues
- **Decoupling**: Services communicate through events
- **Scalability**: Queue-based scaling and backpressure handling

## 🔐 Security Considerations

### Both Architectures Include:
- Azure AD integration for authentication
- Secrets stored in Azure Key Vault
- TLS/HTTPS for all communications
- Network isolation (VNet, NSG, NetworkPolicy)
- RBAC for access control
- Container image scanning
- Vulnerability management

### OpenShift Additional Security:
- Built-in container security (SCCs)
- Pod security policies
- Network policies for pod-to-pod communication
- Service mesh for mTLS between services
- Compliance operator for policy enforcement

## 📈 Scalability & Performance

### Auto-scaling
- **Azure**: App Service auto-scale, Function scaling
- **OpenShift**: Horizontal Pod Autoscaler (HPA), Cluster Autoscaler

### High Availability
- **Azure**: Multi-region deployment, Traffic Manager
- **OpenShift**: Multi-zone deployment, pod anti-affinity

### Performance Optimization
- **Caching**: Redis for frequently accessed data
- **CDN**: Global content delivery for images
- **Database**: Read replicas, connection pooling
- **Queue**: Parallel processing, batch operations

## 💰 Cost Optimization

### Azure Native
- Use cool/archive storage tiers for older images
- Reserved instances for always-on services
- Auto-shutdown for dev/test environments
- Right-size resources based on metrics

### OpenShift on Azure
- Cluster auto-scaling to reduce node count during off-peak
- Use Azure Spot VMs for non-critical workloads
- Resource quotas to prevent over-provisioning
- Hybrid approach: managed services where cost-effective

## 🚀 Migration Strategy

### Phase 1: Preparation (Weeks 1-2)
- [ ] Review current Azure architecture
- [ ] Assess containerization requirements
- [ ] Plan network connectivity
- [ ] Design OpenShift project structure

### Phase 2: Infrastructure (Weeks 3-4)
- [ ] Provision ARO cluster
- [ ] Configure Azure AD integration
- [ ] Set up VNet connectivity
- [ ] Deploy monitoring stack

### Phase 3: Application Migration (Weeks 5-8)
- [ ] Containerize applications
- [ ] Deploy to OpenShift dev environment
- [ ] Migrate databases (or configure connectivity)
- [ ] Deploy messaging infrastructure
- [ ] Set up CI/CD pipelines

### Phase 4: Testing & Validation (Weeks 9-10)
- [ ] Functional testing
- [ ] Performance testing
- [ ] Security testing
- [ ] Load testing
- [ ] Disaster recovery testing

### Phase 5: Production Cutover (Weeks 11-12)
- [ ] Run parallel environments
- [ ] Gradual traffic migration
- [ ] Monitor performance
- [ ] Complete cutover
- [ ] Decommission old infrastructure

## 📚 Additional Resources

### Azure Documentation
- [Azure Red Hat OpenShift](https://learn.microsoft.com/azure/openshift/)
- [Azure Blob Storage](https://learn.microsoft.com/azure/storage/blobs/)
- [Azure Database for PostgreSQL](https://learn.microsoft.com/azure/postgresql/)

### OpenShift Documentation
- [Red Hat OpenShift Documentation](https://docs.openshift.com/)
- [OpenShift on Azure Best Practices](https://docs.openshift.com/aro/4/welcome/index.html)
- [OpenShift Operators](https://docs.openshift.com/container-platform/4.12/operators/index.html)

### Technology-Specific
- [3scale API Management](https://www.3scale.net/api-management/openshift/)
- [AMQ Streams (Kafka)](https://access.redhat.com/products/red-hat-amq)
- [Knative](https://knative.dev/)
- [KServe](https://kserve.github.io/website/)

## 🤝 Contributing

When updating these architecture documents:

1. **Maintain consistency** between all documents
2. **Update diagrams** when making architectural changes
3. **Follow the conventions** in copilot-instructions.md
4. **Provide examples** for complex concepts
5. **Explain trade-offs** for architectural decisions

## ✅ Validation Checklist

Before considering the architecture complete:

- [ ] All Azure services have OpenShift equivalents documented
- [ ] Migration complexity is assessed for each component
- [ ] Cost implications are analyzed
- [ ] Security controls are defined
- [ ] Monitoring and observability strategy is clear
- [ ] High availability approach is documented
- [ ] Disaster recovery plan is outlined
- [ ] Performance requirements are addressed
- [ ] Scalability strategy is defined
- [ ] Network architecture is documented

## 📞 Support & Questions

For questions about:
- **Azure services**: Refer to Azure documentation
- **OpenShift**: Refer to Red Hat OpenShift documentation
- **Architecture decisions**: See comparison summary and architecture docs
- **Implementation details**: See YAML examples in OpenShift architecture doc

---

**Document Version**: 1.0  
**Last Updated**: 2026-01-12  
**Authors**: Cloud Architecture Team  
**Status**: Complete - Ready for Review
