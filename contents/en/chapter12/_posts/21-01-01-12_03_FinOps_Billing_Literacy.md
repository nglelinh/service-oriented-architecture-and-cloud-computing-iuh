---
layout: post
title: 12-03 FinOps and Cloud Billing Literacy
chapter: '12'
order: 4
owner: Nguyen Le Linh
lang: en
categories:
- chapter12
lesson_type: optional
---

Chapter 01-06 introduced FinOps as NIST *measured service* with a feedback loop. Chapter 12’s required notes mention pricing calculators and egress. This optional lesson is **billing literacy for IUH pairs**: how an invoice is structured, which meters surprise students, and what you must turn on *before* any free-tier experiment.

It will not make you a finance team. It will stop a “small demo” from becoming an unexplained bill.

## Learning objectives

- Read a cloud invoice as **meters × rate × time**, not as a single “cloud fee.”
- Tag resources so a two-person project can answer “what did Course Board cost this week?”
- Place **budgets and alerts** *before* the first `apply`, and treat destroy as part of Definition of Done.
- Compare on-demand, committed, and spot-style capacity as *risk*, not as a discount coupon.

## 1. Measured service is a product, not a courtesy

Public cloud bills are usage reports. The provider is not guessing. Typical **meters** you will actually see in student work:

| Meter family | Examples | Why students miss it |
| --- | --- | --- |
| Compute hours | VM, load balancer, NAT gateway | “I turned the VM off” but the NAT stayed |
| Storage GB-month | Disk, object store, snapshots, logs | Snapshots and unused disks after destroy-the-VM |
| I/O and requests | Object PUTs, DB IOPS | Tiny apps, chatty health checks |
| Egress | Bytes leaving a region or to the internet | A classmate in another country downloading images |
| Managed extras | Public IPv4, Container Registry storage | IPv4 addresses became their own line item in the 2020s |

A toy model you can keep in a notebook:

$$
C = \sum_i (q_i \cdot p_i)
$$

where $$q_i$$ is quantity on meter $$i$$ (hours, GB-month, GB egress) and $$p_i$$ is the published unit price. **Unit economics** divide $$C$$ by a unit of useful work (successful Course Board posts, graded submissions, inferences):

$$
u = C / N_{\text{success}}
$$

Chapter 01-06 showed this for GPU inference. Use it for a CRUD app too. If $$N_{\text{success}} = 0$$, you do not have a unit cost—you have a hobby that still invoices.

```python
from dataclasses import dataclass

@dataclass
class Line:
    name: str
    quantity: float
    unit_price: float

    def amount(self) -> float:
        return self.quantity * self.unit_price

def invoice_total(lines: list[Line]) -> float:
    return sum(line.amount() for line in lines)

demo = [
    Line("lb-hours", 72, 0.025),
    Line("nat-hours", 72, 0.045),
    Line("disk-gb-month", 20, 0.10),
    Line("egress-gb", 8, 0.09),
]
print(round(invoice_total(demo), 2))
```

The numbers above are **illustrative**, not a quote. Always open the provider’s current price page.

## 2. The FinOps loop at student scale

The FinOps Foundation popularized a three-phase loop: **Inform → Optimize → Operate**. At IUH pair scale that is:

1. **Inform** — tags, a budget, a weekly screenshot of cost-by-tag.
2. **Optimize** — stop idle, rightsize, prefer local Docker when the cloud adds no learning.
3. **Operate** — destroy is scheduled; one person owns the bill alert email.

Tags that a capstone *must* have:

```text
project=iuh-courseboard
pair=<student-ids-or-team-name>
env=lab|staging|prod
owner=<github-handle>
```

Untagged resources are “lost change.” Providers let you write a policy that *refuses* untagged creates (Chapter 14-02). Even without a policy, the report should list every resource and its tag.

<div class="content-box warning-box">
<p><strong>Budget first, resources second.</strong> Create a monthly budget with an email/SMS at 50% and 80% of a small cap (for example a few USD, or whatever the instructor states). If you cannot create a budget, do not <code>apply</code> paid APIs. Use Lab A on localhost.</p>
</div>

## 3. Price shapes: on-demand, committed, interruptible

