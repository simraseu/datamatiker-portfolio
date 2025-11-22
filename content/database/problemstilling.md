---
title: "Problemstilling"
date: 2025-01-15
draft: false
weight: 1
---

## Fra tre chatbots til én kritisk beslutning

Vi skulle bygge en Blazor-webapp med tre AI-chatbots: én for gæster, én for brugere og én for administratoren. Men jeg stod overfor et fundamentalt spørgsmål: **Hvilken database?**

Min første tanke var simpel: "MongoDB til JSON, PostgreSQL til relationel data, Pinecone til vector search." Tre systemer. Tre databaser. Tre synkroniseringsproblemer.

Men moderne LLM'er komplicerer billedet. GPT-4 er stateless – den husker intet mellem API-kald. Derfor skal **vi** gemme al kontekst. Hver besked. Hver samtale. Hver vector embedding for semantisk søgning.

Dette projekt handler ikke om at vælge "den bedste database". Det handler om at forstå **hvornår** hver teknologi giver værdi, og hvad det koster at vælge forkert.

For at kunne træffe et fagligt begrundet databasevalg, identificerede jeg fire funktionelle krav som alle tre chatbot-typer afhænger af.

---

## Fire kritiske krav

Moderne chatbot-arkitektur stiller fire ikke-negocierbare krav:

### 1. JSON-lagring med hurtig søgning

Chatbeskeder er ikke rene SQL-rækker. De indeholder metadata (timestamps, token counts), nested content og varierende strukturer. JSON er det naturlige format, men ikke alle databaser håndterer JSON lige godt.

**Konkret krav:** Hent 50 samtaler med 500 beskeder på under 100ms.

<details>
<summary><strong>📊 Hvorfor JSON performance matters</strong></summary>

Når en authenticated user åbner deres chathistorik, skal systemet:
- Hente 50 seneste samtaler
- Loade alle beskeder per samtale
- Parse JSON-indhold
- Render i UI

Ved 26× dårligere performance (MongoDB vs PostgreSQL) betyder det:
- PostgreSQL: 50ms → flydende UX
- MongoDB: 1.3 sekunder → frustrerende ventetid

For Guest Chatbot med session-baseret loading bliver denne forskel kritisk.
</details>

---

### 2. Vector embeddings for semantisk søgning

Keyword search finder kun eksakte match. Men når en bruger spørger *"Hvordan lindrer jeg hovedpine?"*, skal systemet finde samtaler om "migræne", "smertelindring" og "Panodil" – selvom ordene er forskellige.

**Løsning:** Vector embeddings (1536-dimensionelle arrays fra OpenAI) gemmes i databasen og søges via cosine similarity.

<details>
<summary><strong>🔍 Semantic search eksempel</strong></summary>

**User query:** "Hvordan lindrer jeg hovedpine?"

**System konverterer til embedding:**
```
[0.023, -0.891, 0.445, ... 1536 dimensions]
```

**Database finder lignende embeddings fra tidligere samtaler:**
- "Jeg har migræne, hvad hjælper?" (similarity: 0.89)
- "Panodil vs Ipren til smertelindring" (similarity: 0.85)
- "Spændingshovedpine øvelser" (similarity: 0.82)

Uden vector search ville kun eksakte matches på "hovedpine" findes.
</details>

---

### 3. Mange samtidige brugere uden performance-tab

Realtidschat tolererer ikke slow queries. Ved 100+ samtidige connections skal systemet:
- Håndtere 1000+ writes/sekund (messages)
- Levere sub-100ms read latency
- Sikre consistency (ingen tabte beskeder)

---

### 4. Problemfri Blazor integration

Entity Framework Core er .NET's standard ORM. Men ikke alle database providers understøtter samme features. Manglende `Include()` support betyder N+1 queries. Ingen transaction support betyder custom workarounds.

**Krav:** 100% EF Core feature support for standard LINQ patterns.

<details>
<summary><strong>💻 Hvad betyder manglende EF Core support?</strong></summary>

**PostgreSQL (standard pattern):**
```csharp
var conversations = await _context.Conversations
    .Where(c => c.UserId == userId)
    .Include(c => c.Messages)  // ✅ Virker direkte
    .ToListAsync();
```

**MongoDB (workaround nødvendig):**
```csharp
var conversations = await _context.Conversations
    .Where(c => c.UserId == userId)
    .ToListAsync();

// ❌ Include() virker ikke - manuel loop
foreach (var conv in conversations) {
    conv.Messages = await _context.Messages
        .Where(m => m.ConversationId == conv.Id)
        .ToListAsync();  // N+1 query problem
}
```

Resultat: 2× mere kode, 11 database roundtrips i stedet for 1.
</details>

---

## Hvad sker der hvis vi vælger forkert?

Tre konkrete failure-scenarier illustrerer konsekvenserne:

### ❌ Scenarie 1: Database uden effektiv JSON-support

**Problem:** Manglende native JSON-håndtering tvinger os til omfattende normalisering med komplekse joins.

