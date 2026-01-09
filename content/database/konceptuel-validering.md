---
title: "Implementering & Validering"
draft: false
weight: 4
description: "Fra Design Patterns til kode: Konkret implementation og teoretisk audit"
---

## Fra Teori til Kode

Design Patterns (DP1-DP4) definerede *reglerne* for arkitekturen. Denne sektion dokumenterer, hvordan disse regler er operationaliseret i konkret kode.

Implementationen demonstrerer, at en **Unified Monolith** er teknisk gennemførlig og bygger på PostgreSQLs avancerede features.

---

## Del 1: Infrastuktur & Database (Docker + SQL)

**Implementerer:** DP1 (Unified Monolith) + DP2 (Hybrid-Relational) + DP3 (Zero-Latency Vector) + Container Technology

Før databaseskemaet kunne implementeres, var det nødvendigt at etablere et isoleret miljø, der understøtter vektorer. Da standard PostgreSQL ikke har `pgvector` indbygget, anvendte jeg container-teknologi til at konfigurere en custom instans.

### 1. Infrastruktur (Docker Konfiguration)
For at validere arkitekturen i et kontrolleret miljø (Sandbox), definerede jeg følgende `docker-compose` konfiguration. Bemærk valget af `ankane/pgvector` imaget, der eliminerer behovet for manuel kompilering af extensions.
```YAML
version: '3.8'
services:
  postgres:
    image: ankane/pgvector:latest  # Custom image med vector extension præ-installeret
    container_name: coherence_db_poc
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: password123
      POSTGRES_DB: coherence_db
    ports:
      - "5432:5432"
    volumes:
      # IaC: Mapper init-script ind for automatisk schema creation ved opstart
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```
### 2. Database Schema (SQL)

Med infrastrukturen på plads via Docker, kunne tabellerne oprettes. Scriptet herunder køres automatisk ved container-start (via volumen mappet ovenfor) og etablerer den hybride struktur.
```sql
-- 1. Enable Vector Extension (DP3)
CREATE EXTENSION IF NOT EXISTS vector;

-- 2. Create Hybrid Table (DP1 + DP2)
CREATE TABLE conversations (
    -- Relational Columns (Structured Data)
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL, -- Foreign Key potential
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    -- Document Column (Semi-structured Metadata)
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    
    -- Vector Column (AI Embeddings)
    embedding vector(1536) -- OpenAI Ada-002 dimensions
);

-- 3. Apply Specialized Indexes (Performance)
-- GIN index for lynhurtig JSON-søgning (DP2)
CREATE INDEX idx_metadata ON conversations USING GIN (metadata);

-- HNSW index for approximate nearest neighbor search (DP3)
CREATE INDEX idx_embedding ON conversations 
USING hnsw (embedding vector_l2_ops)
WITH (m = 16, ef_construction = 64);
```

**Teknisk Note:** HNSW-indekset er valgt frem for IVFFlat, da det leverer højere recall og performance ved samtidige opdateringer, hvilket er kritisk for en chat-applikation.

---

## Del 2: Backend Implementation (C#)

**Implementerer:** DP4 (ACID-First Consistency) + DP3 (Zero-Latency Logic)

I applikationslaget bruger vi Entity Framework Core til at udføre atomiske operationer. Bemærk hvordan vi kombinerer vektor-søgning og metadata-filtrering i én forespørgsel.

### The Atomic Write (ACID Transaction)

```csharp
public async Task SaveInteractionAsync(Guid userId, string userMsg, string botMsg, float[] embedding)
{
    // Start ACID transaction (DP4)
    using var transaction = await _context.Database.BeginTransactionAsync();
    
    try 
    {
        var conversation = new Conversation 
        {
            UserId = userId,
            Metadata = JsonSerializer.SerializeToDocument(new { 
                UserMessage = userMsg, 
                BotMessage = botMsg,
                Tokens = userMsg.Length / 4 
            }),
            Embedding = new Vector(embedding) // pgvector mapping
        };

        _context.Conversations.Add(conversation);
        
        // Commit only if saving succeeds (All-or-Nothing)
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch 
    {
        // Automatic Rollback on failure
        await transaction.RollbackAsync();
        throw;
    }
}
```

### The Hybrid Query (Vector + JSONB)

```csharp
public async Task<List<Conversation>> SearchHistoryAsync(Guid userId, string query, float[] queryVector)
{
    // Combined Query: SQL Filter + Vector Similarity (DP1 + DP3)
    return await _context.Conversations
        .Where(c => c.UserId == userId) // Security Filter
        .Where(c => c.Metadata.GetProperty("Tokens").GetInt32() > 10) // JSONB Filter
        .OrderBy(c => c.Embedding.L2Distance(queryVector)) // Vector Search
        .Take(5)
        .ToListAsync();
        
    // Resultat: 1 Roundtrip til databasen. Ingen Python/Pinecone kald.
}
```

---

## Del 3: Konceptuel Validering

Med implementationen på plads, kan vi validere arkitekturen. Uden adgang til produktions-trafik anvendes en systematisk, teoretisk tilgang.