| Shape | What you buy | Fits a two-week lab? |
| --- | --- | --- |
| **On-demand** | Pay per hour/second, no promise | Default. Easiest to delete. |
| **Committed / reserved / savings plan** | Lower unit price for a 1–3 year promise | Almost never. You will pay after the semester. |
| **Spot / preemptible / low-priority** | Spare capacity, can vanish | Optional for batch; terrible for an unmarked demo on marking day. |

“Free tier” is a **fourth shape** with a time box and a product list. It is not infinite on-demand. Lab C walks the hygiene. The literacy point: **free is a coupon on specific meters**, not a blanket.

## 4. The surprise menu (IUH edition)

Memorize these as *questions*, not as 2026 prices:

1. **NAT gateways** billed per hour *and* per GB. A private subnet that only needed `apt update` once can still cost more than the VM.
2. **Load balancers** idle in a public subnet after you deleted the instances.
3. **Public IPv4** as a separate meter.
4. **Observability ingestion** (log GB, span count) if you pointed a vendor agent at noisy debug logs (Chapter 13-03).
5. **Egress** from object storage to a phone on cellular data during a classroom demo.
6. **Snapshots and AMIs** that survive `terraform destroy` when they were created outside the stack.
7. **GPU / managed AI APIs** — token and accelerator hours from Chapter 01-06; keep them off the default capstone.

If a meter is not teaching a learning outcome, **do not turn it on**.

## 5. Architect for scale *and* cost (the Chapter 12 checklist)

Use this list when you pick AWS vs Azure vs GCP *or* when you stay on Compose:

1. What is the **unit of work** and the target $$u$$?
2. Which component **must** be elastic, and which can be a single container?
3. What happens to cost when load is **zero** (scale-to-zero vs a forgotten VM)?
4. Where does **egress** cross a billing boundary?
5. Do we need **two AZs** for the learning outcome, or is one AZ plus a backup story enough?
6. Is a **managed** database cheaper than the hours you will spend patching Postgres—and than the idle hours if you leave it up?
7. What is the **destroy order**, and who runs it after the demo?
8. Is there a **budget alert** on the account that *this* pair can see?

Chapter 12-01 already said “use a calculator.” This checklist is what you feed the calculator.

## 6. Ethics and shared accounts

Course accounts are not personal unlimited compute.

- Do not mine, train large models, or host public file dumps on a classroom grant.
- Do not share root credentials in a Messenger group.
- If the faculty provides a shared organization, stay inside the **project / subscription** you were given.
- Record estimated spend in the capstone report even when the amount is $$0$$ because you stayed local.

Billing literacy includes **saying no** to architecture that is only impressive on a slide.

## Challenges

- Price pages change; a blog post from 2022 is not a quote.
- Shared-responsibility confusion: the provider meters the disk, you chose to leave it.
- Tag drift when someone clicks in the console (ClickOps vs Chapter 14).
- Currency and tax lines on invoices that students skip when they only look at the big number.

## Exercises

1. Using the toy `invoice_total` model, add a forgotten NAT (200 hours) and say whether the Course Board unit cost is still honest if $$N_{\text{success}}$$ is 40 demo clicks.
2. Write a six-line “end of lab” destroy checklist that includes snapshots, public IPs, and log groups.
3. A teammate wants a reserved 1-year VM for a two-week capstone. Argue for or against using the table in §3.
4. Pick one provider price calculator and estimate **idle** cost of: one small VM + one disk + one unused load balancer for 14 days. State the date you looked.

## Where this goes next

- [01-06 Modern cloud applications]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_06_Modern_Cloud_Applications %}) — FinOps as measured service.
- [Lab C — free-tier hygiene]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %})
- [Capstone]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) — cost paragraph is required.

## References

1. FinOps Foundation, *FinOps Framework* (Inform / Optimize / Operate)—public vocabulary this lesson shrinks to pair scale.
2. Provider pricing and free-tier pages for AWS, Azure, and GCP (verify on the day you estimate).
3. Flexera *State of the Cloud* (see Chapter 01-06) for why spend management is a first-class cloud skill.
4. [infracourse.cloud](https://infracourse.cloud/) — CS 40 includes cost/ethics in the deploy arc. Figures, Course Board examples, and IUH rules here are original.
