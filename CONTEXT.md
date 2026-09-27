# Scalable & Distributed Computing

The shared vocabulary for this coursework. Terms take their standard meaning from the
distributed-systems literature as the labs use it; where the lecturer's wording differs,
the entry notes the lecture phrasing so it can be translated in an exam.

## Language

### Fault tolerance

**Fault tolerance**:
The umbrella property of a system that keeps meeting its specification despite component
failures. It is made up of **Safety**, **Availability** and **Recoverability**.
_Avoid_: Reliability, robustness
_Lecture_: "Fault-Tolerant" (L1 slide 26) means only **Safety**.

**Safety**:
The property that the system never returns an incorrect result under any failure it is
designed for. A system that does nothing at all is trivially safe.
_Lecture_: "Fault-Tolerant" (L1 slide 26).

**Availability**:
The property that the system keeps serving requests while failures stay within a stated
bound (e.g. "a majority of servers are up"). A claim of availability that names no bound
is meaningless.
_Avoid_: Uptime, "highly available" with no bound given
_Lecture_: "Highly Available" (L1 slide 26).

**Recoverability**:
The property that a failed server can restart and rejoin the system with its durable state
intact, without violating **Safety**.
_Lecture_: "Recoverable" (L1 slide 26), which leaves out the durable-state requirement.

### Consistency

**Consistency model**:
The rule that says which results a system may return for operations on replicated or
concurrently-accessed state. **Safety** has no meaning until a model is named.
_Avoid_: "Consistency" and "consistent" on their own — always name the model
_Lecture_: "Consistent" (L1 slide 26) names no model, so it decides nothing.

**Linearizability**:
The model in which every operation appears to take effect at a single instant between its
call and its return, and every client sees one ordering that matches real time. It is the
"C" in CAP, and the model the labs check with Porcupine.
_Avoid_: Strong consistency, atomic consistency

**Eventual consistency**:
The model in which replicas may return differing results, but converge once updates stop.
It allows a read to return a value older than one already returned to another client.
