---
title: "Database Design"
draft: false
weight: 5
description: "Konkret arkitektur baseret på testresultater"
ShowToc: true
TocOpen: true
---

## Fra test til implementering

Jeg har nu valideret at PostgreSQL outperformer MongoDB på alle metrics. Men en database-beslutning er ikke kun et teoretisk valg – den skal omsættes til konkret arkitektur der faktisk fungerer i Blazor-applikationen.

Hvordan struktureres tabeller? Hvilke indexes? Hvordan håndterer jeg de tre forskellige chatbot-typers behov samtidig? Og hvordan sikrer jeg at designet ikke bare fungerer i teorien, men også i praksis med EF Core?

Dette afsnit dokumenterer den konkrete database-arkitektur fra schema-design til index-strategi.

---

## Overordnet Arkitektur-Beslutning

**Valgt løsning:** PostgreSQL 16.1 med pgvector extension som unified platform.

**Hvad dette betyder:**

✅ **Single database system**
- Alle data (users, conversations, messages, embeddings) i ét system
- Ingen separat vector database
- Ingen separat document store

✅ **Hybride capabilities**
- Relationelle strukturer: Foreign keys, ACID transactions
- JSON documents: JSONB kolonner for flexible data
- Vector embeddings: Native pgvector datatype

✅ **Eliminerer kompleksitet**
- ❌ Ingen data duplication
- ❌ Ingen cross-system synkronisering
- ❌ Ingen eventual consistency issues
- ❌ Ingen multiple database connections i kode

**Trade-offs accepteret:**

📊 **PostgreSQL kræver mere initial setup**  
MongoDB Atlas er "click and deploy", PostgreSQL kræver extension installation og configuration. Men investering betales tilbage gennem lavere operational overhead.

📊 **pgvector ~20% langsommere end Pinecone ved pure vector search**  
Men projektets use case kombinerer metadata-filtering med vector search. Her vinder unified platform på total latency (1 roundtrip vs 2+).

**Designprincipper:**

1. **Shared core schema** – Users, Conversations, Messages deles på tværs af alle chatbot-typer
2. **Chatbot-specific optimization** – Indexes og views tilpasset hver chatbot's query patterns
3. **EF Core first** – Schema designet til at fungere problemfrit med Entity Framework
4. **Performance by default** – Indexes på alle foreign keys og frequent query paths

---

## Core Database Schema

Fire primære tabeller danner fundamentet for alle tre chatbot-typer:

---

### 1. Users: Brugerprofiler og authentication

**Formål:** Authentication, profil-data, owner identification

**Key decisions:**
- UUID primary keys (globally unique identifiers)
- `is_owner` flag for Owner Chatbot access control
- Email unique constraint + index for hurtig login

<details>
<summary><strong>📊 SQL schema og design rationale</strong></summary>

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(100) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    is_owner BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_users_email ON users(email);
```

**Design rationale:**

**UUID primary keys:**  
Sikrer globally unique identifiers der fungerer på tværs af distributed scenarios og forenkler frontend state management.

**is_owner flag:**  
Identificerer administrator-brugere der har adgang til Owner Chatbot's analytics og system-wide data.

**Email index:**  
Understøtter hurtig login-lookup ved authentication flow.
</details>

---

### 2. Conversations: Samtale-containers

**Formål:** Grupperer beskeder, tracker metadata, understøtter både guest og authenticated sessions

**Key decisions:**
- JSONB metadata for fleksibel chatbot-specific data
- Partial index kun på guest sessions (space optimization)
- ON DELETE CASCADE for GDPR compliance

<details>
<summary><strong>📊 SQL schema og design rationale</strong></summary>

```sql
CREATE TABLE conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    is_guest_session BOOLEAN DEFAULT FALSE,
    session_id VARCHAR(100),
    metadata JSONB
);

CREATE INDEX idx_conversations_user_updated 
ON conversations(user_id, updated_at DESC);

CREATE INDEX idx_conversations_session 
ON conversations(session_id) 
WHERE is_guest_session = TRUE;

