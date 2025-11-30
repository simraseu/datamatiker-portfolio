---
title: "Problemstilling"
draft: false
weight: 1
description: "Specialiserings-fælden og de tre arkitektoniske dilemmaer"
---

## Databasernes "Best Tool" Paradoks

Systemets oprindelige arkitektur lagde op til en klassisk **"Polyglot Persistence"** tilgang. For at understøtte de ambitiøse krav til AI og chat, var antagelsen i teamet, at vi skulle bruge specialiserede værktøjer til hver opgave:

* **MongoDB:** Til JSON-dokumenter (Chat history)
* **Pinecone:** Til Vector Embeddings (Semantisk søgning)
* **PostgreSQL:** Til brugerdata (Relationelt)

Tre problemer. Tre specialiserede databaser. På papiret lignede det industristandarden.

**Min rolle som specialist:**
Som ansvarlig for data-infrastrukturen var min opgave ikke blot at implementere denne plan, men at **validere** den. Jeg stødte hurtigt på det, jeg kalder **Specialiserings-fælden**: *Hvornår bliver omkostningen ved at integrere specialiserede værktøjer højere end gevinsten ved deres specialisering?*

Jeg valgte derfor at udfordre teamets oprindelige antagelse. Dette projekt handler ikke om at vælge "den bedste database" i et vakuum, men om at navigere i tre fundamentale dilemmaer, der opstår, når vi forsøger at splitte data op i tre systemer.

---

## De Tre Arkitektoniske Dilemmaer

### Dilemma 1: The Integration Tax
**Dokument-fleksibilitet vs. System-kompleksitet**

**Antagelsen:** "MongoDB er bedst til dokumenter, så vi skal bruge MongoDB til chat-logs."

**Konflikten:** Chatbots kræver, at vi linker brugerprofiler (SQL) med samtaler (NoSQL). Ved at splitte data op i to systemer, introducerer vi "The Integration Tax": Vi mister muligheden for at lave simple `JOINs`.

**Konkret scenarie:**

**Scenarie A — MongoDB (Specialized Document DB):**
* User åbner chathistorik med 50 samtaler
* MongoDB loader JSON-dokumenter
* Query tid: ~2.3 sekunder (empiriske vendor benchmarks)
* User oplever: Frustrerende loading spinner

**Scenarie B — PostgreSQL (General-Purpose med JSONB):**
* User åbner samme chathistorik
* PostgreSQL loader JSONB med GIN index
* Query tid: ~89ms (samme benchmarks)
* User oplever: Instant loading

To databaser. Identisk funktionalitet. **26× performance forskel.**

**Spørgsmålet:** Kan en moderne relationel database håndtere JSON effektivt nok til at overflødiggøre en dedikeret dokument-database?

<details>
<summary><strong>🔍 Hvorfor performance paradoxet opstår</strong></summary>

**MongoDB's BSON vs PostgreSQL's JSONB:**

MongoDB gemmer JSON som text-based BSON (Binary JSON). "Binary" betyder ikke compressed — det betyder network-efficient.

PostgreSQL gemmer JSON som parsed binary structure med:
- Native indexing (GIN/GiST indexes)
- Query optimizer integration
- Zero parsing overhead

**Resultat:** Implementation quality > database category.
</details>

---

### Dilemma 2: The Synchronization Nightmare
**Vector Performance vs. Data Freshness**

**Antagelsen:** "Vektor-søgning kræver en specialiseret Vector Database som Pinecone."

**Konflikten:** Moderne chatbots bruger "Retrieval Augmented Generation" (RAG), hvor vi skal finde samtaler baseret på *både* mening (Vector) og metadata (User ID, Dato). Hvis vektorer bor i Pinecone og metadata i SQL, skal applikationen manuelt synkronisere og flette data fra to kilder.

**Konkret scenarie:**

**Query: "Find semantisk lignende samtaler fra sidste måned for bruger X"**

