# EC2 Instances: Types, Tenancy, and Pricing

This guide summarizes the accompanying PDF, **Beyond the Basics: The Builder’s Guide to AWS EC2 Instances - Types, Tenancy, and Pricing**. It explains how to select EC2 compute, where it runs, and how to pay for it.

## The three decisions

Treat these as separate choices that combine to define an instance:

1. **Instance type** selects the hardware profile: CPU, memory, storage, and network balance. A name such as `t3.micro` has a family (`t`), generation (`3`), and size (`micro`). Generation numbers identify a hardware generation; compare current performance and price rather than assuming a newer generation is always faster or cheaper.
2. **Tenancy** controls whether the underlying physical host is shared, dedicated to your AWS account, or allocated as a specific host.
3. **Purchase option** determines the price/commitment trade-off and, for Spot, whether capacity can be interrupted.

These decisions are independent points. For example, a compute-optimized instance can use shared tenancy and On-Demand pricing, or a different tenancy and purchase model if the requirements call for it.

## Instance families

| Family | Resource emphasis | Typical fit | Key point |
| --- | --- | --- | --- |
| **T** - burstable general purpose | Balanced CPU and memory for variable demand | Small services, development, low-to-moderate baseline workloads | Earn CPU credits at low utilization and spend them to burst. Sustained use can exhaust credits and limit performance, depending on the configured credit mode. |
| **M** - general purpose | Balanced resources with steady performance | General application servers and continuously busy APIs | Use when demand is consistently high and burst credits are a poor fit. |
| **C** - compute optimized | Higher compute-to-memory ratio | Encoding, batch processing, gaming servers, CPU-heavy jobs | Suited to work that spends more time processing than holding large datasets in RAM. |
| **R / X** - memory optimized | Large RAM capacity | In-memory databases and caches such as Redis or Memcached | Keeps more active data in memory to reduce slower storage/database reads. |
| **I / D** - storage optimized | High local storage capacity or I/O throughput | Log processing, data-intensive and distributed storage workloads | Instance-store disks are physically attached and are ephemeral; persist data elsewhere when it must survive instance stop/termination or host failure. |

These family letters are representative examples from the PDF, not a complete or fixed catalog. Verify current instance specifications, supported sizes, and regional availability before choosing.

**Architecture diagram:** [Open the diagram](./diagrams/excal_img2_dark.png)

## Tenancy: physical placement

| Option | What is isolated | Who selects the host? | Common reason |
| --- | --- | --- | --- |
| **Shared** (default) | Each customer is logically isolated on shared physical infrastructure | AWS | The usual, flexible, lower-cost default. |
| **Dedicated Instance** | Physical hardware is dedicated to your AWS account | AWS; placement can change after stop/start | A policy or compliance requirement for account-level physical isolation. |
| **Dedicated Host** | A specific physical server is allocated to you, with host/core visibility | You can control placement on the allocated host | Host-level visibility and licensing requirements, including some BYOL scenarios. |

Dedicated tenancy is not a substitute for application security controls or a complete compliance program. Select it to satisfy a specific placement, licensing, or policy requirement.

**Architecture diagram:** [Open the diagram](./diagrams/excal_img3_dark.png)

## Purchase options: cost and flexibility

| Option | Commitment and flexibility | Best suited to |
| --- | --- | --- |
| **On-Demand** | No long-term commitment; pay for running compute | Experiments, unpredictable demand, and new workloads without a known baseline. |
| **Savings Plans** | Commit to a consistent hourly spend for a term; eligible usage receives discounted rates, with flexibility depending on plan type | Predictable baseline usage where flexibility across eligible compute is valuable. |
| **Reserved Instances (RIs)** | A billing discount in exchange for a term commitment; scope and flexibility depend on RI type | Existing workloads where RI attributes and terms fit the usage. Compare with Savings Plans before committing. |
| **Spot** | Uses spare EC2 capacity at variable discounts; AWS can reclaim it | Fault-tolerant, restartable, flexible work such as queue-based batch processing. Never make a single Spot instance the only copy of durable data. |

The PDF quotes maximum discounts of **up to 72%** for Reserved and Savings Plan instances and **up to 90%** for Spot. These are ceilings, not expected savings: actual rates depend on the service, region, instance, plan/term, and current AWS pricing. Spot is spare capacity offered at a Spot price, not a guaranteed auction win or guaranteed availability. AWS generally provides a two-minute interruption notice when reclaiming capacity, but notices are best effort; design for interruption rather than relying on the full warning.

Stopping an EBS-backed instance stops instance compute charges while it is stopped, but attached EBS volumes and other retained resources can continue to incur charges. Terminate disposable lab resources and check for remaining volumes, snapshots, addresses, and other billable resources.

**Architecture diagram:** [Open the diagram](./diagrams/excal_img4_dark.png)

## How the pieces fit together

In a real-world application, you rarely pick just one EC2 setup for everything. Instead, you mix and match instance types, tenancy, and pricing to fit what each part of your system actually does:

* **Protecting background work with a queue:** Spot instances are much cheaper because AWS can reclaim them when capacity is needed elsewhere. By placing tasks in a queue first, a worker job isn't lost if a Spot instance gets interrupted—another worker simply picks it up and retries. Make sure jobs are safe to run more than once without creating duplicates, and save finished results outside the worker.
* **Speeding up reads with a cache:** A cache keeps frequently used data in fast memory (RAM) so your web tier doesn't have to hit the database for every request. Because memory is temporary, treat the cache as a speed boost only—always store your permanent, official ("source-of-truth") data in a durable database or storage service.

| Workload | Example from the PDF | Why it fits |
| --- | --- | --- |
| Web/frontend tier | `t3.medium`, shared tenancy, 1-year Savings Plan | Handles everyday traffic spikes with burstable CPU, while a 1-year Savings Plan lowers costs for a server that stays on 24/7. |
| Background PDF workers | `c5.large`, Spot, queue-backed jobs | CPU-heavy tasks like generating PDFs can pause and retry safely on another worker if AWS reclaims the Spot instance. |
| In-memory cache | `r6g.large`, Dedicated Instance in the example | Memory-optimized instances give fast RAM for caching. Only use Dedicated tenancy if a specific policy or compliance rule requires isolated hardware—it doesn't secure data on its own. |

Instance sizes and prices in this example are just for illustration—always choose based on your app's actual workload and current regional pricing.

**Architecture diagram:** [Open the diagram](./diagrams/excal_img1_dark.png).

## A practical learning exercise

The PDF’s hands-on exercise is to launch a small `t3.micro` instance, keep shared tenancy and On-Demand pricing, connect over SSH, install Nginx, and open its public address in a browser. Then terminate the instance and check for leftover pricing resources. Free Tier eligibility and included amounts depend on current AWS account terms; confirm them in the console before launching.