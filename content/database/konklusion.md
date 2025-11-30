---
title: "Konklusion"
draft: false
weight: 6
description: "Refleksion over læring, Green IT og arkitektonisk modenhed"
---

## Fra Hype til Arkitektur

Jeg startede database-delen med klassisk opdeling:

**"MongoDB til JSON, PostgreSQL til relationer, Pinecone til vectors. Tre problemer — tre specialized tools."**

Det lød logisk. MongoDB markedsfører sig selv som "document database" — selvfølgelig er den bedst til dokumenter? Pinecone er bygget til vector search — selvfølgelig er den hurtigst til embeddings?

Men gennem arbejdet med convergent evidence indsåede jeg at **database categories er marketing, ikke arkitektoniske garantier.**

Konklusionen blev den stik modsatte: **Simplification is the ultimate sophistication.**

Ved at vælge en **Unified Monolith (PostgreSQL)** har jeg ikke bare simplificeret koden (47% færre linjer), jeg har også opnået bedre performance, højere dataintegritet og stærkere GDPR-compliance.

---

## Læringsproces: Hvad var udfordrende?

### 1. Opgøret med "Best Tool" Dogmet

Min største læringsbarriere var at give slip på idéen om, at specialisering altid er lig med kvalitet. I undervisningen og på tech-blogs hører vi ofte: *"Brug graf-databaser til relationer, dokument-databaser til JSON, vektor-databaser til AI."*

**Konkret eksempel:** Da jeg så OnGres benchmark-resultaterne første gang, tænkte jeg: *"Det kan ikke passe. MongoDB ER document database — hvordan kan en relationel database være hurtigere?"*

Svaret var: **Implementation quality > database category.** PostgreSQL's JSONB er bygget med binary storage, GIN indexes og query optimizer integration der langt overgår MongoDB's BSON-implementation.

Det tvang mig til at ændre mit mindset fra at være **"Værktøjs-orienteret"** (Hvad kan MongoDB?) til at være **"Problem-orienteret"** (Hvad kræver min data?).

**Takeaway:** Marketing categories ("document DB", "vector DB") er ikke arkitektoniske garantier. Convergent evidence trumps assumptions.

---

### 2. Validering uden Empiri Kræver Disciplin

Den største metodiske udfordring var at validere design patterns uden live traffic.

**Oprindelig plan:** "Jeg kører benchmarks med mock data og rapporterer resultater."

**Problemet:** Uden rigtige brugere, realistic workload patterns og production-scale data er synthesized benchmarks akademisk usikre. Hvad hvis mine test queries ikke matcher real-world patterns?

**Løsning:** Anvend Data Science-sektionens metodologi — **konceptuel validation gennem convergent literature analysis**. Hvis vendor benchmarks, peer-reviewed studies og production cases alle peger samme retning, er evidensen stærkere end mine egne tests.

**Konkret eksempel:** I stedet for at "teste" at PostgreSQL er hurtigere, validerede jeg claim gennem:
- OnGres vendor benchmark (26× advantage)
- Makris peer-reviewed study (4× advantage, independent dataset)
- PostgreSQL technical docs (mechanism explanation: binary JSONB + GIN indexing)

Tre kilder, forskellige methodologies, samme konklusion = **convergent validity**.

**Takeaway:** Stringent validation er mulig uden empiri når man vælger rigtig metodisk tilgang. Men det kræver disciplin at ikke forfalde til anekdoter eller subjektive vurderinger.

---

### 3. "Polyglot Persistence" Lyder Smart, Men Er Det?

Jeg havde internaliseret "separation of concerns" princippet fra software engineering. Separate databases for separate data types lyder logisk.

**Men realiteten:** Data duplication, sync lag, 3× failure points, 60% ekstra latency, $4,400 årlig TCO overhead.

**Konkret eksempel:** Semantic search query i polyglot setup:
1. App → Pinecone (vectors): 120ms
2. App → MongoDB (metadata): 100ms
3. App merges + filters client-side: 25ms
4. **Total: 245ms + 3 failure points**

Unified platform (PostgreSQL + pgvector):
1. App → PostgreSQL (combined query): 89ms
2. **Total: 89ms + 1 failure point**

**Takeaway:** "Best practices" kan være outdated. Native multi-model databases challenge the polyglot paradigm. Context matters — what worked in 2015 may not be optimal in 2025.

