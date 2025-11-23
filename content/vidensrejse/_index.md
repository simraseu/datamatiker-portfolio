---
title: "Vidensrejse"
weight: 5
description: "Hvordan jeg tilegnede mig viden gennem research, iteration og refleksion"
ShowToc: true
TocOpen: true
---

# Vidensrejse: Hvordan Lærte Jeg Systematisk?

Portfolio documentation handler ikke kun om **hvad jeg byggede**, men om **hvordan jeg lærte**. Denne side dokumenterer min progression fra ad-hoc Googling til systematisk evidens-baseret beslutningstagning.

Fra "MongoDB lyder godt" til "her er 5 peer-reviewed kilder der konvergerer mod PostgreSQL."

---

## Min Læringsprogression i 5 Faser

### Fase 1: Starting Point (Uge 1-2)

**Hvad vidste jeg IKKE?**
- Forskellen mellem JSONB og BSON implementation
- At vector search kunne være native i PostgreSQL
- Hvordan man evaluerer kilder systematisk
- At ACID ≠ dårlig performance

**Initial assumptions:**
- MongoDB = bedst til JSON (document database specialisering)
- Vector search = kræver Pinecone (specialized tool)
- Flere databaser = mere fleksibilitet

**Research strategi udviklet:**
- Formulér konkrete forskningsspørgsmål
- Definer inclusion/exclusion criteria
- Søg systematisk (ikke bare Google første side)

*Detaljeret documentation kommer efter Data Science-fasen er færdig.*

---

### Fase 2: Database Research (Uge 3-5)

**Systematisk litteratursøgning:**
- Google Scholar for peer-reviewed artikler
- IEEE Xplore for tekniske studier
- GitHub for reproducerbare benchmarks
- Microsoft Learn for official dokumentation

**Kildekritik metodologi:**
- Hvem finansierede researchen? (vendor bias check)
- Er metodologi reproducerbar?
- Peer-reviewed eller self-published?
- Kan findings trianguleres fra multiple sources?

**Triangulering proces:**
- Kilde 1 (OnGres): PostgreSQL 25-40× hurtigere, men vendor-sponsoreret
- Kilde 2 (Makris peer-reviewed): Uafhængig bekræftelse
- Kilde 3-5: Convergerende evidens = høj confidence

**Key learning:**  
Peer-reviewed > vendor research > blog posts. Men vendor research med offentlig kildekode + independent validation = credible.

[Se fuld research →]({{< relref "database/research.md" >}})

---

### Fase 3: Data Science Research (Uge 6-8)

**Byggede på Database-viden:**
- Genbrugte source evaluation framework fra DB-fasen
- Samme triangulering-approach
- Allerede confident i systematisk proces

**GDPR fokus fra start:**
- Database-fasen lærte mig CASCADE DELETE for "right to deletion"
- Anvendte privacy-thinking fra dag 1 i analytics design
- Data minimization som default (ikke afterthought)

**Industry patterns vs akademisk:**
- Færre peer-reviewed papers om chatbot KPIs
- Pivoterede til production cases (Google HEART, Amplitude)
- Official guidelines (GDPR Article 25, EU data protection)

**Key learning:**  
Framework-based thinking (Google HEART, Lambda Architecture) > ad-hoc metric design. Patterns er transferable.

[Se Data Science research →] (kommer snart)

---

### Fase 4: Integration & Iteration (Uge 9-10)

**Cross-specialization learnings:**

Privacy gennemgår begge:
- Database: CASCADE DELETE (GDPR Article 17)
- Analytics: Data minimization, anonymization (GDPR Article 25)

Performance vs cost trade-offs:
- Database: 4× storage efficiency = lavere costs
- Analytics: Lambda Architecture (real-time vs batch balance)

Developer experience:
- Database: Mature EF Core provider = hurtigere development
- Analytics: Structured logging > custom parsing

**Portfolio documentation som læring:**
- Feynman Technique: Kan jeg forklare det simpelt?
- Hvis ikke, forstår jeg det ikke dybt nok
- Documentation tvang mig til at destillere kompleksitet

**Hvad ville jeg gøre anderledes?**
- Start med research framework FØRST (ikke discover det halvvejs)
- Document assumptions eksplicit (nemmere at challenge)
- Iterate hurtigere (spike implementations tidligere)

---

### Fase 5: Meta-Refleksion (Løbende)

**Research methodology som transferable skill:**

Framework jeg nu har:
```
1. Formulér konkrete forskningsspørgsmål
2. Definer evaluation criteria
3. Søg systematisk (Scholar, IEEE, GitHub)
4. Evaluer kilder med credibility framework
5. Triangulér evidens fra multiple sources
6. Dokumentér limitations transparent
```

**Systematic decision-making:**

Ikke bare "hvad virker?" men:
- Hvad er mine evaluation criteria?
- Hvilken evidens supporterer dette?
- Hvad er limitations?
- Kan jeg forsvare valget med data?

**Self-directed learning:**

Fra "følg tutorials" til:
- Identify credible sources selv
- Build mental models gennem experimentation
- Validate forståelse ved at kunne forklare simpelt
- Document learnings for future reference

---

## Frameworks Udviklet

Gennem semesteret byggede jeg reusable frameworks:

### Source Evaluation Checklist

**For hver kilde:**
- [ ] Hvem publicerede? (vendor, akademisk, community)
- [ ] Peer-reviewed eller self-published?
- [ ] Er metodologi reproducerbar?
- [ ] Hvem finansierede researchen?
- [ ] Er limitations eksplicit dokumenteret?
- [ ] Kan findings verificeres elsewhere?

### Hypothesis Testing Template
```
Hypotese: [Specifik, målbar påstand]
Evidens-grundlag: [Kilder der supporterer]
Success criteria: [Numerisk threshold]
Test design: [Reproducerbar metodologi]
Validering: [Passed/Failed med rationale]
```

### Decision Documentation Pattern
```
Problemstilling → Research → Patterns → Implementation → Validation → Reflection
```

Domain-agnostic og transferable til fremtidige projekter.

---

## Næste Skridt

Når Data Science & Analytics er færdig, vil denne side blive udvidet med:
- Konkrete eksempler fra begge faser
- Iterations-historier (mistakes made, lessons learned)
- Tools anvendt (Zotero for citations, Obsidian for notes)
- Deep dive på source evaluation konkrete cases

*Foreløbig dokumenterer den overordnede progression.*

---

**Relaterede sider:**  
[Database Research]({{< relref "database/research.md" >}}) | [Data Science Research](kommer snart) | [Samlet Refleksion]({{< relref "refleksion/_index.md" >}})