CREATE INDEX idx_conversations_metadata 
ON conversations USING GIN (metadata);
```

**Design rationale:**

**JSONB metadata kolonne:**  
Tillader fleksibel storage uden schema changes:
- Guest Chatbot: Session preferences (language, theme)
- Authenticated Chatbot: User settings (notification preferences)
- Owner Chatbot: Cached computed metrics (avg response time)

**Partial index på session_id:**  
Kun guest sessions indexeres (WHERE clause). Reducerer index size ved at ekskludere authenticated conversations.

**ON DELETE CASCADE:**  
Automatic cleanup når users slettes – kritisk for GDPR "right to deletion". Alle conversations + messages fjernes atomisk.

**GIN index på JSONB:**  
Hurtig søgning i metadata felter via queries som:
```sql
WHERE metadata->>'category' = 'medical'
```
</details>

---

### 3. Messages: Individuelle chatbeskeder

**Formål:** Lagrer beskeder, embeddings for semantic search, token tracking

**Key decisions:**
- Native vector(1536) datatype via pgvector
- HNSW index for similarity search (95-98% recall)
- Asynchronous embedding generation (decoupled fra message creation)

<details>
<summary><strong>📊 SQL schema og design rationale</strong></summary>

```sql
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
    role VARCHAR(20) NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
    content TEXT NOT NULL,
    timestamp TIMESTAMP DEFAULT NOW(),
    token_count INTEGER,
    embedding vector(1536)
);

CREATE INDEX idx_messages_conversation_time 
ON messages(conversation_id, timestamp);

CREATE INDEX idx_messages_embedding 
ON messages USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**Design rationale:**

**Vector embedding kolonne:**  
pgvector's native VECTOR datatype (1536 dimensions matcher OpenAI's text-embedding-3-small). Embeddings genereres asynchronously via background job – ikke under message creation – så chatbot response time ikke påvirkes af OpenAI API latency (200-500ms).

**HNSW index parameters:**
- `m = 16`: Max connections per layer (balance recall vs speed)
- `ef_construction = 64`: Build-time effort (højere = bedre recall)
- Leverer 95-98% recall ved 10-100× bedre performance end exhaustive search (valideret i H2)

**CHECK constraint på role:**  
Sikrer kun gyldige værdier ('user', 'assistant', 'system'). Database-level validation forhindrer data corruption fra application bugs.

**Composite index (conversation_id, timestamp):**  
Optimeret til hyppig query pattern: "Hent alle beskeder for samtale X, sorteret efter tid"

**Asynchronous embedding workflow:**
```csharp
// Step 1: Gem message (synchronous)
var message = new Message { Content = userInput };
await _context.SaveChangesAsync();

// Step 2: Queue embedding generation (asynchronous)
await _backgroundJobs.Enqueue<GenerateEmbeddingJob>(
    job => job.Execute(message.Id));
```

Messages uden embeddings filtreres fra semantic search indtil embedding er klar.
</details>

---

### 4. Analytics Events: Metrics for Owner Chatbot

**Formål:** Log system events, track performance metrics, business intelligence data

**Key decisions:**
- BIGSERIAL (ikke UUID) for space efficiency ved high volume
- JSONB event_data for flexible metric storage
- Events bevares ved user deletion (historical analytics)

<details>
<summary><strong>📊 SQL schema og design rationale</strong></summary>

```sql
CREATE TABLE analytics_events (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    user_id UUID REFERENCES users(id),
    conversation_id UUID REFERENCES conversations(id),
    event_data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_analytics_type_time 
ON analytics_events(event_type, created_at DESC);
```

**Design rationale:**

**BIGSERIAL primary key:**  
Auto-increment integer (ikke UUID) fordi analytics kan vokse til millioner af events. BIGSERIAL er mere space-efficient (~8 bytes vs ~16 bytes for UUID) og hurtigere ved sequential inserts.

**JSONB event_data:**  
Fleksibel storage af event-specific metrics uden schema changes:
```json
{
  "response_time_ms": 342,
  "token_usage": 156,
  "model": "gpt-4-turbo",
  "error": null,
  "user_satisfaction": 4
}
```

**Compound index (type, time):**  
Understøtter Owner Chatbot queries: "Vis alle 'message_sent' events fra sidste uge"

**No CASCADE on user deletion:**  
Analytics events bevares for historical business intelligence, selv efter user deletion. Personal identifiers anonymiseres via GDPR-compliant data retention policy.
</details>

