---
title: "Phase 4: Distributed Systems"
nav_order: 6
has_children: true
---

# Phase 4: Distributed Systems

The moment a system spans more than one process over a network, a new physics
applies — one that punishes every assumption carried over from single-machine code.
This phase builds the distributed-systems judgment an architect can't do without:
the fallacies that poison naive designs, the consistency-vs-availability trade-off
stated precisely (CAP/PACELC), how services should communicate, how to keep data
correct once you've given up the database transaction, and how to design for the
failures that are guaranteed to come.

| # | Lesson | Status |
|---|--------|--------|
| 14 | [The fallacies of distributed computing](lesson-14-fallacies) | Ready |
| 15 | [CAP, PACELC & consistency models](lesson-15-cap-consistency) | Ready |
| 16 | [Communication styles — sync, async, REST, gRPC, messaging](lesson-16-communication) | Ready |
| 17 | [Distributed data — transactions, sagas, outbox, idempotency](lesson-17-distributed-data) | Ready |
| 18 | [Reliability & resilience patterns](lesson-18-resilience) | Ready |
