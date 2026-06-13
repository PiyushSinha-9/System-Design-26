# Senior / L5 System Design Roadmap

This is a complete ordered roadmap for Senior Software Engineer / L5 system design interviews at companies like Google, Uber, Databricks, Meta, Amazon, Coinbase, Rippling, Salesforce, Microsoft, and similar backend or infrastructure-heavy companies.

## How to use this roadmap

Do not treat this as a reading list only. For Senior system design, the bar is not "I know these topics." The bar is "I can drive an ambiguous design discussion, make tradeoffs, defend correctness, and explain how the system fails and recovers."

Recommended usage:

1. Learn the modules in order.
2. After Module 8, start doing full 45-minute mock designs.
3. For every design, produce:
   - Functional requirements
   - Non-functional requirements
   - Back-of-the-envelope estimates
   - APIs
   - Data model
   - High-level architecture
   - Read path and write path
   - Bottlenecks
   - Failure modes
   - Monitoring and alerting
   - Scale evolution
   - Tradeoffs

---

# Module 0: How Senior / L5 Interviews Are Judged

## 0.1 Senior interview behavior

You must show that you can drive the design.

You should be able to:

- Ask the right clarification questions early
- Set boundaries and scope
- Propose a clear high-level architecture quickly
- Iterate with tradeoffs
- Own major components end to end
- Make decisions using latency, availability, correctness, cost, and operational complexity
- Explain how the system fails, how you detect it, and how you recover it
- Avoid overengineering unless scale or requirements justify it
- Communicate clearly without jumping randomly between topics

## 0.2 Required interview artifacts

In almost every system design interview, you should produce:

- Requirements list
- Non-functional requirements
- Back-of-the-envelope estimates
- High-level architecture diagram
- API design
- Data model
- Read path
- Write path
- Partitioning strategy
- Caching strategy
- Failure modes
- Monitoring plan
- Scale evolution plan

## 0.3 Senior signals

A Senior candidate should naturally discuss:

- Correctness
- Reliability
- Availability
- Scalability
- Cost
- Latency
- Operability
- Security
- Data consistency
- Tradeoffs
- Migration path
- Backward compatibility

---

# Module 1: Interview Framework and Estimation

## 1.1 System design framework

Use a repeatable structure like PEDALS:

- Process: requirements, users, constraints
- Estimation: QPS, storage, bandwidth, peak traffic
- Design: components and flows
- Architecture: storage, compute, cache, queue, messaging
- List: bottlenecks, risks, tradeoffs
- Scale: how the design evolves

## 1.2 Requirements gathering

Always split requirements into:

### Functional requirements

Examples:

- Users can create posts
- Users can upload files
- Drivers send location updates
- Users can search documents
- Merchants can receive payments

### Non-functional requirements

Examples:

- p99 latency under 200 ms for reads
- 99.99 percent availability
- Strong consistency for payments
- Eventual consistency acceptable for feeds
- Data retention for 1 year
- Multi-region disaster recovery
- Low cost at high scale

## 1.3 Estimation mechanics

You should be comfortable estimating:

- DAU and MAU
- Peak traffic factor
- Read/write ratio
- QPS and peak QPS
- Payload size
- Storage per day
- Retention storage
- Bandwidth
- Cache memory requirement
- Number of shards
- Queue throughput
- Worker count
- Latency budget
- Cost intuition

## 1.4 Latency budget

Know how to split latency across components:

Example:

- API gateway: 10 ms
- Auth: 10 ms
- Cache lookup: 5 ms
- DB read: 30 ms
- Service processing: 20 ms
- Network overhead: 20 ms
- Total p99 target: 100 to 200 ms

---

# Module 2: Networking Basics

This module is missing from many system design plans, but it is very useful for backend, infra, cloud, and platform roles.

## 2.1 IP basics

Know:

- What is an IP address?
- IPv4 vs IPv6 at a basic level
- Public IP vs private IP
- Static IP vs dynamic IP
- Source IP and destination IP
- Port numbers
- Inbound traffic vs outbound traffic
- TCP connection basics
- Socket as IP plus port

Example:

```text
Client: 49.37.10.20:54321
Server: 20.42.100.10:443
```

## 2.2 Private IP ranges

Know common private ranges:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

These are commonly used inside VPCs, VNets, office networks, and Kubernetes clusters.

## 2.3 CIDR

You must understand CIDR properly.

Examples:

| CIDR | Approx IP count |
|---|---:|
| /32 | 1 |
| /30 | 4 |
| /29 | 8 |
| /28 | 16 |
| /27 | 32 |
| /26 | 64 |
| /25 | 128 |
| /24 | 256 |
| /20 | 4096 |
| /16 | 65536 |
| /8 | 16777216 |

