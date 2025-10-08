# Research Project: Load Balancing & Networking 

---

# Part A — Load Balancing

### 1) Comparison of Load Balancing Algorithms
**Round Robin**
- **How it works:** Requests are distributed sequentially in turn to each server.
- **Strengths:** Simple, easy to implement, works well for homogeneous backend pools.
- **Weaknesses:** Doesn’t account for server load or session length; can cause uneven distribution if servers have different capacities or requests have variable processing time.
- **Use cases:** Small to medium pools of similar servers, simple web apps, DNS round-robin.

**Least Connections**
- **How it works:** New requests go to the server with the fewest active connections.
- **Strengths:** Adapts to different request durations; better with variable workloads.
- **Weaknesses:** Requires accurate connection counting and health checks; may still be fooled by long-running but idle connections.
- **Use cases:** Applications with variable request times (e.g., long-polling, streaming, API backends).

**IP Hash (Source IP Hashing)**
- **How it works:** Uses a hash of the client's IP (or parts of it) to pick a backend.
- **Strengths:** Provides consistent routing (sticky behavior) without server-side session storage.
- **Weaknesses:** Poorer load distribution if client IPs are uneven; not suitable if a client’s IP changes (mobile users, proxies).
- **Use cases:** When session stickiness is required and you cannot rely on cookies or external session stores.

**Other variants & considerations**
- **Weighted Round Robin / Weighted Least Connections:** Assigns weights to servers to reflect capacity.
- **Least Response Time:** Routes to server with lowest average response time — good for latency-sensitive traffic.
- **Health checks & session persistence:** Any LB should pair the algorithm with active health checks and optional session affinity depending on app design.

---

### 2) High Availability with Load Balancing
**How load balancers contribute to HA**
- **Traffic distribution:** Prevents single-server overload by spreading requests.
- **Redundancy:** Multiple backend pools across availability zones or regions reduce single points of failure.
- **Automated failover:** Health checks detect unhealthy nodes and remove them from rotation; traffic is redirected to healthy nodes.

**Common HA strategies**
- **Active–Passive:** A standby LB takes over if the primary fails (simpler but may have failover time).
- **Active–Active:** Multiple LBs serve traffic simultaneously (better throughput and zero-downtime failures).
- **Global Server Load Balancing (GSLB):** Distributes traffic across regions/data centers using DNS + health checks for disaster recovery and geo-optimization.
- **Anycast:** Same IP announced from multiple locations—routes users to the nearest healthy site.

**Design tips**
- Put LBs in multiple availability zones and use health checks with sensible timeouts.
- Keep services stateless when possible, or use shared session stores (e.g., Redis) so failover is seamless.
- Combine LB HA with autoscaling groups and graceful shutdown hooks so backends leave the pool cleanly.

---

### 3) Scalability and Load Balancing
**Role of LBs in scalability**
- **Horizontal scaling:** LBs distribute traffic to newly provisioned instances automatically.
- **Autoscaling integration:** Metrics (CPU, queue length, request latency) trigger scaling; LBs discover and route to new instances.
- **Rate limiting & buffering:** LBs can protect backends during traffic spikes by rate-limiting or using queuing patterns.

**Patterns**
- **Stateless services:** Encourage horizontal scale; state lives in external stores.
- **Backpressure & circuit breakers:** Prevent cascading failures when backends are overwhelmed.
- **Blue/green & canary deployments:** Use LBs to direct subsets of traffic to new versions for safe rollouts.

---

### 4) Load Balancing in Cloud Environments
**Common cloud offerings (conceptual comparison)**
- **AWS** — Elastic Load Balancing family: ALB (Application Load Balancer, L7), NLB (Network Load Balancer, L4), CLB (Classic, legacy).
  - **ALB:** Path/host-based routing, HTTP/2 & WebSockets, TLS termination, good for microservices.
  - **NLB:** Low latency, preserves client IPs, ideal for TCP/UDP high-throughput services.
