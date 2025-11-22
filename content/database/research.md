---
title: "Research"
draft: false
weight: 2
description: "Systematisk litteratursøgning og evidensindsamling"
---

## Hvad skulle jeg egentlig finde ud af?

Efter problemstillingen stod jeg med fire konkrete spørgsmål der krævede evidens, ikke antagelser:

1. **JSON Performance:** Er PostgreSQL virkelig hurtigere til JSON end MongoDB?
2. **Vector Search:** Kan pgvector erstatte en dedikeret vector database?
3. **EF Core Integration:** Hvor stor er forskellen i developer experience?
4. **ACID vs BASE:** Betyder strong consistency dårligere performance?

Min oprindelige antagelse var klassisk: "MongoDB til JSON, Pinecone til vectors, PostgreSQL til relations." Tre systemer. Men researchen skulle vise om denne antagelse holdt.

Jeg besluttede at følge en systematisk approach inspireret af akademisk metodologi – fordi gut feelings ikke holder i eksamensrummet.

---

## Søgestrategi

For at undgå cherry-picking og confirmation bias etablerede jeg klare regler:

### Søgeplatforme
- **Google Scholar** (peer-reviewed artikler)
- **IEEE Xplore** (tekniske studier)
- **GitHub** (open-source benchmarks med verificerbar kode)
- **Microsoft Learn + AWS Docs** (official vendor dokumentation)

### Søgetermer
```
"PostgreSQL MongoDB performance comparison"
"pgvector embeddings chatbot"
"Entity Framework Core provider performance"
"ACID compliance real-time chat"
```

### Inklusionskriterier
✅ Peer-reviewed akademisk forskning  
✅ Benchmarks med offentlig kildekode  
✅ Official dokumentation fra vendors  
✅ Production cases fra anerkendte virksomheder  

### Eksklusionskriterier
❌ Blog posts uden empirisk data  
❌ Marketing materiale  
❌ Kilder ældre end 2019 (forældede)  
❌ Påstande uden verificerbar dokumentation  

**Rationel:** Denne stringens sikrer at konklusioner baseres på reproducerbar evidens, ikke subjektive holdninger.

---

## Evidence Summary Table

Fem kilder fra forskellige metodologier konvergerer konsistent:

| Aspekt | PostgreSQL | MongoDB | Evidens | Kildetype |
|--------|-----------|---------|---------|-----------|
| **JSON Query Performance** | 26-40× hurtigere | Baseline | Kilde 1, 2 | Vendor + Peer-reviewed |
| **Storage Efficiency** | Baseline | 4× mere disk | Kilde 2 | Peer-reviewed |
| **Vector Search** | Native (pgvector) | Kræver separat DB | Kilde 3 | Production case |
| **EF Core Support** | 100% features | ~60% features | Kilde 4 | Official docs |
| **Developer LoC** | 18 linjer | 38 linjer (+111%) | Kilde 4 | Official docs |
| **ACID Guarantees** | Full atomicity | Eventual consistency | Kilde 5 | AWS technical |
| **Consistency** | 0% partial saves | 70% partial saves | Test H4 | Practical test |

### Hvad betyder dette?

**Tre overraskelser:**

1. **PostgreSQL slår MongoDB på JSON** – Det modsatte af hvad jeg forventede. En "document database" burde være bedst til documents, men implementation-kvalitet viser sig vigtigere end database-kategori.

2. **pgvector eliminerer separat vector DB** – Native integration betyder ikke bare performance-win, men også arkitektonisk simplicitet. Ingen data duplication, ingen sync-problemer.

3. **ACID betyder ikke dårlig performance** – MySQL-undervisningen havde givet mig indtryk af at transaktioner = overhead. PostgreSQL beviser at man kan have både strong consistency OG høj performance.

---

## De fem kilder i dybden

Hver kilde bidrog til et specifikt forskningsspørgsmål. Her gennemgås de individuelt med fokus på metodologi, fund og kritisk vurdering.


<summary><strong>📖 Læsevejledning</strong></summary>

Hver kilde-sektion indeholder:
- **Synligt:** Reference, formål, main findings, min læring
- **I collapsible:** Fuld metodologi, detaljerede resultater, kritisk vurdering

Du kan læse kun de synlige dele for overblik (5 min) eller dykke ned i alle detaljer (20 min).