Key intuition:

- Smaller suffix means larger range
- /16 is bigger than /24
- /24 is bigger than /28
- Bad CIDR planning causes IP exhaustion
- Kubernetes can consume many IPs quickly
- Private endpoints and internal load balancers also need IPs

## 2.4 Subnet

A subnet is a smaller IP range inside a bigger network.

Example:

```text
VNet: 10.0.0.0/16

App subnet: 10.0.1.0/24
Database subnet: 10.0.2.0/24
Private endpoint subnet: 10.0.3.0/24
Kubernetes node subnet: 10.0.4.0/22
```

Why subnets matter:

- Isolation
- Security boundaries
- Routing
- IP management
- Public vs private separation
- Kubernetes node and pod planning

## 2.5 VPC / VNet

AWS and GCP commonly use VPC. Azure uses VNet.

A VPC/VNet is your private network boundary inside the cloud.

It contains:

- CIDR range
- Subnets
- Route tables
- Security rules
- Gateways
- Private DNS
- Internal services

Interview phrasing:

```text
I would place application services inside a private VPC/VNet, expose only the edge layer publicly, and keep databases and storage private.
```

## 2.6 Route table

A route table decides where packets go.

Examples:

```text
Traffic to internet -> NAT Gateway
Traffic to database subnet -> local route
Traffic to on-prem network -> VPN or ExpressRoute
Traffic to another VPC/VNet -> peering
```

You should understand:

- Default route
- Local route
- Internet route
- Private route
- Custom route
- Blackhole route conceptually
- Route priority at a high level

## 2.7 Internet Gateway

Common in AWS-style terminology.

Purpose:

- Allows resources with public IPs to communicate with the internet
- Used for public subnets
- Not used for private DB subnets

## 2.8 NAT Gateway

A NAT Gateway allows private services to access the internet without exposing themselves publicly.

Example:

```text
Private service -> NAT Gateway -> Internet
```

Why it matters:

- Stable outbound public IP
- Hide private workloads
- Third-party API allowlisting
- Secure outbound connectivity
- Multi-cluster egress control

Senior-level explanation:

```text
Without NAT Gateway, outbound IPs may vary across nodes or clusters. With NAT Gateway, outbound traffic can use stable egress IPs that external partners can allowlist.
```

## 2.9 Load balancer networking

Know:

- Public load balancer
- Internal load balancer
- L4 load balancer
- L7 load balancer
- Health checks
- Backend pools
- Listener rules
- TLS termination
- Cross-zone load balancing

## 2.10 NSG / Security Group / Firewall

Azure uses NSG. AWS uses Security Group and NACL.

Know:

- Allow rules
- Deny rules
- Inbound rules
- Outbound rules
- Stateful vs stateless rules
- Least privilege
- Port-based restrictions

Example:

```text
Allow app subnet -> DB subnet on port 5432
Deny internet -> DB subnet
Allow load balancer -> app subnet on port 443
```

## 2.11 Private Endpoint / PrivateLink

Purpose:

- Access managed cloud services privately
- Avoid public internet exposure
- Disable public access for sensitive resources

Examples:

```text
App inside VNet -> Private Endpoint -> Storage Account
App inside VPC -> PrivateLink -> Managed database
```

Know:

- Private endpoint
- Private IP mapping
- Private DNS zone
- Public endpoint disablement
- Service Endpoint vs Private Endpoint in Azure at a high level

## 2.12 DNS fundamentals

Know:

- A record
- CNAME
- TTL
- Public DNS
- Private DNS
- Internal service discovery
- DNS failover
- Split-horizon DNS conceptually

Examples:

```text
api.company.com -> public load balancer
db.internal.company.com -> private database endpoint
```

## 2.13 TLS and certificates

Know:

- TLS handshake at a high level
- Certificate authority
- Certificate rotation
- TLS termination at load balancer
- End-to-end TLS
- mTLS at a high level

## 2.14 Networking failure modes

Be able to discuss:

- DNS misconfiguration
- DNS TTL delaying failover
- NAT Gateway failure or saturation
- Load balancer health check failure
- IP exhaustion
- Route table mistake
- Security group blocking traffic
- TLS certificate expiry
- Connection pool exhaustion
- Ephemeral port exhaustion

---

# Module 3: API Design, Product Surface, and Data Modeling

## 3.1 API design

Know:

- REST vs gRPC
- Resource modeling
- Endpoint design
- Request and response shape
- Pagination
- Cursor vs offset pagination
- Sorting
- Filtering
- API versioning
- Idempotency keys
- Error codes
- Retryable vs non-retryable errors
- Rate limit headers
- Client behavior

