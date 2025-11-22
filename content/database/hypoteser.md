---
title: "Hypoteser"
draft: false
weight: 3
description: "Fra evidens til testbare påstande"
ShowToc: true
TocOpen: true
---

## Hvordan tester jeg evidensen i praksis?

Researchen gav mig fem kilder med konsistente konklusioner. Men litteratur er én ting – praksis er en anden. Virker PostgreSQL's fordele også i min konkrete chatbot-arkitektur? Med mine data? På min hardware?

Det er tid til at oversætte evidensen til fire konkrete hypoteser jeg kan teste empirisk.

---

## Tværgående mønstre i evidensen

Før jeg formulerer hypoteser, hvad viser kilderne samlet set?

### Mønster 1: PostgreSQL transcenderer kategorier

**Fra kilderne:**
- Kilde 1: PostgreSQL 26-40× hurtigere på JSON end MongoDB
- Kilde 2: PostgreSQL bruger 4× mindre disk
- Kilde 3: pgvector eliminerer separat vector database

**Hvad det betyder:**  
PostgreSQL er ikke længere "bare" en relationel database. JSONB + pgvector + ACID gør den til en unified platform der kombinerer styrker fra multiple database-typer uden deres svagheder.

---

### Mønster 2: Specialisering garanterer ikke performance

**Fra kilderne:**
- MongoDB markedsføres som "document database"
- Men PostgreSQL's JSONB outperformer MongoDB's native BSON
- Implementation-kvalitet > database-kategori

**Hvad det betyder:**  
"Best tool for the job" er ikke altid det specialized tool. Nogle gange vinder general-purpose tools med exceptionel implementation.

---

### Mønster 3: Developer experience matters lige så meget som runtime performance

**Fra kilderne:**
- Kilde 4: MongoDB kræver 111% mere kode for identisk funktionalitet
- EF Core workarounds introducerer N+1 queries
- API mixing (EF Core + MongoDB Driver) øger complexity

**Hvad det betyder:**  
Database-valg påvirker ikke kun hvor hurtigt systemet kører, men hvor hurtigt jeg kan bygge det. For et semester-projekt med deadline er development velocity kritisk.

---

### Mønster 4: ACID betyder ikke overhead

**Fra kilderne:**
- Kilde 1: PostgreSQL (ACID) outperformer MongoDB (BASE)
- Kilde 5: Strong consistency uden performance-tab
- Test H4: MongoDB 70% partial saves ved crashes

**Hvad det betyder:**  
Min antagelse om at "transactions = slow" var forkert. Modern ACID implementation kan være både hurtigere OG mere pålidelig end eventual consistency.

---

## Fire testbare hypoteser

Baseret på disse mønstre formulerer jeg fire hypoteser der direkte adresserer projektets behov:

## Hypotese 1: JSON Performance

**Påstand:**  
PostgreSQL's JSON queries vil være mindst 20× hurtigere end MongoDB ved loading af Authenticated Chatbot's samtalehistorik.

**Evidens-grundlag:**
- Kilde 1: 25-40× performance-fordel dokumenteret
- Kilde 2: Peer-reviewed bekræftelse (4× hurtigere queries)

**Hvorfor 20× threshold?**  
Konservativt estimat sikrer at hypotesen kræver konsistent, markant forskel – ikke tilfældige udsving.

### Testdesign

**Målepunkt:** Query execution time (millisekunder)

**Dataset:**
- 50 samtaler per bruger
- 10 beskeder per samtale
- Total: 500 JSON documents

**Operation:**
```sql
SELECT * FROM conversations 
WHERE user_id = X 
INCLUDE messages 
ORDER BY updated_at DESC
```

**Hardware:** Identisk testmiljø for begge databaser

**Success-kriterium:**  
```
PostgreSQL query time < MongoDB query time / 20
```

**Hvis hypotesen validates:** Bekræfter at JSONB outperformer BSON i projektets kontekst  
**Hvis hypotesen falsificeres:** Indikerer at litteratur-resultater ikke holder ved min workload

---

## Hypotese 2: Vector Search Integration

**Påstand:**  
pgvector kan udføre kombinerede metadata+vector queries i én SQL-operation uden client-side merging.

**Evidens-grundlag:**
- Kilde 3: Production validation fra Timescale + Supabase
- pgvector HNSW index kombinerer filtering med similarity search

**Hvorfor denne test matters:**  
Separat vector database ville kræve minimum 2 roundtrips + client-side merging. Native integration skulle eliminere denne kompleksitet.

### Testdesign

**Målepunkt:** Antal database roundtrips

**Operation:**  
"Find semantisk lignende samtaler fra sidste måned for bruger X"

**Filtrering på:**
- `user_id` (metadata)
- `timestamp` (metadata)
- `vector similarity` (embeddings)

**Sammenligning:**

**Approach A - pgvector:**
```sql
SELECT * FROM conversations
WHERE user_id = $1
  AND updated_at > NOW() - INTERVAL '30 days'
ORDER BY embedding <-> $2::vector
LIMIT 10
```
Roundtrips: **1**

**Approach B - Separat vector DB:**
```
1. App → Pinecone: Search vectors (return IDs)
2. App → PostgreSQL: Fetch conversations by IDs
3. App: Client-side filtering på metadata
```
Roundtrips: **2+** (plus client-side processing)

**Success-kriterium:**  
pgvector udfører operation i **1 database roundtrip** med kombineret WHERE clause og ORDER BY vector distance.

