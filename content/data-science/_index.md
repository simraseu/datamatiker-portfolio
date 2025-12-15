---
title: "Data Science & Analytics"
draft: false
weight: 2

---

## Fra 'Log Alt' til Privacy-First Analytics

Analytics for et sundhedsrelateret chatbot-system med tre brugertyper. Men hvordan logger man brugeradfærd uden at overvåge brugerne? Hvordan måler man succes uden at gamificere metrics? Hvordan bygger man både real-time alerts og historiske trends?

Dette er historien om hvordan jeg designede et analytics-system der er både **anvendeligt og etisk forsvarligt** – uden brug af live user data.

---

## Vælg Din Tilgang

Du har to muligheder for at udforske Data Science & Analytics delen:

### 📚 Læs Hele Rejsen (5 Faser)

Følg den komplette proces fra problemidentifikation til konceptuel validering:

1. **[Problemstilling]({{< relref "data-science/problemstilling.md" >}})** – Tre dilemmaer: Vanity metrics, privacy-paradokset, arkitektur-konflikt
2. **[Research]({{< relref "data-science/research.md" >}})** – 5 kilder fra systematisk litteratursøgning (Google, EU, Kleppmann)
3. **[Design Patterns]({{< relref "data-science/design-patterns.md" >}})** – 4 normative patterns med validerings-frameworks
4. **[Implementation]({{< relref "data-science/implementation.md" >}})** – Privacy-first logging og Lambda Lite arkitektur i C#
5. **[Konklusion]({{< relref "data-science/konklusion.md" >}})** – Læring, læringsmål og samfundsperspektiver

### ⚡ Executive Summary (Herunder)

Få hele historien på 3 minutter. Scroll ned for:
- Evidens fra alle 5 kilder
- Validation resultater (DP1-DP4)
- Design beslutningen og rationale
- Læringsmål (Data Science) opfyldt

---

## Executive Summary

### Evidens: Hvad Viste Researchen?

Fem uafhængige kilder fra UX, business, GDPR, systemdesign og etik konvergerede på en fælles retning: at analytics bør designes normativt før implementation.

| Kilde | Type | Main Finding |
|-------|------|--------------|
| **Google HEART** | UX Framework | Metrics skal måle Happiness, Engagement, Adoption, Retention, Task Success |
| **AARRR Pirate Metrics** | Business Framework | Funnel approach: Acquisition → Activation → Retention → Referral → Revenue |
| **GDPR Art. 9 & 25** | EU Regulation | Sensitive data kræver Privacy by Design og anonymization-at-source |
| **Kleppmann Lambda** | Technical Book | Dual-speed analytics: Hot path (real-time) + Cold path (batch) sameksisterer |
| **Goodhart's Law** | Academic Paper | "When a measure becomes a target, it ceases to be a good measure" |

---

### Validation: Holder Patterns i Praksis?

Da jeg ikke har live-brugere, er de 4 design patterns valideret gennem en systematisk **multi-method approach**:

| Pattern | Validation Method | Resultat | Status |
|---------|-------------------|----------|--------|
| **DP1: Layered Metrics** | SMART Criteria Audit | 6/8 KPIs valid (2 vanity elimineret) | ✅ Validated |
| **DP2: Privacy-First** | GDPR Compliance Check | 5/5 articles compliant | ✅ Validated |
| **DP3: Lambda Lite** | Architecture Review | Hot <100ms, Cold <5 sec | ✅ Validated |
| **DP4: Context-Aware** | Goodhart Audit | Gaming-resistant | ✅ Validated |

**Konkrete resultater:** 6/8 KPIs SMART validated, 5/5 GDPR articles compliant (Art. 5, 9, 15, 17, 25), architectural separation verified gennem performance review.

---

### Beslutningen: Privacy-First Analytics via 4 Patterns

Den endelige løsning bygger på fire patterns – valideret, sammenhængende og 100% privacy-compliant:

✅ **Privacy by Design (DP2):** Raw text → Intent Categories før database. Ingen PII persisteres  
✅ **Context-Aware KPIs (DP4):** Guest ≠ Authenticated ≠ Owner success criteria. Goodhart-resistant  
✅ **Lambda Lite (DP3):** Hot path (<1 sec alerts) + Cold path (daily batch) sameksisterer  
✅ **Layered Metrics (DP1):** System/User/Business separation forhindrer metric pollution  

**Hvorfor "log alt" blev fravalgt:**

❌ Raw text bryder GDPR Art. 9 (særlige kategorier af personoplysninger)  
❌ Universelle metrics belønner gaming (Goodhart's Law)  
❌ Single-speed arkitektur tvinger trade-off mellem real-time og historik  

**Trade-offs accepteret:** Privacy-first forhindrer crisis detection, hour bucketing reducerer temporal precision, validation uden live data kræver teoretisk analyse. Men for MVP-context er privacy-first optimal.

---

### Læringsmål Opfyldt

- **Context-Aware KPI Design:** Differentierede KPIs baseret på HEART/AARRR. 6/8 validated, 2 vanity metrics elimineret
- **Privacy-First Data Collection:** GDPR Art. 25 gennem anonymization-at-source. 5/5 articles compliant
- **Dual-Speed Analytics:** Lambda Lite med Hot (<100ms) og Cold (<5 sec) paths. Separation verified
- **Stakeholder Dashboards:** Tre dashboards (Engineering/Product/Executive) uden cross-contamination
- **Validation Without Live Data:** Multi-method validation: SMART audit, GDPR compliance, architecture review, Goodhart audit

---

### Samfundsmæssige Perspektiver

**GDPR Compliance:** Privacy-First Logging sikrer "right to deletion" via anonymization-at-source. GDPR bliver designprincip, ikke compliance burden.

**AI Etik:** Privacy-first design forhindrer både datamisbrug og proaktiv user safety. Trade-off mellem privacy og crisis detection handler om bevidste valg.

**Modgift til Overvågningskapitalisme:** Vi logger adfærd (intents), ikke identitet. Forretningsindsigt uden at krænke privatlivets fred.

---

**Start Rejsen:** [Læs Problemstillingen →]({{< relref "data-science/problemstilling.md" >}})