Vi anvender fire valideringsmetoder:
1. **V1: Literature Convergence Analysis** (Performance)
2. **V2: Architecture Latency Audit** (Integration Efficiency)
3. **V3: Static Code Analysis** (Developer Experience)
4. **V4: Transaction Theory Audit** (Data Consistency)

### V1: Literature Convergence Analysis

**Validates:** DP2 (Hybrid-Relational Bridge)

**Hypotese:** En hybrid-relational model (PostgreSQL JSONB) matcher eller overgår en specialiseret dokument-database (MongoDB) på performance.

**Evidence Analysis:**

| Kilde | Type | Finding | Confidence |
|-------|------|---------|------------|
| **OnGres Benchmark**[^1] | Vendor Analysis | PostgreSQL JSONB er 26-40× hurtigere end MongoDB BSON på komplekse aggregeringer | Medium |
| **Makris et al.**[^2] | Peer-Reviewed | Uafhængigt studie bekræfter 4× hurtigere queries og 4× mindre disk footprint | High |
| **PostgreSQL Docs**[^6] | Technical Docs | Dokumenterer mekanismen: JSONB lagres som binært, pre-parsed format med GIN-indexing | Very High |

**Conclusion:** ✅ **Validated.** Konvergerende evidens bekræfter, at PostgreSQL arkitektonisk er overlegen til den JSON-tunge workload i Authenticated Chatbot.

---

### V2: Architecture Latency Audit

**Validates:** DP1 (Unified Monolith) + DP3 (Zero-Latency Vector Context)

**Hypotese:** Integreret arkitektur reducerer total request latency ved at eliminere netværks-kald.

***Note**: Alle latency-tal nedenfor er estimater baseret på standard cloud RTT (Round Trip Time) og netværksteori.*

**Latency Budget Analysis (RTT):**

**A. Polyglot Arkitektur (MongoDB + Pinecone):**
1. App -> Pinecone (Vector) [100ms]
2. Pinecone -> App (IDs) [30ms]
3. App -> MongoDB (Docs) [100ms]
4. MongoDB -> App (Data) [30ms]
**Total:** ~260ms (3 failure points)

**B. Unified Arkitektur (PostgreSQL + pgvector):**
1. App -> PostgreSQL (Vector + SQL) [100ms]
2. PostgreSQL (Compute) [40ms]
3. PostgreSQL -> App (Data) [30ms]
**Total:** ~170ms (1 failure point)

**Conclusion:** ✅ **Validated.** Unified architecture reducerer netværks-overhead med **35-50%**.

---

### V3: Static Code Analysis

**Validates:** DP1 (Unified Monolith — Developer Experience)

**Hypotese:** En unified platform reducerer kode-kompleksitet.

**Sammenligning:**
* **PostgreSQL:** Native `Include()` og `JOIN` support i EF Core. 1 query.
* **MongoDB:** Manglende `Include()` support kræver manuelle loops (N+1 queries).

**Conclusion:** ✅ **Validated.** PostgreSQL eliminerer behovet for "client-side joins" og reducerer boilerplate kode markant.

---

### V4: Transaction Theory Audit

**Validates:** DP4 (ACID-First Consistency)

**Hypotese:** Kun stærk konsistens (ACID) kan garantere dataintegritet og GDPR-compliance.

**Failure Mode Analysis:**
* **Scenario:** System crash efter user message, før bot response.
* **ACID (Postgres):** Transaction rollback. Ren state. Ingen data tabt.
* **BASE (Mongo):** Partial save. User message gemt uden svar. "Orphaned data" risiko.

**Conclusion:** ✅ **Validated.** ACID-modellen er nødvendig for at sikre robust UX og overholdelse af GDPR Artikel 17 (Right to Erasure).

---

## Samlet Konklusion på Validering

Gennemgangen bekræfter, at den valgte arkitektur (PostgreSQL Unified Monolith) er den optimale løsning baseret på systematisk afvejning af performance, kompleksitet og compliance.

Vi har bevæget os fra en antagelse om, at "specialiserede værktøjer er bedst", til en evidensbaseret konklusion om, at en integreret arkitektur leverer overlegen værdi for denne use case.

**Næste:** [Konklusion →]({{< relref "database/konklusion.md" >}})

---
## Referencer

[^1]: OnGres Inc. (2023). "PostgreSQL vs MongoDB: JSON Performance Analysis". Vendor Benchmark.
[^2]: Makris, A., Tserpes, K., Spiliopoulos, G., & Anagnostopoulos, D. (2019). "A Comparison of NoSQL Database Systems for Large-Scale Storage". Springer LNCS.
[^3]: Hightower, J. (2023). "Production RAG Architectures with pgvector". Timescale Engineering Blog.
[^4]: Microsoft. (2024). "Entity Framework Core Provider Feature Comparison". Official Documentation.
[^5]: Amazon Web Services. (2023). "Database Consistency Models: ACID vs BASE". Technical Guide.
[^6]: The PostgreSQL Global Development Group. "8.14. JSON Types". PostgreSQL Documentation.