- **Azure** — Azure Load Balancer (L4), Application Gateway (L7 + WAF), Front Door (global, L7).
- **Google Cloud** — Cloud Load Balancing (global L7 and regional L4), supports HTTP(S) load balancing globally.

**Feature considerations**
- **Layer (L4 vs L7):** L7 enables routing by URL, headers, TLS offload; L4 is faster and simpler.
- **TLS termination / re-encryption:** Offload costly TLS to LB or pass-through for end-to-end encryption.
- **Integration:** How the LB works with autoscaling, container orchestration, and health checks.
- **Cost model:** Managed LBs are typically charged by throughput and hours/VMs; evaluate operational overhead vs price.

---

### 5) Security Implications of Load Balancers
**Security features LBs provide**
- **TLS termination / mutual TLS:** Offloads TLS and centralizes certificate management.
- **Web Application Firewalls (WAF):** Block common web attacks (XSS, SQLi) at the edge.
- **DDoS mitigation & rate limiting:** Detect and throttle abusive traffic; many cloud LBs integrate with DDoS protections.
- **IP filtering & access controls:** Allowlist/denylist traffic and integrate with identity-aware proxies.

**Operational security tips**
- Ensure proper header handling (`X-Forwarded-For`) to track client IPs correctly.
- Protect health endpoints (not publicly accessible or require auth) to avoid information leaks.
- Log LB access and TLS negotiation metrics for forensic analysis.

---

### 6) Container Orchestration and Load Balancing
**Kubernetes primitives**
- **Service types:** `ClusterIP` (internal), `NodePort` (exposes port on nodes), `LoadBalancer` (provisions cloud LB).
- **Ingress & IngressControllers:** L7 entry point with path/host routing, TLS termination, often backed by controllers like NGINX, Traefik.
- **Headless Services:** For stateful workloads or when client-side discovery is used.

**Service mesh & sidecars**
- Service meshes (Istio, Linkerd) provide L7 load balancing, retries, circuit-breaking, observability, and fine-grained routing rules via sidecar proxies.

**On-prem & metal solutions**
- Use MetalLB to provide `LoadBalancer` semantics in bare-metal Kubernetes.
- Use external LBs or DNS-based solutions where needed.

---

### 7) Load Balancing and Microservices
**Challenges**
- Large numbers of services with frequent scaling and churn.
- Service discovery and dynamic routing.
- Need for fine-grained security, observability, and consistent routing (circuit-breakers, retries).

**Best practices**
- Make services stateless where possible and store session/state in external stores.
- Use service discovery (DNS SRV, Consul, or Kubernetes Services) combined with client-side or server-side LB.
- Adopt a service mesh when you need centralized policies, mutual TLS, and advanced routing.
- Implement centralized metrics and distributed tracing (OpenTelemetry) to understand traffic flows.

---

# Part B — Networking

### 1) Overview of Network Protocols
**HTTP/HTTPS (Application layer)**
- **HTTP:** Stateless request/response protocol for web traffic.
- **HTTPS:** HTTP over TLS provides confidentiality and integrity.

**TCP (Transport)**
- **Connection-oriented:** Reliable, ordered byte stream (retransmission and congestion control). Used for HTTP, SSH, TLS.

**UDP (Transport)**
- **Connectionless:** Low-latency, no retransmission (good for DNS queries, video, VoIP, gaming).

**ICMP (Network diagnostics)**
- Used by `ping` and `traceroute` for network diagnostics and error reporting.

**Why this matters for DevOps**
- Protocol choice affects latency, reliability, and scalability decisions for services.

---

### 2) DNS Resolution and Load Balancing
**DNS basics**
- DNS resolves domain names to IPs; records (A/AAAA, CNAME) and TTL govern caching behavior.

**DNS-based load balancing techniques**
- **Round-robin DNS:** Multiple A records for same name.
- **Weighted DNS:** DNS returns addresses based on configured weights.
- **GeoDNS / latency-based routing:** Returns endpoints based on client location or observed latency.
- **DNS failover:** Using health checks to remove endpoints from DNS responses.

**Trade-offs**
- DNS caching and TTLs introduce propagation delays for changes.
- DNS alone has limited visibility into server health (requires integrated health checks).

---

### 3) Virtual Private Networks (VPNs)
**Types**
- **Site-to-site VPN:** Connects entire networks (data center ↔ cloud). Often uses IPsec or WireGuard.
- **Remote-access VPN:** Individual clients connect securely for admin or developer access.

**Technologies**
- **IPsec:** Widely used, mature, good for site-to-site.
- **SSL/TLS VPNs:** Easier for client access (OpenVPN, TLS-based solutions).
- **WireGuard:** Modern, simple configuration, high performance.

**Use in DevOps**
- Securely bridge on-prem infra and cloud, enable safe remote management and CI/CD access to private resources.

---

### 4) Network Security Best Practices
- **Segmentation:** Use subnets, security groups, and firewalls to limit blast radius.
- **Least privilege & RBAC:** Limit management plane access via roles and MFA.
- **Encryption:** TLS for in-transit data; encrypt sensitive data at rest.
- **Detect & respond:** IDS/IPS, log aggregation, and alerting.
- **Patch & inventory:** Keep devices updated and maintain an asset inventory (NetBox, CMDB).
- **Automation:** Apply compliance as code and automated security checks in pipelines.

---

### 5) IPv4 vs IPv6 Transition
**Why IPv6**
- Vast address space, simplified auto-configuration, better multicast support.

**Migration strategies**
- **Dual-stack:** Run IPv4 and IPv6 in parallel (most common).
- **Translation (NAT64/DNS64):** Allow IPv6-only clients to access IPv4 services.

**Challenges**
- Legacy hardware or middleboxes may not support IPv6.
- Skill gaps and testing complexity.

---

### 6) Network Monitoring and Management Tools
**Packet capture & deep inspection**
- **Wireshark / tcpdump:** Capture and analyze packets for root-cause diagnostics.

**Monitoring & alerting**
- **Prometheus + Grafana:** Metrics collection and dashboards.
- **Nagios / Zabbix / Icinga:** Traditional host/network monitoring and alerts.
- **Netflow/sFlow/IPFIX collectors:** Traffic analysis and flow-based monitoring.

**Best practices**
- Monitor latency, errors, throughput, connection counts, and resource usage.
- Centralize logs and metrics and use traces to correlate problems across layers.

---

### 7) Software-Defined Networking (SDN)
**What is SDN?**
- Decouples control plane (controller) from data plane (switches/routers) for programmatic network control.

**Benefits**
- Centralized policy, automation, dynamic traffic engineering, and easier multi-tenant isolation.

**Examples**
- **OpenFlow**, **Open vSwitch**, vendor solutions (Cisco ACI, VMware NSX).

**Challenges**
- Controller availability/security is critical; migration complexity and vendor lock-in concerns.

---

### 8) Cloud Networking
**Core components**
- **VPC / Virtual Network:** Isolated network space in the cloud.
- **Subnets:** Logical subdivisions (public/private).
- **Security Groups & NACLs:** Instance-level and subnet-level traffic filtering.
- **Peering & Transit Gateways:** Connect VPCs and on-prem networks.

**Best practices**
- Use private subnets for backends, place bastions in a restricted management subnet, and use VPC endpoints for private service access.

---

### 9) Network Automation and DevOps
**Tools & patterns**
- **Ansible & Nornir:** Agentless provisioning and orchestration for routers/switches/firewalls.
- **Terraform:** Declarative infrastructure as code for cloud networking resources.
- **NetBox:** Source-of-truth for network inventory and IPAM.
- **Batfish:** Network config analysis and testing in CI pipelines.

**Workflow ideas**
1. Store desired network state in version control (IaC + NetBox).
2. Run static analysis & unit tests (Batfish, linters).
3. Apply changes via CI with Ansible/Terraform and verify with automated checks.
4. Rollback via stored immutable artifacts if verification fails.

---



---



