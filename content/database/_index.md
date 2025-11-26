---
title: "Database & Storage"
weight: 2
---

## MongoDB, PostgreSQL, Pinecone — eller bare én?

**Problemet:** Et Blazor chatbot-system med tre brugertyper krævede en database-arkitektur. Industristandarden sagde: "MongoDB til JSON, PostgreSQL til relationer, Pinecone til vector search." Tre specialiserede værktøjer.

**Men hvad hvis "specialized" ikke betyder "bedre"?**

Dette er historien om, hvordan jeg gik fra en kompleks **"Polyglot Persistence"** arkitektur (tre databaser) til en **"Unified Monolith"** (én database) — og opdagede, at implementation quality slår marketing categories.

---

## Vælg Din Tilgang

Du har to muligheder for at udforske Database & Storage delen:

### 📚 Læs Hele Rejsen (5 Faser)

Følg den komplette proces fra problemidentifikation til konceptuel validering: 

1. **[Problemstilling]({{< relref "database/problemstilling.md" >}})** — Tre dilemmaer: Integration Tax, Synchronization Nightmare, Consistency Myth
2. **[Research]({{< relref "database/research.md" >}})** — Systematisk triangulering af 5 kilder (vendor + peer-reviewed + production cases)
3. **[Design Patterns]({{< relref "database/design-patterns.md" >}})** — 4 normative patterns (Unified Monolith, ACID-First)
4. **[Konceptuel Validering]({{< relref "database/konceptuel-validering.md" >}})** — 4 stringente audits uden live data
5. **[Konklusion]({{< relref "database/konklusion.md" >}})** — Læring, Green IT og samfundsperspektiver

---

### ⚡ Executive Summary (3 minutter)

Få hele historien nedenfor:

#### Evidens: Hvad Viste Researchen?

Fem uafhængige kilder konvergerede på ét krav: **Unified platform outperformer polyglot persistence.**

| Kilde | Type | Main Finding |
|-------|------|--------------|
| **OnGres Benchmark** | Vendor | PostgreSQL 26-40× hurtigere på JSON end MongoDB |
| **Makris et al.** | Peer-reviewed | PostgreSQL 4× mindre disk, 4× hurtigere queries |
| **Hightower pgvector** | Production case | Native vector search eliminerer netværks-latency |
| **Microsoft EF Core** | Official docs | PostgreSQL 100% vs MongoDB 60% feature support |
| **AWS ACID vs BASE** | Technical guide | Strong consistency uden performance-tab |

**Konvergens:** Alle kilder pegede mod PostgreSQL + pgvector som optimal løsning.

---

#### Validation: Holder Patterns i Praksis?

Fire design patterns valideret gennem konceptuel analyse (uden live data):

| Pattern | Validation Method | Key Result | Status |
|---------|-------------------|------------|--------|
| **DP1: Unified Monolith** | Architecture Audit | 50% netværks-overhead elimineret | ✅ Validated |
| **DP2: Hybrid-Relational** | Literature Convergence | 26× hurtigere (vendor + peer-reviewed) | ✅ Validated |
| **DP3: Zero-Latency Vector** | Architecture Review | 1 roundtrip vs 3 = 60% latency reduction | ✅ Validated |
| **DP4: ACID-First** | Transaction Theory | 0% partial writes (guaranteed by atomicity) | ✅ Validated |

**Konkrete resultater:** Convergent evidence fra 3+ independent sources, architectural certainty for integration, official documentation for ORM support, theoretical guarantees for consistency.

---

#### Beslutningen: PostgreSQL + pgvector som Unified Monolith

Den endelige løsning bygger på fire patterns — valideret gennem theoretical analysis:

✅ **Unified Monolith (DP1):** Én instans håndterer SQL, JSON og Vectors. Eliminerer sync-lag.  
✅ **Hybrid-Relational (DP2):** JSONB + GIN index outperformer MongoDB BSON (26×).  
✅ **Zero-Latency Vector (DP3):** pgvector eliminerer Pinecone roundtrips (50% reduction).  
✅ **ACID-First (DP4):** Transaktioner garanterer 0% partial writes (kritisk for GDPR).

**Hvorfor MongoDB + Pinecone blev fravalgt:**

❌ MongoDB konsistent dårligere performance på JSON (26× langsommere)  
❌ Pinecone kræver separat database → 3 roundtrips, sync complexity  
❌ MongoDB 60% EF Core support → N+1 queries, workarounds  
❌ Eventual consistency → 70% partial saves ved crashes (GDPR risiko)

**Trade-offs accepteret:**

pgvector ~20% langsommere end dedicated Pinecone ved **pure** vector search (no metadata filtering). Men kombinerede queries 2.8× hurtigere grundet eliminated roundtrips. For chatbot use case (metadata+vector always combined) er unified platform optimal.

**TCO Analysis:** $4,400 savings over 3 år ved 10k users (hosting + developer time).

---

#### Samfundsmæssige Perspektiver

**GDPR Compliance:** PostgreSQL's ACID transactions sikrer pålidelig "Right to Erasure" via CASCADE DELETE. MongoDB's eventual consistency introducerer orphaned data risk.

**CO₂ Footprint:** PostgreSQL's 4× storage efficiency betyder ~50 kg CO₂ savings årligt ved 10,000 brugere.

**Økonomisk Impact:** Total cost of ownership fordel på $4,400 over 3 år. Open-source model med community extensions (pgvector) vs vendor lock-in (Pinecone).

---

## Start Rejsen

**Klar til at dykke ned?** [Læs Problemstillingen →]({{< relref "database/problemstilling.md" >}})