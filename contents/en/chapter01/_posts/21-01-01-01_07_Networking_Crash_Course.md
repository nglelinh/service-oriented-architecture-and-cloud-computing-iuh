---
layout: post
title: 01-07 Cloud Networking Crash Course
chapter: '01'
order: 8
owner: Nguyen Le Linh
lang: en
categories:
- chapter01
lesson_type: optional
---

NIST *on-demand network access* is easy to treat as a slogan until a packet has to reach a container and *only* that container. This optional lesson is a short, IUH-oriented map of the five objects you will keep meeting from Chapter 08 onward: **VPC, subnet, security group, DNS, and TLS**. It does not replace provider console tutorials. It gives you language so later Docker, Kubernetes, and Terraform notes are not a pile of unexplained acronyms.

Read this after the required Chapter 01 lessons, and again before you start the [Deploy Track labs]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}).

## Learning objectives

- Sketch an isolated virtual network with public and private subnets and say what each subnet is *for*.
- Distinguish a **security group** (stateful allow-list on an instance or NIC) from a route table (where packets go next).
- Explain why **DNS** and **TLS** are application-facing contracts, not optional decorations.
- Map the same design onto AWS, Azure, and GCP names without memorizing every product SKU.

## 1. Why “the network” is a first-class cloud object

On a laptop, `localhost` hides addressing, routing, and name resolution. In a public cloud, every resource sits in a **virtual private cloud** (VPC): a software-defined network you own, with a CIDR block you chose (for example $$10.0.0.0/16$$). Other customers cannot see your private IPs. You still share the provider’s physical fabric—that is NIST *resource pooling*—but your routing and firewall policy are yours.

A useful IUH picture: the **IUH Course Board** API should accept HTTPS from students on the internet, talk to a database that is *not* on the internet, and refuse every other inbound path. That is already a VPC design, not a Docker flag.

```text
Internet
   │
   ▼
[Internet gateway] ── public subnet ── load balancer / reverse proxy (80/443)
                           │
                     [NAT, if used]
                           │
                     private subnet ── API + workers
                           │
                     private subnet ── database (no public IP)
```

**Design question.** If the database has a public IP “just for this week’s demo,” you do not have a private subnet. You have a time bomb.

## 2. Subnets, routes, and “public” versus “private”

A **subnet** is a slice of the VPC CIDR (for example $$10.0.1.0/24$$) bound to one availability zone. “Public” and “private” are not magic types in the CIDR. They are *routing facts*:

| Kind | Typical route | Who holds a public IP |
| --- | --- | --- |
| Public | default route $$0.0.0.0/0$$ → internet gateway | yes (or a public load balancer in front) |
| Private | default route → NAT gateway / NAT instance, or *no* default route | no |

A **route table** answers “where does this packet go next?” A **security group** answers “is this packet allowed at this NIC?” Students mix them up because both can “block the internet.” They block it at different layers.

Three practical rules for course work:

1. Put **stateful stores** in private subnets. Never give PostgreSQL, Redis, or MinIO a public `0.0.0.0/0` listen “to save time.”
2. Put **ingress** (load balancer, or a single reverse proxy) in a public subnet.
3. If a private subnet needs outbound package downloads, that is what NAT is for—and NAT is a **billable** meter. Local Docker labs should not need a cloud NAT at all.

Azure calls the VPC a **Virtual Network (VNet)**. GCP calls it a **VPC network** and often uses **regional subnets**. The diagram above still holds.

## 3. Security groups are allow-lists, not “the firewall product”

A **security group (SG)** is a *stateful* set of allow rules attached to an instance, ENI, or (on some platforms) a network interface group. Stateful means: if you allow inbound TCP 443, the reply packets are allowed out without a matching outbound rule. Azure **Network Security Groups** and GCP **VPC firewall rules** play the same role with different defaults.

Write SG rules as *intent*, not as a dump of ports:

```text
sg-edge:    inbound 443 from 0.0.0.0/0 ; outbound to sg-api on 8080
sg-api:     inbound 8080 from sg-edge only
sg-db:      inbound 5432 from sg-api only
```

Referencing **another security group** is better than pasting the API’s current private IP. IPs change; the “API tier” identity should not.

<div class="content-box warning-box">
<p><strong>Default-open is a lab smell.</strong> <code>0.0.0.0/0</code> on SSH (22) or on the database port is the most common free-tier accident. Prefer no inbound SSH at all (provider console / SSM / IAP) or SSH from <em>your</em> IP for a few hours, then close it.</p>
</div>

Network ACLs (NACLs), if you meet them, are *stateless* subnet filters. For this course, get SG intent right first. Do not stack NACLs “for security” until you can explain a single TCP handshake through both.

## 4. DNS is how humans and services find the VIP

**DNS** maps a name to records (A/AAAA, CNAME, MX, TXT). In cloud estates you will see two planes:

- **Public DNS** — `courseboard.example.edu` → the load balancer. Students and browsers use this.
- **Private DNS** — `api.internal.courseboard` or `postgres.courseboard.svc` → addresses that should never need to exist on the public internet.

Kubernetes Service DNS (`*.svc.cluster.local`) is the same idea inside a cluster (Chapter 09). A cloud **private hosted zone** is the same idea for VMs and managed databases.