**Konsekvens:** 
- Authenticated Chatbot: 2-3 sekunders load-tid i stedet for 50ms
- Developer friction: 3× mere kode for samme funktionalitet
- Skalerbarhed: Performance degraderer eksponentielt ved vækst

<details>
<summary><strong>📉 Performance breakdown</strong></summary>

**Normalized structure (uden JSON):**
```
Messages → 500 rows
MessageMetadata → 500 rows
MessageTokens → 500 rows
= 3 tables × 500 rows = 1500 rows med JOINs
```

**JSONB structure:**
```
Messages → 500 rows med embedded JSON
= 1 table × 500 rows, ingen JOINs
```

Query complexity: O(n³) vs O(n) - massiv forskel ved scale.
</details>

---

### ❌ Scenarie 2: Ingen native vector search

**Problem:** Separat vector database (Pinecone/Weaviate) kræver:
- Data duplication (beskeder både i main DB og vector DB)
- Synkronisering mellem systemer
- Dobbelt hosting cost
- Kompleks failure handling

**Konsekvens:**
- Semantic search queries kræver 2+ database roundtrips
- Data consistency issues (sync lag)
- Årlig ekstra cost: ~$2,400 for 10k users

<details>
<summary><strong>🔗 Separate database complexity</strong></summary>

**Query flow med separat vector DB:**
1. App → Vector DB: "Find similar conversations" (200ms)
2. Vector DB → App: Returns IDs [conv1, conv2, conv3]
3. App → Main DB: "Get conversations by IDs" (150ms)
4. App: Client-side merging + filtering (50ms)

**Total: 400ms + complexity**

**Med native vector (pgvector):**
1. App → PostgreSQL: Combined query (89ms)

**Total: 89ms, zero sync issues**
</details>

---

### ❌ Scenarie 3: Manglende ACID-garantier

**Problem:** NoSQL eventual consistency kan resultere i partial saves ved crashes.

**Konsekvens:**
- Guest Chatbot: Bruger ser egen besked, men intet bot-svar
- Authenticated Chatbot: "Saved successfully" betyder ikke persistent data
- Owner Chatbot: Analytics dashboards viser inkonsistent data
- GDPR-risiko: Kan ikke garantere data-deletion

<details>
<summary><strong>⚠️ Konkret crash-scenarie</strong></summary>

**System crasher mellem user message og bot response:**

**PostgreSQL (ACID):**
- Transaction rollback automatisk
- Database returnerer til clean state
- User ser: Ingen samtale (forventet opførsel)

**MongoDB (BASE):**
- User message persisteret
- Bot response tabt
- User ser: Halv samtale (broken UX)

Test-resultat: MongoDB 70% partial saves ved 10 simulated crashes.
</details>

---

## Beslutningsmatrix

For at evaluere databaser systematisk definerede jeg vægtede kriterier baseret på projektets krav:

| Kriterium | Vægt | Hvorfor det matters |
|-----------|------|---------------------|
| **AI/ML Kompatibilitet** | 30% | Vector search + JSON er kernefunktionalitet |
| **Performance** | 25% | Realtidschat tolererer ikke slow queries |
| **Blazor Integration** | 20% | EF Core support = udviklingshastighed |
| **Skalerbarbarhed** | 15% | Prototype → produktion uden rewrite |
| **Omkostninger** | 10% | Drift + udvikling skal være bæredygtigt |

<details>
<summary><strong>🎯 Hvorfor denne vægtning?</strong></summary>

**AI/ML top priority (30%):**
Manglende vector support eliminerer semantic search helt. Dårlig JSON performance gør Authenticated Chatbot ubrugeligt. Dette er go/no-go kriterier.

**Performance #2 (25%):**
Chatbot UX lever eller dør på response time. 50ms vs 2 sekunder er forskellen mellem "flydende" og "frustrerende".

**Integration matters (20%):**
Et semester-projekt har begrænset tid. EF Core workarounds spiser udviklingsdage og introducerer bugs.

**Skalerbarhed + Cost lavere (15% + 10%):**
Vigtigt, men kan håndteres senere. Prototype kan køre på billig tier. Production scaling er fremtidig concern.
</details>

---

## Min tilgang

I stedet for at vælge baseret på "hvad folk anbefaler", besluttede jeg at kombinere:

1. **Systematisk research** → Peer-reviewed studier + production cases
2. **Praktisk test** → Benchmarks på realistic data
3. **Arkitektur-design** → Konkret implementation

Målet var ikke at finde "den bedste database", men at forstå **trade-offs** og kunne forsvare valget med evidens.

---

## Næste skridt: Indsamle evidens

Nu skulle jeg finde pålidelige kilder der kunne besvare mine fire forskningsspørgsmål. Men hvilke kilder var troværdige? Og hvordan undgik jeg confirmation bias?

*Hvordan undersøgte jeg disse krav systematisk? Hvilke overraskende resultater fandt jeg?*

**Næste:** [Research →]({{< relref "database/research.md" >}})