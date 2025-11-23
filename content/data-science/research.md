---
title: "Research"
draft: false
weight: 2
description: "Systematisk litteratursøgning: Fra Google-frameworks til EU-lovgivning"
---

## Hvor finder man opskriften på “etisk analytics”?

I problemstillingen identificerede jeg tre centrale dilemmaer:
- Vanity metrics  
- Privacy-paradokset  
- Arkitektur-konflikten  

For at løse dem kunne jeg ikke gætte eller kopiere “best practices”. Jeg havde brug for **evidens**, dokumenteret i både forskning, lovgivning og moderne data-arkitektur.

Min research fokuserede derfor på fire kernespørgsmål:

1. **KPI Design:** Hvordan måler store virksomheder succes uden vanity metrics?
2. **Privacy:** Hvad kræver GDPR *konkret* af arkitektur og logging?
3. **Arkitektur:** Hvordan forener man real-time overvågning med tung historisk analyse?
4. **Validering:** Hvordan undgår jeg at KPI’er bliver misvisende eller skadelige?

---

## Strategi og Metodologi

Jeg anvendte **metodisk triangulering**, hvor fem uafhængige domæner undersøges, og fælles løsninger identificeres. Når teori, lovgivning og industry best practices peger i samme retning, opstår **convergent validity**.

### Inklusionskriterier
**Acceptér kilde hvis:**
- Peer-reviewed forskning (>100 citationer)
- EU-regulation eller officiel guideline
- Teknisk reference fra anerkendt autoritet (O’Reilly, ACM)
- Industrirapporter med dokumenterede cases (Amplitude, Stripe)

**Ekskludér kilde hvis:**
- Vendor marketing  
- Blog posts uden citations  
- “Best practice”-artikler uden empirisk evidens  
- Forældede eller biased whitepapers  

Denne metodiske filtrering reducerede støj og sikrede, at Design Patterns bygger på solid evidens, ikke subjektive meninger.

---

## Evidence Summary Table

| Domæne | Kilde | Main Finding | Anvendelse |
| :--- | :--- | :--- | :--- |
| **UX Metrics** | Google HEART | 5-delt UX-framework (Happiness, Engagement, etc.) | Metric Hierarchy (DP1) |
| **Business** | AARRR Pirate Metrics | Funnel til at måle produktets sundhed | Owner Dashboard (DP1/DP4) |
| **Legal** | GDPR Art. 9 & 25 | Privacy by Design, data minimization | Privacy-First Logging (DP2) |
| **Arkitektur** | Kleppmann: DDIA | Lambda Architecture (Hot/Cold paths) | Dual-Speed Analytics (DP3) |
| **Kritisk Teori** | Goodhart’s Law | Metrikker mister værdi når de bliver mål | KPI Audit Framework (DP4) |

---

## De fem kilder i dybden

## 1. Google HEART Framework (UX)

**Reference:** Rodden et al. (2010), Google Research[^1].

**Hvorfor denne kilde?**  
Chatbots fungerer dialogisk – ikke som websites. HEART måler samtalekvalitet, ikke kun aktivitet.

<details>
<summary><strong>🔍 Detaljer & Relevans</strong></summary>

**HEART består af:**
- Happiness  
- Engagement  
- Adoption  
- Retention  
- Task Success  

**Relevans:**  
- *Guest Chatbot:* Task Success er centrale  
- *Authenticated:* Retention + Task Success  
- *Owner:* Adoption & Retention på systemniveau  

HEART danner øverste del af min **Layered Metrics Pyramid**.
</details>

---

## 2. AARRR Pirate Metrics (Business)

**Reference:** McClure (2007)[^2].

**Hvorfor denne kilde?**  
Owner Dashboard skal måle forretningsværdi, ikke trafikvolumen.

<details>
<summary><strong>🔍 Detaljer & Relevans</strong></summary>

**AARRR består af:**
- Acquisition  
- Activation  
- Retention  
- Referral  
- Revenue  