Example:

```http
POST /v1/payments
Idempotency-Key: abc-123

{
  "user_id": "u1",
  "amount": "100.00",
  "currency": "INR"
}
```

## 3.2 Data modeling

Know:

- Entities
- Relationships
- Access patterns
- Normalization
- Denormalization
- Partition key
- Sort key
- Secondary indexes
- Inverted indexes
- TTL
- Retention
- Archival
- Soft delete
- Hard delete
- Multi-tenant isolation

## 3.3 Storage choice

Know when to use:

- Relational database
- Key-value store
- Document database
- Wide-column store
- Object storage
- Search engine
- Time-series database
- Graph database
- Data warehouse
- Stream log

## 3.4 Precision modeling

Especially important for payments, accounting, trading, booking, and billing.

Know:

- Never use floating point for money
- Use integer minor units or decimal types
- BigInt and BigDecimal
- Rounding strategy
- Currency handling
- Double-entry ledger
- Immutable transaction records
- Reconciliation
- Audit trail

## 3.5 Event sourcing and CQRS

Know:

- Store state changes as events
- Rebuild state from events
- Separate write model from read model
- Materialized views
- Replay for debugging
- Event versioning
- Snapshotting

---

# Module 4: Core Building Blocks

## 4.1 Caching

Must master:

- Cache-aside
- Write-through
- Write-back
- Read-through
- TTL
- Jittered TTL
- Cache invalidation
- Negative caching
- Cache stampede prevention
- Request coalescing
- Distributed locks with timeout
- Local cache vs distributed cache
- LRU and LFU
- Cache consistency expectations
- Hot key handling

## 4.2 Load balancing and service routing

Know:

- L4 vs L7 load balancing
- Sticky sessions
- Stateless services
- API gateway
- Reverse proxy
- Service discovery
- Health checks
- Weighted routing
- Blue-green routing
- Canary routing
- Connection pooling
- Downstream protection

## 4.3 Async processing and background jobs

Know:

- When to go async
- Job queue
- Message queue
- Retries
- Exponential backoff
- Jitter
- DLQ
- Poison messages
- At-least-once delivery
- At-most-once delivery
- Effectively-once business behavior
- Idempotent consumers
- Deduplication keys
- Ordering guarantees
- Outbox pattern

## 4.4 Rate limiting

Know:

- Fixed window
- Sliding window log
- Sliding window counter
- Token bucket
- Leaky bucket
- Distributed rate limiting
- Per-user quota
- Per-IP quota
- Per-token quota
- Global quota
- Local quota batching
- Redis Lua script use case
- Failure behavior when limiter is unavailable

## 4.5 Distributed ID generation

Know:

- UUID
- Auto-increment ID
- Snowflake ID
- Ticket server
- Timestamp plus machine ID plus sequence
- ID ordering
- B-tree fragmentation with random UUIDs
- Clock skew issues

---

# Module 5: Databases and Storage

## 5.1 Practical database performance

Know:

- B-tree indexes
- Composite indexes
- Covering indexes
- Query access patterns
- Read replicas
- Replication lag
- Primary failover
- Backups
- Point-in-time recovery
- Restore testing
- Connection pooling
- Slow query debugging

## 5.2 Transaction isolation

Know at a high level:

- Read committed
- Repeatable read
- Serializable
- Phantom reads
- Lost updates
- Write skew
- Optimistic concurrency control
- Pessimistic locking
- Compare-and-set
- Conditional writes

## 5.3 Storage engine intuition

Know:

- B-tree
- LSM tree
- WAL
- Memtable
- SSTable
- Compaction
- Bloom filter
- Read amplification
- Write amplification
- Space amplification
- Sequential writes vs random writes

## 5.4 Sharding and partitioning

Know:

- Hash-based partitioning
- Range-based partitioning
- Directory-based partitioning
- Consistent hashing
- Virtual nodes
- Rebalancing
- Resharding
- Hot partition
- Hot key
- Salting
- Split hotspots
- Tenant-based partitioning

## 5.5 Data lifecycle

Know:

- TTL
- Archival
- Cold storage
- Hot storage
- Data retention
- Legal hold
- GDPR delete conceptually
- Backup retention
- Schema migration
- Backward compatibility

---

# Module 6: Distributed Systems Fundamentals

## 6.1 Replication and consistency

Know:

- Leader-follower replication
- Multi-leader replication
- Leaderless replication
- Quorum reads and writes
- Strong consistency
- Eventual consistency
- Read-your-writes
- Monotonic reads
- Causal consistency at a high level
- Conflict resolution
- Last-write-wins
- Version vectors conceptually

