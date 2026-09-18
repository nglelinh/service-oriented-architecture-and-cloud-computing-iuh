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

## References

1. Current AWS Free Tier, Azure free services, and GCP free program pages (open on the lab date).
2. GitHub Student Developer Pack (if eligible)—still not a license to skip budgets.
3. [infracourse.cloud](https://infracourse.cloud/) as inspiration for taking DNS/TLS and cost seriously. Procedures above are IUH-original.