**Relevans:**  
Tvinger Owner Dashboard væk fra “Total Users” vanity metrics og over mod:
- Aktivationsrate  
- Fastholdelse  
- Token-effektivitet  
</details>

---

## 3. GDPR Art. 9 & 25 (Privacy)

**Reference:** EU GDPR (2016)[^3].

**Hvorfor denne kilde?**  
Den afgør hvad jeg *må* logge – ikke bare hvad der er smart at logge.

<details>
<summary><strong>🔍 Detaljer & Relevans</strong></summary>

**Art. 9:** Forbud mod behandling af helbredsdata uden eksplicit samtykke.  
**Art. 25:** Privacy by Design → Data Minimization.

**Relevans for min arkitektur:**  
- Raw text kan ikke gemmes  
- UserID må ikke lagres i klartekst  
- Logging skal ske som:  
  `IntentCategory + SessionHash`  

Dette er fundamentet for **DP2: Privacy-First Logging**.
</details>

---

## 4. Designing Data-Intensive Applications (Arkitektur)

**Reference:** Kleppmann (2017)[^4].

**Hvorfor denne kilde?**  
Løser konflikten mellem real-time drift og historiske analyser.

<details>
<summary><strong>🔍 Detaljer & Relevans</strong></summary>

**Lambda Architecture:**
- **Hot Path:** Real-time → hurtige alarmer  
- **Cold Path:** Batch → tunge beregninger  

**Relevans:**  
Chatbotten kræver:
- real-time error alerts (Owner)
- ugentlige retention-udregninger (Product)

Det implementeres som **DP3: Dual-Speed Analytics**.
</details>

---

## 5. Goodhart’s Law (Kritisk Metrik-teori)

**Reference:** Strathern (1997)[^5].

**Hvorfor denne kilde?**  
Den afslører hvornår en KPI bliver skadelig.

<details>
<summary><strong>🔍 Detaljer & Relevans</strong></summary>

**Audit-spørgsmål:**  
1. Vil optimering mod denne metric skade brugeren?  
2. Kan metric’en gamificeres af systemet?  
3. Bør den parres med en kvalitativ kontrolmåling?  

**Eksempler:**
- “Få beskeder per session” → Gode svar? eller ultrakort bot?  
- “Session duration” → Læring? eller frustration?  

Dette bruges i **DP4: Context-Aware KPIs**.
</details>

---

## Samlet Analyse: Fra teori til Design Patterns

De fem kilder peger på én samlet, konsistent løsning. Deres roller fremgår i tabellen:

| Research Source | Design Pattern | Funktion |
|-----------------|----------------|----------|
| HEART + AARRR | **DP1: Layered Metrics Pyramid** | Definerer KPI-struktur for Guest/Auth/Owner |
| GDPR Art. 25 | **DP2: Privacy-First Logging** | Logging uden raw text, hashing ved kilden |
| Kleppmann | **DP3: Dual-Speed Analytics** | Hot Path + Cold Path |
| Goodhart's Law | **DP4: Context-Aware KPIs** | KPI-audit, undgår metric-fejl |

Gennem trianguleringen behøver jeg ikke længere gætte.  
Jeg kan nu formulere fire evidensbaserede **Design Patterns**, der løser problemstillingens dilemmaer med faglig tyngde.

---

**Næste:** [Design Patterns →]({{< relref "data-science/design-patterns.md" >}})

[^1]: Rodden, K., Hutchinson, H., & Fu, X. (2010). "Measuring the User Experience on a Large Scale". Google Research.  
[^2]: McClure, D. (2007). "Startup Metrics for Pirates: AARRR!".  
[^3]: EU GDPR (2016). Articles 9 & 25.  
[^4]: Kleppmann, M. (2017). "Designing Data-Intensive Applications". O’Reilly.  
[^5]: Strathern, M. (1997). “Improving ratings”. European Review.  