## 6.2 Distributed transactions

Know:

- Why 2PC can hurt availability
- Saga pattern
- Orchestration vs choreography
- Compensation
- Idempotency
- Exactly-once business invariants
- Avoiding double charge
- Avoiding double booking
- Avoiding duplicate notifications

## 6.3 Consensus

L5 depth only, not PhD depth.

Know:

- What consensus solves
- Leader election
- Replicated state machine
- Raft high-level mechanics
- Log replication
- Majority quorum
- When to use consensus
- Metadata management
- Membership
- Locks
- Strong ID allocation

## 6.4 Distributed locking

Know:

- Why locks are dangerous
- Leases
- Timeouts
- Fencing tokens
- Zombie process problem
- Redis lock limitations
- ZooKeeper / Chubby style locks conceptually

## 6.5 Conflict resolution for collaborative systems

Know at a high level:

- Operational Transformation
- CRDTs
- Vector clocks
- Causal ordering
- Centralized vs decentralized collaboration

This is useful for Google Docs, Figma-like systems, local-first apps, and collaborative editors.

---

# Module 7: Cloud Infrastructure and Kubernetes

This is especially important for your profile because your experience involves Azure, AKS, Event Hub, Storage, Key Vault, private networking, managed identities, and data pipelines.

## 7.1 Compute primitives

Know:

- VM
- VM scale set
- Container
- Serverless function
- Batch job
- Managed Kubernetes
- Bare metal at a high level
- Autoscaling

## 7.2 Kubernetes basics

Know:

- Pod
- Node
- Deployment
- ReplicaSet
- StatefulSet
- DaemonSet
- Job
- CronJob
- Service
- ClusterIP
- NodePort
- LoadBalancer
- Ingress
- ConfigMap
- Secret
- Namespace
- PersistentVolume
- PersistentVolumeClaim
- Horizontal Pod Autoscaler
- Pod disruption budget
- Readiness probe
- Liveness probe
- Startup probe

## 7.3 Kubernetes networking

Know:

- Pod IP
- Node IP
- Service IP
- ClusterIP
- Ingress controller
- Internal load balancer
- External load balancer
- Network policy
- CNI plugin at a high level
- NAT Gateway for outbound
- IP exhaustion in clusters
- Service discovery inside cluster

Interview phrasing:

```text
Pods are ephemeral, so clients should not call pod IPs directly. A Kubernetes Service gives a stable virtual endpoint. Ingress or a LoadBalancer exposes traffic depending on whether it is internal or external.
```

## 7.4 Kubernetes reliability

Know:

- Rolling updates
- Rollback
- Readiness gate
- Graceful shutdown
- Draining nodes
- Pod anti-affinity
- Resource requests
- Resource limits
- CPU throttling
- OOMKill
- CrashLoopBackOff
- HPA and VPA conceptually
- Cluster autoscaler
- Multi-zone node pools

## 7.5 Service mesh

Know at a high level:

- Sidecar proxy
- mTLS
- Traffic splitting
- Retries
- Circuit breaking
- Observability
- Istio / Envoy conceptually
- When service mesh is overkill

## 7.6 Cloud identity and access

Know:

- IAM
- Managed identity
- Service account
- Workload identity
- Role assignment
- Least privilege
- Secretless authentication
- Token rotation
- Key Vault / Secrets Manager / KMS

## 7.7 Cloud messaging and storage

Know practical use cases for:

- Object storage
- Blob storage
- Managed database
- Managed cache
- Event Hub / Kafka / PubSub / Kinesis
- Queue storage / SQS
- Data lake
- Data warehouse
- CDN
- Key management service

---

# Module 8: Reliability and Resilience

## 8.1 Dependency safety

Know:

- Timeouts
- Retries
- Retry budgets
- Exponential backoff
- Jitter
- Circuit breakers
- Bulkheads
- Backpressure
- Admission control
- Graceful degradation
- Fallbacks

## 8.2 Overload management

Know:

- Rate limiting
- Load shedding
- Priority queues
- Brownout strategy
- Queue length control
- Autoscaling lag
- Thundering herd
- Hotspot mitigation

## 8.3 High availability

Know:

- Single-region HA
- Multi-AZ deployment
- Leader failover
- Replica failover
- Active-passive
- Active-active
- DNS failover
- Anycast conceptually
- RPO
- RTO
- Disaster recovery drills

## 8.4 Failure-mode thinking

For every design, ask:

- What happens if DB is down?
- What happens if cache is down?
- What happens if queue is delayed?
- What happens if a region is down?
- What happens if downstream API is slow?
- What happens if a worker processes the same job twice?
- What happens if a request times out after the write succeeds?
- What happens if one tenant sends 100x traffic?