**Hvis hypotesen validates:** Bekræfter at pgvector eliminerer behovet for separat vector database  
**Hvis hypotesen falsificeres:** Indikerer at native integration har begrænsninger der kræver separat system

---

## Hypotese 3: Developer Experience

**Påstand:**  
PostgreSQL understøtter standard EF Core patterns, hvilket reducerer lines of code med mindst 50% sammenlignet med MongoDB implementation.

**Evidens-grundlag:**
- Kilde 4: PostgreSQL 100% EF Core support vs MongoDB 60%
- MongoDB kræver workarounds for Include() og transactions

**Hvorfor developer experience matters:**  
For et semester-projekt med deadline påvirker code complexity direkte om jeg når i mål. Workarounds koster udviklingsdage og introducerer bugs.

### Testdesign

**Målepunkt:** Lines of code for identisk funktionalitet

**Feature 1:** Eager loading af samtaler med beskeder
```csharp
// PostgreSQL: Include() virker direkte
.Include(c => c.Messages)

// MongoDB: Manuel N+1 loop nødvendig
foreach (var conv in conversations) { ... }
```

**Feature 2:** Transaction med rollback ved fejl
```csharp
// PostgreSQL: Standard EF Core
using var transaction = await _context.Database.BeginTransactionAsync();

// MongoDB: API mixing (EF Core + MongoDB Driver)
using var session = await _mongoClient.StartSessionAsync();
```

**Måles:**
- Total LoC inklusiv workarounds
- Antal extra dependencies (IMongoClient, collections)
- API-paradigmer anvendt (LINQ vs MongoDB syntax)

**Success-kriterium:**  
```
PostgreSQL LoC ≤ 50% af MongoDB LoC
```

**Hvis hypotesen validates:** Bekræfter at mature provider reducerer code complexity  
**Hvis hypotesen falsificeres:** MongoDB's workarounds er måske mindre byrde end forventet

---

## Hypotese 4: Consistency og Data Durability

**Påstand:**  
PostgreSQL's ACID transactions sikrer at partial conversation saves ikke forekommer ved simulated crash scenarios.

**Evidens-grundlag:**
- Kilde 5: ACID atomicity garanterer "alt eller intet"
- MongoDB's eventual consistency tillader partial writes

**Hvorfor consistency matters:**  
Guest/Authenticated Chatbots skal gemme user message + bot response atomisk. Partial saves giver broken UX og potentielle GDPR-issues.

### Testdesign

**Målepunkt:** Data integrity efter simulated crash

**Scenarie:**  
System crasher mellem save af user message og bot response.

**Test procedure:**
```csharp
for (int i = 0; i < 20; i++) {
    using var transaction = await _context.Database.BeginTransactionAsync();
    
    // Save user message
    await _context.SaveChangesAsync();
    
    // Random delay (0-50ms)
    await Task.Delay(Random.Next(0, 50));
    
    // Simulated crash
    if (Random.Next(0, 2) == 0) 
        Environment.FailFast("Crash");
    
    // Save bot response
    await _context.SaveChangesAsync();
    
    await transaction.CommitAsync();
}
```

**Måles:**
- Antal crashes triggered: ~10
- Antal partial saves: Messages uden paired response
- Data integrity percentage

**Success-kriterium:**  
- PostgreSQL: **0 partial saves** (100% rollback)
- MongoDB: **> 0 partial saves** (eventual consistency issues)

**Hvis hypotesen validates:** ACID garantier holder i praksis, MongoDB's BASE accepterer partial writes  
**Hvis hypotesen falsificeres:** Indikerer fundamental misforståelse af consistency models eller test-fejl

---

## Fra hypoteser til validering

Jeg har nu fire testbare hypoteser der direkte adresserer projektets behov:

| Hypotese | Måler | Success-kriterium |
|----------|-------|-------------------|
| **H1: JSON Performance** | Query execution time | PostgreSQL < MongoDB / 20 |
| **H2: Vector Integration** | Database roundtrips | pgvector = 1 roundtrip |
| **H3: Developer Experience** | Lines of code | PostgreSQL ≤ 50% MongoDB LoC |
| **H4: Consistency** | Partial saves | PostgreSQL = 0, MongoDB > 0 |

### Hvad hvis hypoteserne falsificeres?

**H1 falsificeret:** MongoDB måske acceptabel for JSON, litteratur-fordele holder ikke  
**H2 falsificeret:** Separat vector database nødvendig, arkitektur bliver mere kompleks  
**H3 falsificeret:** MongoDB's workarounds mindre byrde end forventet  
**H4 falsificeret:** Fundamental misforståelse af consistency models, test-design fejl  

Særligt kritisk er H2 og H4. Hvis pgvector ikke kan kombinere queries effektivt, må arkitekturen inkludere separat vector database. Hvis consistency-garantier ikke holder, indikerer det enten test-fejl eller at mine antagelser om ACID/BASE er forkerte.

---

## Næste skridt: Praktisk validering

Nu skal hypoteserne testes empirisk. Jeg bygger identiske implementations i både PostgreSQL og MongoDB, genererer realistic chatbot-data, og måler performance, code complexity og data integrity.

*Holder evidensen fra researchen i praktisk test? Eller fandt jeg overraskelser?*

**Næste:** [Praktisk Test →]({{< relref "database/praktisk-test.md" >}})