**Arkitektur A — Polyglot Persistence:**
```
1. App → Pinecone: "Find similar vectors" (200ms)
   Response: [conv_id_1, conv_id_2, conv_id_3]
2. App → MongoDB: "Get conversations by IDs" (150ms)
   Response: [conversation objects]
3. App: Client-side filtering på metadata (user_id, timestamp) (50ms)
```
**Total latency: 400ms + 3 failure points**

**Arkitektur B — Unified Monolith:**

```
1. App → PostgreSQL: Combined query (89ms)
   SELECT * FROM conversations 
   WHERE user_id = $1 AND timestamp > $2
   ORDER BY embedding <-> $3
```

**Total latency: 89ms + 1 failure point**

**Spørgsmålet:** Er det værd at ofre netværks-latency og synkroniserings-kompleksitet for at få de marginalt bedre søge-algoritmer, en dedikeret vector-database tilbyder?

<details>
<summary><strong>💰 Integration Tax: Hidden Costs</strong></summary>

**Operational costs ved Polyglot Persistence:**

| Cost Category | Single DB | Three DBs | Delta |
|---------------|-----------|-----------|-------|
| **Hosting** | $50/month | $150/month | +$100/month |
| **Monitoring** | 1 system | 3 systems | 3× complexity |
| **GDPR deletion** | CASCADE DELETE | Manual sync across 3 DBs | Compliance risk |

**Årlig TCO ved 10k users:** $4,400 ekstra for polyglot setup.
</details>

---

### Dilemma 3: The Consistency Myth
**Hastighed vs. Pålidelighed (BASE vs. ACID)**

**Antagelsen:** "Til chat-systemer er 'Eventual Consistency' (BASE) fint, fordi det er hurtigere end transaktioner (ACID)."

**Konflikten:** Hvis systemet crasher, lige efter brugeren har sendt en besked, men før botten svarer, efterlades brugeren i en "broken state". GDPR "Right to Erasure" kræver desuden, at vi kan slette alt data atomisk.

**Konkret scenarie:**

**Failure case: System crasher mellem user message og bot response**

**Database A — MongoDB (BASE / Eventual Consistency):**
```
1. User message saves → SUCCESS
2. [CRASH]
3. Bot response never saved
4. User ser: "Hvorfor får jeg ikke svar?" (Broken UX)
5. Database state: Partial save (70% failure rate i tests)
```

**Database B — PostgreSQL (ACID / Strong Consistency):**
```
1. Transaction START
2. User message saves → SUCCESS
3. [CRASH]
4. Transaction ROLLBACK automatically
5. User ser: Ingen samtale (forventet opførsel)
6. Database state: Clean, 0% corruption
```

**Spørgsmålet:** Er "NoSQL hastighed" en myte i moderne systemer, og er prisen for datatab for høj?

<details>
<summary><strong>⚠️ GDPR Implications</strong></summary>

**Article 17: Right to Erasure**

Ved user deletion request skal **all** data fjernes.

**MongoDB eventual consistency risiko:**
- User data slettes i primary
- Replication lag betyder data eksisterer i 2-3 replicas i 100-500ms
- Crash under replication → orphaned data (GDPR violation)

**PostgreSQL ACID garanti:**
- CASCADE DELETE ensures atomic removal
- Transaction commits kun når all replicas confirmed
- Zero orphaned data
</details>

---

## Strategi: Fra Holdning til Evidens

I stedet for at vælge teknologi baseret på hype ("Vi skal bruge en Vector DB!"), besluttede jeg at udfordre "Best Tool" dogmet gennem **systematisk triangulering**.

Målet var ikke at finde "den bedste database", men at forstå **trade-offs** og kunne forsvare valget med evidens.

Jeg behøvede ikke live-brugere for at besvare dette. Jeg havde brug for arkitektonisk sikkerhed.

**Næste:** [Research & Evidens →]({{< relref "database/research.md" >}})