---

# Module 9: Observability and Operations

## 9.1 Observability essentials

Know:

- Logs
- Metrics
- Traces
- Golden signals
- Latency
- Traffic
- Errors
- Saturation
- Correlation IDs
- Structured logging
- Dashboards
- Alerting
- Paging vs ticket alerts
- Alert noise reduction

## 9.2 Tracing and sampling

Know:

- Distributed tracing
- Span
- Trace ID
- Head-based sampling
- Tail-based sampling
- Error sampling
- High-latency trace capture
- What you lose with sampling

## 9.3 SLOs and error budgets

Know:

- SLA
- SLO
- SLI
- Availability SLO
- Latency SLO
- Error budget
- Burn rate alerting
- Tradeoff between velocity and reliability

## 9.4 Incident response

Know:

- Detection
- Triage
- Mitigation
- Rollback
- Communication
- Postmortem
- Action items
- Runbooks
- On-call handoff
- Game days

## 9.5 Capacity planning

Know:

- Load testing
- Stress testing
- Soak testing
- Bottleneck identification
- Capacity buffer
- Traffic forecasting
- Cost-performance tradeoff

---

# Module 10: Security, Privacy, and Abuse Prevention

## 10.1 Authentication and authorization

Know:

- OAuth2
- OIDC
- JWT
- Token validation
- Key rotation
- API gateway auth enforcement
- RBAC
- ABAC
- Resource-level permissions
- Tenant isolation

## 10.2 Data security

Know:

- TLS
- Encryption at rest
- KMS
- Envelope encryption
- Secret management
- Secret rotation
- Audit logging
- Principle of least privilege
- Break-glass access

## 10.3 Privacy and compliance

Know at a high level:

- PII
- Data minimization
- Data retention
- Right to delete
- Data residency
- Audit trail
- Access logging
- Compliance-friendly design
- Tokenization
- Masking

## 10.4 Abuse prevention

Know:

- DDoS
- WAF
- Bot protection
- Fraud checks
- Velocity checks
- Device fingerprinting conceptually
- Spam prevention
- Signup abuse
- Credential stuffing
- Quotas and throttling

---

# Module 11: Deployment, Release, and Configuration

This is often missing from system design prep, but Senior candidates should mention it when relevant.

## 11.1 Release strategies

Know:

- Rolling deployment
- Blue-green deployment
- Canary deployment
- Shadow traffic
- Dark launch
- Feature flags
- Kill switches
- Rollback
- Forward fix

## 11.2 Configuration management

Know:

- Static config
- Dynamic config
- Feature flags
- Per-tenant config
- Config rollout
- Config validation
- Safe defaults
- Config audit trail

## 11.3 Schema evolution

Know:

- Backward-compatible schema changes
- Expand and contract migration
- Dual write
- Backfill
- Read old and new schema
- Versioned APIs
- Versioned events
- Protobuf/Avro compatibility at a high level

## 11.4 Data migration

Know:

- Online migration
- Offline migration
- Backfill jobs
- Verification
- Reconciliation
- Rollback strategy
- Traffic cutover
- Shadow reads
- Consistency checks

---

# Module 12: High-Performance and Real-Time Data Systems

## 12.1 Stream processing

Know:

- Tumbling window
- Sliding window
- Session window
- Watermark
- Late-arriving events
- Event time vs processing time
- Exactly-once processing conceptually
- Checkpointing
- Replay
- Lambda architecture
- Kappa architecture

## 12.2 Probabilistic data structures

Know:

- Bloom filter
- HyperLogLog
- Count-Min Sketch
- Top-K approximation
- Tradeoff between accuracy and memory

Use cases:

- Unique visitors
- Deduplication
- Frequency tracking
- Abuse detection
- Cache optimization

## 12.3 Time-series systems

Know:

- Metrics ingestion
- High-cardinality tags
- Downsampling
- Rollups
- Retention
- Compression
- Query latency
- Hot partitions
- Write-heavy workload

## 12.4 Search systems

Know:

- Inverted index
- Tokenization
- Ranking
- Autocomplete
- Prefix search
- N-grams
- Eventual indexing
- Reindexing
- Search relevance at a high level

---

# Module 13: Common Product Domain Designs

Practice at least one full system per category.

## 13.1 Feed and timeline

Know:

- Fanout-on-write
- Fanout-on-read
- Hybrid fanout
- Celebrity users
- Feed cache
- Ranking
- Freshness vs stability
- Pagination

Practice:

- Design Twitter feed
- Design Instagram feed
- Design LinkedIn feed

