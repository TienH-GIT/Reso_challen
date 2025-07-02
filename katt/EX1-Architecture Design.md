# ARCHITECTURE DESIGN PROPOSAL: TechMart Inc.

## Proposed Solution: 
### High-Level Architecture Design
To address the scalability and performance issues at TechMart, we propose migrating from a monolithic system to a modular microservices architecture, hosted on AWS with scalable, containerized services and layered caching.

```mermaid
graph TB
    subgraph "Client Side"
        Browser[User Browser]
    end
    subgraph "Frontend Layer"
        CDN[CloudFront CDN]
        ReactApp[React App on S3]
    end
    subgraph "API Gateway & Routing"
        APIGW[API Gateway / Load Balancer]
    end
    subgraph "Microservices"
        Auth[Auth Service]
        Catalog[Product Catalog Service]
        Cart[Cart Service]
        Checkout[Checkout Service]
        Orders[Order Management Service]
        Search[Search Service]
    end
    subgraph "Data Layer"
        RDS[PostgreSQL (RDS)]
        Redis[Redis Cache]
        S3[S3 Bucket (Static Assets, Logs)]
    end
    subgraph "Monitoring"
        CloudWatch[CloudWatch + APM]
    end

    Browser --> CDN --> ReactApp --> APIGW
    APIGW --> Auth
    APIGW --> Catalog
    APIGW --> Cart
    APIGW --> Checkout
    APIGW --> Orders
    APIGW --> Search
    Auth --> RDS
    Catalog --> Redis & RDS
    Cart --> Redis & RDS
    Checkout --> RDS
    Orders --> RDS
    Search --> Redis & RDS
    Redis --> RDS
    Microservices --> CloudWatch
```

Each core business domain (e.g., Cart, Catalog, Checkout) is implemented as an independent service deployed via Docker containers, managed through AWS ECS or EKS. Static assets and the React frontend are hosted on S3 and served via CloudFront. 
PostgreSQL on Amazon RDS serves as the primary data store, augmented by Redis for caching hot data and session state.

## Technology Choices: 

| Component            | Technology                                        | Justification                                             |
| -------------------- | ------------------------------------------------- | --------------------------------------------------------- |
| **Frontend Hosting** | AWS S3 + CloudFront                               | Cost-effective, global distribution, fast static delivery |
| **Backend Runtime**  | Node.js + Express                                 | Continuity with current team skills, async support        |
| **Microservices**    | Docker + AWS ECS/EKS                              | Isolated deployments, horizontal scaling                  |
| **Database**         | Amazon RDS (PostgreSQL)                           | Managed service, automated backups, read replicas         |
| **Caching Layer**    | Amazon ElastiCache (Redis)                        | Fast access to session data, product info, search         |
| **Monitoring**       | Amazon CloudWatch + APM (e.g., Datadog/New Relic) | Full observability of services and infrastructure         |
| **Search**           | OpenSearch (Amazon Elasticsearch)                 | Scalable and fast search capability                       |
| **CI/CD**            | GitHub Actions + CodePipeline                     | Automated deployments with rollback capability            |

This stack ensures cloud-native scalability while minimizing disruptions to developer workflow and minimizing retraining requirements.

## Scalability Strategy
### Horizontal Service Scaling:
- Services auto-scale via ECS/EKS based on CPU/memory or request count.
- Stateless design ensures multiple instances can handle requests concurrently.

### Data Layer Scaling:
- Read replicas for PostgreSQL handle query load from search and catalog services.
- Redis offloads frequent queries and enables fast access to ephemeral data.

### Load Balancing & Traffic Distribution:
- Application Load Balancer routes traffic to the appropriate service.
- API Gateway provides throttling, routing, and request transformation.

### Auto-scaling and Fault Tolerance:
- Each microservice can scale independently.
- Services deployed across multiple Availability Zones for high availability.

### Content Delivery:
- Static assets are served from CloudFront for faster global performance.


## Trade-offs Analysis
| Approach                              | Pros                                         | Cons                                                    |
| ------------------------------------- | -------------------------------------------- | ------------------------------------------------------- |
| **Monolithic (Current)**              | Simple deployment, low latency (local calls) | Hard to scale, single point of failure, deployment risk |
| **Microservices (Proposed)**          | Scalable, fault-tolerant, service isolation  | Added complexity, operational overhead                  |
| **ECS (Fargate)**                     | No infrastructure management, pay-per-use    | Slightly higher cost per instance vs EC2                |
| **EKS (Kubernetes)**                  | High flexibility, vendor-neutral             | Requires Kubernetes expertise, steeper learning curve   |
| **Redis Caching**                     | Reduces DB load, improves performance        | Cache invalidation needs to be managed carefully        |
| **PostgreSQL RDS with Read Replicas** | Managed, scalable reads                      | Writes still bottlenecked at primary unless sharded     |
| **API Gateway**                       | Secure, flexible routing, rate-limiting      | Higher latency than direct routing in some cases        |

By embracing a microservices-based cloud architecture, TechMart will gain the flexibility to scale, maintain, and evolve its platform in response to rapid growth, seasonal traffic spikes, and business expansion. 
The design emphasizes fault isolation, performance optimization, and observability — ensuring the system meets technical KPIs while enhancing user experience and supporting business growth.

