---
title: "Samlet Refleksion"
weight: 7
description: "Portfolio-niveau oversigt over læring, cross-cutting themes og fremtidige anvendelser"
ShowToc: true
TocOpen: true
---

# Samlet Refleksion: Fra Problemløsning til Frameworks

Dette semester handlede formelt om Database & Storage + Data Science & Analytics. Men reelt handlede det om at udvikle **systematic decision-making frameworks** jeg kan anvende på ethvert teknisk problem.

Denne side binder portfolioen sammen og reflekterer over hvad jeg virkelig lærte.

---

## 10 Læringsmål: Quick Reference

Detaljeret evidens og opfyldelse findes i [Læringsmål]({{< relref "laeringsmaal/_index.md" >}}).

**Database & Storage (1-5):**
1. Vector Search Implementation → pgvector HNSW index
2. ACID vs BASE Trade-offs → Crash simulation testing (0% vs 70% failure)
3. Database-ORM Integration → EF Core provider maturity (53% mindre kode)
4. Holistisk Database-evaluering → TCO analysis ($4,400 savings)
5. Systematisk Research → 5-source triangulation

**Data Science & Analytics (6-10):**
6. KPI Framework Design → Layered Metrics Pattern (Google HEART)
7. Data Collection Strategies → Privacy-first logging (GDPR Article 25)
8. Privacy-First Analytics → Anonymization by default
9. Dashboard Design → Context-aware KPIs per chatbot type
10. Statistical Rigor → Mock data validation (framework audit)

---

## Cross-Cutting Themes

Fire temaer gennemgår begge specialiseringer:

### 1. Privacy & GDPR Er Ikke Ét Lag

**Database-niveau:**
- CASCADE DELETE (GDPR Article 17: right to deletion)
- Foreign key constraints enforcer data integrity
- No orphaned data muligt

**Analytics-niveau:**
- Data minimization (log categories, ikke raw text)
- Anonymization by default (hash user IDs)
- Aggregated metrics preserve privacy

**Learning:**  
Privacy designes ind på **alle niveauer** – database schema, logging patterns, dashboard design skal alle respektere GDPR.

---

### 2. Performance = Cost Efficiency

**Database:**
- 4× mindre disk (PostgreSQL vs MongoDB) = lavere storage cost
- 26× hurtigere queries = lavere compute cost per request
- Unified platform = ingen vector DB subscription

**Analytics:**
- Lambda Architecture: Real-time (expensive) + Batch (cheap) balance
- Materialized views: Pre-compute (disk) vs compute on-demand (CPU)
- Log retention: 30 days (compliant) vs forever (expensive + privacy risk)

**Learning:**  
Performance optimization og cost reduction er ofte aligned. Efficient = billigere.

---

### 3. Developer Experience Som Beslutningsfaktor

**Database:**
- Mature EF Core provider = 53% mindre kode
- Standard LINQ patterns virker direkte
- Workarounds (MongoDB N+1) spiser udviklingsdage

**Analytics:**
- Structured logging > custom parsing
- Type-safe queries (LINQ) > string concatenation (SQL injection risk)
- Dashboard frameworks > custom HTML/JS

**Learning:**  
Database-valg påvirker velocity. Et "hurtigere" system der kræver 2× udviklings-tid er samlet set langsommere.

---

### 4. Unified Platforms > Specialized Systems

**Database:**
- PostgreSQL (unified) > MongoDB + Pinecone (specialized)
- Simplicity gennem integration
- Færre failure points

**Analytics:**
- Layered metrics (unified framework) > ad-hoc per chatbot
- Context-aware KPIs (flexible) > separate systems per role

**Learning:**  
"Best tool for the job" betyder ikke altid specialized tools. General-purpose med exceptional implementation kan vinde.

---

## Teknisk → Meta Progression

**Technical Level:**
```
Antagelser → Research → Validation → Design → Implementation
```

**Meta Level:**
```
Ad-hoc → Systematic → Framework-based → Reusable methodology
```

Den reelle værdi: Ikke konklusionen (PostgreSQL + KPI framework), men **processen** der førte dertil.

---

## Største Overraskelser

### Specialization ≠ Performance

**Forventning:** MongoDB (document DB) bedst til documents.  
**Realitet:** PostgreSQL (general-purpose) outperformer ved relevant workloads.  
**Why:** Implementation quality > database category.

### Privacy Og Utility Er Ikke Trade-off

**Forventning:** GDPR = mindre data = dårligere insights.  
**Realitet:** Data minimization tvinger **bedre metrics** (actionable vs vanity).  
**Why:** Privacy-first er better engineering, ikke burden.

### Documentation Er Læring

**Forventning:** Portfolio = skriv hvad jeg byggede.  
**Realitet:** Documentation tvang mig til at clarify fuzzy forståelse.  
**Why:** Feynman Technique – hvis du ikke kan forklare det, forstår du det ikke.

---

## Transferable Skills

Gennem semesteret udviklede jeg kompetencer ud over Database + Analytics:

**1. Systematic Research Methodology**  
Framework anvendelig til: Framework evaluation, tool selection, architectural decisions

**2. Evidence-Based Decision Making**  
Framework anvendelig til: Build vs buy, performance priorities, team process changes

**3. Privacy-First Engineering**  
Principles anvendelig til: Any system med user data, GDPR compliance, ethical tech

---

## Fremadrettet: Anvendelse

### Praktikforløb

Jeg kan nu:
- Research nye teknologier systematisk (ikke følge hype)
- Design privacy-compliant systems from scratch
- Evaluate tools med clear criteria
- Document architectural decisions med evidens

### Job-Situationer

**Scenario:** "Skal vi migrere fra MySQL til MongoDB?"

**Før:** Google, læs 3 blog posts, stem.

**Nu:**
1. Formulér forskningsspørgsmål (performance? cost? DX?)
2. Research systematisk (benchmarks, production cases)
3. Design test på actual workload
4. Validate med success criteria
5. Document transparent

**Difference:** Evidence-based vs opinion-based.

---

## Afslutning (Kommer Efter Data Science)

Når Data Science & Analytics er færdig, udvides denne side med:
- Konkrete eksempler fra begge faser
- Deep dive på cross-specialization learnings
- Framework applications til real scenarios
- Meta-reflection på portfolioen som helhed

*Foreløbig etablerer den overordnet struktur.*

---

**Relaterede sider:**  
[Database & Storage]({{< relref "database/_index.md" >}}) | [Data Science & Analytics]({{< relref "data-science/_index.md" >}}) | [Vidensrejse]({{< relref "vidensrejse/_index.md" >}}) | [Læringsmål]({{< relref "laeringsmaal/_index.md" >}})