## 13.2 Chat and realtime messaging

Know:

- WebSockets
- SSE
- Long polling
- Message ordering
- Delivery receipts
- Read receipts
- Typing indicators
- Presence
- Offline delivery
- Push notifications
- Fanout

Practice:

- Design WhatsApp
- Design Slack
- Design Discord

## 13.3 Notification system

Know:

- Preferences
- Unsubscribe
- Quiet hours
- Batching
- Throttling
- Provider failover
- Retry
- DLQ
- Template rendering
- Priority

Practice:

- Design notification service
- Design email campaign system
- Design push notification system

## 13.4 File and media systems

Know:

- Multipart upload
- Resumable upload
- Pre-signed URL
- Metadata store
- Blob store
- CDN
- Virus scanning
- Moderation pipeline
- Transcoding
- Thumbnail generation
- Cache invalidation

Practice:

- Design Dropbox
- Design Google Drive
- Design YouTube upload pipeline
- Design Instagram media upload

## 13.5 Payments, booking, and ledger systems

Know:

- Idempotency
- Double-spend prevention
- Holds
- Expiration
- Reconciliation
- Ledger
- Balance
- Audit trail
- Chargeback
- Refund
- Settlement
- Exactly-once business effect

Practice:

- Design payment system
- Design wallet
- Design ticket booking
- Design hotel booking
- Design Coinbase wallet ledger

## 13.6 Geospatial and marketplace systems

Critical for Uber.

Know:

- Geohash
- QuadTree
- Google S2 at a high level
- K-nearest neighbors
- Find drivers within radius
- Real-time driver location updates
- Cell-based aggregation
- Surge pricing
- Dispatch
- Matching
- Hot region handling

Practice:

- Design Uber ride matching
- Design driver location system
- Design surge pricing
- Design food delivery dispatch

## 13.7 Search and discovery

Know:

- Query understanding
- Indexing pipeline
- Ranking signals
- Autocomplete
- Fuzzy search
- Reindexing
- Freshness
- Personalized ranking

Practice:

- Design Google search at high level
- Design product search
- Design log search
- Design autocomplete

## 13.8 Recommendation and ranking systems

Know at a high level:

- Candidate generation
- Ranking
- Features
- Online serving
- Offline training
- Feature store conceptually
- A/B testing
- Feedback loops
- Cold start
- Abuse and gaming

Practice:

- Design YouTube recommendations
- Design news feed ranking
- Design marketplace recommendations

---

# Module 14: Infrastructure Building Block Designs

These are very useful for Databricks, Google, Meta, Amazon, Microsoft, and infra-heavy teams.

## 14.1 Distributed message queue

Know:

- Append-only log
- Topic
- Partition
- Offset
- Consumer group
- Consumer rebalance
- Producer ack
- Replication
- Retention
- Compaction
- Pull vs push
- Backpressure
- Zero-copy conceptually
- Segment files and sparse index at a high level

Practice:

- Design Kafka
- Design PubSub
- Design durable task queue

## 14.2 Distributed key-value store

Know:

- Consistent hashing
- Virtual nodes
- Replication factor
- Quorum
- WAL
- Memtable
- SSTable
- Bloom filter
- Compaction
- Read repair
- Anti-entropy
- Hinted handoff conceptually

Practice:

- Design DynamoDB
- Design Cassandra
- Design Redis-like cache

## 14.3 Distributed lock manager

Know:

- Lease
- Fencing token
- Session
- Ephemeral node
- Heartbeat
- Leader election
- Split brain
- Zombie process

Practice:

- Design ZooKeeper-like lock service
- Design leader election service

## 14.4 Blob/object store

Know:

- Object metadata
- Blob storage
- Multipart upload
- Small file problem
- Packing small files
- Replication
- Erasure coding at a high level
- Checksums
- Read-after-write consistency
- Lifecycle policy

Practice:

- Design S3
- Design Google Cloud Storage
- Design photo storage backend

## 14.5 Distributed scheduler

Know:

- Job queue
- Cron
- Leader election
- Sharding jobs
- Worker heartbeats
- Retry
- Idempotency
- Timer wheel conceptually
- Dependency DAG
- Backfill
- Priority
- Fair scheduling

Practice:

- Design Airflow
- Design distributed cron
- Design Databricks job scheduler

## 14.6 Feature flag and experimentation platform

Know:

- Flag evaluation
- Targeting
- Percentage rollout
- Consistent hashing
- Kill switch
- Audit trail
- Experiment assignment
- Metric collection
- Guardrail metrics

Practice:

- Design LaunchDarkly
- Design A/B testing platform