---

## Relationships Visualiseret
```
User (1) ──────< (N) Conversation (1) ──────< (N) Message
  │                     │                           │
  │                     │                    embedding (vector)
  │                     │
  └──< (N) AnalyticsEvent
```

**Cascade behavior:**
- ✅ Delete User → Cascade deletes Conversations → Cascade deletes Messages
- ✅ Delete Conversation → Cascade deletes Messages
- ❌ AnalyticsEvents bevares (historical data)

---

## Chatbot-Specific Optimizations

Hver chatbot-type har unikke query patterns der kræver tailored optimization:

---

### Guest Chatbot: Session-baseret temporary storage

**Behov:** Midlertidige samtaler med automatic cleanup efter 24 timer

**Key optimizations:**
- Partial index kun på guest sessions (space reduction)
- Scheduled cleanup function (cron job hver time)
- Session-level queries uden user authentication overhead

<details>
<summary><strong>🔧 SQL implementation og rationale</strong></summary>

**Session cleanup function:**
```sql
CREATE OR REPLACE FUNCTION cleanup_guest_sessions()
RETURNS void AS $$
BEGIN
    DELETE FROM conversations
    WHERE is_guest_session = TRUE
      AND updated_at < NOW() - INTERVAL '24 hours';
END;
$$ LANGUAGE plpgsql;
```

**Partial index for efficient lookups:**
```sql
CREATE INDEX idx_conversations_session 
ON conversations(session_id) 
WHERE is_guest_session = TRUE;
```

**Design rationale:**

**24-timers retention:**  
Balance mellem user experience (multi-page sessions) og storage costs. Guest data er temporary by design.

**Partial index:**  
Kun guest sessions indexeres. Reducerer index size betydeligt da authenticated conversations udgør majority over tid.

**Cascade deletion:**  
Når conversations slettes af cleanup function, fjernes messages automatisk via ON DELETE CASCADE. Ingen orphaned data.

**Query pattern:**
```csharp
// Hent guest session
var conversation = await _context.Conversations
    .Where(c => c.SessionId == sessionId && c.IsGuestSession)
    .Include(c => c.Messages)
    .FirstOrDefaultAsync();
```

No user_id filter nødvendig, session_id er sufficient.
</details>

---

### Authenticated Chatbot: Full-text + semantic search

**Behov:** Persistent storage, semantic search i historik, hurtig loading af seneste samtaler

**Key optimizations:**
- GIN index på JSONB metadata (flexible queries)
- HNSW index på embeddings (semantic search)
- Materialized view for conversation summaries (pre-computed aggregations)

<details>
<summary><strong>🔧 SQL implementation og rationale</strong></summary>

**GIN index for metadata queries:**
```sql
CREATE INDEX idx_conversations_metadata 
ON conversations USING GIN (metadata);
```

Understøtter queries:
```sql
WHERE metadata->>'category' = 'medical'
  AND metadata->>'priority' = 'high'
```

**HNSW vector index:**
```sql
CREATE INDEX idx_messages_embedding 
ON messages USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

**Materialized view for "similar conversations":**
```sql
CREATE MATERIALIZED VIEW user_conversation_summaries AS
SELECT 
    c.id,
    c.user_id,
    c.title,
    c.updated_at,
    (SELECT embedding FROM messages 
     WHERE conversation_id = c.id 
     ORDER BY timestamp DESC 
     LIMIT 1) as latest_embedding,
    (SELECT COUNT(*) FROM messages 
     WHERE conversation_id = c.id) as message_count
FROM conversations c
WHERE is_guest_session = FALSE;

CREATE UNIQUE INDEX ON user_conversation_summaries(id);
```

**Refresh strategy:**
```sql
-- Incremental refresh hver 5. minut via cron
REFRESH MATERIALIZED VIEW CONCURRENTLY user_conversation_summaries;
```

**Design rationale:**

**Materialized view benefits:**  
Pre-computed conversation summaries reducerer query complexity for "find similar conversations" feature. I stedet for at compute embeddings + counts on-the-fly, læses pre-computed data.

**CONCURRENTLY refresh:**  
Tillader queries under refresh – ingen downtime. Kræver UNIQUE index på view.

**5-minutters refresh interval:**  
Balance mellem data freshness og compute overhead. Conversations opdateres ikke hyppigere end dette i normal usage.

**Combined metadata + vector query:**
```sql
SELECT * FROM user_conversation_summaries
WHERE user_id = $1
  AND updated_at > NOW() - INTERVAL '30 days'
