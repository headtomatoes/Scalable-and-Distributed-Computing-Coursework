# Lecture 1: compact notes

Sources in this folder: `P1. Intro.pptx.pdf` (37 slides, "s"), `cloudintro.pptx.pdf`
(20 slides, "c"), `1.1. Tricks_in_colab_and_Bash-Tutorial.ipynb`.
Bold terms follow `CONTEXT.md`. The figures in the decks are from 2007–08, so learn the
reasoning and skip memorising the numbers.

**Thesis of the lecture:** one machine can't serve today's users or read today's data.
Expensive supercomputers don't scale far enough, so we use thousands of cheap, unreliable
machines. We make them reliable *in software*, and we give programmers a simple model
(MapReduce) that hides the distribution.

---

## A. Scalable systems (P1 Intro)

### 1. Why scale at all (s3–9)
- Google, Sept 2007: 11,081 machine-years of MapReduce in **one month**. ×12 gives ≈133k
  machines busy full-time, and MapReduce was only part of the workload.
- Reading the web: 20B pages × 20 KB = 400 TB. At 35 MB/s from one disk that takes
  ~4 months, and storing it takes ~1,000 disks.
- The LHC produces 700 MB/s, which is 60 TB/day or ~20 PB/year. Google also serves
  3,000 searches/s.
- The only way out is parallel or distributed processing at very large scale.

### 2. The design space (s10–16)
| Approach | Model | Hardware | Reliability | Weakness |
|---|---|---|---|---|
| SIMD / vector | one instruction over many data | exotic, expensive | expensive parts | cost, narrow use |
| MIMD shared-memory | threads on one memory, one OS | fast interconnect keeps caches coherent | expensive parts | can't grow very large |
| MIMD message-passing | messages between nodes (MPI-style), one OS per node | fast interconnect, often no local disk | expensive parts | cost |
| Client/server | many independent requests | cheap, modest bandwidth | **in software** | never speeds up *one* app |
| **Cluster (middle ground)** | simple model, e.g. MapReduce | cheap commodity boxes | **in software** | problem must be embarrassingly parallel |

- s15, Grapevine (Xerox PARC mail system): an early example of "reliability via
  software". It had replicated registration and message servers, so users could still
  locate, send and retrieve mail when one server failed.

### 3. Embarrassingly parallel (s17–20)
- Nodes have local memory and disk and are linked by commodity Ethernet. Communication is
  expensive, so the problem must split into pieces that barely talk to each other.
- Examples: ray tracing (each ray is independent), building a web index (list the words
  per page, then invert to pages per word), Rosetta@home (s19, volunteer computing where
  each work unit is independent).

### 4. Anatomy of one Google search (s21–25)
1. **DNS** sends you to a nearby cluster, one of many worldwide.
2. A **load balancer** picks a lightly loaded **Google Web Server (GWS)**, which
   coordinates the whole query.
3. The **index is partitioned into shards**. The GWS asks *one* index server per shard
   (through another load balancer). Result: doc IDs ranked by relevance.
4. The **docs are partitioned into shards by hash(docID)**. The GWS asks *one* doc server
   per relevant shard. Result: URL, title and snippet.
5. The ad server and spell checker run in parallel. The GWS builds the HTTP response.

Hundreds of machines take part in every single query.

### 5. Required properties (s26–28)
| Lecture term | Lecture definition | Course term |
|---|---|---|
| Fault-Tolerant | recovers from failures *without performing incorrect actions* | **Safety** |
| Highly Available | keeps providing service while some components have failed | **Availability** (always state the failure bound) |
| Recoverable | failed components restart and rejoin | **Recoverability** (plus durable state intact) |
| Consistent | coordinates actions across components despite concurrency and failure | |
| Scalable | still correct as some dimension grows | |
| Predictable performance | responds in a timely manner | |
| Secure | authenticates access | |

The system also has to be **simple to program** ("mere mortals") and **cheap**. One rack
cost ~$300K and 6,000 racks ≈ $2B. "Robust" hardware costs ≥2×, and a traditional
supercomputer at this scale is unaffordable.

### 6. Assumptions you may NOT make (s29)
This is a version of Deutsch's *Fallacies of Distributed Computing*. Do not assume:
- hardware is reliable or identical
- software is correct
- the network has zero latency, infinite bandwidth, or is secure
- configuration is stable, or there is one administrator

### 7. Replication vs partitioning (s30–31, the slide's question answered)
- **Partitioning** puts *different* data on different machines: index shards, and doc
  shards by hash. It solves **data too big** for one machine, and it parallelises each
  query.
- **Replication** puts the *same* data on several machines: many servers per shard, many
  clusters, many copies of the web. It solves **too many requests** and **failures**.
- How the properties follow (my reasoning, not on the slides):
  - **Availability**: load balancers skip dead replicas and DNS skips dead clusters.
  - **Recoverability**: a server holds read-only shard data, so rejoining means reloading
    its shard.
  - **Safety and consistency are cheap**: the query path only reads, and the index is
    built offline and swapped in.
  - **Scalability**: more data means more shards, and more queries mean more replicas.

### 8. MapReduce (s32–34)
- Many applications share one shape:
  - `map(record) → [(k, v)]`
  - group by key
  - `reduce(k, [v]) → result`
- Word count: map emits `(word, 1)` and reduce sums the ones.
- The runtime library handles distribution, failures and shuffling. The user writes only
  `Map` and `Reduce`.