---

# Module 15: Hardware Sympathy and I/O

This is useful for Databricks, infra, storage, and high-performance backend roles.

## 15.1 Disk I/O

Know:

- Sequential access
- Random access
- IOPS
- Throughput
- fsync
- Page cache
- Write amplification
- HDD vs SSD
- Log-structured storage

## 15.2 Memory

Know:

- Heap
- Stack
- Page cache
- mmap
- Off-heap memory
- GC pauses
- Object allocation overhead
- Buffer pooling

## 15.3 Network I/O

Know:

- Connection pooling
- Keep-alive
- HTTP/1.1 vs HTTP/2 at a high level
- gRPC multiplexing at a high level
- Head-of-line blocking conceptually
- Compression
- Payload size
- Serialization format
- Protobuf vs JSON

## 15.4 Performance debugging

Know:

- CPU bottleneck
- Memory bottleneck
- Disk bottleneck
- Network bottleneck
- Lock contention
- Thread pool starvation
- Queue buildup
- Tail latency
- p50, p95, p99, p999

---

# Module 16: Databricks / Data Platform Specific Depth

This is important if targeting Databricks or data infrastructure teams.

## 16.1 Data lake and warehouse basics

Know:

- Data lake
- Data warehouse
- Lakehouse conceptually
- Object storage as storage layer
- Metadata catalog
- Table format at a high level
- Partition pruning
- Compaction
- Small file problem
- Schema evolution

## 16.2 Distributed query execution

Know at a high level:

- Query planner
- Execution plan
- Stages
- Tasks
- Shuffle
- Broadcast join
- Sort merge join
- Predicate pushdown
- Columnar storage
- Parquet basics
- Vectorized execution conceptually

## 16.3 Job execution platform

Know:

- Driver
- Worker
- Cluster manager
- Scheduling
- Retries
- Speculative execution conceptually
- Checkpointing
- Job isolation
- Resource fairness
- Multi-tenant workloads

## 16.4 Streaming and ingestion

Know:

- Exactly-once sink conceptually
- Checkpoint
- Offset tracking
- Watermark
- Late data
- Replay
- Backfill
- Dead-letter handling
- Schema registry conceptually

Practice:

- Design data ingestion platform
- Design log analytics platform
- Design notebook execution platform
- Design distributed job scheduler
- Design metrics pipeline
- Design real-time fraud detection pipeline

---

# Module 17: Company-Specific Preparation

## 17.1 Google L5

Focus:

- Clean design
- Correct abstractions
- Scalability
- Reliability
- Data modeling
- Simplicity
- Tradeoffs
- Global systems

Practice:

- Design Google Docs
- Design YouTube
- Design Gmail
- Design autocomplete
- Design distributed key-value store
- Design job scheduler
- Design metrics system

## 17.2 Uber Senior

Focus:

- Geospatial systems
- Real-time location
- Matching
- Marketplace dynamics
- Low latency
- Regional partitioning
- Hotspot handling
- Reliability

Practice:

- Design Uber
- Design driver location service
- Design dispatch system
- Design surge pricing
- Design food delivery matching
- Design trip tracking

## 17.3 Databricks Senior

Focus:

- Distributed systems
- Storage
- Metadata
- Scheduling
- Data ingestion
- Query execution
- Streaming
- Multi-tenant infrastructure

Practice:

- Design distributed job scheduler
- Design notebook execution platform
- Design log ingestion pipeline
- Design Kafka-like system
- Design metadata catalog
- Design object-store-backed table system

## 17.4 Meta E5

Focus:

- Social graph
- Feed
- Ranking
- Messaging
- Media
- Realtime
- Large-scale caching

Practice:

- Design news feed
- Design Messenger
- Design Instagram media upload
- Design notification system
- Design story viewer counter
- Design comments system

## 17.5 Amazon L5 / L6

Focus:

- Retail systems
- Inventory
- Payments
- Ordering
- Fulfillment
- Reliability
- Operational excellence

Practice:

- Design order system
- Design inventory service
- Design payment system
- Design recommendation system
- Design distributed queue
- Design rate limiter

## 17.6 Coinbase / fintech

Focus:

- Correctness
- Ledger
- Idempotency
- Reconciliation
- Auditability
- Security
- Compliance

Practice:

- Design wallet
- Design exchange order book at a high level
- Design payment processing
- Design ledger
- Design fraud detection
- Design transaction history

---

# Module 18: Full Mock Design List

Do at least 15 full mocks before serious Senior interviews.

## Must-do mocks