ORDER BY latest_embedding <-> $2::vector
LIMIT 10;
```

Single query kombinerer metadata filtering med vector similarity – valideret i H2.
</details>

---

### Owner Chatbot: Analytics + aggregations

**Behov:** Real-time dashboards, historical trend analysis, performance metrics på tværs af alle users

**Key optimizations:**
- Materialized view for pre-aggregated metrics (sekunder → millisekunder)
- Table partitioning for analytics events (millions of rows)
- Compound indexes på event_type + timestamp

<details>
<summary><strong>🔧 SQL implementation og rationale</strong></summary>

**Pre-aggregated analytics view:**
```sql
CREATE MATERIALIZED VIEW chatbot_analytics AS
SELECT 
    DATE_TRUNC('day', c.created_at) as date,
    c.is_guest_session,
    COUNT(DISTINCT c.id) as conversation_count,
    COUNT(DISTINCT c.user_id) as unique_users,
    AVG((SELECT COUNT(*) FROM messages WHERE conversation_id = c.id)) 
        as avg_messages_per_conversation,
    AVG((SELECT SUM(token_count) FROM messages WHERE conversation_id = c.id)) 
        as avg_tokens_per_conversation,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY c.updated_at - c.created_at) 
        as median_conversation_duration
FROM conversations c
GROUP BY DATE_TRUNC('day', c.created_at), c.is_guest_session;

CREATE INDEX idx_analytics_date ON chatbot_analytics(date DESC);
```

**Table partitioning for analytics_events:**
```sql
CREATE TABLE analytics_events_partitioned (
    LIKE analytics_events INCLUDING ALL
) PARTITION BY RANGE (created_at);

-- Monthly partitions (auto-created via maintenance script)
CREATE TABLE analytics_events_2024_11 
PARTITION OF analytics_events_partitioned
FOR VALUES FROM ('2024-11-01') TO ('2024-12-01');

CREATE TABLE analytics_events_2024_12 
PARTITION OF analytics_events_partitioned
FOR VALUES FROM ('2024-12-01') TO ('2025-01-01');
```

**Compound index for event queries:**
```sql
CREATE INDEX idx_analytics_events_type_time 
ON analytics_events(event_type, created_at DESC);
```

**Design rationale:**

**Materialized view for dashboards:**  
Aggregations på large datasets er compute-intensive. Owner dashboard ville tage 5-10 sekunder uden pre-computation. Med materialized view: ~50-100ms.

**Table partitioning:**  
Analytics events vokser til millioner. Partitioning sikrer:
- Queries kun scanner relevante partitions (partition pruning)
- Efficient purging af old data (DROP partition i stedet for DELETE)
- Vedligeholdelse af indexes er faster (per-partition)

**Monthly partition strategy:**  
Balance mellem antal partitions og partition size. Daily partitions ville give for mange, yearly for store.

**Compound index (type, time):**  
Understøtter Owner queries: "Vis 'message_sent' events fra sidste uge"
```sql
WHERE event_type = 'message_sent'
  AND created_at > NOW() - INTERVAL '7 days'
```

**Query pattern for dashboard:**
```csharp
var metrics = await _context.ChatbotAnalytics
    .Where(a => a.Date >= startDate && a.Date <= endDate)
    .OrderByDescending(a => a.Date)
    .ToListAsync();
