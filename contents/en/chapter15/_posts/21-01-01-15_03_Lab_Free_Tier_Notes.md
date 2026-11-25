---
layout: post
title: 15-03 Lab C — Free-tier notes and IUH cloud hygiene
chapter: '15'
order: 4
owner: Nguyen Le Linh
lang: en
categories:
- chapter15
lesson_type: optional
---

**Format:** reading + checklist.  
**Timebox:** 60–90 minutes if you stay on paper; longer if the instructor opens a real account.  
**Default:** you can finish the Deploy Track **without** a public-cloud account.

Free-tier product lists change. Treat every SKU below as **“verify on the provider page today.”** This page is IUH hygiene, not a coupon guide and not a CS 40 lab.

Every semester, at least one student learns a painful $45 lesson about cloud free tiers: they shut down their EC2 virtual machine, thinking billing stopped, only to receive a surprise invoice at the end of the month for an idle **NAT Gateway** ($0.045/hour = **$32.40/month**!) or an unattached **Elastic IP** ($3.60/month) they forgot was still allocated!

Lab C is your practical **IUH Cloud Hygiene & FinOps Survival Guide**: how to set hard **$1–$5 billing alarms** *before* launching a single resource, which sneaky "always-billing" cloud components sit outside free tiers, and how to lock down root accounts with MFA.

---

## Key ideas: Four Cloud FinOps & Account-Hygiene Fundamentals

1. **Free Tier ≠ Unlimited Free:** Free tiers have strict monthly ceilings (e.g., 750 hours of `t2.micro`/`t3.micro`, 20 GB storage) and **exclude** many networking resources like NAT Gateways, Application Load Balancers, and cross-AZ data transfer!
2. **Zero-Trust Account Hygiene:** Never use the AWS/GCP **root account** for daily labs—enable MFA on root immediately, create a least-privilege IAM user/role, and never commit access keys to GitHub (automated bots scrape leaked keys in under 60 seconds!).
3. **Proactive Billing Alarms (Budget Guardrails):** Setting an email alert at **$1.00** and **$5.00** catches runaway resources on Day 1 instead of Day 30.
4. **Teardown Verification Checklist:** After `tofu destroy` or manual cleanup, always verify the **Billing Dashboard** and **Active Regions** list to ensure zero orphaned disks (EBS) or IPs remain.

---

## Purpose

Know what “free” usually means, what it never meant, and the **order of operations** if you leave localhost.

## What free tier usually is

Providers mix several offers. Names differ; the *shapes* repeat:

| Shape | Typical meaning | Student risk |
| --- | --- | --- |
| Always-free allotment | A small cap every month (e.g. a tiny VM hours budget, a few million function invocations) | Bursting over the cap silently |
| Time-boxed trial credits | Money or hours that expire | Credit ends, same resources stay up |
| Twelve-month new-account offers | Some SKUs free-ish for a year | Still bill companion meters (IPv4, NAT, snapshots) |
| Student / education packs | Extra credits via GitHub Student or faculty programs | Shared org rules; do not treat as personal AWS |

**Never free in spirit:** NAT gateways left on, load balancers with no instances, public IPv4 on every NIC, multi-AZ “because production,” GPU VMs, unmanaged egress to the whole class.

Read [12-03 FinOps]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) before you click a console.

## IUH order of operations

Print this in `NOTES.md` and tick it if you use any cloud:

1. [ ] Instructor (or faculty admin) confirmed the **account type** (personal / university org / playground).
2. [ ] You can open **Billing** and see the currency.
3. [ ] **Budget + alert** at a low cap (instructor number, or a few USD). Email goes to **both** pair members.
4. [ ] Default region is set; you will not create “just one extra” resource in a second region.
5. [ ] MFA on the login you actually use.
6. [ ] No long-lived access keys in chat apps. Prefer console + OIDC later (13-04).
7. [ ] Tags `project`, `pair`, `env`, `owner` on everything.
8. [ ] Destroy plan written *before* create (Lab B).
9. [ ] After the session: empty region check (instances, disks, IPs, snapshots, log groups, registries).
10. [ ] Screenshot or export of **cost = 0 or near 0** for the day, dated.

If step 3 is impossible, **stop**. Go back to Lab A.

## Optional “hello edge” (only after the ticks)

A *sufficient* cloud hello for this course is **one** of:

- A single tiny VM or Cloud Run / Azure Container Apps / Cloud Run-like service in front of **nothing valuable**, with a provider-managed TLS hostname, **or**
- DNS + TLS on a free hostname you already own, pointing at a **local** tunnel the instructor approved (many classrooms will skip tunnels).

You do **not** need to reproduce a three-tier VPC in a personal account. A diagram plus Lab B plan is enough for networking literacy.

### Hello checklist

- [ ] The public URL uses **HTTPS**.
- [ ] The origin is either a managed service with scale-to-zero or a VM you will **halt and destroy** the same day.
- [ ] Security group / firewall: 443 from the internet, **no** 22 from `0.0.0.0/0`.
- [ ] You wrote the destroy commands or console clicks in `NOTES.md` and ran them.

## Classroom ethics

- Do not host pirated content, scanners, or crypto miners.
- Do not store real academic records on a personal free-tier project.
- Do not share root passwords with the class group chat.
- If you accidentally create a billable meter, tell the instructor the same day—not after the invoice.

## Done when

You can explain, without a slide, the difference between “always free,” “credits,” and “I left a NAT on.” If you never opened a cloud account, your `NOTES.md` still lists the ten ticks and **which ones blocked you**—that is a valid Lab C.

---

### Worked Concrete Example: Hidden Monthly Cost of "Forgotten" Idle Lab Resources

Calculate what happens if a student leaves just 3 tiny "invisible" AWS resources running for a 30-day month (720 hours) after terminating their EC2 instance:

| Forgotten Resource | Hourly Rate | 720-Hour Monthly Bill | Covered by Free Tier? |
| :--- | :--- | :--- | :--- |
| **1 × NAT Gateway** (idle!) | $$0.045	ext{ USD/hr}$$ | **$$32.40	ext{ USD}$$** | **NO!** |
| **1 × Application Load Balancer** | $$0.0225	ext{ USD/hr}$$ | **$$16.20	ext{ USD}$$** | Partial (750h first 12 mo only) |
| **1 × Unattached Elastic IP** | $$0.005	ext{ USD/hr}$$ | **$$3.60	ext{ USD}$$** | **NO!** |
| **Total Surprise Bill** | — | **Up to $$52.20	ext{ USD/mo}$$** | — |

That single table is why **zero-spend IaC teardown (`tofu destroy`) + a $1 billing alarm** is mandatory before any student deploys to a real cloud account!

### Explanation: Four Cloud Hygiene Rules Every Student Engineer Should Follow

1. **Create the $1 billing alert *before* creating your first VM:** It takes 2 minutes and protects your credit card permanently.
2. **Work in ONE single cloud region (e.g., `ap-southeast-1` Singapore):** Switching regions randomly hides running resources in another region’s console view!
3. **Never push `.env`, `credentials`, or `.pem` files to GitHub:** Install `gitleaks` or check `git status` before pushing—a leaked AWS key can rack up thousands of dollars in crypto-mining instances within an hour.
4. **Prefer serverless / free student platforms (Render, Fly.io free allowances, GitHub Pages, local Docker) when a full VPC isn’t needed.**

**Quick check.** Why can stopping (rather than terminating) an EC2 instance—or deleting an EC2 instance while leaving a NAT Gateway in the VPC—still generate monthly cloud charges?

---

## Video tham khảo

- **xem trước:** [Introducing the new AWS Free Tier Program \| Amazon Web Services](https://www.youtube.com/watch?v=pisDsIgXGUI) — Amazon Web Services, English. Watch first: AWS’s own announcement of the current free-tier program, which this lab says to re-check on the provider page.
- [AWS Free Tier and the New AWS Experience](https://www.youtube.com/watch?v=7aOb0vZHT5k) — AWS Developers, English. A longer AWS Developers walkthrough of the free tier and the console around it.
- [AWS Cost Optimisation Series: Cost Explorer \| Amazon Web Services](https://www.youtube.com/watch?v=oE8TNKGmc40) — Amazon Web Services, English. Cost Explorer, so the meters behind a “free” account are visible before anything is left running.

## References

1. Current AWS Free Tier, Azure free services, and GCP free program pages (open on the lab date).
2. GitHub Student Developer Pack (if eligible)—still not a license to skip budgets.
3. [infracourse.cloud](https://infracourse.cloud/) as inspiration for taking DNS/TLS and cost seriously. Procedures above are IUH-original.