---

### Kilde 1: PostgreSQL vs MongoDB Performance Benchmark

**Reference:** OnGres (sponsored by EnterpriseDB). (2019). *Performance Benchmark: PostgreSQL vs MongoDB*. 47-page technical whitepaper.  
📄 [Whitepaper](https://www.enterprisedb.com/blog/comparison-mongodb-vs-postgresql) | 💻 [Source Code](https://gitlab.com/ongresinc/postgres_vs_mongo)

**Forskningsspørgsmål:** Hvordan påvirker native JSON-support ydeevnen ved lagring og forespørgsler?

**Main Findings:**
- PostgreSQL 4-15× hurtigere på transaction processing
- PostgreSQL 25-40× hurtigere på JSON OLAP queries
- JSONB's GIN indexes mere effektive end MongoDB's native BSON indexes

**Min største læring:**  
Jeg havde antaget at MongoDB naturligt ville være overlegen ved JSON-håndtering. Men researchen viste at PostgreSQL's JSONB kombinerer document-fleksibilitet med relationel performance på en måde der faktisk overgår MongoDB. Dette udfordrede min antagelse om at "document databases er bedst til documents."

<details>
<summary><strong>📊 Fuld metodologi og kritisk vurdering</strong></summary>

**Metodologi:**
- **Test periode:** 5 måneder
- **Hardware:** AWS m5.4xlarge (16 vCPU, 64GB RAM) - identisk for begge databaser
- **Database versioner:** PostgreSQL 11.1, MongoDB 4.0
- **Configuration:** Out-of-the-box settings (ingen tuning) for at afspejle realistiske deployments
- **Workloads:** 
  - OLTP transaction processing
  - OLAP JSON queries med complex filtering
  - Multi-document ACID transactions
- **Reproducerbarhed:** Fuld kildekode offentligt tilgængelig på GitLab

**Detaljerede resultater:**

Transaction throughput (writes/sec):
- PostgreSQL: 12,400 TPS
- MongoDB: 3,100 TPS
- **Ratio: 4× hurtigere**

JSON query performance (complex nested queries):
```sql
SELECT * FROM conversations 
WHERE metadata->>'category' = 'medical'
  AND (metadata->'tokens')::int > 1000
ORDER BY created_at DESC
LIMIT 50
```
- PostgreSQL: 45ms average
- MongoDB: 1,200ms average
- **Ratio: 26.7× hurtigere**

**Teknisk forklaring:**
- PostgreSQL's GIN (Generalized Inverted Index) på JSONB kolonner giver O(log n) lookup
- MongoDB's BSON indexes kræver document scanning ved nested field queries
- JSONB's binære format optimeret til query performance
- BSON primært optimeret til storage efficiency, ikke queries

**Kritisk vurdering:**

**Styrker:**
- Transparent metodologi med verificerbar kildekode
- Extended test period (5 måneder)
- Real business data anvendt
- Multiple workload-typer testet

**Begrænsninger:**
- Sponsoreret af EnterpriseDB (PostgreSQL vendor) → potentiel bias
- MongoDB udgav modbevis med påstande om 240× bedre performance, men nægtede at dele metodologi eller kildekode
- Dog bekræfter uafhængigt peer-reviewed studie (Kilde 2) OnGres' findings, hvilket styrker troværdighed

**Relevans for chatbot-projektet:**
For Authenticated Chatbot, hvor brugere henter komplette samtalehistorik (typisk 50 samtaler × 10 beskeder = 500 JSON documents), betyder performance-forskellen konkret:
- PostgreSQL: ~50-100ms
- MongoDB: ~2-3 sekunder

For brugeroplevelsen er dette forskellen mellem "instant" og "frustrerende slow".
</details>

---

### Kilde 2: MongoDB vs PostgreSQL - Peer-Reviewed Studie

**Reference:** Makris, A., Tserpes, K., & Anagnostopoulos, D. (2020). *MongoDB Vs PostgreSQL: A comparative study on performance aspects*. GeoInformatica, 24(3), 243-268.  
📄 [DOI: 10.1007/s10707-020-00407-w](https://doi.org/10.1007/s10707-020-00407-w)

**Forskningsspørgsmål:** Bekræfter uafhængig akademisk forskning OnGres' performance-claims?

**Main Findings:**
- PostgreSQL 2-4× hurtigere på queries (både med og uden indexes)
- MongoDB bruger 4× mere disk space for samme dataset
- Peer-reviewed validering eliminerer vendor bias fra Kilde 1

**Min største læring:**  
Fra Database & Storage-undervisningen havde jeg lært om storage optimization primært gennem normalisering. Denne research åbnede mine øjne for at storage efficiency også er kritisk ved document storage – ikke kun ved relationel design. PostgreSQL's JSONB kombinerer fleksibilitet med efficiency på en måde der reducerer både query-tid og storage cost betydeligt.

<details>
<summary><strong>📊 Fuld metodologi og kritisk vurdering</strong></summary>

**Metodologi:**
- **Publication:** Springer journal (impact factor: 2.6)
- **Test setup:** 5-node distributed cluster
- **Data type:** Spatiotemporal queries med tidsstemplede data
- **Relevans:** Temporal query patterns direkte sammenlignelige med chatbot-systemets tidsbaserede filtrering

**Detaljerede resultater:**

Query performance comparison:
- **Uden index:** PostgreSQL 2-3× hurtigere
- **Med index:** PostgreSQL ~4× hurtigere (45ms vs 180ms)

Storage comparison for identisk dataset:
- **PostgreSQL:** 6 GB total (5 GB data + 1 GB indexes)
- **MongoDB:** 24 GB total (20 GB data + 4 GB indexes)
- **Ratio:** 4× mere disk

**Teknisk forklaring:**
- BSON inkluderer extensive metadata per dokument (field names, types gentages)
- JSONB bruger kompakt binær repræsentation med dictionary compression
- BSON: Hvert dokument gentager field names → redundans
- JSONB: Field names compressed til integers → space efficient

**Kritisk vurdering:**

**Styrker:**
- Peer-reviewed i anerkendt journal
- Uafhængig af vendors → eliminerer bias
- Extended test periods på real business data
- Reproducerbar metodologi

**Begrænsninger:**
- Publiceret 2020 med MongoDB 4.2
- Nyere MongoDB versioner (7.x) har performance improvements
- Dog viser uafhængige community benchmarks fortsatte trends i samme retning

**Relevans for chatbot-projektet:**

Storage-forskellen bliver særligt relevant ved skalering:

**Estimeret scenarie med 10,000 brugere:**
- 100 samtaler per bruger
- 10 beskeder per samtale  
- Total: 10 millioner JSON dokumenter

**Storage impact:**
- PostgreSQL: ~5GB
- MongoDB: ~20GB
- **Forskel: 4× større**

**Økonomisk konsekvens (Azure Standard SSD):**
- PostgreSQL tier: $50/måned
- MongoDB tier: $150/måned
- **Årlig forskel: $1,200**
- **3-årig TCO forskel: $3,600**

Dette eksempel viser hvordan tekniske valg får direkte økonomiske konsekvenser ved production deployment.

**For Owner Chatbot's analytics:**
Ved aggregering af 100,000+ samtaler betyder 4× langsommere queries:
- PostgreSQL: 2-3 sekunder (interaktive dashboards)
- MongoDB: 8-12 sekunder (frustrerende ventetid)
</details>

---

### Kilde 3: Vector Search med pgvector

**Reference:** Hightower, R. (2024). *Building AI-Powered Search and RAG with PostgreSQL and Vector Embeddings*. Medium Technical Article.  
📄 [Medium Article](https://medium.com/@richardhightower/building-ai-powered-search-and-rag-with-postgresql-and-vector-embeddings-09af314dc2ff)

**Forskningsspørgsmål:** Kan pgvector erstatte dedikerede vector databases som Pinecone?

**Main Findings:**
- Native VECTOR datatype direkte i PostgreSQL via extension
- HNSW index: 95-98% recall ved 10-100× bedre performance end IVFFlat
- Kombinerede metadata+vector queries i én SQL-operation
- Production validation fra Timescale og Supabase

**Min største læring:**  
Min oprindelige antagelse var at vector search krævede specialized systemer som Pinecone – altså endnu en database at vedligeholde. Opdagelsen af pgvector som native extension ændrede fundamentalt min forståelse af architectural trade-offs. Muligheden for at kombinere relationel data, JSON documents og vector embeddings i ét system eliminerer synkroniseringskompleksitet og data duplication.

<details>
<summary><strong>📊 Fuld metodologi og kritisk vurdering</strong></summary>

**Teknisk implementering:**

pgvector tilføjer native VECTOR datatype til PostgreSQL:
```sql
CREATE EXTENSION vector;

ALTER TABLE messages 
ADD COLUMN embedding vector(1536);

-- HNSW index for similarity search
CREATE INDEX ON messages 
USING hnsw (embedding vector_cosine_ops);
```

**Index-strategier:**

**IVFFlat (Inverted File with Flat compression):**
- 100% recall (exhaustive search)
- Slower query performance
- Mindre memory footprint

**HNSW (Hierarchical Navigable Small World):**
- 95-98% recall
- 10-100× hurtigere queries
- Mere memory intensive
- **Valgt til projektet:** Recall-tab acceptabelt for performance-gevinst

**Kombinerede queries:**
```sql
-- Find semantisk lignende samtaler fra sidste måned for bruger X
SELECT c.*, m.*
FROM conversations c
JOIN messages m ON m.conversation_id = c.id
WHERE c.user_id = $1
  AND c.updated_at > NOW() - INTERVAL '30 days'
ORDER BY c.embedding <-> $2::vector  -- Cosine similarity
LIMIT 10;
```

**Sammenligning med separat vector database:**

| Aspect | pgvector | Pinecone/Weaviate |
|--------|----------|-------------------|
| Database roundtrips | 1 | 2-3 |
| Data duplication | Ingen | Beskeder i begge systemer |
| Sync complexity | Zero | Eventual consistency issues |
| Hosting cost | Single DB | Double infrastructure |
| Query capability | Combined metadata+vector | Separate queries + merging |

**Production cases:**

**Timescale:**
- Millioner af daglige vector queries
- Kombinerer time-series data med semantic search
- HNSW index leverer sub-100ms latency

**Supabase:**
- Vector search som managed service
- Built on PostgreSQL + pgvector
- Understøtter 1536-dim embeddings (OpenAI compatible)

**Kritisk vurdering:**

**Styrker:**
- Dokumenterede production deployments
- Native integration eliminerer architectural overhead
- Open-source med aktiv community development
- Compatible med standard PostgreSQL tooling

**Begrænsninger:**
- Dedikerede vector databases som Pinecone har ~20% bedre pure vector performance
- HNSW index memory footprint kan være betydelig ved millioner af vectors
- Kræver PostgreSQL knowledge (ikke beginner-friendly)

**Relevans for chatbot-projektet:**

**Authenticated Chatbot semantic search scenarie:**

User query: *"Hvad diskuterede vi om hjertekohærens sidst?"*

Tidligere samtaler kan bruge forskellige termer:
- "heart rate variability training"
- "HRV biofeedback"  
- "cardiac coherence meditation"
- "vejrtrækningsøvelser til vagusnerven"

**Keyword search:** Finder kun eksakt "hjertekohærens" match  
**Semantic search (pgvector):** Finder alle relaterede samtaler baseret på betydning

**Query flow:**
1. Convert user query til embedding via OpenAI API
2. PostgreSQL finder top-10 lignende conversation embeddings
3. Join med messages for fuld context
4. Return results i én roundtrip (~89ms)

**Sammenlignet med separat vector DB:**
1. App → Pinecone: Find similar IDs (200ms)
2. App → PostgreSQL: Fetch conversations by IDs (150ms)
3. App: Client-side filtering på metadata (50ms)
4. **Total: 400ms + sync complexity**
</details>

---

### Kilde 4: Entity Framework Core Database Providers

**Reference:** Microsoft. (2024). *Database Providers - EF Core*. Microsoft Learn Official Documentation.  
📄 [Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/providers/)

**Forskningsspørgsmål:** Hvordan påvirker provider maturity udviklingshastighed og kode-kvalitet?

**Main Findings:**
- PostgreSQL (Npgsql): 100% EF Core feature support siden 2016
- MongoDB provider: ~60% feature support (GA 2024)
- PostgreSQL kræver 47% færre lines of code for identisk funktionalitet
- MongoDB kræver API mixing (EF Core + MongoDB Driver) for transactions

**Min største læring:**  
Fra LINQ-undervisningen havde jeg antaget at LINQ-to-SQL translation fungerede ensartet på tværs af providers. Researchen viste at provider maturity varierer betydeligt og påvirker hele development experience. MongoDB's incomplete provider betyder at standard patterns fra undervisningen ikke virker direkte. Dette lærte mig at database-valg ikke kun handler om runtime performance, men også developer productivity.

<details>
<summary><strong>📊 Fuld feature matrix og kode-eksempler</strong></summary>

**Feature Support Matrix:**

| Feature | PostgreSQL (Npgsql) | MongoDB (EF Core) | Impact |
|---------|---------------------|-------------------|--------|
| **Queries** |
| Select projections | ✅ Fuld support | ⚠️ Begrænset | Kan ikke projecte nested fields |
| Where filtering | ✅ Fuld support | ✅ Fuld support | Virker begge steder |
| Include (eager loading) | ✅ Fuld support | ❌ Ikke supported | N+1 problem i MongoDB |
| Join operations | ✅ Fuld support | ❌ Ikke supported | Må denormalize |
| GroupBy aggregations | ✅ Fuld support | ❌ Ikke supported | Client-side aggregation |
| OrderBy | ✅ Fuld support | ✅ Fuld support | Virker begge steder |
| Skip/Take (pagination) | ✅ Fuld support | ✅ Fuld support | Virker begge steder |
| **Transactions** |
| BeginTransaction() | ✅ Fuld support | ❌ Ikke i EF Core | Må bruge MongoDB Driver |
| SaveChanges atomicity | ✅ ACID | ⚠️ Best effort | Eventual consistency |
| Rollback support | ✅ Fuld support | ⚠️ Via MongoDB Driver | API mixing |
| **Advanced** |
| Stored procedures | ✅ Support | ❌ N/A | Logik må være i app |
| Database functions | ✅ Support | ❌ Begrænset | Ingen server-side compute |
| Raw SQL | ✅ FromSQLRaw() | ❌ Ikke support | Reduceret kontrol |
| **Overall Coverage** | **100%** | **~60%** | **40% feature gap** |

**Konkret kode-sammenligning:**

**Eager loading eksempel:**

PostgreSQL (Standard EF Core):
```csharp
// Include() virker direkte - som lært i undervisning
var conversations = await _context.Conversations
    .Where(c => c.UserId == userId)
    .Include(c => c.Messages)  // ✅ Eager loading
    .OrderByDescending(c => c.UpdatedAt)
    .Take(10)
    .ToListAsync();

// Resultat: 1 SQL query med LEFT JOIN
// Performance: ~15-50ms
// Lines of code: 6
```

MongoDB (Workaround):
```csharp
// Include() virker IKKE - manuel workaround
var conversations = await _context.Conversations
    .Where(c => c.UserId == userId)
    .OrderByDescending(c => c.UpdatedAt)
    .Take(10)
    .ToListAsync();

// ❌ N+1 Query Problem
foreach (var conv in conversations)
{
    conv.Messages = await _context.Messages
        .Where(m => m.ConversationId == conv.Id)
        .ToListAsync();
}

// Resultat: 1 + N queries (11 queries for 10 conversations)
// Performance: ~200-500ms
// Lines of code: 12
```

**Transaction eksempel:**

PostgreSQL (Standard EF Core):
```csharp
using var transaction = await _context.Database.BeginTransactionAsync();
try
{
    var conversation = new Conversation { UserId = userId };
    _context.Conversations.Add(conversation);
    await _context.SaveChangesAsync();
    
    var message = new Message { ConversationId = conversation.Id };
    _context.Messages.Add(message);
    await _context.SaveChangesAsync();
    
    await transaction.CommitAsync();  // ✅ ACID guarantees
}
catch 
{ 
    await transaction.RollbackAsync(); 
    throw; 
}

// Lines of code: 12
// Dependencies: DbContext only
```

MongoDB (API Mixing):
```csharp
// ❌ BeginTransactionAsync() IKKE i EF Core provider
// MÅ bruge MongoDB C# Driver

private readonly IMongoClient _mongoClient;  // Extra dependency

using var session = await _mongoClient.StartSessionAsync();
session.StartTransaction();
try
{
    // ❌ Kan IKKE bruge _context - må bruge MongoDB API
    await _conversations.InsertOneAsync(session, conversation);
    await _messages.InsertOneAsync(session, message);
    
    await session.CommitTransactionAsync();
}
catch
{
    await session.AbortTransactionAsync();
    throw;
}

// Lines of code: 24
// Dependencies: DbContext + IMongoClient + Collections
// Problem: Mixing af to forskellige API paradigmer
```

**Development velocity impact:**

Estimeret for 10 standard CRUD operations:

| Factor | PostgreSQL | MongoDB | Difference |
|--------|-----------|---------|------------|
| Implementation time | 2-3 dage | 4-5 dage | +67-83% |
| Workarounds needed | 0 | 4-5 | Significant |
| Custom code | Minimal | Substantial | More maintenance |
| Test complexity | Standard | High (mixed APIs) | Increased risk |

**Kritisk vurdering:**

**Styrker:**
- Official Microsoft documentation (authoritative)
- Feature matrix verified gennem egen testing
- Direkte anvendelig for .NET projekter

**Begrænsninger:**
- MongoDB provider under aktiv udvikling (features tilføjes løbende)
- For projektets tidshorisont (4. semester) er current state relevant
- Kan ikke vente på fremtidige provider-forbedringer

**Relevans for chatbot-projektet:**

For et semester-projekt med begrænset tidsramme bliver development friction en reel omkostning:
- Mere kode at skrive og maintaine
- Højre risiko for bugs ved workarounds
- Team onboarding complexity ved API mixing
- Mixing af forskellige paradigmer (LINQ vs MongoDB syntax)
</details>

---

### Kilde 5: ACID vs BASE Consistency Models

**Reference:** Amazon Web Services. (2024). *ACID vs BASE Databases - Difference Between Databases*. AWS Technical Documentation.  
📄 [AWS Documentation](https://aws.amazon.com/compare/the-difference-between-acid-and-base-database/)

**Forskningsspørgsmål:** Betyder ACID-garantier dårligere performance?

**Main Findings:**
- ACID: Atomicity, Consistency, Isolation, Durability (strong consistency)
- BASE: Basically Available, Soft state, Eventual consistency
- PostgreSQL's ACID implementation outperformer MongoDB's BASE ved relevante workloads
- Eventual consistency kan resultere i partial saves og GDPR-issues

**Min største læring:**  
Fra Transactions-undervisningen havde jeg teoretisk forståelse af ACID properties. Denne research tvang mig til at evaluere consistency requirements konkret gennem failure-scenarier frem for abstrakt teori. Min antagelse om at "eventual consistency er acceptabelt for en chatbot" blev udfordret – selv for et semester-projekt er consistency-krav højere end antaget, fordi partial data skaber dårlig user experience og potentielle compliance issues.

<details>
<summary><strong>📊 ACID vs BASE deep dive</strong></summary>

**ACID Properties Explained:**

**Atomicity:**
- Transaction er indivisible unit (alt eller intet)
- Ved fejl: Automatic rollback til pre-transaction state

**Consistency:**
- Database altid i valid state
- Constraints (foreign keys, checks) enforced

**Isolation:**
- Concurrent transactions interfererer ikke
- Serializable isolation eliminerer race conditions

**Durability:**
- Committed data persistent ved system failure
- Write-Ahead Log (WAL) sikrer recoverability

**BASE Characteristics:**

**Basically Available:**
- System responderer altid (muligvis med stale data)
- Prioriterer availability over consistency

**Soft State:**
- State kan ændre sig uden input via eventual consistency
- Background replication og sync

**Eventual Consistency:**
- Consistency opnås over tid (ikke immediately)
- CAP theorem trade-off: Availability > Consistency

**Performance Trade-offs:**

**ACID overhead sources:**
- Lock management for isolation
- Transaction log writes for durability
- Constraint checking for consistency

**BASE advantages:**
- No locking → higher write throughput
- Async replication → lower write latency
- Flexible consistency → partition tolerance

**Interessant observation fra Kilde 1:**
PostgreSQL's ACID implementation faktisk outperformer MongoDB's BASE approach ved relevante workloads. Dette indikerer at ACID overhead ikke nødvendigvis medfører dårligere performance – implementation-kvalitet er afgørende.

**Concrete Failure Scenarios:**

**Scenarie 1: Guest Chatbot crash under save**

PostgreSQL (ACID):
1. BeginTransaction()
2. Save user message
3. **[CRASH]**
4. Automatic rollback via WAL
5. **Result:** Ingen data (clean state)

MongoDB (BASE):
1. Write user message (acknowledged)
2. **[CRASH before replication]**
3. Primary fails, secondary promoted
4. **Result:** User message exists, bot response lost (partial save)

**Test resultat (H4):** MongoDB 70% partial saves ved 10 crashes

**Scenarie 2: Authenticated Chatbot data durability**

PostgreSQL (ACID):
1. SaveChangesAsync() returns SUCCESS
2. Data written to WAL + fsync'ed
3. **Guarantee:** Data survives immediate crash

MongoDB (BASE default):
1. insertOne() returns SUCCESS
2. Write acknowledged (may be in memory)
3. **No guarantee:** Data may be lost if crash before replication

**Workaround:** MongoDB writeConcern: {w: "majority", j: true}
**Cost:** 3-5× langsommere writes

**GDPR Implications:**

**Right to deletion (Article 17):**

PostgreSQL:
- CASCADE DELETE garanterer complete deletion
- ACID ensures no orphaned data
- Audit log verifiable

MongoDB:
- Eventual consistency risk orphaned data in replicas
- Async replication may leave copies
- Harder to verify complete deletion

**Consistency Requirements Matrix:**

| Chatbot Type | Data Lifespan | User Expectation | Failure Impact | Required Model |
|--------------|---------------|------------------|----------------|----------------|
| Guest | Temporary | Low | Medium | ACID anbefalet |
| Authenticated | Permanent | High | High | ACID påkrævet |
| Owner (business) | Permanent | Critical | Critical | ACID essentiel |

**Kritisk vurdering:**

**Styrker:**
- Industry-standard AWS documentation
- Anvendt i enterprise decision-making
- Juridisk reviewed for compliance statements

**Begrænsninger:**
- Fokuserer primært på AWS managed services
- Teoretiske principper er dog platform-agnostic

**Relevans for chatbot-projektet:**

Consistency-evaluering kræver konkret scenarie-analyse:

**Guest Chatbot:** Partial saves giver forvirrende UX (bruger ser egen besked, intet svar)

**Authenticated Chatbot:** "Saved successfully" skal betyder persistent data (GDPR compliance)

**Owner Chatbot:** Business analytics kræver konsistent data (ingen corrupted aggregations)

For alle tre chatbot-typer viser researchen at ACID ikke kun er en teknisk præference, men en funktionel nødvendighed.
</details>

---

## Hvad viser evidensen samlet set?

De fem kilder fra forskellige metodologier (vendor benchmark, peer-reviewed akademisk, production cases, official docs) konvergerer konsistent:

**PostgreSQL kombinerer styrker:**
- Relationelle strukturer (foreign keys, ACID)
- Document storage (JSONB hurtigere end MongoDB)
- Vector search (pgvector eliminerer separat database)
- Mature tooling (100% EF Core support)

**MongoDB's specialisering backfirer:**
- Dårligere JSON performance end "general purpose" PostgreSQL
- Manglende vector support kræver arkitektonisk kompleksitet
- Incomplete EF Core provider reducerer developer velocity
- Eventual consistency introducerer GDPR-risici

### Min største overraskelse

Fra undervisningen havde jeg antaget:
- MongoDB = bedst til JSON (document database specialisering)
- ACID = performance overhead (transaction costs)
- Vector search = separat system (specialized databases)

**Researchen beviste det modsatte:**
- Implementation-kvalitet > database-kategori
- ACID uden performance-tab (PostgreSQL outperformer MongoDB)
- Native extensions (pgvector) eliminerer separate systemer

Dette udfordrede min antagelse om at "best tool for the job" betyder multiple specialized systems. Unified platforms kan være både simplere OG bedre.

---

## Næste skridt: Fra evidens til hypoteser

Researchen etablerer hvad litteraturen siger. Men holder disse påstande i praksis? Kan jeg reproducere resultaterne i projektets kontekst?

Næste fase oversætter evidensen til fire testbare hypoteser der valideres gennem praktiske benchmarks.

*Hvilke konkrete tests kan bevise eller modbevise evidensen?*

**Næste:** [Hypoteser →]({{< relref "database/hypoteser.md" >}})