```

Pre-computed data = instant dashboard load.
</details>

---

## Integration med Blazor og EF Core

Database-designet skal fungere problemfrit med .NET-økosystemet gennem Entity Framework Core.

---

### DbContext Konfiguration

**Key setup:**
- Npgsql provider for PostgreSQL
- Custom type mappings for vector + JSONB
- Indexes defineret i OnModelCreating
- Connection pooling optimeret for concurrent users

<details>
<summary><strong>💻 DbContext implementation</strong></summary>

```csharp
public class ChatbotDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Conversation> Conversations { get; set; }
    public DbSet<Message> Messages { get; set; }
    public DbSet<AnalyticsEvent> AnalyticsEvents { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Vector type mapping
        modelBuilder.Entity<Message>()
            .Property(m => m.Embedding)
            .HasColumnType("vector(1536)");
        
        // JSONB mapping
        modelBuilder.Entity<Conversation>()
            .Property(c => c.Metadata)
            .HasColumnType("jsonb");
        
        modelBuilder.Entity<AnalyticsEvent>()
            .Property(a => a.EventData)
            .HasColumnType("jsonb");
        
        // Indexes
        modelBuilder.Entity<Conversation>()
            .HasIndex(c => new { c.UserId, c.UpdatedAt });
        
        modelBuilder.Entity<Conversation>()
            .HasIndex(c => c.SessionId)
            .HasFilter("is_guest_session = true");
        
        // Relationships
        modelBuilder.Entity<Conversation>()
            .HasOne<User>()
            .WithMany()
            .HasForeignKey(c => c.UserId)
            .OnDelete(DeleteBehavior.Cascade);
        
        modelBuilder.Entity<Message>()
            .HasOne<Conversation>()
            .WithMany(c => c.Messages)
            .HasForeignKey(m => m.ConversationId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

**Configuration i Program.cs:**
```csharp
builder.Services.AddDbContext<ChatbotDbContext>(options =>
    options.UseNpgsql(
        connectionString,
        npgsqlOptions => {
            npgsqlOptions.UseVector();  // pgvector support
            npgsqlOptions.CommandTimeout(30);
        }
    ));
```

**Connection pooling:**
```
MinPoolSize=10
MaxPoolSize=100
```

Baseret på forventet concurrent user load (10-100 simultaneous connections).
</details>

---

### Repository Pattern

**Formål:** Abstrahere database-detaljer fra Blazor components, centralisere queries, simplificere testing

**Key interfaces:**
- `IConversationRepository` – CRUD operations + semantic search
- `IMessageRepository` – Message persistence + embedding queries
- `IAnalyticsRepository` – Event logging + metrics aggregation

<details>
<summary><strong>💻 Repository implementation eksempel</strong></summary>

```csharp
public interface IConversationRepository
{
    Task<Conversation> CreateAsync(Guid userId, bool isGuest, string? sessionId = null);
    Task<List<Conversation>> GetRecentAsync(Guid userId, int count);
    Task<List<Conversation>> FindSimilarAsync(Guid userId, float[] embedding, int limit);
    Task<Conversation?> GetByIdAsync(Guid id, bool includeMessages = false);
    Task UpdateAsync(Conversation conversation);
    Task DeleteAsync(Guid id);
}

public class ConversationRepository : IConversationRepository
{
    private readonly ChatbotDbContext _context;
    
    public ConversationRepository(ChatbotDbContext context)
    {
        _context = context;
    }
    
    public async Task<List<Conversation>> GetRecentAsync(Guid userId, int count)
    {
        return await _context.Conversations
            .Where(c => c.UserId == userId && !c.IsGuestSession)
            .OrderByDescending(c => c.UpdatedAt)
            .Take(count)
            .Include(c => c.Messages)
            .ToListAsync();
    }
    
    public async Task<List<Conversation>> FindSimilarAsync(
        Guid userId, 
        float[] embedding, 
        int limit)
    {
        // pgvector semantic search
        return await _context.Conversations
            .FromSqlInterpolated($@"
                SELECT * FROM conversations
                WHERE user_id = {userId}
                  AND is_guest_session = false
                  AND updated_at > NOW() - INTERVAL '30 days'
                ORDER BY embedding <-> {embedding}::vector
                LIMIT {limit}")
            .Include(c => c.Messages.OrderBy(m => m.Timestamp))
            .ToListAsync();
    }
}
```

**Benefits:**
- Blazor components kun afhænger af interfaces (testable)
- Database queries centraliserede (maintainable)
- Vector search logic abstracted (simplified for consumers)
</details>

---

### Transaction Boundaries

**Strategi:** ACID transactions for critical operations, non-transactional for background jobs

**Transactional operations:**
- ✅ User message + bot response (atomic save)
- ✅ Conversation creation + first message
- ✅ User registration + welcome conversation

**Non-transactional operations:**
- ⚪ Embedding generation (background job)
- ⚪ Analytics aggregation (eventual consistency acceptable)
- ⚪ Materialized view refresh

<details>
<summary><strong>💻 Transaction implementation patterns</strong></summary>

**Critical atomic operation:**
```csharp
public async Task SaveConversationTurnAsync(
    Guid conversationId, 
    string userMessage, 
    string botResponse)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        // User message
        var userMsg = new Message 
        { 
            ConversationId = conversationId,
            Role = "user",
            Content = userMessage,
            Timestamp = DateTime.UtcNow
        };
        _context.Messages.Add(userMsg);
        await _context.SaveChangesAsync();
        
        // Bot response
        var botMsg = new Message 
        { 
            ConversationId = conversationId,
            Role = "assistant",
            Content = botResponse,
            Timestamp = DateTime.UtcNow
        };
        _context.Messages.Add(botMsg);
        await _context.SaveChangesAsync();
        
        // Update conversation timestamp
        var conversation = await _context.Conversations
            .FindAsync(conversationId);
        conversation.UpdatedAt = DateTime.UtcNow;
        
        await transaction.CommitAsync();
        
        // AFTER commit: Queue embedding generation
        await _backgroundJobs.Enqueue<GenerateEmbeddingJob>(
            job => job.Execute(userMsg.Id));
        await _backgroundJobs.Enqueue<GenerateEmbeddingJob>(
            job => job.Execute(botMsg.Id));
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

**Background job (no transaction):**
```csharp
public async Task GenerateEmbeddingAsync(Guid messageId)
{
    var message = await _context.Messages.FindAsync(messageId);
    
    // Call OpenAI API
    var embedding = await _openAiClient.GetEmbeddingAsync(message.Content);
    
    // Update message (single operation, no transaction needed)
    message.Embedding = embedding;
    await _context.SaveChangesAsync();
}
```

**Rationale:**

**Transactions for user-facing operations:**  
Valideret i H4 – atomic saves forhindrer partial data ved crashes. Critical for user trust.

**No transactions for background jobs:**  
Embedding generation kan tage 200-500ms (OpenAI API latency). Long-running transactions ville holde locks og reducere concurrency. Eventual consistency er acceptable – messages uden embeddings filtreres simpelthen fra semantic search indtil embedding er klar.

**Connection pooling benefit:**  
Non-transactional background jobs frigiver connections hurtigere, hvilket øger throughput for user-facing requests.
</details>

---

## Designet i Overblik

Database-arkitekturen implementerer findings fra research og test-faserne:

**Core beslutninger:**
- ✅ PostgreSQL 16.1 som unified platform
- ✅ pgvector extension for vector embeddings
- ✅ JSONB kolonner for flexible metadata
- ✅ ACID transactions for critical operations
- ✅ Shared schema med chatbot-specific optimizations

**Løser alle fire kritiske krav fra problemstillingen:**

| Krav | Løsning | Valideret i |
|------|---------|-------------|
| **JSON lagring** | JSONB med GIN indexes | H1: 26× hurtigere |
| **Vector search** | pgvector HNSW native | H2: 1 roundtrip |
| **Mange users** | Connection pooling + indexes | Test setup |
| **Blazor integration** | EF Core 100% support | H3: 47% mindre kode |

**Arkitektoniske fordele:**

🎯 **Single system complexity**  
Ingen separate databases at synkronisere, monitere eller maintaine.

🎯 **ACID guarantees**  
100% data integrity valideret i H4 (0% partial saves).

🎯 **Development velocity**  
Standard EF Core patterns = hurtigere development, færre bugs.

🎯 **Cost efficiency**  
4× mindre disk space end MongoDB = lavere hosting costs.

---

## Fra Arkitektur til Refleksion

Jeg har nu gennemført hele rejsen: problemstilling → research → hypoteser → test → design. Database-valget er truffet og arkitekturen dokumenteret.

Men hvad betyder dette større perspektiv? Hvordan kobler det til mine læringsmål? Og hvilke samfundsmæssige implikationer har de tekniske valg?

*Hvad lærte jeg af hele processen? Og hvordan påvirker databasevalg mere end bare teknisk performance?*

**Næste:** [Konklusion →]({{< relref "database/konklusion.md" >}})