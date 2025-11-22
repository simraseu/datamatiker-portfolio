---
title: "Database & Storage"
weight: 2
---

# Fra Problem til PostgreSQL

**Problemet:** Et Blazor chatbot-system med tre chatbot-typer skulle have en database. Men hvilken? MongoDB til JSON? PostgreSQL til relations? Pinecone til vector search? Tre systemer?

Dette er historien om hvordan jeg gik fra antagelser til evidens-baseret beslutning.

---

## Vælg Din Tilgang

Du har to muligheder for at udforske Database & Storage delen:

### 📚 Læs Hele Rejsen (6 Faser)

Følg den komplette proces fra problemidentifikation til implementering:

1. **[Problemstilling]({{< relref "database/problemstilling.md" >}})** - Fire kritiske krav og failure-scenarier
2. **[Research]({{< relref "database/research.md" >}})** - 5 kilder fra systematisk litteratursøgning  
3. **[Hypoteser]({{< relref "database/hypoteser.md" >}})** - 4 testbare påstande med success-kriterier
4. **[Praktisk Test]({{< relref "database/praktisk-test.md" >}})** - Empirisk validering af alle hypoteser
5. **[Database Design]({{< relref "database/design.md" >}})** - Konkret PostgreSQL arkitektur
6. **[Konklusion]({{< relref "database/konklusion.md" >}})** - Læring, læringsmål og samfundsperspektiver
---

### ⚡ Executive Summary (Herunder)

Få hele historien på 3 minutter. Scroll ned for:
- Evidens fra alle 5 kilder
- Test resultater (H1-H4)
- Beslutningen og rationale
- Læringsmål opfyldt
- Samfundsmæssige perspektiver

---

## Executive Summary

### Evidens: Hvad Viste Researchen?

**5 kilder fra systematisk litteratursøgning:**

| Kilde | Type | Main Finding |
|-------|------|--------------|
| OnGres PostgreSQL vs MongoDB | Vendor benchmark | PostgreSQL 25-40× hurtigere på JSON |
| Makris et al. (Springer) | Peer-reviewed | PostgreSQL 4× mindre disk, 4× hurtigere queries |
| Hightower pgvector | Production case | Native vector search eliminerer separat DB |
| Microsoft EF Core Docs | Official docs | PostgreSQL 100% vs MongoDB 60% feature support |
| AWS ACID vs BASE | Technical docs | Strong consistency uden performance-tab |

**Konvergens:** Alle kilder pegede mod PostgreSQL som optimal løsning.

---

### Test Resultater: Holder Evidensen i Praksis?

**4 hypoteser valideret empirisk på min hardware:**

| Hypotese | Success-kriterium | Resultat | Status |
|----------|-------------------|----------|--------|
| **H1: JSON Performance** | PostgreSQL < Mongo/20 | **26.4× hurtigere** | ✅ Validated |
| **H2: Vector Integration** | 1 roundtrip | **1 vs 2+** (89ms vs 245ms) | ✅ Validated |
| **H3: Developer Experience** | ≤ 50% LoC | **47%** (18 vs 38 LoC) | ✅ Validated |
| **H4: Data Consistency** | 0 partial saves | **0% vs 70%** failure | ✅ Validated |

---

### Beslutningen: PostgreSQL + pgvector

**Valgt løsning:** PostgreSQL 16.1 med pgvector extension som unified platform.

**Hvorfor PostgreSQL vandt:**
- ✅ 26× hurtigere JSON queries end MongoDB
- ✅ Native vector search (eliminerer Pinecone)
- ✅ 100% EF Core support (standard patterns virker)
- ✅ ACID guarantees (0% partial saves ved crashes)
- ✅ 4× mindre disk space (lavere TCO: $4,400 savings over 3 år)

**Hvorfor MongoDB blev fravalgt:**
- ❌ Konsistent dårligere performance på alle metrics
- ❌ Manglende native vector support kræver separat database
- ❌ Incomplete EF Core provider (60% features, kræver workarounds)
- ❌ 70% data integrity failure ved crash scenarios

**Trade-offs accepteret:**
- PostgreSQL kræver mere initial setup end managed MongoDB Atlas
- pgvector ~20% langsommere end dedikerede vector databases ved pure vector search
- Men: For projektets use case (kombineret metadata+vector queries) er unified platform optimal

---

### Læringsmål Opfyldt

✅ **Læringsmål 1: Vector Search Implementation**  
Implementerede pgvector HNSW index for semantic search. Kombinerede metadata-filtering med vector similarity i én SQL-operation.

✅ **Læringsmål 2: ACID vs BASE Trade-offs**  
Dokumenterede konkrete failure-scenarier. Validerede empirisk at eventual consistency resulterer i 70% partial saves ved crashes.

✅ **Læringsmål 3: Database-ORM Integration**  
Kvantificerede developer experience impact: PostgreSQL kræver 53% mindre kode end MongoDB grundet mature EF Core provider.

✅ **Læringsmål 4: Holistisk Database-evaluering**  
Evaluerede total cost of ownership: $4,400 savings over 3 år ved 10k users. Koblede tekniske valg til økonomiske konsekvenser.

✅ **Læringsmål 5: Systematisk Research Metodologi**  
Gennemførte peer-reviewed litteratursøgning med clear inclusion/exclusion criteria. Triangulerede evidens fra vendor research, akademiske studier og production cases.

---

### Samfundsmæssige Perspektiver

**GDPR Compliance:**  
PostgreSQL's ACID transactions sikrer pålidelig implementation af "right to deletion" via CASCADE DELETE. MongoDB's eventual consistency introducerer risiko for orphaned data i distributed replicas.

**CO2 Footprint:**  
PostgreSQL's 4× storage efficiency betyder ~50 kg CO2 savings årligt ved 10,000 brugere (baseret på EU gennemsnitlig el-mix). Bedre query performance reducerer CPU-forbrug per request.

**Økonomisk Impact:**  
Total cost of ownership fordel på $4,400 over 3 år. Open-source model med community-driven extensions skaber konkurrence og lavere costs sammenlignet med vendor lock-in.

**Arbejdsmarked:**  
PostgreSQL har 3× flere job postings end MongoDB i Danmark (Q4 2024). Unified platform-trend reducerer krav til specialized skillsets, hvilket forenkler hiring og onboarding.

---

## Start Rejsen

**Klar til at dykke ned?** Vælg dit startpunkt:
