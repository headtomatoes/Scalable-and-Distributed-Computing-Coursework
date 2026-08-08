# Preparation Plan — Scalable & Distributed Computing

**Goal:** deep working understanding of distributed systems. The coursework grade is a
by-product, not the target. Success is measured by what you can *explain and predict*,
not by what compiles.

---

## Rules of engagement

These are load-bearing. They are the reason this works or doesn't.

1. **You write 100% of the lab code.** Claude never produces a working implementation of
   any lab. Not when you're stuck, not at 1am, not "just this once as a reference".
2. **Claude's job** is to explain mechanisms, give illustrative snippets of ≤10 lines,
   ask you to predict what breaks, review what you wrote, and design failure scenarios
   that attack it.
3. **Claude may freely build** non-lab tooling: log visualisers, note scaffolding, the
   glossary, drill banks, this plan.
4. **A green test is not evidence of understanding.** It is evidence the code works.
   The three rituals below supply the missing evidence.

### The three rituals

| Ritual | When | What it catches |
|---|---|---|
| **Predict-then-run** | *Before* running a test | Mental models that are confidently wrong |
| **Post-lab grilling** | When a lab section goes green | Design choices you made by accident |
| **Written explainer** | After each lab section | Gaps that only surface when you write prose |

Explainers go in `docs/explainers/`, one Markdown file per lab section, in your own
words. This is the ritual that gets dropped when you're behind on 3B. Don't drop it —
it's the one that produces revision material you'll actually trust in an exam.

Design decisions you make and might later question (election timeout ranges, when you
persist, how you batch) go in `docs/adr/` as short ADRs. Future-you will ask why.

---

## Environment — verified, not assumed

Established by actually running it on 2026-08-08:

| Item | Status |
|---|---|
| Labs source | MIT **6.5840, 2025 edition**, cloned to `labs/` at upstream `8de621b` |
| Upstream diffing | remote `upstream` + tag `pristine`. `git diff pristine` inside `labs/` shows exactly your work |
| Go | 1.25.1 on Windows (module targets go 1.22 — fine) |
| Labs 2–5 | **Run natively on Windows.** Verified: `raft1` and `kvsrv1` both compile, run, and fail correctly pre-implementation |
| Lab 1 (MapReduce) | **Blocked on Windows.** Needs `-buildmode=plugin`, unsupported on `windows/amd64`; `test-mr.sh` is bash |
| WSL2 Ubuntu | Installed, sees the repo at `/mnt/c/...`, but **Go is not installed in it** |

**The known-good red bar** (this is what "setup works" looks like):

```
$ go test ./raft1/ -run TestInitialElection3A
Test (3A): initial election (reliable network)...
Fatal: expected one leader, got none
--- FAIL: TestInitialElection3A
```

From here, a red bar means *your code*. Never your setup. If you ever suspect the
setup, re-run the above on a clean checkout before debugging anything else.

### Open environment item

Lab 1 needs Go inside WSL Ubuntu (`sudo apt install golang-go`, or better, a current
tarball from go.dev since apt's Go lags). Not done — it's a change to your WSL box and
it's your call. Until then, **start at Lab 2**; nothing downstream depends on Lab 1.

---

## The ladder

Ordered by dependency, not by lab number. Weeks assume ~10h/week steady.

### Week 1 — Lab 2: Key/Value server (`kvsrv1`)

Deceptively small. It is where at-most-once semantics and duplicate detection are
learned cheaply, before Raft makes every bug cost a weekend.

Tests: `TestReliablePut` → `TestManyClientsUnreliable`

**Checkpoint — you can explain, without notes:**
- Why a retried Put is not the same request, and what the client must carry so the
  server can tell.
- Why "the RPC failed" and "the operation didn't happen" are different statements.
- What breaks if the server dedups by request content instead of request identity.

### Weeks 2–3 — Lab 3A: Raft leader election (`raft1`)

Tests: `TestInitialElection3A`, `TestReElection3A`, `TestManyElections3A`

**Checkpoint:**
- Why election timeouts must be randomised, and what happens at what magnitude if they
  aren't.
- Why a term is not a clock, and what a term actually orders.
- The exact conditions under which a server grants a vote — all of them.

### Weeks 4–5 — Lab 3B: Log replication

The hard one. Budget the overrun here, not elsewhere.

Tests: `TestBasicAgree3B`, `TestFailAgree3B`, `TestFailNoAgree3B`, `TestRejoin3B`,
`TestBackup3B`, `TestCount3B`, `TestConcurrentStarts3B`

**Checkpoint:**
- The Log Matching property, and why the `prevLogIndex`/`prevLogTerm` check enforces it.
- Why a leader may never commit an entry from a previous term by counting replicas
  alone (this is Figure 8 — meet it here, it bites in 3C).
- The difference between *committed* and *applied*, and why `commitIndex` never moves
  backwards.

### Week 6 — Lab 3C: Persistence

Tests: `TestPersist13C`–`TestPersist33C`, `TestFigure83C`, `TestFigure8Unreliable3C`,
`TestUnreliableChurn3C`

**Checkpoint:**
- Exactly which three pieces of state must be durable, and the crash that loses
  correctness if any one isn't.
- Why persistence must complete *before* replying, not after.

### Week 7 — Lab 3D: Snapshots

Tests: `TestSnapshotBasic3D` → `TestSnapshotInit3D`

**Checkpoint:**
- Why log compaction forces an `InstallSnapshot` RPC to exist at all.
- What a follower must do when a snapshot's last index is behind its own log.

### Weeks 8–9 — Lab 4: Fault-tolerant KV on Raft (`kvraft1`)

Tests: `TestBasic4A` → `TestRestartSubmit4A`, then `4B` (partitions, persistence),
then `4C` (snapshots).

**Checkpoint:**
- What linearizability means operationally, and how Porcupine falsifies it.
- Why a client request may be applied twice without the client ever knowing, and what
  stops that.
- How a leader knows a read is safe to serve.

### Week 10+ — Lab 5: Sharded KV (`shardkv1`) — stretch

Tests: `5A` (joins/leaves), `5B` (controller recovery), `5C` (partitions).

**Checkpoint:**
- Why shard migration is a distributed transaction wearing a disguise.
- What makes a configuration change safe to apply at different times on different groups.

---

## Syllabus layer

The ladder is the spine regardless of what the course covers. When the official
syllabus PDF lands (drop it in `docs/`), it gets extracted and mapped into an added
layer for anything the labs don't reach — likely candidates: CAP/PACELC, MapReduce/Spark
programming models, cloud deployment, quorum systems, eventual consistency.

Extraction note: `PyPDF2` works on this machine; the page-render path does not
(poppler/`pdftoppm` not installed).

---

## Status

- [x] Repo initialised, labs cloned, upstream + `pristine` tag set
- [x] Toolchain verified against real test runs (Labs 2 and 3)
- [ ] `CONTEXT.md` glossary seeded
- [ ] Go installed in WSL (unblocks Lab 1) — *your decision*
- [ ] Syllabus PDF added
- [ ] Lab 2 started