- The Google stack: MapReduce runs on the **GFS** file system, uses **Chubby** for locks,
  and is complemented by **BigTable** storage. Hadoop is the open-source version and
  Dryad was Microsoft's.

### 9. Cloud economics (s35–37)
- Cloud means computing "out there": Gmail and Salesforce as services, AWS for your own
  apps.
- The provider absorbs purchase, housing, power, operation, and instant expansion or
  shrinking. Startups need little capital and can focus on their core business.
- s36: EC2 in 2008 charged $0.10 per small-instance-hour. So **1,000 CPUs × 1 h costs the
  same as 1 CPU × 1,000 h**: the same price for a 1,000× faster answer, *if the job
  parallelises*. That condition is why sections 3 and 8 matter.

---

## B. Introduction to Cloud Computing deck (NIST view)

**Definition (c2):** computing services (storage, processing, software) delivered over
the internet, on demand, without your own physical infrastructure.

**5 essential characteristics (c3):** on-demand self-service · broad network access ·
resource pooling (multi-tenant) · rapid elasticity · measured service (pay per use).

**4 deployment models (c4–8):**
| Model | Who uses the infrastructure | Trade-off |
|---|---|---|
| Public | anyone (AWS, Azure, GCP) | cheap, elastic, least control |
| Private | one organisation, on- or off-premises | control, compliance, privacy, high upfront cost |
| Hybrid | public + private combined | workload portability, integration complexity |
| Community | a group with shared needs (government, universities) | shared cost, shared compliance |

**3 service models (c9–12).** Stack from the bottom: servers → Infrastructure →
Platform → Application → clients.
| Model | You manage | Provider manages | Examples |
|---|---|---|---|
| IaaS | OS, runtime, app, data | hardware, virtualisation, network, storage | EC2, Azure VMs, Compute Engine |
| PaaS | app, data | everything below the app | Heroku, App Engine, Elastic Beanstalk |
| SaaS | settings and usage | everything | Salesforce, Microsoft 365, Workspace |

**Benefits (c13):** cost savings, scalability, flexibility, reliability, collaboration.
**Use cases (c14):** storage and backup, dev/test, web and mobile hosting, big-data
analytics, disaster recovery.

**NIST reference architecture: 5 actors (c16–17)**
- **Consumer** uses the services.
- **Provider** offers them. Inside the provider:
  - service layer (SaaS/PaaS/IaaS)
  - resource abstraction and control layer
  - physical layer (hardware, facility)
  - service management (business support, provisioning, portability)
  - security and privacy across all layers
- **Auditor** assesses independently: security, privacy and performance audits.
- **Broker** sits between consumers and providers: intermediation, aggregation, arbitrage.
- **Carrier** provides the network that carries the service from provider to consumer.

**Best practices (c18):** plan capacity up front · security first (encryption, patching,
zero trust) · automate disaster recovery and use redundancy · monitor performance ·
optimise cost.

---

## C. Colab and Bash notebook (hands-on, cheat sheet)

**Gotchas worth knowing**
- `!cmd` runs in a throwaway subshell, so `!cd dir` does **not** persist. Use `%cd dir`,
  or chain inside one line: `!cd dir && ls`.
- `%%bash` makes the whole cell a shell script. `%%writefile f.py` writes the cell to a
  file.
- `a & b & c` runs in the **background, concurrently**, so output arrives in any order.
  The notebook's `cat anscombe.json & ls & pwd` printed `pwd` first. `a && b` runs `b`
  only if `a` succeeded.
- `uniq` drops only **adjacent** duplicates, so `sort` first. `sed 's/x/y/'` replaces
  the first match per line and is case-sensitive; add `g` for every match.
- gdown: the notebook's labels are swapped. The `uc?id=` link works. The `open?id=` link
  gave a warning and saved a junk file.

**Commands**
| Purpose | Commands |
|---|---|
| Where am I / what's here | `pwd`, `ls`, `ls -l`, `ls -lh`, `ls ..` |
| Paths | `.` current dir · `..` parent · `~` home · escape spaces `My\ Drive` |
| Files | `mkdir`, `touch`, `cat f`, `cat > f`, `echo x > f` (overwrite), `>>` (append), `cp`, `rm`, `rm -r` |
| Help and pipes | `man ls \| head -20` |
| System | `which`, `echo $PATH`, `echo $HOME`, `uname`, `ps`, `whoami`, `nvidia-smi`, `cat /proc/meminfo` |
| Text | `wc` (lines, words, bytes), `uniq`, `sed`, `awk '{print $4 $5}'` |
| Colab | `drive.mount('/content/gdrive')`, `files.upload()`, `files.download(p)`, `gdown <uc link>` |

Skipped as not examinable: shortcuts, keep-alive JavaScript, pytube, TensorBoard, dark
mode. You can practise the Bash part in your WSL Ubuntu instead of Colab.

**Exercise (do it yourself):** create `Practice/` and inside it `test.txt` containing
`Hello world`. Create an empty `test2.txt` in `Practice`'s parent. Then remove both files.

---

## Links to the labs
- MapReduce (A8) → **Lab 1** (`labs/src/mr`). It is blocked on Windows until Go is in WSL.
- Replication and **Safety / Availability / Recoverability** (A5, A7) → **Labs 3–4** (Raft,
  fault-tolerant key/value store).
- Partitioning into shards (A4, A7) → **Lab 5** (sharded key/value store).