1. Design URL shortener
2. Design rate limiter
3. Design notification system
4. Design chat system
5. Design news feed
6. Design file storage system
7. Design video upload and streaming
8. Design search autocomplete
9. Design payment system
10. Design booking system
11. Design Uber ride matching
12. Design distributed job scheduler
13. Design Kafka
14. Design distributed key-value store
15. Design metrics and logging platform
16. Design feature flag system
17. Design object storage
18. Design real-time location tracking
19. Design data ingestion pipeline
20. Design notebook execution platform

## For each mock, answer these questions

- What are the core requirements?
- What is explicitly out of scope?
- What is the expected traffic?
- What are the APIs?
- What is the data model?
- What is the read path?
- What is the write path?
- What should be cached?
- What should be async?
- What consistency is required?
- How do we shard?
- What are the failure modes?
- How do we monitor?
- How do we deploy safely?
- What breaks first at 10x scale?
- What breaks first at 100x scale?
- What tradeoff did we make?

---

# Module 19: Final Senior / L5 Checklist

Before an interview, you should be able to do the following without notes.

## Design execution

- Drive a 45-minute design discussion
- Clarify requirements quickly
- Define non-functional requirements
- Estimate scale
- Create APIs
- Create data model
- Draw architecture
- Explain read and write flows
- Identify bottlenecks
- Discuss failure modes
- Discuss observability
- Discuss tradeoffs
- Propose evolution plan

## Core concepts

- Caching
- Load balancing
- API gateway
- Queue
- Pub/sub
- Database indexes
- Sharding
- Replication
- Consistency
- Transactions
- Idempotency
- Rate limiting
- Distributed locks
- Object storage
- CDN
- Search indexing
- Stream processing

## Cloud and infra

- IP
- CIDR
- VPC/VNet
- Subnet
- Route table
- NAT Gateway
- Internet Gateway
- NSG/Security Group
- Firewall
- Private Endpoint/PrivateLink
- DNS
- TLS
- Kubernetes
- Ingress
- Service
- Pod
- Node
- HPA
- Rolling deployment
- Canary deployment
- Managed identity/IAM

## Reliability

- Timeout
- Retry
- Backoff
- Jitter
- Circuit breaker
- Bulkhead
- Backpressure
- Load shedding
- Graceful degradation
- RPO/RTO
- Multi-AZ
- Multi-region
- Disaster recovery

## Operability

- Logs
- Metrics
- Traces
- Correlation ID
- Dashboards
- Alerts
- SLO
- Error budget
- Incident response
- Rollback
- Postmortem

## Security

- OAuth2
- OIDC
- JWT
- RBAC
- ABAC
- TLS
- Encryption at rest
- KMS
- Secret rotation
- Audit logs
- Tenant isolation
- Abuse prevention

---

# Module 20: What Is Overkill for Most L5 Interviews

Learn these only after the core modules and mocks are strong:

- Paxos deep proof
- Zab protocol deep mechanics
- Raft implementation details
- CRDT implementation details
- Erasure coding math
- Query optimizer internals in depth
- Kernel networking internals
- Deterministic simulation
- Consensus safety and liveness proofs
- Deep compiler/runtime internals

These can impress in niche interviews, but they should not replace common system design practice.

---

# Final Prep Strategy

## Phase 1: Foundation

Study Modules 0 to 8 first.

Goal:

- You can design common backend systems clearly.
- You understand networking, storage, caching, queues, reliability, and cloud infra.

## Phase 2: Senior muscle

Study Modules 9 to 14.

Goal:

- You can discuss observability, security, rollout, data systems, and infra building blocks.
- You can explain failure modes and tradeoffs naturally.

## Phase 3: Company-specific depth

Study Modules 15 to 17 based on target company.

Goal:

- Uber: geospatial and marketplace systems.
- Databricks: storage, metadata, scheduling, streaming, query execution.
- Google: clean abstractions, global scale, reliability.
- Meta: feed, media, chat, ranking.
- Coinbase: correctness, ledger, security.

## Phase 4: Mock practice

Complete 15 to 20 full mock designs.

Minimum bar before real interviews:

- 5 generic product designs
- 5 infra designs
- 3 company-specific designs
- 2 payment or correctness-heavy designs
- 2 realtime or geospatial designs

---

# Final Verdict

This roadmap is enough for Senior / L5 system design if you use it correctly.

The real deciding factor is not whether you have more topics. The deciding factor is whether you can:

- Take an ambiguous prompt
- Ask smart questions
- Create a simple design
- Deep dive into the right component
- Explain correctness and failure modes
- Make practical tradeoffs
- Communicate like someone who has owned production systems

Once you can do that consistently, this syllabus is enough for Google L5, Uber Senior, Databricks Senior, and similar roles.
