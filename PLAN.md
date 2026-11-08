# Plan

> **In short:** design an independent, version-agnostic Scheduler capability for the Nuxt Layers ecosystem. Scheduler executes registered work through a public structural contract; it does not import or understand the capabilities that declare that work.

**Status:** Architecture design proposal. No Scheduler implementation or public contract is accepted by this document.

## Scope and boundaries

Scheduler **owns** registering and triggering scheduled work, coordination between instances, overlap and concurrency policy, retry/backoff, timeouts, observability, and any Scheduler-owned persistence needed for dispatch and execution state. It may expose administrative status and execution controls only through authorised public interfaces.

Scheduler **does not own** a consumer's task logic, domain data, retention policies, outbox events, event publishing, business transactions, caller identity or permission rules. It must never inspect another capability's database schema or require an application-specific implementation.

A capability **owns** its task names, handlers, cadence requirements, idempotency guarantees and documented operational prerequisites. A composition root may register its public task descriptors with Scheduler, without the declaring capability importing Scheduler. A minimal host must be capable of composing Scheduler independently.

## Governing architecture

- ADR-0012 currently defines task descriptors as `{ name, every, run }[]`, with one host scheduler. It is authoritative unless amended.
- ADR-0008 requires latest accepted `master` during active development, with reproducible lockfiles.
- ADR-0002 requires capability-owned schema and supplied connection; ADR-0009 governs any optional presentation.
- Scheduler must have a complete standalone `CONTRACT.md`, a `capability.json` and `examples/minimal-host` under the Layer Standard.

## Decisions required before implementation

| Topic | Question | Proposed direction (not yet normative) |
|---|---|---|
| Task shape | Does `every` mean cron only, interval only or both? | Use an explicit tagged schedule rather than guessing, or retain legacy cron with a clearly documented interpretation |
| Unique identity | How are task names namespaced and de-duplicated? | Stable capability-owned names; reject conflicting registrations |
| Invocation | How does Scheduler safely invoke `run` across processes/deployments? | In-process handlers for locally registered tasks; persistent dispatch metadata separate from handlers |
| Multiple instances | How is at-most-one active execution coordinated? | Lease or claim mechanism with expiry; do not promise exactly-once delivery |
| Overlap | Can long-running tasks overlap their next scheduled trigger? | Configurable and explicit; conservative non-overlap default |
| Retry | Which errors retry, when and for how long? | Bounded retries and backoff, with traceable final failure |
| Timeout | Who declares cancellation and time limits? | Runtime-enforced timeout; handlers must be cancellation-aware where feasible |
| Delivery semantics | What about duplicate or interrupted execution? | At-least-once compatible; handlers own idempotency |
| Observability | What does registration/execution return? | Task lifecycle, last/next attempt, outcome and errors without leaking secrets |
| Clock/time zone | How are calendar times interpreted? | Explicit time zones and clock semantics; avoid implicit locale or daylight-saving assumptions |
| Access control | Who can register, trigger or inspect tasks? | Privileged server-side actions with fail-closed authorisation |
| Storage | What must survive application restart? | Scheduler owns only schedule and execution metadata; never task domain data |
| Lifecycle | How are registrations updated or removed? | Deterministic registration reconciliation and controlled removal |
| Resource limits | How are tasks bounded? | Limits on concurrency, memory-sensitive payloads and queue growth |

These options require review with the current ADR-0012 task shape and with `nuxt-layers/clock`'s public contract before they become requirements. Do not duplicate Clock's date/time semantics.

## Implementation sequence

| Stage | Status | Exit criterion |
|---|---|---|
| 1. Shared contract | Outstanding | Architecture owners accept the minimum task shape and execution semantics, or explicitly retain ADR-0012 unchanged |
| 2. Design | In progress | Record Scheduler's bounded capability, execution model, threat boundaries, persistence design and failure semantics |
| 3. Public contract | Outstanding | Standalone `CONTRACT.md` specifies installation, ports, calls, task registration, errors, permissions and operations |
| 4. Runtime | Outstanding | Implement registered execution, coordination, lifecycle and observability using Scheduler-owned internals |
| 5. Acceptance | Outstanding | Unit, security and integration coverage; independent Nuxt minimal host and <=50-line composition CI |
| 6. Composition | Outstanding | Verify generic registration of another capability's tasks through its public descriptors, without Scheduler importing it |

No package source, runtime implementation or dependency pin should be committed solely to make an undecided contract appear complete.