---

### 4. Datakonsistens er ikke sexet, men kritisk

I starten af projektet så jeg ACID-transaktioner som "noget gammeldags fra bank-verdenen". Jeg ville hellere fokusere på AI-features.

Gennem analysen af fejlscenarier (DP4) indså jeg dog, at **konsistens er fundamentet for User Experience**. En chatbot, der glemmer svar, er ubrugelig.

**ACID vs BASE: Jeg Troede Transactions Var Langsomme**

Min største misconception var: *"NoSQL er hurtigere fordi eventual consistency = no transaction overhead."*

**Hvad jeg lærte:** Modern ACID implementation (PostgreSQL's MVCC) betyder writes don't block reads. Transaction overhead er ~1-2ms (negligible for chat workloads).

**Men eventual consistency cost:** 70% partial writes under crash scenarios. User ser "Why no response?" (broken UX). GDPR compliance risk (orphaned data).

Dette skift fra "Feature-first" til "Integrity-first" markerer en modning i min tilgang til systemudvikling.

**Takeaway:** Performance myths persist. ACID isn't slow — BASE is unpredictable. For user-facing applications, consistency > theoretical throughput gains.

---

## Opfyldelse af Læringsmål (Database & Storage)

| Læringsmål (fra definition) | Opfyldelse | Evidens | Refleksion |
|:---|:---|:---|:---|
| **Viden: Hybride Datamodeller** | Implementerede en samlet struktur med relationelle data (SQL), dokumenter (JSONB) og vektorer i én tabel. | Design Patterns (DP1+DP2) + Research (Kilde 1+2) | Jeg antog oprindeligt, at "Polyglot Persistence" var nødvendigt. Research viste dog, at PostgreSQLs binære JSONB-format muliggør hybride modeller uden performance-tab, hvilket forenkler arkitekturen markant. |
| **Viden: ACID vs. BASE** | Dokumenterede konkrete failure-scenarier. **Teoretisk analyse** viste, at eventual consistency resulterer i partial writes ved crashes (AWS dokumentation). | Design Patterns (DP4) + Validering (V4) | Jeg troede "NoSQL = hurtigere" pga. manglende transaktioner. Jeg lærte, at moderne MVCC eliminerer transaction overhead, og at BASE-modellens risiko for datatab er uacceptabel for chat-UX. |
| **Færdigheder: PostgreSQL & Vector** | Implementerede pgvector HNSW index for semantisk søgning og kombinerede metadata-filtrering med vektorer i én SQL-operation. | Implementation (Kodeeksempler) + Validering (V2) | Trade-off analysen var nøglen: pgvector er ~20% langsommere end Pinecone ved ren vectorsøgning, men 2.8× hurtigere ved kombinerede queries, da netværkskald elimineres. |
| **Færdigheder: JSONB & Indexering** | Udnyttede GIN-indeksering til at gøre ustrukturerede chat-logs søgbare med performance, der matcher MongoDB. | Research (OnGres Benchmark) + Design Patterns (DP2) | Jeg lærte forskellen på "Text-based JSON" og "Binary JSONB". Implementation quality (hvordan databasen parser data) er vigtigere end database-kategorien ("Document DB"). |
| **Kompetencer: Arkitektonisk Validering** | Evaluerede Total Cost of Ownership ($4,400 savings) og GDPR-compliance. Validerede "Unified Monolith" gennem triangulering af kilder. | Research (TCO Analysis) + Konceptuel Validering | Arkitektoniske valg er ikke kun tekniske, men også økonomiske og etiske. Jeg bevægede mig fra at vælge baseret på hype til at vælge baseret på evidens og konvergens mellem uafhængige kilder. |

---

## Samfundsmæssige Perspektiver

### GDPR Compliance: CASCADE DELETE som Design Principle

PostgreSQL's ACID transactions sikrer pålidelig implementation af "Right to Erasure" (Article 17) via CASCADE DELETE.

**Konkret:** 
```sql
DELETE FROM users WHERE id = $1;  -- Atomic removal af all related data
```

MongoDB's eventual consistency introducerer risiko for orphaned data i distributed replicas (100-500ms replication lag). Hvis crash sker under deletion, kan user data persist i secondary replicas.

**GDPR bliver designprincip, ikke compliance burden.** Atomicity er juridisk compliance by design.

---

### CO₂ Footprint: Storage Efficiency Matters (Green IT)

Et overraskende fund i researchen (Makris et al.) var, at MongoDB bruger op til **4× mere diskplads** end PostgreSQL for samme dataset.

**Beregning (10,000 users, 3 år):**
- MongoDB: 1.1 GB per user = 11 TB total
- PostgreSQL: 250 MB per user = 2.5 TB total
- **Difference:** 8.5 TB saved

**CO₂ impact (EU electricity mix ~0.3 kg CO₂/kWh):**
- HDD storage: ~0.006 kWh/GB/year
- 8.5 TB × 0.006 × 0.3 = **~15 kg CO₂ saved årligt**

Bedre query performance (26× faster) reducerer også CPU-forbrug per request. Aggregeret effekt: **~50 kg CO₂ savings årligt ved 10k users.**

Ved at vælge en storage-effektiv løsning reducerer jeg ikke bare hosting-regningen, men også løsningens miljøaftryk. **Arkitektonisk efficiency har miljøimpact. Effektiv kode er grøn kode.**

---

### Økonomisk Impact: TCO Analysis

Total cost of ownership fordel på **$4,400 over 3 år** (10k users scenario):

| Cost Category | PostgreSQL (Unified) | MongoDB + Pinecone (Polyglot) | Savings |
|---------------|---------------------|----------------------------|---------|
| **Hosting** | $50/month | $150/month | $100/month |
| **Monitoring** | $20/month | $60/month | $40/month |
| **Developer Time** | Baseline | +20% (workarounds) | Significant |
| **Total 3-year** | ~$2,500 | ~$6,900 | **$4,400** |

**Open-source model med community-driven extensions (pgvector) skaber konkurrence og lavere costs sammenlignet med vendor lock-in (Pinecone proprietary).**

Unified platform-trend reducerer krav til specialized skillsets, hvilket forenkler hiring og onboarding.

---

### Arbejdsmarked: PostgreSQL Skills Demand

PostgreSQL har 3× flere job postings end MongoDB i Danmark (LinkedIn data Q4 2024).

**Observation:** Unified platform-trend betyder virksomheder søger generalists med PostgreSQL + extensions, ikke specialists i MongoDB + Pinecone + Redis stack.

**Implikation:** Learning PostgreSQL + pgvector giver bredere employability end specialized tools.

---

## Afslutning: Fra Dilemmaer til Principper

Jeg startede med spørgsmålet: **"MongoDB til JSON, PostgreSQL til relationer, Pinecone til vectors — tre problemer, tre specialized tools?"**

Svaret er: **Nej. Unified platform outperformer polyglot persistence uden trade-offs.**

Database-valg bliver forkert når:
- Specialization assumptions ikke valideres empirisk
- Integration tax ignoreres (sync lag, roundtrips, failure points)
- Performance myths (ACID = slow) internaliseres uden test
- Marketing categories (document DB, vector DB) behandles som arkitektoniske garantier

Database-valg bliver korrekt når:
- Convergent evidence prioriteres over anekdoter
- Total cost of ownership evalueres (teknisk + økonomisk + developer experience)
- Validation methodology matcher claim type (literature convergence for performance, theory analysis for consistency)
- Trade-offs acknowledges (pgvector 20% langsommere end Pinecone for pure vector, men samlet løsning optimal)

**Personlig takeaway:** Database science er ikke kun performance benchmarks. Det er information architecture, consistency model analysis, developer psychology, juridisk compliance og økonomisk reasoning. De tekniske skills (SQL, indexing, transactions) var de nemme dele. De konceptuelle shifts (specialization ≠ performance, validation uden empiri, convergent evidence) var de udfordrende — og de vigtigste.

Den største læring har været skiftet fra **"Hvilken database er bedst?"** til **"Hvilke trade-offs er acceptable for min use case?"**

**Tilbage til udgangspunktet:** "Tre specialized tools" fungerer ikke når integration tax overgår specialization gains. "Unified platform" vinder når implementation quality beats marketing promises.

---

**Portfolio komplet:** Problemstilling → Research → Design Patterns → Konceptuel Validering → Konklusion dokumenterer systematic approach til Database & Storage uden live user data.

**Kritisk erkendelse:** Specialized tools eller unified platform? Det bestemmes af implementation quality og total cost of ownership, ikke marketing categories.