A tiny Python check you can run *locally* to see that resolution and connectivity are different questions:

```python
import socket

def resolve(name: str) -> str:
    return socket.getaddrinfo(name, 443, type=socket.SOCK_STREAM)[0][4][0]

# Resolution can succeed while the TCP path is firewalled.
print(resolve("example.com"))
```

**TTL** is a deployment detail: a five-second lab hostname and a 24-hour campus hostname do not roll back the same way. When you later put CI/CD in front of a DNS cut, say what the TTL is.

## 5. TLS is the contract on that name

**TLS** (the modern name for the protocol people still call “SSL”) authenticates the *server* to the client and encrypts the bytes. For course work you need four facts, not a cryptography course:

1. A **certificate** binds a **public key** to a **name** (SAN), signed by a CA the client already trusts.
2. Browsers match the **name they typed** to the certificate. `https://10.0.1.23` will fight you. Use a name.
3. **HTTP 80** is only a redirector in production designs. The real listener is **443**.
4. Private services still deserve TLS *or* a trusted network-plus-mesh identity (Chapter 09). “It is on a private IP” is not encryption.

Let’s Encrypt and provider-managed certificates (ACM, Azure-managed certs, Google-managed certs) exist so you do not run a hobby CA in production. For **local** labs, `mkcert` or a Compose-terminated Caddy/nginx with a lab certificate is enough. Do not commit private keys.

```python
import ssl
import socket

def peer_name(host: str) -> str:
    ctx = ssl.create_default_context()
    with socket.create_connection((host, 443), timeout=5) as raw:
        with ctx.wrap_socket(raw, server_hostname=host) as tls:
            return tls.getpeercert()["subject"]

# Lab check: the name you asked for is the name on the cert.
print(peer_name("example.com"))
```

<div class="content-box insight-box">
<p><strong>DNS + TLS are one user story.</strong> A perfect VPC with a self-signed certificate on a raw IP still looks “broken” to a classmate’s phone. Name the edge, then terminate TLS on that name.</p>
</div>

## 6. Same design, three vendor vocabularies

| Idea | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Isolated network | VPC | Virtual Network | VPC network |
| Slice + AZ | Subnet | Subnet | Subnet |
| Stateful allow-list | Security group | NSG | VPC firewall rule |
| Public path | IGW + public subnet | Public IP / LB on subnet | Cloud NAT / LB + subnet |
| Public name | Route 53 | Azure DNS | Cloud DNS |
| Managed cert | ACM | Key Vault / Front Door certs | Certificate Manager |

You do not need to deploy all three. You need to *see* that Chapter 12 provider charts are the same five objects with different product names.

## 7. A campus-sized worked example

**IUH Course Board** (the Deploy Track motif):

1. VPC $$10.20.0.0/16$$ in one region, two AZs if the instructor asks for a resilience sketch (two AZs cost more—see the FinOps lesson).
2. Public subnets hold only the load balancer.
3. Private subnets hold the API and the database.
4. `sg-db` accepts 5432 only from `sg-api`.
5. Public DNS `courseboard.<your-lab-domain>` → load balancer.
6. TLS certificate for that name; HTTP redirects to HTTPS.

Compose on a student laptop implements the *same tiers* with user-defined networks and published ports (Lab A). Cloud diagrams should not invent a fourth mystery box called “networking.”

## Challenges

- Overlapping CIDRs when someone later peers a second VPC or a campus VPN.
- NAT and load-balancer **hours** that outlive the demo.
- “Temporary” `0.0.0.0/0` rules that land in a screenshot and then in production.
- Certificates that expire on a Sunday because nobody owned renewal.

## Exercises

1. Draw the Course Board diagram with three security groups and *no* public IP on the database. Label the route that makes the public subnet public.
2. A classmate can `dig` your hostname but the browser hangs. List three distinct layers (DNS, SG, TLS) and what you would check at each.
3. Rewrite these bad rules as intent: `sg-api inbound 0.0.0.0/0 1-65535`, `sg-db inbound 0.0.0.0/0 5432`.
4. Using only the table in §6, give Azure and GCP names for “private subnet + SG-style allow-list + public DNS.”

## Where this goes next

- [08-03 Docker]({{ site.baseurl }}{% multilang_post_url contents/chapter08/21-01-01-08_03_Docker %}): user-defined bridge networks are a local VPC cartoon.
- [09-03 Services]({{ site.baseurl }}{% multilang_post_url contents/chapter09/21-01-01-09_03_Services_Deployments %}): ClusterIP + DNS inside the cluster.
- [13-04 CI/CD]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}): pipelines that must not punch `0.0.0.0/0` “to make deploy work.”
- [Deploy Track hub]({{ site.baseurl }}/contents/en/chapter15/)

## References

1. NIST SP 800-145, *The NIST Definition of Cloud Computing* (on-demand network access, resource pooling).
2. AWS VPC, Azure Virtual Network, and GCP VPC documentation (provider-neutral ideas above; product names in §6).
3. RFC 5280 (certificates) and RFC 8446 (TLS 1.3)—for the *name binding* idea, not for implementing a stack.
4. [infracourse.cloud](https://infracourse.cloud/) (Stanford CS 40, Winter 2024) — deployment-first inspiration for putting networking before “just run a VM.” IUH wording and labs are original.
